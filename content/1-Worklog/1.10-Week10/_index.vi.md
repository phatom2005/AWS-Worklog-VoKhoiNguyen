---
title: "Worklog Tuần 10"
date: 2026-03-09
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Mục tiêu tuần 10:
* Hoàn thiện và deploy SAM template (`template.yaml`).
* Kiểm tra toàn bộ API qua API Gateway (Prod stage).
* Cấu hình CORS, Cognito Authorizer cho API Gateway.

### Các công việc trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành |
|-----|-----------|--------------|-----------------|
| 2 | Review `template.yaml`: cấu hình CORS, Cognito Authorizer, VPC Config (private subnet) | 09/03/2026 | 09/03/2026 |
| 3 | Cấu hình biến môi trường Lambda: DB_HOST, Cognito params, ADMIN_EMAIL | 10/03/2026 | 10/03/2026 |
| 4 | Build & deploy SAM: `sam build` → `sam deploy --guided` | 11/03/2026 | 11/03/2026 |
| 5 | Test các endpoint qua API Gateway URL (Prod stage) với Postman | 12/03/2026 | 12/03/2026 |
| 6 | Fix lỗi cold start, timeout, CORS; tối ưu MemorySize (512MB) và Timeout (30s) | 13/03/2026 | 13/03/2026 |

### Kết quả đạt được tuần 10:
* SAM deploy thành công: Lambda (.NET 8) chạy sau API Gateway với Cognito Authorizer.
* API Gateway Prod stage hoạt động với CORS cho phép origin `https://smarthire-ai.org`.
* Lambda nằm trong VPC private subnet, kết nối được RDS PostgreSQL qua RDS Proxy.
* Tất cả API candidate flow pass test trên môi trường cloud thực tế.
