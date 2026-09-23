---
name: complyadvantage-onboard-and-screen-a-customer
description: >-
  Create a customer in ComplyAdvantage Mesh and screen them against sanctions, warnings,
  PEP and adverse-media data in one workflow, then read the outcome. Use for KYC/AML
  onboarding at account opening.
generated: '2026-08-27'
method: generated
source: openapi/complyadvantage-mesh-api-openapi.json + https://docs.mesh.complyadvantage.com/docs/simple-step-by-step-integration
api: complyadvantage:complyadvantage-mesh-platform-api
base_url: https://api.mesh.complyadvantage.com
operations:
  - createTokenV3
  - entityScreeningGetConfigurations
  - createCustomerAndScreenSync
  - createCustomerAndScreenAsync
  - getWorkflowState
  - customerGetCustomerV2
  - riskScoringManagerServiceAPIGetRiskScore
permissions:
  - Create and screen customers
  - View customers
entitlements:
  - Access to base customer screening functionality
---

# Onboard and screen a customer

Creates the customer record, scores initial risk, screens against the AML database, raises
alerts, and opens a case for an analyst — five steps in one call.

## Before you start

This is a **billable, non-reversible** action. There is no delete-customer operation in the
contract: a customer created in error can only be transitioned to `Closed`
(`customerTransitionCustomerStatus`), never removed. Screens count against the account's
onboarding search volume. Confirm the `external_identifier` is not already in use before
calling — a duplicate returns `409` with `properties.field_name: duplicated`.

## Steps

1. **Mint a token.** `POST /v3/token` (`createTokenV3`) with the access key and secret from
   Settings > Access Management > API Credentials. The token is a bearer token valid for
   86,400 seconds with **no refresh grant** — cache it and mint a fresh one on expiry, do
   not retry a 401 with the same token.
2. **Pick a screening configuration.** `GET /v2/entity-screening/configurations`
   (`entityScreeningGetConfigurations`). Note the `configuration_identifier`. Which lists
   and thresholds apply is a compliance decision, not a technical one — do not create a new
   configuration (`entityScreeningCreateConfiguration`) on your own initiative.
3. **Create and screen.** Choose the mode deliberately:
   - `POST /v2/workflows/sync/create-and-screen` (`createCustomerAndScreenSync`) blocks
     until screening completes and returns hits in
     `step_details.customer-screening.step_output.screening_result`. Set the
     `last_sync_step` query parameter to `ALERTING` if you also need alert identifiers in
     the response. This endpoint is **idempotent by behaviour** — on a downstream error you
     may re-send the byte-identical request and the workflow is retried.
   - `POST /v2/workflows/create-and-screen` (`createCustomerAndScreenAsync`) returns a
     `workflow-instance-identifier` immediately.
   Name rules are strict: supply **exactly one** of `full_name` or `last_name`. With
   `full_name`, every one of `title`, `first_name`, `middle_name`, `last_name`,
   `fathers_name`, `mothers_name` and `suffix` must be omitted.
4. **Wait for the async result.** Either poll
   `GET /v2/workflows/{workflow_instance_identifier}` (`getWorkflowState`) until `status`
   is `COMPLETED` or `ERRORED`, or subscribe to the `WORKFLOW_COMPLETED` webhook. Prefer
   the webhook. Step statuses are `NOT-STARTED`, `IN-PROGRESS`, `COMPLETED`, `SKIPPED`,
   `ERRORED`; `SKIPPED` on the screening step means the customer's risk was prohibited.
5. **Read the record.** `GET /v2/customers/{customer_identifier}` (`customerGetCustomerV2`)
   — the identifier is in
   `step_details.customer-creation.step_output.customer_identifier`. Customer status is
   `Processing`, `Active` or `Closed`.
6. **Read the score.** `GET /v2/customers/{customer_identifier}/scores`
   (`riskScoringManagerServiceAPIGetRiskScore`) for the overall level plus the per-category
   breakdown.

## Interpreting the result

`screening_result` is `HAS_PROFILES` or equivalent, with `aml_types` drawn from the AML
typology: `SANCTION`, `WARNING`, `PEP_CLASS_1`, `PEP_CLASS_2`, `ADVERSE_MEDIA`,
`ADVERSE_MEDIA_V2_CYBERCRIME`. **Treat this enum as open** — the versioning policy states
that adding new enum values is a backwards-compatible change, so handle unknown members
without failing.

## Errors and conventions

- Errors are RFC 9457 problem+json with extra required `identifier` (UUID) and `timestamp`
  members — quote both to support. `type` is always `about:blank`, so route on `status`.
- `403` means the credential's role lacks the permission or the account lacks the
  entitlement. Neither is retryable.
- `429` does **not** use the problem+json envelope — it returns `{"message": "API rate limit
  exceeded"}`. Back off using the `Ratelimit-Reset` response header.
- See `conventions/complyadvantage-conventions.yml`,
  `errors/complyadvantage-problem-types.yml`, `scopes/complyadvantage-scopes.yml`.
