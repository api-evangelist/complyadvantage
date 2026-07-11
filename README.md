# ComplyAdvantage (complyadvantage)

ComplyAdvantage provides AI-driven anti-money laundering (AML) and financial crime risk detection - screening people and companies against sanctions and watchlists, politically exposed persons (PEPs and RCAs), and adverse media, with ongoing monitoring that alerts you when a customer's risk profile changes. The REST API (api.complyadvantage.com, with US and APAC regional bases) exposes searches, monitored searches, case management, comments, tags, and users with api-key auth, plus webhooks for match and monitoring updates. The newer Mesh platform API adds customer lifecycle screening, cases and alerts, transaction monitoring, and fraud detection workflows.

**Access model:** the API reference at [docs.complyadvantage.com](https://docs.complyadvantage.com/) is fully public, but API keys are account-gated - they are generated inside the ComplyAdvantage web platform. The self-serve Starter Plan (from $99/month, sized by a 100-2,000 monitored-entity allowance) includes full API access; payments screening and transaction monitoring require a custom-priced Enterprise agreement. Sandbox accounts are available for integration testing. The OpenAPI document and collections in this repository were modeled from the publicly documented endpoints; paths and parameters are grounded in the published reference, while response schemas are summarized rather than exhaustive.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/complyadvantage/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/complyadvantage/refs/heads/main/apis.yml)

## Tags

- Anti-Money Laundering
- AML
- Fraud Detection
- Sanctions Screening
- Compliance
- PEP Screening
- Adverse Media
- KYC
- Watchlists
- Transaction Monitoring
- Financial Crime
- RegTech

## Timestamps

- **Created:** 2026-07-11
- **Modified:** 2026-07-11

## APIs

### ComplyAdvantage Search API

Screen a person, company, organisation, vessel, or aircraft against ComplyAdvantage's sanctions, watchlists, warnings, fitness and probity, PEP, and adverse media data. Create searches with fuzziness control, exact-match mode, and filters (entity type, birth year, country codes, risk types), list and retrieve past searches, pull full hit details and paginated entities, update match status, risk level, and whitelisting on hits, and download a search certificate PDF for your audit trail.

- **Human URL:** [https://docs.complyadvantage.com/api-docs/](https://docs.complyadvantage.com/api-docs/)
- **Base URL:** `https://api.complyadvantage.com`

#### Tags

- Sanctions Screening
- PEP Screening
- Adverse Media
- AML

#### Properties

- [Documentation](https://docs.complyadvantage.com/)
- [API Reference](https://docs.complyadvantage.com/api-docs/)
- [OpenAPI](openapi/complyadvantage-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Postman Collection](collections/complyadvantage.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/complyadvantage.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### ComplyAdvantage Monitored Search API

Turn any search into ongoing monitoring so customers are automatically re-screened as sanctions lists, PEP registries, and adverse media change. Start and stop monitoring on a search, swap the associated search profile, retrieve what changed (new, updated, and removed matching entities), and acknowledge changes. The `monitored_search_updated` webhook pushes change notifications to your systems.

- **Human URL:** [https://docs.complyadvantage.com/api-docs/](https://docs.complyadvantage.com/api-docs/)
- **Base URL:** `https://api.complyadvantage.com`

#### Tags

- Ongoing Monitoring
- AML
- Alerts

#### Properties

- [API Reference](https://docs.complyadvantage.com/api-docs/)
- [OpenAPI](openapi/complyadvantage-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Postman Collection](collections/complyadvantage.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/complyadvantage.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### ComplyAdvantage Case Management API

Treat screening results as managed compliance cases - assign a search to an analyst, set match status (potential match, true positive, false positive, no match) and risk level, add and retrieve comments on a search, attach and detach key-value tags, and use client references for client-scoped whitelisting of recurring false positives.

- **Human URL:** [https://docs.complyadvantage.com/api-docs/](https://docs.complyadvantage.com/api-docs/)
- **Base URL:** `https://api.complyadvantage.com`

#### Tags

- Case Management
- Compliance
- Audit Trail

#### Properties

- [API Reference](https://docs.complyadvantage.com/api-docs/)
- [OpenAPI](openapi/complyadvantage-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Postman Collection](collections/complyadvantage.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/complyadvantage.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### ComplyAdvantage Users API

List the users on your ComplyAdvantage account so searches and cases can be assigned to the right analyst by user ID.

- **Human URL:** [https://docs.complyadvantage.com/api-docs/](https://docs.complyadvantage.com/api-docs/)
- **Base URL:** `https://api.complyadvantage.com`

#### Tags

- Users
- Administration

#### Properties

- [API Reference](https://docs.complyadvantage.com/api-docs/)
- [OpenAPI](openapi/complyadvantage-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Postman Collection](collections/complyadvantage.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/complyadvantage.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### ComplyAdvantage Mesh Platform API

ComplyAdvantage's newer Mesh platform API, authenticated with OAuth2 bearer tokens (24-hour validity). Covers the full customer lifecycle - customer creation and screening (sync or async), risk scoring, cases and alerts with stage transitions and assignments, transaction screening and monitoring for payments fraud, custom and lookup lists, webhook and email notifications, data exports, user and role administration, and regulatory reporting (CTR/SAR prefill). Documented separately at [docs.mesh.complyadvantage.com](https://docs.mesh.complyadvantage.com/) with its own OpenAPI schema for customers.

- **Human URL:** [https://docs.mesh.complyadvantage.com/](https://docs.mesh.complyadvantage.com/)
- **Base URL:** `https://api.mesh.complyadvantage.com`

#### Tags

- Customer Screening
- Transaction Monitoring
- Fraud Detection
- Cases

#### Properties

- [Documentation](https://docs.mesh.complyadvantage.com/)

## Common Properties

- [GitHub Organization](https://github.com/complyadvantage)
- [LinkedIn](https://www.linkedin.com/company/complyadvantage)
- [Website](https://complyadvantage.com)
- [Documentation](https://docs.complyadvantage.com)
- [Pricing](https://complyadvantage.com/pricing/)
- [Plans](plans/complyadvantage-plans-pricing.yml)
- [Rate Limits](rate-limits/complyadvantage-rate-limits.yml)
- [Fin Ops](finops/complyadvantage-finops.yml)
- [Blog](https://complyadvantage.com/insights/)

## Maintainers

**FN:** Kin Lane
**Email:** kin@apievangelist.com
