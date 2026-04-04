#### IAM permissions
Để triển khai luồng Candidate Flow phi máy chủ (Serverless) và sử dụng các dịch vụ AI, hãy đảm bảo tài khoản AWS (hoặc IAM User/Role) được gắn policy sau. Policy này cho phép bạn tạo và quản lý các tài nguyên cốt lõi trong bài lab.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "SmartHireServerlessLab",
            "Effect": "Allow",
            "Action": [
                "s3:*",
                "sqs:*",
                "lambda:*",
                "dynamodb:*",
                "states:*",
                "textract:*",
                "bedrock:*",
                "apigateway:*",
                "cloudformation:*",
                "cloudwatch:*",
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents",
                "iam:CreateRole",
                "iam:DeleteRole",
                "iam:AttachRolePolicy",
                "iam:DetachRolePolicy",
                "iam:PutRolePolicy",
                "iam:PassRole",
                "iam:GetRole"
            ],
            "Resource": "*"
        }
    ]
}