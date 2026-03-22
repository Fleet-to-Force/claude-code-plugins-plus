---
name: retellai-architecture-variants
description: |
  Choose and implement Retell AI validated architecture blueprints for different scales.
  Use when designing new Retell AI integrations, choosing between monolith/service/microservice
  architectures, or planning migration paths for Retell AI applications.
  Trigger with phrases like "retellai architecture", "retellai blueprint",
  "how to structure retellai", "retellai project layout", "retellai microservice".
allowed-tools: Read, Grep
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
compatible-with: claude-code
tags: [retellai, voice-ai, saas]
---
# Retell AI Architecture Variants

## Overview
Deployment architectures for Retell AI voice agents at different scales. Voice AI systems require real-time processing with strict latency budgets -- architecture choices directly impact call quality. Three validated blueprints cover prototype through enterprise scale, each with progressively lower latency budgets and more sophisticated state management.

## Prerequisites
- Retell AI account with agent configured
- Understanding of WebSocket real-time communication
- Infrastructure for voice processing latency requirements

## Decision Matrix

| Factor | Single Server | Distributed | Event-Driven |
|--------|--------------|-------------|--------------|
| Concurrent Calls | < 10 | 10-100 | 100+ |
| Latency Budget | 800ms | 500ms | 300ms |
| State | In-memory | Redis | Redis + Events |
| Scaling | Vertical | Horizontal | Auto-scaling |

## Instructions

1. **Choose architecture tier** based on concurrent call volume and latency requirements using the decision matrix above.
2. **Set up state management** -- in-memory Map for prototypes, Redis for production, Redis plus Kafka for scale. See [architecture code](references/architecture-code.md) for implementations.
3. **Configure webhook handlers** with response time budgets matching the chosen tier. All handlers must return responses within 1 second.
4. **Add response caching** at the distributed and event-driven tiers to reduce LLM calls and meet tighter latency budgets.

## Error Handling
| Issue | Cause | Solution |
|-------|-------|----------|
| Calls drop under load | Single server bottleneck | Scale to distributed architecture |
| Lost call state | Server restart | Move state to Redis |
| High latency | LLM response too slow | Pre-cache common responses |

## Examples

For complete code implementations of all three architecture tiers (single server, distributed, event-driven), including diagrams and TypeScript classes, see [architecture code](references/architecture-code.md).

## Resources
- [Retell AI Architecture](https://docs.retellai.com/guide/architecture)
- [Retell AI Docs](https://docs.retellai.com)

## Output
- Architecture tier selected based on scale requirements
- State management configured for the chosen tier
- Webhook handlers deployed with appropriate latency budgets
- Response caching implemented for production tiers
