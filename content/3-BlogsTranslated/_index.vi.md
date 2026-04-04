---
title: "Các bài blogs đã dịch"
date: 2026-02-05
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

Tại đây là phần liệt kê và giới thiệu các bài blog AWS mà tôi đã đọc và dịch trong thời gian thực tập, liên quan đến các dịch vụ được sử dụng trong dự án SmartHire AI.

### [Blog 1 - Adaptive sampling với AWS X-Ray để bắt giữ các span quan trọng](3.1-Blog1/)

Bài blog này giới thiệu tính năng **adaptive sampling** trong AWS X-Ray — một cơ chế lấy mẫu thông minh giúp cân bằng giữa chi phí quan sát và khả năng phát hiện sự cố. Bạn sẽ tìm hiểu sự khác biệt giữa *static sampling* (lấy mẫu cố định) và *adaptive sampling* (lấy mẫu thích nghi), cách cấu hình **Sampling Boost** để tự động tăng tỷ lệ lấy mẫu khi hệ thống có lỗi hoặc độ trễ tăng đột biến, và cách sử dụng **Anomaly Span Capture** để ghi lại span lỗi mà không cần trace đầy đủ. Bài viết có liên quan trực tiếp đến việc giám sát Lambda, API Gateway và Step Functions trong dự án SmartHire AI.

### [Blog 2 - Tự động hóa kích hoạt AWS Systems Manager cho hybrid node với Lambda & DynamoDB](3.2-Blog2/)

Bài blog này hướng dẫn cách xây dựng giải pháp **serverless** tự động hóa việc tạo và quản lý thông tin xác thực AWS Systems Manager Hybrid Activations. Giải pháp sử dụng **API Gateway + Lambda + DynamoDB + Parameter Store** theo mô hình event-driven quen thuộc — tương tự kiến trúc backend của SmartHire AI. Bạn sẽ học được cách Lambda quản lý trạng thái bằng DynamoDB (cơ chế lock/unlock), cách Private API Gateway bảo vệ endpoint nội bộ, và cách triển khai toàn bộ bằng AWS CloudFormation.
