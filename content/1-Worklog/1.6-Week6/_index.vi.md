---
title: "Worklog Tuần 6"
date: 2026-02-09
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Mục tiêu tuần 6:
* Xây dựng endpoint upload CV lên S3 (CV Bucket).
* Cài đặt tạo Presigned URL cho phép FE tự PUT file lên S3.
* Tạo `CandidateProfile` với status `PROCESSING` sau khi upload.

### Các công việc trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành |
|-----|-----------|--------------|-----------------|
| 2 | Cài đặt `CVService.UploadToS3Async` — PutObject lên S3 CV Bucket (`cvs/{userId}/{guid}.pdf`) | 09/02/2026 | 09/02/2026 |
| 3 | Cài đặt `CVService.CreateProfileAsync` — INSERT `candidate_profiles` (status: PROCESSING) | 10/02/2026 | 10/02/2026 |
| 4 | Cài đặt `CVService.GeneratePresignedUploadUrlAsync` — tạo Presigned PUT URL (5 phút) | 11/02/2026 | 11/02/2026 |
| 5 | Cài đặt `POST /api/cv/upload` và `POST /api/cv/presigned-url` trong CVController | 12/02/2026 | 12/02/2026 |
| 6 | Validate file: chỉ nhận PDF/DOC/DOCX, giới hạn 10MB; test với Postman | 13/02/2026 | 13/02/2026 |

### Kết quả đạt được tuần 6:
* `POST /api/cv/upload`: nhận file multipart, upload S3, tạo profile, trả về `202 Accepted` + `profileId`.
* `POST /api/cv/presigned-url`: trả về presigned URL để FE tự PUT lên S3, tạo profile `PROCESSING` ngay khi cấp URL.
* Validate đầu vào: extension, kích thước file, xác thực Cognito JWT từ API Gateway.
* File key theo format `cvs/{userId}/{guid}.ext` khớp với filter của S3 Event Notification trong Terraform.
