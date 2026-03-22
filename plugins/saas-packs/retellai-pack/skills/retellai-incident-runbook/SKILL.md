---
name: retellai-incident-runbook
description: |
  Execute Retell AI incident response procedures with triage, mitigation, and postmortem.
  Use when responding to Retell AI-related outages, investigating errors,
  or running post-incident reviews for Retell AI integration failures.
  Trigger with phrases like "retellai incident", "retellai outage",
  "retellai down", "retellai on-call", "retellai emergency", "retellai broken".
allowed-tools: Read, Grep, Bash(kubectl:*), Bash(curl:*)
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
compatible-with: claude-code
tags: [retellai, voice-ai, saas]
---
# Retell AI Incident Runbook

## Overview
Rapid incident response procedures for Retell AI-related outages and degraded service. Provides a structured triage flow, decision tree for isolating Retell AI-side vs internal failures, error-specific remediation playbooks for 401/403/429/5xx responses, and communication templates for stakeholders.

## Prerequisites
- Access to Retell AI dashboard and status page
- kubectl access to production cluster
- Prometheus/Grafana access
- Communication channels (Slack, PagerDuty)

## Severity Levels

| Level | Definition | Response Time | Examples |
|-------|------------|---------------|----------|
| P1 | Complete outage | < 15 min | Retell AI API unreachable |
| P2 | Degraded service | < 1 hour | High latency, partial failures |
| P3 | Minor impact | < 4 hours | Webhook delays, non-critical errors |
| P4 | No user impact | Next business day | Monitoring gaps |

## Quick Triage

```bash
set -euo pipefail
# 1. Check Retell AI status
curl -s https://status.retellai.com | jq

# 2. Check integration health
curl -s https://api.yourapp.com/health | jq '.services.retellai'

# 3. Check error rate (last 5 min)
curl -s localhost:9090/api/v1/query?query=rate(retellai_errors_total[5m])

# 4. Recent error logs
kubectl logs -l app=retellai-integration --since=5m | grep -i error | tail -20
```

## Decision Tree

```
Retell AI API returning errors?
├─ YES: Is status.retellai.com showing incident?
│   ├─ YES → Wait for Retell AI to resolve. Enable fallback.
│   └─ NO → Internal integration issue. Check credentials, config.
└─ NO: Is the service healthy?
    ├─ YES → Likely resolved or intermittent. Monitor.
    └─ NO → Infrastructure issue. Check pods, memory, network.
```

## Instructions

### Step 1: Quick Triage
Run the triage commands above to identify whether the issue originates from Retell AI or internal infrastructure.

### Step 2: Follow Decision Tree
Determine if the issue is Retell AI-side or internal. This determines the remediation path and communication strategy.

### Step 3: Execute Immediate Actions
Apply the appropriate remediation for the error type. See [response procedures](references/response-procedures.md) for error-specific playbooks covering 401/403, 429, and 500/503 responses.

### Step 4: Communicate Status
Update internal and external stakeholders using the templates in [response procedures](references/response-procedures.md). Include impact assessment, current actions, and next update time.

## Output
- Issue identified and categorized by severity level
- Remediation applied per error-specific playbook
- Stakeholders notified with structured updates
- Evidence collected for postmortem analysis

## Error Handling
| Issue | Cause | Solution |
|-------|-------|----------|
| Cannot reach status page | Network issue | Use mobile network or VPN |
| kubectl fails | Auth expired | Re-authenticate with cluster credentials |
| Metrics unavailable | Prometheus down | Check backup metrics or raw logs |
| Secret rotation fails | Permission denied | Escalate to infrastructure admin |

## Examples

### One-Line Health Check
```bash
set -euo pipefail
curl -sf https://api.yourapp.com/health | jq '.services.retellai.status' || echo "UNHEALTHY"
```

For communication templates, postmortem template, and error-specific remediation scripts, see [response procedures](references/response-procedures.md).

## Resources
- [Retell AI Status Page](https://status.retellai.com)
- [Retell AI Support](https://support.retellai.com)

## Next Steps
For data handling compliance during incidents, see `retellai-data-handling`. For collecting debug evidence bundles, see `retellai-debug-bundle`.
