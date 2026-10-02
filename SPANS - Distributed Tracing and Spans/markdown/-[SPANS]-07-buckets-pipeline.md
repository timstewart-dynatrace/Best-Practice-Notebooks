# SPANS-07: Grail Buckets & OpenPipeline

> **Series:** SPANS — Distributed Tracing and Spans | **Notebook:** 7 of 8 | **Created:** December 2025 | **Last Updated:** 10/02/2026

## Data Architecture and Processing for Distributed Traces
This notebook covers Dynatrace Grail's bucket architecture for span storage, OpenPipeline configuration patterns, and data governance strategies.

---

## Table of Contents

1. [Understanding Grail Buckets](#understanding-grail-buckets)
2. [Querying from Specific Buckets](#querying-from-specific-buckets)
3. [Bucket Discovery](#bucket-discovery)
4. [OpenPipeline Concepts](#openpipeline-concepts)
5. [Filtering & Dropping Unwanted Spans](#filtering-dropping-unwanted-spans)
6. [Transforming & Enriching Data](#transforming-enriching-data)
7. [Routing to Different Buckets](#routing-to-different-buckets)
8. [Sampling Strategies](#sampling-strategies)
9. [Measuring Span Traffic per Application](#measuring-span-traffic)
10. [Access Control Patterns](#access-control-patterns)
11. [Best Practices Summary](#best-practices-summary)

---


## Prerequisites

Before starting this notebook, ensure you have:

- ✅ Completed previous SPANS notebooks (01-06)
- ✅ Understanding of Dynatrace Grail architecture
- ✅ Admin access for bucket/pipeline configuration (optional)

### OneAgent Attribute Enrichment (OneAgent 1.333+)

> **Requires:** OneAgent version **1.333** or later

OneAgent can enrich **all telemetry signals** (metrics, spans, logs, events, entities) with custom metadata at the source — before data reaches the Dynatrace platform. This is more efficient than server-side tagging (auto-tags) because enrichment happens on the host and propagates to all Smartscape nodes.

**Primary Fields** (standardized from Semantic Dictionary):
- `dt.security_context` — data governance and access control
- `dt.cost.costcenter` — cost allocation
- `dt.cost.product` — product attribution

**Primary Tags** (custom key-value pairs):
- `primary_tags.environment` — environment identification (production, staging, etc.)
- `primary_tags.team` — team ownership
- `primary_tags.business_unit` — organizational unit

**Configuration:**

```bash
# Set during OneAgent installation
Dynatrace-OneAgent-Linux.sh --set-host-tag="primary_tags.environment=production" --set-host-tag="dt.security_context=confidential"

# Set on existing agents via oneagentctl
oneagentctl --set-host-tag="primary_tags.environment=production"
oneagentctl --set-host-tag="dt.cost.costcenter=12345"

# Per-process via environment variable (overrides host-level)
DT_TAGS="primary_tags.team=platform primary_tags.environment=production"
```

**Benefits over auto-tagging:**
| Aspect | Auto-Tags (Server-Side) | Attribute Enrichment (Agent-Side) |
|--------|------------------------|----------------------------------|
| When applied | After data arrives at platform | At the source, before transmission |
| Scope | Entity tags only | All signals: metrics, spans, logs, events, entities |
| Grail integration | Limited | Full — feeds OpenPipeline routing, bucket assignment, permissions |
| Cost allocation | Not supported | `dt.cost.costcenter`, `dt.cost.product` fields |
| Security context | Not supported | `dt.security_context` for data governance |

> **See:** [Primary Grail fields and tags enrichment through OneAgent](https://docs.dynatrace.com/docs/ingest-from/dynatrace-oneagent/oneagent-attribute-enrichment)

<a id="understanding-grail-buckets"></a>
## 1. Understanding Grail Buckets
Grail stores observability data in **buckets** - logical containers that provide data isolation, retention control, and access management.

![Grail Buckets Architecture](images/07-grail-buckets.png)

<!--MARKDOWN_TABLE_ALTERNATIVE
| Bucket | Retention | Purpose |
|--------|-----------|---------|
| default_spans | 10 days (built-in) | Standard span storage |
| production_traces | 90 days | Extended retention for prod |
| sensitive_spans | Varies | Restricted access, compliance |
-->

### Why Use Buckets?

| Purpose | Benefit |
|---------|---------|
| Cost Control | Different retention periods per bucket |
| Access Control | Restrict who can query which data |
| Compliance | Separate sensitive data for audit |
| Performance | Query specific buckets for efficiency |
| Team Isolation | Each team queries their own bucket |

> 💡 **Tip:** The default bucket for spans is `default_spans`. You create custom buckets in storage management, and OpenPipeline's **Bucket assignment** stage sends records to them. Retention for distributed tracing on Grail is *"Configurable, from 10 days to 10 years of retention time"*; the built-in `default_spans` bucket kept 10 days on the validation tenant (`fetch dt.system.buckets`).

> <sub>**Sources:** [Data retention periods (DT docs)](https://docs.dynatrace.com/docs/manage/data-privacy-and-security/data-privacy/data-retention-periods).</sub>

---

<a id="querying-from-specific-buckets"></a>
## 2. Querying from Specific Buckets
Use the `bucket:` parameter to query from specific buckets for improved performance and cost efficiency.

```dql
// Query spans from the default bucket
// Run cell 6 first to see which buckets your spans are in, and use those names.
fetch spans, from:-1h, bucket: {"default_spans"}
| filter span.kind == "server"
| fields start_time, dt.service.name, span.name, duration
| sort start_time desc
| limit 50
```

```dql
// Query from multiple buckets (if you have custom buckets)
// Replace with your actual bucket names
// Run cell 6 first to see which buckets your spans are in, and use those names.
fetch spans, from:-1h, bucket: {"default_spans"}
| filter span.kind == "server"
| summarize {span_count = count()}, by:{dt.service.name}
| sort span_count desc
| limit 20
```

---

<a id="bucket-discovery"></a>
## 3. Bucket Discovery
Discover which buckets contain your data and their characteristics.

```dql
// Find out which bucket your spans are stored in
fetch spans, from:-1h
| fieldsAdd bucket = dt.system.bucket
| summarize {span_count = count()}, by:{bucket}
| sort span_count desc
```

```dql
// Analyze span distribution by bucket and service
fetch spans, from:-1h
| fieldsAdd bucket = dt.system.bucket
| summarize {span_count = count()}, by:{bucket, dt.service.name}
| sort bucket, span_count desc
| limit 50
```

---

<a id="openpipeline-concepts"></a>
## 4. OpenPipeline Concepts
**OpenPipeline** processes incoming telemetry **before** storage. For spans it can drop records, transform and mask fields, assign buckets, and extract metrics and events. It does **not** sample: per the Adaptive Traffic Management docs, *"OpenPipeline processing cannot be used as an alternative to ATM configuration for controlling trace volume."* (see §8).

![OpenPipeline Flow](images/07-openpipeline-flow.png)

<!--MARKDOWN_TABLE_ALTERNATIVE
| Stage | Purpose | Example |
|-------|---------|---------|
| Route | Choose a pipeline | Production spans → `production-spans` pipeline |
| Processing | Drop record, DQL, add/remove/rename fields | Drop health checks; add `duration_ms`; mask URLs |
| Bucket assignment | Choose the bucket (first match) | Production → long-retention bucket |
| Extract | Metrics (sampling-aware on spans), events | Request count per service |
| Sampling | **Not an OpenPipeline step** | Set at the source: OneAgent ATM or an OTel Collector |
-->

> ⚠️ **Note:** OpenPipeline configuration is done in the Dynatrace UI under **Settings > Process and contextualize > OpenPipeline**. This notebook shows how to identify candidates and verify results.

---

<a id="filtering-dropping-unwanted-spans"></a>
## 5. Filtering & Dropping Unwanted Spans
![OpenPipeline Actions](images/07-openpipeline-actions.png)

<!--MARKDOWN_TABLE_ALTERNATIVE
| Action | Purpose | Example Config |
|--------|---------|----------------|
| Drop record | Remove spans | Matching condition `matchesValue(span.name, "*health*")` |
| DQL processor | Modify fields | `fieldsAdd duration_ms = duration / 1ms` |
| Route + Bucket assignment | Send to a bucket | Route matcher picks the pipeline; Bucket assignment `spans_prod` |
| Sampling aware counter | Extract a metric | Matching condition `span.kind == "server"` |
-->

### Candidates for Dropping

1. **Health checks** - `/health`, `/ready`, `/alive` endpoints
2. **Metrics endpoints** - `/metrics`, `/prometheus`
3. **Static assets** - `.js`, `.css`, `.png` requests
4. **Internal noise** - Very frequent internal operations

```dql
// Find health check spans (candidates for dropping)
// matchesValue with path-shaped patterns avoids false hits: a bare substring
// "ping" also matches "ShippingService/GetQuote".
fetch spans, from:-1h
| filter matchesValue(span.name, "*health*")
    or matchesValue(span.name, "*/ready*")
    or matchesValue(span.name, "*/alive*")
    or matchesValue(span.name, "*/ping")
| summarize {count = count()}, by:{dt.service.name, span.name}
| sort count desc
```

```dql
// Find static asset requests (often low value)
fetch spans, from:-1h
| filter isNotNull(url.path)
| filter endsWith(url.path, ".js") or 
        endsWith(url.path, ".css") or
        endsWith(url.path, ".png") or
        endsWith(url.path, ".ico")
| summarize {count = count()}, by:{dt.service.name}
| sort count desc
```

```dql
// Find high-volume, low-value spans
// High volume but almost no errors = candidates for filtering
fetch spans, from:-1h
| summarize {
    count = count(),
    error_count = countIf(span.status_code == "error")
  }, by:{dt.service.name, span.name}
| fieldsAdd error_rate = (error_count * 100.0) / count
| filter count > 1000 and error_rate < 0.1
| sort count desc
```

### OpenPipeline Example: Drop Health Checks

In **Settings > Process and contextualize > OpenPipeline > Spans**, open the pipeline your spans are routed to and add a **Drop record** processor to the **Processing** stage:

| Field | Value |
|-------|-------|
| Processor | Drop record |
| Matching condition | `matchesValue(span.name, "*health*") or matchesValue(span.name, "*/ready*") or matchesValue(span.name, "*/alive*")` |

Matching conditions accept `matchesValue`, `matchesPhrase`, comparisons and boolean logic — `contains()` and `in()` are rejected. Run the query above first so you know what the condition removes.

---
<a id="transforming-enriching-data"></a>
## 6. Transforming & Enriching Data

Pre-compute fields at ingestion time for faster queries. Each is a processor in the **Processing** stage, with its own matching condition.

### OpenPipeline Example: Add Computed Fields

**DQL** processor, matching condition `true`:

```text
fieldsAdd duration_ms = duration / 1ms, is_slow = duration > 1s
```

### OpenPipeline Example: Add Business Context

Two **Add fields** processors (static values):

| Matching condition | Fields added |
|--------------------|--------------|
| `matchesValue(dt.service.name, "checkout*")` | `business.domain = "commerce"`, `business.criticality = "high"` |
| `matchesValue(dt.service.name, "payment*")` | `business.domain = "finance"`, `business.criticality = "critical"` |

`dt.service.name` is present on every span on the validation tenant; `service.name` only on OpenTelemetry spans — match on the field your spans carry, and confirm it in the pipeline's sample-data preview.

```dql
// Example: Fields you might want to pre-compute
fetch spans, from:-1h
| fieldsAdd 
    duration_ms = duration / 1ms,
    is_error = span.status_code == "error",
    latency_bucket = if(
        duration < 100ms, "fast",
        else: if(duration < 1s, "normal",
        else: "slow"))
| fields dt.service.name, span.name, duration_ms, is_error, latency_bucket
| limit 10
```

```dql
// Identify services by domain for enrichment planning
fetch spans, from:-1h
| summarize {count = count()}, by:{dt.service.name}
| sort count desc
| limit 20
```

---
<a id="routing-to-different-buckets"></a>
## 7. Routing to Different Buckets

Route spans to buckets based on:
- **Retention needs** (short vs. long term)
- **Sensitivity** (PII vs. non-PII)
- **Environment** (prod vs. dev)
- **Cost** (high-value vs. low-value)

A **route** sends a record to a **pipeline** (first match wins); the pipeline's **Bucket assignment** stage sends it to a **bucket** (first match wins). Either can carry the condition.

### OpenPipeline Example: Route by Environment (Bucket assignment processors)

| Matching condition | Bucket |
|--------------------|--------|
| `deployment.environment == "production"` | `spans_production_90d` |
| `deployment.environment == "staging"` | `spans_staging_7d` |
| `true` (last) | `spans_default` |

### OpenPipeline Example: Route by Sensitivity (routes)

| Route matcher | Target pipeline (its bucket) |
|---------------|------------------------------|
| `matchesValue(dt.service.name, "payment*") or matchesValue(dt.service.name, "auth*")` | `sensitive-spans` (`spans_sensitive`) |
| *(default route)* | default pipeline (`default_spans`) |

```dql
// Check what environments/namespaces exist for routing planning
fetch spans, from:-1h
| summarize {count = count()}, by:{k8s.namespace.name}
| sort count desc
```

```dql
// Identify sensitive services for routing
fetch spans, from:-1h
| filter contains(span.name, "payment") or
        contains(span.name, "auth") or
        contains(span.name, "login")
| summarize {count = count()}, by:{dt.service.name, span.name}
| sort count desc
```

---
<a id="sampling-strategies"></a>
## 8. Sampling Strategies

**Sampling is decided at the source, not in OpenPipeline.** Per the Adaptive Traffic Management docs: *"Adaptive Traffic Management sampling decisions are made locally (on OneAgent, or on Envoy when using the Dynatrace sampler) before any Dynatrace backend infrastructure including OpenPipeline is involved. OpenPipeline processing cannot be used as an alternative to ATM configuration for controlling trace volume."*

| Source | Where sampling is set | "Keep all errors" possible? |
|--------|-----------------------|-----------------------------|
| OneAgent | **Adaptive Traffic Management** — head-based, decided once at the start of a trace | No — the decision is made before the outcome is known |
| OpenTelemetry | **OTel Collector** `tail_sampling` processor — decided after the trace completes | Yes — Dynatrace's sample configuration *"keeps errors, traces longer than 500ms, and 20% of all remaining traces"* |
| Either | OpenPipeline **Drop record** | Only for spans you can name (health checks, probes) — dropping by rule is not sampling |

If you tail-sample in a Collector, compute service metrics **before** the sampler. The Dynatrace Collector sampling use case does that with the `spanmetrics` connector, so the request counts stay accurate. See **OPIPE-03** for extrapolating counts from sampled spans.

> <sub>**Sources:** [Adaptive Traffic Management with DPS (DT docs)](https://docs.dynatrace.com/docs/ingest-from/dynatrace-oneagent/adaptive-traffic-management/adaptive-traffic-management-saas-dps), [Sampling with the OTel Collector (DT docs)](https://docs.dynatrace.com/docs/ingest-from/opentelemetry/collector/use-cases/sampling).</sub>

```dql
// Identify high-volume services — candidates for ATM or Collector sampling at the source
fetch spans, from:-1h
| summarize {
    total = count(),
    errors = countIf(span.status_code == "error"),
    slow = countIf(duration > 1s)
  }, by:{dt.service.name}
| fieldsAdd error_rate = (errors * 100.0) / total
| fieldsAdd important = errors + slow
| fieldsAdd droppable = total - errors - slow
| filter total > 10000  // High volume services
| sort total desc
```

```dql
// Calculate potential savings from filtering
fetch spans, from:-1h
| summarize {
    total = count(),
    health_checks = countIf(
        contains(span.name, "health") or 
        contains(span.name, "ready") or
        contains(span.name, "alive")),
    static_assets = countIf(
        endsWith(url.path, ".js") or
        endsWith(url.path, ".css") or
        endsWith(url.path, ".png")),
    errors = countIf(span.status_code == "error"),
    slow = countIf(duration > 1s)
  }
| fieldsAdd droppable = health_checks + static_assets
| fieldsAdd must_keep = errors + slow
| fieldsAdd droppable_pct = (droppable * 100.0) / total
```

---

<a id="measuring-span-traffic"></a>
## 9. Measuring Span Traffic per Application

Understanding how much span data each application generates is critical for effective **data partitioning** and **cost allocation**. Every span has a `dt.ingest.size` attribute that records its byte size at ingestion. By extracting a metric from this field in OpenPipeline, you can track span traffic per application over time.

### Step 1: Verify `dt.ingest.size` Exists

First, confirm that spans carry the `dt.ingest.size` attribute:

```dql
// Verify dt.ingest.size is present on spans
fetch spans, from:-1h
| fieldsKeep span.name, dt.ingest.size
| filter isNotNull(dt.ingest.size)
| limit 10
```

### Step 2: Create a Metric Extraction Rule in OpenPipeline

Navigate to **Settings > Process and contextualize > OpenPipeline > Spans** and create a metric extraction rule:

1. Go to the **Pipelines** tab and create a new pipeline (e.g., `General Pipeline`)
2. In the pipeline's **Metric extraction** stage, add a **Sampling aware value metric** processor — on spans, the metric processors are the sampling-aware variants:
   - **Metric key:** `span.ingest.size.by.app`
   - **Value field:** `dt.ingest.size`
   - **Dimension:** `primary_tags.app` (or your organization's app dimension)
3. Go to **Dynamic Routing** and create a route that sends all spans to this pipeline
4. Click **Save**

> **Note:** The dimension field (`primary_tags.app`) varies by customer. Use whichever enrichment field identifies applications in your environment.

### Step 3: Query Span Traffic per Application

Once the metric is flowing (allow a few minutes after saving), query it:

```dql
// Span ingest volume per application (after metric extraction is configured)
timeseries span_ingest = sum(span.ingest.size.by.app), from:-24h, by:{primary_tags.app}
| fieldsAdd daily_gb = arraySum(span_ingest) / 1073741824
| sort daily_gb desc
```

> **Why this matters:** Knowing which applications generate the most span traffic helps you make informed decisions about bucket partitioning, retention policies, sampling strategies, and cost attribution. See **ORGNZ-03: Bucket Strategy and Design** for the complete data partitioning best practice.

---

<a id="access-control-patterns"></a>
## 10. Access Control Patterns
Use bucket-based queries to implement access control patterns.

![Bucket Access Control](images/07-bucket-access-control.png)

<!--MARKDOWN_TABLE_ALTERNATIVE
| Team/Role | Bucket Access | Retention |
|-----------|---------------|-----------|
| Team A (Frontend) | spans_frontend | 35 days |
| Team B (Backend) | spans_backend | 35 days |
| Platform/SRE | All buckets | 35 days |
| Compliance | audit_spans | 1 year |
-->

> 💡 **Tip:** Bucket permissions are configured in the Dynatrace UI under **Account Management > Identity & Access Management**.

```dql
// Data retention analysis: span volume by day
fetch spans, from:-7d
| fieldsAdd day = bin(start_time, 24h)
| summarize {span_count = count()}, by:{day}
| sort day desc
```

```dql
// Volume analysis by service (for cost allocation)
fetch spans, from:-1h
| summarize {
    span_count = count(),
    avg_duration_ms = avg(duration) / 1ms
  }, by:{dt.service.name}
| sort span_count desc
| limit 30
```

---

<a id="best-practices-summary"></a>
## Best Practices Summary
### DO ✅

- Drop health checks and metrics endpoints
- Pre-compute commonly used fields
- Route by environment and sensitivity
- Mask PII before storage
- Reduce volume at the source (OneAgent ATM, Collector tail sampling) and drop named noise with Drop record

### DON'T ❌

- Drop error spans (you'll need them for RCA)
- Drop slow spans (they indicate problems)
- Expect OpenPipeline to sample — it cannot
- Forget to test rules before deploying

---

## Summary

In this notebook, you learned:

✅ **Grail bucket architecture** for organizing and isolating data  
✅ **Querying from specific buckets** using the bucket: parameter  
✅ **Bucket discovery** to understand data distribution  
✅ **OpenPipeline concepts** — routes, processors, bucket assignment  
✅ **Filtering & dropping** unwanted spans (health checks, static assets)  
✅ **Transforming & enriching** data with computed fields  
✅ **Routing to buckets** by environment and sensitivity  
✅ **Measuring span traffic** per application using OpenPipeline metric extraction  
✅ **Sampling** happens at the source (ATM, Collector), not in OpenPipeline  
✅ **Access control patterns** using bucket-based isolation  

---

## Next Steps

Continue to **SPANS-08: Cost-Efficient DQL Queries** to learn:
- Optimizing DQL queries for cost efficiency
- Query cost estimation techniques
- Best practices for production queries
- Indexed fields and performance strategies

🆕 **New Addition (March 2026):** For configuring span processing pipelines (filtering, enrichment, sampling-aware metrics), see **OPIPE-02: Span Processing & Enrichment**.

---

## References

- [Processing in OpenPipeline (DT docs)](https://docs.dynatrace.com/docs/platform/openpipeline/concepts/processing)
- [Adaptive Traffic Management with DPS (DT docs)](https://docs.dynatrace.com/docs/ingest-from/dynatrace-oneagent/adaptive-traffic-management/adaptive-traffic-management-saas-dps)
- [Sampling with the OTel Collector (DT docs)](https://docs.dynatrace.com/docs/ingest-from/opentelemetry/collector/use-cases/sampling)
- [Data retention periods (DT docs)](https://docs.dynatrace.com/docs/manage/data-privacy-and-security/data-privacy/data-retention-periods)
- [DQL best practices (DT docs)](https://docs.dynatrace.com/docs/platform/grail/dynatrace-query-language/dql-best-practices)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
