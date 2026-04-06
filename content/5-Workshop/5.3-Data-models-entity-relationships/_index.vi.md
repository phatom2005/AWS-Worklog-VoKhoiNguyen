---
title : "Data Models & Entity Relationships"
date : 2026-01-05
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

### A. Sơ đồ Cơ sở Dữ liệu Thống nhất

![image](https://res.cloudinary.com/diahbkjog/image/upload/v1775493744/5_tckslm.png)

### B. Chiến lược Lưu trữ Dữ liệu

#### 1. Tài sản Thô (Amazon S3)

- **Bucket**: `smarthire-raw-assets`
- **Đường dẫn**: `candidates/{userId}/cv.pdf`
- **Mục đích**: Sao lưu bất biến và theo dõi lịch sử kiểm toán cho các tệp PDF CV.
- **Lưu ý**: Các tệp PDF JD không còn được tải lên S3. Văn bản JD được lưu trữ trong RDS `Jobs.Description`.

#### 2. Dữ liệu Quan hệ & Vector (Amazon RDS PostgreSQL)

Đóng vai trò là **Nguồn Sự thật (Source of Truth)** cho các thực thể nghiệp vụ và tìm kiếm ngữ nghĩa.

- **Bảng Nghiệp vụ**: `Users`, `Jobs`, `Applications`, `Companies`.
- **Bảng AI**: `CandidateProfiles` (dữ liệu hồ sơ do AI trích xuất).
- **Pgvector**: `CandidateEmbeddings`, `JobEmbeddings` (vector Cohere 1024 chiều).
- **Nguồn JD**: Cột `Jobs.Description` được đọc trực tiếp bởi Lambda `CvJdProcessor`.

#### 3. Lưu trữ Nóng / Bộ đệm (Amazon DynamoDB)

- **Bảng**: `ApplicationTracking` (Thiết kế Bảng Đơn - Single Table Design)
- **Mục đích**: Thực hiện các thao tác đọc/ghi tần suất cao cho các cập nhật UI theo thời gian thực (đề xuất, xếp hạng, kết quả phân tích).