# OPLOGS-01: OpenPipeline Fundamentals

> **Series:** OPLOGS — OpenPipeline Logs | **Notebook:** 1 of 8 | **Created:** December 2025 | **Last Updated:** 06/30/2026

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
6. [Environment Summary](#environment-summary)
7. [📝 Summary](#summary)
8. [➡️ Next Steps](#next-steps)
9. [📚 References](#references)

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
| Data Masking | Limited | Full regex support |
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
| **3. Processing** | Mask · Drop · Transform · Parse · Extract (counter / value / histogram / Smartscape / bizevent / SDLC / Davis) · Cost / Security | All in-pipeline work — masking, filtering, transformation, and extraction are processor categories *within* this stage |
| **4. Storage** | Bucket assignment / No storage assignment | Persist to Grail bucket, or skip retention |
| **Output** | Grail | Logs, Spans, Metrics, Events storage |
-->

> **Doc alignment (May 2026):** Per the official `/concepts/data-flow` documentation, OpenPipeline has a **four-stage flow**: Ingest → Routing → Processing → Storage. The diagram above shows processor categories within Processing (Mask, Filter, Transform, Extract) inline so you can see the in-pipeline execution order — they are not independent stages.

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
| `status` | Status string (alternative to loglevel) |
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

## 6. Pipeline Stages Overview

OpenPipeline processes data through ordered stages:

### Stage 1: Routing
- Matches incoming data to the appropriate pipeline
- Based on source, content, or metadata

### Stage 2: Masking (Security)
- Redacts sensitive data BEFORE processing
- Protects PII, credentials, secrets
- Applied early for security compliance

### Stage 3: Filtering
- Drops unwanted records
- Reduces storage costs
- Removes noise (debug logs, health checks)

### Stage 4: Processing
- Parses structured data from content
- Adds enrichment fields
- Transforms and normalizes data

### Stage 5: Extraction
- Creates metrics from log data
- Generates events and business events

### Stage 6: Storage
- Routes to appropriate Grail bucket
- Applies retention policies

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
✅ **Pipeline stages** - Routing, Masking, Filtering, Processing, Extraction, Storage  

---

<a id="next-steps"></a>
## ➡️ Next Steps
Continue to **OPLOGS-02: Migration Guide** to learn how to migrate from classic log ingestion to OpenPipeline v2.0.

---

<a id="references"></a>
## 📚 References
- [OpenPipeline Documentation](https://docs.dynatrace.com/docs/platform/openpipeline)
- [Grail Data Lakehouse](https://docs.dynatrace.com/docs/platform/grail)
- [DQL Reference](https://docs.dynatrace.com/docs/platform/grail/dynatrace-query-language)
- [Log ingestion API (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/logs/lma-log-ingestion/lma-log-ingestion-via-api) — delivery reliability: HTTP 204 success, retryable codes + exponential backoff, ActiveGate disk-queue buffering, `503 Usable space limit reached`

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
