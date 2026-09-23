---
name: complyadvantage-monitor-and-rescreen-customers
description: >-
  Keep an existing ComplyAdvantage Mesh customer under ongoing AML monitoring, rescreen on
  demand, react to risk-level changes, and produce a screening certificate. Use for
  perpetual KYC and periodic review.
generated: '2026-08-27'
method: generated
source: openapi/complyadvantage-mesh-api-openapi.json + https://docs.mesh.complyadvantage.com/docs/onboarding
api: complyadvantage:complyadvantage-mesh-platform-api
base_url: https://api.mesh.complyadvantage.com
operations:
  - createTokenV3
  - customerGetMonitoringConfig
  - customerUpdateCustomerMonitorConfiguration
  - orchestrationLayerAPIPostV2RescreenCustomerWorkflowsSync
  - updateCustomer
  - riskScoringManagerServiceAPIGetRiskScore
  - riskScoringManagerServiceAPIUpdateRiskScore
  - alertServiceCreateAlertMute
  - alertServiceDeleteAlertMute
  - screeningCertificatesServiceAPIPostReportsV2
  - auditGetCustomerAuditTrailV2
permissions:
  - View customer monitoring status
  - Monitor and unmonitor customers
  - View customers
  - Update customers
  - Create and delete mutes
entitlements:
  - Access to base customer monitoring functionality
  - Account has access to monitor on demand functionality
---

# Monitor and rescreen customers

## Turn monitoring on

`GET /v2/customers/{customer_identifier}/monitor` (`customerGetMonitoringConfig`) reads the
current state; `PATCH /v2/customers/{customer_identifier}/monitor`
(`customerUpdateCustomerMonitorConfiguration`) sets it. If no monitoring configuration is
assigned, rescreening falls back to the configuration used at initial screening.

## Rescreen on demand

`POST /v2/customers/{customer_identifier}/workflows/sync/rescreen`
(`orchestrationLayerAPIPostV2RescreenCustomerWorkflowsSync`) screens against the latest data
and raises an alert if new relevant risk appears.

This is the **one operation in the entire 163-operation contract that accepts an idempotency
key**: send `X-ComplyAdvantage-Idempotency-Key` (optional, max 255 chars, a UUID is
recommended). Use it — rescreen events count towards the account's onboarding search
volumes, so a retried request without a key is a billable duplicate. No retention window for
a replayed key is published, so do not assume a key is honoured indefinitely.

## Update the record

`POST /v2/customers/{customer_identifier}/workflows/sync/update-and-rescore`
(`updateCustomer`) updates the customer and recalculates risk. Two traps:

- **The customer profile must be supplied in full.** This endpoint overwrites the previous
  record; any field you omit is treated as removed and is deleted.
- **`products` is all-or-nothing.** Omit the key entirely to leave products untouched;
  supply it and you must include every product — new, changed and unchanged.

It is documented as idempotent by behaviour: on a downstream error, re-send the identical
request.

## Risk scores

`GET /v2/customers/{customer_identifier}/scores`
(`riskScoringManagerServiceAPIGetRiskScore`) returns the overall score and level plus a
weighted per-category breakdown.

`PATCH /v2/customers/{customer_identifier}/scores`
(`riskScoringManagerServiceAPIUpdateRiskScore`) overrides the level manually. **This has a
one-way side effect**: the score type becomes `MANUAL` and the system stops recalculating it
automatically. The override persists until explicitly changed again — so the value is
reversible with no deadline, but setting the value back does not by itself restore automatic
scoring. Do not override on an agent's own judgement; this is a compliance decision.

## Suppress noise

`POST /v2/alerts/mutes` (`alertServiceCreateAlertMute`) mutes a recurring alert for a
customer;
`DELETE /v2/alerts/mutes/{alert_mute_key}/customer/{customer_identifier}`
(`alertServiceDeleteAlertMute`) unmutes it. Fully reversible, no window published. A v3 pair
(`alertServiceCreateAlertMuteV3` / `alertServiceDeleteAlertMuteV3`) exists alongside it.

## React to changes

Subscribe to `CUSTOMER_RISK_SCORE_CHANGED`, `CUSTOMER_RISK_LEVEL_INCREASED` and
`CUSTOMER_RISK_LEVEL_DECREASED`. Each carries both `risk_score` and `previous_risk_score`,
so the delta needs no prior state on your side. One event type per webhook configuration —
three events means three configurations. See
`asyncapi/complyadvantage-webhooks.yml`.

## Evidence

`POST /v2/customers/{customer_identifier}/reports`
(`screeningCertificatesServiceAPIPostReportsV2`) generates a screening certificate.
**Read the status code carefully**: `201` means generated and the response carries a download
URL; `200` means the data is not ready yet and you should retry shortly. Treating `200` as
success is the mistake this endpoint invites.

`GET /v2/audit/customers/{customer_identifier}` (`auditGetCustomerAuditTrailV2`) is the
append-only trail for regulatory defensibility.
