# IAM-07: Audit Logging and Compliance

> **Series:** IAM — IAM Administration | **Notebook:** 7 of 12 | **Created:** January 2026 | **Last Updated:** 08/03/2026

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
| **Dynatrace Environment** | SaaS with Gen3 IAM and audit logging enabled |
| **Permissions** | `logs.read` with access to audit log bucket |
| **Data Retention** | Audit logs retained per compliance requirements |

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
| `log.source` | Log category | `audit` |
| `content` | Event details | JSON with action details |
| `dt.security.user` | Acting user | `user@company.com` |
| `dt.source_entity` | Source entity | Client IP, service |

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
fetch logs, from: now() - 24h
| filter matchesPhrase(log.source, "audit")
| filter matchesPhrase(content, "login") or matchesPhrase(content, "authentication")
| fields timestamp, content
| sort timestamp desc
| limit 100
```

```dql
// Failed login attempts - critical for security monitoring
fetch logs, from: now() - 7d
| filter matchesPhrase(log.source, "audit")
| filter matchesPhrase(content, "failed") and matchesPhrase(content, "login")
| fields timestamp, content
| sort timestamp desc
| limit 50
```

```dql
// Login activity by hour to identify patterns
fetch logs, from: now() - 7d
| filter matchesPhrase(log.source, "audit")
| filter matchesPhrase(content, "login")
| summarize logins = count(), by: { hour = bin(timestamp, 1h) }
| sort hour asc
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
// Off-hours login detection (outside 8am-6pm)
fetch logs, from: now() - 7d
| filter matchesPhrase(log.source, "audit")
| filter matchesPhrase(content, "login")
| fields timestamp, content
| filter getHour(timestamp) < 8 or getHour(timestamp) > 18
| sort timestamp desc
| limit 50
```

<a id="authorization-and-access-events"></a>
## 3. Authorization and Access Events
Track permission checks and access attempts to sensitive resources.

```dql
// Access denied events - indicates permission issues or attacks
fetch logs, from: now() - 7d
| filter matchesPhrase(log.source, "audit")
| filter matchesPhrase(content, "denied") or matchesPhrase(content, "forbidden") or matchesPhrase(content, "unauthorized")
| fields timestamp, content
| sort timestamp desc
| limit 100
```

```dql
// Permission changes and grants
fetch logs, from: now() - 30d
| filter matchesPhrase(log.source, "audit")
| filter matchesPhrase(content, "permission") or matchesPhrase(content, "grant") or matchesPhrase(content, "role")
| fields timestamp, content
| sort timestamp desc
| limit 100
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
Monitor all changes to IAM configuration.

```dql
// User creation and deletion events
fetch logs, from: now() - 30d
| filter matchesPhrase(log.source, "audit")
| filter matchesPhrase(content, "user") and (matchesPhrase(content, "created") or matchesPhrase(content, "deleted"))
| fields timestamp, content
| sort timestamp desc
| limit 100
```

```dql
// Group membership changes
fetch logs, from: now() - 30d
| filter matchesPhrase(log.source, "audit")
| filter matchesPhrase(content, "group") and (matchesPhrase(content, "added") or matchesPhrase(content, "removed") or matchesPhrase(content, "membership"))
| fields timestamp, content
| sort timestamp desc
| limit 100
```

```dql
// Policy changes - critical for compliance
fetch logs, from: now() - 30d
| filter matchesPhrase(log.source, "audit")
| filter matchesPhrase(content, "policy") and (matchesPhrase(content, "created") or matchesPhrase(content, "modified") or matchesPhrase(content, "deleted"))
| fields timestamp, content
| sort timestamp desc
| limit 100
```

```dql
// Token creation and revocation
fetch logs, from: now() - 30d
| filter matchesPhrase(log.source, "audit")
| filter matchesPhrase(content, "token") and (matchesPhrase(content, "created") or matchesPhrase(content, "revoked"))
| fields timestamp, content
| sort timestamp desc
| limit 100
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
// SOC2: User access review - all users who accessed system in period
fetch logs, from: now() - 90d
| filter matchesPhrase(log.source, "audit")
| filter matchesPhrase(content, "login")
| summarize last_login = max(timestamp), login_count = count(), by: { content }
| sort last_login desc
| limit 500
```

