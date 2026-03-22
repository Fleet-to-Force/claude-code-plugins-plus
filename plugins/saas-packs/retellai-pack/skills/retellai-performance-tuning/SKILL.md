---
name: retellai-performance-tuning
description: |
  Optimize Retell AI API performance with caching, batching, and connection pooling.
  Use when experiencing slow API responses, implementing caching strategies,
  or optimizing request throughput for Retell AI integrations.
  Trigger with phrases like "retellai performance", "optimize retellai",
  "retellai latency", "retellai caching", "retellai slow", "retellai batch".
allowed-tools: Read, Write, Edit
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
compatible-with: claude-code
tags: [retellai, voice-ai, saas]
---
# Retell AI Performance Tuning

## Overview
Optimize Retell AI voice agent latency and call quality for production deployments. Covers reducing voice-to-voice latency through agent configuration tuning (responsiveness, interruption sensitivity, voice speed), LLM prompt optimization for faster responses, WebSocket connection pooling for concurrent calls, and call analytics caching with LRU eviction.

## Prerequisites
- Retell AI account with API key
- `retell-sdk` npm package installed
- WebSocket infrastructure for real-time calls
- Understanding of voice agent architecture

## Instructions

### Step 1: Optimize Agent LLM Configuration
Configure the voice agent for low latency by increasing responsiveness (0.9), raising interruption sensitivity (0.8), using a pre-cached voice, disabling ambient sound, and boosting frequently-used keywords. See [optimization patterns](references/optimization-patterns.md) for the complete agent configuration.

### Step 2: Optimize LLM Prompt for Speed
Keep the system prompt concise with explicit brevity rules: responses under 2 sentences, no filler words, one question at a time. Use `gpt-4o-mini` for faster inference than `gpt-4o`. Full prompt template in [optimization patterns](references/optimization-patterns.md).

### Step 3: Implement WebSocket Connection Pooling
Maintain a connection pool keyed by call ID to avoid reconnection overhead. Check `readyState` before reusing connections and clean up on close events. Implementation in [optimization patterns](references/optimization-patterns.md).

### Step 4: Cache Call Analytics
Use an LRU cache for completed call details since they are immutable after the call ends. Set a 15-minute TTL and only cache calls with status `ended`. This dramatically reduces API calls when building dashboards or reports.

## Error Handling
| Issue | Cause | Solution |
|-------|-------|----------|
| High voice latency | Complex LLM prompt | Shorten prompt, use gpt-4o-mini |
| WebSocket disconnect | Network instability | Implement reconnection with backoff |
| Unnatural pauses | Low responsiveness setting | Increase responsiveness to 0.8+ |
| Missed interrupts | Low sensitivity | Increase interruption_sensitivity to 0.7+ |

## Examples

For agent configuration, prompt templates, WebSocket pooling, LRU caching, and latency monitoring, see [optimization patterns](references/optimization-patterns.md).

## Resources
- [Retell AI API Reference](https://docs.retellai.com/api-references)
- [Retell Agent Configuration](https://docs.retellai.com/build-agent)
- [Voice Latency Optimization](https://docs.retellai.com/optimize-latency)

## Output
- Agent configured with optimized latency settings
- LLM prompt tuned for sub-second responses
- WebSocket connection pool deployed for concurrent calls
- Call analytics caching reducing API load by 80%+
