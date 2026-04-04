---
title: "Worklog Tuần 4"
date: 2026-01-26
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4:
* Tìm hiểu giám sát hệ thống với Amazon CloudWatch.
* Cấu hình DNS tích hợp với Amazon Route53.
* Sử dụng AWS CLI để quản lý tài nguyên từ command line.
* Tổng hợp kiến thức qua bài thực hành Highly Available Web Application.

### Các công việc trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu |
|-----|-----------|--------------|-----------------|----------|
| 2 | **Create System Monitor with Amazon CloudWatch** — tạo Dashboard, Alarm (CPU/Memory), Logs Insights; thiết lập alert qua SNS | 26/01/2026 | 26/01/2026 | https://cloudjourney.awsstudygroup.com/ |
| 3 | **Set up hybrid DNS with Amazon Route53** — tạo Hosted Zone, record A/CNAME, tìm hiểu Routing Policy (Latency, Failover) | 27/01/2026 | 27/01/2026 | https://cloudjourney.awsstudygroup.com/ |
| 4 | **Using AWS CLI on Amazon EC2** — thực hành các lệnh CLI quản lý EC2, S3, IAM; cấu hình profile và region | 28/01/2026 | 28/01/2026 | https://cloudjourney.awsstudygroup.com/ |
| 5 | **Highly Available Web Application Workshop** — deploy ứng dụng web trên Multi-AZ với ALB, Auto Scaling Group, RDS Multi-AZ | 29/01/2026 | 29/01/2026 | https://cloudjourney.awsstudygroup.com/ |
| 6 | Tổng hợp kiến trúc SmartHire-AI, đọc source code: `template.yaml`, `main.tf`, `ingestion_trigger.py`; map kiến thức 4 tuần vào dự án thực tế | 30/01/2026 | 30/01/2026 | — |

### Kết quả đạt được tuần 4:
* Tạo được CloudWatch Dashboard + Alarm cảnh báo qua SNS — đúng với cách SmartHire dùng CloudWatch giám sát Lambda và ECS.
* Cấu hình Route53 Hosted Zone và hiểu luồng DNS resolution — nền tảng cho domain `smarthire-ai.org` của dự án.
* Thành thạo AWS CLI cơ bản: quản lý S3, EC2, IAM từ terminal.
* Hoàn thành Highly Available Web App Workshop: hiểu kiến trúc ALB + Auto Scaling + RDS Multi-AZ — đúng với hạ tầng SmartHire (ECS Fargate + RDS Multi-AZ + RDS Proxy).
* Sẵn sàng bắt đầu coding module Candidate Flow từ tuần 5.
