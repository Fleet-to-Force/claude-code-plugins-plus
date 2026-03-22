---
name: retellai-security-basics
description: |
  Apply Retell AI security best practices for secrets and access control.
  Use when securing API keys, implementing least privilege access,
  or auditing Retell AI security configuration.
  Trigger with phrases like "retellai security", "retellai secrets",
  "secure retellai", "retellai API key security".
allowed-tools: Read, Write, Grep
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
compatible-with: claude-code
tags: [retellai, voice-ai, saas]
---
# Retell AI Security Basics

## Overview
Security best practices for Retell AI API keys, tokens, and access control. Covers environment variable management with `.gitignore` protection, secret rotation procedures, least-privilege scope assignment per environment, webhook signature verification using HMAC-SHA256, and audit logging for compliance tracking.

## Prerequisites
- Retell AI SDK installed
- Understanding of environment variables
- Access to Retell AI dashboard

## Least Privilege Scopes

| Environment | Recommended Scopes |
|-------------|-------------------|
| Development | `read:*` |
| Staging | `read:*, write:limited` |
| Production | Only required scopes |

## Instructions

1. **Configure environment variables** with API keys in `.env` files that are git-ignored. Never commit secrets to version control. See [security patterns](references/security-patterns.md) for the setup.
2. **Set up secret rotation** by generating new keys in the Retell AI dashboard, updating environment variables, verifying connectivity, and then revoking old keys.
3. **Apply least privilege** by using separate API keys per environment with minimal scopes. Development keys should be read-only, production keys should have only the scopes required by the application.
4. **Verify webhook signatures** using HMAC-SHA256 with timing-safe comparison to prevent replay attacks on webhook endpoints.
5. **Add audit logging** to track all API operations with user ID, action, resource, and result for compliance and debugging.

## Output
- Secure API key storage in environment variables
- Environment-specific access controls with minimal scopes
- Webhook signature verification preventing unauthorized access
- Audit logging enabled for compliance tracking

## Error Handling
| Security Issue | Detection | Mitigation |
|----------------|-----------|------------|
| Exposed API key | Git scanning tools | Rotate immediately, revoke old key |
| Excessive scopes | Audit log review | Reduce to minimum required permissions |
| Missing rotation | Key age monitoring | Schedule quarterly rotation |

## Examples

For environment setup, rotation scripts, service account patterns, webhook verification, and audit logging, see [security patterns](references/security-patterns.md).

## Resources
- [Retell AI Security Guide](https://docs.retellai.com/security)
- [Retell AI API Scopes](https://docs.retellai.com/scopes)

## Next Steps
For production deployment security, see `retellai-prod-checklist`. For multi-environment key management, see `retellai-multi-env-setup`.
