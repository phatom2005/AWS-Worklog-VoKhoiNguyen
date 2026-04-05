---
title: "Đề xuất dự án"
date: 2026-01-05
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# SmartHire AI
## Giải pháp Tự động hóa Trích xuất CV và Đánh giá Ứng viên trên AWS

### 1. Tóm tắt điều hành

**SmartHire AI** là nền tảng hỗ trợ tuyển dụng được xây dựng theo kiến trúc **100% Serverless** trên AWS, nhắm đến việc tự động hóa hai quy trình lõi tốn nhiều thời gian nhất của bộ phận nhân sự: tiếp nhận - bóc tách hồ sơ ứng viên (CV) và tổng hợp - đánh giá kết quả phỏng vấn. Hệ thống ứng dụng kiến trúc **hướng sự kiện (Event-Driven)** kết hợp các dịch vụ **Generative AI (Amazon Bedrock)** để mang lại hiệu suất cao, tiết kiệm chi phí băng thông và đảm bảo tính công bằng trong việc chấm điểm ứng viên.

### 2. Tuyên bố vấn đề

#### Vấn đề hiện tại là gì?

Việc tải lên các tệp CV dung lượng lớn (PDF, DOCX) qua máy chủ trung gian thường gây nghẽn băng thông, làm giảm trải nghiệm ứng viên và tăng chi phí hạ tầng. HR và Technical Leader phải đọc thủ công từng CV để lọc kỹ năng, tiêu tốn nhiều nhân giờ và dễ bỏ sót ứng viên tiềm năng. Sau các buổi phỏng vấn, việc tổng hợp nhận xét và đối chiếu điểm số giữa các ứng viên thường thiếu tính đồng nhất và phụ thuộc vào cảm tính của người phỏng vấn.

#### Giải pháp

Nền tảng sử dụng **S3 Presigned URL** để ứng viên tải CV trực tiếp lên Amazon S3 mà không qua Backend, loại bỏ hoàn toàn nghẽn băng thông. Luồng **S3 Event Notification → SQS → Lambda → Amazon Textract** tự động trích xuất văn bản thô bất đồng bộ ngay khi CV được tải lên. **Amazon Bedrock (Claude 3.5 Sonnet)** phân tích văn bản thô thành dữ liệu JSON có cấu trúc lưu vào DynamoDB. Đồng thời, **AWS Step Functions** điều phối quy trình chấm điểm phỏng vấn song song theo phương pháp STAR, sinh báo cáo tự động cho HR.

#### Lợi ích và giá trị mang lại

Giải pháp giúp HR tập trung vào các công việc có giá trị cao thay vì đọc thủ công từng hồ sơ, giảm **80%** thời gian sàng lọc CV. Báo cáo sau phỏng vấn được tạo tự động trong **dưới 90 giây** nhờ xử lý song song, đảm bảo tiêu chí đánh giá đồng nhất và khách quan giữa các ứng viên. Kiến trúc Serverless Pay-as-you-go giúp tối ưu chi phí vận hành và có khả năng scale vô hạn trong các đợt tuyển dụng lớn.

### 3. Kiến trúc giải pháp

Nền tảng sử dụng kiến trúc phi máy chủ (Serverless), phân tách rõ ràng giữa **API layer** và các luồng **background processing** nặng. Toàn bộ xử lý dữ liệu được thực hiện bất đồng bộ để không ảnh hưởng đến trải nghiệm người dùng:

![Kiến trúc tổng thể SmartHire AI](images/2-Proposal/smarthire_architecture.png)

#### Các dịch vụ AWS sử dụng

