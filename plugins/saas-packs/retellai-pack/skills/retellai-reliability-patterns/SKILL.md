---
name: retellai-reliability-patterns
description: |
  Implement Retell AI reliability patterns including circuit breakers, idempotency, and graceful degradation.
  Use when building fault-tolerant Retell AI integrations, implementing retry strategies,
  or adding resilience to production Retell AI services.
  Trigger with phrases like "retellai reliability", "retellai circuit breaker",
  "retellai idempotent", "retellai resilience", "retellai fallback", "retellai bulkhead".
allowed-tools: Read, Write, Edit
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
compatible-with: claude-code
tags: [retellai, voice-ai, saas]
---
# Retell AI Reliability Patterns

## Overview
Production reliability patterns for Retell AI voice agent deployments. Voice calls are latency-critical -- any failure produces audible silence or call drops, making reliability patterns essential for production telephony. Covers WebSocket reconnection with backoff, response latency budgeting (800ms for processing, 200ms for network), call state persistence in Redis for server restarts, and concurrent call capacity management.

## Prerequisites
- Retell AI configured with production agent
- WebSocket infrastructure for real-time communication
- Redis for call state persistence
- Monitoring for sub-second latency requirements

## Instructions

### Step 1: Implement WebSocket Resilience
Add automatic reconnection with exponential backoff for WebSocket connections. Retell uses WebSockets for real-time voice, and connections can drop during calls due to network issues. Limit reconnection attempts to 3 to avoid infinite loops. See [resilience code](references/resilience-code.md) for the implementation.

### Step 2: Budget Response Latency
Set an 800ms processing budget for webhook responses, leaving 200ms for network transit within the 1-second voice-response requirement. Use Redis caching for the fast path and race against a fallback response generator. Full pattern in [resilience code](references/resilience-code.md).

### Step 3: Persist Call State
Store conversation context in Redis with a 1-hour TTL so that webhook server restarts do not lose mid-call context. Implement state recovery on reconnection and archive completed calls for analytics.

### Step 4: Manage Concurrent Calls
Track active calls and enforce capacity limits to prevent overload. Queue or reject new calls when at maximum concurrent capacity. Expose utilization metrics for monitoring dashboards.

## Error Handling
| Issue | Cause | Solution |
|-------|-------|----------|
| Audible silence | Webhook latency > 1s | Cache responses, enforce latency budget |
| Call drops | WebSocket disconnect | Auto-reconnect with exponential backoff |
| Lost context mid-call | Server restart | Persist call state in Redis with TTL |
| Calls rejected | Over concurrent limit | Track capacity, queue overflow calls |

## Examples

For WebSocket resilience, latency budgeting, call state management, capacity tracking, and dashboard metrics, see [resilience code](references/resilience-code.md).

## Resources
- [Retell AI Docs](https://docs.retellai.com)
- [Voice Agent Architecture](https://docs.retellai.com/guide/architecture)

## Output
- WebSocket connections resilient with automatic reconnection
- Webhook responses within 800ms latency budget
- Call state persisted and recoverable across server restarts
- Concurrent call capacity tracked with health metrics exposed
