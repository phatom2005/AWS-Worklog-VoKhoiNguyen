---
title : "Tạo hàm Lambda sinh Presigned URL"
date : 2024-01-01 
weight : 1
chapter : false
pre : " <b> 5.3.1 </b> "
---

#### Tổng quan tác vụ

Trong bước này, bạn sẽ tạo một hàm AWS Lambda bằng Node.js. Hàm này có nhiệm vụ tiếp nhận yêu cầu từ Frontend, sau đó sử dụng thư viện AWS SDK để tạo ra một đường dẫn (URL) có chữ ký số. Đường dẫn này cho phép người dùng tải tệp tin lên S3 Bucket mà không cần thông qua bất kỳ lớp xác thực phức tạp nào khác ở phía Client.

#### Các bước thực hiện

1. Truy cập vào [AWS Lambda Console](https://us-east-1.console.aws.amazon.com/lambda/home?region=us-east-1#/functions).
2. Nhấn nút **Create function**.
3. Thiết lập cấu hình cơ bản:
   * Chọn **Author from scratch**.
   * **Function name**: `SmartHire-GeneratePresignedURL`.
   * **Runtime**: Chọn **Node.js 18.x** hoặc mới hơn.
   * **Architecture**: `x86_64`.

![create-lambda](/images/5-Workshop/5.3-S3-presigned-url/create-lambda.png)

4. Tại mục **Permissions**, mở rộng phần **Change default execution role**:
   * Chọn **Create a new role with basic Lambda permissions**. (Chúng ta sẽ bổ sung quyền truy cập S3 ở bước sau).
5. Nhấn **Create function**.

#### Triển khai mã nguồn (Code)

Tại tab **Code**, thay thế nội dung file `index.mjs` bằng đoạn mã dưới đây. Đoạn mã này sử dụng `@aws-sdk/s3-request-presigner` để tạo lệnh `PutObjectCommand`.

```javascript
import { S3Client, PutObjectCommand } from "@aws-sdk/client-s3";
import { getSignedUrl } from "@aws-sdk/s3-request-presigner";

const s3Client = new S3Client({ region: process.env.AWS_REGION });

export const handler = async (event) => {
    const bucketName = process.env.UPLOAD_BUCKET;
    const fileKey = `resumes/${Date.now()}-${event.fileName}`;
    
    const command = new PutObjectCommand({
        Bucket: bucketName,
        Key: fileKey,
        ContentType: event.contentType,
    });

    try {
        // Sinh URL có hiệu lực trong 300 giây (5 phút)
        const presignedUrl = await getSignedUrl(s3Client, command, { expiresIn: 300 });
        
        return {
            statusCode: 200,
            body: JSON.stringify({ uploadUrl: presignedUrl, fileKey: fileKey }),
        };
    } catch (err) {
        console.error(err);
        return { statusCode: 500, body: "Could not generate URL" };
    }
};