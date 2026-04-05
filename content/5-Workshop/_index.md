---
title: "Workshop"
date: 2026-01-05
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# SmartHire AI — Backend Candidate Flow Workshop

This workshop guides you through building the **SmartHire AI Backend (Candidate Flow)** — a serverless, event-driven CV processing and candidate evaluation system on AWS. By the end, you will have a fully functional pipeline that receives CV uploads, extracts text, analyzes content using Generative AI, and delivers results in real-time.

![SmartHire AI Architecture](/images/5-Workshop/smarthire_architecture.png)

### Table of Contents

**5.1.** [Overview](5.1-Workshop-overview/)

**5.2.** [Prerequisites](5.2-Prerequisite/)

**5.3.** [Foundation Setup — Storage & Queue](5.3-foundation/)

&emsp; 5.3.1. [Create S3 Bucket for CV Uploads](5.3-foundation/5.3.1-s3-bucket/)

&emsp; 5.3.2. [Configure S3 Bucket Policy & CORS](5.3-foundation/5.3.2-s3-policy/)

&emsp; 5.3.3. [Create SQS Queue & Dead Letter Queue](5.3-foundation/5.3.3-sqs-queue/)

**5.4.** [IAM Roles & Policies](5.4-iam/)

&emsp; 5.4.1. [Create Lambda Execution Roles](5.4-iam/5.4.1-lambda-roles/)

&emsp; 5.4.2. [Create S3 Event Notification Policy](5.4-iam/5.4.2-s3-event-policy/)

**5.5.** [Backend API — SAM Deployment](5.5-backend-api/)

&emsp; 5.5.1. [Configure SAM Template](5.5-backend-api/5.5.1-sam-template/)

&emsp; 5.5.2. [API Gateway & Cognito Authorizer](5.5-backend-api/5.5.2-apigw-cognito/)

&emsp; 5.5.3. [Deploy .NET 8 Lambda Function](5.5-backend-api/5.5.3-deploy-lambda/)

**5.6.** [Event-Driven Pipeline — CV Processing](5.6-pipeline/)

&emsp; 5.6.1. [S3 Event Notification → SQS](5.6-pipeline/5.6.1-s3-event/)

&emsp; 5.6.2. [Ingestion Trigger Lambda](5.6-pipeline/5.6.2-ingestion-trigger/)

&emsp; 5.6.3. [CV/JD Processor Lambda (Textract + Bedrock)](5.6-pipeline/5.6.3-cv-processor/)

**5.7.** [AI Evaluation — Step Functions Workflow](5.7-step-functions/)

&emsp; 5.7.1. [Job Suggestion Engine](5.7-step-functions/5.7.1-job-suggestion/)

&emsp; 5.7.2. [Candidate Ranking Engine](5.7-step-functions/5.7.2-candidate-ranking/)

**5.8.** [Infrastructure as Code — Terraform](5.8-terraform/)

&emsp; 5.8.1. [Terraform Modules Overview](5.8-terraform/5.8.1-modules/)

&emsp; 5.8.2. [Deploy with Terraform](5.8-terraform/5.8.2-deploy/)

**5.9.** [Verify & Test](5.9-verify/)

**5.10.** [Cleanup](5.10-cleanup/)

**5.11.** [Appendices — Source Code](5.11-appendices/)
