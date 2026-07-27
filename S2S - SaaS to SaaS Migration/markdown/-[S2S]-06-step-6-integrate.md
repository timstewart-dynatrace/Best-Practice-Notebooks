# S2S-06: Step 6 — Integrate: Cloud, Dashboards, and Workflows

> **Series:** S2S — SaaS to SaaS Migration | **Notebook:** 6 of 9 | **Phase:** Upgrade | **Step:** Integrate | **Created:** March 2026 | **Last Updated:** 07/24/2026

## Overview

With agents reporting to the target tenant and core configuration imported (Step 5), this step reconnects everything that depends on external systems: cloud integrations, dashboards, workflows, notification channels, synthetic monitors, and extensions. These integrations were deliberately deferred until agents were providing entity context — without entities, cloud metrics have no topology to attach to, dashboards show no data, and workflows have no triggers.

This step completes the Upgrade phase. After this step, the target tenant is fully operational and the migration enters the Run phase (Steps 7–9).

### S2S Migration Framework

| Phase | Steps | Focus |
|-------|-------|-------|
| Plan | 1. Discover → 2. Strategize → 3. Design | Understand what exists, choose approach, design target |
| **Upgrade** | 4. Prepare → 5. Execute → **6. Integrate** | Build target, move config, connect integrations |
| Run | 7. Expand → 8. Enable → 9. Optimize | Roll out agents, enable teams, tune and decommission |

> **You are here: Step 6 — Integrate.** Agents are reporting to the target tenant and core configuration is deployed (Step 5). Now you reconnect cloud integrations, migrate dashboards and workflows, and restore all external connections.

---

## Table of Contents

