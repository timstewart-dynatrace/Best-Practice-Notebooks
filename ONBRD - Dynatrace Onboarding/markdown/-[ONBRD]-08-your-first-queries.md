# ONBRD-08: Your First Queries

> **Series:** ONBRD — Dynatrace Onboarding | **Notebook:** 8 of 10 | **Created:** December 2025 | **Last Updated:** 08/03/2026

## Learning DQL Fundamentals
Dynatrace Query Language (DQL) is how you access data in Grail. This notebook introduces the core concepts and patterns you'll use daily.

---

## Table of Contents

1. [DQL Basics](#dql-basics)
2. [The Pipeline Model](#the-pipeline-model)
3. [Fetching Data](#fetching-data)
4. [Filtering](#filtering)
5. [Selecting Fields](#selecting-fields)
6. [Aggregating with Summarize](#aggregating-with-summarize)
7. [Sorting and Limiting](#sorting-and-limiting)
8. [Time Ranges](#time-ranges)
9. [Common Patterns](#common-patterns)

---

## Prerequisites

- Dynatrace environment with data (ONBRD-05, ONBRD-06, ONBRD-07)
- DQL query permissions
- Access to Notebooks or the DQL query interface

<a id="dql-basics"></a>
## 1. DQL Basics
DQL is a **pipeline-based query language**—not SQL. Data flows through a series of commands connected by the pipe (`|`) operator.

![DQL Pipeline](images/08-dql-pipeline.png)
<!-- MARKDOWN_TABLE_ALTERNATIVE
| Stage | Command | Result |
|-------|---------|--------|
| 1 | fetch logs | All logs |
| 2 | filter status | Only errors |
| 3 | fields select | Just fields we need |
| 4 | sort order | Ordered output |
-->

### DQL vs SQL

| DQL | SQL | Note |
|-----|-----|------|
| `fetch logs` | `SELECT * FROM logs` | Start with data source |
| `\| filter x == "y"` | `WHERE x = 'y'` | Use `==`, double quotes |
| `\| fields a, b` | `SELECT a, b` | Field selection after fetch |
| `\| summarize count()` | `SELECT COUNT(*)` | Aggregation command |
| `by: {field}` | `GROUP BY field` | Grouping syntax |
| `{"a", "b"}` | `('a', 'b')` | Array syntax (curly braces) |

<a id="the-pipeline-model"></a>
## 2. The Pipeline Model
Each command in the pipeline operates on the output of the previous command:

```dql
fetch logs                        // 1. Get all logs
| filter loglevel == "error"     // 2. Keep only errors
| filter timestamp > now() - 1h  // 3. Last hour only
| fields timestamp, content      // 4. Select columns
| sort timestamp desc            // 5. Order by time
| limit 100                      // 6. Take first 100
```

### Order Matters

- **Filter early** - Reduces data before expensive operations
- **Select fields** - Reduces memory usage
- **Aggregate** - Summarize before sorting
- **Sort** - Order the final results
- **Limit** - Control output size

<a id="fetching-data"></a>
## 3. Fetching Data
Every DQL query starts with a `fetch` command specifying the data source.

```dql
// Fetch logs (most common)
fetch logs, from:-1h
| limit 10
```

```dql
// Fetch spans (distributed traces)
fetch spans, from:-1h
| limit 10
```

```dql
// Fetch entity data (hosts)
fetch dt.entity.host
| limit 10

// Smartscape equivalent (dt.entity.* is deprecated but still functional):
//   smartscapeNodes "HOST"
//   | limit 10
// Caveat: Smartscape reflects CURRENT live topology and can report fewer entities
// than the classic entity store; for a pre-migration discovery inventory keep the
// classic query above.
```

```dql
// Fetch problems
fetch dt.davis.problems, from:-24h
| limit 10
```

### Common Data Sources

| Source | Description |
|--------|-------------|
| `logs` | Log records |
| `spans` | Distributed trace spans |
| `events` | System events |
| `bizevents` | Business events |
| `dt.entity.host` | Host entities |
| `dt.entity.service` | Service entities |
| `dt.entity.process_group` | Process group entities |
| `dt.davis.problems` | Detected problems |

<a id="filtering"></a>
## 4. Filtering
Use `filter` to narrow results. Filter as early as possible for performance.

```dql
// Filter by equality
fetch logs, from:-1h
| filter loglevel == "error"
| limit 20
```

```dql
// Filter with multiple conditions (AND)
fetch logs, from:-1h
| filter loglevel == "error"
| filter timestamp > now() - 1h
| limit 20
```

```dql
// Filter with or condition
fetch logs, from:-1h
| filter loglevel == "error" or loglevel == "warn"
| limit 20
```

```dql
// Filter using IN for multiple values
fetch logs, from:-1h
| filter in(loglevel, {"error", "warn", "fatal"})
| limit 20
```

```dql
// Filter with string matching
fetch logs, from:-1h
| filter contains(content, "timeout")
| limit 20
```

### Filter Operators

| Operator | Example | Description |
|----------|---------|-------------|
| `==` | `field == "value"` | Equals |
| `!=` | `field != "value"` | Not equals |
| `>`, `<` | `count > 10` | Greater/less than |
| `>=`, `<=` | `count >= 10` | Greater/less or equal |
| `and`, `or` | `a == 1 and b == 2` | Logical operators |
| `in()` | `in(field, {"a", "b"})` | Value in set |
| `contains()` | `contains(field, "text")` | Substring match |
| `isNull()` | `isNull(field)` | Field is null |
| `isNotNull()` | `isNotNull(field)` | Field is not null |

<a id="selecting-fields"></a>
## 5. Selecting Fields
Use `fields` to select specific columns. This improves readability and performance.

```dql
// Select specific fields from logs
fetch logs, from:-1h
| fields timestamp, loglevel, log.source, content
| limit 20
```

```dql
// Create calculated fields (duration / 1ms uses duration arithmetic — preferred)
fetch spans, from:-1h
| fields span.name, 
         duration,
         duration_ms = duration / 1ms
| limit 20
```

```dql
// Rename fields with aliases
fetch dt.entity.host
| fields name = entity.name,
         status = state,
         os = osType
| limit 20

// Smartscape note (dt.entity.* is deprecated but still functional): this query uses the
// classic-only fields state / monitoringMode, which have NO Smartscape node equivalent
// (Smartscape expresses liveness via node lifetime, not a state field). Keep the classic
// query above for state / monitoring-mode detail.
// Other fields do map: osType -> os.type (LINUX -> OS_TYPE_LINUX); entity.name -> name.
```

### fieldsAdd vs fields

| Command | Effect |
|---------|--------|
| `fields` | Keeps only specified fields |
| `fieldsAdd` | Adds new fields, keeps all existing |

```dql
// fieldsAdd keeps existing fields and adds new ones
fetch spans, from:-1h
| fieldsAdd duration_ms = duration / 1ms
| fields span.name, duration, duration_ms
| limit 10
```

<a id="aggregating-with-summarize"></a>
## 6. Aggregating with Summarize
Use `summarize` to aggregate data. Combine with `by:` for grouping.

```dql
// Simple count
fetch logs, from: now() - 1h
| summarize total_logs = count()
```

```dql
// Count by group
fetch logs, from: now() - 1h
| summarize log_count = count(), by: {loglevel}
| sort log_count desc
```

```dql
// Multiple aggregations
fetch spans, from: now() - 1h
| filter span.kind == "server"
| summarize 
    request_count = count(),
    avg_duration = avg(duration),
    max_duration = max(duration),
    by: {service.name}
| sort request_count desc
| limit 20
```

```dql
// Conditional counting
fetch spans, from: now() - 1h
| filter span.kind == "server"
| summarize
    total = count(),
    errors = countIf(span.status_code == "error"),
    by: {service.name}
| fieldsAdd error_rate = 100.0 * errors / total
| sort error_rate desc
| limit 20
```

### Common Aggregation Functions

| Function | Description |
|----------|-------------|
| `count()` | Count records |
| `countIf(condition)` | Count where condition is true |
| `sum(field)` | Sum values |
| `avg(field)` | Average value |
| `min(field)` | Minimum value |
| `max(field)` | Maximum value |
| `percentile(field, 95)` | 95th percentile |

<a id="sorting-and-limiting"></a>
## 7. Sorting and Limiting
Control output order and size.

```dql
// Sort descending (newest first)
fetch logs, from: now() - 1h
| fields timestamp, loglevel, content
| sort timestamp desc
| limit 20
```

```dql
// Sort by multiple fields
fetch spans, from: now() - 1h
| filter span.kind == "server"
| summarize request_count = count(), by: {service.name}
| sort request_count desc
| limit 10
```

```dql
// Find slowest spans
fetch spans, from: now() - 1h
| fields span.name, service.name, duration
| sort duration desc
| limit 10
```

<a id="time-ranges"></a>
## 8. Time Ranges
Control the time range for your queries.

```dql
// Last hour (using the from: parameter on fetch)
fetch logs, from: now() - 1h
| summarize count()
```

```dql
// Specific time range
fetch logs, from: now() - 24h, to: now() - 12h
| summarize count()
```

```dql
// Last 7 days
fetch dt.davis.problems, from: now() - 7d
| summarize problem_count = count(), by: {event.status}
```

### Time Units

| Unit | Example | Description |
|------|---------|-------------|
| `s` | `now() - 30s` | Seconds |
| `m` | `now() - 15m` | Minutes |
| `h` | `now() - 2h` | Hours |
| `d` | `now() - 7d` | Days |

<a id="common-patterns"></a>
## 9. Common Patterns
Here are patterns you'll use frequently.

### Error Investigation

```dql
// Find error logs with context
fetch logs, from: now() - 1h
| filter loglevel == "error"
| fields timestamp, log.source, content
| sort timestamp desc
| limit 50
```

### Service Performance

```dql
// Service response time summary (use duration / 1ms for ns→ms conversion)
fetch spans, from: now() - 1h
| filter span.kind == "server"
| summarize {
    requests = count(),
    avg_ms = avg(duration) / 1ms,
    p95_ms = percentile(duration, 95) / 1ms
  }, by: {service.name}
| sort requests desc
| limit 20
```

### Log Volume Analysis

```dql
// Log volume by source and severity
fetch logs, from: now() - 1h
| summarize log_count = count(), by: {log.source, loglevel}
| sort log_count desc
| limit 30
```

### Entity Inventory

```dql
// Host inventory with details
fetch dt.entity.host
| fields 
    name = entity.name,
    state,
    os = osType,
    cores = cpuCores
| sort name
| limit 50

// Smartscape note (dt.entity.* is deprecated but still functional): this query uses the
// classic-only fields state / monitoringMode, which have NO Smartscape node equivalent
// (Smartscape expresses liveness via node lifetime, not a state field). Keep the classic
// query above for state / monitoring-mode detail.
// Other fields do map: osType -> os.type (LINUX -> OS_TYPE_LINUX); cpuCores -> cores; entity.name -> name.
```

## 10. Next Steps

With DQL fundamentals covered:

1. **ONBRD-09: Setting Up Alerts** — Configure alerting and notifications
2. Practice with your own data
3. Explore the DQL documentation for advanced functions
4. Try creating queries in Notebooks

### Beyond the Basics

- **`makeTimeseries`** — convert event-based data (logs, spans, bizevents) into time-bucketed metric series. Different from the `timeseries` command, which queries pre-ingested metrics. Example: `fetch logs | makeTimeseries error_rate = countIf(loglevel == "ERROR"), interval:5m, by:{k8s.cluster.name}`
- **Iterative array expressions** — `arrayAvg`, `arraySum`, `iAny`, etc. for working with timeseries arrays in `fieldsAdd` and `filter`
- **Smartscape topology navigation** — `smartscapeNodes`, `smartscapeEdges`, `traverse` for entity relationships (the modern alternative to `dt.entity.*`)

### Where to Go Deeper

- **OPLOGS / OPMIG / OPIPE** — Log query patterns, OpenPipeline transformations
- **SPANS series** — Span-specific DQL patterns
- **DASH series** — Building dashboards from DQL queries
- **AIOPS series** — Davis problem queries, anomaly detection DQL

### DQL Checklist

- [ ] Understand the pipeline model
- [ ] Can fetch different data types
- [ ] Can filter with conditions
- [ ] Can select and calculate fields (using duration arithmetic, not nanosecond constants)
- [ ] Can aggregate with summarize
- [ ] Can sort and limit results
- [ ] Can specify time ranges
- [ ] Know that `makeTimeseries` exists for event → series conversion

---

## Summary

In this notebook, you learned:

- DQL uses a pipeline model (not SQL)
- `fetch` starts every query
- `filter` narrows results
- `fields` selects and calculates columns
- `summarize` aggregates data
- `sort` and `limit` control output
- Time ranges with `from:` and `to:`
- Duration arithmetic (`duration / 1ms`) is preferred over nanosecond constants

---

## References

- [DQL Reference](https://docs.dynatrace.com/docs/platform/grail/dynatrace-query-language)
- [DQL Functions](https://docs.dynatrace.com/docs/platform/grail/dynatrace-query-language/functions)
- [DQL Commands](https://docs.dynatrace.com/docs/platform/grail/dynatrace-query-language/commands)
- [Dynatrace Query Language (DT docs)](https://docs.dynatrace.com/docs/platform/grail/dynatrace-query-language)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
