---
name: retellai-local-dev-loop
description: |
  Configure Retell AI local development with hot reload and testing.
  Use when setting up a development environment, configuring test workflows,
  or establishing a fast iteration cycle with Retell AI.
  Trigger with phrases like "retellai dev setup", "retellai local development",
  "retellai dev environment", "develop with retellai".
allowed-tools: Read, Write, Edit, Bash(npm:*), Bash(pnpm:*), Grep
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
compatible-with: claude-code
tags: [retellai, voice-ai, saas]
---
# Retell AI Local Dev Loop

## Overview
Set up a fast, reproducible local development workflow for Retell AI integrations. Covers project scaffolding with a `retellai/` module for client wrapper, configuration, and utilities, environment variable management with `.env.local` templates, hot reload using `tsx watch`, vitest test configuration with SDK mocking, and debug mode for verbose logging.

## Prerequisites
- Completed `retellai-install-auth` setup
- Node.js 18+ with npm/pnpm
- Code editor with TypeScript support
- Git for version control

## Instructions

1. **Create the project structure** with `src/retellai/` for client wrapper, config, and utilities. Add a `tests/` directory and `.env.local` for secrets. See [dev setup](references/dev-setup.md) for the full directory layout.
2. **Configure the environment** by copying `.env.example` to `.env.local`, installing dependencies, and starting the dev server.
3. **Set up hot reload** using `tsx watch` for the dev server and `vitest --watch` for tests. This gives sub-second feedback on code changes.
4. **Configure testing** with vitest and SDK mocking. Use `vi.mock()` to replace the Retell AI SDK with controlled responses in unit tests. See [dev setup](references/dev-setup.md) for mock examples.
5. **Use debug mode** with `DEBUG=RETELLAI=* npm run dev` for verbose logging of all SDK operations during development.

## Output
- Working development environment with hot reload on file changes
- Configured test suite with SDK mocking for offline testing
- Environment variable management with `.env.local` template
- Fast iteration cycle from code change to test result

## Error Handling
| Error | Cause | Solution |
|-------|-------|----------|
| Module not found | Missing dependency | Run `npm install` |
| Port in use | Another process on same port | Kill process or change port in config |
| Env not loaded | Missing .env.local | Copy from .env.example |
| Test timeout | Slow network or unmocked call | Increase test timeout or add mock |

## Examples

For project structure, environment setup, hot reload config, vitest setup, SDK mocking, and debug mode, see [dev setup](references/dev-setup.md).

## Resources
- [Retell AI SDK Reference](https://docs.retellai.com/sdk)
- [Vitest Documentation](https://vitest.dev/)
- [tsx Documentation](https://github.com/esbuild-kit/tsx)

## Next Steps
For production-ready code patterns, see `retellai-sdk-patterns`. For CI pipeline setup, see `retellai-ci-integration`.
