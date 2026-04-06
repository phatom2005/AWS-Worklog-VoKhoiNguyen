---
title : "Observability & Monitoring"
date : 2026-01-05
weight : 6
chapter : false
pre : " <b> 5.8. </b> "
---

### A. Phạm vi Truy vết (Tracing Scope)

Việc đo lường (instrumentation) bằng X-Ray được áp dụng cho các luồng xử lý CV/JD:

1. **IngestionTrigger**: Theo dõi việc xác thực tệp và lời gọi Step Functions.
2. **Step Functions Workflow**: Trực quan hóa đường đi điều phối (CvJdProcessor → Engine).
3. **CvJdProcessor**: Theo dõi các lời gọi Textract (CV), đọc từ RDS (JD), và tạo vector nhúng (embedding generation).
4. **Engines**: Theo dõi các truy vấn pgvector, hoạt động ghi vào DynamoDB và quá trình tạo ảnh chụp (snapshot) bằng Claude.

### B. Các Số liệu (Metrics) Quan trọng Cần Giám sát

- **Độ trễ Toàn trình (End-to-End Latency)**: Thời gian từ lúc tải lên/tạo công việc cho đến khi hoàn thành xếp hạng.
- **Tỷ lệ Lỗi (Error Rates)**: Các lỗi rớt (failures) trong Textract, hiện tượng bóp nghẹt (throttling) ở Bedrock, hoặc timeout từ RDS.
- **Phụ thuộc Dịch vụ**: Lambda → RDS, Lambda → Bedrock, Lambda → DynamoDB.
- **Khởi động lạnh (Cold Starts)**: Tác động của việc khởi tạo Lambda đối với thông lượng xử lý.

### C. Chi tiết Triển khai

- **Quyền IAM (IAM Permissions)**: Cấp quyền `xray:PutTraceSegments` và `xray:PutTelemetryRecords` cho các role thực thi của Lambda.
- **Biến Môi trường**: `AWS_XRAY_CONTEXT_MISSING` được đặt thành `LOG_ERROR`.
- **Cấu hình Truy vết (Tracing Config)**: Kích hoạt Active tracing trên tất cả các Lambda và Step Functions state machine.