- **Amazon S3**: Lưu CV gốc, sinh Presigned URL cho ứng viên tải trực tiếp, lưu báo cáo PDF/HTML cuối cùng.
- **Amazon SQS**: Hàng đợi tin nhắn giúp hệ thống không bị quá tải khi hàng nghìn CV được tải lên đồng thời.
- **AWS Lambda**: Các Worker thực thi logic gọi AI, xử lý dữ liệu và lưu trữ kết quả.
- **Amazon Textract**: Trích xuất quang học (OCR) văn bản từ các định dạng file PDF, DOCX.
- **Amazon Bedrock**: Cung cấp mô hình LLM (Claude 3.5 Sonnet) để phân tích ngữ nghĩa CV và chấm điểm câu trả lời phỏng vấn.
- **AWS Step Functions**: Điều phối Workflow nhiều bước để chấm điểm ứng viên song song.
- **Amazon DynamoDB**: Lưu metadata ứng viên, cấu trúc kỹ năng đã bóc tách và trạng thái xử lý.
- **Amazon API Gateway**: Cổng giao tiếp giữa Frontend và Backend Lambda.
- **Amazon Cognito**: Xác thực và phân quyền người dùng (ứng viên, HR, recruiter).

#### Thiết kế các thành phần

- **CV Upload Layer**: Frontend gọi API Gateway → Lambda → sinh S3 Presigned URL; ứng viên PUT file trực tiếp lên S3.
- **Event-Driven Pipeline**: S3 Event Notification kích hoạt SQS → Lambda Worker gọi Textract và Bedrock → lưu JSON vào DynamoDB.
- **AI Evaluation Workflow**: Step Functions Express Workflow điều phối Parse → ScoreParallel → Aggregate → GenerateReport.
- **Notification Layer**: Lambda cập nhật trạng thái xử lý, thông báo kết quả về Frontend qua AppSync (GraphQL Subscription).
- **Auth & Security**: Cognito User Pool cấp JWT token; API Gateway dùng Cognito Authorizer xác thực mọi request.

### 4. Triển khai kỹ thuật

**Các giai đoạn triển khai**

Dự án triển khai theo 4 giai đoạn liên tiếp trong thời gian thực tập:

- **Giai đoạn 1 — Hạ tầng lưu trữ & API Upload (Tuần 1):** Thiết lập S3 bucket, cấu hình IAM Role, viết Lambda API tạo Presigned URL cho Frontend. Ứng viên tải CV trực tiếp lên S3 không qua Backend.
- **Giai đoạn 2 — Event-Driven Pipeline & Textract (Tuần 2):** Cấu hình S3 Event Notification → SQS trigger → Lambda Worker tích hợp Amazon Textract để bóc tách text thô từ PDF/DOCX.
- **Giai đoạn 3 — Tích hợp Bedrock & DynamoDB (Tuần 3):** Thiết kế Prompt Engineering, gọi Amazon Bedrock (Claude 3.5 Sonnet) phân tích text thô thành JSON (`skills`, `yearsOfExperience`, `education`), lưu vào DynamoDB.
- **Giai đoạn 4 — Step Functions & Hoàn thiện (Tuần 4):** Xây dựng Step Functions Express Workflow với Parallel State chấm điểm phỏng vấn đồng thời, sinh báo cáo PDF/HTML lưu lại S3. Kiểm thử bảo mật, tối ưu và hoàn thiện tài liệu.

**Yêu cầu kỹ thuật**

- **CV Parsing Pipeline:** S3 bucket (prefix `cvs/`), SQS Standard Queue với DLQ, Lambda runtime Python 3.12, Amazon Textract AnalyzeDocument API, Amazon Bedrock InvokeModel API (claude-3-5-sonnet), DynamoDB table với Single Table Design.
- **AI Evaluation Workflow:** Step Functions Express Workflow, Parallel State với tối đa 10 nhánh đồng thời, Lambda kết nối Bedrock để sinh nhận xét tổng quan, S3 lưu báo cáo output.
- **API & Auth:** API Gateway REST API, Cognito User Pool + App Client, JWT Authorizer trên mọi endpoint protected.

### 5. Lộ trình triển khai

**Tiến độ dự án**

- **Tuần 1:** Thiết lập S3 bucket, viết API tạo Presigned URL; cấu hình IAM Role cho Lambda; kiểm thử luồng upload trực tiếp từ Frontend.
- **Tuần 2:** Xây dựng Event-Driven Pipeline: S3 Event Notification → SQS → Lambda; tích hợp và kiểm thử Amazon Textract với các định dạng PDF, DOCX khác nhau.
- **Tuần 3:** Thiết kế Prompt Engineering cho Amazon Bedrock; chuẩn hóa JSON output; lưu metadata vào DynamoDB; kiểm thử accuracy của kết quả bóc tách.
- **Tuần 4:** Xây dựng Step Functions State Machine với Parallel chấm điểm; sinh báo cáo PDF/HTML; kiểm thử bảo mật (IAM, Cognito, S3 Bucket Policy) và hoàn thiện tài liệu.

