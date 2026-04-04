---
title : "Viết mã nguồn Lambda Worker (Textract & Bedrock)"
date : 2024-01-01 
weight : 2
chapter : false
pre : " <b> 5.4.2 </b> "
---

Trong phần này, bạn sẽ tạo một hàm Lambda chạy ngầm (Worker) để nhận thông báo từ SQS, sau đó gọi các dịch vụ AI để bóc tách thông tin ứng viên.

1. Mở [AWS Lambda console](https://us-east-1.console.aws.amazon.com/lambda/home) và tạo một hàm mới tên là `SmartHire-CVParsingWorker` (Runtime: Node.js 18.x).
2. Tại phần **Configuration > Triggers**, nhấn **Add trigger** và chọn **SQS**. Chọn hàng đợi `Candidate-CV-Queue` của bạn.
3. Thay thế đoạn code trong `index.mjs` bằng mã nguồn xử lý tích hợp Textract và Bedrock sau:

```javascript
import { TextractClient, DetectDocumentTextCommand } from "@aws-sdk/client-textract";
import { BedrockRuntimeClient, InvokeModelCommand } from "@aws-sdk/client-bedrock-runtime";
import { DynamoDBClient, PutItemCommand } from "@aws-sdk/client-dynamodb";

const textract = new TextractClient();
const bedrock = new BedrockRuntimeClient();
const dynamodb = new DynamoDBClient();

export const handler = async (event) => {
    // Lấy thông tin file S3 từ tin nhắn SQS
    const sqsBody = JSON.parse(event.Records[0].body);
    const bucket = sqsBody.Records[0].s3.bucket.name;
    const key = decodeURIComponent(sqsBody.Records[0].s3.object.key.replace(/\+/g, ' '));

    try {
        // 1. Gọi Amazon Textract để đọc chữ từ PDF
        const textractResponse = await textract.send(new DetectDocumentTextCommand({
            Document: { S3Object: { Bucket: bucket, Name: key } }
        }));
        const rawText = textractResponse.Blocks.filter(b => b.BlockType === 'LINE').map(b => b.Text).join(' ');

        // 2. Gọi Amazon Bedrock (Claude 3.5) để trích xuất JSON
        const prompt = `Trích xuất kỹ năng, số năm kinh nghiệm, học vấn từ CV sau. Trả về format JSON hợp lệ: { "skills": [], "experience_years": 0, "education": "" }. CV: ${rawText}`;
        const bedrockResponse = await bedrock.send(new InvokeModelCommand({
            modelId: "anthropic.claude-3-5-sonnet-20240620-v1:0",
            contentType: "application/json",
            body: JSON.stringify({
                anthropic_version: "bedrock-2023-05-31",
                max_tokens: 1000,
                messages: [{ role: "user", content: prompt }]
            })
        }));
        const aiResult = JSON.parse(new TextDecoder().decode(bedrockResponse.body)).content[0].text;
        const candidateData = JSON.parse(aiResult);

        // 3. Lưu vào DynamoDB
        await dynamodb.send(new PutItemCommand({
            TableName: process.env.DYNAMODB_TABLE,
            Item: {
                "CandidateID": { S: key },
                "Skills": { S: candidateData.skills.join(", ") },
                "Experience": { N: candidateData.experience_years.toString() }
            }
        }));

        console.log("Xử lý CV thành công!");
        return { statusCode: 200 };
    } catch (error) {
        console.error("Lỗi xử lý CV:", error);
        throw error;
    }
};