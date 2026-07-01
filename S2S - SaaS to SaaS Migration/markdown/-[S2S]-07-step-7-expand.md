# S2S-07: Step 7 — Expand: OpenPipeline, SLOs, and Alerting

> **Series:** S2S — SaaS to SaaS Migration | **Notebook:** 7 of 10 | **Phase:** Run | **Step:** Expand | **Created:** March 2026 | **Last Updated:** 07/01/2026

## Overview

With agents reporting to the target tenant (Step 5) and cloud integrations reconnected (Step 6), this step expands monitoring coverage to include data processing pipelines, service-level objectives, and alerting. OpenPipeline rules control how data is enriched, routed, and masked. SLOs define reliability targets. Alerting ensures the right people are notified when those targets are at risk. All three must be migrated in a coordinated sequence — buckets first, then pipelines, then SLOs and alerts that consume the processed data.

> **S2S Migration Journey — 3 Phases / 9 Steps**
>
> **Plan:** 1. Discover | 2. Strategize | 3. Design
>
> **Upgrade:** 4. Prepare | 5. Execute | 6. Integrate
>
> **Run:** **7. Expand** | 8. Enable | 9. Optimize

---

## Table of Contents

1. [OpenPipeline Rule Migration](#openpipeline-rule-migration)
2. [Grail Bucket Configuration](#grail-bucket-configuration)
3. [SLO Migration](#slo-migration)
4. [Alerting and Notification Migration](#alerting-and-notification-migration)
5. [Data Retention Optimization](#data-retention-optimization)
6. [Step Completion Checklist](#step-completion-checklist)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Step 5 Complete** | OneAgents and DynaKube operators redirected to target tenant and reporting data |
| **Step 6 Complete** | Cloud integrations (AWS, Azure, GCP) reconnected in target tenant |
| **Source Tenant Access** | Admin access with `settings.read`, `openpipeline.read` scopes |
| **Target Tenant Access** | Admin access with `settings.write`, `openpipeline.write`, `slo.write` scopes |
| **Monaco CLI** | v2.x installed for bulk export/import of OpenPipeline and SLO configuration |
| **Terraform** | v1.5+ with Dynatrace provider (for IAM-gated SLO policies only) |

<a id="openpipeline-rule-migration"></a>

## 1. OpenPipeline Rule Migration

OpenPipeline processing rules define how ingested data is enriched, extracted, routed, and masked before it lands in Grail. These rules must be migrated **after** Grail buckets exist in the target tenant (because routing rules reference bucket names) and **before** SLOs and alerts (because they consume processed data).

### Processing Rule Types

| Rule Type | What It Does | Migration Notes |
|-----------|-------------|------------------|
| **Enrichment** | Add fields from lookup tables or entity properties | Port as-is if field names match |
| **Extraction** | Parse structured data from unstructured content (DPL patterns) | Port as-is |
| **Routing** | Direct data to specific Grail buckets based on conditions | **Update bucket references** to match target bucket names |
| **Masking** | Redact sensitive data (PII, credentials, tokens) | Port as-is — verify compliance requirements match |

### Enrichment Propagation Delay

> **Warning:** On Kubernetes clusters (especially EKS), enrichment rules typically take ~45 minutes to propagate cluster-wide after deployment (observed; actual time varies by cluster size and ActiveGate configuration). Routing rules that depend on enriched attributes (e.g., `app.namespace`, `k8s.cluster.name`) will not match incoming data until propagation completes — data lands in `default_logs` during the gap.
>
> **Mitigation:** If deploying via Terraform or CI/CD, add a validation gate between enrichment deployment and routing deployment. Either use a fixed wait (45-60 min on EKS) or query for enriched fields before proceeding:
> ```
> fetch logs, from:-5m | filter isNotNull(app.namespace) | summarize count()
> ```
> If count > 0, enrichment is propagated and routing rules can be deployed safely.

### Deployment Order (Critical)

OpenPipeline and related resources must be deployed in strict dependency order:

| Order | Resource | Why This Order |
|-------|----------|----------------|
| 1 | Grail buckets | Routing rules reference buckets — they must exist first |
| 2 | K8s enrichment rules | Tags feed OpenPipeline conditions and downstream queries |
| — | *Wait for enrichment propagation (~45 min on K8s)* | *Routing rules fail silently if enriched fields are not yet available* |
| 3 | OpenPipeline processing rules | Now routing targets and enrichment sources exist |
| 4 | Segments | Segment definitions may reference buckets and tags |

> **Monaco note:** `monaco deploy` resolves these dependencies automatically within a single run. However, it does **not** wait for enrichment propagation — if you deploy buckets, enrichment, and routing in one `monaco deploy`, routing rules may not match incoming data for ~45 minutes. For production migrations, consider splitting the deploy into two runs with a validation gate between them.

### Coordinated Deployment Strategy

For consolidation scenarios where multiple source tenants have different OpenPipeline rules:

| Approach | When to Use | Risk |
|----------|-------------|------|
| **Merge** | Rules from both tenants are complementary (different data sources) | Low — rules do not conflict |
| **Prioritize** | Rules overlap — choose the more mature or comprehensive set | Medium — requires rule-by-rule comparison |
| **Redesign** | Rules from both tenants are outdated or inefficient | High effort, best long-term result |

### Export and Import with Monaco

```bash
# Export OpenPipeline rules from source tenant
monaco download manifest.yaml --environment source-tenant --type openpipeline

# Review and update bucket references in the exported YAML
# Then deploy to target tenant
monaco deploy manifest.yaml --environment target-tenant --dry-run
monaco deploy manifest.yaml --environment target-tenant
```

> **Warning:** If routing rules reference buckets that do not exist in the target tenant, the deploy will succeed but data will be routed to the `default_logs` bucket instead. Always verify bucket existence first.

### Audit Source OpenPipeline Configuration

Query the source tenant to understand which OpenPipeline configurations are actively managed:

```dql
// Audit log: OpenPipeline configuration changes (last 30 days)
fetch logs, from:-30d
| filter matchesPhrase(log.source, "audit")
| filter contains(content, "openpipeline")
| summarize changes = count(), by:{dt.audit.user}
| sort changes desc
| limit 10
```

### Verify OpenPipeline Rules in Target Tenant

After deploying OpenPipeline rules, verify that data is being processed correctly by checking for enrichment fields on recent log records:

```dql
// Verify OpenPipeline enrichment is active in target tenant
// Check for enriched fields on recent logs
fetch logs, from:-1h
| filter isNotNull(dt.security_context)
| summarize enriched_records = count()
| fieldsAdd status = if(enriched_records > 0, then: "Enrichment rules active", else: "No enriched records — check OpenPipeline config")
```

<a id="grail-bucket-configuration"></a>

## 2. Grail Bucket Configuration

Grail buckets control data storage, retention, and access. For consolidation scenarios, bucket naming is critical — multiple source tenants may have identically named custom buckets.

### Bucket Naming Strategy for Consolidation

| Strategy | Pattern | Example | Best For |
|----------|---------|---------|----------|
| **Prefix by source** | `<source>_<original>` | `us-east_app_logs` | Initial migration — avoids all collisions |
| **Prefix by business unit** | `<bu>_<original>` | `retail_app_logs` | Post-merger with clear org boundaries |
| **Unified** | `<original>` with enrichment tags | `app_logs` + `source_tenant` tag | Long-term target after validation |

> **Recommendation:** Start with prefixed buckets. You can consolidate into unified buckets later once data flows are validated. Starting with separate buckets preserves the ability to roll back.

### Retention Alignment

When consolidating tenants, retention policies may differ. Align on the **longer** retention period to avoid compliance violations:

| Source Tenant | Source Retention | Target Retention | Rationale |
|-------------|-----------------|-----------------|------------|
| US-East | 90 days | 360 days | EU-West has PCI requirement — use longer |
| EU-West | 360 days | 360 days | Compliance driver |
| Security logs | 365 days | 365 days | Regulatory minimum |
| Dev/staging | 7 days | 14 days | Increase slightly for migration debugging |

### Inventory Target Tenant Buckets

After bucket deployment, verify all expected buckets exist with correct retention:

```dql
// List all Grail buckets with retention and estimated size
fetch dt.system.buckets
| fields display_name, dt.system.table, retention_days, estimated_uncompressed_bytes
| fieldsAdd size_gb = round(estimated_uncompressed_bytes / 1073741824.0, decimals: 2)
| sort size_gb desc
```

### Verify Data Is Landing in Correct Buckets

After OpenPipeline routing rules are deployed, confirm that data is being routed to the intended buckets:

```dql
// Verify log routing: count records per bucket (last 1 hour)
fetch logs, from:-1h
| summarize record_count = count(), by:{dt.system.bucket}
| sort record_count desc
```

<a id="slo-migration"></a>

## 3. SLO Migration

SLO definitions are portable via Monaco (`slo-v2` type) or Terraform (`dynatrace_slo_v2`), but the metric expressions inside them often contain entity IDs that are **not portable**. Every SLO must be audited for hardcoded entity references before migration.

### SLO Migration Workflow

| Step | Action | Tool |
|------|--------|------|
| 1 | Export SLO definitions from source tenant | `monaco download --type slo-v2` |
| 2 | Audit metric expressions for entity IDs | Manual review or script |
| 3 | Replace entity IDs with tag-based selectors | Manual edit |
| 4 | Deploy SLOs to target tenant | `monaco deploy` |
| 5 | Verify SLO evaluation after data accumulates | DQL query (below) |

### Entity ID to Tag Conversion

```
# BEFORE (hardcoded entity ID — breaks on migration)
metric_expression = "(100)*(builtin:service.errors.server.successCount:splitBy()):merge(0):default(0):sort(value(auto,descending)):limit(20):names:fold(auto)/((builtin:service.requestCount.server:splitBy()):merge(0):default(0):sort(value(auto,descending)):limit(20):names:fold(auto))"
entity_filter = "type(SERVICE),entityId(SERVICE-5E6F7A8B)"

# AFTER (tag-based — survives migration)
entity_filter = "type(SERVICE),tag(app:checkout),tag(env:production)"
```

### Evaluation Window Considerations

SLOs with rolling evaluation windows (e.g., 30-day) will show incomplete data in the target tenant until sufficient history accumulates:

| Evaluation Window | Time to Full Data | Recommendation |
|-------------------|-------------------|----------------|
| 7-day rolling | 7 days | Accept partial data during parallel period |
| 30-day rolling | 30 days | Consider shortening to 7-day temporarily |
| Calendar month | End of month | Start SLOs at beginning of calendar month |

> **Important:** SLOs that report 0% or 100% during the transition are expected. Communicate this to stakeholders before enabling SLOs in the target tenant.

### Terraform SLO Resource Pattern

```hcl
resource "dynatrace_slo_v2" "checkout_availability" {
  name               = "Checkout Service Availability"
  enabled            = true
  evaluation_type    = "AGGREGATE"
  evaluation_window  = "-1w"
  target_success     = 99.9
  target_warning     = 99.95
  metric_expression  = "(100)*(builtin:service.errors.server.successCount:splitBy()):merge(0):default(0)/((builtin:service.requestCount.server:splitBy()):merge(0):default(0))"
  filter             = "type(SERVICE),tag(app:checkout),tag(env:production)"

  # required block — omitting it fails terraform apply schema validation
  error_budget_burn_rate {
    burn_rate_visualization_enabled = true
  }
}
```

### Verify SLO Evaluation in Target Tenant

After SLOs are deployed and data has accumulated, verify that they are evaluating correctly:

```dql
// Audit log: SLO configuration changes (last 7 days)
// Use this to verify SLOs were successfully deployed to target tenant
fetch logs, from:-7d
| filter matchesPhrase(log.source, "audit")
| filter contains(content, "slo")
| summarize changes = count(), by:{dt.audit.user}
| sort changes desc
| limit 10
```

<a id="alerting-and-notification-migration"></a>

## 4. Alerting and Notification Migration

Alerting configuration includes three components that must be migrated together: alerting profiles (which problems trigger alerts), notification rules (where alerts are sent), and maintenance windows (when alerts are suppressed).

### Alerting Migration Sequence

| Order | Component | Dependencies | Monaco Type |
|-------|-----------|-------------|-------------|
| 1 | Alerting profiles | None | `settings` (builtin:alerting.profile) |
| 2 | Notification integrations | Webhook URLs, credential vault entries | `settings` (builtin:problem.notifications) |
| 3 | Notification rules | Alerting profiles must exist | `settings` |
| 4 | Maintenance windows | Management zones (for scope) | `settings` (builtin:alerting.maintenance-window) |

### Webhook URL Updates

Notification rules that send alerts to external systems (PagerDuty, Slack, ServiceNow, email) typically use tenant-specific webhook URLs or API keys:

| Integration | What Changes | Action |
|------------|-------------|--------|
| **PagerDuty** | Integration key (routing key) | May keep same key if routing to same PD service |
| **Slack** | Webhook URL | Update if using per-tenant Slack app |
| **ServiceNow** | Instance URL, credentials | Update credential vault entry in target |
| **Email** | Recipient list | Review and update distribution lists |
| **Custom webhook** | Endpoint URL, authentication | Update URL and credentials |

> **Important during parallel operation:** If both source and target tenants have active alerting, you will receive **duplicate alerts**. Consider disabling alerting in the target tenant until cutover, or routing target alerts to a staging channel.

### Maintenance Window Strategy

| Approach | When to Use |
|----------|-------------|
| **Global suppression** | Deploy a tenant-wide maintenance window in target during parallel period |
| **Per-zone suppression** | Suppress alerts for migrated zones only |
| **No suppression** | If team is ready to triage duplicate alerts |

<a id="data-retention-optimization"></a>

## 5. Data Retention Optimization

The migration is an opportunity to rationalize data retention across data types. In the source tenant, retention policies may have accumulated organically. The target tenant starts clean.

### Recommended Retention by Data Type

| Data Type | Production | Staging | Security/Compliance |
|-----------|-----------|---------|--------------------|
| **Logs** | 35–90 days | 7–14 days | 365 days |
| **Spans** | 35 days | 7 days | 90 days |
| **Metrics** | 5 years (built-in) | 5 years | 5 years |
| **Events** | 35 days | 14 days | 90 days |
| **Business events** | 90–365 days | 35 days | 365 days |

### Cost Optimization Strategies

| Strategy | Impact | Implementation |
|----------|--------|----------------|
| Reduce verbose log retention | High | Route debug/info logs to short-retention bucket |
| Drop noisy spans | Medium | OpenPipeline filter for health-check spans |
| Aggregate before store | Medium | Use OpenPipeline extraction to create metrics from logs |
| Separate compliance data | Low effort, high governance value | Dedicated bucket with extended retention |

### Compare Source vs Target Bucket Sizes

Run this query in both tenants to compare data volumes and confirm parity:

```dql
// Bucket size comparison — run in both source and target tenants
fetch dt.system.buckets
| fields display_name, dt.system.table, retention_days, estimated_uncompressed_bytes
| fieldsAdd size_gb = round(estimated_uncompressed_bytes / 1073741824.0, decimals: 2)
| fieldsAdd cost_tier = if(retention_days > 365, then: "Extended",
    else: if(retention_days > 90, then: "Standard", else: "Short"))
| sort size_gb desc
```

<a id="step-completion-checklist"></a>

## 6. Step Completion Checklist

Before proceeding to **Step 8 — Enable**, confirm that you have completed each item:

| Checkpoint | Status |
|-----------|--------|
| Grail buckets created in target tenant with correct retention | [ ] |
| OpenPipeline rules deployed (enrichment, extraction, routing, masking) | [ ] |
| Data is landing in correct buckets (verified via DQL) | [ ] |
| Enrichment fields are present on new records | [ ] |
| SLO definitions exported and entity IDs replaced with tags | [ ] |
| SLOs deployed to target tenant | [ ] |
| Alerting profiles migrated | [ ] |
| Notification rules migrated with updated webhook URLs | [ ] |
| Maintenance windows configured for parallel operation period | [ ] |
| Data retention policies aligned and documented | [ ] |
| Stakeholders informed of SLO evaluation timeline (rolling window fill period) | [ ] |

---

## Next Step

> **S2S-08: Step 8 — Enable** — Manage parallel operation, communicate to stakeholders, establish Dynatrace Intelligence baselines, and prepare for cutover.

| Completed | Next | Remaining |
|-----------|------|-----------|
| ~~1. Discover~~ → ~~2. Strategize~~ → ~~3. Design~~ → ~~4. Prepare~~ → ~~5. Execute~~ → ~~6. Integrate~~ → ~~7. Expand~~ | **8. Enable** | 9. Optimize |

---

## Summary

In Step 7, you:

- Migrated OpenPipeline processing rules (enrichment, extraction, routing, masking) following the dependency order: buckets → enrichment → pipelines → segments
- Configured Grail buckets in the target tenant with a prefix strategy for consolidation and aligned retention policies
- Migrated SLO definitions after replacing hardcoded entity IDs with tag-based selectors
- Migrated alerting profiles, notification rules, and maintenance windows with updated webhook URLs
- Optimized data retention across data types for cost and compliance

### Additional Resources

- [OpenPipeline Documentation](https://docs.dynatrace.com/docs/platform/openpipeline)
- [Grail Buckets](https://docs.dynatrace.com/docs/platform/grail/grail-buckets)
- [SLO Configuration](https://docs.dynatrace.com/docs/observe-and-explore/service-level-objectives)
- [Alerting and Notifications](https://docs.dynatrace.com/docs/observe-and-explore/notifications-and-alerting)
- [Monaco Configuration as Code](https://docs.dynatrace.com/docs/deliver/configuration-as-code/monaco)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