### 6. Ước tính chi phí

Bạn có thể tham khảo ước tính chi phí trên [AWS Pricing Calculator](https://calculator.aws/).

#### Chi phí hạ tầng AWS (ước tính/tháng)

- **AWS Lambda:** $0.00/tháng (dưới ngưỡng Free Tier — 1 triệu request đầu miễn phí).
- **Amazon S3 Standard:** ~$0.05/tháng (lưu trữ CV và báo cáo, ~2 GB).
- **Amazon SQS:** $0.00/tháng (dưới 1 triệu request/tháng — Free Tier).
- **Amazon Textract:** ~$0.015/trang (AnalyzeDocument API).
- **Amazon Bedrock (Claude 3.5 Sonnet):** ~$0.10–$0.30/tháng tùy số lượng CV xử lý.
- **AWS Step Functions:** $0.00/tháng (4,000 lần chuyển trạng thái đầu miễn phí).
- **Amazon DynamoDB:** $0.00/tháng (25 GB lưu trữ + 25 WCU/RCU — Free Tier).
- **Amazon API Gateway:** ~$0.01/tháng (dưới 1 triệu call).

**Tổng ước tính: ~$0.50–$1.00/tháng** (phụ thuộc lưu lượng CV thực tế)

### 7. Đánh giá rủi ro

#### Ma trận rủi ro

- **Prompt Engineering không chính xác:** Tác động cao, xác suất trung bình — kết quả bóc tách CV bị sai trường dữ liệu.
- **Lambda Timeout:** Tác động trung bình, xác suất thấp — file CV quá lớn hoặc Textract/Bedrock chậm phản hồi.
- **Chi phí Bedrock vượt ngân sách:** Tác động trung bình, xác suất thấp — nếu lưu lượng CV tăng đột biến.
- **DLQ tích lũy message:** Tác động thấp, xác suất trung bình — Lambda Worker xử lý thất bại liên tục.

#### Chiến lược giảm thiểu

- **Prompt Engineering:** Xây dựng test suite với CV đa dạng (tiếng Việt, tiếng Anh, nhiều định dạng), đánh giá accuracy theo từng iteration.
- **Lambda Timeout:** Tăng timeout phù hợp (tối đa 15 phút), tách bước Textract và Bedrock thành 2 Lambda riêng nếu cần.
- **Chi phí:** Thiết lập AWS Budget Alert cảnh báo khi vượt ngưỡng $5/tháng; dùng Bedrock On-Demand thay vì Provisioned.
- **DLQ:** Cấu hình CloudWatch Alarm theo dõi `ApproximateNumberOfMessagesNotVisible`; tạo runbook xử lý DLQ định kỳ.

#### Kế hoạch dự phòng

- Nếu Bedrock không khả dụng: fallback sang Regex-based parsing để giữ pipeline hoạt động.
- Nếu Step Functions lỗi: kích hoạt lại execution thủ công qua AWS Console hoặc CLI.

### 8. Kết quả kỳ vọng

#### Cải thiện kỹ thuật

Tốc độ tải lên và xử lý hồ sơ mượt mà, không gián đoạn nhờ kiến trúc bất đồng bộ. Hệ thống có khả năng scale vô hạn để đáp ứng các đợt tuyển dụng lớn mà không cần nâng cấp server vật lý.

#### Giá trị lâu dài

Giảm **80%** thời gian đọc và phân loại hồ sơ thủ công. Báo cáo sau phỏng vấn được tạo tự động trong **dưới 90 giây** nhờ xử lý song song. Tiêu chí đánh giá đồng nhất, khách quan, có thể tái sử dụng cho nhiều vị trí tuyển dụng khác nhau. Mô hình Serverless Pay-as-you-go tối ưu chi phí vận hành, chỉ tính phí theo lưu lượng thực tế sử dụng.
