# ORGNZ-07: Advanced Permission Patterns

> **Series:** ORGNZ — Organize Data: Buckets, Segments, Security | **Notebook:** 7 of 10 | **Created:** January 2026 | **Last Updated:** 09/24/2026

## Overview

This notebook covers advanced permission patterns including record-level permissions, field-based access, and combining multiple access control mechanisms for enterprise-scale data governance.

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Dynatrace Environment** | SaaS environment with Grail enabled |
| **Permissions** | Account admin or IAM policy management access |
| **Knowledge** | Completed ORGNZ-06 (Security Context) |
| **Data** | At least 1 hour of log data |

---

## Table of Contents

1. [Record-Level Permissions](#record-level-permissions)
2. [Record-Level Policy Examples](#record-level-policy-examples)
3. [Field-Level Access](#field-level-access)
4. [Combined Permission Patterns](#combined-permission-patterns)
5. [Policy Boundaries](#policy-boundaries)
6. [Enterprise Architecture Patterns](#enterprise-architecture-patterns)
7. [Testing Permissions](#testing-permissions)
8. [Best Practices](#best-practices)

---

## Learning Objectives

By the end of this notebook, you will:
- Implement record-level permissions using various attributes
- Understand field-level access restrictions
- Use policy boundaries for reusable condition management
- Combine bucket, record, and field-level policies
- Design enterprise-scale permission architectures

<a id="record-level-permissions"></a>
## Record-Level Permissions
Record-level permissions filter data at query time based on record attributes:

![Record-Level Permissions Flow](images/07-record-level-permissions.png)

<!-- MARKDOWN_TABLE_ALTERNATIVE
| Step | Action |
|------|--------|
| 1 | User Query: fetch logs |
| 2 | IAM Policy Evaluation |
| 3 | Record Filter (namespace, host group, security context) |
| 4 | Only authorized records returned |
For environments where SVG doesn't render
-->

### Supported Record-Level Conditions

| Condition | Description | Example | Tables it applies to |
|-----------|-------------|---------|----------------------|
| `storage:k8s.namespace.name` | Kubernetes namespace | `= 'production'` | events, security.events, bizevents, logs, metrics, spans, smartscape |
| `storage:k8s.cluster.name` | Kubernetes cluster | `= 'main-cluster'` | events, security.events, bizevents, logs, metrics, spans, smartscape |
| `storage:host.name` | Host name | `= 'web-server-01'` | events, security.events, bizevents, logs, metrics, spans, smartscape |
| `storage:dt.host_group.id` | Host group | `STARTSWITH 'prod-'` | events, security.events, bizevents, logs, metrics, spans, smartscape |
| `storage:aws.account.id` | AWS account | `= '123456789012'` | events, security.events, bizevents, logs, metrics, spans, smartscape |
| `storage:gcp.project.id` | GCP project | `= 'my-project'` | events, security.events, bizevents, logs, metrics, spans, smartscape |
| `storage:azure.subscription` | Azure subscription | `= 'sub-id'` | events, security.events, bizevents, logs, metrics, spans, smartscape |
| `storage:azure.resource.group` | Azure resource group | `= 'my-rg'` | events, security.events, bizevents, logs, metrics, spans, smartscape |
| `storage:dt.security_context` | Custom context | `MATCH ('team-*')` | events, security.events, bizevents, system, logs, metrics, spans, entities, smartscape, user.events, user.sessions |
| `storage:log.source` | Log source | `= '/var/log/audit/audit.log'` | logs |
| `storage:metric.key` | Metric key | `STARTSWITH 'dt.host.'` | metrics |
| `storage:event.kind` | Event kind | `= 'DAVIS_PROBLEM'` | events, security.events, bizevents, system |
| `storage:event.type` | Event type | `= 'CUSTOM_INFO'` | events, security.events, bizevents, system |
| `storage:event.provider` | Event provider | `= 'my-app'` | events, security.events, bizevents, system |
| `storage:frontend.name` | Frontend (RUM application) name | `= 'www.example.com'` | user.events, user.sessions, metrics, smartscape |

A condition only restricts the tables listed for it. `storage:log.source` is the natural condition for audit-log access — it scopes `storage:logs:read` to named log sources without a dedicated bucket.

> <sub>**Sources:** [Permissions in Grail (DT docs)](https://docs.dynatrace.com/docs/platform/grail/organize-data/assign-permissions-in-grail) — supported record-level fields and the tables each applies to, read 09/24/2026.</sub>

<a id="record-level-policy-examples"></a>
## Record-Level Policy Examples
### Kubernetes Namespace Isolation

```json
{
  "name": "production-namespace-access",
  "description": "Access only to production namespace data",
  "statementQuery": "ALLOW storage:buckets:read WHERE storage:bucket-name STARTSWITH 'default_'; ALLOW storage:logs:read WHERE storage:k8s.namespace.name = 'production';"
}
```

### Host Group Based Access

```json
{
  "name": "web-tier-access",
  "description": "Access to web tier hosts only",
  "statementQuery": "ALLOW storage:buckets:read WHERE storage:bucket-name STARTSWITH 'default_'; ALLOW storage:logs:read, storage:metrics:read WHERE storage:dt.host_group.id STARTSWITH 'web-';"
}
```

### Combined Namespace and Host Group

```json
{
  "name": "prod-web-tier-access",
  "description": "Production web tier only",
  "statementQuery": "ALLOW storage:buckets:read WHERE storage:bucket-name STARTSWITH 'default_'; ALLOW storage:logs:read WHERE storage:k8s.namespace.name = 'production' AND storage:dt.host_group.id STARTSWITH 'web-';"
}
```

### Cloud Account Isolation

```json
{
  "name": "aws-team-account-access",
  "description": "Access to specific AWS account",
  "statementQuery": "ALLOW storage:buckets:read WHERE storage:bucket-name STARTSWITH 'default_'; ALLOW storage:logs:read, storage:metrics:read, storage:spans:read WHERE storage:aws.account.id = '123456789012';"
}
```

<a id="field-level-access"></a>
## Field-Level Access
Field-level access in Grail uses **fieldsets**, not per-field permissions. A fieldset is a named group of sensitive fields, and reading those fields requires `storage:fieldsets:read` on that fieldset. A user without it does not get masked values — the fields are left out: *"If you don't have sufficient permissions, sensitive fields won't be shown in the result."*

> **Corrected 09/24/2026.** Earlier versions of this notebook showed `DENY storage:logs:read:user.email` statements. There is no per-field permission suffix, and a policy written that way will not validate.

| Fieldset | Covers |
|----------|--------|
| `builtin-sensitive-spans` | Span fields considered sensitive |
| `builtin-request-attributes-spans` | Span fields holding request-attribute data marked sensitive |
| `builtin-sensitive-user-events-and-sessions` | Sensitive fields in `user.events` and `user.sessions` |
| Custom fieldset | Fields you name, scoped to buckets or tables |

*"The predefined fieldsets apply to spans, user.events and user.sessions only. They don't apply to logs or events."* For log fields such as `user.email`, define a custom fieldset: *"You can define your custom fieldsets, and to which scope they apply (either buckets, or tables, otherwise all buckets and tables)."*

### Fieldset Policy Example

```json
{
  "name": "sensitive-span-fields",
  "description": "Allow reading the built-in sensitive span fields",
  "statementQuery": "ALLOW storage:fieldsets:read WHERE storage:fieldset-name = \"builtin-sensitive-spans\";"
}
```

Grant this only to groups that need the fields; everyone else keeps `storage:spans:read` and does not see them. To mask PII *values* rather than hide whole fields, mask at capture or ingest (OneAgent or OpenPipeline masking) — see **OPLOGS-08**.

> <sub>**Sources:** [Permissions in Grail (DT docs)](https://docs.dynatrace.com/docs/platform/grail/organize-data/assign-permissions-in-grail).</sub>

<a id="combined-permission-patterns"></a>
## Combined Permission Patterns
### Pattern 1: Layered Access Control

Combine bucket and record-level permissions:

```
Layer 1: Bucket access
  ALLOW storage:buckets:read WHERE bucket-name IN ('team_logs', 'shared_logs')

Layer 2: Record filtering
  ALLOW storage:logs:read WHERE k8s.namespace.name = 'team-namespace'

Layer 3: Sensitive fields (optional, only for groups that need them)
  ALLOW storage:fieldsets:read WHERE storage:fieldset-name = "<fieldset-name>"
```

### Pattern 2: Multi-Team Shared Bucket

Multiple teams share a bucket with security context isolation:

```json
// Team A policy
{
  "name": "team-a-shared-bucket",
  "statementQuery": "ALLOW storage:buckets:read WHERE storage:bucket-name = 'shared_logs'; ALLOW storage:logs:read WHERE storage:dt.security_context MATCH ('team-a*');"
}

// Team B policy
{
  "name": "team-b-shared-bucket",
  "statementQuery": "ALLOW storage:buckets:read WHERE storage:bucket-name = 'shared_logs'; ALLOW storage:logs:read WHERE storage:dt.security_context MATCH ('team-b*');"
}
```

### Pattern 3: Environment-Based Tiering

```json
// Production access (restricted)
{
  "name": "production-access",
  "statementQuery": "ALLOW storage:buckets:read WHERE storage:bucket-name STARTSWITH 'prod_'; ALLOW storage:logs:read WHERE storage:dt.security_context = 'env:production';"
}

// Non-production access (broader)
{
  "name": "non-production-access",
  "statementQuery": "ALLOW storage:buckets:read WHERE storage:bucket-name STARTSWITH 'default_'; ALLOW storage:logs:read WHERE storage:dt.security_context MATCH ('env:dev*', 'env:staging*', 'env:qa*');"
}
```

### Pattern 4: Multi-Dimensional MATCH() for Transversal Teams

When teams need access by component type (database, networking, OS) across multiple applications, encode multiple dimensions into the security context and use MATCH() to slice across application boundaries:

```json
// Database team: all DB-layer components across all applications (Grail)
{
  "name": "db-team-grail-access",
  "statementQuery": "ALLOW storage:buckets:read WHERE storage:bucket-name STARTSWITH 'default_'; ALLOW storage:logs:read WHERE storage:dt.security_context MATCH ('comp:db*');"
}

// Application team: all component layers for one application (Grail)
{
  "name": "easytrade-team-access",
  "statementQuery": "ALLOW storage:buckets:read WHERE storage:bucket-name STARTSWITH 'default_'; ALLOW storage:logs:read WHERE storage:dt.security_context MATCH ('*/app:easytrade');"
}
```

> MATCH() supports `*` wildcards at any position and is **storage domain only**. For Classic entity access (hosts, services), use `startsWith` per component type. The context format is `comp:<component>/bu:<business-unit>/app:<application>` — place the transversal dimension first so `startsWith` cuts correctly. See **IAM-05: Boundary Design** for the full two-domain pattern.

<a id="policy-boundaries"></a>
## Policy Boundaries

**Policy boundaries** decouple the "what" (policy) from the "where" (conditions), enabling reusable condition sets that can be applied across multiple policies.

### What Are Boundaries?

| Aspect | Description |
|--------|-------------|
| **Purpose** | Bundle conditions for reuse across multiple policies |
| **Scope** | Record-level and resource-level restrictions |
| **Relationship** | Always used together with a policy — boundaries alone don't restrict anything |
| **Limit** | Maximum 10 restrictions per boundary |

### How Boundaries Work

While **policies** define _which features and data_ users can access, **boundaries** define _where_ users can access them:

```
Policy: ALLOW storage:logs:read, storage:metrics:read, storage:spans:read
Boundary: storage:k8s.namespace.name = "production"
           storage:dt.host_group.id STARTSWITH "prod-"
```

When assigned together, the user gets read access to logs, metrics, and spans — but only for production namespace data on production host groups.

### Boundary Rules

| Rule | Detail |
|------|--------|
| No AND operator | Each line is one condition (implicitly AND-combined) |
| No logical operators | For complex logic, use policy templating |
| Reusable | Same boundary can be applied to multiple policies |
| Max 10 restrictions | Create additional boundaries if more are needed |

### Creating Boundaries

1. Go to **Account Management** > **Identity & access management** > **Policy management**
2. Select the **Boundaries** tab
3. Select **Create boundary**
4. Enter boundary name and conditions (one per line)
5. Save and assign to group policies

### Example: Regional Boundary

```
# EU Region Boundary
storage:bucket-name STARTSWITH "eu_"
storage:azure.subscription = "eu-subscription-id"
```

This boundary can then be applied alongside any policy (logs access, metrics access, etc.) to restrict all data access to the EU region.

> **Tip:** Use boundaries when you have the same set of conditions (e.g., "production only" or "EU region only") that need to apply to multiple policies. This avoids duplicating conditions across every policy definition.

<a id="enterprise-architecture-patterns"></a>
## Enterprise Architecture Patterns
### Tiered Access Model

![Tiered Access Model](images/07-tiered-access-model.png)

<!-- MARKDOWN_TABLE_ALTERNATIVE
| Tier | Role | Access |
|------|------|--------|
| 1 | Platform Admins | All buckets, all data types, all records |
| 2 | Team Leads | Default + team buckets, team context |
| 3 | Developers | Default buckets, own namespace only |
| 4 | Read-Only Viewers | Default buckets, public context only |
For environments where SVG doesn't render
-->

### Geographic/Regulatory Model

| Region | Buckets | Context | Policy |
|--------|---------|---------|--------|
| EU | eu_* | region:eu | bucket-name STARTSWITH 'eu_' AND security_context MATCH ('region:eu*') |
| US | us_* | region:us | bucket-name STARTSWITH 'us_' AND security_context MATCH ('region:us*') |

> **Tip:** Use policy boundaries to define regional restrictions once, then apply the same boundary to all team policies within that region.

<a id="testing-permissions"></a>
## Testing Permissions

### DQL: Access Distribution Audit

Measure coverage of key access control fields across your log data:

```dql
// Verify accessible data distribution across all key access control dimensions
fetch logs, from:-1h
| summarize
    total = count(),
    withSecurityContext = countIf(isNotNull(dt.security_context)),
    withNamespace = countIf(isNotNull(k8s.namespace.name)),
    withHostGroup = countIf(isNotNull(dt.host_group.id))
```

<a id="best-practices"></a>
## Best Practices
| Practice | Rationale |
|----------|----------|
| Start with least privilege | Grant minimum required access |
| Use groups, not individuals | Easier management, consistent access |
| Use policy boundaries for reuse | Avoid duplicating conditions across policies |
| Document all policies | Audit trail and governance |
| Test policies before deployment | Prevent access issues |
| Regular access reviews | Remove unnecessary permissions |
| Use MATCH for array fields | Required for correct evaluation |
| Combine bucket + record level | Defense in depth |

## Next Steps

Continue with the ORGNZ series:
- **ORGNZ-08**: Grail Segments

## References

- [Configure advanced permissions with security context](https://docs.dynatrace.com/docs/platform/grail/organize-data/advanced-permission-setup)
- [Policy boundaries](https://docs.dynatrace.com/docs/manage/identity-access-management/permission-management/manage-user-permissions-policies/iam-policy-boundaries)
- [IAM policy reference](https://docs.dynatrace.com/docs/manage/identity-access-management/permission-management/manage-user-permissions-policies/advanced/iam-policystatements)
- [Enhance data management with Grail](https://www.dynatrace.com/news/blog/enhance-data-management-with-grail-ultimate-guide-to-custom-buckets-and-security-policies/)

---

<sub>*This notebook was AI-generated from Dynatrace documentation and enterprise best practices. It is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
