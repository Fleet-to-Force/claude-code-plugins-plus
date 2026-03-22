---
name: retellai-deploy-integration
description: |
  Deploy Retell AI integrations to Vercel, Fly.io, and Cloud Run platforms.
  Use when deploying Retell AI-powered applications to production,
  configuring platform-specific secrets, or setting up deployment pipelines.
  Trigger with phrases like "deploy retellai", "retellai Vercel",
  "retellai production deploy", "retellai Cloud Run", "retellai Fly.io".
allowed-tools: Read, Write, Edit, Bash(vercel:*), Bash(fly:*), Bash(gcloud:*)
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
compatible-with: claude-code
tags: [retellai, voice-ai, saas]
---
# Retell AI Deploy Integration

## Overview
Deploy Retell AI voice agent applications to production platforms. Covers configuring voice agent webhooks, deploying WebSocket endpoints for real-time audio streaming, and managing API credentials across Fly.io, Cloud Run, and Vercel. Voice agents require persistent WebSocket connections, making platform choice critical -- serverless functions timeout mid-call.

## Prerequisites
- Retell AI API key stored in `RETELL_API_KEY` environment variable
- Voice agent configured in Retell AI dashboard
- HTTPS endpoint for webhooks and WebSocket connections
- Platform CLI installed (vercel, fly, or gcloud)

## Instructions

1. **Configure secrets** on the deployment platform. Use `fly secrets set` for Fly.io or `gcloud secrets create` for Cloud Run. Never commit API keys to source control.
2. **Deploy the WebSocket server** with persistent connections enabled. Fly.io is recommended because it supports long-lived WebSocket connections natively. See [deployment configs](references/deployment-configs.md) for server code and `fly.toml` configuration.
3. **Set up webhook endpoints** for receiving call lifecycle events (call_ended, call_analyzed). Process events asynchronously after returning a 200 response.
4. **Register the webhook URL** with the Retell AI agent using the API. Point both `webhook_url` and `websocket_url` to the deployed endpoints.

## Error Handling
| Issue | Cause | Solution |
|-------|-------|----------|
| WebSocket disconnect | Server restart | Use `min_machines_running: 1` on Fly.io |
| Audio latency | Wrong region | Deploy close to Retell AI servers (US East) |
| Webhook signature fail | Wrong secret | Verify secret in Retell AI dashboard |
| Call quality issues | Network jitter | Use dedicated VM, not serverless |

## Examples

For WebSocket server code, Fly.io config, webhook handlers, and agent registration commands, see [deployment configs](references/deployment-configs.md).

## Resources
- [Retell AI Documentation](https://docs.retellai.com)
- [Retell AI WebSocket API](https://docs.retellai.com/websocket)
- [Fly.io WebSocket Guide](https://fly.io/docs/reference/runtime-environment/#websocket)

## Next Steps
For multi-environment configuration (dev/staging/prod), see `retellai-multi-env-setup`. For production readiness verification, see `retellai-prod-checklist`.

## Output
- Voice agent server deployed with WebSocket support
- Webhook endpoints registered and receiving events
- API secrets configured securely on the platform
- Health checks verified for the deployed service
