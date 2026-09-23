---
name: complyadvantage-triage-a-screening-case
description: >-
  Work a ComplyAdvantage Mesh case end to end — read its alerts and risks, decision each
  risk, transition the case through its stage machine, and leave an audit trail. Use for
  alert adjudication and analyst workflow automation.
generated: '2026-08-27'
method: generated
source: openapi/complyadvantage-mesh-api-openapi.json + https://docs.mesh.complyadvantage.com/docs/viewing-screening-results
api: complyadvantage:complyadvantage-mesh-platform-api
base_url: https://api.mesh.complyadvantage.com
operations:
  - createTokenV3
  - casesService/v2/cases.get
  - casesService/v2/customers/customer_identifier/cases.get
  - casesService/v2/cases/case_identifier.get
  - casesService/v2/cases/case_identifier/alerts.get
  - alertServiceGetRisks
  - alertServiceChangeRiskStatus
  - alertServiceTransitionAlertState
  - casesService/v2/cases/states.get
  - casesService/v2/cases/case_identifier/transition.post
  - casesService/v2/cases/case_identifier/assign.post
  - casesService/v2/cases/case_identifier/notes.post
  - auditGetCaseAuditTrailV2
permissions:
  - View cases (customer onboarding)
  - View cases (customer monitoring)
  - View alerts
  - Update alerts
  - Update risks
  - Update cases
---

# Triage a screening case

## Find the case

`GET /v2/cases` (`casesService/v2/cases.get`) with filters and sort, or
`GET /v2/customers/{customer_identifier}/cases`
(`casesService/v2/customers/customer_identifier/cases.get`) when you already have the
customer. Both paginate with `page_number` / `page_size` and return
`first`/`prev`/`self`/`next`/`total_count`.

Case read access is **partitioned by the product that raised the case** — you need one of
"View cases (customer onboarding)", "(customer monitoring)", "(payment screening)" or
"(transaction monitoring)". A `403` on a case list usually means the wrong one of the four.

## Read the evidence

1. `GET /v2/cases/{case_identifier}` (`casesService/v2/cases/case_identifier.get`) — read
   `case_stage` and ignore `case_state`, which is deprecated in favour of it. If any new
   stage is added to the account's workflow, `case_state` reports it as `USER_DEFINED`.
2. `GET /v2/cases/{case_identifier}/alerts`
   (`casesService/v2/cases/case_identifier/alerts.get`) — the alerts grouped into this case.
3. `GET /v2/alerts/{alert_identifier}/risks` (`alertServiceGetRisks`) — the individual risk
   findings inside each alert. This is the level a decision is actually made at.

## Decide

- **Per risk:** `POST /v2/alerts/{alert_identifier}/risks/{risk_identifier}/decision`
  (`alertServiceChangeRiskStatus`).
- **Per alert:** `POST /v2/alerts/{alert_identifier}/transition`
  (`alertServiceTransitionAlertState`).
- **Per case:** first `GET /v2/cases/states` (`casesService/v2/cases/states.get`) to read
  the stages this case can legally move to — stages are **account-configured**, so never
  hard-code a target stage. Then
  `POST /v2/cases/{case_identifier}/transition`
  (`casesService/v2/cases/case_identifier/transition.post`).

Stages carry a `stage_type` (`INITIAL`, `DECISION`) and a `decision_type`
(`POSITIVE`, and others). A transition into a `DECISION` stage is the consequential move.

## Reversibility

Case and alert transitions are reversible **only in the direction the account's stage
machine permits** — `GET /v2/cases/states` is the authority, and no time window is
published for any of it. Notes are never deleted. Before transitioning into a decision
stage, leave the reasoning as a note:
`POST /v2/cases/{case_identifier}/notes`
(`casesService/v2/cases/case_identifier/notes.post`).

## Hand off and audit

- Assign: `POST /v2/cases/{case_identifier}/assign`
  (`casesService/v2/cases/case_identifier/assign.post`).
- Audit trail: `GET /v2/audit/cases/{case_identifier}` (`auditGetCaseAuditTrailV2`).

## Bulk

`POST /v2/cases/transition/bulk`, `POST /v2/cases/assign/bulk` and
`POST /v2/cases/notes/bulk` each handle **up to 100 cases** and answer `207 Multi-Status` —
you must read the per-item results, not just the HTTP status.
