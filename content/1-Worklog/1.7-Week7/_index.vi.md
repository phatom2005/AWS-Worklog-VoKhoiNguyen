---
title: "Worklog Tuần 7"
date: 2026-02-16
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu tuần 7:
* Hiểu luồng xử lý bất đồng bộ: SQS → IngestionTrigger → Step Functions.
* Đọc code Lambda `ingestion_trigger.py` và `cv_jd_processor.py`.
* Tìm hiểu DynamoDB (Net Store) dùng để track trạng thái CV và lưu JD vector cache.

### Các công việc trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành |
|-----|-----------|--------------|-----------------|
| 2 | Cài đặt `CVService.PublishCVParseQueueAsync` — gửi message `{profileId, fileKey, jobId, jdText}` vào SQS `cv-upload-queue` | 16/02/2026 | 16/02/2026 |
| 3 | Đọc hiểu `ingestion_trigger.py`: validate file S3 qua HeadObject, fetch JD từ DynamoDB, route đến Step Functions | 17/02/2026 | 17/02/2026 |
| 4 | Tìm hiểu DynamoDB Single Table Design: `PK=CANDIDATE#<id>, SK=CV#<key>` (status tracking); `PK=JOB#<id>, SK=METADATA` (JD vector cache) | 18/02/2026 | 18/02/2026 |
| 5 | Đọc hiểu luồng Step Functions → `cv_jd_processor` → Routing Choice → `JobSuggestionEngine` / `CandidateRankingEngine` | 19/02/2026 | 19/02/2026 |
| 6 | Test end-to-end: upload CV → SQS → IngestionTrigger → DynamoDB status `PROCESSING` → Step Functions start | 20/02/2026 | 20/02/2026 |

### Kết quả đạt được tuần 7:
* Hiểu rõ 3 bước của IngestionTrigger: (1) parse SQS payload, (2) validate file S3 qua HeadObject, (3) `save_status_to_dynamodb` → `start_processing`.
* `start_processing` ưu tiên gọi `sfn.start_execution(STATE_MACHINE_ARN)` nếu được cấu hình; fallback sang Direct Invoke `vector_ops`.
* DynamoDB lưu trạng thái `PROCESSING` → `DONE` cho từng CV; lưu cache `jdVector` (Cohere Embed) để tái sử dụng khi nhiều ứng viên nộp cùng job.
* Luồng upload → SQS → IngestionTrigger → Step Functions hoạt động đúng trên môi trường test.
