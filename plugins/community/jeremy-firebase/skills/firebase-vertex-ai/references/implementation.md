# Firebase Vertex AI: Implementation Guide

## Firebase Project Structure

A production Firebase + Vertex AI project follows this layout:

```
project-root/
├── firebase.json                 # Service config, rewrites, emulators
├── .firebaserc                   # Project aliases (dev/staging/prod)
├── firestore.rules               # Security rules (version controlled)
├── firestore.indexes.json        # Composite indexes
├── storage.rules                 # Storage security rules
├── functions/
│   ├── package.json              # Dependencies (@google-cloud/vertexai, firebase-admin)
│   ├── tsconfig.json             # TypeScript strict mode, es2022 target
│   └── src/
│       ├── index.ts              # Barrel file: exports all functions
│       ├── config.ts             # Secret definitions, shared constants
│       ├── vertex/
│       │   ├── client.ts         # VertexAI singleton factory
│       │   ├── chat.ts           # Chat completion callable
│       │   ├── embeddings.ts     # Embedding generation trigger
│       │   └── moderation.ts     # Content moderation trigger
│       ├── middleware/
│       │   ├── auth.ts           # Auth token verification helpers
│       │   └── validation.ts     # Input validation with zod schemas
│       └── triggers/
│           ├── on-user-create.ts # Auth: create user profile doc
│           └── on-post-create.ts # Firestore: auto-generate embeddings
├── public/ or dist/              # Hosting static assets
├── .env.local                    # Local env vars (git-ignored)
└── .gitignore                    # Must include: .env*, serviceAccount*.json
```

## Cloud Functions + Vertex AI Pattern

### Singleton Client

Create one `VertexAI` instance per cold start, not per request:

```typescript
// functions/src/vertex/client.ts
import { VertexAI, GenerativeModel } from "@google-cloud/vertexai";

let _vertex: VertexAI | null = null;

export function getVertex(projectId: string): VertexAI {
  if (!_vertex) {
    _vertex = new VertexAI({ project: projectId, location: "us-central1" });
  }
  return _vertex;
}

export function getChatModel(projectId: string): GenerativeModel {
  return getVertex(projectId).getGenerativeModel({ model: "gemini-2.5-flash" });
}

export function getEmbeddingModel(projectId: string): GenerativeModel {
  return getVertex(projectId).getGenerativeModel({ model: "text-embedding-004" });
}
```

### Secret Management

Define secrets at module scope, reference in function options:

```typescript
// functions/src/config.ts
import { defineSecret } from "firebase-functions/params";

export const gcpProjectId = defineSecret("GCP_PROJECT_ID");

// Provision the secret before first deploy:
// firebase functions:secrets:set GCP_PROJECT_ID
```

```typescript
// functions/src/vertex/chat.ts
import { onCall } from "firebase-functions/v2/https";
import { gcpProjectId } from "../config";
import { getChatModel } from "./client";

export const chat = onCall(
  { secrets: [gcpProjectId], memory: "512MiB", timeoutSeconds: 120 },
  async (req) => {
    const model = getChatModel(gcpProjectId.value());
    // ... use model
  }
);
```

### Error Handling Pattern

Wrap Vertex AI calls with retry logic for transient failures:

```typescript
async function callWithRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  baseDelayMs = 1000
): Promise<T> {
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (err: any) {
      const retryable = ["INTERNAL", "DEADLINE_EXCEEDED", "RESOURCE_EXHAUSTED"];
      if (attempt === maxRetries || !retryable.includes(err.code)) {
        throw err;
      }
      const delay = baseDelayMs * Math.pow(2, attempt) + Math.random() * 500;
      await new Promise((r) => setTimeout(r, delay));
    }
  }
  throw new Error("Unreachable");
}
```

## Security Rules Design

