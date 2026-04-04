---
title: "Week 5 Worklog"
date: 2026-02-02
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Week 5 Objectives:
* Design the database schema for the candidate flow.
* Implement AuthService integrating Amazon Cognito.
* Complete the registration, email confirmation, and login flows.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date |
|-----|-----------|--------------|-----------------|
| Mon | Design `users`, `candidate_profiles` tables (Model, EF Core Migration) | 02/02/2026 | 02/02/2026 |
| Tue | Implement `AuthService.RegisterAsync` — SignUp Cognito User Pool, sync `CognitoSub` to DB | 02/03/2026 | 02/03/2026 |
| Wed | Implement `AuthService.LoginAsync` — InitiateAuth (USER_PASSWORD_AUTH) + sync user DB | 02/04/2026 | 02/04/2026 |
| Thu | Implement `EmailService`: email confirmation (ConfirmSignUp), resend OTP (ResendConfirmationCode) | 02/05/2026 | 02/05/2026 |
| Fri | Test with Postman: register → confirm email → login → receive JWT (AccessToken / IdToken) | 02/06/2026 | 02/06/2026 |

### Week 5 Achievements:
* `users` and `candidate_profiles` tables were created via EF Core Migration.
* Completed `AuthService`: registered users to Cognito User Pool, automatically synced `CognitoSub` to DB.
* `EmailService` confirmed email via OTP and resent the code when needed.
* The register → confirm → login → JWT flow works stably in the dev environment.