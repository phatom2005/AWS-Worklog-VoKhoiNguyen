---
title : "Cấu hình S3 Event Notification với SQS"
date : 2024-01-01 
weight : 1
chapter : false
pre : " <b> 5.4.1 </b> "
---

1. Truy cập vào S3 Console, chọn Bucket lưu trữ CV của bạn (ví dụ: `smarthire-cv-uploads-...`).
2. Chuyển sang tab **Properties** (Thuộc tính) và kéo xuống phần **Event notifications** (Thông báo sự kiện).
3. Nhấn **Create event notification**:
   * **Event name**: `S3-To-SQS-CV-Upload`
   * **Prefix**: `resumes/` (Chỉ kích hoạt khi file được tải vào thư mục này).
   * **Event types**: Tích chọn `All object create events` (Tất cả sự kiện tạo đối tượng).
   * **Destination**: Chọn `SQS queue` và chọn tên hàng đợi `Candidate-CV-Queue` (đã được tạo sẵn từ phần 5.2).
4. Nhấn **Save changes**.

![S3 Event Notification](/images/5-Workshop/5.4-Processing/s3-event.png)