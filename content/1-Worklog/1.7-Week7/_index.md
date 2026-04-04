---
title: "Week 7 Worklog"
date: 2026-02-16
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Week 7 Objectives:
* Understand the asynchronous processing flow: SQS → IngestionTrigger → Step Functions.
* Read the Lambda code for `ingestion_trigger.py` and `cv_jd_processor.py`.
* Learn about DynamoDB (Net Store) used for tracking CV status and caching JD vectors.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date |
|-----|-----------|--------------|-----------------|
| Mon | Implement `CVService.PublishCVParseQueueAsync` — send `{profileId, fileKey, jobId, jdText}` message to SQS `cv-upload-queue` | 02/16/2026 | 02/16/2026 |
| Tue | Understand `ingestion_trigger.py`: validate S3 file via HeadObject, fetch JD from DynamoDB, route to Step Functions | 02/17/2026 | 02/17/2026 |
| Wed | Learn DynamoDB Single Table Design: `PK=CANDIDATE#<id>, SK=CV#<key>` (status tracking); `PK=JOB#<id>, SK=METADATA` (JD vector cache) | 02/18/2026 | 02/18/2026 |
| Thu | Understand Step Functions flow → `cv_jd_processor` → Routing Choice → `JobSuggestionEngine` / `CandidateRankingEngine` | 02/19/2026 | 02/19/2026 |
| Fri | End-to-end testing: upload CV → SQS → IngestionTrigger → DynamoDB status `PROCESSING` → Step Functions start | 02/20/2026 | 02/20/2026 |

### Week 7 Achievements:
* Clearly understood the 3 steps of IngestionTrigger: (1) parse SQS payload, (2) validate S3 file via HeadObject, (3) `save_status_to_dynamodb` → `start_processing`.
* `start_processing` prioritizes calling `sfn.start_execution(STATE_MACHINE_ARN)` if configured; fallbacks to Direct Invoke `vector_ops`.
* DynamoDB stores `PROCESSING` → `DONE` status for each CV; caches `jdVector` (Cohere Embed) for reuse when multiple candidates apply for the same job.
* The upload → SQS → IngestionTrigger → Step Functions flow works correctly in the test environment.