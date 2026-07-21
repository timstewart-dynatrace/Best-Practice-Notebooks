# ORGNZ-10: Advanced Segment Definitions

> **Series:** ORGNZ — Organize Data: Buckets, Segments, Security | **Notebook:** 10 of 10 | **Created:** February 2026 | **Last Updated:** 07/21/2026

## Overview

This notebook is a deep-dive into the mechanics of creating effective segment definitions in Dynatrace Grail. While **ORGNZ-08** introduced segments, their structure, and design patterns, this notebook focuses on the practical details: filter condition syntax, data type include rules, metadata enrichment for segments, advanced variable patterns, and troubleshooting techniques.

The content draws from real-world implementation patterns, including host group-based segmentation strategies used in enterprise environments.

---

## Table of Contents

1. [Filter Condition Syntax](#filter-condition-syntax)
2. [Data Type Include Rules](#data-type-include-rules)
3. [Primary Grail Fields and Enrichment](#primary-grail-fields-and-enrichment)
4. [Advanced Variable Patterns](#advanced-variable-patterns)
5. [Host Group-Based Segments](#host-group-based-segments)
6. [Segment Visibility and Sharing](#segment-visibility-and-sharing)
7. [Cross-App Integration](#cross-app-integration)
8. [Known Limitations and Workarounds](#known-limitations-and-workarounds)
9. [Troubleshooting and Performance](#troubleshooting-and-performance)
10. [Segments via Settings API and Terraform](#segments-via-settings-api-and-terraform)
11. [Consuming Segments via the Query API](#consuming-segments-via-the-query-api)
12. [Davis Problem Segment Include Shape](#davis-problem-segment-include-shape)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Dynatrace Environment** | SaaS environment with Grail enabled |
| **Permissions** | `storage:filter-segments:read` and `storage:filter-segments:write` |
| **Knowledge** | Completed ORGNZ-08 (Grail Segments fundamentals) |
| **Data** | At least 1 hour of log, span, and entity data |

## Learning Objectives

By the end of this notebook, you will:
- Master filter condition syntax including operators, field access, and combining conditions
- Understand data type include rules and the one-include-per-type constraint
- Know which metadata fields propagate across all signals (and which don't)
- Create advanced variable definitions using DQL entity queries
- Build host group-based segments from real-world naming conventions
- Troubleshoot common segment issues and apply performance best practices

<a id="filter-condition-syntax"></a>

## 1. Filter Condition Syntax

Segment filter conditions define which data a segment includes. Understanding the available operators and field access patterns is critical for building effective segments.

### Supported Operators

| Operator | Example | Notes |
|----------|---------|-------|
| `=` (equals) | `k8s.namespace.name = "production"` | Exact match, best performance |
| `!=` (not equals) | `k8s.namespace.name != "kube-system"` | Exclude specific values |
| `starts-with` | `dt.host_group.id = $platform*` | Prefix match with wildcard `*` |
| `IN` | `k8s.namespace.name IN ("prod", "staging")` | Match any value in list |
| `NOT IN` | `k8s.namespace.name NOT IN ("kube-system")` | Exclude list of values |

> **Important — operator availability depends on data type:**
> - The underlying operator set is **`=` and `in()`** only. `starts-with`, `contains`, and `ends-with` are not separate operators — they are expressed by placing a **wildcard `*` inside the value** (`"prefix*"`, `"*text*"`, `"*suffix"`).
> - **Signal data includes** (logs, spans, events, metrics) accept wildcards freely, so all three matching modes are available.
> - **Classic entity includes** (`dt.entity.*`): documented wildcard support applies to the `entity.name` property. Other entity *properties* support exact `equals` only, so wildcard matching is unavailable on them. Tag conditions test **membership in the tag set** rather than a substring, so verify tag-based entity includes against your own tenant.
> - An asterisk may follow a variable name in a starts-with condition — `foo = $bar*`.
>
> For cross-signal consistency, prefer prefix-based naming conventions (e.g., `prod-web-tier` not `web-tier-prod`) so `starts-with` works everywhere.

### Field Access Patterns

| Pattern | Example | Use Case |
|---------|---------|----------|
| Direct field | `k8s.namespace.name = "production"` | Primary Grail Fields |
| Tag matching | `matchesValue(tags, "env:production")` | Entity tags |
| Entity name | `entity.name = "my-service"` | Specific entity |
| Host group | `dt.host_group.id = "prod-web"` | Host group filtering (signal data) |
| JSON field | `content$.uri = "/health"` | Nested JSON in log content |

### Combining Conditions

- **AND** — Multiple conditions within a single include are AND-combined:
  ```
  k8s.namespace.name = "production" AND k8s.cluster.name = "main-cluster"
  ```
- **OR** — Use the `or` keyword within a single include (you cannot have multiple includes for the same data type):
  ```
  k8s.namespace.name = "production" OR k8s.namespace.name = "staging"
  ```

### Test Your Filter Conditions

Before creating a segment, always validate your filter conditions with DQL. Run the equivalent filter clause as a DQL query to verify the results match your expectations.

<a id="data-type-include-rules"></a>

## 2. Data Type Include Rules

Segments use **includes** to define which data types are filtered. Understanding how includes work is essential for building segments that behave as expected.

### One Include Per Data Type

A segment can have **only one include block per data type**. You cannot define two separate include blocks for `logs`. To match multiple conditions, combine them with `OR` within a single include.

### Supported Data Types

| Data Type | Category | Example Filter |
|-----------|----------|----------------|
| `logs` | Signal data | `k8s.namespace.name = "prod"` |
| `spans` | Signal data | `service.name starts-with "checkout"` |
| `events` | Signal data | `event.type = "CUSTOM_INFO"` |
| `bizevents` | Signal data | `event.provider = "my-app"` |
| `dt.entity.host` | Classic entity | `tags contains "env:prod"` |
| `dt.entity.service` | Classic entity | `entity.name starts-with "payment"` |
| `dt.entity.process_group` | Classic entity | `relationship: runsOn dt.entity.host` |
| `dt.entity.kubernetes_cluster` | Classic entity | `entity.name = "main-cluster"` |

### How Conditions Apply Per Data Type

When a segment is active and a user queries a specific data type, **only the matching include rule is injected**:

- Running `fetch logs` → only the `logs` include condition applies
- Running `fetch dt.entity.host` → only the `dt.entity.host` include condition applies
- Entity include rules do **NOT** filter signal data (and vice versa)

> **Critical:** Not all metadata is available on all data types. For example, `java.jar.file` exists only on spans. If you filter a segment by `java.jar.file`, it will only apply to spans — not to logs, metrics, or events. **Always use Primary Grail Fields for cross-signal segment filtering.**

### Include Limits

| Limit | Value |
|-------|-------|
| Maximum includes per segment | 20 |
| Include blocks per data type | 1 |
| Expressions per filter condition | 10 |

Since each data type gets at most 1 include, a segment can cover up to 20 distinct data types or entity types.

<a id="primary-grail-fields-and-enrichment"></a>

## 3. Primary Grail Fields and Enrichment

The foundation of effective segment definitions is **metadata enrichment**. In Dynatrace's 3rd-generation architecture, each data point is treated independently. Unlike 2nd-gen where signals were tightly coupled to entities via Management Zones, in 3rd-gen every signal (logs, spans, metrics, events) must be enriched with the corresponding metadata.

### Primary Grail Fields

Primary Grail Fields are infrastructure-related fields that are **automatically propagated across ALL data types** (spans, logs, metrics, topology). These are the ideal fields for segment filter conditions because they guarantee cross-signal consistency.

| Primary Grail Field | DQL Field Name | Source |
|---------------------|----------------|--------|
| **Host Group** | `dt.host_group.id` | OneAgent host group configuration |
| **Kubernetes Cluster** | `k8s.cluster.name` | K8s integration |
| **Kubernetes Namespace** | `k8s.namespace.name` | K8s integration |
| **AWS Account** | `aws.account.id` | AWS integration |
| **Azure Subscription** | `azure.subscription` | Azure integration |
| **GCP Project** | `gcp.project.id` | GCP integration |

> **Best Practice:** Always prefer Primary Grail Fields in segment definitions. They are indexed, automatically enriched, and consistent across all signal types.

### Fields That Also Propagate to Service Metrics

Some fields go further and propagate to derived data like service metrics:

| Field | Propagates to Service Metrics |
|-------|------------------------------|
| `dt.security_context` | Yes |
| `dt.cost.costcenter` | Yes |
| `dt.cost.product` | Yes |
| Host group (`dt.host_group.id`) | Yes |
| Other host tags | **No** |

### Primary Grail Tags

When Primary Grail Fields don't align with how your organization wants to segment data (e.g., shared infrastructure with multiple apps on one host group), use **Primary Grail Tags**.

- Identified by the prefix `primary_tags.`
- Example: `primary_tags.stage=prod`, `primary_tags.team=platform`
- Set via host properties, environment variables (`DT_TAGS`), or OpenPipeline
- Propagated across all data points, similar to Primary Grail Fields

> **Limitation:** Primary Grail Tags do not yet enrich Kubernetes metrics or events. They work for logs, spans, and topology data. Check the [Dynatrace documentation](https://docs.dynatrace.com/docs/ingest-from/setup-on-k8s/guides/metadata-automation/k8s-metadata-telemetry-enrichment) for the latest supported signal types.

### Enrichment Approaches

| Scenario | Approach | Effort | Granularity |
|----------|----------|--------|-------------|
| **Dedicated Infrastructure** | Host group + host properties | Low | Host-level |
| **Shared Infrastructure** | `DT_TAGS` environment variable per process | Higher | Process-level |
| **Kubernetes** | Primary Grail Fields (automatic) | None | Namespace/cluster |
| **Cloud (AWS/Azure/GCP)** | Native cloud tags | None | Account/subscription |

### Enrichment via Host Properties

For dedicated infrastructure, enrich hosts via **Deployment Status** → select hosts → **Modify host properties**:

1. `dt.security_context=<team-or-app>` — For IAM access control
2. `dt.cost.costcenter=<cost-center>` — For cost allocation
3. `dt.cost.product=<product>` — For product tracking
4. `primary_tags.<key>=<value>` — For custom enrichment

> **Restart Requirements:**
> - **Logs**: No application restart needed — OneAgent log module auto-enriches after its own restart
> - **Spans**: Application restart **required** to apply enrichment to traces
> - **Metrics**: OneAgent restart applies automatically

### Verify Enrichment Coverage

Before building segments, verify that the fields you plan to filter on actually exist on your signal data. Fields with 0% coverage need enrichment via OneAgent configuration or OpenPipeline.

<a id="advanced-variable-patterns"></a>

## 4. Advanced Variable Patterns

Variables make segments dynamic by letting users select values at runtime (e.g., choosing a host group or namespace from a dropdown).

### Primary vs Secondary Variables

The variable definition is a DQL query. The columns in the result set determine the variable behavior:

| Column Position | Role | Behavior |
|----------------|------|----------|
| **First column** | Primary variable | Shown in the dropdown selector |
| **Additional columns** | Secondary variables | Available for use in filter conditions |

### Variable Rules

| Rule | Detail |
|------|--------|
| Maximum values in dropdown | 10,000 per variable |
| Selected values per segment | 100 maximum |
| Wildcard | Allowed **after** variable name: `$cluster*` (starts-with) |
| No wildcard in names | Variable names and values cannot contain `*` |
| Permissions | Users must have read access to entities queried by the variable DQL |
| Empty dropdown | Usually means the user lacks permission for the variable query |

### Variable DQL Examples

The following queries demonstrate how to create variable definitions for common use cases.

### Multi-Include Segment Pattern: Tag-Based Variables

The CloudFoundry query above creates three variables from process group tags:

| Variable | Example Value | Use |
|----------|---------------|-----|
| `$value` | `TeamA` | Display in dropdown |
| `$id` | `PROCESS_GROUP-1234567` | Filter signal data by entity ID |
| `$AppTag` | `[CloudFoundry]Organization:TeamA` | Match entity tags |

Use these variables across **multiple includes** to build a segment that filters both entity and signal data:

![Multi-Include Segment Pattern](images/10-multi-include-segment-pattern.png)

<!-- MARKDOWN_TABLE_ALTERNATIVE
| Include Type | Filter | Category |
|-------------|--------|----------|
| Process Group | tags = $AppTag | Entity include (anchor) |
| Process Group Instance | instance_of Process Group | Entity include (relationship traversal) |
| Host | runs Process Group | Entity include (relationship traversal) |
| Service | runs_on Process Group | Entity include (relationship traversal) |
| Spans | dt.entity.process_group in ($id) | Signal include (entity ID filter) |
| Logs | dt.entity.process_group in ($id) | Signal include (entity ID filter) |
| Metrics | dt.entity.process_group in ($id) | Signal include (entity ID filter) |
| Events | dt.entity.process_group in ($id) | Signal include (entity ID filter) |
For environments where SVG doesn't render
-->

| Include Type | Filter | How It Works |
|-------------|--------|--------------|
| **Process Group** | `tags = $AppTag` | Matches process groups by tag |
| **Process Group Instance** | `instance_of Process Group` | Relationship traversal — includes instances of matched process groups |
| **Host** | `runs Process Group` | Relationship traversal — includes hosts running matched process groups |
| **Service** | `runs_on Process Group` | Relationship traversal — includes services on matched process groups |
| **Spans** | `dt.entity.process_group in ($id)` | Signal include — filters spans by process group entity ID |

> **Key Insight:** This pattern bridges entity and signal data in a single segment. Entity includes use tag matching and relationship traversal to cascade across the topology, while signal includes use the entity ID variable to filter spans, logs, and metrics directly. This is especially powerful when Primary Grail Fields are not available for your use case and you need to segment by application-level metadata like CloudFoundry organizations or custom tags.

### Using Variables in Filter Conditions

After defining a variable query, reference the columns in your segment filter:

```
Filter: dt.host_group.id = $host_group OR dt.entity.host_group = $id
```

Where `$host_group` references the primary variable (first column) and `$id` references the secondary variable (second column).

**Wildcard with variables** — Use `$variable*` to match prefixes:

```
Filter: dt.host_group.id = $platform*
```

This matches all host groups that **start with** the selected platform value.

<a id="host-group-based-segments"></a>

## 5. Host Group-Based Segments

A common enterprise pattern is encoding multiple dimensions into host group names using a naming convention. This section shows how to create segments from these encoded dimensions.

### Real-World Pattern

Consider a customer using the host group naming convention `<platform>_<app>_<stage>`:

| Host Group | Platform | App | Stage |
|------------|----------|-----|-------|
| `k8s_multi_prod` | k8s | multi | prod |
| `k8s_easytravel_staging` | k8s | easytravel | staging |
| `onPrem_easytravel_staging` | onPrem | easytravel | staging |
| `onPrem_easytrade_prod` | onPrem | easytrade | prod |

### Best Practice: One Segment Per Dimension

Create a **separate segment for each dimension** (platform, app, stage). This gives users the flexibility to combine them as needed.

### Step 1: Extract Dimensions with DQL

Use DQL to parse the host group name and extract each dimension as a variable:

### Step 2: Create Variable DQL for Each Dimension

**Platform variable:**
```dql
fetch dt.entity.host_group
| parse entity.name, """LD:platform '_' LD:app '_' LD:stage"""
| dedup platform
| fields platform
```

**App variable:**
```dql
fetch dt.entity.host_group
| parse entity.name, """LD:platform '_' LD:app '_' LD:stage"""
| dedup app
| fields app
```

**Stage variable:**
```dql
fetch dt.entity.host_group
| parse entity.name, """LD:platform '_' LD:app '_' LD:stage"""
| dedup stage
| fields stage
```

### Step 3: Define Filter Conditions

| Segment | Filter Condition | Notes |
|---------|-----------------|-------|
| Platform | `dt.host_group.id = $platform*` | Matches host groups starting with selected platform |
| App | `dt.host_group.id contains $app` | Matches host groups containing selected app (signal includes only) |
| Stage | `dt.host_group.id ends-with _$stage` | Matches host groups ending with selected stage (signal includes only) |

> **Note:** contains and ends-with matching (wildcards anywhere in the value) works on signal data includes. On classic entity includes, documented wildcard support applies to `entity.name`; other entity properties are exact-equals only, so prefix-based naming conventions are the reliable pattern there. Always test with the segment preview before deploying.

### Step 4: Validate Across Signal Types

After creating the segments, validate they filter correctly across:
- **Logs** — Check record count reduction when segment is applied
- **Distributed Traces** — Verify spans are filtered by the host group dimension
- **Hosts / Services** — Confirm entity filtering matches expectations
- **Infrastructure** — Verify metrics are scoped correctly

> **Tip:** The host group is a Primary Grail Field, so it propagates across all signal types. This makes it an excellent candidate for segment filtering.

> **K8s analog.** For Kubernetes-heavy tenants, the equivalent Primary Grail Fields are `k8s.cluster.name` (cluster-scoped segment) and `k8s.namespace.name` (namespace-scoped segment). The canonical [Segment Kubernetes clusters (DT docs)](https://docs.dynatrace.com/docs/manage/segments/use-cases/segments-use-cases-kubernetes-clusters) walkthrough uses exactly this pattern — one filter condition (e.g. `k8s.cluster.name = "gke-klu"`) applied across signals and entities. The Kubernetes-namespace entity type is `dt.entity.cloud_application_namespace` (not the more intuitive-sounding `dt.entity.kubernetes_namespace` — which does not exist).

<a id="segment-visibility-and-sharing"></a>

## 6. Segment Visibility and Sharing

### Visibility Settings

| Setting | Behavior |
|---------|----------|
| **Unlisted** (default) | Visible only in the owner's list of segments |
| **Anyone in the environment** | Listed in everyone's segment list in the environment |

### Permissions Model

| Permission | Action | Included In |
|------------|--------|-------------|
| `storage:filter-segments:read` | View and use segments | Dynatrace Standard User |
| `storage:filter-segments:write` | Create and edit segments | Dynatrace Standard User |
| `storage:filter-segments:share` | Share with others | Dynatrace Standard User |
| `storage:filter-segments:delete` | Delete segments | Dynatrace Standard User |
| `storage:filter-segments:admin` | Manage segment permissions | Dynatrace Professional User |

### Governance Recommendations

- **Platform team** owns environment and infrastructure segments and sets them to *Anyone in the environment*
- **Application teams** own app-specific segments, kept *Unlisted*, and surface them by reference from team-owned notebooks or dashboards (there is no direct "share with group" mechanism on the segment itself — sharing flows through the containing document)
- Maintain a small set of "official" environment-wide segments
- Use consistent naming: `team-`, `env-`, `app-`, `region-` prefixes

> **Visibility ≠ access control.** The visibility setting only affects whether a segment is listed in other users' segment pickers. **Anyone with `storage:filter-segments:read` can apply any segment they know the ID of** — IAM policies on the underlying data (logs/spans/buckets) are what actually gate what they see when the segment is applied.

> <sub>**Sources:** [Visibility of segments (DT docs)](https://docs.dynatrace.com/docs/manage/segments/concepts/segments-concepts-visibility) — verbatim: *"the **Visibility** setting doesn't affect general access to segments"* and *"Unlisted segments can still be made available to others by being referenced in apps, such as in shared notebooks and dashboards."*</sub>

<a id="cross-app-integration"></a>

## 7. Cross-App Integration

### Cross-App Persistence

When you select a segment in one Dynatrace app (e.g., Logs), the selection **persists** as you navigate to other apps (e.g., Distributed Traces, Problems). This means you don't need to reapply filters when drilling into different data types during an investigation.

### App Support Matrix

| App | Segment Support | Notes |
|-----|----------------|-------|
| **Dashboards** | Full | Dashboard-level and tile-level segments |
| **Notebooks** | Full | Segment selector at top of notebook |
| **Logs** | Full | Filters all log queries |
| **Distributed Traces** | Full | Filters span queries |
| **Problems** | Full | Requires events include with `event.kind = "DAVIS_PROBLEM"` |
| **SLOs** | Full | Filters SLO evaluation scope |
| **Site Reliability Guardian** | Full | Filters validation scope |
| **Workflows** | Full | Available in workflow DQL steps |
| **Smartscape** | Full | Filters topology view |

### Dashboard Segment Behavior

- **Dashboard-level segment** applies to all tiles
- **Tile-level segment** overrides the dashboard segment for that specific tile
- Use this pattern for executive dashboards that show cross-team metrics alongside team-specific views

<a id="known-limitations-and-workarounds"></a>

## 8. Known Limitations and Workarounds

| Limitation | Detail | Workaround |
|------------|--------|------------|
| **detected problems require event includes** | To filter problems with segments, define an events include with `event.kind = "DAVIS_PROBLEM"`; entity includes alone do not filter problem records | Add event-type includes alongside entity includes |
| **Single relationship traversal** | Entity relationships can only traverse one hop | Target specific entity types directly in includes |
| **Inclusions only, no exclusions** | Cannot say "everything EXCEPT team-X" | Explicitly include what you want (MZ supported exclusions; segments do not) |
| **Entity properties are equals-only** | Wildcard matching on classic entity includes is documented for `entity.name`; other entity properties support exact `equals` only | Use prefix-based naming conventions; put the broader condition on a signal include |
| **1 include per data type** | Cannot have two `logs` include blocks | Combine conditions with `OR` within a single include |
| **Max 20 includes per segment** | Hard limit on rule count | Consolidate rules; use variables for flexibility |
| **Max 10 expressions per filter** | Limits filter complexity | Use OpenPipeline to pre-enrich data with simpler filter fields |
| **Max 10 segments per query** | Cannot stack unlimited segments | Design broader segments instead of many narrow ones |
| **Variable dropdown empty** | Users without entity read permissions see empty dropdown | Ensure variable DQL targets entities the user can read |

<a id="troubleshooting-and-performance"></a>

## 9. Troubleshooting and Performance

### Common Issues

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Empty variable dropdown | Missing entity read permissions | Grant `dt.entity.*:read` to user's policy |
| Segment shows no data | Filter field not enriched on signal data | Verify enrichment with audit queries from Section 3 |
| Segment filters entities but not logs | Entity includes don't apply to signals | Add separate signal-type includes (logs, spans) |
| Unexpected data included | OR logic combining values | Review filter condition logic within the include |
| Segment missing in app | Segment set to unlisted and not shared | Share segment or set visibility to public |
| Segment works for logs but not spans | Field exists on logs but not spans | Use Primary Grail Fields for cross-signal consistency |

### Performance Best Practices

Every segment filter condition is injected into every query the user runs. Keep conditions simple:

1. **Prefer Primary Grail Fields** — They are indexed and optimized for filtering
2. **Use exact matches** — `=` is faster than `starts-with`
3. **Minimize expressions** — Fewer conditions = better query performance
4. **Use OpenPipeline pre-processing** — If conditions would be complex, pre-enrich a simple field (like `dt.security_context`) and filter on that instead
5. **Narrow scope first** — Start with the most restrictive condition in AND combinations

### Naming Conventions

| Prefix | Use Case | Examples |
|--------|----------|----------|
| `team-` | Team-based segments | `team-platform-infra`, `team-checkout` |
| `env-` | Environment segments | `env-production`, `env-staging` |
| `app-` | Application segments | `app-checkout-full`, `app-payments` |
| `region-` | Regional segments | `region-us-east`, `region-eu-west` |

### Segment Lifecycle

1. **Design** — Identify data types, filter conditions, variables
2. **Test** — Validate filter logic with DQL queries
3. **Create** — Build in the Segments app
4. **Share** — Distribute to appropriate users/groups
5. **Monitor** — Check for empty results, permission issues
6. **Retire** — Remove when no longer needed; review quarterly

### Discover Available Filter Candidates

### DQL: Enrichment Coverage Audit

Audit primary Grail field population before building segment definitions:

```dql
// Audit primary Grail field coverage on logs — low-coverage fields need enrichment before segment use
fetch logs, from:-1h
| summarize
    total = count(),
    with_namespace = countIf(isNotNull(k8s.namespace.name)),
    with_cluster = countIf(isNotNull(k8s.cluster.name)),
    with_host_group = countIf(isNotNull(dt.host_group.id)),
    with_security_ctx = countIf(isNotNull(dt.security_context))
| fieldsAdd
    ns_pct = round(toDouble(with_namespace) / toDouble(total) * 100, decimals: 1),
    cluster_pct = round(toDouble(with_cluster) / toDouble(total) * 100, decimals: 1),
    hg_pct = round(toDouble(with_host_group) / toDouble(total) * 100, decimals: 1),
    sec_pct = round(toDouble(with_security_ctx) / toDouble(total) * 100, decimals: 1)
```

<a id="segments-via-settings-api-and-terraform"></a>

## 10. Segments via Settings API and Terraform

Segments authored in the Segments app are convenient for exploration, but **production-grade segment libraries should be managed as code**. Manual segments drift, get orphaned when their creator leaves, and lack the review/promotion discipline that buckets and IAM policies already have in mature tenants.

### Schema

Segments are managed through the Settings 2.0 API under schema **`builtin:filter-segments`**. The object body carries the same fields you fill in the UI:

| Field | Purpose |
|-------|---------|
| `name` | Stable internal identifier (used by automation; cannot contain `*`) |
| `description` | Free-text purpose statement |
| `includes` | Array of include blocks — one per data type (see §2) |
| `variables` | Optional DQL-backed variable definitions (see §4) |
| `visibility` | `PRIVATE` (creator + shares) or `PUBLIC` (everyone) |

### Create via API — minimal example

```bash
curl -X POST "$DT_ENV/api/v2/settings/objects" \
  -H "Authorization: Bearer $DT_PLATFORM_TOKEN" \
  -H "Content-Type: application/json" \
  -d '[{
    "schemaId": "builtin:filter-segments",
    "scope": "environment",
    "value": {
      "name": "env-production",
      "description": "Production workloads across all platforms",
      "visibility": "PUBLIC",
      "includes": [
        { "dataType": "logs",  "filter": "dt.host_group.id starts-with \"prod-\"" },
        { "dataType": "spans", "filter": "dt.host_group.id starts-with \"prod-\"" }
      ]
    }
  }]'
```

> **Auth scheme:** Platform Tokens (`dt0s16.*` / `dt0s01.*`) require the `Bearer` header; classic API Tokens (`dt0c01.*`) require `Authorization: Api-Token`. Mixing schemes returns 401 even with all the right scopes — see AUTOM-04 §3 for the routing rules.

### Manage via Terraform

The `dynatrace-oss/dynatrace` Terraform provider exposes segments through the `dynatrace_segment` resource. The structural shape mirrors the Settings 2.0 schema above. Required scopes on the credential:

- `storage:filter-segments:read` and `storage:filter-segments:write` for create/update
- `storage:filter-segments:share` if the segment's visibility is set beyond the owner (there is no share-with-specific-group mechanism — see §6)
- `storage:filter-segments:delete` if `terraform destroy` should remove segments

See AUTOM-04 §6 for the resource-table-driven catalog (including which auth scheme each Terraform resource requires) and AUTOM-09 for the broader GitOps repo layout and state-backend recommendations. The `dynatrace_segment` resource is Platform-Token-friendly; combined auth (`DYNATRACE_HTTP_OAUTH_PREFERENCE=true`) is the recommended default so the Settings 2.0 object carries the calling service user as `owner` for IAM filtering downstream.

### When IaC is worth the overhead

| Scenario | Author in UI | Manage as code |
|----------|-------------|----------------|
| One-off investigation segment used by a single engineer | Yes | No |
| Standing "official" segments shared across teams (env, team, app) | No | Yes |
| Segments referenced from dashboards or workflows that are themselves in code | No | Yes |
| Segments enforcing a compliance scope (segment + bucket + policy triplet) | No | Yes |
| Segments whose `variables` block contains DQL the team is iterating on | Yes (then promote) | Yes |

The pragmatic split most tenants converge on: **exploration in the UI, promotion to code** the moment a segment becomes load-bearing for someone else's workflow.

### Drift between UI and code

When a segment is managed in code but someone edits it through the UI, the next `terraform apply` (or settings-objects PUT) reverts the change. Two defenses:

1. **Document ownership** — add a `description` like `Managed in Terraform repo X — open a PR, do not edit here`.
2. **Restrict write scope** in production: only the IaC service user holds `storage:filter-segments:write` for `PUBLIC` segments; everyone else gets `read` + private-segment authoring.

> <sub>**Sources:** [Get started with segments — analyze monitoring data (DT docs)](https://docs.dynatrace.com/docs/manage/segments/getting-started/segments-getting-started-analyze-monitoring-data), [Settings 2.0 API schemas (DT docs)](https://docs.dynatrace.com/docs/dynatrace-api/environment-api/settings/schemas). **Derived:** the UI-vs-code split table is a synthesis of community IaC practice — there is no Dynatrace doc that prescribes when to promote a segment from UI to code.</sub>

<a id="consuming-segments-via-the-query-api"></a>

## 11. Consuming Segments via the Query API

A common point of confusion: **there is no inline DQL syntax that references a saved segment by name.** You will not find a `loadSegments:` argument on `fetch`, nor a `| filter <segment-name>` pipe. Segments are applied as **query context**, passed alongside the DQL string in the Grail Query API request body.

### Request shape

Segments attach to the **`/platform/storage/query/v1/query:execute`** request body as a `segments` array. Each element references a segment by its stable identifier:

```http
POST /platform/storage/query/v1/query:execute
Authorization: Bearer <platform-token>
Content-Type: application/json

{
  "query": "fetch logs | summarize count = count(), by:{loglevel}",
  "segments": [
    { "segmentId": "<segment-uuid>" }
  ]
}
```

The Grail query engine merges the segment's include filters into the executing query before scanning — same effect as if the filters had been hand-written into the DQL.

### Where the API call happens

You almost never invoke the Query API directly. The Dynatrace surfaces that read and write to Grail pass `segments` for you, transparently:

| Surface | How segments are passed |
|---------|------------------------|
| **Notebooks** | Segment selector at top of notebook is injected into every cell's API call |
| **Dashboards** | Dashboard-level selector + per-tile override; injected into each tile's API call |
| **Logs, Distributed Traces, Problems, SLOs apps** | App's segment selector is included in every Grail call the app makes |
| **Workflows DQL task** | Task config carries a segments array; injected at execution time |
| **Site Reliability Guardian validators** | Validator config attaches segments to its evaluation queries |
| **Direct API consumers** (custom apps, scripts, terraform-provider-dynatrace queries) | **You** must add the `segments` array to the request body |

### Implication for custom integrations

If you are writing a script that queries Grail through the API — for example, a Workflow JavaScript task using `@dynatrace-sdk/client-query`, a CI/CD job emitting a Site Reliability Guardian-style check, or a custom Dynatrace app — and you want it to honor a saved segment, **you must pass the `segments` array yourself**. The DQL string alone will not pick up "the currently active segment in the app" — there is no such thing outside a UI session.

### Variables in segments — how the value is supplied

If a segment defines variables (§4), the consumer must also supply the **selected values** for those variables. In the UI this is the dropdown selection; in the API the value travels in the same `segments` array element. Consult the [Grail service overview (Dynatrace Developer)](https://developer.dynatrace.com/develop/platform-services/services/grail-service/) for the precise field name and shape — it has evolved sprint-to-sprint and pinning a specific JSON example here would create drift.

### Why this matters for design

Because segments are query context, **a segment's effective behavior depends on who is calling**. The same segment applied by a user with `storage:logs:read` on bucket A returns rows from bucket A; applied by a user without that scope, the same segment returns nothing from bucket A. Segments do not bypass IAM — they layer on top of it. This is the §9 "Segments filters entities but not logs" troubleshooting story restated as a design principle.

> <sub>**Sources:** [Segments in DQL queries (DT docs)](https://docs.dynatrace.com/docs/manage/segments/concepts/segments-concepts-queries), [Grail service overview (Dynatrace Developer)](https://developer.dynatrace.com/develop/platform-services/services/grail-service/). **Softened:** In community practice, the most common cause of "my segment works in the app but not in my script" is the script's API call missing the `segments` array — verify against your own integrations.</sub>

<a id="davis-problem-segment-include-shape"></a>

## 12. Davis Problem Segment Include Shape

The Problems app shows up as "Full" support in the cross-app integration table in §7, but with a critical asterisk noted in §8: **entity includes alone do not filter problem records**. Davis problems are stored as events, so the segment needs an explicit events include keyed on `event.kind`.

### Include shape

| Data type | Filter | Effect |
|-----------|--------|--------|
| `events` | `event.kind = "DAVIS_PROBLEM"` AND your scoping condition | Filters problem records to the team / env / app scope |
| `dt.entity.host` (optional) | scoping condition on the same dimension | Also scopes the host list shown in problem detail views |
| `dt.entity.service` (optional) | scoping condition on the same dimension | Also scopes the affected services shown |

### Worked example — production problems only

```yaml
name: env-production-problems
description: "Production-scoped Davis problems and the affected entities"
visibility: PUBLIC

includes:
  - dataType: events
    filter: 'event.kind = "DAVIS_PROBLEM" AND dt.host_group.id starts-with "prod-"'

  - dataType: dt.entity.host
    filter: 'tags = "environment:production"'

  - dataType: dt.entity.service
    filter: 'tags = "environment:production"'
```

When this segment is active in the Problems app, three things filter together:

1. The **problem feed** is scoped by the `events` include (the load-bearing rule).
2. The **affected hosts** list inside a problem is scoped by `dt.entity.host`.
3. The **affected services** list inside a problem is scoped by `dt.entity.service`.

Drop the `events` include and the problem feed stops filtering — entity includes alone are not enough.

### Why this works

Davis problems live in the `events` data object with `event.kind = "DAVIS_PROBLEM"`. The Problems app is a UI layer on top of that data; the segment injects its `events` filter into every Grail query the app issues. Entity includes filter the topology views, not the problem records themselves.

### Validating the include

Before saving the segment, run the equivalent DQL to make sure the filter produces the problem set you expect:

```dql
fetch events, from:-24h
| filter event.kind == "DAVIS_PROBLEM"
| filter startsWith(dt.host_group.id, "prod-")
| summarize count = count(), by:{event.status}
```

If the count is zero, the underlying scoping field probably isn't enriched on problem events in your tenant — fall back to a Primary Grail Field that is (§3 audit query).

> <sub>**Sources:** [Problems app (DT docs)](https://docs.dynatrace.com/docs/dynatrace-intelligence/problems-app), [Segments in DQL queries (DT docs)](https://docs.dynatrace.com/docs/manage/segments/concepts/segments-concepts-queries). **Derived:** the three-include "load-bearing events plus optional entity scoping" pattern is a synthesis — the docs note the requirement for an `events` include with `DAVIS_PROBLEM` but do not prescribe pairing it with entity includes; that comes from operator experience with the Problems app's drill-down behavior.</sub>

## Summary

In this notebook you learned:

1. **Filter condition syntax** — Operators (`=`, `!=`, `starts-with`, `contains`), field access, AND/OR logic
2. **Data type include rules** — One include per type, conditions scoped to queried data type
3. **Primary Grail Fields** — Which metadata propagates across all signals (and which doesn't)
4. **Enrichment approaches** — Dedicated vs shared infrastructure, host properties vs DT_TAGS
5. **Advanced variables** — Primary/secondary, DQL queries, permission requirements
6. **Host group-based segments** — Parsing naming conventions, one segment per dimension
7. **Visibility and sharing** — Public vs unlisted, governance model
8. **Cross-app integration** — Persistence, dashboard vs tile-level segments
9. **Limitations and troubleshooting** — detected problem event includes, entity operator restrictions, performance tips
10. **Segments via Settings API and Terraform** — `builtin:filter-segments` schema, the `dynatrace_segment` Terraform resource, UI-vs-code decision criteria, drift management
11. **Consuming segments via the Query API** — `segments` array on `/platform/storage/query/v1/query:execute`; no inline DQL syntax; custom integrations must pass segments themselves
12. **Davis problem segment include shape** — Why entity includes alone are not enough; the load-bearing `events` include with `event.kind = "DAVIS_PROBLEM"`

## Series Summary

| Notebook | Topic | Key Takeaway |
|----------|-------|--------------|
| ORGNZ-01 | Introduction | Three pillars of data organization |
| ORGNZ-02 | Grail Buckets | Bucket fundamentals and limits |
| ORGNZ-03 | Bucket Strategy | Naming, retention, design patterns |
| ORGNZ-04 | Permissions Overview | Permission levels and policy structure |
| ORGNZ-05 | Bucket-Level Access | IAM policies for bucket isolation |
| ORGNZ-06 | Security Context | Fine-grained access with dt.security_context |
| ORGNZ-07 | Advanced Permissions | Record and field-level patterns |
| ORGNZ-08 | Grail Segments | Segment fundamentals and design patterns |
| ORGNZ-09 | Enterprise Patterns | Combined approaches at scale |
| **ORGNZ-10** | **Advanced Segment Definitions** | **Filter syntax, enrichment, variables, IaC, Query API, Davis problems, troubleshooting** |

## References

- [Grail Segments overview (DT docs)](https://docs.dynatrace.com/docs/manage/segments)
- [Include data in segments (DT docs)](https://docs.dynatrace.com/docs/manage/segments/concepts/segments-concepts-includes)
- [Variables in segments (DT docs)](https://docs.dynatrace.com/docs/manage/segments/concepts/segments-concepts-variables)
- [Visibility of segments (DT docs)](https://docs.dynatrace.com/docs/manage/segments/concepts/segments-concepts-visibility)
- [Segment Kubernetes clusters — use case (DT docs)](https://docs.dynatrace.com/docs/manage/segments/use-cases/segments-use-cases-kubernetes-clusters)
- [Segment limits (DT docs)](https://docs.dynatrace.com/docs/manage/segments/reference/segments-reference-limits)
- [Segments in DQL queries (DT docs)](https://docs.dynatrace.com/docs/manage/segments/concepts/segments-concepts-queries)
- [Supported data types in segments (DT docs)](https://docs.dynatrace.com/docs/manage/segments/reference/segments-reference-data-types)
- [Get started with segments — analyze monitoring data (DT docs)](https://docs.dynatrace.com/docs/manage/segments/getting-started/segments-getting-started-analyze-monitoring-data)
- [Settings 2.0 API schemas (DT docs)](https://docs.dynatrace.com/docs/dynatrace-api/environment-api/settings/schemas)
- [Grail service overview (Dynatrace Developer)](https://developer.dynatrace.com/develop/platform-services/services/grail-service/)
- [Problems app (DT docs)](https://docs.dynatrace.com/docs/dynatrace-intelligence/problems-app)

---

<sub>*This notebook was AI-generated from Dynatrace documentation and enterprise best practices. It is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
