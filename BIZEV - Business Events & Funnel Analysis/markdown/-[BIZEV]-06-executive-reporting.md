# BIZEV-06: Executive Reporting

> **Series:** BIZEV — Business Events & Funnel Analysis | **Notebook:** 6 of 7 | **Created:** March 2026 | **Last Updated:** 08/04/2026

## Overview

Executive stakeholders need a clear, business-focused view of platform health — not infrastructure metrics and log counts. This notebook demonstrates how to build executive-ready queries that combine technical and business signals, create SLA/SLO tracking from a business perspective, design queries suitable for scheduled report workflows, and present data in formats that non-technical audiences can immediately understand. The queries here are designed to populate dashboards and automated reports.

---

## Table of Contents

1. [Executive Dashboard Queries](#executive-dashboard-queries)
2. [Combined Technical and Business View](#combined-technical-and-business-view)
3. [SLA and SLO Tracking](#sla-and-slo-tracking)
4. [Scheduled Reporting Patterns](#scheduled-reporting-patterns)
5. [Business Value Dashboards](#business-value-dashboards)
6. [Weekly Business Review Queries](#weekly-business-review-queries)
7. [Summary](#summary)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Dynatrace Environment** | SaaS with Grail enabled |
| **Permissions** | `storage:bizevents:read`, `storage:events:read`, `storage:metrics:read` |
| **Data** | Business events with transaction and revenue data; detected problems |
| **Knowledge** | BIZEV-01 through BIZEV-05 |

<a id="executive-dashboard-queries"></a>

## 1. Executive Dashboard Queries

An executive dashboard answers three questions at a glance:

1. **Is the business running?** — Transaction volume is normal
2. **Are there problems?** — Error rates are within tolerance
3. **What is the trend?** — KPIs are stable or improving

### The Executive Summary Query

This single query provides a high-level business snapshot.

```dql
// Executive snapshot: business health in the last 24 hours
fetch bizevents, from:-24h
| summarize {total_transactions = count(),
           distinct_event_types = countDistinct(event.type),
           distinct_providers = countDistinct(event.provider),
           earliest_event = min(timestamp),
           latest_event = max(timestamp)}
| fieldsAdd reporting_period = "Last 24 Hours",
           data_freshness = if(latest_event > now() - 5m, then: "CURRENT", else: "STALE")
```

```dql
// Top business event types by volume — the key business processes
fetch bizevents, from:-24h
| summarize volume = count(), by:{event.type}
| sort volume desc
| limit 10
```

<a id="combined-technical-and-business-view"></a>

## 2. Combined Technical and Business View

Executives want to understand the business impact of technical issues. These queries combine detected problem data with business event metrics.

```dql
// Active detected problems — what technical issues are happening right now?
fetch dt.davis.problems, from:-24h
| filter event.status == "ACTIVE"
| fieldsKeep display_id, event.name, event.category, event.start
| sort event.start desc
```

```dql
// Problem summary with MTTR — executive-friendly problem overview
fetch dt.davis.problems, from:-7d
| filter event.status == "CLOSED"
| filter dt.davis.is_duplicate == false
| fieldsAdd duration_hours = resolved_problem_duration / 1h
| summarize {problem_count = count(),
           avg_resolution_hours = avg(duration_hours),
           max_resolution_hours = max(duration_hours),
           total_impact_hours = sum(duration_hours)}, by:{event.category}
| sort problem_count desc
```

```dql
// Side-by-side: business events and problems over the past week
// Business event volume
fetch bizevents, from:-7d
| fieldsAdd day = bin(timestamp, 1d)
| summarize daily_transactions = count(), by:{day}
| sort day asc
```

```dql
// Problem count per day — overlay with business event volume
fetch dt.davis.problems, from:-7d
| filter dt.davis.is_duplicate == false
| fieldsAdd day = bin(timestamp, 1d)
| summarize daily_problems = count(), by:{day}
| sort day asc
```

<a id="sla-and-slo-tracking"></a>

## 3. SLA and SLO Tracking

SLA (Service Level Agreement) and SLO (Service Level Objective) tracking from a business perspective focuses on business availability and transaction success rates rather than infrastructure uptime.

### Business SLO Definitions

| SLO | Target | Measurement |
|-----|--------|-------------|
| Transaction Success Rate | > 99.5% | Successful / Total transactions |
| Business Availability | > 99.9% | Hours with transactions / Total hours |
| Mean Time to Resolve | < 2 hours | Avg problem resolution time |
| Revenue Impact per Incident | < $5,000 | Revenue delta during incidents |

```dql
// Transaction success rate SLO — last 7 days
fetch bizevents, from:-7d
| filter in(event.type, {"com.myapp.payment.processed", "com.myapp.payment.failed"})
| summarize {total = count(),
           successful = countIf(event.type == "com.myapp.payment.processed"),
           failed = countIf(event.type == "com.myapp.payment.failed")}
| fieldsAdd success_rate = round(toDouble(successful) / toDouble(total) * 100, decimals: 3),
           slo_target = 99.5,
           slo_met = if(toDouble(successful) / toDouble(total) * 100 >= 99.5, then: "YES", else: "NO")
```

```dql
// Business availability — count hours with at least 1 business event
fetch bizevents, from:-7d
| fieldsAdd hour_bucket = bin(timestamp, 1h)
| summarize events_in_hour = count(), by:{hour_bucket}
| summarize hours_with_data = count()
| fieldsAdd total_hours = 168,
           availability_pct = round(toDouble(hours_with_data) / 168.0 * 100, decimals: 2)
```

```dql
// MTTR SLO — mean time to resolve over the last 30 days
fetch dt.davis.problems, from:-30d
| filter event.status == "CLOSED"
| filter dt.davis.is_duplicate == false and dt.davis.is_frequent_event == false
| fieldsAdd duration_hours = resolved_problem_duration / 1h
| summarize {avg_mttr = avg(duration_hours),
           median_mttr = median(duration_hours),
           p95_mttr = percentile(duration_hours, 95),
           problem_count = count()}
| fieldsAdd mttr_target_hours = 2.0,
           slo_met = if(avg_mttr <= 2.0, then: "YES", else: "NO")
```

<a id="scheduled-reporting-patterns"></a>

## 4. Scheduled Reporting Patterns

These queries are designed for use in Dynatrace Workflows to generate scheduled reports. They produce clean, self-contained result sets suitable for email or Slack notifications.

### Daily Business Report Query

Send this as a daily morning report to stakeholders.

```dql
// Daily report: yesterday's business summary
fetch bizevents, from:bin(now(), 24h) - 24h, to:bin(now(), 24h)
| summarize {total_events = count(),
           distinct_types = countDistinct(event.type),
           distinct_providers = countDistinct(event.provider)}
| fieldsAdd report_date = bin(now(), 24h) - 24h,
           report_type = "Daily Business Summary"
```

```dql
// Daily report: top event types by volume (yesterday)
fetch bizevents, from:bin(now(), 24h) - 24h, to:bin(now(), 24h)
| summarize volume = count(), by:{event.type}
| sort volume desc
| limit 10
```

### Workflow Integration

To schedule these reports in Dynatrace:

1. Navigate to **Workflows** in the Dynatrace UI
2. Create a new workflow with a **Time trigger** (e.g., daily at 8:00 AM)
3. Add a **DQL query** action with the report query
4. Add a **Send notification** action (email, Slack, Teams) with the query results

```yaml
# Example workflow definition
trigger:
  type: time
  schedule: "0 8 * * 1-5"  # Weekdays at 8 AM

actions:
  - name: "Run daily report"
    type: dql_query
    query: |
      fetch bizevents, from:bin(now(), 24h) - 24h, to:bin(now(), 24h)
      | summarize total_events = count(), by:{event.type}
      | sort total_events desc
      | limit 10

  - name: "Send to Slack"
    type: notification
    channel: "#business-reports"
    template: "daily_business_summary"
```

<a id="business-value-dashboards"></a>

## 5. Business Value Dashboards

Dashboard tiles should each answer one specific question. Here are the essential tiles for a business value dashboard.

### Tile 1: Current Transaction Rate

```dql
// Dashboard tile: current transactions per minute (last 15 min)
fetch bizevents, from:-15m
| summarize tx_count = count()
| fieldsAdd tx_per_minute = round(toDouble(tx_count) / 15.0, decimals: 1)
```

```dql
// Dashboard tile: transaction trend (last 6 hours, 15-min intervals)
fetch bizevents, from:-6h
| makeTimeseries tx_count = count(), interval:15m
```

```dql
// Dashboard tile: event distribution by provider (pie chart)
fetch bizevents, from:-24h
| summarize volume = count(), by:{event.provider}
| sort volume desc
| limit 8
```

### Tile 4: Problem Impact Summary

```dql
// Dashboard tile: problem impact summary (last 7 days)
fetch dt.davis.problems, from:-7d
| filter dt.davis.is_duplicate == false
| summarize {active = countIf(event.status == "ACTIVE"),
           closed_today = countIf(event.status == "CLOSED" and event.end > bin(now(), 24h)),
           total_week = count()}
```

<a id="weekly-business-review-queries"></a>

## 6. Weekly Business Review Queries

These queries produce the data needed for a weekly business review meeting.

```dql
// Weekly review: daily business event summary for the past week
fetch bizevents, from:-7d
| fieldsAdd day = bin(timestamp, 1d)
| summarize {daily_volume = count(),
           event_types = countDistinct(event.type)}, by:{day}
| sort day asc
```

```dql
// Weekly review: problems resolved this week with resolution time
fetch dt.davis.problems, from:-7d
| filter event.status == "CLOSED"
| filter dt.davis.is_duplicate == false
| fieldsAdd resolution_hours = round(resolved_problem_duration / 1h, decimals: 1)
| fieldsKeep display_id, event.name, event.category, event.start, event.end, resolution_hours
| sort event.start desc
```

```dql
// Weekly review: business event volume comparison — this week vs last week
fetch bizevents, from:-7d
| summarize this_week = count()
| append [
    fetch bizevents, from:-14d, to:-7d
    | summarize last_week = count()
  ]
```

```dql
// Weekly review: busiest hours of the week (for capacity planning)
fetch bizevents, from:-7d
| fieldsAdd dow = getDayOfWeek(timestamp),
           hour = getHour(timestamp)
| summarize volume = count(), by:{dow, hour}
| sort volume desc
| limit 20
```

<a id="summary"></a>

## 7. Summary

This is the last of the core analysis notebooks in the BIZEV series. Across the series, you have learned:

| Notebook | Key Capability |
|----------|----------------|
| **BIZEV-01** | Business events data model and exploration |
| **BIZEV-02** | Instrumentation methods (OneAgent, API, SDK, OpenPipeline) |
| **BIZEV-03** | Conversion funnel analysis and drop-off detection |
| **BIZEV-04** | Revenue impact analysis and incident correlation |
| **BIZEV-05** | Business KPI definition, metric extraction, health scoring |
| **BIZEV-06** | Executive reporting, SLA tracking, scheduled reports |
| **BIZEV-07** | Gen2 vs Gen3 adoption paths — business events without (or before) the full move |

### Key Takeaways for Executive Reporting

1. **Lead with business outcomes** — Transactions, revenue, conversion rates — not CPU or memory
2. **Provide context** — Always compare against baselines (yesterday, last week, last month)
3. **Quantify impact** — Translate technical incidents into business metrics (lost revenue, failed transactions)
4. **Automate delivery** — Use Workflows to schedule reports so stakeholders receive them proactively
5. **Keep it simple** — One KPI per dashboard tile, clear labels, consistent time periods

### References

- [Dynatrace Business Analytics](https://docs.dynatrace.com/docs/observe/business-observability)
- [Dynatrace Dashboards](https://docs.dynatrace.com/docs/analyze-explore-automate/dashboards-classic)
- [Workflows (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/workflows)
- [Service-level objectives (DT docs)](https://docs.dynatrace.com/docs/deliver/service-level-objectives)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
