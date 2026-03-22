---
name: retellai-enterprise-rbac
description: |
  Configure Retell AI enterprise SSO, role-based access control, and organization management.
  Use when implementing SSO integration, configuring role-based permissions,
  or setting up organization-level controls for Retell AI.
  Trigger with phrases like "retellai SSO", "retellai RBAC",
  "retellai enterprise", "retellai roles", "retellai permissions", "retellai SAML".
allowed-tools: Read, Write, Edit
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
compatible-with: claude-code
tags: [retellai, voice-ai, saas]
---
# Retell AI Enterprise RBAC

## Overview
Control access to Retell AI voice agents, phone numbers, and call recordings through organization-level roles and scoped API key management. Retell uses per-minute pricing for voice calls, so RBAC must govern who can create voice agents, assign phone numbers (each incurs monthly cost), access call recordings, and modify agent prompts that control conversation behavior.

## Prerequisites
- Retell AI account with team plan (per-minute call pricing)
- Organization admin access at dashboard.retellai.com
- At least one phone number provisioned

## Instructions

1. **Define role-based access** with a YAML matrix covering org_admin, agent_developer, call_operator, and auditor roles. Each role specifies permissions and restrictions. See [RBAC config](references/rbac-config.md) for the complete matrix.
2. **Create scoped API keys** per team: agent developers get `agent:read/write` + `call:read`, call center integrations get `call:create/read` only. Enforce rate limits per key matching the team's needs.
3. **Protect agent prompt changes** by storing agent configurations in git and requiring PR review for production agents. Monitor `last_modified_at` timestamps for unauthorized edits.
4. **Control phone number assignment** to agents -- restrict to org admins since each number represents monthly cost and company voice identity.
5. **Audit call recordings and transcripts** regularly. Review recent calls for cost data, duration patterns, and compliance with recording policies.

## Error Handling
| Issue | Cause | Solution |
|-------|-------|----------|
| `403` on agent update | Key missing `agent:write` scope | Create key with appropriate write scope |
| Phone number unassigned | Admin removed assignment | Reassign via phone number API |
| Call recording inaccessible | Retention policy expired | Extend retention in org settings |
| Agent prompt regression | Unauthorized edit | Store configs in git, require PR reviews |

## Examples

For role matrix YAML, scoped API key creation, prompt protection, phone number assignment, and call audit commands, see [RBAC config](references/rbac-config.md).

## Output
- Role matrix defined with per-role permissions and restrictions
- Scoped API keys created per team with rate limits
- Agent prompt changes gated behind PR review
- Phone number assignment restricted to org admins

## Resources
- [Retell AI Documentation](https://docs.retellai.com)
- [Retell AI API Keys](https://docs.retellai.com/api-references)
