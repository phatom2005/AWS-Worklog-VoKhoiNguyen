---
title: "Worklog Tuần 12"
date: 2026-03-23
weight: 12
chapter: false
pre: " <b> 1.12. </b> "
---

### Mục tiêu tuần 12:
* Kiểm thử end-to-end toàn bộ candidate flow trên môi trường production.
* Sửa lỗi phát sinh, tối ưu hiệu năng.
* Hoàn thiện tài liệu API, viết báo cáo thực tập.

### Các công việc trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành |
|-----|-----------|--------------|-----------------|
| 2 | Test end-to-end: register → login → upload CV → SQS → IngestionTrigger → Step Functions → AI Engines → AppSync notification | 23/03/2026 | 23/03/2026 |
| 3 | Test edge case: file lớn >50MB, format không hợp lệ, token hết hạn, xóa CV người khác | 24/03/2026 | 24/03/2026 |
| 4 | Debug lỗi phát sinh: Lambda timeout, lỗi CORS, DLQ message, AppSync SigV4 auth | 25/03/2026 | 27/03/2026 |
| 5 | Hoàn thiện Swagger doc, README và tài liệu API endpoint candidate flow | 28/03/2026 | 01/04/2026 |
| 6 | Viết báo cáo thực tập, tổng hợp worklog, review với mentor | 02/04/2026 | 04/04/2026 |

### Kết quả đạt được tuần 12:
* Toàn bộ candidate flow hoạt động ổn định trên prod: upload CV → SQS → IngestionTrigger → Step Functions → cv_jd_processor → JobSuggestionEngine + CandidateRankingEngine → AppSync push real-time.
* Xử lý đầy đủ edge case: validate file, phân quyền xóa CV, token hết hạn, DLQ retry.
* Tài liệu API đầy đủ (Swagger), README cập nhật với hướng dẫn deploy SAM + Terraform.
* Báo cáo thực tập hoàn chỉnh, worklog 12 tuần được tổng hợp và review.
