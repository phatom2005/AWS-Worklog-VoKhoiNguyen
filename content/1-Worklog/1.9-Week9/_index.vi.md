---
title: "Worklog Tuần 9"
date: 2026-03-02
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Mục tiêu tuần 9:
* Đọc hiểu hai AI engine: `job_suggestion_engine.py` (cho Candidate) và `candidate_ranking_engine.py` (cho Recruiter).
* Xây dựng API REST phục vụ Recruiter Dashboard và Candidate Profile.
* Hoàn thiện endpoint quản lý CV (xem, xóa) kết hợp dữ liệu từ RDS PostgreSQL.

### Các công việc trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành |
|-----|-----------|--------------|-----------------|
| 2 | Đọc hiểu `job_suggestion_engine.py`: dùng Bedrock (Cohere Embed v3) để vector hóa CV, tìm matching jobs từ DynamoDB, push kết quả qua AppSync | 02/03/2026 | 02/03/2026 |
| 3 | Đọc hiểu `candidate_ranking_engine.py`: rank ứng viên theo `matchingScore` dùng Bedrock (Claude 3.5 Sonnet), push kết quả qua AppSync | 03/03/2026 | 03/03/2026 |
| 4 | Cài đặt `GET /api/candidates/{profileId}` và `GET /api/cv/my-profiles` — API profile cho Candidate | 04/03/2026 | 04/03/2026 |
| 5 | Cài đặt `GET /api/jobs/{jobId}/candidates` — danh sách ứng viên đã rank theo job; `DELETE /api/cv/{profileId}` với phân quyền | 05/03/2026 | 05/03/2026 |
| 6 | Cài đặt `CandidateService`: GetCandidatesByJob, GetCandidateDetails, UpdateCandidateStatus (RDS PostgreSQL qua RDS Proxy) | 06/03/2026 | 06/03/2026 |

### Kết quả đạt được tuần 9:
* Nắm rõ Routing Choice trong Step Functions: nếu là CV upload → `JobSuggestionEngine` + `CandidateRankingEngine`; nếu là JD upload → chỉ cập nhật DynamoDB vector cache.
* `JobSuggestionEngine` gọi Bedrock Cohere Embed v3 để embed CV, so khớp vector với JD đã cache trong DynamoDB, push kết quả qua AppSync mutation `publishJobSuggestions` → Candidate nhận real-time.
* `CandidateRankingEngine` dùng Claude 3.5 Sonnet để phân tích và rank ứng viên, push kết quả qua `publishCandidateRanking` → Recruiter Dashboard nhận real-time.
* API REST candidate profile hoàn chỉnh; bảo mật: chỉ xóa được CV của chính mình (`profile.UserId != user.Id` → 403 Forbid).
