---
name: Create and list Spindl tracking links
description: Create a Spindl short (redirect) tracking link for a campaign and list existing links for the organization.
api: openapi/spindl-openapi.yml
operations: [createLink, listLinks]
---

# Create and list Spindl tracking links

Use Spindl's Server-to-Server API to create campaign redirect links and enumerate existing ones.

## Auth
- Send the header `X-API-Key: <your_api_key>` on every request.
- The API key is the **Server-to-Server** key from Settings (https://app.spindl.xyz/settings), NOT the client-side SDK key.
- Base URL: `https://api.spindl.xyz/v1`.

## Steps
1. **Create a link** — `createLink` (`POST /links`). Body requires `name` (descriptive internal name, e.g. "Winter Campaign - Twitter Post 1") and `url` (the destination to redirect to). The response returns `link.link`, a `https://spindl.link/...` short URL to distribute.
2. **List links** — `listLinks` (`GET /links`) returns all links sorted by `createdAt` descending, each with `totalVisits` and `latestVisit` for reporting.

## Rules
- Errors return `{ "statusCode": <int>, "message": <string> }` — surface `message` to the user.
- No pagination: `listLinks` returns the full set. No idempotency key: do not blindly retry `createLink`, or you may create duplicate links.
