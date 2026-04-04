---
title: "Week 2 Worklog"
date: 2026-01-12
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Week 2 Objectives:
* Deploy network infrastructure with Amazon VPC (subnet, routing, security).
* Deploy the first application on Amazon EC2.
* Grant application permissions to access AWS services via IAM Role.
* Use AWS Cloud9 as a Cloud IDE directly in the browser.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
|-----|-----------|--------------|-----------------|----------|
| Mon | **Deploy network infrastructure with Amazon VPC** — create VPC, public/private subnets, Internet Gateway, Route Table, Security Group | 01/12/2026 | 01/12/2026 | https://cloudjourney.awsstudygroup.com/ |
| Tue | **Getting Started with Amazon EC2** — create EC2 instance, choose AMI, instance type, key pair; SSH into instance | 01/13/2026 | 01/13/2026 | https://cloudjourney.awsstudygroup.com/ |
| Wed | **Practice EC2:** deploy a simple web app on EC2, configure Security Group to open ports 80/443 | 01/14/2026 | 01/14/2026 | https://cloudjourney.awsstudygroup.com/ |
| Thu | **Grant application permissions with IAM Role** — create IAM Role, attach to EC2 Instance Profile to access S3/DynamoDB without access keys | 01/15/2026 | 01/15/2026 | https://cloudjourney.awsstudygroup.com/ |
| Fri | **Use the Cloud IDE with AWS Cloud9** — create Cloud9 environment on EC2, write and run code directly in the browser | 01/16/2026 | 01/16/2026 | https://cloudjourney.awsstudygroup.com/ |

### Week 2 Achievements:
* Successfully created a VPC with public/private subnets, configured Internet Gateway and Route Table.
* Launched an EC2 instance, connected via SSH, and successfully deployed a basic web application.
* Understood IAM Role vs IAM User: applications running on EC2 use Roles instead of hardcoded access keys (AWS best practice).
* Set up AWS Cloud9 as a cloud-based development environment, ready to replace local environments.