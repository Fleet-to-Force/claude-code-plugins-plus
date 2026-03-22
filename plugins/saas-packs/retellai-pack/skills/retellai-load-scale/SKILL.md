---
name: retellai-load-scale
description: |
  Implement Retell AI load testing, auto-scaling, and capacity planning strategies.
  Use when running performance tests, configuring horizontal scaling,
  or planning capacity for Retell AI integrations.
  Trigger with phrases like "retellai load test", "retellai scale",
  "retellai performance test", "retellai capacity", "retellai k6", "retellai benchmark".
allowed-tools: Read, Write, Edit, Bash(k6:*), Bash(kubectl:*)
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
compatible-with: claude-code
tags: [retellai, voice-ai, saas]
---
# Retell AI Load & Scale

## Overview
Load testing, scaling strategies, and capacity planning for Retell AI voice agent integrations. Voice calls demand consistent sub-second latency under load, making capacity planning critical for production deployments. This skill covers k6 test authoring, Kubernetes HPA configuration, connection pooling, and capacity estimation formulas.

## Prerequisites
- k6 load testing tool installed
- Kubernetes cluster with HPA configured
- Prometheus for metrics collection
- Test environment API keys

## Scaling Patterns

### Horizontal Scaling
```yaml
# kubernetes HPA
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: retellai-integration-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: retellai-integration
  minReplicas: 2
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Pods
      pods:
        metric:
          name: retellai_queue_depth
        target:
          type: AverageValue
          averageValue: 100
```

## Capacity Planning

### Metrics to Monitor
| Metric | Warning | Critical |
|--------|---------|----------|
| CPU Utilization | > 70% | > 85% |
| Memory Usage | > 75% | > 90% |
| Request Queue Depth | > 100 | > 500 |
| Error Rate | > 1% | > 5% |
| P95 Latency | > 1000ms | > 3000ms |

## Instructions

### Step 1: Create Load Test Script
Write a k6 test script with appropriate thresholds for voice API latency. See [load testing examples](references/load-testing-examples.md) for complete k6 scripts and connection pooling patterns.

### Step 2: Configure Auto-Scaling
Set up HPA with CPU and custom metrics (e.g., queue depth). Configure `minReplicas` to at least 2 for high-availability voice services.

### Step 3: Run Load Test
Execute the test and collect metrics. Start with low virtual user counts and ramp gradually to identify the saturation point without triggering rate limits.

### Step 4: Analyze and Document
Record results using the benchmark template in [load testing examples](references/load-testing-examples.md). Compare P95 latency against the 1-second voice-response budget.

## Output
- Load test script created and validated
- HPA configured with CPU and custom metrics
- Benchmark results documented with baseline metrics
- Capacity recommendations defined with scaling thresholds

## Error Handling
| Issue | Cause | Solution |
|-------|-------|----------|
| k6 timeout | Rate limited | Reduce RPS or add jitter |
| HPA not scaling | Wrong metrics | Verify custom metric name in Prometheus |
| Connection refused | Pool exhausted | Increase pool size or add queue |
| Inconsistent results | Warm-up needed | Add ramp-up phase before steady state |

## Examples

For complete k6 scripts, connection pooling code, capacity estimation functions, and benchmark templates, see [load testing examples](references/load-testing-examples.md).

```bash
# Quick k6 smoke test
k6 run --vus 10 --duration 30s retellai-load-test.js
```

## Resources
- [k6 Documentation](https://k6.io/docs/)
- [Kubernetes HPA](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
- [Retell AI Rate Limits](https://docs.retellai.com/rate-limits)

## Next Steps
For reliability patterns including circuit breakers and graceful degradation, see `retellai-reliability-patterns`. For performance tuning at the application level, see `retellai-performance-tuning`.
