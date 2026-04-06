---
title : "Infrastructure Components"
date : 2026-01-05
weight : 6
chapter : false
pre : " <b> 5.7. </b> "
---

### A. Các hàm Lambda

| Lambda | Nguồn (Source) | Timeout | Bộ nhớ | VPC | Mục đích |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `ingestion-trigger` | `iac/lambda/ingestion_trigger/` | 30s | 256 MB | Không | Xác thực tải lên CV từ S3, khởi chạy Step Functions |
| `cv-jd-processor` | `iac/lambda/cv_jd_processor/` | 300s | 1024 MB | Có | Xử lý CV/JD thống nhất (Textract/PII/Claude/Embed) |
| `job-suggestion-engine` | `iac/lambda/job_suggestion_engine/` | 90s | 3008 MB | Có | Tìm các công việc phù hợp nhất cho ứng viên |
| `candidate-ranking-engine` | `iac/lambda/candidate_ranking_engine/` | 90s | 3008 MB | Có | Xếp hạng các ứng viên phù hợp nhất cho công việc |

### B. Các tệp Terraform

| Mục đích | Đường dẫn (Path) |
| :--- | :--- |
| Điều phối gốc (Thông báo S3, module xử lý) | `iac/terraform/main.tf` |
| Lambdas của Pipeline + Step Functions + DynamoDB + AppSync API/Resolvers | `iac/terraform/modules/processing/main.tf` |
| GraphQL schema của AppSync | `iac/terraform/modules/processing/appsync_schema.graphql` |
| Mẫu ASL | `iac/terraform/modules/processing/state_machine_cv_processing.asl.json` |
| Các đầu ra (Outputs) của quá trình xử lý | `iac/terraform/modules/processing/outputs.tf` |
| Hàng đợi SQS | `iac/terraform/modules/queue/main.tf` |

### C. Các Tài nguyên Đã gỡ bỏ

- Lambda `text_processor` (đã gộp vào `cv_jd_processor`)
- Lambda `vector_ops` (đã gộp vào `cv_jd_processor`)
- Thông báo của S3 bucket cho tiền tố JD (`jobs/*.pdf`)