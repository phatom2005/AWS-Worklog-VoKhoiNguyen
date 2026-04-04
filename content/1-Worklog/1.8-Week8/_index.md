---
title: "Week 8 Worklog"
date: 2026-02-23
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Week 8 Objectives:
* Learn about AWS AppSync and the GraphQL Subscription mechanism.
* Read and understand `appsync_notifier.py` — the module for sending real-time notifications from Lambda.
* Learn about SNS for Management & Observability (CloudWatch Alarm → SNS).

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date |
|-----|-----------|--------------|-----------------|
| Mon | Learn AWS AppSync: GraphQL Schema, Resolvers, Subscriptions, IAM SigV4 auth | 02/23/2026 | 02/23/2026 |
| Tue | Understand `appsync_notifier.py`: `notify_candidate_job_suggestions` and `notify_recruiter_candidate_ranking` functions | 02/24/2026 | 02/24/2026 |
| Wed | Understand GraphQL Mutations: `publishJobSuggestions` (→ candidate) and `publishCandidateRanking` (→ recruiter) triggering corresponding Subscriptions | 02/25/2026 | 02/25/2026 |
| Thu | Learn SNS: used for Management & Observability — CloudWatch Alarm → SNS Topic → email/alert | 02/26/2026 | 02/26/2026 |
| Fri | Test AppSync mutations using Postman + AWS Console; verify Subscription receives real-time events | 02/27/2026 | 02/27/2026 |

### Week 8 Achievements:
* Understood the AppSync Subscription mechanism: FE subscribes to `onJobSuggestions(candidateId)` and `onCandidateRanking(jobId)` — when Lambda calls the mutation, FE receives real-time data via WebSocket managed by AppSync.
* `appsync_notifier.py` uses pure stdlib SigV4 (no extra dependencies needed) to authenticate with the AppSync endpoint.
* SNS is utilized for observability: receiving alerts from CloudWatch Alarms when Lambda error rates spike or the SQS DLQ receives messages.
* Clearly separated 2 notification channels: **Candidates** receive job suggestion results; **Recruiters** receive ranked candidate lists.