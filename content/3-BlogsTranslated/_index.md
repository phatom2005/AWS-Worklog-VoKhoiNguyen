---
title: "Translated Blog Posts"
date: 2026-02-05
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

This section lists and introduces the AWS blog posts I read and translated during my internship, related to the services used in the SmartHire AI project.

### [Blog 1 - Adaptive Sampling with AWS X-Ray to Capture Critical Spans](3.1-Blog1/)

This blog introduces the **adaptive sampling** feature in AWS X-Ray — an intelligent sampling mechanism that balances observability costs with incident detection capability. You will learn the difference between *static sampling* (fixed-rate) and *adaptive sampling* (dynamic), how to configure **Sampling Boost** to automatically increase the sampling rate when errors or latency spikes occur, and how to use **Anomaly Span Capture** to record error spans without requiring a full trace. This post is directly relevant to monitoring Lambda, API Gateway, and Step Functions in the SmartHire AI project.

### [Blog 2 - Automating AWS Systems Manager Activation for Hybrid Nodes with Lambda & DynamoDB](3.2-Blog2/)

This blog walks through building a **serverless** solution that automates the creation and management of AWS Systems Manager Hybrid Activations credentials. The solution uses **API Gateway + Lambda + DynamoDB + Parameter Store** following an event-driven pattern — the same architectural style as the SmartHire AI backend. You will learn how Lambda manages state with DynamoDB (lock/unlock mechanism), how a Private API Gateway protects internal endpoints, and how to deploy the entire solution with AWS CloudFormation.
