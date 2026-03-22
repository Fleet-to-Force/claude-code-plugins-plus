---
name: retellai-cost-tuning
description: |
  Optimize Retell AI costs through tier selection, sampling, and usage monitoring.
  Use when analyzing Retell AI billing, reducing API costs,
  or implementing usage monitoring and budget alerts.
  Trigger with phrases like "retellai cost", "retellai billing",
  "reduce retellai costs", "retellai pricing", "retellai expensive", "retellai budget".
allowed-tools: Read, Grep
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
compatible-with: claude-code
tags: [retellai, voice-ai, saas]
---
# Retell AI Cost Tuning

## Overview
Reduce Retell AI voice agent costs by optimizing call duration, choosing the right LLM backend, and implementing conversation design patterns that resolve calls faster. Retell charges per minute of voice call with rates varying by model and voice quality. A verbose greeting adds 5 seconds per call, which at 1000 calls/month wastes 83 billable minutes.

## Prerequisites
- Retell AI dashboard access with billing visibility
- Understanding of agent call patterns and average durations
- Ability to modify agent prompts and configurations

## Instructions

1. **Analyze call duration distribution** by agent. Query the Retell API for recent calls and group by agent to find which agents have the highest average duration and cost. See [cost optimization](references/cost-optimization.md) for the jq query.
2. **Set maximum call duration** on each agent to prevent runaway costs from stuck or looping conversations. A 5-minute cap limits a single call to ~$0.50.
3. **Optimize conversation design** for brevity: shorten greetings, use quick confirmations ("Got it"), and minimize closing pleasantries. Each pattern saves 5-10 seconds per call.
4. **Use cheaper LLM backends** for simple tasks. Route FAQ and appointment scheduling through fast/cheap models, reserving capable models for complex sales or support scenarios.
5. **Monitor daily costs** with automated queries that track calls today, total cost, average cost per call, and projected monthly spend.

## Error Handling
| Issue | Cause | Solution |
|-------|-------|----------|
| High per-call cost | Agent prompts too verbose | Shorten greetings, confirmations, closings |
| Stuck calls burning minutes | Agent in conversation loop | Set `max_call_duration_seconds` |
| Cost spike from one agent | Agent handling unexpected topic | Add fallback to transfer to human |
| Budget exceeded | No daily spending cap | Run daily cost monitoring with alerts |

## Examples

For call analysis queries, duration caps, conversation design patterns, LLM selection, and daily cost monitoring, see [cost optimization](references/cost-optimization.md).

## Output
- Cost analysis completed per agent with highest-cost agents identified
- Maximum call duration configured on all agents
- Conversation prompts optimized for brevity
- Daily cost monitoring with projected monthly spend

## Resources
- [Retell AI Pricing](https://www.retellai.com/pricing)
- [Retell AI API Reference](https://docs.retellai.com/api-references)
