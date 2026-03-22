# Firebase Vertex AI: Examples

## Example 1: Gemini-Backed Chat API on Firebase

A Cloud Function that accepts user messages, calls Gemini, and returns streamed responses. Includes auth verification and conversation history stored in Firestore.

### Function Code

```typescript
// functions/src/vertex/chat.ts
import { onCall, HttpsError } from "firebase-functions/v2/https";
import { defineSecret } from "firebase-functions/params";
import { VertexAI } from "@google-cloud/vertexai";
import * as admin from "firebase-admin";

const projectId = defineSecret("GCP_PROJECT_ID");
const db = admin.firestore();

export const chat = onCall(
  { secrets: [projectId], memory: "512MiB", timeoutSeconds: 120 },
  async (req) => {
    if (!req.auth) {
      throw new HttpsError("unauthenticated", "Sign in required");
    }

    const { message, conversationId } = req.data;
    if (!message || typeof message !== "string" || message.length > 4000) {
      throw new HttpsError("invalid-argument", "Message must be 1-4000 chars");
    }

    const vertex = new VertexAI({
      project: projectId.value(),
      location: "us-central1",
    });
    const model = vertex.getGenerativeModel({ model: "gemini-2.5-flash" });

    // Load conversation history from Firestore
    let history: { role: string; parts: { text: string }[] }[] = [];
    if (conversationId) {
      const convDoc = await db
        .collection("users")
        .doc(req.auth.uid)
        .collection("conversations")
        .doc(conversationId)
        .get();
      if (convDoc.exists) {
        history = convDoc.data()?.messages || [];
      }
    }

    // Call Gemini with history
    const chatSession = model.startChat({ history });
    const result = await chatSession.sendMessage(message);
    const responseText = result.response.candidates?.[0]?.content?.parts?.[0]?.text || "";

    // Persist updated conversation
    const convRef = conversationId
      ? db.collection("users").doc(req.auth.uid).collection("conversations").doc(conversationId)
      : db.collection("users").doc(req.auth.uid).collection("conversations").doc();

    const updatedMessages = [
      ...history,
      { role: "user", parts: [{ text: message }] },
      { role: "model", parts: [{ text: responseText }] },
    ];

    await convRef.set(
      { messages: updatedMessages, updatedAt: admin.firestore.FieldValue.serverTimestamp() },
      { merge: true }
    );

    return { response: responseText, conversationId: convRef.id };
  }
);
```

### Security Rules

```javascript
match /users/{userId}/conversations/{convId} {
  allow read, write: if request.auth != null && request.auth.uid == userId;
}
```

### Test Command

```bash
firebase emulators:start --only functions,firestore,auth
# In another terminal:
curl -X POST http://localhost:5001/PROJECT/us-central1/chat \
  -H "Content-Type: application/json" \
  -d '{"data":{"message":"Explain Firebase Auth in 2 sentences"}}'
```

---

## Example 2: Firestore-Powered RAG with Citations

Ingest documents, generate embeddings, and answer questions with source attribution.

### Ingestion Function

```typescript
// functions/src/vertex/ingest.ts
import { onDocumentCreated } from "firebase-functions/v2/firestore";
import { VertexAI } from "@google-cloud/vertexai";
import * as admin from "firebase-admin";

const db = admin.firestore();

export const onDocCreate = onDocumentCreated("documents/{docId}", async (event) => {
  const snap = event.data;
  if (!snap) return;

  const { title, content, source } = snap.data();
  const text = `${title}\n\n${content}`;

  const vertex = new VertexAI({ project: process.env.GCP_PROJECT_ID!, location: "us-central1" });
  const embedModel = vertex.getGenerativeModel({ model: "text-embedding-004" });
  const embedResult = await embedModel.embedContent({ content: { role: "user", parts: [{ text }] } });

  await db.collection("embeddings").doc(event.params.docId).set({
    docId: event.params.docId,
    title,
    source,
    vector: embedResult.embedding.values,
    createdAt: admin.firestore.FieldValue.serverTimestamp(),
  });
});
```

### Query Function with Citations

