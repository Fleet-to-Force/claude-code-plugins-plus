---
name: retellai-sdk-patterns
description: |
  Apply production-ready Retell AI SDK patterns for TypeScript and Python.
  Use when implementing Retell AI integrations, refactoring SDK usage,
  or establishing team coding standards for Retell AI.
  Trigger with phrases like "retellai SDK patterns", "retellai best practices",
  "retellai code patterns", "idiomatic retellai".
allowed-tools: Read, Write, Edit
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
compatible-with: claude-code
tags: [retellai, voice-ai, saas]
---
# Retell AI SDK Patterns

## Overview
Production-ready patterns for Retell AI SDK usage in TypeScript and Python. Covers client lifecycle management (singleton and factory patterns), structured error handling with typed exceptions, automatic retry with exponential backoff, multi-tenant client isolation, and runtime response validation using Zod schemas.

## Prerequisites
- Completed `retellai-install-auth` setup
- Familiarity with async/await patterns
- Understanding of error handling best practices

## Instructions

1. **Create a singleton client** to avoid redundant connections and ensure consistent configuration. Use lazy initialization with a module-level variable. See [code patterns](references/code-patterns.md).
2. **Add error handling wrappers** that catch `RetellAIError` specifically, extract error codes for structured logging, and return result tuples instead of throwing.
3. **Configure retry logic** with exponential backoff starting at 1 second and capping at 32 seconds. Only retry on 429 and 5xx status codes.
4. **Validate API responses** at runtime using Zod schemas to catch breaking changes early. This is especially important during SDK upgrades.

## Output
- Type-safe client singleton with lazy initialization
- Robust error handling with structured logging
- Automatic retry with exponential backoff for transient failures
- Runtime validation for API responses using Zod

## Error Handling
| Pattern | Use Case | Benefit |
|---------|----------|---------|
| Safe wrapper | All API calls | Prevents uncaught exceptions |
| Retry logic | Transient failures | Improves reliability |
| Type guards | Response validation | Catches API changes early |
| Logging | All operations | Debugging and monitoring |

## Examples

For singleton, factory, retry, Python context manager, and Zod validation patterns, see [code patterns](references/code-patterns.md).

## Resources
- [Retell AI SDK Reference](https://docs.retellai.com/sdk)
- [Retell AI API Types](https://docs.retellai.com/types)
- [Zod Documentation](https://zod.dev/)

## Next Steps
Apply these patterns in `retellai-core-workflow-a` for real-world usage. For multi-tenant client isolation, see the factory pattern in [code patterns](references/code-patterns.md).
