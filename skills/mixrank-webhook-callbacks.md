---
name: Receive MixRank results asynchronously over a webhook
description: Turn a slow email prospect or validate call into an asynchronous one with the `webhook` query parameter, then verify the Standard Webhooks signature on the delivery before trusting it.
api: openapi/mixrank-email-api-openapi.yml
operations: [prospectEmail, validateEmail]
---

# Receive MixRank results asynchronously over a webhook

`prospectEmail` and `validateEmail` can take a long time to answer. MixRank lets you
opt out of waiting: pass a `webhook` URL on the request and MixRank hands the result
back later as an HTTP callback. This is the only event surface MixRank publishes —
there is no subscription API, no event catalogue, and no AsyncAPI document.

## Auth

API key as a path segment: `https://api.mixrank.com/v2/json/{api_key}/...`.
The same key is the HMAC secret used to sign deliveries, so an agent that can call
the API can also verify its callbacks — never expose the webhook receiver publicly
without checking the signature.

## Flow

1. **Ask asynchronously.** Call `prospectEmail` (`GET /email/prospect`) or
   `validateEmail` (`GET /email/validate`) with an extra `webhook` query parameter
   holding an **https** URL you control.
2. **Take the acknowledgement.** Instead of the normal payload you get
   `202` with `{"id": ..., "status": "accepted"}`. Record that `id` — it is how you
   will correlate the delivery. Do not poll; there is no status endpoint for this.
3. **Receive the delivery.** MixRank POSTs to your URL when the result is ready:
   `{"id": ..., "type": ..., "timestamp": ..., "data": ...}`. `id` repeats the
   acknowledgement id, `type` names the event, `timestamp` is when the result was
   produced, and `data` is the response you would have received synchronously —
   or an error message if the request could not be answered.
4. **Verify before you trust it.** The delivery follows
   [Standard Webhooks](https://www.standardwebhooks.com/) and carries three headers:
   - `Webhook-Id` — mirrors the body's `id`
   - `Webhook-Timestamp` — unix seconds
   - `Webhook-Signature` — `v1,<base64 hmac-sha256 of "<id>.<timestamp>.<body>">`,
     keyed with your API key
   Recompute the HMAC and compare in constant time. **Reject** a body whose
   signature does not match, and one whose timestamp is older than you are willing
   to accept. Any off-the-shelf Standard Webhooks client library will do this.
5. **Answer 2xx.** MixRank retries with backoff until your endpoint returns 2xx and
   abandons the delivery after 10 attempts. A non-2xx from a slow handler costs you
   the result, so acknowledge first and process after.

## Notes

- The `type` field names the event, but MixRank does not publish the list of event
  type values. Do not hard-code a value you have not observed on your own account.
- This is a *per-request* callback, not a subscription: nothing is delivered unless
  a specific call asked for it. There is no replay endpoint — a delivery abandoned
  after 10 attempts is gone, and the work must be re-requested.
- Deliveries are not idempotency-keyed, but `Webhook-Id` repeats the request `id`,
  so dedupe on it. MixRank documents no `Idempotency-Key` header anywhere in the
  API — see `conventions/mixrank-conventions.yml`.
- The bulk path is different and separately documented: `createEmailBulkJob` is
  polled with `getEmailBulkJob`, not delivered by webhook. See
  `skills/mixrank-verify-email.md`.
- Full capture: `asyncapi/mixrank-webhooks.yml`.
