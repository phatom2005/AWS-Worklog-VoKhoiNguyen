#### Giới thiệu về Kiến trúc Hướng sự kiện (Event-Driven) & AI Serverless

+ **Kiến trúc hướng sự kiện (EDA):** Là một mô hình thiết kế phần mềm trong đó các dịch vụ siêu nhỏ (microservices) giao tiếp với nhau thông qua việc phát (emit) và phản hồi (respond) các sự kiện. Trong AWS, điều này giúp hệ thống tách rời (decoupled), có khả năng tự động mở rộng theo nhu cầu và chịu lỗi cực tốt mà không cần duy trì máy chủ liên tục.
+ **Tích hợp AI phi máy chủ (Serverless AI):** Thay vì tự triển khai và quản lý các mô hình Machine Learning nặng nề trên EC2, chúng ta có thể sử dụng các dịch vụ AI được quản lý hoàn toàn của AWS (như Amazon Textract, Amazon Bedrock) và gọi chúng thông qua AWS Lambda. Điều này giúp đẩy nhanh tốc độ phát triển tính năng đọc hiểu CV và đánh giá ứng viên.

#### Tổng quan về workshop
Trong bài workshop này, thay vì xây dựng một Backend Server nguyên khối (Monolith) để xử lý mọi yêu cầu tải lên và phân tích hồ sơ, bạn sẽ xây dựng một **Data Pipeline (Đường ống dữ liệu)** hoàn toàn phi máy chủ, mô phỏng luồng Candidate Flow lõi của hệ thống SmartHire AI.

Bạn sẽ thiết lập và kết nối các thành phần sau:
+ **Tầng lưu trữ & Tối ưu tải lên (Amazon S3):** Nơi nhận trực tiếp các tệp CV (PDF/DOCX) từ người dùng cuối thông qua cơ chế Presigned URL, giúp giảm tải hoàn toàn cho Backend API.
+ **Tầng đệm & Hàng đợi (Amazon SQS):** Đóng vai trò làm bộ đệm (buffer) giữa S3 và tầng xử lý. Khi có hàng trăm ứng viên nộp CV cùng lúc, SQS đảm bảo hệ thống không bị quá tải (Rate limiting) và không làm mất bất kỳ hồ sơ nào.
+ **Tầng bóc tách dữ liệu (Lambda + Textract + Bedrock):** Các hàm Lambda hoạt động như những Worker chạy ngầm. Chúng lấy CV từ hàng đợi, bóc tách văn bản thô bằng Textract, sau đó truyền cho mô hình AI Claude (qua Bedrock) để trích xuất kỹ năng, kinh nghiệm thành định dạng JSON có cấu trúc.
+ **Tầng lưu trữ dữ liệu (Amazon DynamoDB):** Nơi lưu trữ thông tin ứng viên (Candidate Profile) đã được làm sạch và phân loại.
+ **Tầng điều phối luồng (AWS Step Functions):** Một công cụ điều phối trực quan (orchestrator) để kết nối các bước gom dữ liệu, gọi AI chấm điểm (thực thi song song) và xuất báo cáo cuối cùng.

Việc chia nhỏ hệ thống như vậy giúp nắm vững tư duy thiết kế hệ thống phân tán (Distributed System) và cách thức các dịch vụ AWS tương tác bảo mật với nhau thông qua IAM Role.