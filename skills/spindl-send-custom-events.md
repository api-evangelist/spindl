---
name: Send server-side custom events to Spindl
description: Ingest custom attribution events server-side, identifying users by wallet address or customerUserId.
api: openapi/spindl-openapi.yml
operations: [sendCustomEvents]
---

# Send server-side custom events to Spindl

Track user actions for attribution by sending custom events to Spindl's events host.

## Auth
- Send the header `X-API-Key: <your_api_key>` (Server-to-Server key).
- Endpoint: `POST https://spindl.link/events/server` (note: the events host is `spindl.link`, not `api.spindl.xyz`).
- Set `Content-Type: application/json`.

## Steps
1. Build an **array** of event objects (send one or many). Each event needs:
   - `type`: use `"CUSTOM"`.
   - `data.name`: required, 3–100 chars, alphanumeric plus `_ : -` and spaces.
   - `data.properties`: optional JSON object (max 16KB; keys/values ≤ 1,000 chars).
   - `identity`: **either** `address` (a valid wallet address) **or** `customerUserId` (unique id or email). Supplying both improves identity stitching.
2. POST the array via `sendCustomEvents`. A success returns **204 No Content**.

## Rules
- No idempotency key exists — retrying a failed POST can duplicate events. Prefer batching into one array over per-event retries.
- Malformed payloads return `{ "statusCode", "message" }` with a 4xx status.
