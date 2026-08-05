# OPIPE-05: Business & Security Event Pipelines

> **Series:** OPIPE — OpenPipeline Beyond Logs | **Notebook:** 5 of 6 | **Created:** March 2026 | **Last Updated:** 08/04/2026

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
| **Permissions** | `storage:bizevents:read`, `storage:events:read`, `openpipeline:configurations:write` |
| **Data** | Business events ingested via OneAgent RUM, API, or OpenTelemetry |
| **Recommended** | **OPIPE-01** (multi-scope architecture), **BIZEV** series for business event analytics |

<a id="business-events-scope"></a>
## 1. Business Events Scope

Business events represent **user transactions and business actions** — purchases, sign-ups, page views, API calls, form submissions. They are ingested through:

- **OneAgent RUM** — Automatic capture of user actions on web and mobile apps
- **Business Events API** — Custom events sent from backend services
- **OpenTelemetry** — Span-derived business events

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
| `event.type` contains `"purchase"` | `business.domain` | `"commerce"` | Domain classification |
| `event.provider == "mobile-app"` | `channel` | `"mobile"` | Channel attribution |
| `event.provider == "www.myshop.com"` | `channel` | `"web"` | Channel attribution |
| Custom: `order.total > 1000` | `order.tier` | `"high_value"` | Revenue tier classification |
| Custom: `user.country` matches `{"US", "CA"}` | `region` | `"north_america"` | Geographic grouping |

### Security Context for Business Events

Sensitive business data (revenue figures, user details) should be access-controlled:

| Condition | `dt.security_context` Value | Rationale |
|-----------|----------------------------|-----------|
| `event.type` contains `"payment"` | `"finance-team"` | Payment data restricted to finance |
| `event.type` contains `"signup"` | `"marketing-team"` | User acquisition data for marketing |
| Default | `"product-team"` | General product analytics |

<a id="business-event-metric-extraction"></a>
## 3. Business Event Metric Extraction

Extract metrics from business events for dashboards, SLOs, and alerting.

### Example: E-Commerce Metrics

| Metric Key | Aggregation | Condition | Dimensions |
|-----------|-------------|-----------|------------|
| `bizevent.purchase_count` | count | `event.type == "purchase.completed"` | `event.provider`, `channel` |
| `bizevent.purchase_revenue` | sum(`order.total`) | `event.type == "purchase.completed"` | `event.provider`, `channel` |
| `bizevent.cart_abandonment` | count | `event.type == "cart.abandoned"` | `event.provider` |
| `bizevent.signup_count` | count | `event.type == "user.signup"` | `channel`, `region` |

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

Security events include threat detections, vulnerability findings, audit records, and compliance-relevant actions. They require special handling:

- **Never drop** — Security events must be retained for compliance, even if volume is high
- **Never sample** — Every security event is potentially significant
- **Long retention** — Compliance frameworks (SOC 2, HIPAA, PCI-DSS) require 1-7 year retention
- **Strict access control** — Only security teams should access security event data

### Key Security Event Types

| Event Type | Source | Description |
|-----------|--------|------------|
| Runtime vulnerability detections | OneAgent | Third-party library vulnerabilities |
| Attack detections | Application Security | SQL injection, command injection attempts |
| Audit events | Platform | Configuration changes, user actions |
| Custom security events | API ingestion | SIEM, IDS/IPS, firewall events |

```dql
// Security event overview
fetch events, from:-24h
| filter event.kind == "SECURITY_EVENT"
| summarize event_count = count(), by:{event.type}
| sort event_count desc
```

<a id="security-event-routing"></a>
## 5. Security Event Routing and Compliance

Security events should always be routed to **dedicated buckets** with extended retention and restricted access.

### Routing Strategy

| Pipeline | Routing Condition | Target Bucket | Retention | Security Context |
|----------|-------------------|---------------|-----------|------------------|
| `vulnerability-detections` | `event.type` matches `"*VULNERABILITY*"` | `security_vulnerabilities` | 365 days | `"security-team"` |
| `attack-detections` | `event.type` matches `"*ATTACK*"` | `security_attacks` | 365 days | `"security-team"` |
| `audit-events` | `event.type` matches `"*AUDIT*"` | `security_audit` | 365 days | `"security-team"` |
| `compliance-events` | Custom compliance markers | `compliance_events` | 2555 days (7 years) | `"compliance-team"` |

### Compliance Framework Requirements

| Framework | Retention Requirement | Key Event Types |
|-----------|----------------------|----------------|
| **SOC 2** | 1 year minimum | Access logs, configuration changes, incident records |
| **HIPAA** | 6 years | Access to PHI, authentication events, audit trails |
| **PCI-DSS** | 1 year (3 months readily available) | Cardholder data access, authentication, network events |
| **GDPR** | As long as necessary | Data access, consent, deletion requests |

<a id="event-to-metric"></a>
## 6. Event-to-Metric Extraction for KPIs

Extract operational KPIs from both business and security events for dashboards and alerting.

### Business KPIs

| KPI | Metric Source | Extraction Rule |
|-----|-------------|----------------|
| Conversion rate | `purchase.completed` / `product.viewed` | Extract count for each, compute ratio in dashboard |
| Average order value | `order.total` from `purchase.completed` | Extract avg aggregation |
| Revenue per hour | `order.total` from `purchase.completed` | Extract sum aggregation, interval 1h |

### Security KPIs

| KPI | Metric Source | Extraction Rule |
|-----|-------------|----------------|
| Attack attempts per hour | Attack detection events | Extract count, interval 1h |
| Open vulnerabilities | Vulnerability events | Extract countDistinct by CVE ID |
| Failed auth attempts | Authentication events | Extract count where `status == "FAILED"` |

```dql
// Security event trend over 7 days
fetch events, from:-7d
| filter event.kind == "SECURITY_EVENT"
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

- [Get business events from logs and spans (DT docs)](https://docs.dynatrace.com/docs/observe/business-observability/bo-events-capturing/bo-events-capturing-logs-and-spans)
- [Business event bucket assignment via OpenPipeline (DT docs)](https://docs.dynatrace.com/docs/observe/business-observability/bo-event-processing/bo-bucket-assignment-openpipeline)
- [Application Security (DT docs)](https://docs.dynatrace.com/docs/secure/application-security)
- [Data retention periods (DT docs)](https://docs.dynatrace.com/docs/manage/data-privacy-and-security/data-privacy/data-retention-periods)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
