---
title: "Blog 2 - Tự động hóa AWS Systems Manager Activation với Lambda & DynamoDB"
date: 2026-03-09
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---
# Tự động hóa kích hoạt AWS Systems Manager cho Hybrid Node Registration

> **Nguồn gốc:** [Automate AWS Systems Manager activation for hybrid-managed node registration](https://aws.amazon.com/blogs/mt/automate-aws-systems-manager-activation-for-hybrid-managed-node-registration/) — AWS Cloud Operations Blog
>
> **Tác giả:** Justin Thomas (Sr. Cloud Support Engineer, AWS Premium Support)

---

## Giới thiệu

**AWS Systems Manager (SSM)** là dịch vụ AWS cho phép bạn xem và kiểm soát các server trên cloud AWS lẫn hạ tầng on-premises. Để đưa các server và VM trong môi trường hybrid vào quản lý của Systems Manager, bạn cần tạo **Managed Instance Activation** — bao gồm Activation Code và Activation ID.

Vấn đề hiện tại: việc tạo và quản lý thủ công các thông tin xác thực Hybrid Activations rất tẻ nhạt. Khi Activation Code hết hạn hoặc đạt giới hạn đăng ký, người quản trị phải tạo lại thủ công và cấu hình lại các server mới. Bài viết này trình bày giải pháp tự động hóa toàn bộ quy trình đó bằng **API Gateway + Lambda + DynamoDB**.

---

## Kiến trúc giải pháp

Giải pháp được triển khai thông qua **AWS CloudFormation** và bao gồm các thành phần sau:

| Dịch vụ AWS | Vai trò |
|-------------|---------|
| **Amazon API Gateway** | REST API kiểu Private, tích hợp với Lambda. Server on-premises gọi GET request để nhận Activation Code/ID |
| **AWS Lambda** | Cung cấp Hybrid Activations Code/ID cho server. Tự động tạo activation code mới nếu code cũ hết hạn hoặc đạt giới hạn đăng ký |
| **Amazon DynamoDB** | Lưu trạng thái xử lý: Lambda cập nhật bảng thành `Locked` khi đang phục vụ request, `Unlocked` sau khi hoàn thành |
| **Amazon VPC Endpoint** | Cho phép truy cập API Gateway URL riêng tư từ mạng on-premises |
| **AWS Systems Manager Parameter Store** | Lưu trữ Activation ID và Activation Code được mã hóa bằng KMS |

### Luồng thực thi

1. Web client (server on-premises) gọi private API Gateway endpoint.
2. DNS server on-premises forward request đến VPC DNS để lấy IP của VPC Endpoint.
3. Request được gửi đến private IP của VPC Endpoint.
4. Resource Policy của API Gateway kiểm tra request có đến từ VPC Endpoint không — nếu không thì từ chối (403 Forbidden).
5. API Gateway chuyển request đến Lambda.
6. Lambda cập nhật trạng thái DynamoDB thành `Locked`.
7. Lambda lấy thông tin xác thực từ Parameter Store và trả về cho client.

---

## Các bước triển khai

### Bước 1: Tạo VPC Endpoint cho API Gateway

Tạo VPC Endpoint loại Interface cho API Gateway trong VPC của bạn, gắn Security Group cho phép TCP port 443.

{{% notice tip %}}
Nếu VPC Endpoint cho API Gateway đã tồn tại trong VPC, bỏ qua bước này và ghi lại VPC Endpoint ID hiện có.
{{% /notice %}}

### Bước 2: Tạo KMS Key

Tạo **AWS KMS Key** (symmetric) để mã hóa dữ liệu trong Parameter Store, nơi lưu Activation Code và Activation ID.

### Bước 3: Tạo API Gateway và Lambda

Triển khai CloudFormation stack để tạo Private API Gateway và Lambda function. Sau khi stack tạo xong, lấy **API Gateway Invoke URL** từ phần Outputs.

Kiểm tra kết quả từ server on-premises:

```bash
curl -s https://<invoke-url>/lambdastage | jq '.'
```

Kết quả trả về dạng JSON:

```json
{
  "ActivationId": "e50a8437-23dd-4326-9e79-5e3b7573493e",
  "ActivationCode": "vVcH9zJX4ROy2XTsh5cb"
}
```

---

## Script tự động đăng ký SSM Agent

### Linux (Shell Script)

```bash
#!/bin/bash
credentials=$(curl -s https://<invoke-url>/lambdastage)
activationcode=$(echo $credentials | jq -r '.ActivationCode')
activationid=$(echo $credentials | jq -r '.ActivationId')

mkdir /tmp/ssm
curl https://s3.amazonaws.com/ec2-downloads-windows/SSMAgent/latest/linux_amd64/amazon-ssm-agent.rpm \
  -o /tmp/ssm/amazon-ssm-agent.rpm
sudo dnf install -y /tmp/ssm/amazon-ssm-agent.rpm
sudo systemctl stop amazon-ssm-agent
sudo amazon-ssm-agent -register -code $activationcode -id $activationid -region us-east-1
sudo systemctl start amazon-ssm-agent
```

### Windows (PowerShell)

```powershell
$credential = Invoke-WebRequest -URI https://<invoke-url>/lambdastage | Select-Object -ExpandProperty Content
$credentialPSObject = $credential | ConvertFrom-Json
$code = $credentialPSObject.ActivationCode
$id   = $credentialPSObject.ActivationId

Start-Process .\AmazonSSMAgentSetup.exe -ArgumentList @("/q", "/log", "install.log", "CODE=$code", "ID=$id", "REGION=us-east-1") -Wait
```

---

## Dọn dẹp tài nguyên

1. Hủy đăng ký server khỏi Systems Manager.
2. Xóa CloudFormation stack `apigateway-lambda-setup`.
3. Xóa CloudFormation stack `apigateway-vpcendpoint-setup`.
4. Xóa KMS Key đã tạo.

---

## Điểm liên quan đến dự án SmartHire AI

Bài blog này minh họa **serverless pattern quen thuộc** được áp dụng trong SmartHire AI:

| Pattern trong blog | Tương đương trong SmartHire AI |
|--------------------|--------------------------------|
| API Gateway → Lambda | API upload CV → Lambda tạo Presigned URL |
| Lambda cập nhật DynamoDB trạng thái `Locked/Unlocked` | Lambda cập nhật DynamoDB trạng thái `PROCESSING/DONE` |
| Parameter Store lưu secrets | SmartHire dùng Secrets Manager / env var Lambda |
| Private API Gateway bảo vệ endpoint | Cognito Authorizer bảo vệ endpoint SmartHire |
| CloudFormation tự động hóa deployment | SAM template tự động hóa Lambda + API Gateway |

---

## Kết luận

Giải pháp này cho thấy sức mạnh của **event-driven serverless pattern**: Lambda xử lý logic nghiệp vụ, DynamoDB quản lý trạng thái, Parameter Store bảo vệ secrets, và API Gateway làm cổng giao tiếp an toàn. Đây là pattern nền tảng mà SmartHire AI ứng dụng xuyên suốt trong CV Parsing Pipeline và AI Evaluation Workflow.
