---
title : "Tự động hóa đánh giá bằng Step Functions"
date : 2024-01-01 
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

#### Giải quyết bài toán Timeout với xử lý song song

Sau khi kết thúc một buổi phỏng vấn trực tuyến, hệ thống SmartHire AI lưu lại rất nhiều câu hỏi và câu trả lời của ứng viên vào cơ sở dữ liệu. Nếu chúng ta chỉ dùng một hàm Lambda chạy vòng lặp tuần tự để gọi AI (Amazon Bedrock) chấm điểm từng câu, thời gian thực thi rất dễ vượt quá giới hạn 15 phút của Lambda và gây ra lỗi (Timeout).

**Giải pháp:** Chúng ta sẽ sử dụng **AWS Step Functions** - một dịch vụ điều phối (Orchestration) cho phép định nghĩa các luồng công việc (Workflow) trực quan. Điểm ăn tiền nhất ở đây là việc sử dụng trạng thái **Map** (hoặc Parallel) để gọi AI chấm điểm *hàng loạt câu hỏi cùng một lúc*. Điều này giúp giảm thời gian tạo báo cáo đánh giá cuối cùng từ vài phút xuống chỉ còn dưới 90 giây!

Trong phần này, bạn sẽ thiết lập một State Machine (Cỗ máy trạng thái) cho luồng đánh giá ứng viên.

![Step Functions Architecture](/images/5-Workshop/5.5-Automation/architecture.png)