---
name: sync-completed-calls-to-crm
description: >-
  Pull completed Allo calls — with recordings, transcripts, and AI summaries —
  and sync them into a CRM or database. Use for nightly/periodic sync workflows.
api: Allo API
base_url: https://api.withallo.com
auth: 'Authorization: Api-Key ak_live_<key>  (scope: CONVERSATIONS_READ)'
operations:
  - listConversations
  - searchAllConversations
  - getConversationItem
  - batchGetConversationItems
generated: '2026-07-21'
method: generated
source: openapi/the-mobile-first-company-allo-openapi.json
---

# Sync completed calls to your CRM

Grounded in real Allo operations. Every request needs
`Authorization: Api-Key ak_live_<key>` with the `CONVERSATIONS_READ` scope.

## Steps

1. **List recent conversations.** Call `listConversations`
   (`GET /v2/api/conversations`) with `page`/`size` (max `size` = 100). Read the
   `pagination` object and keep paging while `has_more` is `true`.
   - For incremental sync, prefer `last_activity_since` so you only fetch
     conversations with new activity instead of re-fetching everything.

2. **Narrow to completed calls.** Use `searchAllConversations`
   (`POST /v2/api/conversations/items/search`) to filter items — keyword search
   runs across transcripts, summaries, and message content, and supports date and
   type filters. Only calls in a completed state carry a recording, transcript,
   and summary.

3. **Hydrate details in bulk.** For the item ids you need full detail on, call
   `batchGetConversationItems` (`POST /v2/api/conversations/items/batch`, max 100
   ids per request) to fetch summary, tags, and `recording_url` in one round
   trip. For a single item use `getConversationItem`
   (`GET /v2/api/conversations/items/{id}`).
   - Item ids are prefixed: `cll-` for calls, `msg-` for messages. Passing an
     unrecognized prefix returns `INVALID_ITEM_ID`.

4. **Write into your system.** Map `id`, `from_number`, `to`, `type`, `result`,
   `summary`, and `recording_url` onto your CRM records.

## Rules

- **Rate limits:** reads are capped at 20 req/s. Respect `X-RateLimit-Remaining`
  and back off on `429 RATE_LIMIT_EXCEEDED` using `Retry-After`.
- **Pagination:** always drive the loop off `pagination.has_more`, not a fixed
  page count (see conventions/the-mobile-first-company-conventions.yml).
- **Errors:** handle `CONVERSATION_ITEM_NOT_FOUND` (404) gracefully; see
  errors/the-mobile-first-company-problem-types.yml.
- **Cost/efficiency:** batch (step 3) instead of N single-item GETs; use
  `total_count` to size the job without walking every page.
