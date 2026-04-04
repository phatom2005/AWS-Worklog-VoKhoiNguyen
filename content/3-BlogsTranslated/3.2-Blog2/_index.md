---
title: "Blog 2 - Automating AWS Systems Manager Activation with Lambda & DynamoDB"
date: 2026-03-09
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# Automating AWS Systems Manager Activation for Hybrid Node Registration

> **Source:** [Automate AWS Systems Manager activation for hybrid-managed node registration](https://aws.amazon.com/blogs/mt/automate-aws-systems-manager-activation-for-hybrid-managed-node-registration/) — AWS Cloud Operations Blog
>
> **Author:** Justin Thomas (Sr. Cloud Support Engineer, AWS Premium Support)

---

## Introduction

**AWS Systems Manager (SSM)** is an AWS service that lets you view and control servers both on AWS and in your on-premises infrastructure. To bring servers and VMs in a hybrid environment under Systems Manager management, you need to create a **Managed Instance Activation** — which provides an Activation Code and Activation ID.

The problem: manually creating and managing Hybrid Activations credentials is tedious and error-prone. When an Activation Code expires or reaches its registration limit, administrators must manually recreate it and reconfigure affected servers. This post presents a solution that fully automates this process using **API Gateway + Lambda + DynamoDB**.

---

## Solution Architecture

The solution is deployed via **AWS CloudFormation** and consists of the following components:

| AWS Service | Role |
|-------------|------|
| **Amazon API Gateway** | Private REST API integrated with Lambda. On-premises servers send a GET request to receive the Activation Code/ID |
| **AWS Lambda** | Provides the Hybrid Activations Code/ID to the server. Automatically creates a new activation code if the existing one is expired or has reached its registration limit |
| **Amazon DynamoDB** | Tracks processing state: Lambda sets the table to `Locked` while serving a request, and `Unlocked` after completing it |
| **Amazon VPC Endpoint** | Allows private access to the API Gateway URL from the on-premises network |
| **AWS Systems Manager Parameter Store** | Stores the Activation ID and Activation Code, encrypted with KMS |

### Execution Flow

1. The web client (on-premises server) calls the private API Gateway endpoint.
2. The on-premises DNS server forwards the request to VPC DNS to resolve the VPC Endpoint's private IP.
3. The request is sent to the private IP of the VPC Endpoint.
4. The API Gateway Resource Policy verifies the request originates from the VPC Endpoint — otherwise it is rejected (403 Forbidden).
5. API Gateway passes the request to Lambda.
6. Lambda updates the DynamoDB state key to `Locked`.
7. Lambda retrieves the credentials from Parameter Store and returns them to the client.

---

## Deployment Steps

### Step 1: Create a VPC Endpoint for API Gateway

Create an Interface VPC Endpoint for API Gateway in your VPC, with a Security Group allowing TCP port 443.

{{% notice tip %}}
If a VPC Endpoint for API Gateway already exists in your VPC, skip this step and note the existing VPC Endpoint ID.
{{% /notice %}}

### Step 2: Create a KMS Key

Create an **AWS KMS Key** (symmetric) to encrypt the data stored in Parameter Store (Activation Code and Activation ID).

### Step 3: Create the API Gateway and Lambda

Deploy a CloudFormation stack to create the Private API Gateway and Lambda function. Once the stack is complete, copy the **API Gateway Invoke URL** from the Outputs section.

Test from your on-premises server:

```bash
curl -s https://<invoke-url>/lambdastage | jq '.'
```

Expected JSON response:

```json
{
  "ActivationId": "e50a8437-23dd-4326-9e79-5e3b7573493e",
  "ActivationCode": "vVcH9zJX4ROy2XTsh5cb"
}
```

---

## Automated SSM Agent Registration Scripts

### Linux (Shell Script)

```bash
#!/bin/bash
credentials=$(curl -s https://<invoke-url>/lambdastage)
activationcode=$(echo $credentials | jq -r '.ActivationCode')
activationid=$(echo $credentials | jq -r '.ActivationId')

mkdir /tmp/ssm
curl https://s3.amazonaws.com/ec2-downloads-windows/SSMAgent/latest/linux_amd64/amazon-ssm-agent.rpm \
  -o /tmp/ssm/amazon-ssm-agent.rpm
sudo dnf install -y /tmp/ssm/amazon-ssm-agent.rpm
sudo systemctl stop amazon-ssm-agent
sudo amazon-ssm-agent -register -code $activationcode -id $activationid -region us-east-1
sudo systemctl start amazon-ssm-agent
```

### Windows (PowerShell)

```powershell
$credential = Invoke-WebRequest -URI https://<invoke-url>/lambdastage | Select-Object -ExpandProperty Content
$credentialPSObject = $credential | ConvertFrom-Json
$code = $credentialPSObject.ActivationCode
$id   = $credentialPSObject.ActivationId

Start-Process .\AmazonSSMAgentSetup.exe -ArgumentList @("/q", "/log", "install.log", "CODE=$code", "ID=$id", "REGION=us-east-1") -Wait
```

---

## Cleanup

1. Deregister servers from Systems Manager.
2. Delete the `apigateway-lambda-setup` CloudFormation stack.
3. Delete the `apigateway-vpcendpoint-setup` CloudFormation stack.
4. Delete the KMS Key created during the walkthrough.

---

## Relevance to SmartHire AI

This blog illustrates a **serverless pattern** that is directly applied in SmartHire AI:

| Pattern in this blog | Equivalent in SmartHire AI |
|----------------------|---------------------------|
| API Gateway → Lambda | CV upload API → Lambda generates Presigned URL |
| Lambda updates DynamoDB state (`Locked` / `Unlocked`) | Lambda updates DynamoDB state (`PROCESSING` / `DONE`) |
| Parameter Store stores secrets | SmartHire uses Secrets Manager / Lambda env vars |
| Private API Gateway protects internal endpoints | Cognito Authorizer protects SmartHire endpoints |
| CloudFormation automates deployment | SAM template automates Lambda + API Gateway deployment |

---

## Conclusion

This solution demonstrates the power of the **event-driven serverless pattern**: Lambda handles business logic, DynamoDB manages state, Parameter Store protects secrets, and API Gateway serves as a secure communication gateway. This is the foundational pattern that SmartHire AI applies throughout its CV Parsing Pipeline and AI Evaluation Workflow.
