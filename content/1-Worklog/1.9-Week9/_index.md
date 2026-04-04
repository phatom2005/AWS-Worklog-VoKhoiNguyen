---
title: "Week 9 Worklog"
date: 2026-03-02
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Week 9 Objectives:
* Read and understand two AI engines: `job_suggestion_engine.py` (for Candidates) and `candidate_ranking_engine.py` (for Recruiters).
* Build REST APIs for the Recruiter Dashboard and Candidate Profile.
* Complete CV management endpoints (view, delete) integrating data from RDS PostgreSQL.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date |
|-----|-----------|--------------|-----------------|
| Mon | Understand `job_suggestion_engine.py`: uses Bedrock (Cohere Embed v3) to vectorize CVs, find matching jobs from DynamoDB, and push results via AppSync | 03/02/2026 | 03/02/2026 |
| Tue | Understand `candidate_ranking_engine.py`: ranks candidates by `matchingScore` using Bedrock (Claude 3.5 Sonnet), pushes results via AppSync | 03/03/2026 | 03/03/2026 |
| Wed | Implement `GET /api/candidates/{profileId}` and `GET /api/cv/my-profiles` — profile APIs for Candidates | 03/04/2026 | 03/04/2026 |
| Thu | Implement `GET /api/jobs/{jobId}/candidates` — ranked candidate list by job; `DELETE /api/cv/{profileId}` with authorization | 03/05/2026 | 03/05/2026 |
| Fri | Implement `CandidateService`: GetCandidatesByJob, GetCandidateDetails, UpdateCandidateStatus (RDS PostgreSQL via RDS Proxy) | 03/06/2026 | 03/06/2026 |

### Week 9 Achievements:
* Mastered Routing Choice in Step Functions: if CV upload → `JobSuggestionEngine` + `CandidateRankingEngine`; if JD upload → only update DynamoDB vector cache.
* `JobSuggestionEngine` calls Bedrock Cohere Embed v3 to embed CVs, matches vectors with cached JDs in DynamoDB, and pushes results via the `publishJobSuggestions` AppSync mutation → Candidates receive them in real-time.
* `CandidateRankingEngine` uses Claude 3.5 Sonnet to analyze and rank candidates, pushes results via `publishCandidateRanking` → Recruiter Dashboard receives them in real-time.
* Fully completed candidate profile REST APIs; security: users can only delete their own CVs (`profile.UserId != user.Id` → 403 Forbidden).