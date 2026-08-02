---
name: send-sms-follow-up
description: >-
  Send a follow-up SMS from one of your Allo numbers after a call, safely and
  idempotently, and optionally tag the conversation.
api: Allo API
base_url: https://api.withallo.com
auth: 'Authorization: Api-Key ak_live_<key>  (scopes: PHONE_NUMBERS_READ, SMS_SEND, TAGS_WRITE)'
operations:
  - getNumbers
  - listNumbers
  - sendSMS
  - addTags
generated: '2026-07-21'
method: generated
source: openapi/the-mobile-first-company-allo-openapi.json
---

# Send an SMS follow-up

Grounded in real Allo operations. Requests need
`Authorization: Api-Key ak_live_<key>`.

## Steps

1. **Pick a sending number.** Call `listNumbers` (`GET /v2/api/numbers`, scope
   `PHONE_NUMBERS_READ`) — or the v1 `getNumbers` (`GET /v1/api/numbers`) — and
   choose an Allo number that is SMS-enabled and in the same country as the
   recipient.

2. **Send the message.** Call `sendSMS` (`POST /v1/api/sms`, scope `SMS_SEND`)
   with the `allo_number`, the recipient number, and the body. For French
   recipients use `sendSMSFrance` (`POST /v1/api/sms#france`) with a verified
   Sender ID.
   - **Phone format:** all numbers must be E.164 (e.g. `+14155551234`) or you get
     `INVALID_PHONE_FORMAT`.
   - **Idempotency (required for safe retries):** send a unique
     `Idempotency-Key` header (a UUID). A successful response is stored for 1
     hour; retrying with the same key to the same endpoint replays the stored
     response (`Idempotency-Replayed: true`) instead of sending twice. Reusing a
     key on a different endpoint returns `409 IDEMPOTENCY_KEY_REUSE`.

3. **Tag the conversation (optional).** Call `addTags`
   (`POST /v2/api/conversations/items/{id}/tags`, scope `TAGS_WRITE`) with a
   non-empty `tags` array to mark the call as followed-up. List valid tags with
   `listTags` (`GET /v2/api/tags`).

## Rules

- **Writes are limited to 5 req/s** — back off on `429`.
- **Common send errors** (see errors/the-mobile-first-company-problem-types.yml):
  `NUMBER_NOT_SMS_ENABLED`, `TO_NUMBER_COUNTRY_MISMATCH`,
  `LANDLINE_NUMBER_NOT_SUPPORTED`, `MESSAGE_NOT_COMPLIANT`,
  `TRIAL_SMS_LIMIT_REACHED` / `SMS_LIMIT_REACHED` (upgrade or wait).
- **Trial accounts** cannot use the API (`API_KEY_TRIAL_NOT_ALLOWED`).
- Never hardcode the API key; store it in an environment variable.
