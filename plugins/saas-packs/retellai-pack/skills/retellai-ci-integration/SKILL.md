---
name: retellai-ci-integration
description: |
  Configure Retell AI CI/CD integration with GitHub Actions and testing.
  Use when setting up automated testing, configuring CI pipelines,
  or integrating Retell AI tests into your build process.
  Trigger with phrases like "retellai CI", "retellai GitHub Actions",
  "retellai automated tests", "CI retellai".
allowed-tools: Read, Write, Edit, Bash(gh:*)
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
compatible-with: claude-code
tags: [retellai, voice-ai, saas]
---
# Retell AI CI Integration

## Overview
Set up CI/CD pipelines for Retell AI integrations with automated testing, coverage reporting, and release workflows. Covers GitHub Actions configuration for PR and push triggers, secret management for test API keys, conditional integration tests that skip when credentials are unavailable, and branch protection rules requiring Retell AI tests to pass.

## Prerequisites
- GitHub repository with Actions enabled
- Retell AI test API key
- npm/pnpm project configured

## Instructions

1. **Create a GitHub Actions workflow** that runs on push to main and pull requests. Configure Node.js 20, npm caching, unit tests with coverage, and integration tests. See [CI configs](references/ci-configs.md) for the complete YAML.
2. **Configure secrets** using `gh secret set RETELLAI_API_KEY` to store the test API key securely. Use separate secrets for staging and production environments.
3. **Add integration tests** that conditionally skip when `RETELLAI_API_KEY` is not set. This allows contributors without API keys to run unit tests while CI runs the full suite.
4. **Set up branch protection** requiring the `test` and `retellai-integration` status checks to pass before merging.
5. **Configure a release workflow** triggered by version tags (`v*`) that runs integration tests against production before publishing.

## Output
- Automated test pipeline running on every PR
- PR checks configured with branch protection
- Coverage reports uploaded for review
- Release workflow validating production readiness

## Error Handling
| Issue | Cause | Solution |
|-------|-------|----------|
| Secret not found | Missing configuration | Add secret via `gh secret set` |
| Tests timeout | Network issues or slow API | Increase timeout or add mock fallback |
| Auth failures in CI | Invalid or expired key | Rotate secret value in repository settings |

## Examples

For GitHub Actions workflows, integration test patterns, release workflow, and branch protection config, see [CI configs](references/ci-configs.md).

## Resources
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Retell AI CI Guide](https://docs.retellai.com/ci)

## Next Steps
For deployment patterns after CI passes, see `retellai-deploy-integration`. For production readiness verification, see `retellai-prod-checklist`.
