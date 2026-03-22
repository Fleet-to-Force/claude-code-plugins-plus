---
name: retellai-migration-deep-dive
description: |
  Execute Retell AI major re-architecture and migration strategies with strangler fig pattern.
  Use when migrating to or from Retell AI, performing major version upgrades,
  or re-platforming existing integrations to Retell AI.
  Trigger with phrases like "migrate retellai", "retellai migration",
  "switch to retellai", "retellai replatform", "retellai upgrade major".
allowed-tools: Read, Write, Edit, Bash(npm:*), Bash(node:*), Bash(kubectl:*)
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
compatible-with: claude-code
tags: [retellai, voice-ai, saas]
---
# Retell AI Migration Deep Dive

## Current State
!`npm list 2>/dev/null | head -20`
!`pip freeze 2>/dev/null | head -20`

## Overview
Comprehensive guide for migrating to or from Retell AI, or performing major version upgrades. Covers the strangler fig pattern for gradual traffic shifting, adapter layer abstraction for safe parallel runs, batch data migration with error recovery, and feature-flag-controlled rollout with rollback procedures.

## Prerequisites
- Current system documentation
- Retell AI SDK installed
- Feature flag infrastructure
- Rollback strategy tested

## Migration Types

| Type | Complexity | Duration | Risk |
|------|-----------|----------|------|
| Fresh install | Low | Days | Low |
| From competitor | Medium | Weeks | Medium |
| Major version | Medium | Weeks | Medium |
| Full replatform | High | Months | High |

## Instructions

### Step 1: Assess Current Configuration
Document existing implementation and data inventory. Identify all integration points, data types, record counts, and customizations. See [migration implementation](references/migration-implementation.md) for the assessment script and data inventory interface.

### Step 2: Build Adapter Layer
Create an abstraction layer that implements a common `ServiceAdapter` interface for both the old system and Retell AI. This enables parallel runs and gradual traffic shifting without changing application code. Full adapter pattern in [migration implementation](references/migration-implementation.md).

### Step 3: Migrate Data
Run batch data migration with error handling. Process records in batches of 100, log progress, and collect errors for retry. The migration function with error recovery is in [migration implementation](references/migration-implementation.md).

### Step 4: Shift Traffic
Use feature flags to gradually route traffic from the old system to Retell AI. Start at 10%, monitor error rates, then increase to 50% and finally 100%. Rollback by setting the flag to 0%.

## Output
- Migration assessment complete with integration point inventory
- Adapter layer implemented for parallel operation
- Data migrated successfully with error report
- Traffic fully shifted to Retell AI with verified rollback path

## Error Handling
| Issue | Cause | Solution |
|-------|-------|----------|
| Data mismatch | Transform errors | Validate transform logic with sample data first |
| Performance drop | No caching | Add caching layer to adapter |
| Rollback triggered | Errors spiked | Reduce traffic percentage, investigate root cause |
| Validation failed | Missing data | Check batch processing logs for skipped records |

## Examples

### Quick Migration Status
```typescript
const status = await validateRetellAIMigration();
console.log(`Migration ${status.passed ? 'PASSED' : 'FAILED'}`);
status.checks.forEach(c => console.log(`  ${c.name}: ${c.result.success}`));
```

For strangler fig diagrams, phase-by-phase implementation, rollback scripts, and validation code, see [migration implementation](references/migration-implementation.md).

## Resources
- [Strangler Fig Pattern](https://martinfowler.com/bliki/StranglerFigApplication.html)
- [Retell AI Migration Guide](https://docs.retellai.com/migration)

## Next Steps
For advanced troubleshooting during migration issues, see `retellai-advanced-troubleshooting`. For CI pipeline setup to validate migration changes, see `retellai-ci-integration`.
