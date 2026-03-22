# Firestore Operations Manager: Examples

## Example 1: Fix a Missing Composite Index

A query fails with `FAILED_PRECONDITION: The query requires an index`. Diagnose and fix.

### The Failing Query

```typescript
// This query needs a composite index on (status ASC, createdAt DESC)
const snapshot = await db
  .collection("orders")
  .where("status", "==", "pending")
  .orderBy("createdAt", "desc")
  .limit(50)
  .get();
```

### Diagnosis

The error message includes a URL like:
```
https://console.firebase.google.com/v1/r/project/PROJECT_ID/firestore/indexes?create_composite=...
```

You can click this link to auto-create the index in Firebase Console, or add it manually.

### Fix: Add to firestore.indexes.json

```json
{
  "indexes": [
    {
      "collectionGroup": "orders",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "status", "order": "ASCENDING" },
        { "fieldPath": "createdAt", "order": "DESCENDING" }
      ]
    }
  ],
  "fieldOverrides": []
}
```

### Deploy

```bash
firebase deploy --only firestore:indexes

# Check index build status
firebase firestore:indexes
# Wait until status shows "READY" (can take minutes for large collections)
```

### Prevention

Before deploying new query code, always check if the combination of `where()` + `orderBy()` on different fields requires a composite index. Single-field queries and queries where the `orderBy` field matches the `where` equality field do not need composite indexes.

---

## Example 2: Batch Migrate 100k Documents with Checkpoints

Add a `status` field (default: `"active"`) to all documents in the `users` collection.

### Migration Script

```typescript
import * as admin from "firebase-admin";
admin.initializeApp();
const db = admin.firestore();

const MIGRATION_ID = "add-status-field-2026-03";
const BATCH_SIZE = 500;
const COLLECTION = "users";

interface Checkpoint {
  lastDocId: string;
  processed: number;
  skipped: number;
  status: string;
}

async function getCheckpoint(): Promise<Checkpoint | null> {
  const doc = await db.collection("_migrations").doc(MIGRATION_ID).get();
  return doc.exists ? (doc.data() as Checkpoint) : null;
}

async function saveCheckpoint(cp: Checkpoint): Promise<void> {
  await db.collection("_migrations").doc(MIGRATION_ID).set({
    ...cp,
    updatedAt: admin.firestore.FieldValue.serverTimestamp(),
  });
}

async function migrate() {
  let checkpoint = await getCheckpoint();
  let processed = checkpoint?.processed || 0;
  let skipped = checkpoint?.skipped || 0;
  let hasMore = true;

  console.log(checkpoint ? `Resuming from doc ${checkpoint.lastDocId}` : "Starting fresh");

  while (hasMore) {
    let query = db.collection(COLLECTION).orderBy("__name__").limit(BATCH_SIZE);

    if (checkpoint?.lastDocId) {
      const lastDoc = await db.collection(COLLECTION).doc(checkpoint.lastDocId).get();
      if (lastDoc.exists) {
        query = query.startAfter(lastDoc);
      }
    }

    const snapshot = await query.get();
    if (snapshot.empty) {
      hasMore = false;
      break;
    }

    const batch = db.batch();
    let batchCount = 0;

    for (const doc of snapshot.docs) {
      if (doc.data().status !== undefined) {
        skipped++;
        continue; // Already has status field -- idempotent
      }
      batch.update(doc.ref, { status: "active" });
      batchCount++;
    }

    if (batchCount > 0) {
      await batch.commit();
    }
    processed += batchCount;

    const lastDoc = snapshot.docs[snapshot.docs.length - 1];
    checkpoint = { lastDocId: lastDoc.id, processed, skipped, status: "in_progress" };
    await saveCheckpoint(checkpoint);

    console.log(`Batch done. Processed: ${processed}, Skipped: ${skipped}`);
  }

  await saveCheckpoint({ lastDocId: checkpoint?.lastDocId || "", processed, skipped, status: "completed" });
  console.log(`Migration complete. Processed: ${processed}, Skipped: ${skipped}`);
}

migrate().catch(console.error);
```

### Run

```bash
# Test against emulator first
export FIRESTORE_EMULATOR_HOST=localhost:8080
npx ts-node migrate-add-status.ts

# Then run against production
unset FIRESTORE_EMULATOR_HOST
GOOGLE_APPLICATION_CREDENTIALS=./serviceAccount.json npx ts-node migrate-add-status.ts
```

### Cost Estimate

```
100,000 document reads: 100,000 × $0.06/100k = $0.06
100,000 document writes: 100,000 × $0.18/100k = $0.18
Checkpoint writes: ~200 × $0.18/100k = ~$0.00
Total: ~$0.24
```

---

## Example 3: Design Security Rules for Multi-Tenant App

Each tenant has their own data. Users belong to one tenant. Admins manage their tenant.

### Data Model

