---
name: Serve a Spindl ad unit and record the impression
description: Fetch an onchain ad recommendation for a wallet at a publisher placement, render it, and post the impression and click back to Spindl.
api: openapi/spindl-ads-api-openapi.yml
operations: [fetchRecommendations, trackAdEvent, serveAdIframe]
---

# Serve a Spindl ad unit and record the impression

Use Spindl's Ads (Embed) API to fetch a targeted ad unit for a wallet address and report back when it is shown or clicked.

## Prerequisites
- Your account must be **approved as a Publisher** by the Spindl team — this is a human step, not self-serve.
- You need a `publisher_id` (issued by the Spindl team) and at least one `placement_id` (from the Placements tab).
- You need a **Publisher API Token** from the Settings screen. This is a secret: never ship it in browser or mobile client code.

## Auth
- Send `X-API-ACCESS-KEY: <publisher api token>` on every request.
- This is **not** the same header as the Server-to-Server API. That surface uses `X-API-Key`. Do not reuse a key across surfaces.
- Base URL: `https://e.spindlembed.com/v1`.

## Steps
1. **Fetch a recommendation** — `fetchRecommendations` (`GET /render/{publisher_id}`). Required query params: `placement_id`, `address` (the `0x…` wallet), `limit` (usually `1`), and `country` (2-letter code — behind Cloudflare you can pass `CF-IPCountry` straight through). Optionally pass `chain_id` for the chain the unit renders on.
2. **Render the unit.** Each item in `items[]` carries `type` (`card`, `iframe`, `discord`), `title`, `description`, `imageUrl`, `imageAltText`, an optional `context.text` explaining why it was returned, and `ctas[]` of `{title, href}`. An empty `items[]` means no ad is available — render nothing.
3. **Post the impression** — `trackAdEvent` (`POST /external/track`) with body `{"type": "impression", "impression_id": "<impressionId from step 1>"}`, every time a unit is actually shown on screen.
4. **Post the click** — the same `trackAdEvent` call with `"type": "click"` and the same `impression_id`.

## Alternative: no API calls
- React: drop in `BannerEmbed` from `@spindl-xyz/embed-react` with `publisherId` and `placementId` (optionally `address` and a `properties` JSON blob). It renders an iframe and handles tracking for you.
- Plain HTML: use `serveAdIframe` as an iframe `src` — `https://e.spindlembed.com/v1/serve?publisher_id=…&placement_id=…&address=…`. This path is unauthenticated by design; no publisher token goes to the browser.

## Rules
- Errors on this surface use the **grpc-gateway** envelope `{"code": <int>, "message": <string>, "details": []}`. `code` is a **gRPC** status code, not an HTTP status — a `401` carries `code: 16` (UNAUTHENTICATED) and an unknown route carries `code: 5` (NOT_FOUND). Do not surface `code` to a user as an HTTP status.
- A missing key returns HTTP 401 with `www-authenticate: API key is required`.
- There is **no idempotency key**. Posting the same `impression_id` twice is deduplicated by Spindl only insofar as the id is server-issued; do not retry blindly in a loop.
- No rate limits are published and no `RateLimit-*`/`Retry-After` headers are returned. Back off conservatively on 5xx.
- `limit` is a required parameter, not an optional one; omitting it is a client error.
