---
title : "Optimized Component Architecture"
date : 2026-01-05
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

### A. CvJdProcessor (Lambda Xử lý Được Hợp nhất)

Xử lý cả luồng CV và JD trong một Lambda duy nhất với logic rẽ nhánh:

| Khía cạnh           | Luồng CV (Ứng viên)                              | Luồng JD (Nhà tuyển dụng)                     |
| ------------------- | ------------------------------------------------ | --------------------------------------------- |
| **Kích hoạt (Trigger)** | Step Functions (qua S3 → SQS → IngestionTrigger) | Step Functions (qua lời gọi trực tiếp từ backend .NET) |
| **Nguồn Văn bản** | Textract (PDF trên S3)                           | RDS `Jobs.Description`                        |
| **Che Dữ liệu PII** | Có (spaCy NER)                                   | Không                                         |
| **Phân tích bằng Claude** | Có (Agent 1 + Agent 2)                           | Không                                         |
| **Tạo Vector Nhúng (Embedding)** | Có (Cohere search_document)                      | Có (Cohere search_document)                   |
| **Cross-Encoder** | Có (nếu có cung cấp JD)                          | Không                                         |
| **Thời gian chờ (Timeout)** | 300s                                             | 300s                                          |
| **Bộ nhớ (Memory)** | 1024 MB                                          | 1024 MB                                       |
| **VPC** | Có (Truy cập RDS để đọc JD)                      | Có (Truy cập RDS để đọc JD)                   |

### B. Trách nhiệm của các Engine (Độc lập)

**JobSuggestionEngine (Luồng Ứng viên)**

- Nhận: `profile_id`, `file_key`, `cv_vector`, `masked_cv_text`, `parsed_data`, `scoring_details`, `interview_guide`, `masking_report`
- Lưu: Vector nhúng CV → RDS (`candidate_embeddings`)
- Lưu: Hồ sơ ứng viên + vector nhúng → RDS (`candidate_profiles_ai`, `candidate_embeddings`)
- Thực hiện: Tìm kiếm ANN bằng pgvector cho top 20 công việc
- Xếp hạng lại: Cross-Encoder (top 5 công việc)
- Chấm điểm: Kết hợp - Hybrid (35% BI-Encoder, 65% Cross-Encoder)
- Tạo: Lời giải thích mức độ phù hợp cho mỗi công việc (Claude)
- Lưu trữ: Top 5 đề xuất → DynamoDB (`SK=JOB_SUGGESTIONS`)

**CandidateRankingEngine (Luồng Nhà tuyển dụng)**

- Nhận: `job_id`, `jd_vector`, `jd_text`, `job_title`, `masked_cv_text`
- Lưu: Vector nhúng JD → RDS (`job_embeddings`)
- Tái sử dụng: Văn bản JD làm đầu vào vector nhúng `search_query` cho tìm kiếm bất đối xứng (asymmetric retrieval)
- Nhúng lại: JD thành `search_query` (tìm kiếm bất đối xứng)
- Thực hiện: Tìm kiếm ANN bằng pgvector cho top 50 ứng viên
- Làm giàu dữ liệu: Lấy hồ sơ ứng viên + hướng dẫn phỏng vấn từ DynamoDB
- Xếp hạng lại: Cross-Encoder (top 15 ứng viên)
- Chấm điểm: Kết hợp - Hybrid (30% BI-Encoder, 70% Cross-Encoder)
- Tạo: Bản tóm tắt ứng viên cho mỗi thứ hạng (Claude)
- Lưu trữ: Top 15 ứng viên được xếp hạng → DynamoDB (`SK=CANDIDATE_RANKING`)

### C. Điều phối: Step Functions (Luồng xử lý Thống nhất)

**Máy trạng thái (State Machine): `cv_processing`**

```json
{
  "Comment": "Điều phối xử lý CV/JD của SmartHire: bộ xử lý thống nhất -> chuyển hướng đến engine ghép nối",
  "StartAt": "CvJdProcessor",
  "States": {
    "CvJdProcessor": {
      "Type": "Task",
      "Resource": "arn:aws:states:::lambda:invoke",
      "Parameters": {
        "FunctionName": "${cv_jd_processor_arn}",
        "Payload.$": "$"
      },
      "OutputPath": "$.Payload",
      "Retry": [
        {
          "ErrorEquals": [
            "Lambda.ServiceException",
            "Lambda.AWSLambdaException",
            "Lambda.SdkClientException",
            "States.TaskFailed"
          ],
          "IntervalSeconds": 2,
          "MaxAttempts": 3,
          "BackoffRate": 2
        }
      ],
      "Next": "RoutingChoice"
    },
    "RoutingChoice": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.profile_id",
          "StringEquals": "recruiter",
          "Next": "CandidateRanking"
        }
      ],
      "Default": "JobSuggestion"
    },
    "JobSuggestion": {
      "Type": "Task",
      "Resource": "arn:aws:states:::lambda:invoke",
      "Parameters": {
        "FunctionName": "${job_suggestion_engine_arn}",
        "Payload.$": "$"
      },
      "OutputPath": "$.Payload",
      "Retry": [
        {
          "ErrorEquals": [
            "Lambda.ServiceException",
            "Lambda.AWSLambdaException",
            "Lambda.SdkClientException",
            "States.TaskFailed"
          ],
          "IntervalSeconds": 3,
          "MaxAttempts": 3,
          "BackoffRate": 2
        }
      ],
      "End": true
    },
    "CandidateRanking": {
      "Type": "Task",
      "Resource": "arn:aws:states:::lambda:invoke",
      "Parameters": {
        "FunctionName": "${candidate_ranking_engine_arn}",
        "Payload.$": "$"
      },
      "OutputPath": "$.Payload",
      "Retry": [
        {
          "ErrorEquals": [
            "Lambda.ServiceException",
            "Lambda.AWSLambdaException",
            "Lambda.SdkClientException",
            "States.TaskFailed"
          ],
          "IntervalSeconds": 3,
          "MaxAttempts": 3,
          "BackoffRate": 2
        }
      ],
      "End": true
    }
  }
}