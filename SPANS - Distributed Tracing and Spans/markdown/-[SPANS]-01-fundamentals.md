# SPANS-01: Spans & Distributed Tracing Fundamentals

> **Series:** SPANS — Distributed Tracing and Spans | **Notebook:** 1 of 8 | **Created:** December 2025 | **Last Updated:** 10/02/2026

## Understanding the Building Blocks of Observability

This notebook introduces distributed tracing concepts and demonstrates how to query span data in Dynatrace using DQL. You'll learn what spans are, how they form traces, and how to explore your distributed systems.


> 🆕 **New Addition (March 2026)**
>
> **Companion Series**
> - **OPIPE** — Processing spans at ingestion (filtering, enrichment, metric extraction)? See **OPIPE-02: Span Processing & Enrichment**
> - **OPLOGS** — For OpenPipeline log processing, see **OPLOGS-01: OpenPipeline Fundamentals**

---

## Table of Contents

1. [What is Distributed Tracing?](#what-is-distributed-tracing)
2. [Understanding Spans](#understanding-spans)
3. [Span Anatomy](#span-anatomy)
4. [Span Kinds](#span-kinds)
5. [Trace Structure](#trace-structure)
6. [Your First Span Query](#your-first-span-query)

---


## Prerequisites

Before starting this notebook, ensure you have:

- ✅ Access to a Dynatrace environment with span data
- ✅ Permission to read spans (`storage:spans:read`)
- ✅ Basic understanding of microservices architecture

<a id="what-is-distributed-tracing"></a>
## 1. What is Distributed Tracing?
In modern microservices architectures, a single user request often travels through dozens of services. **Distributed tracing** captures this journey, showing:

- The **path** a request takes through your system
- **Timing** of each operation along the way
- **Dependencies** between services
- Where **errors** and **latency** occur

![Distributed Tracing Flow](images/01-distributed-tracing-flow.png)

<!--MARKDOWN_TABLE_ALTERNATIVE
| Step | Component | Action |
|------|-----------|--------|
| 1 | Browser | User initiates checkout request |
| 2 | Frontend | Routes request, creates root span |
| 3 | Cart Service | Retrieves cart items |
| 4 | Payment Service | Processes payment |
| 5 | Inventory | Updates stock levels |
| 6 | Database | Persists transaction |
-->

Without distributed tracing, debugging issues in this flow would require correlating logs from each service manually—an error-prone and time-consuming process.

<a id="understanding-spans"></a>
## 2. Understanding Spans
A **span** represents a single unit of work in a distributed system. Think of it as a timer that captures:

- **What** operation was performed
- **When** it started and ended
- **How long** it took
- **Whether** it succeeded or failed
- **Context** about the operation (HTTP method, database query, etc.)

### Key Span Concepts

| Concept | Description | Example |
|---------|-------------|----------|
| **Trace** | Collection of spans forming a request flow | Checkout transaction |
| **Span** | Single operation within a trace | Database query |
| **Root Span** | First span in a trace (no parent) | HTTP request to frontend |
| **Child Span** | Span triggered by another span | Service calling database |

<a id="span-anatomy"></a>
## 3. Span Anatomy
Every span contains these essential fields:

![Span Anatomy](images/01-span-anatomy.png)

<!--MARKDOWN_TABLE_ALTERNATIVE
| Field | Description |
|-------|-------------|
| trace.id | Links span to its parent trace |
| span.id | Unique identifier for this span |
| span.parent_id | ID of the calling span (null for root) |
| span.name | Operation name (e.g., "POST /checkout") |
| span.kind | Role: server, client, internal, etc. |
| start_time | When the operation started |
| end_time | When the operation completed |
| duration | How long it took (duration type, nanosecond precision) |
| span.status_code | ok or error; absent when the status is unset |
| attributes{} | Additional context (HTTP, DB, custom) |
-->

### Core Span Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `trace.id` | uid | Unique identifier linking all spans in a trace (16 bytes, hex when shown as a string) |
| `span.id` | uid | Unique identifier for this specific span (8 bytes) |
| `span.parent_id` | uid | ID of parent span (null for root spans) |
| `span.name` | string | Operation name (e.g., "GET /api/products") |
| `start_time` | timestamp | When the span started |
| `end_time` | timestamp | When the span ended |
| `duration` | duration | `end_time` − `start_time`, nanosecond precision — compare it with duration literals (`duration > 100ms`), never bare integers |

### Service Context Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `dt.service.name` | string | Dynatrace service name from service detection — present on every span |
| `dt.smartscape.service` | smartscape ID | Service ID for joins (`dt.entity.service` is the `deprecated` predecessor) |
| `service.name` | string | Logical service name set by OpenTelemetry SDKs — absent on OneAgent spans |
| `service.namespace` | string | Optional OpenTelemetry grouping that scopes `service.name` (`experimental`) |

> **Which service field?** On the validation tenant, `dt.service.name` was set on all 322,799 spans in an hour and `service.name` on 21,586 (6.7%) — the OpenTelemetry ones. Grouping by `service.name` puts every OneAgent span in one null row, so this series uses `dt.service.name`. Names are not unique, though: on the validation tenant `dt.service.name == "frontend"` covered **4** different services (and `image-provider` 2), from different applications. Where identity matters — error counts per service, dependency maps — group by `dt.smartscape.service`, or by both.

### Status Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `span.status_code` | string | `ok` or `error` — the field is **absent** when the status is unset, which is most spans |
| `span.status_message` | string | Optional error text when the status is `error` (`experimental`) |

Because unset is stored as a missing field, `span.status_code != "error"` does not count successes — on the validation tenant 307,312 of 318,628 spans in an hour had no status at all, against 240 `ok` and 11,076 `error`. Count failures with `span.status_code == "error"` and derive successes as total minus errors.

> <sub>**Sources:** [Trace semantic conventions (DT docs)](https://docs.dynatrace.com/docs/semantic-dictionary/model/trace) — *"The span status is only present if it is explicitly set to error or ok."* **Dictionary:** `trace.id`/`span.id`/`span.parent_id` (`stable`, `uid`), `duration` (`stable`, `duration`), `span.status_code` (`stable`), `span.status_message` (`experimental`), `dt.service.name` (`stable`), `service.name` (`stable`), `service.namespace` (`experimental`), `dt.entity.service` (`deprecated`), read 10/02/2026.</sub>

<a id="span-kinds"></a>
## 4. Span Kinds
The `span.kind` attribute indicates the span's role in the distributed transaction:

![Span Kinds](images/01-span-kinds.png)

<!--MARKDOWN_TABLE_ALTERNATIVE
| Kind | Description | Example |
|------|-------------|---------|
| server | Inbound request to a service | HTTP request handler |
| client | Outbound call to another service | HTTP client request |
| internal | Internal operation within a service | Business logic execution |
| producer | Sending a message to a queue | Kafka producer |
| consumer | Receiving a message from a queue | Kafka consumer |
-->

> 💡 **Tip:** When a service calls another service, you'll see a CLIENT span on the caller and a SERVER span on the receiver. Both spans share the same `trace.id`.

<a id="trace-structure"></a>
## 5. Trace Structure
A **trace** is a tree of spans connected by parent-child relationships:

![Trace Tree](images/01-trace-tree.png)

<!--MARKDOWN_TABLE_ALTERNATIVE
| Service | Operation | Duration | Parent |
|---------|-----------|----------|--------|
| Frontend | POST /checkout | 450ms | (root) |
| Cart Service | GetCart | 45ms | Frontend |
| Redis | GET cart:user123 | 5ms | Cart Service |
| Payment Service | ProcessPayment | 280ms | Frontend |
| Fraud Check | ValidateCard | 120ms | Payment Service |
| Payment Gateway | Charge | 150ms | Payment Service |
| Inventory | ReserveItems | 35ms | Frontend |
| PostgreSQL | UPDATE | 12ms | Inventory |
-->

### Parent-Child Relationships

- Each span (except root) has a `span.parent_id` pointing to its parent
- Root spans have `span.parent_id = null`
- All spans in a trace share the same `trace.id`
- A child span normally starts at or after its parent — but spans from different hosts carry their own clocks, so small skews can make a child appear to start first

<a id="your-first-span-query"></a>
## 6. Your First Span Query
Let's explore span data using DQL. We'll start with the most basic query and progressively add complexity.

> ⚠️ **Important:** Always use `limit` when exploring data to avoid processing millions of spans.

```dql
// Basic span query - fetch spans from the last hour
fetch spans, from:-1h
| limit 100
```

### Selecting Specific Fields

Instead of retrieving all span attributes, select only the fields you need for better performance and readability:

```dql
// Select specific span fields for analysis
fetch spans, from:-1h
| fields start_time,
         trace.id,
         span.id,
         span.name,
         span.kind,
         dt.service.name,
         duration,
         span.status_code
| sort start_time desc
| limit 50
```

### Understanding Duration

`duration` is a **duration** value with nanosecond precision. Divide it by a duration literal to get a plain number in the unit you want:

> 💡 **Tip:** `duration / 1ms` gives milliseconds as a number. `duration / 1000000` does not — dividing a duration by a plain number is still a duration.

```dql
// Convert duration from nanoseconds to milliseconds and seconds
fetch spans, from:-1h
| fields start_time,
         span.name,
         dt.service.name,
         duration,
         duration_ms = duration / 1ms,     // Convert to milliseconds
         duration_sec = duration / 1s  // Convert to seconds
| sort duration desc
| limit 20
```

### Filtering Spans

Use filters to focus on specific spans. Filter early in your query for better performance:

```dql
// Filter spans by service and span kind
fetch spans, from:-1h
| filter span.kind == "server"
| filter duration > 100ms
| fields start_time,
         dt.service.name,
         span.name,
         duration_ms = duration / 1ms,
         span.status_code
| sort duration_ms desc
| limit 50
```

### Discovering Services in Your Environment

Find all services that are generating span data:

```dql
// Discover all services with span data
fetch spans, from:-1h
| filter span.kind == "server"
| summarize {span_count = count()}, by:{dt.service.name}
| sort span_count desc
| limit 50
```

---

## Summary

In this notebook, you learned:

✅ **What distributed tracing is** and why it's essential for modern architectures  
✅ **What spans are** and their core attributes  
✅ **Span anatomy** including trace.id, span.id, duration, and status  
✅ **Span kinds** (server, client, internal, producer, consumer)  
✅ **Trace structure** with parent-child relationships  
✅ **Basic DQL queries** to fetch and explore span data  
✅ **Duration conversion** with duration literals (`duration / 1ms`)  

---

## Next Steps

Continue to **SPANS-02: Querying Spans with DQL** to learn:
- Filtering spans by service, operation, and attributes
- Finding specific traces by trace.id
- Querying HTTP and database spans
- Combining multiple filters for precise analysis

---

## References

- [Trace semantic conventions (DT docs)](https://docs.dynatrace.com/docs/semantic-dictionary/model/trace)
- [Distributed traces (DT docs)](https://docs.dynatrace.com/docs/observe/application-observability/distributed-traces)
- [Traces (opentelemetry.io)](https://opentelemetry.io/docs/concepts/signals/traces/)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