```
tenants/{tenantId}                    # Tenant profile
tenants/{tenantId}/members/{userId}   # Membership with role
tenants/{tenantId}/projects/{projId}  # Tenant's projects
```

### Security Rules

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    function isAuth() { return request.auth != null; }

    function isMember(tenantId) {
      return isAuth() &&
        exists(/databases/$(database)/documents/tenants/$(tenantId)/members/$(request.auth.uid));
    }

    function getMemberRole(tenantId) {
      return get(/databases/$(database)/documents/tenants/$(tenantId)/members/$(request.auth.uid)).data.role;
    }

    function isTenantAdmin(tenantId) {
      return isMember(tenantId) && getMemberRole(tenantId) == 'admin';
    }

    // Tenant document: admins can update, members can read
    match /tenants/{tenantId} {
      allow read: if isMember(tenantId);
      allow update: if isTenantAdmin(tenantId);
      allow create, delete: if false; // Only via Admin SDK
    }

    // Members: admins manage, members read
    match /tenants/{tenantId}/members/{userId} {
      allow read: if isMember(tenantId);
      allow write: if isTenantAdmin(tenantId);
    }

    // Projects: members read, members create, admins delete
    match /tenants/{tenantId}/projects/{projId} {
      allow read: if isMember(tenantId);
      allow create: if isMember(tenantId) &&
        request.resource.data.createdBy == request.auth.uid;
      allow update: if isMember(tenantId) &&
        resource.data.createdBy == request.auth.uid;
      allow delete: if isTenantAdmin(tenantId);
    }
  }
}
```

### Emulator Test

```typescript
import { initializeTestEnvironment, assertSucceeds, assertFails } from "@firebase/rules-unit-testing";
import { readFileSync } from "fs";

const testEnv = await initializeTestEnvironment({
  projectId: "demo-test",
  firestore: { rules: readFileSync("firestore.rules", "utf8") },
});

// Setup: create tenant and member docs via admin context
const admin = testEnv.withSecurityRulesDisabled(async (ctx) => {
  const db = ctx.firestore();
  await db.doc("tenants/acme").set({ name: "Acme Corp" });
  await db.doc("tenants/acme/members/alice").set({ role: "admin" });
  await db.doc("tenants/acme/members/bob").set({ role: "member" });
});

// Alice (admin) can update tenant
const alice = testEnv.authenticatedContext("alice");
await assertSucceeds(alice.firestore().doc("tenants/acme").update({ name: "Acme Inc" }));

// Bob (member) cannot update tenant
const bob = testEnv.authenticatedContext("bob");
await assertFails(bob.firestore().doc("tenants/acme").update({ name: "Bob Corp" }));

// Bob can read projects
await assertSucceeds(bob.firestore().collection("tenants/acme/projects").get());

// Eve (not a member) cannot read anything
const eve = testEnv.authenticatedContext("eve");
await assertFails(eve.firestore().doc("tenants/acme").get());
```

---

## Example 4: Paginated Query with Cursor-Based Pagination

Fetch products sorted by price, 20 per page, with next/previous page support.

```typescript
import * as admin from "firebase-admin";
const db = admin.firestore();

interface PaginationCursor {
  lastPrice: number;
  lastDocId: string;
}

async function getProductPage(
  category: string,
  pageSize: number = 20,
  cursor?: PaginationCursor
) {
  let query = db
    .collection("products")
    .where("category", "==", category)
    .orderBy("price", "asc")
    .orderBy("__name__", "asc")  // Tiebreaker for stable pagination
    .limit(pageSize + 1);         // Fetch one extra to detect next page

  if (cursor) {
    query = query.startAfter(cursor.lastPrice, cursor.lastDocId);
  }

  const snapshot = await query.get();
  const hasNextPage = snapshot.docs.length > pageSize;
  const docs = snapshot.docs.slice(0, pageSize);

  const items = docs.map((doc) => ({ id: doc.id, ...doc.data() }));

  const nextCursor: PaginationCursor | null = hasNextPage
    ? { lastPrice: docs[docs.length - 1].data().price, lastDocId: docs[docs.length - 1].id }
    : null;

  return { items, nextCursor, hasNextPage };
}

// Usage
const page1 = await getProductPage("electronics");
console.log(`Page 1: ${page1.items.length} items, hasNext: ${page1.hasNextPage}`);

if (page1.nextCursor) {
  const page2 = await getProductPage("electronics", 20, page1.nextCursor);
  console.log(`Page 2: ${page2.items.length} items, hasNext: ${page2.hasNextPage}`);
}
```

### Required Composite Index

```json
{
  "collectionGroup": "products",
  "queryScope": "COLLECTION",
  "fields": [
    { "fieldPath": "category", "order": "ASCENDING" },
    { "fieldPath": "price", "order": "ASCENDING" }
  ]
}
```

Note: `__name__` is automatically included as a tiebreaker in Firestore indexes, so you do not need to add it explicitly to `firestore.indexes.json`.
