---
title: "Blog 1 - Adaptive Sampling với AWS X-Ray"
date: 2026-03-09
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---


# Adaptive Sampling với AWS X-Ray để bắt giữ các span quan trọng

> **Nguồn gốc:** [Adaptive sampling with AWS X-Ray to capture critical spans](https://aws.amazon.com/blogs/mt/adaptive-sampling-with-aws-x-ray-to-capture-critical-spans/) — AWS Cloud Operations Blog
>
> **Tác giả:** Kartik Bheemisetty (Sr. Technical Account Manager), Dejun Hu (Sr. Software Development Engineer, AWS X-Ray)

---

## Giới thiệu

Các ứng dụng doanh nghiệp sử dụng **AWS X-Ray** tạo ra lượng lớn dữ liệu distributed tracing trên nhiều service. Chiến lược lấy mẫu tĩnh (static sampling) giúp kiểm soát chi phí bằng cách chỉ ghi lại một tỷ lệ cố định các request. Tuy nhiên, cách tiếp cận này thường bỏ sót dữ liệu quan trọng trong các sự cố ngắn hoặc khi độ trễ đột ngột tăng. Tracing toàn bộ request để có tầm nhìn tối đa lại có thể làm tăng chi phí đáng kể.

**Adaptive sampling** (lấy mẫu thích nghi) giúp bạn kiểm soát và dự đoán chi phí trong các sự cố, đột biến độ trễ hoặc điều kiện lỗi. Cơ chế này tự động điều chỉnh hành vi lấy mẫu dựa trên điều kiện runtime, cho phép thu thập nhiều trace và span có giá trị hơn khi phát hiện bất thường.

---

## Điều kiện tiên quyết

- Quen thuộc với các khái niệm X-Ray: traces, segments, và sampling.
- Đã bật **CloudWatch Application Signals** cho các service liên quan.
- Sử dụng **AWS Distro for OpenTelemetry (ADOT) SDK** phiên bản Java 2.11.5 / Python 0.15.0 trở lên.
- Tích hợp ADOT SDK với Amazon CloudWatch Agent hoặc OpenTelemetry Collector.

---

## Static Sampling vs Adaptive Sampling

**Static sampling** (lấy mẫu tĩnh): X-Ray dùng các rule cố định xác định tỷ lệ lấy mẫu, reservoir quota và điều kiện matching. Cách này hiệu quả cho kiểm soát chi phí nhưng không phản ứng với bất thường runtime và có thể bỏ sót trace trong các sự cố ngắn.

**Adaptive sampling** (lấy mẫu thích nghi): Xây dựng trên các sampling rule hiện có và bổ sung hành vi động thông qua hai cơ chế bổ trợ nhau:

1. **Sampling Boost** — Tự động tăng tỷ lệ lấy mẫu khi ADOT SDK phát hiện bất thường.
2. **Anomaly Span Capture** — Bắt giữ các span quan trọng ngay cả khi trace đầy đủ không được lấy mẫu.

{{% notice info %}}
X-Ray hoạt động hoàn toàn dựa trên **parent-based sampling**: service root (điểm vào đầu tiên) ra quyết định lấy mẫu. Các service downstream không thể ghi đè quyết định này.
{{% /notice %}}

---

## Cách hoạt động của Sampling Boost

**Sampling Boost** hữu ích trong môi trường dùng baseline sampling thấp và cần cải thiện khả năng nhìn thấy các bất thường tái diễn mà không tăng chi phí tracing tổng thể.

### Cơ chế hoạt động

X-Ray quan sát số lượng bất thường trong một cửa sổ đánh giá và ước tính mức tăng tỷ lệ lấy mẫu cần thiết để capture được ít nhất một request bất thường. Bất thường càng xảy ra thường xuyên, mức tăng tỷ lệ lấy mẫu cần thiết càng thấp.

Ví dụ cấu hình sampling rule có Sampling Boost cho Service A:

```json
{
  "RuleName": "test",
  "Priority": 1,
  "ReservoirSize": 0,
  "FixedRate": 0.01,
  "ServiceName": "ServiceA",
  "ServiceType": "*",
  "Host": "*",
  "HTTPMethod": "*",
  "URLPath": "*",
  "SamplingRateBoost": {
    "MaxRate": 0.80,
    "CooldownWindowMinutes": 1
  }
}
```

- **FixedRate: 0.01** — Bình thường chỉ lấy mẫu 1% request.
- **MaxRate: 0.80** — Khi boost kích hoạt, tỷ lệ lấy mẫu tăng tối đa lên 80%.
- **CooldownWindowMinutes: 1** — Sau mỗi lần boost, hệ thống chờ 1 phút trước khi boost tiếp theo.

### Đặc điểm chính

- Cấu hình như một phần của X-Ray sampling rule.
- Biên độ boost được tính toán dựa trên số lượng bất thường quan sát được.
- Mỗi lần boost là tạm thời và theo sau bởi khoảng cooldown.
- Tỷ lệ lấy mẫu chỉ tăng đủ để capture ít nhất một trace bất thường, không vượt quá mức tối đa đã cấu hình.

---

## Cách hoạt động của Anomaly Span Capture

**Anomaly Span Capture** hoạt động độc lập với X-Ray sampling rule. Khác với Sampling Boost, cơ chế này không phụ thuộc vào sampling roots hay parent-based sampling. Nó bắt giữ span bất thường bất cứ khi nào chúng xảy ra, dựa trên điều kiện được định nghĩa trong cấu hình local.

Cấu hình local cho ADOT SDK (biến môi trường):

```yaml
AWS_XRAY_ADAPTIVE_SAMPLING_CONFIG="{
  version: 1.0,
  anomalyConditions: [
    {errorCodeRegex: \"^(500|501)$\", usage: \"both\"},
    {highLatencyMs: 100, usage: \"sampling-boost\"}
  ],
  anomalyCaptureLimit: {anomalyTracesPerSecond: 1}
}"
```

Cấu hình này sẽ kích hoạt Sampling Boost và Anomaly Span Capture khi service trả về HTTP 500/501, và kích hoạt Sampling Boost khi độ trễ vượt 100ms.

### Đặc điểm chính

- Cấu hình local ở cấp độ SDK.
- Độc lập với sampling rules và sampling roots.
- Bắt giữ span theo yêu cầu thay vì qua probabilistic sampling.
- Tạo ra partial trace giới hạn trong một service — cung cấp thông tin debug rất cụ thể.

---

## Kết hợp Sampling Boost và Anomaly Span Capture

Kết hợp hai cơ chế này giúp:
- Giảm baseline sampling rate để tiết kiệm chi phí.
- Bắt giữ bất thường một cách xác định (deterministic).
- Thu thập thêm trace một cách cơ hội khi sự cố kéo dài.

---

## Giám sát Sampling Boost

Khi bạn định nghĩa một sampling rule có Sampling Boost, X-Ray tự động phát ra metric với tên **SamplingRate**. Bạn có thể xem metric này trong CloudWatch để theo dõi khi nào và mức độ boost được kích hoạt.

---

## Best Practices

| Thực hành | Giải thích |
|-----------|-----------|
| Định nghĩa sampling rule per root service | Tránh aggregate hành vi sampling trên nhiều path khác nhau, cho kết quả dự đoán được hơn |
| Dùng baseline sampling thấp (FixedRate ~1%) | Adaptive sampling mang lại lợi ích lớn nhất khi baseline thấp |
| Cấu hình CooldownWindow phù hợp | 1 phút cho phép boost liên tục trong sự cố kéo dài; 10 phút giúp kiểm soát chi phí tốt hơn |
| Thiết lập MaxRate theo mức chi phí chấp nhận được | MaxRate 0.25 (25%) cung cấp coverage tốt mà không tốn kém quá mức |

---

## Kết luận

Adaptive sampling trong AWS X-Ray giúp bạn phản ứng linh hoạt với bất thường runtime bằng cách kết hợp **Sampling Boost** với **Anomaly Span Capture**. Giải pháp này thu thập dữ liệu chẩn đoán quan trọng mà không tăng steady-state trace volume và chi phí.

Đối với dự án SmartHire AI, kỹ thuật này có thể được áp dụng để giám sát Lambda Worker (bóc tách CV, gọi Bedrock) và Step Functions Workflow, đảm bảo bắt giữ được trace trong trường hợp timeout hoặc lỗi AI inference mà không phát sinh chi phí monitoring quá cao.