### Principle: Deny by Default, Allowlist per Collection

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Deny everything not explicitly matched
    match /{document=**} {
      allow read, write: if false;
    }

    // Helper functions
    function isAuth() { return request.auth != null; }
    function isOwner(uid) { return isAuth() && request.auth.uid == uid; }
    function hasRole(role) {
      return isAuth() &&
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == role;
    }
    function isAdmin() { return hasRole('admin'); }

    // Users: owner read/write, admin read
    match /users/{userId} {
      allow read: if isOwner(userId) || isAdmin();
      allow create: if isOwner(userId);
      allow update: if isOwner(userId) &&
        !request.resource.data.diff(resource.data).affectedKeys().hasAny(['role', 'createdAt']);
      allow delete: if isAdmin();
    }

    // Posts: public read, owner write, admin delete
    match /posts/{postId} {
      allow read: if true;
      allow create: if isAuth() && request.resource.data.authorId == request.auth.uid;
      allow update: if isOwner(resource.data.authorId);
      allow delete: if isOwner(resource.data.authorId) || isAdmin();
    }

    // Embeddings: read by authenticated users, write by admin/functions only
    match /embeddings/{embeddingId} {
      allow read: if isAuth();
      allow write: if isAdmin();
    }

    // Conversations: private to owner
    match /users/{userId}/conversations/{convId} {
      allow read, write: if isOwner(userId);
    }
  }
}
```

### Field-Level Validation in Rules

Enforce document schema at the rules layer to catch malformed writes:

```javascript
match /posts/{postId} {
  allow create: if isAuth()
    && request.resource.data.keys().hasAll(['title', 'content', 'authorId'])
    && request.resource.data.title is string
    && request.resource.data.title.size() > 0
    && request.resource.data.title.size() <= 200
    && request.resource.data.content is string
    && request.resource.data.content.size() <= 50000
    && request.resource.data.authorId == request.auth.uid;
}
```

## Emulator Testing Strategy

### Configuration

```json
{
  "emulators": {
    "auth": { "port": 9099 },
    "functions": { "port": 5001 },
    "firestore": { "port": 8080 },
    "hosting": { "port": 5000 },
    "storage": { "port": 9199 },
    "ui": { "enabled": true, "port": 4000 }
  }
}
```

### Rules Unit Tests

Test security rules independently using `@firebase/rules-unit-testing`:

```typescript
// tests/firestore-rules.test.ts
import { initializeTestEnvironment, assertSucceeds, assertFails } from "@firebase/rules-unit-testing";
import { readFileSync } from "fs";

const testEnv = await initializeTestEnvironment({
  projectId: "demo-test",
  firestore: { rules: readFileSync("firestore.rules", "utf8") },
});

// Authenticated user can read their own profile
const alice = testEnv.authenticatedContext("alice");
await assertSucceeds(alice.firestore().collection("users").doc("alice").get());

// Unauthenticated user cannot read profiles
const unauth = testEnv.unauthenticatedContext();
await assertFails(unauth.firestore().collection("users").doc("alice").get());

// User cannot escalate their role
const aliceWithRole = testEnv.authenticatedContext("alice");
await assertFails(
  aliceWithRole.firestore().collection("users").doc("alice").update({ role: "admin" })
);
```

### Smoke Test Script

```bash
#!/bin/bash
set -euo pipefail

echo "Starting emulators..."
firebase emulators:start --only auth,functions,firestore &
EMU_PID=$!
sleep 5

echo "Testing function health endpoint..."
HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:5001/demo-project/us-central1/health)
if [ "$HTTP_CODE" != "200" ]; then
  echo "FAIL: health endpoint returned $HTTP_CODE"
  kill $EMU_PID; exit 1
fi

echo "Testing Firestore write via emulator..."
curl -s -X POST "http://localhost:8080/v1/projects/demo-project/databases/(default)/documents/test" \
  -H "Content-Type: application/json" \
  -d '{"fields":{"msg":{"stringValue":"smoke test"}}}' | grep -q "smoke test"

echo "All smoke tests passed"
kill $EMU_PID
```

## Deployment Pipeline

### Deploy Order

Deploy in dependency order to avoid broken rewrites or missing functions:

1. `firebase deploy --only firestore:rules` -- security rules first
2. `firebase deploy --only firestore:indexes` -- indexes (may take minutes)
3. `firebase deploy --only storage:rules` -- storage security
4. `firebase deploy --only functions` -- backend logic
5. `firebase deploy --only hosting` -- frontend (depends on function rewrites)

### CI/CD with GitHub Actions

```yaml
name: Deploy Firebase
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: "20" }
      - run: npm ci
      - run: cd functions && npm ci && npm run build
      - run: npm run build
      - run: npm test
      - uses: FirebaseExtended/action-hosting-deploy@v0
        with:
          repoToken: ${{ secrets.GITHUB_TOKEN }}
          firebaseServiceAccount: ${{ secrets.FIREBASE_SERVICE_ACCOUNT }}
          projectId: ${{ secrets.FIREBASE_PROJECT_ID }}
          channelId: live
```

## Environment Management

### .firebaserc

```json
{
  "projects": {
    "default": "myapp-dev",
    "staging": "myapp-staging",
    "prod": "myapp-prod"
  }
}
```

### Switching Environments

```bash
firebase use default          # dev
firebase use staging          # staging
firebase use prod             # production

# Deploy to specific environment
firebase deploy --only functions --project myapp-staging
```

### Per-Environment Secrets

Each Firebase project has its own Secret Manager. Set secrets per project:

```bash
firebase use staging
firebase functions:secrets:set GCP_PROJECT_ID    # enters staging project ID

firebase use prod
firebase functions:secrets:set GCP_PROJECT_ID    # enters prod project ID
```

### Environment-Specific Function Config

Use `defineString` for non-secret environment values:

```typescript
import { defineString } from "firebase-functions/params";

const environment = defineString("ENVIRONMENT", { default: "dev" });
const vertexRegion = defineString("VERTEX_REGION", { default: "us-central1" });
```

Set via `.env.<project>` files in `functions/`:
```
# functions/.env.myapp-staging
ENVIRONMENT=staging
VERTEX_REGION=us-central1
```
