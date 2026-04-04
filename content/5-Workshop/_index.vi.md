# Tự động hóa trích xuất CV và Đánh giá năng lực ứng viên bằng AI

#### Tổng quan

Trong bài Workshop này, chúng ta sẽ học cách thiết kế và triển khai một luồng xử lý dữ liệu hồ sơ ứng viên (Candidate CV Pipeline) hoàn toàn phi máy chủ (Serverless) và hướng sự kiện (Event-driven) trên nền tảng AWS. 

Quá trình tuyển dụng truyền thống thường gặp vấn đề nghẽn mạng khi ứng viên tải lên các file CV dung lượng lớn qua Backend server, cũng như tốn nhiều rào cản nhân lực để đọc và tổng hợp thông tin. Bài lab này sẽ giải quyết các vấn đề đó thông qua các kiến trúc hiện đại.

Chúng ta sẽ tập trung triển khai 3 nội dung lõi (module) sau:
+ **Event-driven Upload (S3 & SQS)** - Tối ưu hóa trải nghiệm tải file bằng S3 Presigned URL (Bypass server). Cấu hình S3 Event Notifications để tự động đẩy sự kiện "Có CV mới" vào hàng đợi Amazon SQS, đảm bảo hệ thống không bị quá tải khi có lượng lớn hồ sơ nộp cùng lúc.
+ **AI CV Parsing (Textract & Bedrock)** - Phát triển các Lambda Worker xử lý bất đồng bộ. Hệ thống sẽ dùng Amazon Textract để bóc tách văn bản thô (OCR) từ PDF/DOCX, sau đó dùng mô hình Claude trên Amazon Bedrock để trích xuất các thông tin quan trọng (kỹ năng, kinh nghiệm) thành định dạng JSON có cấu trúc.
+ **Parallel AI Evaluation (Step Functions)** - Xây dựng một Workflow tự động hóa bằng AWS Step Functions. Sử dụng trạng thái `Parallel` để gọi AI chấm điểm song song các câu trả lời phỏng vấn của ứng viên, giúp giảm thời gian sinh báo cáo (Report) từ vài phút xuống dưới 90 giây.

#### Nội dung

1. [Tổng quan về workshop](5.1-Workshop-overview/)
2. [Chuẩn bị môi trường & IAM Role](5.2-Prerequiste/)
3. [Tối ưu luồng Upload với S3 Presigned URL](5.3-S3-presigned-url/)
4. [Xây dựng Pipeline bóc tách CV (Textract & Bedrock)](5.4-Cv-parsing-pipeline/)
5. [Tự động hóa đánh giá bằng Step Functions](5.5-Step-functions-evaluation/)
6. [Dọn dẹp tài nguyên](5.6-Cleanup/)