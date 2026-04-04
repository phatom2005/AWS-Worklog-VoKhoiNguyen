---
title : "Định nghĩa State Machine"
date : 2024-01-01 
weight : 1
chapter : false
pre : " <b> 5.5.1 </b> "
---

AWS Step Functions sử dụng ngôn ngữ cấu trúc JSON có tên là **Amazon States Language (ASL)** để định nghĩa luồng chạy.

#### Các bước tạo State Machine

1. Mở [AWS Step Functions console](https://us-east-1.console.aws.amazon.com/states/home).
2. Nhấn nút **Create state machine**.
3. Tại màn hình khởi tạo, chọn **Write your workflow in code** (Viết quy trình bằng mã).
4. Xóa mã mặc định và dán đoạn ASL JSON dưới đây vào trình chỉnh sửa. Đoạn mã này mô tả 3 bước: Gom dữ liệu -> Chấm điểm song song -> Tổng hợp kết quả.

```json
{
  "Comment": "Quy trình chấm điểm phỏng vấn song song của SmartHire AI",
  "StartAt": "Lấy danh sách câu trả lời",
  "States": {
    "Lấy danh sách câu trả lời": {
      "Type": "Task",
      "Resource": "arn:aws:states:::dynamodb:query",
      "Parameters": {
        "TableName": "InterviewAnswers",
        "KeyConditionExpression": "InterviewID = :id",
        "ExpressionAttributeValues": {
          ":id": {"S.$": "$.interview_id"}
        }
      },
      "Next": "Chấm điểm song song"
    },
    "Chấm điểm song song": {
      "Type": "Map",
      "ItemProcessor": {
        "ProcessorConfig": { "Mode": "INLINE" },
        "StartAt": "Gọi AI Bedrock chấm điểm",
        "States": {
          "Gọi AI Bedrock chấm điểm": {
            "Type": "Task",
            "Resource": "arn:aws:states:::lambda:invoke",
            "Parameters": {
              "FunctionName": "SmartHire-AIGraderLambda",
              "Payload": { "answer.$": "$.Item" }
            },
            "End": true
          }
        }
      },
      "Next": "Tổng hợp điểm và Xuất báo cáo"
    },
    "Tổng hợp điểm và Xuất báo cáo": {
      "Type": "Task",
      "Resource": "arn:aws:states:::lambda:invoke",
      "Parameters": {
        "FunctionName": "SmartHire-ReportGeneratorLambda"
      },
      "End": true
    }
  }
}
```

5. Ở góc trên bên phải, bạn sẽ thấy AWS tự động vẽ ra một biểu đồ khối (Visual Graph) cực kỳ trực quan phản ánh đúng JSON bạn vừa nhập.
6. Nhấn **Next**.
7. Đặt tên là `SmartHire-EvaluationWorkflow`.
8. Ở phần **Permissions**, chọn *Create new role* để AWS tự động tạo các quyền cần thiết cho phép Step Functions được gọi Lambda và DynamoDB.
9. Nhấn **Create state machine**.

![Create State Machine](/images/5-Workshop/5.5-Automation/create-state-machine.png)