```typescript
// functions/src/vertex/rag-query.ts
import { onCall, HttpsError } from "firebase-functions/v2/https";
import { VertexAI } from "@google-cloud/vertexai";
import * as admin from "firebase-admin";

const db = admin.firestore();

export const ragQuery = onCall({ memory: "1GiB", timeoutSeconds: 180 }, async (req) => {
  if (!req.auth) throw new HttpsError("unauthenticated", "Sign in required");

  const { query } = req.data;
  const vertex = new VertexAI({ project: process.env.GCP_PROJECT_ID!, location: "us-central1" });

  // Generate query embedding
  const embedModel = vertex.getGenerativeModel({ model: "text-embedding-004" });
  const queryEmbed = await embedModel.embedContent({
    content: { role: "user", parts: [{ text: query }] },
  });

  // Vector search in Firestore
  const results = await db
    .collection("embeddings")
    .findNearest("vector", queryEmbed.embedding.values, {
      limit: 5,
      distanceMeasure: "COSINE",
    })
    .get();

  // Fetch full documents
  const sources = await Promise.all(
    results.docs.map(async (emb) => {
      const doc = await db.collection("documents").doc(emb.data().docId).get();
      return { id: emb.data().docId, title: emb.data().title, content: doc.data()?.content || "" };
    })
  );

  // Generate answer with citations
  const genModel = vertex.getGenerativeModel({ model: "gemini-2.5-flash" });
  const contextBlock = sources.map((s, i) => `[${i + 1}] ${s.title}: ${s.content}`).join("\n\n");
  const prompt = `Answer based on the sources below. Cite sources as [1], [2], etc.\n\nSources:\n${contextBlock}\n\nQuestion: ${query}`;

  const result = await genModel.generateContent(prompt);
  const answer = result.response.candidates?.[0]?.content?.parts?.[0]?.text || "";

  return { answer, sources: sources.map((s, i) => ({ citation: i + 1, title: s.title, id: s.id })) };
});
```

---

## Example 3: Content Moderation with Gemini in Firestore Trigger

Automatically moderate user-generated content when written to Firestore.

```typescript
// functions/src/vertex/moderation.ts
import { onDocumentCreated } from "firebase-functions/v2/firestore";
import { VertexAI } from "@google-cloud/vertexai";

export const moderatePost = onDocumentCreated("posts/{postId}", async (event) => {
  const snap = event.data;
  if (!snap) return;

  const { content, authorId } = snap.data();

  const vertex = new VertexAI({ project: process.env.GCP_PROJECT_ID!, location: "us-central1" });
  const model = vertex.getGenerativeModel({ model: "gemini-2.5-flash" });

  const result = await model.generateContent({
    contents: [{ role: "user", parts: [{ text: `
      Evaluate this user-generated content for safety. Return JSON only.
      {"safe": boolean, "categories": string[], "confidence": number, "reason": string}

      Content: ${content}
    ` }] }],
    generationConfig: { responseMimeType: "application/json" },
  });

  const moderation = JSON.parse(result.response.candidates?.[0]?.content?.parts?.[0]?.text || "{}");

  await snap.ref.update({
    moderation: {
      safe: moderation.safe ?? false,
      categories: moderation.categories || [],
      confidence: moderation.confidence || 0,
      reviewedAt: new Date().toISOString(),
    },
    visible: moderation.safe === true,
  });

  // Flag unsafe content for human review
  if (!moderation.safe) {
    const db = await import("firebase-admin").then((a) => a.firestore());
    await db.collection("moderation_queue").add({
      postId: event.params.postId,
      authorId,
      reason: moderation.reason,
      flaggedAt: new Date().toISOString(),
    });
  }
});
```

---

## Example 4: Full-Stack Deploy with Auth + Functions + Hosting

Complete deployment script with environment targeting and smoke tests.

```bash
#!/bin/bash
set -euo pipefail

ENV="${1:-dev}"
echo "Deploying to: $ENV"

# Select project
firebase use "$ENV"

# Build functions
echo "Building Cloud Functions..."
cd functions && npm ci && npm run build && cd ..

# Build frontend
echo "Building frontend..."
npm run build

# Deploy security rules first (safest order)
echo "Deploying security rules..."
firebase deploy --only firestore:rules,storage:rules

# Deploy indexes
echo "Deploying Firestore indexes..."
firebase deploy --only firestore:indexes

# Deploy functions
echo "Deploying Cloud Functions..."
firebase deploy --only functions

# Deploy hosting last (depends on function rewrites)
echo "Deploying Hosting..."
firebase deploy --only hosting

# Smoke tests
echo "Running smoke tests..."
PROJECT_ID=$(firebase use | grep -oP 'Active Project: \K\S+' || echo "unknown")
BASE_URL="https://${PROJECT_ID}.web.app"

# Test hosting responds
HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" "$BASE_URL")
if [ "$HTTP_CODE" != "200" ]; then
  echo "FAIL: Hosting returned $HTTP_CODE"
  exit 1
fi
echo "PASS: Hosting returns 200"

# Test function endpoint
API_CODE=$(curl -s -o /dev/null -w "%{http_code}" "$BASE_URL/api/health")
if [ "$API_CODE" != "200" ]; then
  echo "WARN: /api/health returned $API_CODE (function may still be cold-starting)"
fi

echo "Deployment complete: $BASE_URL"
```

### firebase.json for this setup

```json
{
  "hosting": {
    "public": "dist",
    "ignore": ["firebase.json", "**/.*", "**/node_modules/**"],
    "rewrites": [
      { "source": "/api/**", "function": "api" },
      { "source": "**", "destination": "/index.html" }
    ]
  },
  "functions": [
    { "source": "functions", "codebase": "default", "runtime": "nodejs20" }
  ],
  "firestore": { "rules": "firestore.rules", "indexes": "firestore.indexes.json" },
  "storage": { "rules": "storage.rules" },
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
