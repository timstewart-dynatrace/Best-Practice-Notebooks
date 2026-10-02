# OPIPE-05: Business & Security Event Pipelines

> **Series:** OPIPE — OpenPipeline Beyond Logs | **Notebook:** 5 of 6 | **Created:** March 2026 | **Last Updated:** 10/02/2026

## Processing Business Transactions and Security Events at Ingestion

Business events and security events are two OpenPipeline scopes with distinct governance requirements. Business events drive revenue analytics, conversion tracking, and SLO calculations. Security events drive compliance reporting, threat detection, and audit trails. Both benefit from ingestion-time processing — enrichment, routing, and metric extraction — but the rules are different.

This notebook covers the Business Events and Security Events scopes in OpenPipeline. For log processing, see **OPLOGS-01: OpenPipeline Fundamentals**. For compliance and masking patterns, see **OPLOGS-08: Security & Data Protection** and **OPMIG-08: Security & Masking**.

---

## Table of Contents

1. [Business Events Scope](#business-events-scope)
2. [Business Event Enrichment](#business-event-enrichment)
3. [Business Event Metric Extraction](#business-event-metric-extraction)
4. [Security Events Scope](#security-events-scope)
5. [Security Event Routing and Compliance](#security-event-routing)
6. [Event-to-Metric Extraction for KPIs](#event-to-metric)
7. [Summary](#summary)
8. [Next Steps](#next-steps)
9. [References](#references)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Dynatrace Environment** | SaaS with Grail and business events enabled |
| **Permissions** | `storage:bizevents:read`, `storage:security.events:read`; `settings:objects:write` on the `builtin:openpipeline.bizevents.*` / `builtin:openpipeline.security.events.*` schemas to configure pipelines (OpenPipeline configuration is stored as Settings objects) |
| **Data** | Business events ingested via OneAgent capture rules, the RUM API, the business events API, or OpenPipeline extraction |
| **Recommended** | **OPIPE-01** (multi-scope architecture), **BIZEV** series for business event analytics |

<a id="business-events-scope"></a>
## 1. Business Events Scope

Business events represent **user transactions and business actions** — purchases, sign-ups, page views, API calls, form submissions. They are ingested through:

- **OneAgent** — capture rules on incoming requests to monitored services
- **RUM (web and mobile)** — an explicit call to the RUM JavaScript API, OneAgent for Mobile, or OpenKit; user actions are not turned into business events automatically
- **Business events API** — JSON events sent from external systems and backend services
- **OpenPipeline** — business events extracted from logs and spans (Data extraction stage)
- **Workflows** — the *Ingest business event* action

### Key Business Event Fields

| Field | Description | Example |
|-------|-------------|--------|
| `event.type` | The business event type | `com.myapp.purchase.completed` |
| `event.provider` | The source system | `www.myshop.com` |
| `event.category` | Event classification | `purchase`, `signup`, `pageview` |
| `event.group` | Logical grouping | `checkout-flow`, `onboarding` |
| Custom attributes | Business-specific data | `order.total`, `product.category`, `user.tier` |

### Discovering Your Business Event Landscape

```dql
// Business event volume by type and provider
fetch bizevents, from:-24h
| summarize event_count = count(), by:{event.type, event.provider}
| sort event_count desc
| limit 20
```

```dql
// Business event trend by type over 24h
fetch bizevents, from:-24h
| makeTimeseries event_count = count(), by:{event.type}, interval:1h
```

<a id="business-event-enrichment"></a>
## 2. Business Event Enrichment

Business events often arrive with technical identifiers that need business context. OpenPipeline enrichment adds meaningful fields at ingestion.

### Common Enrichment Patterns

| Condition | Enrichment Field | Value | Purpose |
|-----------|-----------------|-------|--------|
| `matchesValue(event.type, "*purchase*")` | `business.domain` | `"commerce"` | Domain classification |
| `event.provider == "mobile-app"` | `channel` | `"mobile"` | Channel attribution |
| `event.provider == "www.myshop.com"` | `channel` | `"web"` | Channel attribution |
| Custom: `order.total > 1000` | `order.tier` | `"high_value"` | Revenue tier classification |
| Custom: `user.country == "US" or user.country == "CA"` | `region` | `"north_america"` | Geographic grouping |

### Security Context for Business Events

Sensitive business data (revenue figures, user details) should be access-controlled:

| Condition | `dt.security_context` Value | Rationale |
|-----------|----------------------------|-----------|
| `matchesValue(event.type, "*payment*")` | `"finance-team"` | Payment data restricted to finance |
| `matchesValue(event.type, "*signup*")` | `"marketing-team"` | User acquisition data for marketing |
| `true` (last) | `"product-team"` | General product analytics |

The Permission stage runs **first match only**, so list the specific conditions first and the `true` catch-all last.

<a id="business-event-metric-extraction"></a>
## 3. Business Event Metric Extraction

Extract metrics from business events for dashboards, SLOs, and alerting.

### Example: E-Commerce Metrics

| Metric Key | Processor | Condition | Dimensions |
|-----------|-------------|-----------|------------|
| `bizevent.purchase_count` | Counter metric | `event.type == "purchase.completed"` | `event.provider`, `channel` |
| `bizevent.purchase_revenue` | Value metric on `order.total` | `event.type == "purchase.completed"` | `event.provider`, `channel` |
| `bizevent.cart_abandonment` | Counter metric | `event.type == "cart.abandoned"` | `event.provider` |
| `bizevent.signup_count` | Counter metric | `event.type == "user.signup"` | `channel`, `region` |

### Conversion Funnel Metrics

Track progression through a multi-step funnel by extracting a count metric at each stage:

| Funnel Stage | Event Type | Metric Key |
|-------------|-----------|------------|
| Product viewed | `product.viewed` | `funnel.stage_1_views` |
| Added to cart | `cart.item_added` | `funnel.stage_2_adds` |
| Checkout started | `checkout.started` | `funnel.stage_3_checkout` |
| Purchase completed | `purchase.completed` | `funnel.stage_4_purchase` |

```dql
// Conversion funnel from business events (last 24h)
fetch bizevents, from:-24h
| summarize views = countIf(event.type == "product.viewed"),
    adds = countIf(event.type == "cart.item_added"),
    checkouts = countIf(event.type == "checkout.started"),
    purchases = countIf(event.type == "purchase.completed")
```

<a id="security-events-scope"></a>
## 4. Security Events Scope

Security events include threat detections, vulnerability findings, compliance findings, and events from external security tools. They are stored in their own table, **`security.events`** — not in `events` — so query them with `fetch security.events`. They require special handling:

- **Never drop** — Security events must be retained for compliance, even if volume is high
- **Never sample** — Every security event is potentially significant
- **Long retention** — compliance frameworks and internal policy commonly call for a year or more (see §5)
- **Strict access control** — Only security teams should access security event data

### Key Security Event Types

| Category | `event.type` | Description |
|----------|--------------|-------------|
| Vulnerabilities | `VULNERABILITY_FINDING`, `VULNERABILITY_SCAN`, `VULNERABILITY_STATE_REPORT_EVENT`, `VULNERABILITY_STATUS_CHANGE_EVENT`, … | Vulnerability findings and state from Dynatrace and third-party tools |
| Detections | `DETECTION_FINDING` | Alerts from security tools; Runtime Application Protection attacks carry `product.name == "Runtime Application Protection"` |
| Compliance | `COMPLIANCE_FINDING`, `COMPLIANCE_SCAN_COMPLETED` | Security Posture Management scan results |

> <sub>**Sources:** [Threat Observability concepts (DT docs)](https://docs.dynatrace.com/docs/secure/threat-observability/concepts) — event-type tables per category.</sub>

> **Platform audit events are not security events.** Configuration changes and user actions are recorded as `AUDIT_EVENT` records in `dt.system.events`, outside the security-events scope.

```dql
// Security event overview (security events are stored in security.events, not events)
fetch security.events, from:-24h
| summarize event_count = count(), by:{event.type}
| sort event_count desc
```

<a id="security-event-routing"></a>
## 5. Security Event Routing and Compliance

Security events should always be routed to **dedicated buckets** with extended retention and restricted access.

### Routing Strategy

| Pipeline | Routing Condition | Target Bucket | Retention | Security Context |
|----------|-------------------|---------------|-----------|------------------|
| `vulnerability-events` | `matchesValue(event.type, "VULNERABILITY_*")` | `security_vulnerabilities` | 365 days | `"security-team"` |
| `detection-findings` | `event.type == "DETECTION_FINDING"` | `security_detections` | 365 days | `"security-team"` |
| `compliance-events` | `matchesValue(event.type, "COMPLIANCE_*")` | `compliance_events` | 2555 days (7 years) | `"compliance-team"` |

Platform audit events (`AUDIT_EVENT`) are not in this scope — they live in `dt.system.events` (§4). Which security event types pass through your pipelines depends on how they are ingested, so check the routing preview against real records before relying on a route.

### Compliance Framework Requirements

The periods below are the ones commonly cited for each framework, not legal advice — confirm the period that applies to you with your compliance team and auditor before you size a bucket:

| Framework | Commonly cited retention | Key Event Types |
|-----------|--------------------------|----------------|
| **SOC 2** | No fixed period in the framework — set by your policy and auditor; one year is common | Access logs, configuration changes, incident records |
| **HIPAA** | Six years for required documentation; log retention follows your policy | Access to PHI, authentication events, audit trails |
| **PCI-DSS** | One year of audit log history, with the most recent three months immediately available | Cardholder data access, authentication, network events |
| **GDPR** | No longer than necessary for the purpose | Data access, consent, deletion requests |

<a id="event-to-metric"></a>
## 6. Event-to-Metric Extraction for KPIs

Extract operational KPIs from both business and security events for dashboards and alerting.

### Business KPIs

| KPI | Metric Source | Extraction Rule |
|-----|-------------|----------------|
| Conversion rate | `purchase.completed` / `product.viewed` | Extract count for each, compute ratio in dashboard |
| Average order value | `order.total` from `purchase.completed` | Value metric; average at query time |
| Revenue per hour | `order.total` from `purchase.completed` | Value metric; sum it per hour at query time |

### Security KPIs

| KPI | Metric Source | Extraction Rule |
|-----|-------------|----------------|
| Attack attempts per hour | `DETECTION_FINDING` events | Counter metric; sum per hour at query time |
| Open vulnerabilities | Vulnerability events | Not a metric-extraction job — metric processors count, sum and histogram; they do not count distinct values. Query `security.events` directly |
| Failed auth attempts | Authentication events | Extract count where `status == "FAILED"` |

```dql
// Security event trend over 7 days
fetch security.events, from:-7d
| makeTimeseries event_count = count(), by:{event.type}, interval:24h
```

---

<a id="summary"></a>
## Summary

In this notebook you learned:

- **Business events scope** — Sources, key fields, and discovery queries for transaction data
- **Business event enrichment** — Adding business context, channel attribution, and security context at ingestion
- **Metric extraction from business events** — Conversion funnels, revenue metrics, and operational KPIs
- **Security events scope** — Never drop, never sample, always retain with extended retention
- **Compliance-driven routing** — Dedicated buckets with retention aligned to SOC 2, HIPAA, PCI-DSS requirements
- **Event-to-metric extraction** — KPI dashboards for both business and security monitoring

---

<a id="next-steps"></a>
## Next Steps

Continue to **OPIPE-06: Cross-Scope Design Patterns** to learn how to correlate data across scopes, cascade processing from spans to events to metrics, and design a production-ready OpenPipeline architecture.

---

<a id="references"></a>
## References

- [Business event capture (DT docs)](https://docs.dynatrace.com/docs/observe/business-observability/bo-events-capturing)
- [Get business events from logs and spans (DT docs)](https://docs.dynatrace.com/docs/observe/business-observability/bo-events-capturing/bo-events-capturing-logs-and-spans)
- [Business event bucket assignment via OpenPipeline (DT docs)](https://docs.dynatrace.com/docs/observe/business-observability/bo-event-processing/bo-bucket-assignment-openpipeline)
- [Application Security (DT docs)](https://docs.dynatrace.com/docs/secure/application-security)
- [Threat Observability concepts (DT docs)](https://docs.dynatrace.com/docs/secure/threat-observability/concepts)
- [IAM policy statements (DT docs)](https://docs.dynatrace.com/docs/manage/identity-access-management/permission-management/manage-user-permissions-policies/advanced/iam-policystatements) — lists `storage:security.events:read`
- [Data retention periods (DT docs)](https://docs.dynatrace.com/docs/manage/data-privacy-and-security/data-privacy/data-retention-periods)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
