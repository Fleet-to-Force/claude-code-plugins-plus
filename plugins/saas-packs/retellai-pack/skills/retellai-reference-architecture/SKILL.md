---
name: retellai-reference-architecture
description: |
  Implement Retell AI reference architecture with best-practice project layout.
  Use when designing new Retell AI integrations, reviewing project structure,
  or establishing architecture standards for Retell AI applications.
  Trigger with phrases like "retellai architecture", "retellai best practices",
  "retellai project structure", "how to organize retellai", "retellai layout".
allowed-tools: Read, Grep
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
compatible-with: claude-code
tags: [retellai, voice-ai, saas]
---
# Retell AI Reference Architecture

## Overview
Production architecture for AI voice agents with Retell AI. Covers agent design, LLM configuration, telephony integration, custom tool functions, and call analytics for conversational AI applications. The architecture connects phone/web interfaces through Retell's voice pipeline to backend webhook handlers and tool endpoints.

## Prerequisites
- Retell AI account with API key
- `retell-sdk` npm package
- Twilio or phone number for inbound/outbound calls
- WebSocket server for custom LLM (optional)

## Architecture Diagram

```
┌──────────────────────────────────────────────────────┐
│              Phone/Web Interface                      │
│  Twilio Number │ Web Call │ SIP Trunk                 │
└──────────┬───────────────────────────────────────────┘
           │
           ▼
┌──────────────────────────────────────────────────────┐
│              Retell AI Platform                       │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────┐   │
│  │ Voice Agent  │  │ LLM Engine   │  │ Voice     │   │
│  │ (config)     │  │ (prompts)    │  │ (TTS/STT) │   │
│  └──────┬───────┘  └──────┬───────┘  └───────────┘   │
│         │                 │                           │
│         ▼                 ▼                           │
│  ┌──────────────────────────────────────────────┐     │
│  │         Custom Tool Functions                 │     │
│  │  Book Appt │ Check Status │ Transfer Call    │     │
│  └──────────────────────┬───────────────────────┘     │
└─────────────────────────┼───────────────────────────┘
                          │
                          ▼
┌──────────────────────────────────────────────────────┐
│              Your Backend                             │
│  Webhook Handler │ Tool Endpoints │ Call Analytics    │
└──────────────────────────────────────────────────────┘
```

## Instructions

### Step 1: Agent and LLM Configuration
Create an LLM with a personality prompt and custom tool definitions. Configure the voice agent with voice ID, language, responsiveness, and interruption sensitivity. See [architecture examples](references/architecture-examples.md) for complete TypeScript implementation.

### Step 2: Voice Agent Setup
Configure agent voice parameters including speed, temperature, and backchannel responses. Match voice settings to the use case -- customer service agents need lower speed and higher backchannel, while notification agents can use faster speech.

### Step 3: Tool Function Endpoints
Implement webhook endpoints for custom tools (e.g., booking, status checks) and call lifecycle events. Each tool returns a result string that the agent speaks to the caller. See [architecture examples](references/architecture-examples.md).

### Step 4: Initiate Outbound Calls
Use the Retell API to create phone calls with agent ID, from/to numbers, and optional metadata for tracking campaigns or customer context.

## Error Handling
| Issue | Cause | Solution |
|-------|-------|----------|
| Agent not responding | LLM prompt too complex | Simplify prompt, use shorter responses |
| Tool call fails | Webhook URL unreachable | Verify endpoint is publicly accessible via HTTPS |
| Poor voice quality | High latency | Use gpt-4o-mini, increase responsiveness |
| Call drops | WebSocket timeout | Check network stability, add reconnection logic |

## Examples

For complete agent setup, tool endpoint implementation, webhook handlers, and outbound call code, see [architecture examples](references/architecture-examples.md).

## Resources
- [Retell AI Documentation](https://docs.retellai.com)
- [Retell Agent Setup](https://docs.retellai.com/build-agent)
- [Retell API Reference](https://docs.retellai.com/api-references)

## Output
- Architecture diagram documenting component interactions
- Agent and LLM configuration applied
- Tool function endpoints deployed and registered
- Outbound call capability verified with test call
