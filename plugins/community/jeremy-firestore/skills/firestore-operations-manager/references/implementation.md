# Firestore Operations Manager: Implementation Guide

## Firestore Data Model Patterns

### Subcollections vs Root Collections

| Pattern | When to Use | Trade-off |
|---------|-------------|-----------|
| Root collection (`/orders`) | Need to query across all documents regardless of parent | Requires storing parent IDs as fields; no automatic scoping |
| Subcollection (`/users/{uid}/orders`) | Data is naturally owned by a parent; security rules scope by parent | Cannot query across all users' orders without collection group query |
| Collection group query | Need cross-parent queries on subcollections | Must create collection group index; applies same security rules to all subcollections with that name |

### Denormalization Strategy

Firestore has no joins. Duplicate data where read patterns require it:

```typescript
// When creating an order, store the user name directly
await db.collection("orders").add({
  userId: "alice123",
  userName: "Alice Smith",     // Denormalized from users/alice123
  items: [...],
  total: 49.99,
  createdAt: admin.firestore.FieldValue.serverTimestamp(),
});
```

Update denormalized copies when the source changes:

```typescript
// When user updates their name, update all their orders
async function updateUserName(userId: string, newName: string) {
  const orders = await db.collection("orders")
    .where("userId", "==", userId)
    .get();

  const batches: admin.firestore.WriteBatch[] = [];
  let batch = db.batch();
  let count = 0;

  orders.forEach((doc) => {
    batch.update(doc.ref, { userName: newName });
    count++;
    if (count % 500 === 0) {
      batches.push(batch);
      batch = db.batch();
    }
  });
  batches.push(batch);

  for (const b of batches) {
    await b.commit();
  }
}
```

## Batch Write Mechanics

### The 500-Operation Limit

Each `WriteBatch.commit()` accepts a maximum of 500 operations. Each `set()`, `update()`, or `delete()` call counts as one operation. Exceeding 500 throws `INVALID_ARGUMENT`.

### Chunked Batch Writer

```typescript
async function batchWrite<T>(
  items: T[],
  writeFn: (batch: admin.firestore.WriteBatch, item: T) => void,
  options: { batchSize?: number; onProgress?: (done: number, total: number) => void } = {}
) {
  const { batchSize = 500, onProgress } = options;
  let processed = 0;

  for (let i = 0; i < items.length; i += batchSize) {
    const batch = db.batch();
    const chunk = items.slice(i, i + batchSize);

    for (const item of chunk) {
      writeFn(batch, item);
    }

    await batch.commit();
    processed += chunk.length;
    onProgress?.(processed, items.length);
  }

  return { processed };
}

// Usage: update all documents
const docs = await db.collection("users").get();
await batchWrite(
  docs.docs,
  (batch, doc) => batch.update(doc.ref, { migrated: true }),
  { onProgress: (done, total) => console.log(`${done}/${total}`) }
);
```

### Retry Logic for Batch Commits

```typescript
async function commitWithRetry(
  batch: admin.firestore.WriteBatch,
  maxRetries = 3
): Promise<void> {
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      await batch.commit();
      return;
    } catch (err: any) {
      if (attempt === maxRetries) throw err;
      if (err.code === 10 /* ABORTED */ || err.code === 14 /* UNAVAILABLE */) {
        const delay = Math.pow(2, attempt) * 1000 + Math.random() * 500;
        await new Promise((r) => setTimeout(r, delay));
        continue;
      }
      throw err; // Non-retryable error
    }
  }
}
```

## Composite Index Design

### When Indexes Are Required

| Query Pattern | Index Needed? |
|---------------|---------------|
| Single `where('field', '==', value)` | No (auto-indexed) |
| Single `where('field', '>', value)` | No (auto-indexed) |
| `where('a', '==', v1).where('b', '==', v2)` | Yes (composite) |
| `where('a', '==', v).orderBy('b')` | Yes (composite, unless a == b) |
| `where('a', '>', v).orderBy('a')` | No (same field) |
| `where('a', '>', v).orderBy('b')` | Yes (composite; `a` must be first `orderBy`) |
| `where('a', 'array-contains', v).orderBy('b')` | Yes (composite) |

### Index File Format

```json
{
  "indexes": [
    {
      "collectionGroup": "products",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "category", "order": "ASCENDING" },
        { "fieldPath": "price", "order": "ASCENDING" }
      ]
    },
    {
      "collectionGroup": "orders",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "userId", "order": "ASCENDING" },
        { "fieldPath": "createdAt", "order": "DESCENDING" }
      ]
    }
  ],
  "fieldOverrides": []
}
```

### Deploy and Monitor

```bash
# Deploy indexes
firebase deploy --only firestore:indexes

# Check status (indexes build asynchronously)
firebase firestore:indexes
# Output shows CREATING → READY
```

Large collections (> 1M documents) can take hours to index. Deploy indexes well before deploying query code.

## Security Rules Testing Workflow

### Setup

```bash
npm install -D @firebase/rules-unit-testing firebase-admin
```

### Test File Structure

