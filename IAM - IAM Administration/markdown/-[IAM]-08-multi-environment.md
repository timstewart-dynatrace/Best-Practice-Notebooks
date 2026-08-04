# IAM-08: Multi-Environment IAM

> **Series:** IAM — IAM Administration | **Notebook:** 8 of 12 | **Created:** January 2026 | **Last Updated:** 08/03/2026

## Scaling IAM Across Multiple Environments
Enterprise organizations typically have multiple Dynatrace environments (development, staging, production). This notebook covers strategies for managing IAM consistently across environments while maintaining security boundaries.

---

## Table of Contents

1. [Multi-Environment Architecture](#multi-environment-architecture)
2. [Account-Level vs Environment-Level Permissions](#account-level-vs-environment-level-permissions)
3. [Centralized vs Federated Models](#centralized-vs-federated-models)
4. [Cross-Environment Access Patterns](#cross-environment-access-patterns)
5. [Break-Glass Procedures](#break-glass-procedures)
6. [Synchronizing IAM Configuration](#synchronizing-iam-configuration)
7. [Multi-Tenant Considerations](#multi-tenant-considerations)
8. [Monitoring Multi-Environment IAM](#monitoring-multi-environment-iam)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Dynatrace Environments** | Multiple environments under one account |
| **Permissions** | Account-level IAM admin |
| **Knowledge** | IAM-01 through 07 completed |

<a id="multi-environment-architecture"></a>
## 1. Multi-Environment Architecture
Understanding the relationship between accounts and environments is foundational.

### Account and Environment Hierarchy

```
Dynatrace Account (Organization Level)
├── Environment: Production
│   ├── Entities, Logs, Traces
│   └── Environment-specific settings
├── Environment: Staging
│   ├── Entities, Logs, Traces
│   └── Environment-specific settings
└── Environment: Development
    ├── Entities, Logs, Traces
    └── Environment-specific settings
```

### Permission Scopes

| Scope | What It Controls | Example |
|-------|-----------------|----------|
| **Account** | Users, groups, SSO, billing | Create users, manage IdP |
| **Environment** | Data access, settings, configurations | Read logs, modify alerting |
| **Segment** | Subset of environment data | Access only prod-frontend data |

### Common Environment Patterns

| Pattern | Environments | Use Case |
|---------|--------------|----------|
| **Standard** | Dev, Stage, Prod | Software development |
| **Regional** | US, EU, APAC | Geographic compliance |
| **BU-based** | Retail, Finance, HR | Business unit isolation |
| **Tenant** | Customer A, B, C | MSP/multi-tenant |

<a id="account-level-vs-environment-level-permissions"></a>
## 2. Account-Level vs Environment-Level Permissions
Understanding where permissions are managed is critical for proper access control.

### Account-Level Permissions

Managed in Account Management, apply across all environments:

| Permission | Description |
|------------|-------------|
| `account-iam-admin` | Full IAM administration |
| `account-viewer` | View account information |
| `account-company-admin` | Manage account settings |
| `account-env-admin` | Create/manage environments |

### Environment-Level Permissions

Scoped to specific environments:

| Permission | Description |
|------------|-------------|
| `storage:logs:read` | Read logs in environment |
| `storage:metrics:read` | Read metrics |
| `settings:objects:read` | Read configuration |
| `settings:objects:write` | Modify configuration |

### Permission Assignment Matrix

| Role | Account Perms | Prod | Stage | Dev |
|------|---------------|------|-------|-----|
| IAM Admin | `account-iam-admin` | - | - | - |
| Prod Viewer | - | Read | - | - |
| Prod Editor | - | Read/Write | - | - |
| Dev Team | - | Read | Read/Write | Full |
| Platform Team | - | Full | Full | Full |

<a id="centralized-vs-federated-models"></a>
## 3. Centralized vs Federated Models
Choose a governance model based on organization structure.

### Centralized Model

Single IAM team manages all access.

| Aspect | Approach |
|--------|----------|
| Group creation | Central team only |
| Policy authoring | Central team only |
| Access requests | Ticketing system |
| Approvals | Central team |

**Pros:**
- Consistent standards
- Single point of control
- Easier compliance

**Cons:**
- Bottleneck for requests
- Slower response time
- Less team autonomy

### Federated Model

Delegated administration to teams.

| Aspect | Approach |
|--------|----------|
| Group creation | Team leads (within namespace) |
| Policy authoring | Central team defines templates |
| Access requests | Direct to team lead |
| Approvals | Team lead + central audit |

**Pros:**
- Faster access provisioning
- Team ownership
- Scales with organization

**Cons:**
- Requires governance framework
- Risk of inconsistency
- More complex auditing

### Hybrid Model (Recommended)

| Responsibility | Owner |
|----------------|-------|
| Account-level IAM | Central team |
| Production access | Central team |
| Non-prod access | Team leads |
| Policy templates | Central team |
| Custom policies | Team leads (with approval) |

<a id="cross-environment-access-patterns"></a>
## 4. Cross-Environment Access Patterns
Define how users access multiple environments.

### Pattern 1: Environment-Specific Groups

Separate groups per environment.

| Group | Environment | Permissions |
|-------|-------------|-------------|
| `dt-prod-viewers` | Production | Read-only |
| `dt-stage-editors` | Staging | Read/Write |
| `dt-dev-admins` | Development | Full |

**Use when:** Strict environment isolation required.

### Pattern 2: Role-Based Groups

Same role, different environment permissions.

| Group | Prod | Stage | Dev |
|-------|------|-------|-----|
| `dt-sre-team` | Editor | Editor | Editor |
| `dt-dev-team` | Viewer | Editor | Admin |
| `dt-security-team` | Viewer | Viewer | Viewer |

**Use when:** Consistent role across environments with varying access levels.

### Pattern 3: Escalation-Based Access

Production access requires explicit escalation.

| Group | Normal Access | Escalated Access |
|-------|---------------|------------------|
| `dt-oncall` | Stage/Dev | Production (during incident) |
| `dt-deploy` | Stage | Production (during deploy window) |

**Use when:** Production access is exception, not default.

<a id="break-glass-procedures"></a>
## 5. Break-Glass Procedures
Emergency access when normal procedures can't be followed.

### What is Break-Glass?

Emergency access mechanism for critical incidents when:
- Normal approvers unavailable
- SSO/IdP is down
- Critical incident requires immediate elevated access

### Break-Glass Components

| Component | Purpose |
|-----------|----------|
| Emergency account | Local account (not SSO) |
| Elevated permissions | Admin or full access |
| Secure storage | Password vault, split secrets |
| Audit logging | All actions logged |
| Post-incident review | Mandatory after use |

### Break-Glass Setup

1. **Create emergency account**
   - Local Dynatrace account (not SSO-dependent)
   - Strong, complex password
   - MFA with hardware token

2. **Assign minimal necessary permissions**
   - Typically `account-iam-admin` + environment admin
   - Or scoped to specific emergency needs

3. **Secure the credentials**
   - Store in enterprise vault (CyberArk, HashiCorp)
   - Or use split-knowledge (2+ people needed)
   - Document retrieval procedure

4. **Define usage criteria**
   - When break-glass can be used
   - Who can authorize
   - Notification requirements

5. **Establish post-incident process**
   - Rotate credentials after each use
   - Review all actions taken
   - Document in incident report

### Break-Glass Policy Template

```
BREAK-GLASS ACCESS POLICY

Authorized Situations:
□ P1/P2 incident AND normal approvers unavailable
□ SSO/IdP outage preventing authentication
□ Security incident requiring immediate response

NOT Authorized:
□ Convenience (approver busy but reachable)
□ Non-urgent requests
□ Bypassing normal approval process

Usage Steps:
1. Attempt normal access/approval first
2. Document why break-glass is needed
3. Notify security team
4. Retrieve credentials from vault
5. Perform necessary actions only
6. Log out immediately when done
7. Notify IAM team for credential rotation
8. Complete post-incident review within 24h
```

<a id="synchronizing-iam-configuration"></a>
## 6. Synchronizing IAM Configuration
Keep IAM configuration consistent across environments.

### Configuration as Code

Store IAM configuration in version control:

```
iam-config/
├── groups/
│   ├── platform-team.yaml
│   ├── dev-team.yaml
│   └── security-team.yaml
├── policies/
│   ├── environment-viewer.yaml
│   ├── environment-editor.yaml
│   └── log-reader.yaml
└── boundaries/
    ├── production-boundary.yaml
    └── sensitive-data-boundary.yaml
```

### Synchronization Approaches

| Approach | Pros | Cons |
|----------|------|------|
| **Manual** | Simple | Error-prone, doesn't scale |
| **Terraform** | Version controlled, repeatable | Learning curve |
| **Dynatrace API** | Direct control | Custom scripting needed |
| **Monaco** | Dynatrace-specific | Configuration complexity |

### Change Management Process

| Step | Action | Approval |
|------|--------|----------|
| 1 | Create PR with IAM change | Developer |
| 2 | Review configuration | IAM team |
| 3 | Apply to dev environment | Automated |
| 4 | Test in dev | Developer |
| 5 | Apply to staging | Automated |
| 6 | Validate in staging | IAM team |
| 7 | Apply to production | Manual approval + deploy |

<a id="multi-tenant-considerations"></a>
## 7. Multi-Tenant Considerations
Special considerations for MSPs and multi-tenant deployments.

### Multi-Tenant Architecture

| Model | Isolation Level | Use Case |
|-------|-----------------|----------|
| **Separate Accounts** | Highest | Different legal entities |
| **Separate Environments** | High | Business unit isolation |
| **Boundaries within Environment** | Medium | Team isolation |
| **Segments only** | Lower | Data filtering only |

### Tenant Isolation Checklist

```
□ Data cannot be accessed across tenants
□ Users cannot see other tenants exist
□ Configuration cannot impact other tenants
□ Audit logs are tenant-specific
□ Billing is tenant-specific
```

### MSP Considerations

| Aspect | Recommendation |
|--------|----------------|
| Customer environments | Separate environment per customer |
| MSP admin access | Dedicated MSP admin group |
| Customer admin access | Scoped to their environment only |
| Audit | Separate audit logs per customer |
| Compliance | Document isolation in SOC2 |

<a id="monitoring-multi-environment-iam"></a>
## Monitoring Multi-Environment IAM
Query patterns for multi-environment oversight.

```dql
// Cross-environment access attempts
fetch logs, from: now() - 7d
| filter matchesPhrase(log.source, "audit")
| filter matchesPhrase(content, "environment")
| fields timestamp, content
| sort timestamp desc
| limit 100
```

```dql
// Account-level administrative actions
fetch logs, from: now() - 30d
| filter matchesPhrase(log.source, "audit")
| filter matchesPhrase(content, "account") and (matchesPhrase(content, "admin") or matchesPhrase(content, "created") or matchesPhrase(content, "modified"))
| fields timestamp, content
| sort timestamp desc
| limit 100
```

```dql
// Break-glass or emergency account usage
fetch logs, from: now() - 90d
| filter matchesPhrase(log.source, "audit")
| filter matchesPhrase(content, "emergency") or matchesPhrase(content, "break-glass") or matchesPhrase(content, "breakglass")
| fields timestamp, content
| sort timestamp desc
| limit 50
```

## Next Steps

With multi-environment IAM understood, proceed to troubleshooting:

### Recommended Path

1. **IAM-09: Troubleshooting Access Issues** - Debug permission problems systematically

### Multi-Environment Checklist

Before moving on, ensure you have:

- [ ] Documented all environments and their purposes
- [ ] Defined governance model (centralized, federated, hybrid)
- [ ] Created environment-specific access groups
- [ ] Established break-glass procedure
- [ ] Implemented IAM configuration as code
- [ ] Documented multi-tenant isolation (if applicable)

---

## Summary

In this notebook, you learned:

- Multi-environment architecture and hierarchy
- Account-level vs environment-level permissions
- Centralized, federated, and hybrid governance models
- Cross-environment access patterns
- Break-glass procedures for emergency access
- IAM configuration synchronization approaches
- Multi-tenant considerations for MSPs

---

## References

- [Account Management](https://docs.dynatrace.com/docs/manage/account-management)
- [Account Management (DT docs)](https://docs.dynatrace.com/docs/manage/account-management)
- [Dynatrace Terraform Provider](https://registry.terraform.io/providers/dynatrace-oss/dynatrace/latest/docs)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
