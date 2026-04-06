---
title : "Detailed Data Flows"
date : 2026-01-05
weight : 2 
chapter : false
pre : " <b> 5.2. </b> "
---

### A. CvJdProcessor Lambda — Luồng CV

**Kích hoạt (Trigger):** Step Functions (từ IngestionTrigger thông qua S3 → SQS)

**Đầu vào (Input):**

```json
{
  "profile_id": "candidate-uuid",
  "job_id": "job-uuid | null",
  "file_key": "candidates/{candidateId}/cv.pdf",
  "bucket": "smarthire-raw-assets",
  "jd_text": "Văn bản mô tả công việc (tùy chọn, từ cache DynamoDB)",
  "job_title": "Senior Backend Engineer (tùy chọn)",
  "required_skills": ["AWS", "Python"],
  "jd_vector": [0.12, 0.45, "...1024 số thực (tùy chọn, đã lưu cache)"]
}
```

**Các Bước Xử lý:**

| Bước | Thao tác | Dịch vụ | Mô tả |
| :--- | :--- | :--- | :--- |
| 1 | Textract | AWS Textract | Trích xuất văn bản thô từ file PDF trên S3 (bất đồng bộ) |
| 2 | PII Masking | spaCy NER | Ẩn các nhãn PERSON, ORG, GPE, LOC, NORP bằng các token trung tính |
| 3 | Claude Agent 1 | Bedrock (Claude) | Trích xuất kỹ năng, thâm niên, kinh nghiệm, điểm mạnh/lỗ hổng |
| 4 | Claude Agent 2 | Bedrock (Claude) | Tạo 3 câu hỏi phỏng vấn nhắm đúng mục tiêu |
| 5 | Bi-Encoder | Bedrock (Cohere) | Nhúng văn bản CV đã ẩn danh dưới dạng `search_document` (1024-dim) |
| 6 | Cross-Encoder | sentence-transformers | Chấm điểm các đoạn CV so với văn bản JD (nếu có cung cấp JD) |
| 7 | Hybrid Score | Local computation | Kết hợp điểm số BI (35%) + CE (65%) |

**Đầu ra (Output):**

```json
{
  "profile_id": "candidate-uuid",
  "file_key": "candidates/{candidateId}/cv.pdf",
  "bucket": "smarthire-raw-assets",
  "job_id": "job-uuid | null",
  "job_title": "Senior Backend Engineer",
  "masked_cv_text": "Văn bản CV đã ẩn danh (đã xóa PII, tối đa 50K ký tự)",
  "cv_vector": [0.12, 0.45, "...1024 số thực"],
  "parsed_data": {
    "seniority_estimate": "Senior",
    "frontend_skills": ["React", "TypeScript"],
    "backend_skills": ["Python", "AWS Lambda"],
    "devops_skills": ["Docker", "Terraform"],
    "soft_skills": ["Leadership"],
    "years_experience": 5,
    "matching_score": 82.5,
    "strengths": "Kinh nghiệm backend vững chắc...",
    "gaps": "Thiếu kinh nghiệm frontend..."
  },
  "scoring_details": {
    "bi_encoder_score": 78.5,
    "cross_encoder_score": 85.2,
    "hybrid_score": 82.5,
    "bi_encoder_weight": 0.35,
    "cross_encoder_weight": 0.65,
    "method": "hybrid_bi_cross_encoder"
  },
  "interview_guide": [
    {
      "question": "Hãy giải thích kiến trúc event-driven...",
      "skill_targeted": "System Design",
      "rationale": "Kiểm tra sự hiểu biết về các pattern bất đồng bộ"
    }
  ],
  "masking_report": {
    "blind_screening": "applied",
    "total_entities_masked": 12,
    "by_label": { "PERSON": 3, "ORG": 5, "GPE": 4 }
  },
  "jd_text": "Văn bản mô tả công việc",
  "jd_vector": [0.12, 0.45, "...1024 số thực | null"]
}
```

### B. CvJdProcessor Lambda — Luồng JD

**Kích hoạt (Trigger):** Step Functions (từ .NET backend gọi `StartExecution`)

**Đầu vào (Input):**

```json
{
  "profile_id": "recruiter",
  "job_id": "job-uuid"
}
```

**Các Bước Xử lý:**

| Bước | Thao tác | Dịch vụ | Mô tả |
| :--- | :--- | :--- | :--- |
| 1 | Read JD | RDS (PostgreSQL) | `SELECT Title, Description FROM Jobs WHERE Id = job_id` |
| 2 | Bi-Encoder | Bedrock (Cohere) | Nhúng văn bản JD dưới dạng `search_document` (1024-dim) |

**Đầu ra (Output):**

```json
{
  "profile_id": "recruiter",
  "job_id": "job-uuid",
  "job_title": "Senior Backend Engineer",
  "jd_text": "Mô tả JD thô từ bảng Jobs trong RDS",
  "jd_vector": [0.12, 0.45, "...1024 số thực"],
  "masked_cv_text": "Giống với jd_text (để tương thích ngược với CandidateRankingEngine)"
}
```

### C. JobSuggestionEngine (Luồng Ứng viên)

**Đầu vào (Input):** Đầu ra từ luồng CV của CvJdProcessor (xem ở trên)

**Đầu ra (Ghi vào DynamoDB + Phát qua AppSync):**

```json
{
  "PK": "CANDIDATE#{profile_id}",
  "SK": "JOB_SUGGESTIONS",
  "type": "JOB_SUGGESTIONS",
  "candidateId": "candidate-uuid",
  "suggestions": [
    {
      "jobId": "job-uuid",
      "jobTitle": "Senior Backend Engineer",
      "biScore": 78.5,
      "crossScore": 82.1,
      "finalScore": 80.8,
      "matchExplanation": "Rất phù hợp do..."
    }
  ],
  "suggestionCount": 5,
  "updatedAt": "2026-03-30T10:30:00Z",
  "expiresAt": 1746000000
}
```

**Dữ liệu phát (payload) của AppSync:**

```json
{
  "candidateId": "candidate-uuid",
  "suggestions": "[ ...AWSJSON... ]",
  "updatedAt": "2026-03-30T10:30:00Z"
}
```

### D. CandidateRankingEngine (Luồng Nhà tuyển dụng)

**Đầu vào (Input):** Đầu ra từ luồng JD của CvJdProcessor (xem ở trên)

**Đầu ra (Ghi vào DynamoDB + Phát qua AppSync):**

```json
{
  "PK": "JOB#{job_id}",
  "SK": "CANDIDATE_RANKING",
  "type": "CANDIDATE_RANKING",
  "jobId": "job-uuid",
  "rankedCandidates": [
    {
      "candidateId": "candidate-uuid",
      "rank": 1,
      "seniority": "Senior",
      "yearsExperience": 5,
      "biEncoderScore": 85.2,
      "crossEncoderScore": 88.7,
      "finalScore": 87.3,
      "candidateSnapshot": "Ứng viên có 5 năm kinh nghiệm backend...",
      "topSkills": ["AWS", "Python", "Lambda"],
      "interviewGuide": [],
      "applicationStatus": "POOL"
    }
  ],
  "candidateCount": 15,
  "updatedAt": "2026-03-30T10:30:00Z"
}
```

**Dữ liệu phát (payload) của AppSync:**

```json
{
  "jobId": "job-uuid",
  "rankedCandidates": "[ ...AWSJSON... ]",
  "updatedAt": "2026-03-30T10:30:00Z"
}
```