```typescript
// tests/firestore.rules.test.ts
import {
  initializeTestEnvironment,
  assertSucceeds,
  assertFails,
  RulesTestEnvironment,
} from "@firebase/rules-unit-testing";
import { readFileSync } from "fs";

let testEnv: RulesTestEnvironment;

beforeAll(async () => {
  testEnv = await initializeTestEnvironment({
    projectId: "demo-test",
    firestore: { rules: readFileSync("firestore.rules", "utf8") },
  });
});

afterAll(() => testEnv.cleanup());
afterEach(() => testEnv.clearFirestore());

describe("users collection", () => {
  test("authenticated user can read own profile", async () => {
    // Seed data as admin
    await testEnv.withSecurityRulesDisabled(async (ctx) => {
      await ctx.firestore().doc("users/alice").set({ name: "Alice", role: "user" });
    });

    const alice = testEnv.authenticatedContext("alice");
    await assertSucceeds(alice.firestore().doc("users/alice").get());
  });

  test("user cannot read another user profile", async () => {
    await testEnv.withSecurityRulesDisabled(async (ctx) => {
      await ctx.firestore().doc("users/bob").set({ name: "Bob", role: "user" });
    });

    const alice = testEnv.authenticatedContext("alice");
    await assertFails(alice.firestore().doc("users/bob").get());
  });

  test("unauthenticated user cannot read any profile", async () => {
    const unauth = testEnv.unauthenticatedContext();
    await assertFails(unauth.firestore().doc("users/alice").get());
  });
});
```

### Run Tests

```bash
# Start emulator and run tests
firebase emulators:exec --only firestore "npx jest tests/firestore.rules.test.ts"

# Or with emulator already running
FIRESTORE_EMULATOR_HOST=localhost:8080 npx jest tests/firestore.rules.test.ts
```

## Migration Strategies

### Strategy 1: Backfill (Add New Field)

Add a field to existing documents without modifying existing data.

```typescript
// Query: all docs missing the new field
let query = db.collection("users")
  .where("status", "==", null)   // Won't work -- Firestore can't query missing fields
  .limit(500);

// Correct approach: query all, skip docs that already have the field
let query = db.collection("users")
  .orderBy("__name__")
  .limit(500);

// In the batch loop:
for (const doc of snapshot.docs) {
  if (doc.data().status !== undefined) {
    skipped++;
    continue;  // Idempotent: skip already-migrated
  }
  batch.update(doc.ref, { status: "active" });
}
```

### Strategy 2: Transform (Modify Existing Field)

Change field values (e.g., rename enum values, change types).

```typescript
// Transform: change "type" from string to standardized enum
const TRANSFORMS: Record<string, string> = {
  "free": "free_tier",
  "Free": "free_tier",
  "premium": "premium_tier",
  "Premium": "premium_tier",
  "enterprise": "enterprise_tier",
};

for (const doc of snapshot.docs) {
  const currentType = doc.data().type;
  const newType = TRANSFORMS[currentType];
  if (!newType || currentType === newType) {
    skipped++;
    continue;
  }
  batch.update(doc.ref, { type: newType });
}
```

### Strategy 3: Collection Move

Move documents from one collection to another (e.g., restructuring data model).

```typescript
async function moveCollection(source: string, target: string) {
  let lastDoc: admin.firestore.DocumentSnapshot | null = null;
  let moved = 0;

  while (true) {
    let query = db.collection(source).orderBy("__name__").limit(500);
    if (lastDoc) query = query.startAfter(lastDoc);

    const snapshot = await query.get();
    if (snapshot.empty) break;

    const batch = db.batch();
    for (const doc of snapshot.docs) {
      batch.set(db.collection(target).doc(doc.id), doc.data());
      batch.delete(doc.ref);  // Delete from source
    }
    await batch.commit();

    lastDoc = snapshot.docs[snapshot.docs.length - 1];
    moved += snapshot.docs.length;
    console.log(`Moved ${moved} documents`);
  }
}
```

**Warning**: This is not atomic across batches. If the script fails midway, some documents exist in both collections. Run a reconciliation query afterward.

## Emulator Setup

### firebase.json Emulator Config

```json
{
  "emulators": {
    "firestore": { "port": 8080 },
    "auth": { "port": 9099 },
    "ui": { "enabled": true, "port": 4000 }
  }
}
```

### Connect Admin SDK to Emulator

```typescript
// Set before initializing admin SDK
process.env.FIRESTORE_EMULATOR_HOST = "localhost:8080";
process.env.FIREBASE_AUTH_EMULATOR_HOST = "localhost:9099";

import * as admin from "firebase-admin";
admin.initializeApp({ projectId: "demo-test" });
```

### Seed Emulator with Test Data

```bash
# Export from production (careful with PII)
gcloud firestore export gs://my-bucket/export-2026-03

# Import into emulator
firebase emulators:start --import=./emulator-seed-data

# Or seed programmatically in test setup
```

### Persist Emulator Data Between Restarts

```bash
firebase emulators:start \
  --export-on-exit=./emulator-data \
  --import=./emulator-data
```

This writes Firestore data to `./emulator-data/` on shutdown and reloads it on next start. Add `emulator-data/` to `.gitignore`.
