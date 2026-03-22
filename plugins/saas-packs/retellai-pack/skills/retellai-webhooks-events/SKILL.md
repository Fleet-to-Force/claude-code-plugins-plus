---
name: retellai-webhooks-events
description: |
  Implement Retell AI webhook signature validation and event handling.
  Use when setting up webhook endpoints, implementing signature verification,
  or handling Retell AI event notifications securely.
  Trigger with phrases like "retellai webhook", "retellai events",
  "retellai webhook signature", "handle retellai events", "retellai notifications".
allowed-tools: Read, Write, Edit, Bash(curl:*)
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
compatible-with: claude-code
tags: [retellai, voice-ai, saas]
---
# Retell AI Webhooks & Events

## Overview
Handle Retell AI webhooks for real-time voice call lifecycle events. Retell AI fires webhooks when calls start, end, or encounter events during conversation. This skill covers HMAC-SHA256 signature verification, event routing for all call lifecycle states, transcript and sentiment processing, and outbound call initiation via the API.

## Prerequisites
- Retell AI account with API access
- Retell AI API key stored in `RETELL_API_KEY` environment variable
- HTTPS endpoint for receiving webhook deliveries
- Voice agent configured in Retell AI dashboard

## Webhook Event Types

| Event | Trigger | Payload |
|-------|---------|---------|
| `call_started` | AI agent picks up | Call ID, agent ID, from/to numbers |
| `call_ended` | Call completes | Call ID, duration, end reason, transcript |
| `call_analyzed` | Post-call analysis done | Sentiment, summary, custom data |
| `agent_transfer` | Transfer to human | Call ID, transfer number, context |
| `voicemail_detected` | Voicemail reached | Call ID, voicemail status |
| `call_error` | Call fails | Error code, error message |

## Instructions

### Step 1: Configure Webhook Endpoint
Set up an Express endpoint that validates HMAC-SHA256 signatures before processing any event. Use `express.raw()` middleware to access the raw body for signature verification. See [webhook handlers](references/webhook-handlers.md) for the complete implementation with timing-safe comparison.

### Step 2: Route Call Events
Implement a switch-based event router that dispatches to handler functions for each event type. Process `call_ended` for transcript storage, `call_analyzed` for sentiment-based alerting, and `agent_transfer` for human handoff tracking.

### Step 3: Process Call Results
Store call records in the database with duration, transcript, and completion timestamp. Extract action items from transcripts using LLM analysis. Route negative-sentiment calls to escalation channels. Full handler code in [webhook handlers](references/webhook-handlers.md).

### Step 4: Initiate Outbound Calls
Use the Retell API to create phone calls with agent ID, from/to numbers, and a webhook URL for receiving events. See [webhook handlers](references/webhook-handlers.md) for the curl command.

## Error Handling
| Issue | Cause | Solution |
|-------|-------|----------|
| Invalid signature | Wrong webhook secret | Verify secret in Retell AI dashboard settings |
| No transcript | Short call or error | Check `end_reason` for early termination |
| Transfer failed | Invalid transfer number | Verify transfer number is active and reachable |
| Missing analysis | Analysis not configured | Enable post-call analysis in agent settings |

## Examples

For signature verification, event routing, call processing, outbound API calls, and action item extraction, see [webhook handlers](references/webhook-handlers.md).

## Resources
- [Retell AI API Documentation](https://docs.retellai.com)
- [Retell AI Webhooks](https://docs.retellai.com/webhooks)

## Next Steps
For deployment of webhook endpoints to production platforms, see `retellai-deploy-integration`. For securing webhook secrets, see `retellai-security-basics`.

## Output
- Webhook endpoint deployed with signature verification
- Event router handling all call lifecycle events
- Call records stored with transcripts and sentiment analysis
- Outbound call capability configured and tested
