---
title: "Blog 1 - Adaptive Sampling with AWS X-Ray"
date: 2026-03-09
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Adaptive Sampling with AWS X-Ray to Capture Critical Spans

> **Source:** [Adaptive sampling with AWS X-Ray to capture critical spans](https://aws.amazon.com/blogs/mt/adaptive-sampling-with-aws-x-ray-to-capture-critical-spans/) — AWS Cloud Operations Blog
>
> **Authors:** Kartik Bheemisetty (Sr. Technical Account Manager), Dejun Hu (Sr. Software Development Engineer, AWS X-Ray)

---

## Introduction

Enterprise applications using **AWS X-Ray** generate large volumes of distributed tracing data across multiple services. Static sampling strategies keep costs down by capturing a fixed percentage of traffic. However, they frequently miss critical data during intermittent failures or sudden latency spikes. Tracing every request for maximum visibility at scale may significantly increase costs.

**Adaptive sampling** helps you control and predict costs during active incidents, latency spikes, or fault conditions. It dynamically adjusts sampling behavior based on runtime conditions, enabling you to capture more relevant traces and spans when anomalies occur.

---

## Prerequisites

- Familiarity with X-Ray concepts: traces, segments, and sampling.
- **CloudWatch Application Signals** enabled for the relevant services.
- **AWS Distro for OpenTelemetry (ADOT) SDK** — Java version 2.11.5 / Python version 0.15.0 or higher.
- Integration with the ADOT SDK and either the Amazon CloudWatch Agent or the OpenTelemetry Collector.

---

## Static Sampling vs Adaptive Sampling

**Static sampling:** X-Ray uses fixed rules that define a sampling rate, reservoir quota, and matching conditions. Effective for cost control, but does not react to runtime anomalies and can result in missing traces during short-lived failures.

**Adaptive sampling:** Builds on existing sampling rules and introduces dynamic behavior through two complementary mechanisms:

1. **Sampling Boost** — Automatically increases the sampling rate when the ADOT SDK detects anomalies.
2. **Anomaly Span Capture** — Captures critical spans even when a full trace is not sampled.

{{% notice info %}}
X-Ray relies entirely on **parent-based sampling**: the root service (first instrumented entry point) makes the sampling decision. Downstream services cannot override an upstream decision.
{{% /notice %}}

---

## How Sampling Boost Works

**Sampling Boost** is most useful in environments that use a conservative baseline sampling rate and need improved visibility into recurring anomalies without increasing overall tracing cost.

### Mechanism

X-Ray observes the number of anomalies detected within an evaluation window and estimates how much the sampling rate needs to increase to capture at least one anomalous request. The more frequently anomalies occur, the smaller the required rate increase.

Example sampling rule with Sampling Boost configured for Service A:

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

- **FixedRate: 0.01** — Baseline: only 1% of requests are sampled.
- **MaxRate: 0.80** — When boost is triggered, the sampling rate increases up to 80%.
- **CooldownWindowMinutes: 1** — After each boost, a 1-minute cooldown window applies before the next boost can occur.

### Key Characteristics

- Configured as part of an X-Ray sampling rule.
- Boost magnitude is calculated based on the number of anomalies observed.
- Each boost is temporary, followed by a cooldown period.
- The sampling rate is increased only as much as needed to capture at least one anomalous trace, never exceeding the configured maximum.

---

## How Anomaly Span Capture Works

**Anomaly Span Capture** operates independently of X-Ray sampling rules. Unlike Sampling Boost, it does not rely on sampling roots or parent-based sampling decisions. It records anomalous spans whenever they occur, based on conditions defined in local SDK configuration.

Local SDK configuration example (environment variable):

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

This configuration triggers Sampling Boost and Anomaly Span Capture when the service returns HTTP 500/501, and triggers Sampling Boost when latency exceeds 100ms.

### Key Characteristics

- Configured locally at the SDK level.
- Independent of sampling rules and sampling roots.
- Captures spans on demand rather than through probabilistic sampling.
- Produces a partial trace scoped to a single service — providing highly actionable debugging information.

---

## Using Both Together

Combining Sampling Boost and Anomaly Span Capture allows you to:
- Lower the baseline sampling rate to reduce cost.
- Capture anomalies deterministically.
- Collect additional traces opportunistically when issues persist.

---

## Monitoring Sampling Boost

When a sampling rule has Sampling Boost enabled, X-Ray automatically emits a metric named **SamplingRate**. You can monitor this metric in CloudWatch to observe when and how strongly the boost is triggered.

---

## Best Practices

| Practice | Explanation |
|----------|-------------|
| Define sampling rules per root service | Avoids aggregating sampling behavior across multiple paths; leads to more predictable boost results |
| Use a low baseline rate (FixedRate ~1%) | Adaptive sampling provides the most benefit when the baseline is low |
| Set CooldownWindow appropriately | 1 minute allows continuous boosting during persistent incidents; 10 minutes better controls cost |
| Set MaxRate based on acceptable cost | MaxRate of 0.25 (25%) provides substantial coverage without excessive cost |

---

## Conclusion

Adaptive sampling in AWS X-Ray helps you respond dynamically to runtime anomalies by combining **Sampling Boost** with **Anomaly Span Capture**. This enables collection of critical diagnostic data without increasing steady-state trace volume and cost.

For the SmartHire AI project, this technique can be applied to monitor Lambda Workers (CV parsing, Bedrock inference) and Step Functions Workflows — ensuring traces are captured during timeouts or AI inference errors without incurring excessive monitoring costs.
