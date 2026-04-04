---
title: "Week 11 Worklog"
date: 2026-03-16
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

### Week 11 Objectives:
* Understand and participate in infrastructure deployment using Terraform.
* Review candidate flow-related modules: storage, queue, processing.
* Ensure the IaC infrastructure syncs with the written code.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date |
|-----|-----------|--------------|-----------------|
| Mon | Review `storage` module: S3 CV Bucket, lifecycle, `cvs/` and `jds/` prefixes | 03/16/2026 | 03/16/2026 |
| Tue | Review `queue` module: SQS `cv-upload-queue`, DLQ, S3 Event Notification filter `.pdf` | 03/17/2026 | 03/17/2026 |
| Wed | Review `processing` module: IngestionTrigger Lambda, Step Functions, `cv_jd_processor`, DynamoDB Net Store; ECR for AI engines | 03/18/2026 | 03/18/2026 |
| Thu | Participate in `terraform plan` / `terraform apply` with the team | 03/19/2026 | 03/19/2026 |
| Fri | Verify ECS task (ai-core), AppSync endpoint, CloudWatch + X-Ray are functioning after apply | 03/20/2026 | 03/20/2026 |

### Week 11 Achievements:
* Confirmed the Terraform `storage`, `queue`, and `processing` modules synchronize with the BE code.
* S3 Event Notifications are correctly attached to the queue ARN, filtering with the `cvs/` prefix and `.pdf` suffix.
* ECS Task (ai-core), AppSync, CloudWatch Alarms → SNS operate correctly after `terraform apply`.
* Fully understood the IaC infrastructure and the role of each module in the architecture diagram.