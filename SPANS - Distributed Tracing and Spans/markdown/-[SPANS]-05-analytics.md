# SPANS-05: Advanced Span Analytics

> **Series:** SPANS — Distributed Tracing and Spans | **Notebook:** 5 of 8 | **Created:** December 2025 | **Last Updated:** 09/18/2026

## Time-Series Analysis and Complex Aggregations
This notebook covers advanced analytical techniques for span data, including time-series analysis, trend detection, and complex aggregations for building dashboards and reports.

---

## Table of Contents

1. [Time-Series with makeTimeseries](#time-series-with-maketimeseries)
2. [Trend Analysis](#trend-analysis)
3. [Complex Aggregations](#complex-aggregations)
4. [Comparison Queries](#comparison-queries)
5. [Dashboard-Ready Queries](#dashboard-ready-queries)

---


## Prerequisites

Before starting this notebook, ensure you have:

- ✅ Completed previous SPANS notebooks (01-04)
- ✅ Understanding of summarize and aggregation functions
- ✅ Familiarity with time-based filtering

<a id="time-series-with-maketimeseries"></a>
## 1. Time-Series with makeTimeseries
Use `makeTimeseries` to create time-series data for visualization and trend analysis.

![Timeseries Analytics](images/05-timeseries-analytics.png)

<!--MARKDOWN_TABLE_ALTERNATIVE
| Function | Purpose | makeTimeseries Support |
|----------|---------|------------------------|
| count() | Count records | ✅ Yes |
| countIf() | Conditional count | ✅ Yes |
| avg() | Average value | ✅ Yes |
| sum() | Total sum | ✅ Yes |
| min()/max() | Extremes | ✅ Yes |
| percentile() | Distribution | ❌ No - use bin() |
-->

> ⚠️ **Important:** `makeTimeseries` does NOT support `percentile()` or arithmetic expressions in aggregations. Use time-bucketed `summarize` for those.

```dql
// Request volume over time by service
fetch spans, from:-24h
| filter span.kind == "server"
| makeTimeseries {
    request_count = count(),
    error_count = countIf(span.status_code == "error")
  }, by:{service.name}, interval: 5m
```

```dql
// Average duration over time (without grouping)
fetch spans, from:-24h
| filter span.kind == "server"
| makeTimeseries {
    request_count = count(),
    avg_duration_ns = avg(duration)
  }, interval: 5m
```

```dql
// For percentile trends, use time-bucketed summarize instead
fetch spans, from:-1h
| filter span.kind == "server"
| fieldsAdd time_bucket = bin(start_time, 10m)
| summarize {
    request_count = count(),
    p95_duration_ms = percentile(duration, 95) / 1ms,
    error_count = countIf(span.status_code == "error")
  }, by:{time_bucket, service.name}
| sort time_bucket asc
| limit 200
```

---

<a id="trend-analysis"></a>
## 2. Trend Analysis
Identify trends and patterns in your span data over time.

```dql
// Hourly error rate trend
fetch spans, from:-1h
| filter span.kind == "server"
| fieldsAdd hour_bucket = bin(start_time, 1h)
| summarize {
    total_requests = count(),
    errors = countIf(span.status_code == "error")
  }, by:{hour_bucket}
| fieldsAdd error_rate_pct = (errors * 100.0) / total_requests
| sort hour_bucket asc
| limit 48
```

```dql
// Latency trend by service (10-minute buckets)
fetch spans, from:-1h
| filter span.kind == "server"
| fieldsAdd time_bucket = bin(start_time, 10m)
| summarize {
    request_count = count(),
    avg_duration_ms = avg(duration) / 1ms,
    p90_duration_ms = percentile(duration, 90) / 1ms
  }, by:{time_bucket, service.name}
| sort time_bucket asc, service.name
| limit 300
```

```dql
// Identify services with degrading performance
fetch spans, from:-1h
| filter span.kind == "server"
| fieldsAdd time_bucket = bin(start_time, 30m)
| summarize {
    request_count = count(),
    p95_ms = percentile(duration, 95) / 1ms
  }, by:{time_bucket, service.name}
| filter p95_ms > 500
| sort time_bucket desc, p95_ms desc
| limit 50
```

---

<a id="complex-aggregations"></a>
## 3. Complex Aggregations
Perform multi-dimensional analysis with complex aggregation patterns.

```dql
// Service health scorecard
fetch spans, from:-1h
| filter span.kind == "server"
| summarize {
    total_requests = count(),
    error_count = countIf(span.status_code == "error"),
    avg_duration_ms = avg(duration) / 1ms,
    p50_duration_ms = percentile(duration, 50) / 1ms,
    p95_duration_ms = percentile(duration, 95) / 1ms,
    p99_duration_ms = percentile(duration, 99) / 1ms,
    max_duration_ms = max(duration) / 1ms
  }, by:{service.name}
| fieldsAdd error_rate_pct = (error_count * 100.0) / total_requests
| fieldsAdd health_score = if(error_rate_pct > 5, "Critical", 
                            else: if(error_rate_pct > 1, "Warning", 
                            else: "Healthy"))
| sort error_rate_pct desc
| limit 30
```

```dql
// Endpoint-level analysis with multiple metrics
fetch spans, from:-1h
| filter span.kind == "server"
| summarize {
    request_count = count(),
    error_count = countIf(span.status_code == "error"),
    slow_count = countIf(duration > 1s),  // > 1 second
    avg_duration_ms = avg(duration) / 1ms,
    p95_duration_ms = percentile(duration, 95) / 1ms
  }, by:{service.name, span.name}
| fieldsAdd error_rate_pct = (error_count * 100.0) / request_count
| fieldsAdd slow_rate_pct = (slow_count * 100.0) / request_count
| filter request_count > 10
| sort error_rate_pct desc
| limit 50
```

```dql
// HTTP method distribution with performance metrics
fetch spans, from:-1h
| filter isNotNull(http.request.method)
| summarize {
    request_count = count(),
    error_count = countIf(span.status_code == "error"),
    avg_duration_ms = avg(duration) / 1ms
  }, by:{http.request.method, service.name}
| sort request_count desc
| limit 30
```

---

<a id="comparison-queries"></a>
## 4. Comparison Queries
Compare metrics across different dimensions to identify outliers and patterns.

```dql
// Compare span kinds: server vs client performance
fetch spans, from:-1h
| filter in(span.kind, {"server", "client"})
| summarize {
    span_count = count(),
    error_count = countIf(span.status_code == "error"),
    avg_duration_ms = avg(duration) / 1ms,
    p95_duration_ms = percentile(duration, 95) / 1ms
  }, by:{span.kind}
| fieldsAdd error_rate_pct = (error_count * 100.0) / span_count
```

```dql
// Compare success vs error span characteristics
fetch spans, from:-1h
| filter span.kind == "server"
| summarize {
    span_count = count(),
    avg_duration_ms = avg(duration) / 1ms,
    p95_duration_ms = percentile(duration, 95) / 1ms
  }, by:{span.status_code, service.name}
| sort service.name, span.status_code
| limit 50
```

```dql
// HTTP status code distribution by service
fetch spans, from:-1h
| filter isNotNull(http.response.status_code)
| fieldsAdd status_class = if(http.response.status_code >= 500, "5xx",
                            else: if(http.response.status_code >= 400, "4xx",
                            else: if(http.response.status_code >= 300, "3xx",
                            else: "2xx")))
| summarize {count = count()}, by:{status_class, service.name}
| sort service.name, status_class
```

---

<a id="dashboard-ready-queries"></a>
## 5. Dashboard-Ready Queries
Queries optimized for use in Dynatrace dashboards and reports.

> 💡 **Tip:** These queries are designed to produce clean output suitable for dashboard tiles.

```dql
// Dashboard: Request rate and error rate over time
fetch spans, from:-24h
| filter span.kind == "server"
| makeTimeseries {
    requests = count(),
    errors = countIf(span.status_code == "error")
  }, interval: 5m
```

```dql
// Dashboard: Service health summary (single value tiles)
fetch spans, from:-1h
| filter span.kind == "server"
| summarize {
    total_requests = count(),
    total_errors = countIf(span.status_code == "error"),
    unique_services = countDistinct(service.name),
    avg_latency_ms = avg(duration) / 1ms
  }
| fieldsAdd overall_error_rate_pct = (total_errors * 100.0) / total_requests
```

```dql
// Dashboard: Top 10 services by request volume
fetch spans, from:-1h
| filter span.kind == "server"
| summarize {
    requests = count(),
    errors = countIf(span.status_code == "error"),
    p95_ms = percentile(duration, 95) / 1ms
  }, by:{service.name}
| fieldsAdd error_rate = (errors * 100.0) / requests
| sort requests desc
| limit 10
```

```dql
// Dashboard: Recent errors list
fetch spans, from:-1h
| filter span.status_code == "error"
| fields start_time, service.name, span.name, span.status_message
| sort start_time desc
| limit 20
```

---

## Summary

In this notebook, you learned:

✅ **Time-series analysis** with makeTimeseries and its limitations  
✅ **Time-bucketed summarize** for percentile trends  
✅ **Trend analysis** to identify patterns over time  
✅ **Complex aggregations** for multi-dimensional analysis  
✅ **Comparison queries** to identify outliers  
✅ **Dashboard-ready queries** for visualization  

---

## Next Steps

Continue to **SPANS-06: Security Analysis with Spans** to learn:
- Detecting security-relevant patterns in traces
- Analyzing authentication and authorization flows
- Finding anomalous behavior
- Security audit queries

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
