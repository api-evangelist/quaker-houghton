---
name: List Quaker Houghton upcoming events
description: >-
  Read the public event calendar on home.quakerhoughton.com through the stable Events Calendar
  REST namespace, page through it correctly, and fall back gracefully when the calendar is empty
  (it was — total 0 — on 2026-09-17).
api: openapi/_original/quaker-houghton-tribe-events-v1-openapi-original.json
operations:
  - GET /events            # tribe/events/v1 declares no operationIds; keyed by method + path
  - GET /events/{id}
  - GET /venues/{id}
  - GET /organizers/{id}
related_operation_ids:
  - getEvents      # tec/v1 equivalents — gated behind the experimental-acknowledgement header
  - getEvent
  - getVenue
  - getOrganizer
auth: none (anonymous reads)
generated: '2026-09-17'
method: generated
---

# List Quaker Houghton upcoming events

Quaker Houghton publishes no product API. This skill covers the one anonymously callable read
surface on its host — the Events Calendar plugin's `tribe/events/v1` namespace — and nothing else.

## Steps

1. **Call the stable namespace, not the experimental one.**
   `GET https://home.quakerhoughton.com/wp-json/tribe/events/v1/events?page=1&per_page=20`
   Do not use `/wp-json/tec/v1/events`: it answers `400 missing_experimental_endpoint_acknowledgement`
   until a plugin-specific acknowledgement header is sent.
2. **Narrow the window with `start_date` / `end_date`** (also `starts_after`, `ends_before`,
   `search`, `categories`, `tags`, `venue`, `organizer`, `featured`). The default window is today
   through two years ahead — read it back from `rest_url` in the response.
3. **Page with the body, not a cursor.** The body carries `events[]`, `total`, `total_pages` and
   `rest_url`; the response also exposes `X-WP-Total`, `X-WP-TotalPages` and an RFC 5988 `Link`
   header. Stop when `page >= total_pages`.
4. **Handle the empty calendar.** `total: 0` with `events: []` is a valid, expected answer — it is
   what the host returned on 2026-09-17. Report "no published events", do not retry.
5. **Resolve related records only if needed.** Each event embeds `venue` and `organizer` objects
   plus `categories[]` and `tags[]` (Term). `GET /venues/{id}` and `GET /organizers/{id}` exist
   for direct lookup; `by-slug/{slug}` variants exist for every entity.

## Rules

- **Read-only.** Every POST/DELETE needs a WordPress Application Password Quaker Houghton does not
  issue; never attempt a write.
- **Errors** arrive as `{"code","message","data":{"status"}}` — `400` bad parameter, `403`
  inaccessible entity, `404` missing entity or page past the end. See
  `errors/quaker-houghton-problem-types.yml`.
- **No rate-limit headers** are returned; be polite (the site serves 1.5 MB HTML pages and is
  slow) and cache `rest_url` results.
- **Do not confuse this with the MCP server** at `/wp-json/mcp/mcp-oauth-server`, which is
  OAuth-gated and whose tools are unknown — see `mcp/quaker-houghton-mcp.yml`.
