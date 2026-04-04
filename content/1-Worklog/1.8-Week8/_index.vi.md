---
title: "Worklog Tuần 8"
date: 2026-02-23
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Mục tiêu tuần 8:
* Tìm hiểu AWS AppSync và cơ chế GraphQL Subscription.
* Đọc và hiểu `appsync_notifier.py` — module gửi real-time notification từ Lambda.
* Tìm hiểu SNS cho Management & Observability (CloudWatch Alarm → SNS).

### Các công việc trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành |
|-----|-----------|--------------|-----------------|
| 2 | Tìm hiểu AWS AppSync: GraphQL Schema, Resolver, Subscription, IAM SigV4 auth | 23/02/2026 | 23/02/2026 |
| 3 | Đọc hiểu `appsync_notifier.py`: hàm `notify_candidate_job_suggestions` và `notify_recruiter_candidate_ranking` | 24/02/2026 | 24/02/2026 |
| 4 | Hiểu GraphQL Mutations: `publishJobSuggestions` (→ candidate) và `publishCandidateRanking` (→ recruiter) trigger Subscription tương ứng | 25/02/2026 | 25/02/2026 |
| 5 | Tìm hiểu SNS: dùng cho Management & Observability — CloudWatch Alarm → SNS Topic → email/alert | 26/02/2026 | 26/02/2026 |
| 6 | Test AppSync mutation bằng Postman + AWS Console; kiểm tra Subscription nhận event real-time | 27/02/2026 | 27/02/2026 |

### Kết quả đạt được tuần 8:
* Hiểu cơ chế AppSync Subscription: FE subscribe `onJobSuggestions(candidateId)` và `onCandidateRanking(jobId)` — khi Lambda gọi mutation, FE nhận dữ liệu real-time qua WebSocket được AppSync quản lý.
* `appsync_notifier.py` dùng SigV4 thuần stdlib (không cần thêm dependency) để authenticate với AppSync endpoint.
* SNS được dùng cho observability: nhận alert từ CloudWatch Alarm khi Lambda error rate tăng hoặc SQS DLQ có message.
* Tách rõ 2 notification channel: **Candidate** nhận kết quả job suggestion; **Recruiter** nhận danh sách ứng viên đã rank.
