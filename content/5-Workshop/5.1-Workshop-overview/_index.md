---
title: "Overview"
date: 2026-01-05
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

# Overview

## What We Are Building

**SmartHire AI Backend (Candidate Flow)** is a serverless system that automates two critical HR processes:

1. **CV Upload & Parsing Pipeline** — Candidates upload CVs directly to S3 via Presigned URLs. An event-driven pipeline automatically extracts text (Amazon Textract) and analyzes skills/experience using Generative AI (Amazon Bedrock).

2. **AI Evaluation Workflow** — AWS Step Functions orchestrates parallel scoring of interview answers, generating standardized reports for HR.

## Architecture

The system follows a clear separation between the **API layer** (synchronous, .NET 8 on Lambda) and **background processing** (asynchronous, Python Lambda workers):

![SmartHire AI Architecture](/images/5-Workshop/smarthire_architecture.png)

### Data Flow

**CV Upload Flow:**
```
Frontend → API Gateway → Lambda (.NET 8) → Generate S3 Presigned URL
                                          ↓
Candidate → PUT file directly to S3 (bypass server)
                                          ↓
S3 Event Notification → SQS Queue → Ingestion Trigger Lambda
                                          ↓
                              CV/JD Processor Lambda
                              (Textract + Bedrock)
                                          ↓
                              DynamoDB (parsed results)
                                          ↓
                              AppSync Notification → Frontend
```

**AI Evaluation Flow:**
```
Recruiter triggers evaluation → API Gateway → Lambda (.NET 8)
                                                    ↓
                                          Step Functions Workflow
                                          ├── Job Suggestion Engine (Cohere Embed v3)
                                          └── Candidate Ranking Engine (Claude 3.5 Sonnet)
                                                    ↓
                                          DynamoDB + S3 (report)
                                                    ↓
                                          AppSync Notification → Frontend
```

### AWS Services Used

| Service | Role |
|---------|------|
| **Amazon S3** | CV storage, Presigned URL generation, report output |
| **Amazon SQS** | Message queue with DLQ for CV processing pipeline |
| **AWS Lambda** | .NET 8 API (SAM) + Python workers (Terraform) |
| **Amazon Textract** | OCR — extract text from PDF/DOCX |
| **Amazon Bedrock** | LLM (Claude 3.5 Sonnet) for CV analysis & scoring |
| **AWS Step Functions** | Orchestrate parallel evaluation workflow |
| **Amazon DynamoDB** | Store parsed CV data, processing status, evaluation results |
| **Amazon API Gateway** | REST API with Cognito Authorizer |
| **Amazon Cognito** | User authentication (candidates, HR, recruiters) |
| **Amazon ECR** | Container registry for Python Lambda images |
| **Amazon RDS (PostgreSQL)** | Primary relational database for application data |

### Infrastructure as Code

The project uses two IaC tools:

- **AWS SAM** (`backend/template.yaml`) — Deploys the .NET 8 API Lambda + API Gateway
- **Terraform** (`iac/terraform/`) — Deploys all supporting infrastructure: networking, storage, queue, processing, auth, database, CI/CD

#### Terraform Modules

| Module | Resources |
|--------|-----------|
| `networking` | VPC, subnets, security groups, NAT, VPC endpoints |
| `database` | RDS PostgreSQL, Bastion host, RDS Proxy, Secrets Manager |
| `auth` | Cognito User Pool, Cognito Lambda triggers |
| `storage` | S3 bucket for raw assets (CVs, JDs) |
| `queue` | SQS standard queue + DLQ for CV processing |
| `processing` | Ingestion trigger, CV/JD processor, Job Suggestion Engine, Candidate Ranking Engine, DynamoDB, Step Functions, CloudWatch alarms |
| `ecr` | ECR repositories for Lambda container images |
| `frontend` | CloudFront distribution, S3 static hosting |
| `dns` | Route53 hosted zone, ACM certificate |
| `iam` | GitHub OIDC provider, deployment roles |
| `cicd` | CodePipeline for AI Lambda auto-deployment |
