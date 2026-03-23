---
name: navan-reference-architecture
description: |
  Use when designing a production Navan API integration architecture — API gateway, token management, data sync pipelines, ERP connectors, and monitoring stack.
  Trigger with "navan reference architecture" or "navan integration architecture".
allowed-tools: Read, Write, Edit, Bash(curl:*), Bash(jq:*), Grep
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
tags: [saas, navan, travel]
compatible-with: claude-code
---

# Navan Reference Architecture

## Overview

Production-grade architecture for Navan API integrations. Navan provides raw REST endpoints with OAuth 2.0 — no SDK, no webhooks, no sandbox. This architecture handles those constraints with five purpose-built layers.

## Prerequisites

- Navan API credentials from Admin > Travel admin > Settings > Integrations
- Cloud infrastructure (AWS, GCP, or Azure) for hosting integration services
- Data warehouse for BOOKING and TRANSACTION tables
- Understanding of OAuth 2.0 client credentials flow

## Instructions

### Architecture Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                        CONSUMERS                                 │
│   Travel Dashboard  │  Expense Reports  │  Finance System        │
└────────┬────────────┴────────┬──────────┴────────┬───────────────┘
         │                     │                    │
┌────────▼─────────────────────▼────────────────────▼───────────────┐
│  LAYER 1: API GATEWAY                                             │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────────────────┐  │
│  │ Rate Limiter │  │ Request Log  │  │ Circuit Breaker (5xx)   │  │
│  └─────────────┘  └──────────────┘  └─────────────────────────┘  │
└────────┬─────────────────────────────────────────────────────────┘
         │
┌────────▼─────────────────────────────────────────────────────────┐
│  LAYER 2: TOKEN MANAGEMENT SERVICE                                │
│  ┌──────────────────┐  ┌──────────────┐  ┌────────────────────┐  │
│  │ OAuth Client Cred │  │ Token Cache  │  │ Auto-Refresh       │  │
│  │ POST /authenticate│  │ (Redis/KMS)  │  │ (before expiry)    │  │
│  └──────────────────┘  └──────────────┘  └────────────────────┘  │
└────────┬─────────────────────────────────────────────────────────┘
         │
┌────────▼─────────────────────────────────────────────────────────┐
│  LAYER 3: NAVAN API CLIENT                                        │
│  ┌───────────────┐  ┌──────────────┐  ┌────────────────────────┐ │
│  │ /get_user_trips│  │ /get_users   │  │ /get_admin_trips      │ │
│  │ /get_invoices  │  │ /get_itin_pdf│  │ /reauthenticate       │ │
│  └───────────────┘  └──────────────┘  └────────────────────────┘ │
└────────┬────────────────────┬────────────────────────────────────┘
         │                    │
┌────────▼──────────┐ ┌──────▼─────────────────────────────────────┐
│  LAYER 4: DATA    │ │  LAYER 5: MONITORING                       │
│  SYNC PIPELINE    │ │  ┌──────────┐ ┌─────────┐ ┌────────────┐  │
│ ┌───────────────┐ │ │  │ API Call  │ │ Error   │ │ Token      │  │
│ │ Fivetran /    │ │ │  │ Metrics  │ │ Alerts  │ │ Expiry     │  │
│ │ Airbyte /     │ │ │  │ (volume, │ │ (PD/    │ │ Monitor    │  │
│ │ Estuary       │ │ │  │  latency)│ │  Slack) │ │            │  │
│ ├───────────────┤ │ │  └──────────┘ └─────────┘ └────────────┘  │
│ │ BOOKING table │ │ │                                             │
│ │ (weekly full) │ │ └─────────────────────────────────────────────┘
│ ├───────────────┤ │
│ │ TRANSACTION   │ │
│ │ (incremental) │ │
│ ├───────────────┤ │
│ │ ERP Connector │ │
│ │ (SAP/NetSuite)│ │
│ └───────────────┘ │
└───────────────────┘
```

### Layer 1 — API Gateway

The gateway provides rate limiting, request logging, and circuit breaking before any call reaches Navan.

```bash
# Example: test gateway → Navan connectivity
curl -s -w "connect: %{time_connect}s | ttfb: %{time_starttransfer}s | total: %{time_total}s\n" \
  -o /dev/null "https://app.navan.com/api/v1/authenticate"
