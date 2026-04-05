---
title : "Kiểm thử luồng sự kiện bóc tách CV"
date : 2024-01-01 
weight : 3
chapter : false
pre : " <b> 5.4.3 </b> "
---

Bây giờ chúng ta sẽ kiểm tra xem luồng hệ thống hoạt động từ đầu đến cuối có mượt mà hay không.

1. Quay lại bước **5.3.2**, sử dụng đường dẫn URL (Presigned URL) hoặc công cụ như Postman/cURL để tải một file CV bằng PDF mẫu lên S3 Bucket của bạn.
2. Ngay khi file được tải lên, S3 sẽ gửi tin nhắn sang SQS, và Lambda sẽ âm thầm chạy ngầm phía sau (Background Processing).
3. Khoảng 5-10 giây sau, bạn mở dịch vụ **Amazon DynamoDB**, chọn bảng `CandidateProfiles` và nhấn **Explore table items**. 

Nếu bạn thấy một bản ghi mới xuất hiện chứa các kỹ năng và số năm kinh nghiệm của ứng viên đã được phân tích gọn gàng, xin chúc mừng, bạn đã xây dựng thành công một **AI Data Pipeline** hoàn chỉnh!

![DynamoDB Result](/images/5-Workshop/5.4-Processing/dynamo-result.png)