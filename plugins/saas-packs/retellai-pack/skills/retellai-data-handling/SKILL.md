---
name: retellai-data-handling
description: |
  Implement Retell AI PII handling, data retention, and GDPR/CCPA compliance patterns.
  Use when handling sensitive data, implementing data redaction, configuring retention policies,
  or ensuring compliance with privacy regulations for Retell AI integrations.
  Trigger with phrases like "retellai data", "retellai PII",
  "retellai GDPR", "retellai data retention", "retellai privacy", "retellai CCPA".
allowed-tools: Read, Write, Edit
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
compatible-with: claude-code
tags: [retellai, voice-ai, saas]
---
# Retell AI Data Handling

## Overview
Manage voice call data from Retell AI agents with compliance and privacy controls. Covers call recording consent configuration via agent prompts, transcript PII redaction using regex patterns (phone, SSN, card, email, ZIP), call data retention policies with automated expiration cleanup, and webhook data filtering to ensure PII never reaches persistent storage.

## Prerequisites
- Retell AI account with API key
- `retell-sdk` npm package
- Understanding of call recording laws (varies by jurisdiction)
- Database for call record storage

## Instructions

### Step 1: Configure Call Recording Consent
Set up the voice agent with a mandatory consent disclosure at the start of every call. Configure the `begin_message` to include the recording notice, and handle caller refusal with a transfer to a human agent. See [data patterns](references/data-patterns.md) for the consent agent implementation.

### Step 2: Implement PII Redaction
Apply regex-based PII detection to all transcript text before storage. The pattern set covers phone numbers, SSNs, credit card numbers, email addresses, and ZIP codes. Always redact before writing to any persistent store or log. Full pattern definitions in [data patterns](references/data-patterns.md).

### Step 3: Set Retention Policies
Define retention periods (default 90 days) and calculate expiration dates from call end timestamps. Schedule automated cleanup to delete expired recordings and clear transcript data. Implementation details in [data patterns](references/data-patterns.md).

### Step 4: Filter Webhook Data
Apply redaction in the webhook handler before storing call records. Transform Retell AI's raw transcript objects into redacted format and attach retention metadata before persisting.

## Error Handling
| Issue | Cause | Solution |
|-------|-------|----------|
| PII in stored transcripts | No redaction applied | Always redact before storage, audit existing data |
| Missing consent | Agent skips consent prompt | Include consent in `begin_message` field |
| Recordings not deleted | No retention enforcement | Schedule cleanup cron for expired records |
| Caller PII in tool args | Phone/email passed to tools | Redact tool argument logs separately |

## Examples

### Quick Compliance Check
```typescript
const report = await complianceReport(records);
console.log(`Total calls: ${report.totalCalls}`);
console.log(`With recordings: ${report.withRecordings}`);
console.log(`Expiring this week: ${report.expiringThisWeek}`);
```

For consent agent setup, PII regex patterns, retention policy code, webhook filtering, and compliance reporting, see [data patterns](references/data-patterns.md).

## Resources
- [Retell AI Privacy](https://www.retellai.com/privacy)
- [Retell Call Data](https://docs.retellai.com/api-references/get-call)

## Output
- Call recording consent configured in agent prompts
- PII redaction applied to all transcript storage paths
- Retention policies enforced with automated cleanup
- Compliance report generated for audit purposes
