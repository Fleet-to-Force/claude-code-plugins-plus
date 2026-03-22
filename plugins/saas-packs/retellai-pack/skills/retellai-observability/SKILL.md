---
name: retellai-observability
description: |
  Set up comprehensive observability for Retell AI integrations with metrics, traces, and alerts.
  Use when implementing monitoring for Retell AI operations, setting up dashboards,
  or configuring alerting for Retell AI integration health.
  Trigger with phrases like "retellai monitoring", "retellai metrics",
  "retellai observability", "monitor retellai", "retellai alerts", "retellai tracing".
allowed-tools: Read, Write, Edit
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
compatible-with: claude-code
tags: [retellai, voice-ai, saas]
---
# Retell AI Observability

## Overview
Monitor Retell AI voice agent performance, call quality, and costs in production. Key signals include call completion rate (successful vs dropped/failed calls), average call duration, conversational latency (time between user speech and agent response), per-minute cost tracking by agent, and goal achievement rate. Short calls (<10s) often indicate agent prompt issues where the bot fails to engage.

## Prerequisites
- Retell AI account with active voice agents
- API access for call data queries
- Webhook endpoint for real-time call events

## Instructions

1. **Set up webhook metrics emission** by instrumenting the call event handler to emit counters for calls, errors, and costs, plus histograms for call duration. See [monitoring configs](references/monitoring-configs.md) for the webhook handler code.
2. **Track conversational latency** by querying the Retell API for `avg_agent_response_latency_ms` on recent calls. Latency above 2 seconds degrades call quality.
3. **Build per-agent performance reports** that calculate completion rate, average duration, and total cost for each agent. Emit as Prometheus gauges for dashboard visualization.
4. **Configure Prometheus alert rules** for high drop rate (>10%), high latency (P95 >2s), cost spikes (>$50/hour), and short calls (<10s at P25). Full alert YAML in [monitoring configs](references/monitoring-configs.md).
5. **Create dashboard panels** covering call volume by agent, completion rate, duration distribution, cost trends, latency percentiles, and disconnect reason breakdown.

## Error Handling
| Issue | Cause | Solution |
|-------|-------|----------|
| High call drop rate | Agent prompt causing hang-ups | Review and simplify agent greeting prompt |
| Latency >2 seconds | LLM response slow | Use faster model or reduce prompt complexity |
| Unexpected high costs | Long average call duration | Add conversation time limits in agent config |
| No webhook events | Endpoint unreachable | Verify webhook URL and SSL certificate |

## Examples

For webhook handler, latency queries, performance reports, and Prometheus alert rules, see [monitoring configs](references/monitoring-configs.md).

## Output
- Call quality metrics emitted from webhook handler
- Per-agent performance reports generated automatically
- Alert rules configured for drop rate, latency, cost, and duration anomalies
- Dashboard panels visualizing all key voice agent metrics

## Resources
- [Retell AI API Reference](https://docs.retellai.com/api-references)
- [Prometheus Alerting Rules](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/)
