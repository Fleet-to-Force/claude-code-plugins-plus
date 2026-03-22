---
name: retellai-advanced-troubleshooting
description: |
  Apply Retell AI advanced debugging techniques for hard-to-diagnose issues.
  Use when standard troubleshooting fails, investigating complex race conditions,
  or preparing evidence bundles for Retell AI support escalation.
  Trigger with phrases like "retellai hard bug", "retellai mystery error",
  "retellai impossible to debug", "difficult retellai issue", "retellai deep debug".
allowed-tools: Read, Grep, Bash(kubectl:*), Bash(curl:*), Bash(tcpdump:*)
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
compatible-with: claude-code
tags: [retellai, voice-ai, saas]
---
# Retell AI Advanced Troubleshooting

## Overview
Deep debugging techniques for complex Retell AI issues that resist standard troubleshooting. Covers systematic layer-by-layer isolation (network, DNS, TLS, auth, API, parsing), timing analysis for latency anomalies, memory leak detection, race condition identification, and structured support escalation with evidence bundles.

## Prerequisites
- Access to production logs and metrics
- kubectl access to clusters
- Network capture tools available
- Understanding of distributed tracing

## Instructions

### Step 1: Collect Evidence Bundle
Run the comprehensive debug script to gather logs, metrics, network captures, traces, and configuration state. See [debug techniques](references/debug-techniques.md) for the full collection script.

### Step 2: Systematic Isolation
Test each layer independently to identify the failure point. The six-layer test covers network connectivity, DNS resolution, TLS handshake, authentication, API response, and response parsing. Full implementation in [debug techniques](references/debug-techniques.md).

### Step 3: Create Minimal Reproduction
Strip down to the simplest failing case. Use a fresh client instance with no customization and the simplest possible API call to confirm the issue is reproducible outside the application context.

### Step 4: Analyze Timing and Resources
Attach the `TimingAnalyzer` to measure latency at each stage. Check for memory leaks by tracking heap usage over time. Detect race conditions with concurrency checkers. All patterns available in [debug techniques](references/debug-techniques.md).

### Step 5: Escalate with Evidence
Use the support escalation template with all collected evidence. Include severity level, request IDs, timestamps, steps to reproduce, and workarounds already attempted.

## Output
- Comprehensive debug bundle collected with logs, metrics, and traces
- Failure layer identified through systematic isolation
- Minimal reproduction created for support team
- Support escalation submitted with structured evidence

## Error Handling
| Issue | Cause | Solution |
|-------|-------|----------|
| Cannot reproduce | Race condition | Add timing analysis, increase sample size |
| Intermittent failure | Timing-dependent | Collect metrics over longer window |
| No useful logs | Missing instrumentation | Add structured debug logging |
| Memory growth | Resource leak | Use heap profiling at 1-minute intervals |

## Examples

### Quick Layer Test
```bash
set -euo pipefail
# Test each layer in sequence
curl -v https://api.retellai.com/health 2>&1 | grep -E "(Connected|TLS|HTTP)"
```

For complete debug scripts, timing analysis, race condition detection, and escalation templates, see [debug techniques](references/debug-techniques.md).

## Resources
- [Retell AI Support Portal](https://support.retellai.com)
- [Retell AI Status Page](https://status.retellai.com)

## Next Steps
For load testing after resolving issues, see `retellai-load-scale`. For common error quick-fixes, start with `retellai-common-errors` before escalating to advanced techniques.
