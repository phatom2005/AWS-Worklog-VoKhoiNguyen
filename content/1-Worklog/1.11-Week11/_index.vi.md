---
title: "Worklog Tuần 11"
date: 2026-03-16
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

### Mục tiêu tuần 11:
* Hiểu và tham gia triển khai hạ tầng bằng Terraform.
* Rà soát các module liên quan đến candidate flow: storage, queue, processing.
* Đảm bảo hạ tầng IaC đồng bộ với code đã viết.

### Các công việc trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành |
|-----|-----------|--------------|-----------------|
| 2 | Rà soát module `storage`: S3 CV Bucket, lifecycle, prefix `cvs/`, `jds/` | 16/03/2026 | 16/03/2026 |
| 3 | Rà soát module `queue`: SQS `cv-upload-queue`, DLQ, S3 Event Notification filter `.pdf` | 17/03/2026 | 17/03/2026 |
| 4 | Rà soát module `processing`: Lambda IngestionTrigger, Step Functions, `cv_jd_processor`, DynamoDB Net Store; ECR cho AI engines | 18/03/2026 | 18/03/2026 |
| 5 | Tham gia `terraform plan` / `terraform apply` cùng team | 19/03/2026 | 19/03/2026 |
| 6 | Kiểm tra ECS task (ai-core), AppSync endpoint, CloudWatch + X-Ray hoạt động sau apply | 20/03/2026 | 20/03/2026 |

### Kết quả đạt được tuần 11:
* Xác nhận Terraform module `storage`, `queue`, `processing` đồng bộ với code BE.
* S3 Event Notification gắn đúng queue ARN, filter prefix `cvs/` và suffix `.pdf`.
* ECS Task (ai-core), AppSync, CloudWatch Alarm → SNS hoạt động sau `terraform apply`.
* Hiểu toàn bộ hạ tầng IaC và vai trò từng module trong kiến trúc diagram.
