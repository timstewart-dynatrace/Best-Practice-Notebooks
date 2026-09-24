# OPLOGS-01: OpenPipeline Fundamentals

> **Series:** OPLOGS — OpenPipeline Logs | **Notebook:** 1 of 8 | **Created:** December 2025 | **Last Updated:** 09/24/2026

## Understanding the Unified Data Ingestion Framework
This notebook introduces OpenPipeline, Dynatrace's unified data processing framework for logs, traces, metrics, and events.

> 🆕 **New Addition (March 2026)**
>
> **Companion Series**
> - **OPMIG** — Migrating from classic logs to OpenPipeline? Start with **OPMIG-01: Why Migrate**
> - **OPIPE** — Ready to process spans, metrics, and events? Continue to **OPIPE-01: OpenPipeline as a Multi-Scope Platform**
> - **SPANS** — Need to query and analyze distributed traces? See **SPANS-01: Fundamentals**

---

## Table of Contents

1. [What is OpenPipeline?](#what-is-openpipeline)
2. [OpenPipeline Architecture](#openpipeline-architecture)
3. [Exploring Your OpenPipeline Data](#exploring-your-openpipeline-data)
4. [Key OpenPipeline Fields](#key-openpipeline-fields)
5. [Data Sources Explained](#data-sources-explained)
6. [Pipeline Stages](#pipeline-stages-overview)
7. [Environment Summary](#environment-summary)
8. [📝 Summary](#summary)
9. [➡️ Next Steps](#next-steps)
10. [📚 References](#references)

---


## Prerequisites

- ✅ Access to a Dynatrace environment with log data
- ✅ DQL query permissions (viewer role minimum)
- ✅ Basic understanding of log management concepts

<a id="what-is-openpipeline"></a>
## 1. What is OpenPipeline?
**OpenPipeline** is Dynatrace's unified data ingestion and processing framework that replaces classic log ingestion. It provides:

- **Unified Processing**: Single framework for logs, metrics, traces, and business events
- **Real-time Transformation**: Parse, enrich, mask, and route data at ingestion
- **Grail Storage**: Direct integration with Dynatrace's data lakehouse
- **Flexible Routing**: Send data to different buckets with custom retention
- **Cost Control**: Drop unnecessary data before storage

### OpenPipeline vs Classic Log Ingestion

| Feature | Classic Logs | OpenPipeline v2.0 |
|---------|--------------|-------------------|
| Data Processing | Post-ingestion | At ingestion time |
| Storage | Log Storage v1 | Grail Data Lakehouse |
| Query Language | Limited | Full DQL Support |
| Retention | Global | Per-bucket configurable |
| Data Masking | Limited | DQL processor with DPL (`replacePattern`), `replaceString`, `ipMask` |
| Parsing | Basic | DPL (Dynatrace Pattern Language) |
| Custom Routing | No | Yes, by content/source |

<a id="openpipeline-architecture"></a>
## 2. OpenPipeline Architecture
![OpenPipeline Architecture](images/01-openpipeline-architecture.png)

<!-- MARKDOWN_TABLE_ALTERNATIVE
| Stage | Component | Function |
|-------|-----------|----------|
| **Data Sources** *(Ingest)* | OneAgent, Log Ingest API, OTLP, Generic Ingest, custom | Ingestion entry points; source recorded as `dt.openpipeline.source` |
| **1. Ingest** | Built-in / Ready-made / Custom sources (custom supports optional pre-processing) | Records enter the platform |
| **2. Routing** | Pipeline Selection | Dynamic (DQL matcher) or static (custom sources only) |
| **3. Pipeline** | Fixed sequence of stages: **Processing** (DQL, Add/Remove/Rename fields, Drop record, GeoIP lookup (Early Access), Inline lookup) → Smartscape node → Smartscape edge → Permission → Product allocation → Cost allocation → **Bucket assignment** → **Metric extraction** → **Davis** → **Data extraction** | Masking, parsing, transformation and dropping all happen in the first stage (Processing); extraction stages run after bucket assignment |
| **4. Storage** | Grail bucket chosen by the Bucket assignment stage (or not stored — No storage assignment) | Persist to Grail bucket, or skip retention |
| **Output** | Grail | Logs, Spans, Metrics, Events storage |
-->

> **Stage model (verified 09/24/2026):** data flows Ingest → Routing → pipeline → Storage. Inside a pipeline, per *Processing in OpenPipeline*, "The sequence of stages is fixed for all pipelines and cannot be modified." Masking, dropping, parsing and transformation are processors in the first stage, **Processing**; **Bucket assignment** comes seventh, and **Metric extraction**, **Davis** and **Data extraction** run after it. The diagram's Mask / Filter / Process boxes are all inside that first stage.
>
> <sub>**Sources:** [Processing in OpenPipeline (DT docs)](https://docs.dynatrace.com/docs/platform/openpipeline/concepts/processing)</sub>

<a id="exploring-your-openpipeline-data"></a>
## 3. Exploring Your OpenPipeline Data
Let's start by discovering what data sources and pipelines are active in your environment.

```dql
// Discover data sources feeding OpenPipeline
fetch logs, from: now() - 1h
| summarize {log_count = count()}, by: {dt.openpipeline.source}
| sort log_count desc
```

```dql
// See which pipelines are processing your logs
fetch logs, from: now() - 1h
| summarize {log_count = count()}, by: {dt.openpipeline.pipelines}
| sort log_count desc
```

```dql
// Check storage bucket distribution
fetch logs, from: now() - 1h
| summarize {log_count = count()}, by: {dt.system.bucket}
| sort log_count desc
```

<a id="key-openpipeline-fields"></a>
## 4. Key OpenPipeline Fields
OpenPipeline adds metadata fields to every log record:

### Pipeline Metadata

| Field | Description | Example |
|-------|-------------|----------|
| `dt.openpipeline.source` | How the log was ingested | `oneagent`, `/api/v2/logs/ingest`, `/api/v2/otlp/v1/logs` |
| `dt.openpipeline.pipelines` | Pipeline(s) that processed the log | `["logs:pipeline_Default_Pipeline_2798"]` |
| `dt.system.bucket` | Storage bucket name | `default_logs`, `custom_logs` |

### Core Log Fields

| Field | Description |
|-------|-------------|
| `timestamp` | When the log was generated |
| `content` | The log message body |
| `loglevel` | Log severity (ERROR, WARN, INFO, DEBUG, NONE) |
| `status` | Severity group derived from `loglevel`: ERROR (SEVERE, ERROR, CRITICAL, ALERT, FATAL, EMERGENCY), WARN, INFO (INFO, TRACE, DEBUG, NOTICE), NONE — use it for error-class counts |
| `log.source` | Source identifier (e.g., "Container Output") |
| `log.iostream` | Stream type (stdout, stderr) |

```dql
// View a sample log record with all key fields
fetch logs, from: now() - 1h
| fields timestamp, content, loglevel, status, log.source, log.iostream,
         dt.openpipeline.source, dt.openpipeline.pipelines, dt.system.bucket
| limit 5
```

```dql
// Analyze log levels in your environment
fetch logs, from: now() - 1h
| summarize {count = count()}, by: {loglevel}
| sort count desc
```

<a id="data-sources-explained"></a>
## 5. Data Sources Explained
### OneAgent (`oneagent`)
Logs collected automatically by Dynatrace OneAgent from:
- Container stdout/stderr
- Process log files
- System logs

### Log Ingest API (`/api/v2/logs/ingest`)
Logs sent directly via the Dynatrace API:
- Custom application logs
- Third-party integrations
- Cloud provider logs (AWS, Azure, GCP)

#### Delivery reliability
The Log Ingest API acknowledges accepted data with **HTTP 204 No Content** (empty body). When a request fails on a *retryable* response code (each endpoint documents which codes are retryable), the **client is responsible for retrying with exponential backoff** — Dynatrace does not retry on the sender's behalf. The ActiveGate that serves the endpoint buffers ingested data to an on-disk queue (default **300 MB**, configured by `disk_queue_max_size_mb`; location by `disk_queue_path`) and forwards it to Dynatrace in batches; when that queue fills, the API returns **`503 Usable space limit reached`** as a back-pressure signal — increase `disk_queue_max_size_mb` if this recurs under normal load. Cloud-forwarder integrations (e.g. Amazon Data Firehose) implement their own buffering and smart-retry against the OpenPipeline endpoint.

### OTLP (`/api/v2/otlp/v1/logs`)
OpenTelemetry Protocol logs:
- OpenTelemetry Collector
- Fluent Bit with OTLP output
- Custom OTLP exporters

```dql
// Compare volume by data source
fetch logs, from: now() - 24h
| summarize {
    log_count = count(),
    unique_hosts = countDistinct(dt.entity.host)
  }, by: {dt.openpipeline.source}
| sort log_count desc
```

```dql
// Logs per hour by source (trend analysis)
fetch logs, from: now() - 24h
| makeTimeseries {log_count = count()}, by: {dt.openpipeline.source}, interval: 1h
```

<a id="pipeline-stages-overview"></a>
## 6. Pipeline Stages Overview

A record first passes **Routing**, which picks the pipeline (a dynamic DQL matcher, or a static route for custom ingest sources). Inside the pipeline it then runs through a **fixed** sequence of ten stages — the order cannot be changed:

| # | Stage | What it does in a log pipeline | Processors | Executed |
|---|-------|--------------------------------|------------|----------|
| 1 | **Processing** | Parse values into fields, transform the schema, filter records, mask sensitive data | DQL, Add fields, Remove fields, Rename fields, Drop record, GeoIP lookup (Early Access), Inline lookup | All matches |
| 2 | Smartscape node | Extract Smartscape nodes from matching records | Smartscape node | All matches |
| 3 | Smartscape edge | Extract Smartscape edges from matching records | Smartscape edge | All matches |
| 4 | Permission | Apply a security context to matching records | Set security context | First match only |
| 5 | Product allocation | Assign product or application usage | DPS Cost Allocation - Product | First match only |
| 6 | Cost allocation | Assign cost-center usage | DPS Cost Allocation - Cost Center | First match only |
| 7 | **Bucket assignment** | Choose the Grail bucket — or not to store the record | Bucket assignment, No storage assignment | First match only |
| 8 | Metric extraction | Extract metrics from matching records | Counter metric, Histogram metric, Value metric | All matches |
| 9 | Davis | Extract a Davis event and re-ingest it into another pipeline | Davis event | All matches |
| 10 | Data extraction | Extract a business event or SDLC event and re-ingest it | Business event, Software development lifecycle event | All matches |

Masking, dropping and parsing are not separate stages: they are processors in stage 1, and within a stage they run in the order you place them. Storage in the chosen Grail bucket happens after the pipeline, so a record's retention is set by stage 7 — which is also why the extraction stages (8–10) still see records assigned to **No storage assignment**.

> <sub>**Sources:** [Processing in OpenPipeline (DT docs)](https://docs.dynatrace.com/docs/platform/openpipeline/concepts/processing) — *"each processor output becomes the input for the next one"*, [OpenPipeline processing examples (DT docs)](https://docs.dynatrace.com/docs/platform/openpipeline/use-cases/processing-examples) — *"The record continues through all pipeline stages, including metric extraction, and is not stored only at the end."*</sub>

<a id="environment-summary"></a>
## 7. Environment Summary
Let's get a complete picture of your OpenPipeline environment:

```dql
// Complete environment summary
fetch logs, from: now() - 1h
| summarize {
    total_logs = count(),
    unique_sources = countDistinct(dt.openpipeline.source),
    unique_buckets = countDistinct(dt.system.bucket),
    unique_hosts = countDistinct(dt.entity.host),
    error_count = countIf(loglevel == "ERROR" OR status == "ERROR"),
    warn_count = countIf(loglevel == "WARN" OR status == "WARN")
  }
```

```dql
// Top log sources by entity
fetch logs, from: now() - 1h
| filter isNotNull(dt.entity.host)
| summarize {log_count = count()}, by: {host.name, log.source}
| sort log_count desc
| limit 15
```

---

<a id="summary"></a>
## 📝 Summary
In this notebook, you learned:

✅ **What OpenPipeline is** - Dynatrace's unified data processing framework  
✅ **Architecture** - Data flow from sources through processing to Grail  
✅ **Key fields** - `dt.openpipeline.source`, `dt.openpipeline.pipelines`, `dt.system.bucket`  
✅ **Data sources** - OneAgent, Log Ingest API, OTLP  
✅ **Pipeline stages** - a fixed sequence: Processing (mask, drop, parse, transform) → … → Bucket assignment → Metric / Davis / Data extraction  

---

<a id="next-steps"></a>
## ➡️ Next Steps
Continue to **OPLOGS-02: Migration Guide** to learn how to migrate from classic log ingestion to OpenPipeline v2.0.

---

<a id="references"></a>
## 📚 References
- [OpenPipeline (DT docs)](https://docs.dynatrace.com/docs/platform/openpipeline)
- [Processing in OpenPipeline (DT docs)](https://docs.dynatrace.com/docs/platform/openpipeline/concepts/processing)
- [Grail (DT docs)](https://docs.dynatrace.com/docs/platform/grail)
- [Dynatrace Query Language (DT docs)](https://docs.dynatrace.com/docs/platform/grail/dynatrace-query-language)
- [Automatic log enrichment (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/logs/lma-log-ingestion/lma-log-ingestion-via-api/lma-log-data-transformation) — *"a status attribute is created with a value that is a sum of loglevel values based on the following grouping"*
- [Log ingestion API (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/logs/lma-log-ingestion/lma-log-ingestion-via-api) — delivery reliability: HTTP 204 success, retryable codes + exponential backoff, ActiveGate disk-queue buffering, `503 Usable space limit reached`

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
