---
title: "Worklog Tuần 3"
date: 2026-01-19
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu tuần 3:
* Tìm hiểu lưu trữ tĩnh và object storage với Amazon S3.
* Tạo và quản lý cơ sở dữ liệu với Amazon RDS.
* Tối ưu chi phí với Amazon Lightsail và tự động mở rộng với EC2 Autoscaling.

### Các công việc trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu |
|-----|-----------|--------------|-----------------|----------|
| 2 | **Hosting static website with Amazon S3** — tạo S3 bucket, bật static website hosting, cấu hình bucket policy, upload file HTML/CSS | 19/01/2026 | 19/01/2026 | https://cloudjourney.awsstudygroup.com/ |
| 3 | **Create a database with Amazon RDS** — tạo RDS PostgreSQL instance, cấu hình subnet group, security group; kết nối từ EC2 | 20/01/2026 | 20/01/2026 | https://cloudjourney.awsstudygroup.com/ |
| 4 | **Thực hành RDS:** tạo database, chạy query cơ bản, tìm hiểu automated backup, Multi-AZ deployment | 21/01/2026 | 21/01/2026 | https://cloudjourney.awsstudygroup.com/ |
| 5 | **Optimize compute costs with Amazon Lightsail** — tìm hiểu Lightsail vs EC2, tạo Lightsail instance, so sánh pricing | 22/01/2026 | 22/01/2026 | https://cloudjourney.awsstudygroup.com/ |
| 6 | **Automate Application Scaling with EC2 Autoscaling** — tạo Launch Template, Auto Scaling Group, scaling policy; test scale out/in | 23/01/2026 | 23/01/2026 | https://cloudjourney.awsstudygroup.com/ |

### Kết quả đạt được tuần 3:
* Host thành công static website lên S3 với custom bucket policy; hiểu S3 object storage khác file system truyền thống như thế nào.
* Tạo và kết nối RDS PostgreSQL — nền tảng cho database của dự án SmartHire (RDS Multi-AZ + RDS Proxy).
* Hiểu sự khác biệt Lightsail vs EC2 và khi nào nên chọn từng loại.
* Cấu hình EC2 Auto Scaling Group với scaling policy — hiểu cơ chế scale tự động mà SmartHire dùng cho ECS.
