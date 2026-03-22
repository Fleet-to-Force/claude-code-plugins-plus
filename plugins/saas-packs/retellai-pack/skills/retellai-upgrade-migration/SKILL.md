---
name: retellai-upgrade-migration
description: |
  Analyze, plan, and execute Retell AI SDK upgrades with breaking change detection.
  Use when upgrading Retell AI SDK versions, detecting deprecations,
  or migrating to new API versions.
  Trigger with phrases like "upgrade retellai", "retellai migration",
  "retellai breaking changes", "update retellai SDK", "analyze retellai version".
allowed-tools: Read, Write, Edit, Bash(npm:*), Bash(git:*)
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
compatible-with: claude-code
tags: [retellai, voice-ai, saas]
---
# Retell AI Upgrade & Migration

## Current State
!`npm list 2>/dev/null | head -20`
!`pip freeze 2>/dev/null | head -20`

## Overview
Guide for upgrading Retell AI SDK versions and handling breaking changes safely. Covers version checking, changelog review, upgrade branch creation, breaking change remediation (import paths, configuration objects, method signatures), deprecation monitoring in development mode, and rollback procedures with pinned versions.

## Prerequisites
- Current Retell AI SDK installed
- Git for version control
- Test suite available
- Staging environment for validation

## Instructions

1. **Check the current version** with `npm list @retellai/sdk` and compare against the latest release with `npm view @retellai/sdk version`.
2. **Review the changelog** for breaking changes between the current and target versions. Pay attention to renamed imports, changed configuration objects, and removed features.
3. **Create an upgrade branch** with `git checkout -b upgrade/retellai-sdk-vX.Y.Z`, install the new version, and run the test suite to identify failures.
4. **Fix breaking changes** following the patterns in [upgrade guide](references/upgrade-guide.md): import path changes (v1 `Client` -> v2 `RetellAIClient`), configuration object changes (`key` -> `apiKey`), and method signature updates.
5. **Add deprecation monitoring** in development mode to catch warnings about soon-to-be-removed features proactively. See [upgrade guide](references/upgrade-guide.md) for the monitoring code.

## Output
- Updated SDK version installed and tested
- Breaking changes identified and fixed
- Test suite passing on the upgrade branch
- Rollback procedure documented with pinned version

## Error Handling
| SDK Version | API Version | Node.js | Breaking Changes |
|-------------|-------------|---------|------------------|
| 3.x | 2024-01 | 18+ | Major refactor |
| 2.x | 2023-06 | 16+ | Auth changes |
| 1.x | 2022-01 | 14+ | Initial release |

## Examples

For import changes, configuration changes, rollback procedure, and deprecation monitoring, see [upgrade guide](references/upgrade-guide.md).

## Resources
- [Retell AI Changelog](https://github.com/retellai/sdk/releases)
- [Retell AI Migration Guide](https://docs.retellai.com/migration)

## Next Steps
For CI integration during upgrades, see `retellai-ci-integration`. For production deployment after successful upgrade, see `retellai-prod-checklist`.
