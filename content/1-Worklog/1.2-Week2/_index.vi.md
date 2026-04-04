---
title: "Worklog Tuần 2"
date: 2026-01-12
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2:
* Triển khai hạ tầng mạng với Amazon VPC (subnet, routing, security).
* Deploy ứng dụng đầu tiên lên Amazon EC2.
* Cấp quyền cho ứng dụng truy cập AWS services thông qua IAM Role.
* Sử dụng AWS Cloud9 làm Cloud IDE trực tiếp trên trình duyệt.

### Các công việc trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu |
|-----|-----------|--------------|-----------------|----------|
| 2 | **Deploy network infrastructure with Amazon VPC** — tạo VPC, public/private subnet, Internet Gateway, Route Table, Security Group | 12/01/2026 | 12/01/2026 | https://cloudjourney.awsstudygroup.com/ |
| 3 | **Getting Started with Amazon EC2** — tạo EC2 instance, chọn AMI, instance type, key pair; SSH vào instance | 13/01/2026 | 13/01/2026 | https://cloudjourney.awsstudygroup.com/ |
| 4 | **Thực hành EC2:** deploy ứng dụng web đơn giản lên EC2, cấu hình Security Group mở port 80/443 | 14/01/2026 | 14/01/2026 | https://cloudjourney.awsstudygroup.com/ |
| 5 | **Grant application permissions with IAM Role** — tạo IAM Role, gắn vào EC2 Instance Profile để truy cập S3/DynamoDB không cần access key | 15/01/2026 | 15/01/2026 | https://cloudjourney.awsstudygroup.com/ |
| 6 | **Use the Cloud IDE with AWS Cloud9** — tạo Cloud9 environment trên EC2, viết và chạy code trực tiếp trên browser | 16/01/2026 | 16/01/2026 | https://cloudjourney.awsstudygroup.com/ |

### Kết quả đạt được tuần 2:
* Tạo được VPC với public/private subnet, cấu hình Internet Gateway và Route Table.
* Launch EC2 instance, kết nối SSH, deploy ứng dụng web cơ bản thành công.
* Hiểu IAM Role vs IAM User: ứng dụng chạy trên EC2 dùng Role thay vì access key (best practice).
* Thiết lập AWS Cloud9 làm môi trường phát triển trên cloud, sẵn sàng thay thế môi trường local.
