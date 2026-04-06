---
title: "Workshop"
date: 2026-01-05
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Kiến trúc Hệ thống Ghép nối SmartHire (v3.0 — Luồng xử lý Thống nhất)

#### Tổng quan

**SmartHire-AI** là một nền tảng tuyển dụng thông minh được xây dựng trên AWS, mang đến sự đột phá trong quy trình tuyển dụng. Ứng viên tải lên CV và nhận được các đề xuất công việc được cá nhân hóa, trong khi nhà tuyển dụng quản lý các bài đăng công việc và xem danh sách ứng viên được xếp hạng theo thời gian thực. Nền tảng này tận dụng AI và các dịch vụ serverless của AWS để đảm bảo khả năng mở rộng và hiệu suất tối đa.

#### Các Tính năng Chính

- **Dành cho Ứng viên**: Đăng nhập, tải lên CV (định dạng PDF) và nhận các đề xuất công việc với thông tin cập nhật theo thời gian thực.
- **Dành cho Nhà tuyển dụng**: Tạo và quản lý các bài đăng công việc, xem danh sách ứng viên được xếp hạng ngay khi họ ứng tuyển.
- **Cập nhật Thời gian thực**: Tính năng GraphQL subscriptions của AWS AppSync đẩy (push) kết quả ngay lập tức.
- **Ghép nối bằng AI**: Bedrock, Comprehend và Textract hỗ trợ sức mạnh cho quá trình ghép nối thông minh.

### Các Thay đổi Chính

- **Không Tải lên JD bằng PDF**: Văn bản Mô tả Công việc (Job Description - JD) được đọc trực tiếp từ RDS (`Jobs.Description`) thay vì phải tải tệp PDF lên S3.
- **Hợp nhất Lambda Xử lý**: Hai hàm `text_processor` và `vector_ops` được gộp chung thành một Lambda duy nhất là `cv_jd_processor`.
- **Kích hoạt JD Trực tiếp**: Backend .NET gọi trực tiếp `StartExecution` của Step Functions để xử lý JD (bỏ qua bước `S3/SQS/IngestionTrigger`).
- **Đơn giản hóa State Machine**: Số bước của Step Functions giảm từ 4 trạng thái xuống còn 3 (`CvJdProcessor → RoutingChoice → Engine`).
- **Giảm Độ trễ**: Giảm bớt một lần khởi động lạnh (cold start) của Lambda cho mỗi lần thực thi (do 2 Lambda đã được gộp thành 1).
- **Giao tiếp Thời gian thực qua AppSync**: Các công cụ ghép nối (matching engines) phát (publish) các bản cập nhật thông qua GraphQL mutations của AppSync, và frontend nhận dữ liệu thông qua các subscriptions đã được lọc.

### Giải thích Nhanh (Góc nhìn Đơn giản)
Nếu bạn chỉ cần nhớ 4 điều, hãy ghi nhớ những điều sau:

1. Ứng viên tải lên CV -> hệ thống phân tích CV -> tính toán các công việc phù hợp nhất -> đẩy thông báo theo thời gian thực đến giao diện (UI) của ứng viên.
2. Nhà tuyển dụng tạo công việc -> hệ thống tạo vector nhúng (embed) cho JD -> xếp hạng các ứng viên hàng đầu -> đẩy thông báo theo thời gian thực đến UI của nhà tuyển dụng.
3. Step Functions luôn luôn có 3 bước: `CvJdProcessor -> RoutingChoice -> Engine`.
4. AppSync chỉ đóng vai trò là lớp phân phối thời gian thực (Lambda phát dữ liệu, frontend đăng ký nhận dữ liệu), chứ không phải là lớp tính toán.

### Giao ước API Thời gian thực (Hiện tại)
- Mutation: `publishJobSuggestions(candidateId, suggestions, updatedAt)`
- Mutation: `publishCandidateRanking(jobId, rankedCandidates, updatedAt)`
- Subscription: `onJobSuggestions(candidateId)`
- Subscription: `onCandidateRanking(jobId)`

Lưu ý Quan trọng: trường (field) trong schema được sử dụng là `updatedAt` (không phải `timestamp`).

#### Nội dung

1. [Quy trình Hệ thống](5.1-System-workflows/)
2. [Luồng Dữ liệu Chi tiết](5.2-Detailed-data-flows/)
3. [Mô hình Dữ liệu & Mối quan hệ Thực thể](5.3-Data-models-entity-relationships/)
4. [Kiến trúc Thành phần Tối ưu](5.4-Optimized-component-architecture/)
5. [Lược đồ RDS Postgres](5.5-RDS-postgres-schema/)
6. [Lược đồ DynamoDB](5.6-DynamoDB-schema/)
7. [Các Thành phần Hạ tầng](5.7-Infrastructure-components/)
8. [Khả năng Quan sát & Giám sát](5.8-Observability-monitoring/)