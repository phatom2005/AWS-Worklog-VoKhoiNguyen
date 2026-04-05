---
title : "Tối ưu luồng Upload với S3 Presigned URL"
date : 2024-01-01 
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

#### Khái niệm S3 Presigned URL (Bypass Server)

Trong phần này, thiết lập một cơ chế bảo mật cho phép ứng viên tải trực tiếp hồ sơ CV (PDF, DOCX) lên Amazon S3 mà không cần phải truyền file thông qua máy chủ Backend.

**Tại sao chúng ta không upload thẳng qua API thông thường?**
Trong các kiến trúc Serverless, các dịch vụ như API Gateway và AWS Lambda có những giới hạn cứng (hard limits) về kích thước gói tin (Payload limit: 10MB cho API Gateway, 6MB cho Lambda) và thời gian thực thi (Timeout). Nếu ứng viên tải lên một file CV thiết kế có dung lượng lớn, hệ thống sẽ báo lỗi `413 Payload Too Large` hoặc bị timeout.

**Giải pháp:** Sử dụng **S3 Presigned URL**. Lúc này, Backend chỉ đóng vai trò "người gác cổng". Khi ứng viên muốn nộp hồ sơ:
1. Frontend gọi một API nhẹ tới Backend để xin quyền upload.
2. Backend (Lambda) tạo một **S3 Presigned URL** (đường dẫn tạm thời, chỉ có hiệu lực trong 5 phút) và trả về cho Frontend.
3. Frontend sử dụng URL này để thực hiện thao tác `PUT` file trực tiếp vào S3 Bucket.

Cách tiếp cận này giúp giảm hoàn toàn băng thông tải file cho Backend, tránh lỗi timeout và tiết kiệm chi phí tính toán đáng kể.

#### Nội dung

- [Tạo hàm sinh Presigned URL](5.3.1-create-presigned-url/)
- [Kiểm thử tải file trực tiếp (Test Upload)](5.3.2-test-upload/)