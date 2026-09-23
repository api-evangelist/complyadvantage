---
name: complyadvantage-receive-and-verify-webhooks
description: >-
  Configure, verify and process ComplyAdvantage Mesh webhooks — Standard Webhooks v1
  HMAC-SHA256 signature verification, replay protection, and at-least-once delivery
  handling. Use when building the receiving end of a Mesh integration.
generated: '2026-08-27'
method: generated
source: >-
  openapi/complyadvantage-mesh-api-openapi.json +
  https://docs.mesh.complyadvantage.com/docs/webhooks
api: complyadvantage:complyadvantage-mesh-platform-api
base_url: https://api.mesh.complyadvantage.com
operations:
  - createTokenV3
  - notificationCreateWebhookConfiguration
  - notificationGetWebhookConfigurations
  - notificationPatchWebhookConfiguration
  - notificationTestWebhookConfiguration
permissions:
  - Create and update webhooks
  - View webhooks
---

# Receive and verify ComplyAdvantage webhooks

## Configure

`POST /v2/notifications/configurations/webhook`
(`notificationCreateWebhookConfiguration`) with `name`, `url`, `is_active` and a single
`type`. One event type per configuration — subscribing to all ten means ten calls. Valid
types:

`CASE_CREATED`, `CASE_TRANSITIONED`, `CASE_ALERT_LIST_UPDATED`, `WORKFLOW_COMPLETED`,
`TRANSACTION_REVIEWED`, `TRANSACTION_MONITORING_ASYNC_COMPLETED`,
`CUSTOMER_RISK_SCORE_CHANGED`, `CUSTOMER_RISK_LEVEL_INCREASED`,
`CUSTOMER_RISK_LEVEL_DECREASED`, and the deprecated `CASE_STATE_UPDATED` (use
`CASE_TRANSITIONED`).

`POST /v2/notifications/configurations/webhook/test`
(`notificationTestWebhookConfiguration`) sends a sample payload to a URL you nominate — do
this before going live. **There is no delete operation for a webhook configuration**; the
only way to stop delivery is `PATCH /v2/notifications/configurations/webhook/{identifier}`
(`notificationPatchWebhookConfiguration`) with `is_active: false`.

## Allowlist the source

Deliveries come from a fixed set of IPs per region — for example
`99.81.100.211, 99.81.116.122, 99.81.123.126` for `api.eu.mesh.complyadvantage.com` and
`34.67.184.6, 35.225.126.158, 34.70.66.162` for `api.us.mesh.complyadvantage.com`. The full
per-region table is in `asyncapi/complyadvantage-webhooks.yml`.

## Verify the signature

Signing is **opt-in per account** — ask support to enable it and issue a 256-bit secret.
The format is Standard Webhooks v1, so any Standard Webhooks library works out of the box.
If you implement it yourself:

1. Capture the **raw request body** before any JSON parsing. Frameworks that parse and
   re-serialize will change whitespace and key order and break the signature.
2. Read `webhook-id`, `webhook-timestamp` (Unix seconds, not milliseconds) and
   `webhook-signature`.
3. Reject if `webhook-timestamp` is outside ±5 minutes of now.
4. Base64-decode the secret using the **standard alphabet** (`+`, `/`, `=`), not URL-safe,
   to get 32 raw bytes. There is no `whsec_` prefix.
5. Build `{webhook-id}.{webhook-timestamp}.{body}` joined by literal dots.
6. `base64(HMAC-SHA256(secret, canonical_string))`.
7. Strip the `v1,` prefix from each space-separated entry in `webhook-signature` and compare
   with a **constant-time** comparison. Accept if **any** entry matches — multiple
   signatures appear during a zero-downtime secret rotation.

## Process idempotently

Delivery is **at-least-once**. Three retries at 1s, 5s and 10s, and an attempt that has not
responded within **5 seconds** is recorded as failed and retried — so a slow endpoint sees
duplicates exactly like an unavailable one.

`webhook-id` is stable across every attempt of the same message. Use it as the idempotency
key: check it against ids you have already handled, skip if seen, otherwise write the
payload somewhere durable, return `2xx` immediately, and do the real work asynchronously.

## Read the payload version

Payloads carry their own `api_version` (`v1`, `v2`, `v3`) independent of the URL path
version. `CASE_CREATED` and `CASE_TRANSITIONED` emit both v1 and v2 — use v2, which carries
`case_stage` and the customer `external_identifier`. Ignore `case_state`; it is deprecated
and reports `USER_DEFINED` for any stage added after your integration.
