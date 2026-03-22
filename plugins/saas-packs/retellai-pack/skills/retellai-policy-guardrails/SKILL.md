---
name: retellai-policy-guardrails
description: |
  Implement Retell AI lint rules, policy enforcement, and automated guardrails.
  Use when setting up code quality rules for Retell AI integrations, implementing
  pre-commit hooks, or configuring CI policy checks for Retell AI best practices.
  Trigger with phrases like "retellai policy", "retellai lint",
  "retellai guardrails", "retellai best practices check", "retellai eslint".
allowed-tools: Read, Write, Edit, Bash(npx:*)
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
compatible-with: claude-code
tags: [retellai, voice-ai, saas]
---
# Retell AI Policy Guardrails

## Overview
Policy enforcement for Retell AI voice agent deployments. Voice AI agents interact with real people over phone calls, requiring strict controls around conversation content, call recording consent, data handling, and cost management. This skill implements content filtering for blocked topics, two-party recording consent, per-minute cost controls with daily budgets, and human escalation triggers.

## Prerequisites
- Retell AI agent configured
- Understanding of telephony compliance (TCPA, GDPR)
- Call recording consent requirements for applicable jurisdictions

## Instructions

### Step 1: Define Conversation Content Boundaries
Create a policy object listing blocked topics (medical/legal/financial advice, political opinions, competitor pricing) and required disclosures (AI identity, recording notice). Apply content filtering to every agent response before delivery. See [policy code](references/policy-code.md) for the enforcement function.

### Step 2: Implement Call Recording Consent
Deliver a consent disclosure on the first turn of every call. Check the caller's response on the second turn and disable recording if consent is refused. This satisfies two-party consent jurisdictions. Full webhook implementation in [policy code](references/policy-code.md).

### Step 3: Enforce Cost Controls
Cap concurrent calls and maximum call duration to prevent runaway costs. Track daily spending against a budget threshold and block new calls when the budget is exhausted. Voice calls cost per minute, so a stuck call can burn through budget quickly.

### Step 4: Configure Human Escalation Triggers
Detect when callers request a human agent using trigger phrases ("speak to a human", "supervisor", "manager") and initiate a graceful transfer. Log escalation events for reporting.

## Error Handling
| Issue | Cause | Solution |
|-------|-------|----------|
| Compliance violation | Agent gives restricted advice | Apply content filtering on all responses |
| Recording without consent | Missing disclosure | Enforce consent script on first turn |
| Runaway call costs | No duration/budget limits | Set `maxCallDuration` and daily budget cap |
| Frustrated caller | No human escalation | Detect escalation triggers, transfer promptly |

## Examples

For content filtering, consent workflows, cost control classes, escalation detection, and policy dashboards, see [policy code](references/policy-code.md).

## Resources
- [Retell AI Docs](https://docs.retellai.com)
- [TCPA Compliance](https://www.fcc.gov/general/telemarketing-and-robocalls)

## Output
- Content policy enforced on all agent responses
- Recording consent collected on every call
- Cost controls active with daily budget monitoring
- Human escalation triggers configured and logged
