---
title : "Thiết lập luồng bóc tách CV tự động"
date : 2024-01-01 
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

#### Tổng quan

Sau khi ứng viên tải file CV lên Amazon S3 thành công (từ bài lab 5.3), chúng ta cần một hệ thống tự động đọc hiểu và phân loại các kỹ năng trong CV đó. Để hệ thống không bị quá tải khi có hàng nghìn người nộp hồ sơ cùng lúc, chúng ta sẽ sử dụng kiến trúc **Hướng sự kiện (Event-Driven)**.

Quy trình hoạt động:
1. S3 gửi một sự kiện (Event) vào hàng đợi **Amazon SQS** mỗi khi có file mới.
2. Hàm **AWS Lambda (Worker)** sẽ lấy sự kiện từ SQS ra để xử lý dần.
3. Lambda gọi **Amazon Textract** để trích xuất văn bản thô (OCR) từ file PDF.
4. Lambda tiếp tục gửi đoạn văn bản thô đó cho **Amazon Bedrock (mô hình Claude 3.5)** để trích xuất ra định dạng JSON chứa các kỹ năng, số năm kinh nghiệm.
5. Cuối cùng, lưu thông tin vào **Amazon DynamoDB**.