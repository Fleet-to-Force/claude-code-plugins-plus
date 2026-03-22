---
name: retellai-known-pitfalls
description: |
  Identify and avoid Retell AI anti-patterns and common integration mistakes.
  Use when reviewing Retell AI code for issues, onboarding new developers,
  or auditing existing Retell AI integrations for best practices violations.
  Trigger with phrases like "retellai mistakes", "retellai anti-patterns",
  "retellai pitfalls", "retellai what not to do", "retellai code review".
allowed-tools: Read, Grep
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
compatible-with: claude-code
tags: [retellai, voice-ai, saas]
---
# Retell AI Known Pitfalls

## Overview
Real gotchas when building voice AI agents with Retell AI. Retell handles telephony, speech-to-text, and text-to-speech in a WebSocket pipeline -- latency sensitivity and audio-specific failure modes require different thinking than typical REST APIs. This skill documents the top four anti-patterns with bad/good code comparisons and explains why each one causes production issues.

## Prerequisites
- Retell AI API key and agent configured
- Understanding of WebSocket-based communication
- Awareness of telephony concepts (PSTN, SIP)

## Instructions

### Step 1: Manage Voice Latency Budget
Retell pipelines must respond in under 1 second for natural conversation. Avoid synchronous database queries and external LLM calls in the webhook handler hot path. Pre-compute context, cache responses in Redis, and defer heavy processing to background tasks. See [anti-patterns](references/anti-patterns.md) for bad vs good code comparison.

### Step 2: Handle Call State Transitions
Calls can disconnect at any point. Track active calls in a Map, clean up resources on `call_ended` events, and run a periodic cleanup timer for missed end events (zombie sessions). Full pattern in [anti-patterns](references/anti-patterns.md).

### Step 3: Configure Audio Quality
Poor audio input causes misrecognition. Always configure `ambient_sound`, `responsiveness`, `interruption_sensitivity`, and `enable_backchannel` rather than relying on defaults. Defaults are optimized for quiet rooms, not real-world call environments.

### Step 4: Respect Concurrent Call Limits
Retell enforces concurrent call limits per plan. Track active concurrent calls and return 429 when at capacity, rather than letting the API fail with a cryptic error. Check plan limits and implement pre-flight capacity checks.

## Error Handling
| Issue | Cause | Solution |
|-------|-------|----------|
| Awkward silences | Webhook response > 1s | Cache context, respond under 200ms |
| Misrecognition | Background noise | Set `ambient_sound` config explicitly |
| Zombie sessions | Missed `call_ended` events | Run periodic cleanup timer (60s interval) |
| Calls rejected | Hit concurrent limit | Track active calls, queue overflow |
| One-sided audio | WebSocket connection drop | Implement reconnection logic with backoff |

## Examples

For complete bad-vs-good code comparisons covering latency, state management, audio config, and concurrency, see [anti-patterns](references/anti-patterns.md).

## Resources
- [Retell AI Docs](https://docs.retellai.com)
- [Voice Agent Best Practices](https://docs.retellai.com/guide/best-practices)

## Output
- Webhook handler optimized for sub-200ms response time
- Call state tracking implemented with cleanup for zombie sessions
- Audio quality configured for real-world environments
- Concurrent call limits enforced with capacity tracking
