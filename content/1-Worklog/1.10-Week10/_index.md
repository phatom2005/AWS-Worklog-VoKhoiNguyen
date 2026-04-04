---
title: "Week 10 Worklog"
date: 2026-03-09
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Week 10 Objectives:
* Finalize and deploy the SAM template (`template.yaml`).
* Test all APIs via API Gateway (Prod stage).
* Configure CORS and Cognito Authorizer for API Gateway.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date |
|-----|-----------|--------------|-----------------|
| Mon | Review `template.yaml`: configure CORS, Cognito Authorizer, VPC Config (private subnet) | 03/09/2026 | 03/09/2026 |
| Tue | Configure Lambda environment variables: DB_HOST, Cognito params, ADMIN_EMAIL | 03/10/2026 | 03/10/2026 |
| Wed | Build & deploy SAM: `sam build` → `sam deploy --guided` | 03/11/2026 | 03/11/2026 |
| Thu | Test endpoints via API Gateway URL (Prod stage) using Postman | 03/12/2026 | 03/12/2026 |
| Fri | Fix cold start, timeout, CORS issues; optimize MemorySize (512MB) and Timeout (30s) | 03/13/2026 | 03/13/2026 |

### Week 10 Achievements:
* Successfully deployed SAM: Lambda (.NET 8) running behind API Gateway with Cognito Authorizer.
* API Gateway Prod stage is functioning with CORS allowing the `https://smarthire-ai.org` origin.
* Lambda is placed within the VPC private subnet, successfully connecting to RDS PostgreSQL via RDS Proxy.
* All candidate flow APIs passed testing in the actual cloud environment.