```dql
// SOC2: Administrative action log
fetch logs, from: now() - 90d
| filter matchesPhrase(log.source, "audit")
| filter matchesPhrase(content, "admin") or matchesPhrase(content, "created") or matchesPhrase(content, "deleted") or matchesPhrase(content, "modified")
| fields timestamp, content
| sort timestamp desc
| limit 500
```

### SOX - Segregation of Duties

SOX requires segregation of duties and change authorization.

```dql
// SOX: Privileged user activity (admin actions)
fetch logs, from: now() - 30d
| filter matchesPhrase(log.source, "audit")
| filter matchesPhrase(content, "admin")
| summarize action_count = count(), by: { content }
| sort action_count desc
| limit 100
```

```dql
// SOX: Configuration changes requiring approval
fetch logs, from: now() - 30d
| filter matchesPhrase(log.source, "audit")
| filter matchesPhrase(content, "policy") or matchesPhrase(content, "settings")
| filter matchesPhrase(content, "modified") or matchesPhrase(content, "created") or matchesPhrase(content, "deleted")
| fields timestamp, content
| sort timestamp desc
| limit 200
```

### HIPAA - PHI Access Logging

HIPAA requires logging of access to protected health information.

```dql
// HIPAA: All data access events for PHI-related resources
fetch logs, from: now() - 30d
| filter matchesPhrase(log.source, "audit")
| filter matchesPhrase(content, "read") or matchesPhrase(content, "access") or matchesPhrase(content, "query")
| fields timestamp, content
| sort timestamp desc
| limit 500
```

```dql
// HIPAA: Access anomalies - bulk data access detection
fetch logs, from: now() - 7d
| filter matchesPhrase(log.source, "audit")
| summarize event_count = count(), by: { hour = bin(timestamp, 1h), content }
| filter event_count > 100
| sort event_count desc
| limit 50
```

### PCI-DSS - Cardholder Data Access

PCI-DSS requires tracking access to cardholder data.

```dql
// PCI-DSS: Failed access attempts (potential attack indicator)
fetch logs, from: now() - 30d
| filter matchesPhrase(log.source, "audit")
| filter matchesPhrase(content, "denied") or matchesPhrase(content, "failed")
| summarize failure_count = count(), by: { day = bin(timestamp, 1d) }
| sort day desc
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
// Access review: Users who haven't logged in for 90 days
fetch logs, from: now() - 180d
| filter matchesPhrase(log.source, "audit")
| filter matchesPhrase(content, "login")
| summarize last_login = max(timestamp), by: { content }
| filter last_login < now() - 90d
| sort last_login asc
| limit 100
```

```dql
// Access review: High-activity users (potential over-privileged)
fetch logs, from: now() - 30d
| filter matchesPhrase(log.source, "audit")
| summarize action_count = count(), by: { content }
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
// Dashboard: Authentication trend (for line chart)
fetch logs, from: now() - 30d
| filter matchesPhrase(log.source, "audit")
| filter matchesPhrase(content, "login")
| summarize logins = count(), by: { day = bin(timestamp, 1d) }
| sort day asc
```

```dql
// Dashboard: Failed access by day (for bar chart)
fetch logs, from: now() - 30d
| filter matchesPhrase(log.source, "audit")
| filter matchesPhrase(content, "denied") or matchesPhrase(content, "failed")
| summarize failures = count(), by: { day = bin(timestamp, 1d) }
| sort day asc
```

```dql
// Dashboard: Recent administrative actions (for table)
fetch logs, from: now() - 7d
| filter matchesPhrase(log.source, "audit")
| filter matchesPhrase(content, "created") or matchesPhrase(content, "deleted") or matchesPhrase(content, "modified")
| fields timestamp, content
| sort timestamp desc
| limit 20
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

- [Audit Logs](https://docs.dynatrace.com/docs/manage/account-management/audit-logs)
- [DQL Reference](https://docs.dynatrace.com/docs/discover-dynatrace/references/dynatrace-query-language)
- [SOC2 Trust Principles](https://www.aicpa-cima.com/topic/audit-assurance/audit-and-assurance-greater-than-soc-2)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