1. [Cloud Integration Migration](#cloud-integration-migration)
2. [Cloud Transformation Scenarios](#cloud-transformation-scenarios)
3. [Dashboard Migration](#dashboard-migration)
4. [Workflow Migration](#workflow-migration)
5. [Notification Integration Migration](#notification-integration-migration)
6. [Synthetic Monitor Migration](#synthetic-monitor-migration)
7. [Extension Migration](#extension-migration)
8. [Step Completion Checklist](#step-completion-checklist)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Step 5 Complete** | All agents reporting to target tenant, configuration imported, validation queries passing |
| **Cloud Provider Access** | AWS IAM admin, Azure AD admin, or GCP IAM admin for credential creation |
| **Target Tenant Access** | API token with `WriteConfig`, `settings.write`, `credentialVault.write` scopes |
| **Notification Channel Access** | Admin access to Slack, PagerDuty, ServiceNow, Teams, or email systems |
| **Synthetic Private Locations** | Network access from private location hosts to target tenant |
| **Extension Host** | Host-based ActiveGate available for Extensions 2.0 (not K8s-based) |

### Order of Operations

This notebook covers **operations 7–8** of the Upgrade phase:

| # | Operation | Section | Dependency |
|---|-----------|---------|------------|
| 1 | Provision target tenant and configure SSO/IAM | Step 4 | Step 3 design deliverables |
| 2 | Export configuration from source tenant | Step 4 | Source tenant access |
| 3 | Deploy ActiveGates and prepare K8s operators | Step 4 | Target tenant provisioned |
| 4 | Import configuration to target tenant | Step 5 | Operations 1–3 complete |
| 5 | Redirect agents to target tenant | Step 5 | Configuration imported |
| 6 | Validate data flow in target tenant | Step 5 | Agents redirected |
| **7** | **Reconnect cloud integrations** | Sections 1–2 | Agents reporting |
| **8** | **Migrate dashboards, workflows, and extensions** | Sections 3–7 | Integrations connected |

> **Bold** operations are covered in this notebook. Operations 1–6 were completed in Steps 4 and 5.

<a id="cloud-integration-migration"></a>
## 1. Cloud Integration Migration

Cloud integrations are tenant-specific. Credentials, IAM trust policies, and monitoring scopes must be recreated in the target tenant. Monaco can deploy cloud integration **configurations** but **cannot export credentials** — those must be recreated manually in each cloud provider.

### AWS Integration

| Component | Source | Target Action |
|-----------|--------|---------------|
| **IAM Role** | `arn:aws:iam::role/DynatraceMonitoring` | Create new role with target tenant external ID in trust policy |
| **CloudWatch metrics** | Configured per region | Recreate with same region scope |
| **Log Forwarder** | Lambda function forwarding CloudWatch Logs | Deploy new Lambda stack with target tenant ingest URL |
| **ActiveGate S3 extension** | S3 log ingestion via AG | Reconfigure AG extension with target tenant |

#### AWS Trust Policy Update

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::root"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "sts:ExternalId": "<target-tenant-external-id>"
        }
      }
    }
  ]
}
```

> **The external ID changes per tenant.** The target tenant generates a new external ID. Update the IAM role's trust policy with this new ID. The old external ID (source tenant) should be removed after migration validation.

### Azure Integration

| Component | Source | Target Action |
|-----------|--------|---------------|
| **App Registration** | Service principal in Azure AD | Create new App Registration or reuse existing with updated redirect URI |
| **Azure Monitor** | Subscription-scoped metrics | Configure same subscriptions in target tenant |
| **Event Hub** | Log forwarding via Event Hub consumer | Create new consumer group for target tenant ActiveGate |
| **Log Analytics** | Diagnostic settings | Update diagnostic settings to also forward to target (during transition) |

### GCP Integration

| Component | Source | Target Action |
|-----------|--------|---------------|
| **Service Account** | JSON key for Cloud Monitoring | Generate new key for same service account, or create new SA |
| **Pub/Sub** | Log forwarding via Pub/Sub subscription | Create new subscription for target tenant |
| **Cloud Monitoring** | Project-scoped metrics | Configure same projects in target tenant |

### Validate Cloud Integration Data

After configuring cloud integrations in the target tenant, verify data is flowing:

```dql
// Target tenant: check for detected problems indicating integration issues
fetch dt.davis.problems, from:-2h
| filter contains(event.name, "cloud") or contains(event.name, "integration") or contains(event.name, "AWS") or contains(event.name, "Azure") or contains(event.name, "GCP")
| fields display_id, event.name, event.status, event.category
| sort timestamp desc
| limit 10
```

```dql
// Target tenant: verify deployment events are being captured
fetch events, from:-24h
| filter event.kind == "CUSTOM_DEPLOYMENT"
| summarize deployment_count = count()
| fieldsAdd status = if(deployment_count > 0, then: "Deployment events flowing", else: "No deployment events — check CI/CD integration")
```

<a id="cloud-transformation-scenarios"></a>
## 2. Cloud Transformation Scenarios

When the SaaS-to-SaaS migration includes a cloud provider change (e.g., AWS to Azure), the integration work is significantly more complex. Not only do credentials change, but the entire monitoring stack changes.

### AWS → Azure Transformation

| Component | AWS (Source) | Azure (Target) | Migration Notes |
|-----------|-------------|----------------|------------------|
| **Compute monitoring** | CloudWatch metrics | Azure Monitor metrics | Different metric keys — dashboard queries must be rewritten |
| **Container platform** | EKS | AKS | DynaKube CR changes; K8s monitoring remains similar |
| **Log forwarding** | Lambda → Dynatrace | Event Hub → ActiveGate | New ingestion pipeline; different log format |
| **Identity** | IAM Role (AssumeRole) | Service Principal (OAuth) | Completely different auth model |
| **Load balancer** | ALB/NLB metrics | Azure LB / App Gateway | Different metric keys |
| **Serverless** | Lambda | Azure Functions | Different entity types and metrics |

### Azure → AWS Transformation

| Component | Azure (Source) | AWS (Target) | Migration Notes |
|-----------|---------------|-------------|------------------|
| **Compute monitoring** | Azure Monitor metrics | CloudWatch metrics | Different metric keys |
| **Container platform** | AKS | EKS | DynaKube CR changes |
| **Log forwarding** | Event Hub → ActiveGate | Lambda → Dynatrace | Deploy new Lambda forwarder stack |
| **Identity** | Service Principal (OAuth) | IAM Role (AssumeRole) | Create role with trust policy |

> **Dashboard impact:** Cloud transformation migrations require rewriting all cloud-specific dashboard queries. CloudWatch metric keys (e.g., `aws.ec2.cpu_utilization`) differ from Azure Monitor metric keys (e.g., `azure.vm.percentage_cpu`). Plan for dashboard recreation, not migration.

<a id="dashboard-migration"></a>
## 3. Dashboard Migration

Dashboards were imported in Step 5 as part of the Monaco deploy. This section covers the post-import remediation required to make dashboards functional.

### Dashboard Types and Migration Path

| Type | Monaco Type | Migration Path | Post-Import Work |
|------|------------|----------------|------------------|
| **Classic dashboards** | `api` (dashboard) | Monaco deploy | Entity ID remapping, ownership reset |
| **Gen3 dashboards** | `document` | Monaco deploy | Minimal — DQL queries are entity-independent |
| **Gen3 notebooks** | `document` | Monaco deploy | Minimal — DQL queries are entity-independent |

### Entity ID Remediation

Classic dashboards frequently contain hardcoded entity IDs in tile filters. These must be updated to reference entities in the target tenant:

| Pattern | Source | Target |
|---------|--------|--------|
| Tile entity filter | `entityId("HOST-1A2B3C4D")` | `tag("env:production")` |
| SLO reference | `sloId("slo-abc-123")` | New SLO ID from target tenant |
| Management zone filter | `mzId(12345)` | New MZ ID from target tenant |

> **Gen3 advantage:** Gen3 dashboards use DQL queries that reference data by field values (tags, names), not entity IDs. If you are migrating to Gen3, consider rebuilding classic dashboards as Gen3 documents instead of remapping entity IDs.

### Ownership Reset

Dashboards imported via Monaco are owned by the API token user. Reassign ownership to the appropriate teams:

| Dashboard Category | New Owner | Method |
|-------------------|-----------|--------|
| Platform overview | Platform team | Manual reassignment in UI |
| Application dashboards | App team leads | Manual reassignment in UI |
| Executive dashboards | Dashboard admin group | Manual reassignment in UI |
| Shared dashboards | Set to "shared" visibility | Bulk update via API |

<a id="workflow-migration"></a>
## 4. Workflow Migration

Workflows (Dynatrace Workflows, formerly AutomationEngine) require special handling because they contain actor permissions, triggers, and external integrations that are tenant-specific.

### Workflow Components

| Component | Portable? | Migration Notes |
|-----------|----------|------------------|
| **Workflow definition** | Yes | Export via Monaco (`automation`) or Terraform |
| **Trigger configuration** | Partial | Dynatrace Intelligence triggers reference entity IDs — update selectors |
| **Actor permissions** | No | Reassign actor (the identity that executes the workflow) in target tenant |
| **External connections** | No | Webhook URLs, API keys, OAuth tokens must be recreated |
| **JavaScript/Python actions** | Yes | Code is portable; external URLs may need updating |

### Terraform Export/Import

```bash
# Export workflows from source tenant
terraform import dynatrace_automation_workflow.my_workflow <workflow-id>

# Update configuration for target tenant
# - Change trigger entity selectors
# - Update webhook URLs
# - Reassign actor

# Apply to target tenant
terraform apply -var-file="target-tenant.tfvars"
```

### Trigger Reconfiguration

| Trigger Type | Migration Action |
|-------------|------------------|
| **detected problem** | Update entity selectors (replace hardcoded IDs with tags) |
| **Event** | Update event type filters if namespace/service names changed |
| **Schedule (cron)** | Port as-is — cron expressions are tenant-independent |
| **Manual** | Port as-is |

### Actor Permissions

Every workflow has an **actor** — the identity (user or service account) under which the workflow executes. The actor must have appropriate permissions in the target tenant:

```
Source actor: user@company.com (admin in source tenant)
Target actor: Same user, or dedicated service account
Required scopes: Depends on workflow actions (settings.write, events.ingest, etc.)
```

> **Best practice:** Use a dedicated **service account** as the workflow actor instead of a personal user account. Service accounts survive employee turnover and can be scoped precisely.

<a id="notification-integration-migration"></a>
## 5. Notification Integration Migration

Notification integrations connect Dynatrace alerting to external systems. Each integration type has different migration requirements.

### Integration Type Matrix

| Integration | Configuration Migrated via Monaco? | Credential Update Required? | Notes |
|------------|-----------------------------------|---------------------------|-------|
| **Email** | Yes | No | Recipient addresses are portable |
| **Slack** | Yes (structure) | Yes | Webhook URL must be updated to new channel/workspace |
| **Microsoft Teams** | Yes (structure) | Yes | Incoming webhook URL must be recreated |
| **PagerDuty** | Yes (structure) | Yes | Integration key must be updated |
| **ServiceNow** | Yes (structure) | Yes | Instance URL, credentials must be updated |
| **Webhook (generic)** | Yes (structure) | Yes | URL and auth headers must be verified |
| **Jira** | Yes (structure) | Yes | Project keys and credentials must be updated |
| **OpsGenie** | Yes (structure) | Yes | API key must be updated |

### Update Checklist

For each notification integration:

| Step | Action | Validation |
|------|--------|------------|
| 1 | Verify integration was imported by Monaco | Integration visible in target tenant UI |
| 2 | Update credentials/API keys | Store in credential vault |
| 3 | Update webhook URLs if environment-specific | Test URL is reachable |
| 4 | Send test notification | Verify delivery in target system |
| 5 | Validate alerting profile linkage | Profile references correct notification |

> **Test every notification channel.** Send a test alert from the target tenant to each notification integration. Do not assume that a successful import means the notification will work — credentials, URLs, and permissions may have changed.

<a id="synthetic-monitor-migration"></a>
## 6. Synthetic Monitor Migration

Synthetic monitors are imported via Monaco but require post-import validation for location assignments and credential references.

### Monitor Types

| Type | Monaco Migration | Post-Import Work |
|------|-----------------|------------------|
| **HTTP monitors** | Full export/import | Update credential vault references |
| **Browser monitors** | Full export/import | Verify script actions still work |
| **Browser click-path** | Full export/import | Re-record if target application URL changed |

### Location Assignment

| Location Type | Migration Notes |
|--------------|------------------|
| **Public locations** | Global — same location IDs across tenants. Port as-is. |
| **Private locations** | Tenant-specific. Deploy new private location AG in target tenant. |

### Private Location Migration

| Step | Action |
|------|--------|
| 1 | Create private synthetic location in target tenant UI |
| 2 | Deploy ActiveGate with synthetic capability at that location |
| 3 | Update monitor location assignments to use new location ID |
| 4 | Verify monitors execute from the new location |

### Validate Synthetic Monitors

After migration, verify synthetic monitors are executing in the target tenant:

```dql
// Target tenant: count active synthetic monitors
fetch dt.entity.synthetic_test
| summarize monitor_count = count()
| fieldsAdd validation = "Compare against source tenant synthetic monitor count"

// Smartscape note (dt.entity.* is deprecated but still functional): this entity type is
// not yet available on Grail Smartscape (smartscapeNodes has no equivalent node type),
// so keep the classic dt.entity.* query above.
```

<a id="extension-migration"></a>
## 7. Extension Migration

Extensions 2.0 provide monitoring for technologies not covered by OneAgent (databases, network devices, cloud APIs). They run on host-based ActiveGates.

### Extension Migration Steps

| Step | Action | Notes |
|------|--------|-------|
| 1 | Inventory extensions from source tenant | List all active extensions and their versions |
| 2 | Verify extension availability in target tenant | Extensions are published to the Dynatrace Hub — verify same version exists |
| 3 | Install extensions in target tenant | Via Hub or API |
| 4 | Configure monitoring settings | Recreate endpoint configurations |
| 5 | Assign to host-based ActiveGate | Must be host-based AG (not K8s-based) |
| 6 | Validate data collection | Check for new metrics from extension |

### Extension Types

| Type | Migration Path |
|------|---------------|
| **Hub extensions** | Install from Hub in target tenant — configuration is portable via Monaco |
| **Custom extensions** | Upload `.zip` to target tenant, recreate configuration |
| **JMX extensions** | Port JMX configuration file, update endpoint references |
| **SNMP extensions** | Port SNMP configuration, update device IP addresses if changed |

> **Host-based ActiveGate required.** Extensions 2.0 cannot run on Kubernetes-based ActiveGates. Ensure at least one host-based AG is deployed in the target tenant (as prepared in Step 4, Section 4).

<a id="step-completion-checklist"></a>
## 8. Step Completion Checklist

Before proceeding to Step 7 (Expand), verify all integration tasks are complete:

| Deliverable | Status | Owner | Notes |
|-------------|--------|-------|-------|
| **AWS integration configured** | ☐ | Cloud / Platform | IAM role trust policy updated, CloudWatch metrics flowing |
| **Azure integration configured** | ☐ | Cloud / Platform | App Registration created, Azure Monitor connected |
| **GCP integration configured** | ☐ | Cloud / Platform | Service account key generated, Cloud Monitoring connected |
| **Log forwarding active** | ☐ | Cloud / Platform | Lambda/Event Hub/Pub/Sub sending to target tenant |
| **Classic dashboards remediated** | ☐ | Platform | Entity IDs remapped, ownership reassigned |
| **Gen3 dashboards validated** | ☐ | Platform | DQL queries returning data |
| **Workflows migrated** | ☐ | Platform | Actors assigned, triggers reconfigured |
| **Notification integrations tested** | ☐ | Platform | Test alert sent to each channel |
| **Synthetic monitors executing** | ☐ | Platform | All monitors running from correct locations |
| **Extensions installed and collecting** | ☐ | Platform | Extension metrics visible in target tenant |
| **No critical detected problems** | ☐ | Platform | All migration-related problems resolved or documented |

> **Phase transition:** Completing this checklist ends the **Upgrade** phase. The target tenant is now fully operational. The **Run** phase (Steps 7–9) focuses on expanding agent coverage, enabling teams, and optimizing the target tenant.

---

## Next Step

Continue to **S2S-07: Step 7 — Expand: Agent Rollout and Coverage** to expand agent coverage to remaining hosts, enable team access, and begin the Run phase.

| Completed | Next | Remaining |
|-----------|------|-----------|
| ~~1. Discover~~ → ~~2. Strategize~~ → ~~3. Design~~ → ~~4. Prepare~~ → ~~5. Execute~~ → ~~6. Integrate~~ | **7. Expand** | 8. Enable → 9. Optimize |

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
