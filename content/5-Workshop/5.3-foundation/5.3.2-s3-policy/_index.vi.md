---
title : "Kiểm thử luồng tải tệp (Test Upload)"
date : 2024-01-01 
weight : 2
chapter : false
pre : " <b> 5.3.2 </b> "
---

Trong phần này, chúng ta sẽ đóng vai trò là ứng dụng Frontend: Gọi hàm Lambda để xin cấp quyền tải file, sau đó sử dụng đường dẫn (Presigned URL) nhận được để tải trực tiếp một file CV mẫu lên Amazon S3.

#### 1. Chạy thử hàm Lambda để lấy Presigned URL

1. Mở [AWS Lambda console](https://us-east-1.console.aws.amazon.com/lambda/home) và chọn hàm `SmartHire-GeneratePresignedURL` bạn vừa tạo.
2. Chuyển sang tab **Test**.
3. Trong phần **Event name**, nhập `TestGenerateURL`.
4. Trong phần **Event JSON**, dán đoạn dữ liệu sau để mô phỏng yêu cầu từ ứng viên muốn tải lên file PDF:

```json
{
  "fileName": "cv_backend_dev.pdf",
  "contentType": "application/pdf"
}
```

![Test Event](/images/5-Workshop/5.3-S3-presigned-url/lambda-test-event.png)

5. Nhấn nút **Save**, sau đó nhấn nút **Test** màu xanh.
6. Mở rộng phần **Execution result** (Kết quả thực thi). Bạn sẽ thấy trạng thái `200` và một chuỗi JSON trả về chứa `uploadUrl`. Hãy **copy toàn bộ đường link** trong giá trị `uploadUrl` (Đường link này rất dài và chứa các chữ ký bảo mật).

![Execution Result](/images/5-Workshop/5.3-S3-presigned-url/lambda-result.png)

{{% notice info %}}
Đường link này chỉ có hiệu lực trong vòng 5 phút (300 giây) đúng như chúng ta đã cấu hình trong mã nguồn Node.js.
{{% /notice %}}

#### 2. Thực hiện tải file trực tiếp lên S3 (Bypass Server)

Bây giờ, chúng ta sẽ mô phỏng việc Frontend dùng đường link này để đẩy file lên. Bạn có thể sử dụng **Postman** (chọn method PUT, tab Body chọn `binary` và đính kèm file) hoặc dùng công cụ dòng lệnh **cURL** trên máy tính của bạn.

Dưới đây là hướng dẫn sử dụng cURL (bạn mở Terminal hoặc Command Prompt trên máy tính cá nhân):

1. Tạo một file CV mẫu đơn giản bằng lệnh sau (hoặc bạn tự chuẩn bị 1 file `.pdf` bất kỳ):
```bash
echo "Noi dung CV cua ung vien Backend" > cv_backend_dev.pdf
```

2. Thực hiện lệnh `PUT` file lên S3. Hãy thay thế `<DÁN_URL_CỦA_BẠN_VÀO_ĐÂY>` bằng đường link `uploadUrl` bạn vừa copy ở bước trên (nhớ giữ nguyên cặp dấu ngoặc kép `""` bao quanh link):
```bash
curl -X PUT -T "cv_backend_dev.pdf" -H "Content-Type: application/pdf" "<DÁN_URL_CỦA_BẠN_VÀO_ĐÂY>"
```

![cURL Upload](/images/5-Workshop/5.3-S3-presigned-url/curl-upload.png)

Nếu lệnh chạy thành công mà không báo lỗi gì (trả về trống hoặc có HTTP status 200), nghĩa là file của bạn đã được tải lên thành công!

#### 3. Kiểm tra dữ liệu trên Amazon S3

1. Đi đến [Amazon S3 Console](https://s3.console.aws.amazon.com/s3/home).
2. Nhấn vào tên Bucket chứa CV của bạn (ví dụ: `smarthire-cv-uploads-...`).
3. Mở thư mục `resumes/`. Bạn sẽ thấy tệp tin `cv_backend_dev.pdf` (có tiền tố là timestamp thời gian) đã xuất hiện ở đây.

![Check S3](/images/5-Workshop/5.3-S3-presigned-url/check-s3-bucket.png)

#### Tóm tắt

Chúc mừng bạn đã hoàn thành việc thiết kế luồng Upload tối ưu cho dự án Serverless! Nhờ cơ chế Presigned URL, quá trình truyền tải tệp tin nặng diễn ra hoàn toàn giữa máy khách và S3, giúp Backend của SmartHire AI tránh được tình trạng nghẽn mạng và giảm thiểu chi phí vận hành.