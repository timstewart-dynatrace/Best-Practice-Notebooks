# SPANS-02: Querying Spans with DQL

> **Series:** SPANS — Distributed Tracing and Spans | **Notebook:** 2 of 8 | **Created:** December 2025 | **Last Updated:** 05/21/2026

## Mastering Span Queries in Dynatrace
This notebook covers essential techniques for querying and filtering span data to find exactly what you need. You'll learn to filter by service, operation, and attributes to quickly locate relevant traces.

---

## Table of Contents

1. [DQL is NOT SQL!](#dql-is-not-sql)
2. [Filtering by Service](#filtering-by-service)
3. [Filtering by Span Kind](#filtering-by-span-kind)
4. [Filtering by Operation Name](#filtering-by-operation-name)
5. [String Matching Functions](#string-matching-functions)
6. [Finding Specific Traces](#finding-specific-traces)
7. [HTTP Span Queries](#http-span-queries)
8. [Database Span Queries](#database-span-queries)
9. [Working with NULL Values](#working-with-null-values)
10. [Combining Multiple Filters](#combining-multiple-filters)

---


## Prerequisites

Before starting this notebook, ensure you have:

- ✅ Completed **SPANS-01: Fundamentals**
- ✅ Access to a Dynatrace environment with span data
- ✅ DQL query permissions

<a id="dql-is-not-sql"></a>
## 1. DQL is NOT SQL!
⚠️ **CRITICAL:** DQL has different syntax from SQL. Memorize these differences:

![DQL Pipeline Model](images/02-dql-pipeline-spans.png)

<!--MARKDOWN_TABLE_ALTERNATIVE
| DQL Stage | Purpose | Example |
|-----------|---------|---------|
| fetch | Retrieve data source | fetch spans |
| filter | Narrow down records | filter span.kind == "server" |
| fieldsAdd | Compute new fields | fieldsAdd duration_ms = duration / 1ms |
| summarize | Aggregate data | summarize count(), by:{service.name} |
| sort | Order results | sort duration desc |
| limit | Restrict output | limit 100 |
-->

### Key Differences

| Concept | SQL | DQL |
|---------|-----|-----|
| **Arrays** | `('a', 'b')` parentheses | `{"a", "b"}` curly braces |
| **Comparison** | `=` single equals | `==` double equals |
| **String quotes** | `'single quotes'` | `"double quotes"` |
| **NULL checks** | `IS NULL` / `IS NOT NULL` | `isNull()` / `isNotNull()` |
| **Membership** | `IN (...)` | `in(field, {...})` |
| **Grouping** | `GROUP BY` | `by: {...}` in summarize |

---

<a id="filtering-by-service"></a>
## 2. Filtering by Service
Filter spans to focus on specific services. Use `dt.entity.service` for reliable filtering (entity ID), or `service.name` for display name.

> 💡 **Tip:** `dt.entity.service` is always populated and indexed. `service.name` may not always be available.

```dql
// Filter spans for a specific service using exact match
fetch spans, from:-1h
| filter service.name == "checkout"
| fields start_time, span.name, span.kind, duration
| sort start_time desc
| limit 50
```

```dql
// Use in() with curly braces {} for multiple values (NOT parentheses!)
fetch spans, from:-1h
| filter in(service.name, {"checkout", "payment", "cart"})
| fields start_time, service.name, span.name, span.kind, duration
| sort start_time desc
| limit 100
```

```dql
// Count spans by service entity (more reliable)
fetch spans, from:-1h
| filter isNotNull(dt.entity.service)
| summarize {span_count = count()}, by: {dt.entity.service}
| sort span_count desc
| limit 20
```

---

<a id="filtering-by-span-kind"></a>
## 3. Filtering by Span Kind
Filter spans based on their role in the distributed transaction.

⚠️ **IMPORTANT:** `span.kind` values are **lowercase**!

| Kind | Description | Use Case |
|------|-------------|----------|
| `"server"` | Handles incoming request | Find inbound API calls |
| `"client"` | Makes outgoing request | Find calls to dependencies |
| `"internal"` | Internal processing | Find business logic |
| `"producer"` | Sends async message | Find message publishers |
| `"consumer"` | Receives async message | Find message consumers |

```dql
// Find all SERVER spans (inbound requests)
// Note: "server" is lowercase, not "SERVER"
fetch spans, from:-1h
| filter span.kind == "server"
| fields start_time, service.name, span.name, duration
| sort duration desc
| limit 50
```

```dql
// Find CLIENT spans (outbound calls to dependencies)
fetch spans, from:-1h
| filter span.kind == "client"
| fields start_time, service.name, span.name, duration
| sort duration desc
| limit 50
```

```dql
// Count spans by kind to understand your traffic patterns
fetch spans, from:-1h
| summarize {span_count = count()}, by: {span.kind}
| sort span_count desc
```

---

<a id="filtering-by-operation-name"></a>
## 4. Filtering by Operation Name
Find spans for specific operations or endpoints using the `span.name` attribute:

```dql
// Find spans for a specific operation/endpoint
fetch spans, from:-1h
| filter contains(span.name, "checkout")
| fields start_time, service.name, trace.id, span.name, duration, span.status_code
| sort start_time desc
| limit 50
```

```dql
// Find all POST operations (write operations)
fetch spans, from:-1h
| filter startsWith(span.name, "POST")
| fields start_time, service.name, span.name, duration, span.status_code
| sort start_time desc
| limit 50
```

### 4.1. Endpoint-Name Span Identity (Enhanced Endpoints)

Since Dynatrace v1.329, the **Enhanced Endpoints for SDv1** setting changes what populates `span.name` for HTTP server spans on auto-instrumented services. Environments created at v1.333+ have it always on; older environments may need to enable it explicitly at *Settings → Process and contextualize → Services → Service detection v1*.

**Practical impact on DQL queries:**

| Before Enhanced Endpoints | After Enhanced Endpoints |
|---|---|
| `span.name == "GET /users/42"` matched one literal path | `span.name == "GET /users/{id}"` matches the endpoint template; the per-request URL goes elsewhere |
| Every distinct URL produced a distinct `span.name` value | Endpoint templates collapse path-variable variations into one named endpoint |
| Metrics aggregated into `NON_KEY_REQUESTS` for unmarked requests | Each detected endpoint emits individual `dt.service.request.*` metrics |

**Filtering rule of thumb:**

- Use `contains(span.name, "/users")` or `startsWith(span.name, "GET /users")` for portable filtering — both forms work before and after the setting is enabled.
- Exact equality on a literal URL (`span.name == "GET /users/42"`) only worked when endpoints weren't templated. Switch to `contains()` on the path stem.
- For services without the `http.route` span attribute (often Nginx, Apache, IIS in front of a backend), endpoints may collapse to `GET /*`. Use request-naming rules to recover named endpoints — see [Enhanced endpoints for SDv1 (DT docs)](https://docs.dynatrace.com/docs/observe/application-observability/services/service-detection/service-detection-v1/enhanced-endpoints-sdv1).

**Services not affected:** external services, background activity, queue listeners, key-value stores — Enhanced Endpoints does not create endpoints for these.

> <sub>**Sources:** [Enhanced endpoints for SDv1 (DT docs)](https://docs.dynatrace.com/docs/observe/application-observability/services/service-detection/service-detection-v1/enhanced-endpoints-sdv1).</sub>

---

<a id="string-matching-functions"></a>
## 5. String Matching Functions
DQL provides several string matching functions:

| Function | Description | Example |
|----------|-------------|----------|
| `contains(field, "text")` | Substring match | `contains(span.name, "user")` |
| `startsWith(field, "text")` | Prefix match | `startsWith(span.name, "GET")` |
| `endsWith(field, "text")` | Suffix match | `endsWith(url.path, ".json")` |
| `matchesPhrase(field, "pattern")` | Wildcard pattern | `matchesPhrase(span.name, "GET /api/*")` |
| `in(field, {"a", "b"})` | Multiple values | `in(span.kind, {"server", "client"})` |

```dql
// Contains - partial match anywhere in the string
fetch spans, from:-1h
| filter contains(span.name, "Get")
| fields span.name, service.name
| dedup span.name
| limit 20
```

```dql
// startsWith and endsWith - prefix/suffix matching
fetch spans, from:-1h
| filter startsWith(span.name, "GET") or endsWith(span.name, "query")
| fields span.name
| dedup span.name
| limit 20
```

```dql
// matchesPhrase - wildcard pattern matching with *
fetch spans, from:-1h
| filter matchesPhrase(span.name, "GET /api/*")
| fields span.name
| dedup span.name
| limit 20
```

```dql
// Use dedup to see unique span names per service
fetch spans, from:-1h
| filter span.kind == "server"
| fields service.name, span.name
| dedup service.name, span.name
| sort service.name asc
| limit 50
```

---

<a id="finding-specific-traces"></a>
## 6. Finding Specific Traces
Locate all spans belonging to a specific trace:

```dql
// First, find some trace IDs to work with
fetch spans, from:-1h
| filter span.kind == "server"
| fields start_time, trace.id, span.name, service.name
| sort start_time desc
| limit 10
```

```dql
// Find all spans for a specific trace ID
// Replace YOUR_TRACE_ID with an actual trace.id from above
fetch spans, from:-1h
// | filter trace.id == "YOUR_TRACE_ID"
| fields start_time, span.id, span.parent_id, span.name, service.name, duration
| sort start_time asc
| limit 100
```

```dql
// Find root spans (entry points) - spans without a parent
fetch spans, from:-1h
| filter isNull(span.parent_id)
| fields start_time, trace.id, span.name, service.name, duration, span.status_code
| sort start_time desc
| limit 50
```

---

<a id="http-span-queries"></a>
## 7. HTTP Span Queries
Query HTTP-specific span attributes for API troubleshooting:

| Attribute | Description |
|-----------|-------------|
| `http.request.method` | HTTP method (GET, POST, etc.) |
| `http.response.status_code` | HTTP status code (200, 404, 500) |
| `http.route` | URL route pattern (use this, not url.path for aggregation) |
| `url.path` | Full URL path (may contain PII) |

```dql
// Query HTTP spans with response status codes
fetch spans, from:-1h
| filter isNotNull(http.response.status_code)
| fields start_time, 
         service.name, 
         http.request.method, 
         http.route,
         http.response.status_code,
         duration
| sort start_time desc
| limit 100
```

```dql
// Find HTTP 5xx errors (server errors)
fetch spans, from:-1h
| filter http.response.status_code >= 500 
      and http.response.status_code < 600
| fields start_time, 
         service.name, 
         http.request.method, 
         http.route,
         http.response.status_code,
         span.status_message,
         duration
| sort start_time desc
| limit 50
```

```dql
// Summarize HTTP status codes by route
fetch spans, from:-1h
| filter isNotNull(http.response.status_code)
| summarize {status_count = count()}, by: {http.response.status_code}
| sort http.response.status_code asc
```

---

<a id="database-span-queries"></a>
## 8. Database Span Queries
Analyze database operations captured as spans:

| Attribute | Description |
|-----------|-------------|
| `db.system` | Database type (mysql, postgresql, redis) |
| `db.name` | Database name |
| `db.operation` | Operation type (SELECT, INSERT, UPDATE) |
| `db.statement` | The database query (may contain sensitive data) |

```dql
// Find all database spans
fetch spans, from:-1h
| filter isNotNull(db.system)
| fields start_time,
         service.name,
         db.system,
         db.name,
         db.operation,
         duration
| sort duration desc
| limit 50
```

```dql
// Find slow database queries (over 100ms)
fetch spans, from:-1h
| filter isNotNull(db.system) 
      and duration > 100ms
| fieldsAdd duration_ms = duration / 1ms
| fields start_time,
         service.name,
         db.system,
         db.operation,
         db.statement,
         duration_ms
| sort duration_ms desc
| limit 50
```

```dql
// Summarize database usage
fetch spans, from:-1h
| filter isNotNull(db.system)
| summarize {
    db_span_count = count(),
    avg_duration_ms = avg(duration) / 1ms
  }, by: {db.system, db.name}
| sort db_span_count desc
```

---

<a id="working-with-null-values"></a>
## 9. Working with NULL Values
⚠️ **DQL uses tri-state boolean logic.** Comparisons with NULL don't work like SQL!

![NULL Handling in DQL](images/02-null-handling-dql.png)

<!--MARKDOWN_TABLE_ALTERNATIVE
| Expression | Returns | Explanation |
|------------|---------|-------------|
| `field == null` | NULL | Does NOT return true! |
| `field != null` | NULL | Does NOT return true! |
| `isNull(field)` | true/false | Returns true if field is null |
| `isNotNull(field)` | true/false | Returns true if field is NOT null |
-->

```dql
// Find spans that have database information
fetch spans, from:-1h
| filter isNotNull(db.system)
| summarize {db_span_count = count()}, by: {db.system, db.name}
| sort db_span_count desc
```

```dql
// Find HTTP spans with missing route (potential instrumentation issue)
fetch spans, from:-1h
| filter isNotNull(http.request.method) and isNull(http.route)
| fields service.name, span.name, http.request.method, url.path
| dedup service.name, span.name
| limit 20
```

---

<a id="combining-multiple-filters"></a>
## 10. Combining Multiple Filters
Build complex queries by combining multiple filter conditions:

> 💡 **Performance Tip:** Apply more restrictive filters first for better query performance.

```dql
// Complex filter: Find slow SERVER spans in the checkout service
fetch spans, from:-1h
| filter service.name == "checkout"
      and span.kind == "server"
      and duration > 500ms
| fieldsAdd duration_ms = duration / 1ms
| fields start_time,
         span.name,
         duration_ms,
         span.status_code
| sort duration_ms desc
| limit 50
```

```dql
// Find error spans for specific services and operations
fetch spans, from:-1h
| filter span.status_code == "error"
      and in(service.name, {"payment", "checkout"})
      and span.kind == "server"
| fieldsAdd duration_ms = duration / 1ms
| fields start_time,
         service.name,
         span.name,
         span.status_message,
         trace.id,
         duration_ms
| sort start_time desc
| limit 50
```

```dql
// Find spans: either server errors OR slow successful requests
fetch spans, from:-1h
| filter http.response.status_code >= 500
      or (http.response.status_code >= 200 
          and http.response.status_code < 300 
          and duration > 1s)
| fieldsAdd duration_ms = duration / 1ms
| fields start_time,
         service.name,
         http.request.method,
         http.route,
         http.response.status_code,
         duration_ms
| sort start_time desc
| limit 100
```

---

## Summary

In this notebook, you learned:

✅ **DQL ≠ SQL** - Critical syntax differences (arrays use `{}`, use `==`, `isNull()`)  
✅ **Filter by service** using exact match, `in()`, and pattern matching  
✅ **Filter by span kind** - values are lowercase (`"server"`, `"client"`)  
✅ **String matching** - `contains()`, `startsWith()`, `endsWith()`, `matchesPhrase()`  
✅ **Find traces** by trace.id and identify root spans  
✅ **Query HTTP spans** including status codes, methods, routes  
✅ **Analyze database spans** to find slow queries  
✅ **Handle NULL values** with `isNull()` / `isNotNull()`  
✅ **Use `dedup`** to see unique values  
✅ **Combine filters** for complex, precise queries  

---

## Next Steps

Continue to **SPANS-03: Trace Analysis & Troubleshooting** to learn:
- Identifying error patterns and failure points
- Latency analysis across services
- Root cause analysis techniques
- Tracing request flows through your system

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
