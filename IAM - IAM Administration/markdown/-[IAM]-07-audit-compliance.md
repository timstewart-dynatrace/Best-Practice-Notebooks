# IAM-07: Audit Logging and Compliance

> **Series:** IAM — IAM Administration | **Notebook:** 7 of 12 | **Created:** January 2026 | **Last Updated:** 09/24/2026

## Meeting Regulatory and Security Requirements
Audit logging is essential for compliance (SOC2, SOX, HIPAA, PCI-DSS) and security operations. This notebook covers querying audit logs, building compliance reports, and conducting access reviews.

---

## Table of Contents

1. [Audit Log Fundamentals](#audit-log-fundamentals)
2. [Authentication Event Monitoring](#authentication-event-monitoring)
3. [Authorization and Access Events](#authorization-and-access-events)
4. [Configuration Change Tracking](#configuration-change-tracking)
5. [Compliance Reporting Patterns](#compliance-reporting-patterns)
6. [Access Review Workflows](#access-review-workflows)
7. [Building Compliance Dashboards](#building-compliance-dashboards)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Dynatrace Environment** | SaaS with Grail — Grail-based audit logging is on by default and cannot be turned off |
| **Permissions** | `ALLOW storage:system:read WHERE storage:event.kind = "AUDIT_EVENT";` and `ALLOW storage:buckets:read WHERE storage:bucket-name = "dt_system_events";` |
| **Account audit log** | The `account-company-info` account permission, to review account-level IAM changes in Account Management |
| **Data Retention** | Environment audit events are kept for one year; plan exports if a framework needs longer |

<a id="audit-log-fundamentals"></a>
## 1. Audit Log Fundamentals
Dynatrace audit logs capture all security-relevant events in your environment.

### What Gets Logged

| Category | Events |
|----------|--------|
| **Authentication** | Logins, logouts, MFA events, SSO |
| **Authorization** | Access attempts, permission checks |
| **User Management** | User creation, modification, deletion |
| **Group Management** | Group changes, membership updates |
| **Policy Changes** | Policy creation, modification, deletion |
| **Token Operations** | Token creation, revocation, usage |
| **Configuration** | Settings changes, API calls |

### Audit Log Fields

| Field | Description | Example |
|-------|-------------|----------|
| `timestamp` | Event time | `2026-01-26T14:30:00Z` |
| `event.kind` | Always `AUDIT_EVENT` for audit records | `AUDIT_EVENT` |
| `event.type` | Action | `LOGIN`, `GET`, `POST`, `CREATE`, `DELETE` |
| `event.outcome` | Result — `success` / `failure`, or an HTTP status | `success`, `403` |
| `event.provider` | Component that recorded the event | `API_GATEWAY`, `SETTINGS`, `API_TOKEN` |
| `user.id` | Acting user (a UUID, not an email address) | `37e52d7a-…` |
| `user.organization` | Organization of the acting user | `DYNATRACE` |
| `authentication.type` / `authentication.grant.type` | How the caller authenticated | `OAUTH2`, `TOKEN` |
| `resource` | The API path or object acted on | `/platform/document/v1/…` |
| `origin.address` | Client IP address | `203.0.113.10` |
| `request.source` | Where the request came from | `BROWSER`, `DT_SERVERLESS`, `OTHER` |

> **Two audit records, two places.** Activity *inside this environment* — sign-ins, API calls, settings and token changes — is recorded as `AUDIT_EVENT` records in `dt.system.events`, which every query in this notebook reads. **Account-level IAM changes** — groups, memberships, policies, boundaries — are recorded in the account audit log (Account Management > Settings > Audit log), not here.
>
> <sub>**Sources:** [Audit logs on Grail (DT docs)](https://docs.dynatrace.com/docs/manage/data-privacy-and-security/configuration/audit-logs-grail) — *"Grail-based audit logging is enabled by default and can't be turned off."*, [Account Management audit logs (DT docs)](https://docs.dynatrace.com/docs/manage/account-management/audit-logs) — *"Dynatrace provides audit logs of all changes to your account-level identity and access (IAM) management settings"*.</sub>

### Retention Requirements

| Compliance | Minimum Retention |
|------------|-------------------|
| SOC2 | 1 year |
| SOX | 7 years |
| HIPAA | 6 years |
| PCI-DSS | 1 year online, 1+ archive |
| GDPR | As long as necessary |

<a id="authentication-event-monitoring"></a>
## 2. Authentication Event Monitoring
Track who is accessing your Dynatrace environment and how.

```dql
// All authentication events in the last 24 hours
// Data object corrected 08/12/2026. The Dynatrace audit trail is NOT in `logs`: this cell used
// `fetch logs | filter matchesPhrase(log.source, "audit")`, and no log.source on a Grail tenant
// contains "audit" — the filter matched nothing, silently, forever. Platform audit records live in
// `dt.system.events` with `event.kind == "AUDIT_EVENT"` (265,000+ records over 7 days here), and
// they are STRUCTURED, so the old `matchesPhrase(content, ...)` string-scraping is replaced by real
// field predicates. Key fields: user.id, user.organization, event.type (GET/POST/PUT/PATCH/DELETE/
// CREATE/LOGIN/app.opened), event.outcome (HTTP status or "success"), authentication.type
// (OAUTH2/TOKEN/NONE), authentication.grant.type, resource (the API path), origin.address,
// origin.type, request.source, dt.app.id, event.provider.
// Enumerate your own with:
//   fetch dt.system.events, from:-24h | filter event.kind == "AUDIT_EVENT" | limit 1
fetch dt.system.events, from:-24h
| filter event.kind == "AUDIT_EVENT"
| filter isNotNull(authentication.type)
| summarize events = count(), by:{authentication.type, event.outcome}
| sort events desc
```

```dql
// Failed sign-in attempts - critical for security monitoring
// Corrected 09/24/2026: a failed sign-in is event.type == "LOGIN" with event.outcome == "failure".
// The previous "any non-2xx outcome" predicate counted 404 GETs and even 304 Not Modified.
// Data object corrected 08/12/2026. The Dynatrace audit trail is NOT in `logs`: this cell used
// `fetch logs | filter matchesPhrase(log.source, "audit")`, and no log.source on a Grail tenant
// contains "audit" — the filter matched nothing, silently, forever. Platform audit records live in
// `dt.system.events` with `event.kind == "AUDIT_EVENT"` (265,000+ records over 7 days here), and
// they are STRUCTURED, so the old `matchesPhrase(content, ...)` string-scraping is replaced by real
// field predicates. Key fields: user.id, user.organization, event.type (GET/POST/PUT/PATCH/DELETE/
// CREATE/LOGIN/app.opened), event.outcome (HTTP status or "success"), authentication.type
// (OAUTH2/TOKEN/NONE), authentication.grant.type, resource (the API path), origin.address,
// origin.type, request.source, dt.app.id, event.provider.
// Enumerate your own with:
//   fetch dt.system.events, from:-24h | filter event.kind == "AUDIT_EVENT" | limit 1
fetch dt.system.events, from:-7d
| filter event.kind == "AUDIT_EVENT"
| filter event.type == "LOGIN" and event.outcome == "failure"
| summarize failures = count(), by:{user.id, origin.address}
| sort failures desc
| limit 25
```

```dql
// Login activity by hour to identify patterns
// Data object corrected 08/12/2026. The Dynatrace audit trail is NOT in `logs`: this cell used
// `fetch logs | filter matchesPhrase(log.source, "audit")`, and no log.source on a Grail tenant
// contains "audit" — the filter matched nothing, silently, forever. Platform audit records live in
// `dt.system.events` with `event.kind == "AUDIT_EVENT"` (265,000+ records over 7 days here), and
// they are STRUCTURED, so the old `matchesPhrase(content, ...)` string-scraping is replaced by real
// field predicates. Key fields: user.id, user.organization, event.type (GET/POST/PUT/PATCH/DELETE/
// CREATE/LOGIN/app.opened), event.outcome (HTTP status or "success"), authentication.type
// (OAUTH2/TOKEN/NONE), authentication.grant.type, resource (the API path), origin.address,
// origin.type, request.source, dt.app.id, event.provider.
// Enumerate your own with:
//   fetch dt.system.events, from:-24h | filter event.kind == "AUDIT_EVENT" | limit 1
fetch dt.system.events, from:-7d
| filter event.kind == "AUDIT_EVENT"
| filter isNotNull(authentication.type)
| makeTimeseries logins = count(), interval:1h
```

### Authentication Patterns to Monitor

| Pattern | Indicates | Query Focus |
|---------|-----------|-------------|
| Multiple failed logins | Brute force attempt | Failed + same user |
| Off-hours logins | Unusual access | Timestamp analysis |
| New location logins | Account compromise | IP/location change |
| Rapid successive logins | Automation or sharing | Time between logins |
| Login after offboarding | Process failure | User status check |

```dql
// Off-hours activity detection (outside 08:00-18:00 UTC)
// Data object corrected 08/12/2026. The Dynatrace audit trail is NOT in `logs`: this cell used
// `fetch logs | filter matchesPhrase(log.source, "audit")`, and no log.source on a Grail tenant
// contains "audit" — the filter matched nothing, silently, forever. Platform audit records live in
// `dt.system.events` with `event.kind == "AUDIT_EVENT"` (265,000+ records over 7 days here), and
// they are STRUCTURED, so the old `matchesPhrase(content, ...)` string-scraping is replaced by real
// field predicates. Key fields: user.id, user.organization, event.type (GET/POST/PUT/PATCH/DELETE/
// CREATE/LOGIN/app.opened), event.outcome (HTTP status or "success"), authentication.type
// (OAUTH2/TOKEN/NONE), authentication.grant.type, resource (the API path), origin.address,
// origin.type, request.source, dt.app.id, event.provider.
// Enumerate your own with:
//   fetch dt.system.events, from:-24h | filter event.kind == "AUDIT_EVENT" | limit 1
fetch dt.system.events, from:-7d
| filter event.kind == "AUDIT_EVENT"
| fieldsAdd hour = formatTimestamp(timestamp, format: "HH")
| filter toLong(hour) < 8 or toLong(hour) >= 18
| summarize off_hours_events = count(), by:{user.id}
| sort off_hours_events desc
| limit 25
```

<a id="authorization-and-access-events"></a>
## 3. Authorization and Access Events
Track permission checks and access attempts to sensitive resources.

> **Group, policy, boundary and binding changes are account-level, and this environment's audit events are not their change log.** *"Dynatrace provides audit logs of all changes to your account-level identity and access (IAM) management settings"* — *"Administrators can view these logs in Account Management > Settings > Audit log"*, where they are kept for up to ten years (also queryable through the Account Audits API). The query below shows IAM API calls made **through this environment** — useful for spotting automation, not a change log. On the validation tenant, 30 days of these events held no group or policy change at all: only user lookups and scheduled `/lookups/iam_*` file uploads, which the query now excludes.
>
> <sub>**Sources:** [Account Management audit logs (DT docs)](https://docs.dynatrace.com/docs/manage/account-management/audit-logs).</sub>

```dql
// Access denied events - indicates permission issues or attacks
// Data object corrected 08/12/2026. The Dynatrace audit trail is NOT in `logs`: this cell used
// `fetch logs | filter matchesPhrase(log.source, "audit")`, and no log.source on a Grail tenant
// contains "audit" — the filter matched nothing, silently, forever. Platform audit records live in
// `dt.system.events` with `event.kind == "AUDIT_EVENT"` (265,000+ records over 7 days here), and
// they are STRUCTURED, so the old `matchesPhrase(content, ...)` string-scraping is replaced by real
// field predicates. Key fields: user.id, user.organization, event.type (GET/POST/PUT/PATCH/DELETE/
// CREATE/LOGIN/app.opened), event.outcome (HTTP status or "success"), authentication.type
// (OAUTH2/TOKEN/NONE), authentication.grant.type, resource (the API path), origin.address,
// origin.type, request.source, dt.app.id, event.provider.
// Enumerate your own with:
//   fetch dt.system.events, from:-24h | filter event.kind == "AUDIT_EVENT" | limit 1
fetch dt.system.events, from:-7d
| filter event.kind == "AUDIT_EVENT"
| filter event.outcome == "403"
| summarize denials = count(), by:{user.id, resource}
| sort denials desc
| limit 25
```

```dql
// IAM API activity in this environment (account-level IAM changes: see the note above)
// Data object corrected 08/12/2026. The Dynatrace audit trail is NOT in `logs`: this cell used
// `fetch logs | filter matchesPhrase(log.source, "audit")`, and no log.source on a Grail tenant
// contains "audit" — the filter matched nothing, silently, forever. Platform audit records live in
// `dt.system.events` with `event.kind == "AUDIT_EVENT"` (265,000+ records over 7 days here), and
// they are STRUCTURED, so the old `matchesPhrase(content, ...)` string-scraping is replaced by real
// field predicates. Key fields: user.id, user.organization, event.type (GET/POST/PUT/PATCH/DELETE/
// CREATE/LOGIN/app.opened), event.outcome (HTTP status or "success"), authentication.type
// (OAUTH2/TOKEN/NONE), authentication.grant.type, resource (the API path), origin.address,
// origin.type, request.source, dt.app.id, event.provider.
// Enumerate your own with:
//   fetch dt.system.events, from:-24h | filter event.kind == "AUDIT_EVENT" | limit 1
fetch dt.system.events, from:-30d
| filter event.kind == "AUDIT_EVENT"
| filter in(event.type, {"POST", "PUT", "PATCH", "DELETE", "CREATE", "UPDATE"}) and contains(resource, "iam")
| filter not startsWith(resource, "/lookups/")
| summarize changes = count(), by:{user.id, event.type}
| sort changes desc
```

### Access Event Categories

| Event Type | Security Relevance | Monitoring Priority |
|------------|-------------------|---------------------|
| Access denied | Permission misconfiguration or probe | High |
| Privilege escalation | Critical security event | Critical |
| Cross-environment access | Boundary violation | High |
| Admin action | Privileged operation | Medium |
| Bulk data access | Exfiltration risk | High |

<a id="configuration-change-tracking"></a>
## 4. Configuration Change Tracking
Track changes recorded in this environment. Account-level IAM changes (groups, policies, boundaries, bindings) are in the account audit log — see the note in Section 3.

```dql
// User creation and deletion events
// Data object corrected 08/12/2026. The Dynatrace audit trail is NOT in `logs`: this cell used
// `fetch logs | filter matchesPhrase(log.source, "audit")`, and no log.source on a Grail tenant
// contains "audit" — the filter matched nothing, silently, forever. Platform audit records live in
// `dt.system.events` with `event.kind == "AUDIT_EVENT"` (265,000+ records over 7 days here), and
// they are STRUCTURED, so the old `matchesPhrase(content, ...)` string-scraping is replaced by real
// field predicates. Key fields: user.id, user.organization, event.type (GET/POST/PUT/PATCH/DELETE/
// CREATE/LOGIN/app.opened), event.outcome (HTTP status or "success"), authentication.type
// (OAUTH2/TOKEN/NONE), authentication.grant.type, resource (the API path), origin.address,
// origin.type, request.source, dt.app.id, event.provider.
// Enumerate your own with:
//   fetch dt.system.events, from:-24h | filter event.kind == "AUDIT_EVENT" | limit 1
fetch dt.system.events, from:-30d
| filter event.kind == "AUDIT_EVENT"
| filter in(event.type, {"CREATE", "DELETE"})
| fields timestamp, user.id, event.type, resource, event.outcome
| sort timestamp desc
| limit 50
```

```dql
// IAM API activity in this environment — recent calls (group membership itself is account-level)
// Data object corrected 08/12/2026. The Dynatrace audit trail is NOT in `logs`: this cell used
// `fetch logs | filter matchesPhrase(log.source, "audit")`, and no log.source on a Grail tenant
// contains "audit" — the filter matched nothing, silently, forever. Platform audit records live in
// `dt.system.events` with `event.kind == "AUDIT_EVENT"` (265,000+ records over 7 days here), and
// they are STRUCTURED, so the old `matchesPhrase(content, ...)` string-scraping is replaced by real
// field predicates. Key fields: user.id, user.organization, event.type (GET/POST/PUT/PATCH/DELETE/
// CREATE/LOGIN/app.opened), event.outcome (HTTP status or "success"), authentication.type
// (OAUTH2/TOKEN/NONE), authentication.grant.type, resource (the API path), origin.address,
// origin.type, request.source, dt.app.id, event.provider.
// Enumerate your own with:
//   fetch dt.system.events, from:-24h | filter event.kind == "AUDIT_EVENT" | limit 1
fetch dt.system.events, from:-30d
| filter event.kind == "AUDIT_EVENT"
| filter in(event.type, {"POST", "PUT", "PATCH", "DELETE", "CREATE", "UPDATE"}) and contains(resource, "iam")
| filter not startsWith(resource, "/lookups/")
| fields timestamp, user.id, event.type, resource
| sort timestamp desc
| limit 50
```

```dql
// IAM API activity in this environment, by user (policy changes themselves: account audit log)
// Data object corrected 08/12/2026. The Dynatrace audit trail is NOT in `logs`: this cell used
// `fetch logs | filter matchesPhrase(log.source, "audit")`, and no log.source on a Grail tenant
// contains "audit" — the filter matched nothing, silently, forever. Platform audit records live in
// `dt.system.events` with `event.kind == "AUDIT_EVENT"` (265,000+ records over 7 days here), and
// they are STRUCTURED, so the old `matchesPhrase(content, ...)` string-scraping is replaced by real
// field predicates. Key fields: user.id, user.organization, event.type (GET/POST/PUT/PATCH/DELETE/
// CREATE/LOGIN/app.opened), event.outcome (HTTP status or "success"), authentication.type
// (OAUTH2/TOKEN/NONE), authentication.grant.type, resource (the API path), origin.address,
// origin.type, request.source, dt.app.id, event.provider.
// Enumerate your own with:
//   fetch dt.system.events, from:-24h | filter event.kind == "AUDIT_EVENT" | limit 1
fetch dt.system.events, from:-30d
| filter event.kind == "AUDIT_EVENT"
| filter in(event.type, {"POST", "PUT", "PATCH", "DELETE", "CREATE", "UPDATE"}) and contains(resource, "iam")
| filter not startsWith(resource, "/lookups/")
| summarize iam_api_calls = count(), by:{user.id}
| sort iam_api_calls desc
```

```dql
// Token lifecycle: API and ActiveGate token creation, update and deletion
// Corrected 09/24/2026: token lifecycle events carry event.provider API_TOKEN / ACTIVE_GATE_TOKEN,
// and their resource is the token ID, so the previous contains(resource, "token") filter missed
// every one of them and matched only scheduled /lookups/iam_tokens uploads.
// Data object corrected 08/12/2026. The Dynatrace audit trail is NOT in `logs`: this cell used
// `fetch logs | filter matchesPhrase(log.source, "audit")`, and no log.source on a Grail tenant
// contains "audit" — the filter matched nothing, silently, forever. Platform audit records live in
// `dt.system.events` with `event.kind == "AUDIT_EVENT"` (265,000+ records over 7 days here), and
// they are STRUCTURED, so the old `matchesPhrase(content, ...)` string-scraping is replaced by real
// field predicates. Key fields: user.id, user.organization, event.type (GET/POST/PUT/PATCH/DELETE/
// CREATE/LOGIN/app.opened), event.outcome (HTTP status or "success"), authentication.type
// (OAUTH2/TOKEN/NONE), authentication.grant.type, resource (the API path), origin.address,
// origin.type, request.source, dt.app.id, event.provider.
// Enumerate your own with:
//   fetch dt.system.events, from:-24h | filter event.kind == "AUDIT_EVENT" | limit 1
fetch dt.system.events, from:-30d
| filter event.kind == "AUDIT_EVENT"
| filter in(event.provider, {"API_TOKEN", "ACTIVE_GATE_TOKEN"})
| fields timestamp, user.id, event.provider, event.type, resource, event.outcome
| sort timestamp desc
| limit 50
```

### Change Tracking Summary

| Change Type | Audit Requirement | Review Frequency |
|-------------|-------------------|------------------|
| User lifecycle | Log all | Weekly |
| Group changes | Log all | Weekly |
| Policy changes | Log + approve | On change |
| Admin actions | Log + review | Daily |
| Token operations | Log all | Monthly |

<a id="compliance-reporting-patterns"></a>
## 5. Compliance Reporting Patterns
Standard queries for compliance frameworks.

### SOC2 - Access Control Evidence

SOC2 requires evidence of access controls and monitoring.

```dql
// SOC2: Users who signed in during the period
// Corrected 09/24/2026: this cell queried `fetch logs | filter matchesPhrase(log.source, "audit")`,
// which matches nothing on most tenants, or unrelated file-based audit logs (for example database
// .aud files). Platform audit records are AUDIT_EVENT records in dt.system.events.
fetch dt.system.events, from:-90d
| filter event.kind == "AUDIT_EVENT"
| filter event.type == "LOGIN" and event.outcome == "success"
| summarize {last_login = takeMax(timestamp), login_count = count()}, by:{user.id}
| sort last_login desc
| limit 500
```

```dql
// SOC2: Administrative action log
// Data object corrected 08/12/2026. The Dynatrace audit trail is NOT in `logs`: this cell used
// `fetch logs | filter matchesPhrase(log.source, "audit")`, and no log.source on a Grail tenant
// contains "audit" — the filter matched nothing, silently, forever. Platform audit records live in
// `dt.system.events` with `event.kind == "AUDIT_EVENT"` (265,000+ records over 7 days here), and
// they are STRUCTURED, so the old `matchesPhrase(content, ...)` string-scraping is replaced by real
// field predicates. Key fields: user.id, user.organization, event.type (GET/POST/PUT/PATCH/DELETE/
// CREATE/LOGIN/app.opened), event.outcome (HTTP status or "success"), authentication.type
// (OAUTH2/TOKEN/NONE), authentication.grant.type, resource (the API path), origin.address,
// origin.type, request.source, dt.app.id, event.provider.
// Enumerate your own with:
//   fetch dt.system.events, from:-24h | filter event.kind == "AUDIT_EVENT" | limit 1
fetch dt.system.events, from:-90d
| filter event.kind == "AUDIT_EVENT"
| filter in(event.type, {"POST", "PUT", "PATCH", "DELETE", "CREATE", "UPDATE"})
| fields timestamp, user.id, user.organization, event.type, resource, event.outcome
| sort timestamp desc
| limit 100
```

### SOX - Segregation of Duties

SOX requires segregation of duties and change authorization.

```dql
// SOX: Privileged user activity (write operations)
// Data object corrected 08/12/2026. The Dynatrace audit trail is NOT in `logs`: this cell used
// `fetch logs | filter matchesPhrase(log.source, "audit")`, and no log.source on a Grail tenant
// contains "audit" — the filter matched nothing, silently, forever. Platform audit records live in
// `dt.system.events` with `event.kind == "AUDIT_EVENT"` (265,000+ records over 7 days here), and
// they are STRUCTURED, so the old `matchesPhrase(content, ...)` string-scraping is replaced by real
// field predicates. Key fields: user.id, user.organization, event.type (GET/POST/PUT/PATCH/DELETE/
// CREATE/LOGIN/app.opened), event.outcome (HTTP status or "success"), authentication.type
// (OAUTH2/TOKEN/NONE), authentication.grant.type, resource (the API path), origin.address,
// origin.type, request.source, dt.app.id, event.provider.
// Enumerate your own with:
//   fetch dt.system.events, from:-24h | filter event.kind == "AUDIT_EVENT" | limit 1
fetch dt.system.events, from:-30d
| filter event.kind == "AUDIT_EVENT"
| filter in(event.type, {"POST", "PUT", "PATCH", "DELETE", "CREATE", "UPDATE"})
| summarize privileged_actions = count(), by:{user.id, event.type}
| sort privileged_actions desc
| limit 25
```

```dql
// SOX: Configuration changes requiring approval
// Data object corrected 08/12/2026. The Dynatrace audit trail is NOT in `logs`: this cell used
// `fetch logs | filter matchesPhrase(log.source, "audit")`, and no log.source on a Grail tenant
// contains "audit" — the filter matched nothing, silently, forever. Platform audit records live in
// `dt.system.events` with `event.kind == "AUDIT_EVENT"` (265,000+ records over 7 days here), and
// they are STRUCTURED, so the old `matchesPhrase(content, ...)` string-scraping is replaced by real
// field predicates. Key fields: user.id, user.organization, event.type (GET/POST/PUT/PATCH/DELETE/
// CREATE/LOGIN/app.opened), event.outcome (HTTP status or "success"), authentication.type
// (OAUTH2/TOKEN/NONE), authentication.grant.type, resource (the API path), origin.address,
// origin.type, request.source, dt.app.id, event.provider.
// Enumerate your own with:
//   fetch dt.system.events, from:-24h | filter event.kind == "AUDIT_EVENT" | limit 1
fetch dt.system.events, from:-30d
| filter event.kind == "AUDIT_EVENT"
| filter in(event.type, {"POST", "PUT", "PATCH", "DELETE", "CREATE", "UPDATE"}) and contains(resource, "settings")
| fields timestamp, user.id, event.type, resource, event.outcome
| sort timestamp desc
| limit 50
```

### HIPAA - PHI Access Logging

HIPAA requires logging of access to protected health information.

```dql
// HIPAA: All data access events
// Data object corrected 08/12/2026. The Dynatrace audit trail is NOT in `logs`: this cell used
// `fetch logs | filter matchesPhrase(log.source, "audit")`, and no log.source on a Grail tenant
// contains "audit" — the filter matched nothing, silently, forever. Platform audit records live in
// `dt.system.events` with `event.kind == "AUDIT_EVENT"` (265,000+ records over 7 days here), and
// they are STRUCTURED, so the old `matchesPhrase(content, ...)` string-scraping is replaced by real
// field predicates. Key fields: user.id, user.organization, event.type (GET/POST/PUT/PATCH/DELETE/
// CREATE/LOGIN/app.opened), event.outcome (HTTP status or "success"), authentication.type
// (OAUTH2/TOKEN/NONE), authentication.grant.type, resource (the API path), origin.address,
// origin.type, request.source, dt.app.id, event.provider.
// Enumerate your own with:
//   fetch dt.system.events, from:-24h | filter event.kind == "AUDIT_EVENT" | limit 1
fetch dt.system.events, from:-30d
| filter event.kind == "AUDIT_EVENT"
| filter event.type == "GET"
| summarize reads = count(), by:{user.id, resource}
| sort reads desc
| limit 50
```

```dql
// HIPAA: Access anomalies - bulk access detection
// Data object corrected 08/12/2026. The Dynatrace audit trail is NOT in `logs`: this cell used
// `fetch logs | filter matchesPhrase(log.source, "audit")`, and no log.source on a Grail tenant
// contains "audit" — the filter matched nothing, silently, forever. Platform audit records live in
// `dt.system.events` with `event.kind == "AUDIT_EVENT"` (265,000+ records over 7 days here), and
// they are STRUCTURED, so the old `matchesPhrase(content, ...)` string-scraping is replaced by real
// field predicates. Key fields: user.id, user.organization, event.type (GET/POST/PUT/PATCH/DELETE/
// CREATE/LOGIN/app.opened), event.outcome (HTTP status or "success"), authentication.type
// (OAUTH2/TOKEN/NONE), authentication.grant.type, resource (the API path), origin.address,
// origin.type, request.source, dt.app.id, event.provider.
// Enumerate your own with:
//   fetch dt.system.events, from:-24h | filter event.kind == "AUDIT_EVENT" | limit 1
fetch dt.system.events, from:-7d
| filter event.kind == "AUDIT_EVENT"
| summarize requests = count(), by:{user.id}
| filter requests > 1000
| sort requests desc
| limit 25
```

### PCI-DSS - Cardholder Data Access

PCI-DSS requires tracking access to cardholder data.

```dql
// PCI-DSS: Failed access attempts (potential attack indicator)
// Corrected 09/24/2026: failures are "failure" or an HTTP status of 400 or above. The previous
// "not 2xx" predicate also counted 304 Not Modified, which is a success.
// Data object corrected 08/12/2026. The Dynatrace audit trail is NOT in `logs`: this cell used
// `fetch logs | filter matchesPhrase(log.source, "audit")`, and no log.source on a Grail tenant
// contains "audit" — the filter matched nothing, silently, forever. Platform audit records live in
// `dt.system.events` with `event.kind == "AUDIT_EVENT"` (265,000+ records over 7 days here), and
// they are STRUCTURED, so the old `matchesPhrase(content, ...)` string-scraping is replaced by real
// field predicates. Key fields: user.id, user.organization, event.type (GET/POST/PUT/PATCH/DELETE/
// CREATE/LOGIN/app.opened), event.outcome (HTTP status or "success"), authentication.type
// (OAUTH2/TOKEN/NONE), authentication.grant.type, resource (the API path), origin.address,
// origin.type, request.source, dt.app.id, event.provider.
// Enumerate your own with:
//   fetch dt.system.events, from:-24h | filter event.kind == "AUDIT_EVENT" | limit 1
fetch dt.system.events, from:-30d
| filter event.kind == "AUDIT_EVENT"
| filter event.outcome == "failure" or toLong(event.outcome) >= 400
| summarize failures = count(), by:{user.id, origin.address, event.outcome}
| sort failures desc
| limit 50
```

### Compliance Query Reference

| Framework | Key Reports | Frequency |
|-----------|-------------|------------|
| SOC2 | User access, admin actions, logical access | Quarterly |
| SOX | Privileged users, change authorization | Quarterly |
| HIPAA | PHI access, anomaly detection | Monthly |
| PCI-DSS | Failed access, CDE access | Daily/Weekly |
| GDPR | Data access, consent tracking | On request |

<a id="access-review-workflows"></a>
## 6. Access Review Workflows
Periodic access reviews are required by most compliance frameworks.

### Access Review Process

| Step | Action | Owner |
|------|--------|-------|
| 1 | Generate user access report | IAM Admin |
| 2 | Distribute to managers | IAM Admin |
| 3 | Review and certify access | Managers |
| 4 | Remove inappropriate access | IAM Admin |
| 5 | Document review completion | IAM Admin |

```dql
// Access review: most and least active users in the window
// Data object corrected 08/12/2026. The Dynatrace audit trail is NOT in `logs`: this cell used
// `fetch logs | filter matchesPhrase(log.source, "audit")`, and no log.source on a Grail tenant
// contains "audit" — the filter matched nothing, silently, forever. Platform audit records live in
// `dt.system.events` with `event.kind == "AUDIT_EVENT"` (265,000+ records over 7 days here), and
// they are STRUCTURED, so the old `matchesPhrase(content, ...)` string-scraping is replaced by real
// field predicates. Key fields: user.id, user.organization, event.type (GET/POST/PUT/PATCH/DELETE/
// CREATE/LOGIN/app.opened), event.outcome (HTTP status or "success"), authentication.type
// (OAUTH2/TOKEN/NONE), authentication.grant.type, resource (the API path), origin.address,
// origin.type, request.source, dt.app.id, event.provider.
// Enumerate your own with:
//   fetch dt.system.events, from:-24h | filter event.kind == "AUDIT_EVENT" | limit 1
fetch dt.system.events, from:-180d
| filter event.kind == "AUDIT_EVENT"
| summarize {events = count(), last_seen = takeMax(timestamp)}, by:{user.id}
| sort last_seen asc
| limit 50
```

```dql
// Access review: High-activity users (potential over-privilege)
// Corrected 09/24/2026: this cell queried `fetch logs | filter matchesPhrase(log.source, "audit")`,
// which matches nothing on most tenants, or unrelated file-based audit logs (for example database
// .aud files). Platform audit records are AUDIT_EVENT records in dt.system.events.
fetch dt.system.events, from:-30d
| filter event.kind == "AUDIT_EVENT"
| summarize action_count = count(), by:{user.id}
| sort action_count desc
| limit 50
```

### Access Review Checklist

```
□ All users verified as current employees
□ Access levels appropriate for job function
□ No separation of duties violations
□ Privileged access justified and approved
□ Inactive accounts identified for removal
□ Service accounts still required
□ Review documented and signed off
```

### Review Frequency by Access Level

| Access Level | Review Frequency | Approver |
|--------------|------------------|----------|
| Viewer | Annually | Manager |
| Editor | Semi-annually | Manager + Security |
| Admin | Quarterly | Director + Security |
| Account Admin | Monthly | CISO |

<a id="building-compliance-dashboards"></a>
## 7. Building Compliance Dashboards
Create dashboards for ongoing compliance monitoring.

### Dashboard Components

| Section | Metrics | Chart Type |
|---------|---------|------------|
| Authentication | Logins/failures over time | Line chart |
| Access Denied | Failed access attempts | Bar chart |
| User Changes | Create/delete/modify | Table |
| Admin Activity | Privileged actions | Table |
| Token Operations | Creates/revocations | Table |

```dql
// Dashboard: Sign-in trend (for line chart)
// Corrected 09/24/2026: this cell queried `fetch logs | filter matchesPhrase(log.source, "audit")`,
// which matches nothing on most tenants, or unrelated file-based audit logs (for example database
// .aud files). Platform audit records are AUDIT_EVENT records in dt.system.events.
fetch dt.system.events, from:-30d
| filter event.kind == "AUDIT_EVENT"
| filter event.type == "LOGIN" and event.outcome == "success"
| makeTimeseries logins = count(), interval:1d
```

```dql
// Dashboard: Failed access by day (for bar chart) — "failure" or HTTP 400 and above
// Data object corrected 08/12/2026. The Dynatrace audit trail is NOT in `logs`: this cell used
// `fetch logs | filter matchesPhrase(log.source, "audit")`, and no log.source on a Grail tenant
// contains "audit" — the filter matched nothing, silently, forever. Platform audit records live in
// `dt.system.events` with `event.kind == "AUDIT_EVENT"` (265,000+ records over 7 days here), and
// they are STRUCTURED, so the old `matchesPhrase(content, ...)` string-scraping is replaced by real
// field predicates. Key fields: user.id, user.organization, event.type (GET/POST/PUT/PATCH/DELETE/
// CREATE/LOGIN/app.opened), event.outcome (HTTP status or "success"), authentication.type
// (OAUTH2/TOKEN/NONE), authentication.grant.type, resource (the API path), origin.address,
// origin.type, request.source, dt.app.id, event.provider.
// Enumerate your own with:
//   fetch dt.system.events, from:-24h | filter event.kind == "AUDIT_EVENT" | limit 1
fetch dt.system.events, from:-30d
| filter event.kind == "AUDIT_EVENT"
| filter event.outcome == "failure" or toLong(event.outcome) >= 400
| makeTimeseries failures = count(), interval:1d
```

```dql
// Dashboard: Recent administrative actions (for table)
// Data object corrected 08/12/2026. The Dynatrace audit trail is NOT in `logs`: this cell used
// `fetch logs | filter matchesPhrase(log.source, "audit")`, and no log.source on a Grail tenant
// contains "audit" — the filter matched nothing, silently, forever. Platform audit records live in
// `dt.system.events` with `event.kind == "AUDIT_EVENT"` (265,000+ records over 7 days here), and
// they are STRUCTURED, so the old `matchesPhrase(content, ...)` string-scraping is replaced by real
// field predicates. Key fields: user.id, user.organization, event.type (GET/POST/PUT/PATCH/DELETE/
// CREATE/LOGIN/app.opened), event.outcome (HTTP status or "success"), authentication.type
// (OAUTH2/TOKEN/NONE), authentication.grant.type, resource (the API path), origin.address,
// origin.type, request.source, dt.app.id, event.provider.
// Enumerate your own with:
//   fetch dt.system.events, from:-24h | filter event.kind == "AUDIT_EVENT" | limit 1
fetch dt.system.events, from:-7d
| filter event.kind == "AUDIT_EVENT"
| filter in(event.type, {"POST", "PUT", "PATCH", "DELETE", "CREATE", "UPDATE"})
| fields timestamp, user.id, event.type, resource, event.outcome
| sort timestamp desc
| limit 25
```

### Alert Thresholds

Configure alerts based on audit log patterns:

| Condition | Threshold | Severity |
|-----------|-----------|----------|
| Failed logins | > 5 in 10 min same user | Warning |
| Failed logins | > 20 in 10 min any | Critical |
| Admin action | Any outside business hours | Warning |
| Policy change | Any | Info |
| User deletion | Any | Info |
| Bulk data access | > 1000 queries/hour | Warning |

## Next Steps

With compliance monitoring in place, proceed to multi-environment management:

### Recommended Path

1. **IAM-08: Multi-Environment IAM** - Scale IAM across environments
2. **IAM-09: Troubleshooting Access Issues** - Debug permission problems

### Compliance Checklist

Before moving on, ensure you have:

- [ ] Identified applicable compliance frameworks
- [ ] Created queries for required audit reports
- [ ] Established access review schedule
- [ ] Built compliance monitoring dashboard
- [ ] Configured alerts for critical events
- [ ] Documented audit log retention policy

---

## Summary

In this notebook, you learned:

- Audit log fundamentals and retention requirements
- Authentication event monitoring patterns
- Authorization and access event tracking
- Configuration change monitoring
- Compliance-specific query patterns (SOC2, SOX, HIPAA, PCI-DSS)
- Access review workflows
- Compliance dashboard components

---

## References

- [Audit logs on Grail (DT docs)](https://docs.dynatrace.com/docs/manage/data-privacy-and-security/configuration/audit-logs-grail)
- [Account Management audit logs (DT docs)](https://docs.dynatrace.com/docs/manage/account-management/audit-logs)
- [DQL Reference](https://docs.dynatrace.com/docs/platform/grail/dynatrace-query-language)
- [SOC2 Trust Principles](https://www.aicpa-cima.com/resources/landing/system-and-organization-controls-soc-suite-of-services)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
