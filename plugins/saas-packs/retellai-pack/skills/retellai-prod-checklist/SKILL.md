---
name: retellai-prod-checklist
description: |
  Execute Retell AI production deployment checklist and rollback procedures.
  Use when deploying Retell AI integrations to production, preparing for launch,
  or implementing go-live procedures.
  Trigger with phrases like "retellai production", "deploy retellai",
  "retellai go-live", "retellai launch checklist".
allowed-tools: Read, Bash(kubectl:*), Bash(curl:*), Grep
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
compatible-with: claude-code
tags: [retellai, voice-ai, saas]
---
# Retell AI Production Checklist

## Overview
Complete checklist for deploying Retell AI integrations to production with gradual rollout and rollback procedures. Covers pre-deployment configuration verification, code quality gates, infrastructure readiness, documentation requirements, and a phased rollout strategy (canary 10% -> 50% -> 100%) with monitoring at each stage.

## Prerequisites
- Staging environment tested and verified
- Production API keys available
- Deployment pipeline configured
- Monitoring and alerting ready

## Instructions

1. **Verify pre-deployment configuration**: Production API keys in secure vault, environment variables set, API key scopes minimized (least privilege), webhook endpoints configured with HTTPS, webhook secrets stored securely.
2. **Run code quality checks**: All tests passing (`npm test`), no hardcoded credentials, error handling covers all Retell AI error types, rate limiting/backoff implemented, logging is production-appropriate.
3. **Confirm infrastructure setup**: Health check endpoint includes Retell AI connectivity, monitoring/alerting configured, circuit breaker pattern implemented, graceful degradation configured for Retell AI outages.
4. **Complete documentation**: Incident runbook created, key rotation procedure documented, rollback procedure documented, on-call escalation path defined.
5. **Execute gradual rollout** starting with canary (10%), monitoring for 10 minutes, then 50%, then 100%. See [deployment steps](references/deployment-steps.md) for kubectl commands and health check implementation.

## Alerting Thresholds

| Alert | Condition | Severity |
|-------|-----------|----------|
| API Down | 5xx errors > 10/min | P1 |
| High Latency | p99 > 5000ms | P2 |
| Rate Limited | 429 errors > 5/min | P2 |
| Auth Failures | 401/403 errors > 0 | P1 |

## Output
- Deployed Retell AI integration with verified health checks
- Monitoring active with alerting thresholds configured
- Rollback procedure tested and documented
- Gradual rollout completed at 100%

## Error Handling
For rollback procedure, health check implementation, and gradual rollout commands, see [deployment steps](references/deployment-steps.md).

## Examples

### Quick Health Verification
```bash
set -euo pipefail
curl -sf https://api.yourapp.com/health | jq '.services.retellai'
```

## Resources
- [Retell AI Status](https://status.retellai.com)
- [Retell AI Support](https://docs.retellai.com/support)

## Next Steps
For version upgrades after production deployment, see `retellai-upgrade-migration`. For monitoring setup, see `retellai-observability`.
