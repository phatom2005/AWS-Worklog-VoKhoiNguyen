---
title: "Week 4 Worklog"
date: 2026-01-26
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 Objectives:
* Learn system monitoring with Amazon CloudWatch.
* Configure integrated DNS with Amazon Route53.
* Use AWS CLI to manage resources from the command line.
* Synthesize knowledge through the Highly Available Web Application practice.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
|-----|-----------|--------------|-----------------|----------|
| Mon | **Create System Monitor with Amazon CloudWatch** — create Dashboard, Alarm (CPU/Memory), Logs Insights; set up alerts via SNS | 01/26/2026 | 01/26/2026 | https://cloudjourney.awsstudygroup.com/ |
| Tue | **Set up hybrid DNS with Amazon Route53** — create Hosted Zone, A/CNAME records, learn Routing Policies (Latency, Failover) | 01/27/2026 | 01/27/2026 | https://cloudjourney.awsstudygroup.com/ |
| Wed | **Using AWS CLI on Amazon EC2** — practice CLI commands to manage EC2, S3, IAM; configure profile and region | 01/28/2026 | 01/28/2026 | https://cloudjourney.awsstudygroup.com/ |
| Thu | **Highly Available Web Application Workshop** — deploy a web app on Multi-AZ with ALB, Auto Scaling Group, RDS Multi-AZ | 01/29/2026 | 01/29/2026 | https://cloudjourney.awsstudygroup.com/ |
| Fri | Synthesize SmartHire-AI architecture, read source code: `template.yaml`, `main.tf`, `ingestion_trigger.py`; map 4 weeks of knowledge into the real project | 01/30/2026 | 01/30/2026 | — |

### Week 4 Achievements:
* Created CloudWatch Dashboard + Alarms with SNS alerts — aligning with how SmartHire uses CloudWatch to monitor Lambda and ECS.
* Configured Route53 Hosted Zone and understood the DNS resolution flow — the foundation for the project's `smarthire-ai.org` domain.
* Mastered basic AWS CLI: managed S3, EC2, IAM from the terminal.
* Completed the Highly Available Web App Workshop: understood the ALB + Auto Scaling + RDS Multi-AZ architecture — aligning with SmartHire infrastructure (ECS Fargate + RDS Multi-AZ + RDS Proxy).
* Ready to start coding the Candidate Flow module from week 5.