---
name: complyadvantage-screen-and-review-a-transaction
description: >-
  Submit a transaction to ComplyAdvantage Mesh for monitoring and payment screening, read
  the evaluation outcome, and decide a held transaction. Use for real-time payment
  screening and transaction-monitoring automation.
generated: '2026-08-27'
method: generated
source: openapi/complyadvantage-mesh-api-openapi.json
api: complyadvantage:complyadvantage-mesh-platform-api
base_url: https://api.mesh.complyadvantage.com
operations:
  - createTokenV3
  - orchestrationLayerAPIPostV3TransactionProcess
  - meshSearchServiceGetTransactions
  - activityStoreGetTransactionUsingInternalIdentifier
  - activityStoreGetTransactionUsingExternalIdentifier
  - activityStoreListTransactionVersions
  - activityStoreSetTransactionRiskTypes
  - activityStorePostTransactionReviewDecision
  - auditGetTransactionAuditTrailV2
permissions:
  - View transactions
  - View cases (transaction monitoring)
entitlements:
  - Access to base transaction monitoring functionality
  - Access to base payment screening functionality
---

# Screen and review a transaction

## Submit

`POST /v3/transactions/process` (`orchestrationLayerAPIPostV3TransactionProcess`) creates and
evaluates a transaction. Supply your own external identifier — every retrieval path accepts
it, so you never have to persist a ComplyAdvantage id.

## Read the outcome

- Synchronously, from the response.
- Asynchronously, from the `TRANSACTION_MONITORING_ASYNC_COMPLETED` webhook, which carries
  `monitoring_outcome` plus `scenario_evaluations[]` — the per-scenario results with
  `scenario_name`, `priority` and `scenario_outcome` (e.g. `HOLD`).

Retrieve later with `GET /v3/transactions/{identifier}`
(`activityStoreGetTransactionUsingInternalIdentifier`),
`GET /v3/transactions/external/{identifier}`
(`activityStoreGetTransactionUsingExternalIdentifier`), or list with
`GET /v3/transactions` (`meshSearchServiceGetTransactions`).

Transactions are **versioned**: `GET /v3/transactions/{identifier}/versions`
(`activityStoreListTransactionVersions`) returns every revision, and a single version is
addressable. This is the audit substrate for a monitoring decision.

## Classify

`PUT /v3/transactions/{identifier}/risk_types` (`activityStoreSetTransactionRiskTypes`) sets
or clears risk types. It is a `PUT`, so it replaces the whole set — read before you write.

## Decide a held transaction — STOP AND READ

`POST /v3/transactions/{identifier}/review`
(`activityStorePostTransactionReviewDecision`) releases or rejects a transaction the
monitoring engine put on `HOLD`.

**This is the most consequential and least reversible operation in the ComplyAdvantage
contract.** No reversal, cancel, void or undo operation for a review decision exists
anywhere in the spec, and no window is published because there is nothing to reverse it
with. Releasing a transaction that should have been held is a compliance failure with money
attached; rejecting one that should have been released is a blocked customer payment.

An agent must not call this autonomously. Escalate to a human, capture the decision and its
reasoning, and confirm the outcome afterwards via
`GET /v2/audit/transactions/{transaction_identifier}` (`auditGetTransactionAuditTrailV2`) and
the `TRANSACTION_REVIEWED` webhook, which reports `evaluation_outcome` and `review_decision`.

## Conventions

Bearer token from `POST /v3/token`, 24-hour life, no refresh. Errors are RFC 9457 problem
details except `429`, which returns `{"message": "API rate limit exceeded"}` — read
`Ratelimit-Limit`, `Ratelimit-Remaining` and `Ratelimit-Reset` on every response.
