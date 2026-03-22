---
name: retellai-rate-limits
description: |
  Implement Retell AI rate limiting, backoff, and idempotency patterns.
  Use when handling rate limit errors, implementing retry logic,
  or optimizing API request throughput for Retell AI.
  Trigger with phrases like "retellai rate limit", "retellai throttling",
  "retellai 429", "retellai retry", "retellai backoff".
allowed-tools: Read, Write, Edit
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
compatible-with: claude-code
tags: [retellai, voice-ai, saas]
---
# Retell AI Rate Limits

## Overview
Handle Retell AI rate limits gracefully with exponential backoff, jitter, and idempotency keys. Retell AI enforces per-minute and per-day request limits that vary by pricing tier (Free: 60/min, Pro: 300/min, Enterprise: 1000/min). Proper rate limit handling prevents dropped voice calls and ensures reliable API interactions under load.

## Prerequisites
- Retell AI SDK installed
- Understanding of async/await patterns
- Access to rate limit response headers

## Rate Limit Tiers

| Tier | Requests/min | Requests/day | Burst |
|------|-------------|--------------|-------|
| Free | 60 | 1,000 | 10 |
| Pro | 300 | 10,000 | 50 |
| Enterprise | 1,000 | 100,000 | 200 |

## Instructions

1. **Add exponential backoff** with jitter to all API calls. Start at 1 second, double on each retry, cap at 32 seconds, and add random jitter to prevent thundering herd. See [rate limit code](references/rate-limit-code.md) for the implementation.
2. **Use idempotency keys** for non-GET requests to make retries safe. Generate deterministic keys from operation parameters using SHA-256 hashing.
3. **Configure queue-based throttling** using p-queue with concurrency and interval limits matching the plan tier. This prevents bursts from triggering 429 responses.
4. **Monitor rate limit headers** from API responses. Track `X-RateLimit-Remaining` and proactively throttle when remaining requests drop below 5.

## Output
- Reliable API calls with automatic retry on 429 and 5xx
- Idempotent requests preventing duplicate operations on retry
- Rate limit headers monitored with proactive throttling
- Request queue enforcing per-tier concurrency limits

## Error Handling
| Header | Description | Action |
|--------|-------------|--------|
| X-RateLimit-Limit | Max requests allowed | Monitor usage against limit |
| X-RateLimit-Remaining | Remaining requests | Throttle when below 5 |
| X-RateLimit-Reset | Reset timestamp (epoch) | Wait until reset if exhausted |
| Retry-After | Seconds to wait | Honor this value before retrying |

## Examples

For exponential backoff, idempotency key generation, p-queue configuration, and rate limit monitoring, see [rate limit code](references/rate-limit-code.md).

## Resources
- [Retell AI Rate Limits](https://docs.retellai.com/rate-limits)
- [p-queue Documentation](https://github.com/sindresorhus/p-queue)

## Next Steps
For security configuration including API key rotation, see `retellai-security-basics`. For handling rate limits during load testing, see `retellai-load-scale`.
