---
title: "Worklog Tuần 5"
date: 2026-02-02
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5:
* Thiết kế schema database cho candidate flow.
* Cài đặt AuthService tích hợp Amazon Cognito.
* Hoàn thiện luồng đăng ký, xác nhận email và đăng nhập.

### Các công việc trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành |
|-----|-----------|--------------|-----------------|
| 2 | Thiết kế bảng `users`, `candidate_profiles` (Model, Migration EF Core) | 02/02/2026 | 02/02/2026 |
| 3 | Cài đặt `AuthService.RegisterAsync` — SignUp Cognito User Pool, sync `CognitoSub` vào DB | 03/02/2026 | 03/02/2026 |
| 4 | Cài đặt `AuthService.LoginAsync` — InitiateAuth (USER_PASSWORD_AUTH) + sync user DB | 04/02/2026 | 04/02/2026 |
| 5 | Cài đặt `EmailService`: xác nhận email (ConfirmSignUp), gửi lại OTP (ResendConfirmationCode) | 05/02/2026 | 05/02/2026 |
| 6 | Test với Postman: register → confirm email → login → nhận JWT (AccessToken / IdToken) | 06/02/2026 | 06/02/2026 |

### Kết quả đạt được tuần 5:
* Bảng `users` và `candidate_profiles` được tạo qua EF Core Migration.
* `AuthService` hoàn chỉnh: đăng ký user lên Cognito User Pool, tự động sync `CognitoSub` vào DB.
* `EmailService` xác nhận email qua OTP và gửi lại code khi cần.
* Luồng register → confirm → login → JWT hoạt động ổn định trên môi trường dev.
