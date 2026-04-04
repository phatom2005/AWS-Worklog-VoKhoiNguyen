---
title: "Week 6 Worklog"
date: 2026-02-09
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Week 6 Objectives:
* Build the endpoint to upload CVs to S3 (CV Bucket).
* Implement the creation of Presigned URLs to allow FE to PUT files directly to S3.
* Create `CandidateProfile` with `PROCESSING` status after upload.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date |
|-----|-----------|--------------|-----------------|
| Mon | Implement `CVService.UploadToS3Async` — PutObject to S3 CV Bucket (`cvs/{userId}/{guid}.pdf`) | 02/09/2026 | 02/09/2026 |
| Tue | Implement `CVService.CreateProfileAsync` — INSERT `candidate_profiles` (status: PROCESSING) | 02/10/2026 | 02/10/2026 |
| Wed | Implement `CVService.GeneratePresignedUploadUrlAsync` — generate Presigned PUT URL (5 minutes) | 02/11/2026 | 02/11/2026 |
| Thu | Implement `POST /api/cv/upload` and `POST /api/cv/presigned-url` in CVController | 02/12/2026 | 02/12/2026 |
| Fri | Validate file: only accept PDF/DOC/DOCX, 10MB limit; test with Postman | 02/13/2026 | 02/13/2026 |

### Week 6 Achievements:
* `POST /api/cv/upload`: received multipart file, uploaded to S3, created profile, returned `202 Accepted` + `profileId`.
* `POST /api/cv/presigned-url`: returned presigned URL for FE to directly PUT to S3, created `PROCESSING` profile as soon as the URL was issued.
* Input validation: extension, file size, Cognito JWT authentication from API Gateway.
* File key with format `cvs/{userId}/{guid}.ext` matches the filter of S3 Event Notification in Terraform.