```

**Key decisions:**
- **Rate limiter**: Token bucket at 80% of Navan's observed rate limit to provide buffer
- **Circuit breaker**: Open after 5 consecutive 5xx responses; half-open after 60 seconds
- **Request log**: Structured JSON with correlation ID, endpoint, response code, and latency

### Layer 2 — Token Management Service

Centralized OAuth lifecycle management. Navan uses `client_credentials` grant type via `POST /authenticate`.

```bash
# Token acquisition
TOKEN_RESPONSE=$(curl -s -X POST "https://app.navan.com/api/v1/authenticate" \
  -H "Content-Type: application/json" \
  -d "{\"client_id\":\"$NAVAN_CLIENT_ID\",\"client_secret\":\"$NAVAN_CLIENT_SECRET\"}")

TOKEN=$(echo "$TOKEN_RESPONSE" | jq -r '.token')
EXPIRES=$(echo "$TOKEN_RESPONSE" | jq -r '.expires_at // .expires_in')
echo "Token acquired, expires: $EXPIRES"

# Token refresh (use /reauthenticate for existing sessions)
curl -s -X POST "https://app.navan.com/api/v1/reauthenticate" \
  -H "Authorization: Bearer $TOKEN" | jq '.token'
```

**Design principles:**
- Cache tokens in Redis or KMS-encrypted storage — never in application memory across restarts
- Refresh tokens proactively 5 minutes before expiry
- Support multi-tenant scenarios with per-tenant credential isolation

### Layer 3 — Navan API Client

Thin wrapper around Navan's REST endpoints with consistent error handling:

| Endpoint | Method | Purpose | Data Table |
|----------|--------|---------|------------|
| `/authenticate` | POST | OAuth token acquisition | — |
| `/reauthenticate` | POST | Token refresh | — |
| `/get_users` | GET | Employee directory | — |
| `/get_user_trips` | GET | Traveler's bookings | BOOKING |
| `/get_admin_trips` | GET | All organization bookings | BOOKING |
| `/get_itineraries_pdf` | GET | Itinerary documents | — |
| `/get_invoices_poc` | GET | Invoice data | TRANSACTION |

### Layer 4 — Data Sync Pipeline

Navan has no push/webhook mechanism — all data sync is poll-based.

| Table | Sync Strategy | Frequency | Connector |
|-------|--------------|-----------|-----------|
| BOOKING | Full refresh | Weekly | Fivetran, Airbyte, or Estuary |
| TRANSACTION | Incremental (by date range) | Daily/hourly | Fivetran, Airbyte, or custom |

**Connector selection:**
- **Fivetran**: Managed, pre-built Navan connector, minimal configuration
- **Airbyte**: Open-source, self-hosted option, custom connector support
- **Estuary**: Real-time CDC where available, hybrid approach

### Layer 5 — Monitoring Stack

| Metric | Alert Threshold | Channel |
|--------|----------------|---------|
| API error rate | > 5% over 5 minutes | PagerDuty (P2) |
| Token refresh failure | Any failure | PagerDuty (P1) |
| API response latency | p95 > 5 seconds | Slack |
| Data sync staleness | BOOKING > 8 days old | Slack |
| Rate limit proximity | > 80% utilization | Slack |

## Output

- Architecture diagram adapted to your cloud provider and tooling
- Component specifications for each of the five layers
- Technology recommendations based on existing infrastructure
- Data flow documentation for BOOKING and TRANSACTION pipelines

## Error Handling

| Failure Mode | Architecture Response |
|-------------|---------------------|
| Token expired | Layer 2 auto-refreshes; Layer 1 retries transparently |
| Rate limited (429) | Layer 1 queues requests; Layer 5 alerts on sustained throttling |
| API outage (5xx) | Layer 1 circuit breaker opens; consumers get cached data |
| Data sync gap | Layer 4 runs catch-up sync; Layer 5 alerts on staleness |

## Examples

Validate the full stack end-to-end:

```bash
# End-to-end integration test
echo "1. Auth..." && \
TOKEN=$(curl -s -X POST "https://app.navan.com/api/v1/authenticate" \
  -H "Content-Type: application/json" \
  -d "{\"client_id\":\"$NAVAN_CLIENT_ID\",\"client_secret\":\"$NAVAN_CLIENT_SECRET\"}" \
  | jq -r '.token') && \
echo "2. Users..." && \
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://app.navan.com/api/v1/get_users" | jq '. | length' && \
echo "3. Trips..." && \
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://app.navan.com/api/v1/get_admin_trips" | jq '. | length'
```

## Resources

- [Navan Integrations](https://navan.com/integrations) — Connector catalog and partner ecosystem
- [Navan Security](https://navan.com/security) — Infrastructure details (AWS, TLS, AES/KMS)
- [Navan Help Center](https://app.navan.com/app/helpcenter) — API documentation and support

## Next Steps

- Use `navan-prod-checklist` to validate each layer before launch
- Use `navan-data-sync` for detailed data pipeline configuration
- Use `navan-observability` for monitoring stack implementation details
