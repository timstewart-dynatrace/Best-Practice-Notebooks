# DASH-04: Operations Dashboards

> **Series:** DASH — Dashboard Design & Building | **Notebook:** 4 of 7 | **Created:** March 2026 | **Last Updated:** 08/12/2026

## Overview

Operations dashboards serve as the real-time control panel for SREs, platform engineers, and NOC teams. Unlike executive dashboards that summarize weekly trends, operations dashboards focus on what is happening right now — service response times, error rates, log volume anomalies, active problems, and infrastructure health. This notebook covers tile selection, query patterns for real-time monitoring, and layout strategies for operations-tier dashboards.

---

## Table of Contents

1. [Service Response Time Monitoring](#service-response-time)
2. [Error Rate by Service](#error-rate-by-service)
3. [Log Volume and Anomaly Tracking](#log-volume-tracking)
4. [Active Problems and Alerts](#active-problems)
5. [Infrastructure Health](#infrastructure-health)
6. [Deployment Correlation](#deployment-correlation)
7. [Operations Dashboard Layout](#ops-layout)
8. [Summary and Next Steps](#summary-and-next-steps)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Dynatrace Environment** | SaaS or Managed with Grail enabled |
| **Permissions** | `storage:logs:read`, `storage:metrics:read`, `storage:events:read`, `storage:spans:read` |
| **Data** | Active services with span data, log ingestion, host metrics |
| **Prior Reading** | DASH-01, DASH-02 |

<a id="service-response-time"></a>

## 1. Service Response Time Monitoring

Response time is the most watched metric on an operations dashboard. Show it as a timeseries line chart with p50, p90, and p95 lines for context.

### Response Time Trend (All Services)

```dql
// Service response time trend — p50, p90, p95 over last 2 hours
//
// Corrected 08/12/2026: `makeTimeseries` takes a BARE aggregation — dividing inside it
// (`percentile(duration, 50) / 1ms`) fails with "the parameter has to be an expression-based
// timeseries aggregation". Aggregate first, convert afterwards in `fieldsAdd`. Note the unit
// conversion also changes: makeTimeseries yields a numeric array in NANOSECONDS, not a duration,
// so `/ 1ms` silently yields null on the array — divide by 1000000.0 instead.
fetch spans, from:-2h
| filter span.kind == "server"
| makeTimeseries p50 = percentile(duration, 50), p90 = percentile(duration, 90), p95 = percentile(duration, 95), interval:5m
| fieldsAdd p50_ms = p50[] / 1000000.0, p90_ms = p90[] / 1000000.0, p95_ms = p95[] / 1000000.0
| fieldsRemove p50, p90, p95
```

### Response Time by Service (Top 10 Slowest)

```dql
// Top 10 slowest services by average response time
fetch spans, from:-1h
| filter span.kind == "server"
| summarize avg_ms = avg(duration) / 1ms, p95_ms = percentile(duration, 95) / 1ms, request_count = count(), by:{dt.entity.service}
| sort p95_ms desc
| limit 10
```

<a id="error-rate-by-service"></a>

## 2. Error Rate by Service

Error rate as a percentage shows which services are degraded. Display as a bar chart sorted by error rate descending.

### Current Error Rate Table

```dql
// Error rate by service — operations bar chart
fetch spans, from:-1h
| filter span.kind == "server"
| summarize total = count(), errors = countIf(span.status_code == "error"), by:{dt.entity.service}
| fieldsAdd error_rate_pct = round(100.0 * errors / total, decimals: 2)
| sort error_rate_pct desc
| limit 15
```

### Error Rate Trend Over Time

```dql
// Overall error rate trend — operations line chart
fetch spans, from:-2h
| filter span.kind == "server"
| makeTimeseries total = count(), errors = countIf(span.status_code == "error"), interval:5m
| fieldsAdd error_rate = arrayAvg(errors) / arrayAvg(total) * 100
```

<a id="log-volume-tracking"></a>

## 3. Log Volume and Anomaly Tracking

Sudden log volume changes often indicate problems before Dynatrace Intelligence detects them. Track log volume by level and source.

### Log Volume by Severity Level

```dql
// Log volume by severity — stacked area chart
fetch logs, from:-2h
| filterOut loglevel == "NONE"
| makeTimeseries log_count = count(), interval:5m, by:{loglevel}
```

### Top Log Sources by Error Volume

```dql
// Top 10 log sources generating errors — operations table
fetch logs, from:-1h
| filter loglevel == "ERROR"
| summarize error_count = count(), by:{log.source}
| sort error_count desc
| limit 10
```

### Log Volume Spike Detection

Compare current hour to the same hour yesterday to detect anomalous volume increases.

```dql
// Log volume comparison: current hour vs same hour yesterday
fetch logs, from:-1h
| summarize current_count = count()
| append [
    fetch logs, from:now() - 25h, to:now() - 24h
    | summarize yesterday_count = count()
  ]
```

<a id="active-problems"></a>

## 4. Active Problems and Alerts

Active problems should be front and center on every operations dashboard. Show both the count and the detail.

### Active Problems List

```dql
// Active problems detail table — operations dashboard
fetch dt.davis.problems, from:-24h
| filter event.status == "ACTIVE"
| filter dt.davis.is_duplicate == false
| fieldsAdd duration_min = (now() - event.start) / 1m
| fieldsKeep display_id, event.name, event.category, duration_min
| sort duration_min desc
```

### Problem Status Over Time

```dql
// Concurrently open problems over time — area chart
fetch dt.davis.problems, from:-24h
| makeTimeseries concurrent = count(), spread:timeframe(from:event.start, to:coalesce(event.end, now()))
```

<a id="infrastructure-health"></a>

## 5. Infrastructure Health

Infrastructure tiles provide context for service-level issues. CPU, memory, and disk metrics help operations teams determine if a problem is service-specific or infrastructure-wide.

### Host CPU Usage

```dql
// Host CPU usage — operations line chart
timeseries avg_cpu = avg(dt.host.cpu.usage), from:-2h, by:{dt.entity.host}
```

### Top Hosts by CPU — Identifying Hot Spots

```dql
// Top 5 hosts by average CPU usage
timeseries avg_cpu = avg(dt.host.cpu.usage), from:-1h, by:{dt.entity.host}
| fieldsAdd avg_cpu_val = arrayAvg(avg_cpu)
| sort avg_cpu_val desc
| limit 5
```

### Host Memory Usage

```dql
// Host memory usage — operations line chart
timeseries avg_mem = avg(dt.host.memory.usage), from:-2h, by:{dt.entity.host}
```

<a id="deployment-correlation"></a>

## 6. Deployment Correlation

Deployments are the most common cause of production issues. Overlaying deployment events on service metrics helps operations teams immediately correlate "what changed."

### Recent Deployment Events

```dql
// Recent deployment events — operations context table
fetch events, from:-6h
| filter event.kind == "DEPLOYMENT_EVENT" or event.type == "CUSTOM_DEPLOYMENT"
| fieldsKeep timestamp, event.type, dt.entity.service, event.name
| sort timestamp desc
| limit 20
```

<a id="ops-layout"></a>

## 7. Operations Dashboard Layout

![Operations Dashboard Layout](images/04-operations-dashboard-layout.png)
<!-- MARKDOWN_TABLE_ALTERNATIVE
| Section | Tiles | Reading Order |
|---------|-------|---------------|
| 1. Status Bar (top) | Active problems, error rate, log errors | First — always visible |
| 2. Service Health | Response time trend, error rate by service | Top-left anchor (most actionable) |
| 3. Log Intelligence | Log volume by level, top error sources | Z-pattern reading |
| 4. Infrastructure | CPU by host, memory by host | Continue Z-pattern |
| 5. Change Context | Recent deployments | Bottom-right (correlates with above) |
Anti-patterns to avoid: spaghetti charts, all-24h tiles, no problem context, missing deploys.
For environments where SVG doesn't render
-->

### Recommended Tile Arrangement

| Section | Tiles | Chart Type |
|---------|-------|------------|
| **Status Bar** (top) | Active problems count, overall error rate, log error count | Single values |
| **Service Health** | Response time trend, error rate by service | Line chart, bar chart |
| **Log Intelligence** | Log volume by level, top error sources | Stacked area, table |
| **Infrastructure** | CPU by host, memory by host | Line charts |
| **Change Context** | Recent deployments | Table |

### Operations Dashboard Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| Too many services on one chart | Unreadable spaghetti lines | Use variables or top-N filtering |
| All tiles show 24h data | Hides real-time spikes | Use 1-2h for real-time tiles |
| No problem context | Team sees metrics but not root cause | Add active problems table |
| Missing deployment markers | Cannot correlate changes with incidents | Add deployment events section |

<a id="summary-and-next-steps"></a>

## 8. Summary and Next Steps

In this notebook you learned:

- How to build service response time and error rate tiles for real-time monitoring
- Log volume tracking and spike detection queries
- Active problem tables with duration tracking
- Infrastructure health tiles for CPU and memory
- Deployment event tracking for change correlation
- Recommended layout and common anti-patterns for operations dashboards

**Next:** In **DASH-05: Engineering Dashboards**, we move to the deepest tier — trace-level analysis, database performance, endpoint metrics, and before/after deployment comparisons.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
