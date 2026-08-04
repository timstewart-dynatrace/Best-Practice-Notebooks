# BIZEV-04: Revenue Impact Analysis

> **Series:** BIZEV — Business Events & Funnel Analysis | **Notebook:** 4 of 7 | **Created:** March 2026 | **Last Updated:** 08/04/2026

## Overview

When a Dynatrace detected problem fires, the immediate question from stakeholders is: "What is the business impact?" This notebook demonstrates how to correlate Dynatrace Intelligence problems with business event data to quantify revenue loss during incidents, measure SLA impact, compare incident periods against baselines, and filter analysis to business hours. The goal is to translate technical incidents into business language that executives and product owners understand.

---

## Table of Contents

1. [Connecting Problems to Business Impact](#connecting-problems-to-business-impact)
2. [Revenue During Incidents vs Baseline](#revenue-during-incidents-vs-baseline)
3. [Calculating Revenue Loss](#calculating-revenue-loss)
4. [Business Hours Filtering](#business-hours-filtering)
5. [Day-Over-Day Business Comparison](#day-over-day-business-comparison)
6. [SLA Impact Measurement](#sla-impact-measurement)
7. [Building an Impact Timeline](#building-an-impact-timeline)
8. [Summary and Next Steps](#summary-and-next-steps)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Dynatrace Environment** | SaaS with Grail enabled |
| **Permissions** | `storage:bizevents:read`, `storage:events:read` |
| **Data** | Business events with revenue/amount fields; detected problems data |
| **Knowledge** | BIZEV-01 through BIZEV-03, detected problems concepts |

<a id="connecting-problems-to-business-impact"></a>

## 1. Connecting Problems to Business Impact

Dynatrace Intelligence detects infrastructure and application problems. Business events capture transactions. By correlating the two, you can answer: "During the 2-hour outage on Tuesday, how many transactions were lost?"

### Approach

1. Fetch detected problems with their time windows (`event.start` to `event.end`)
2. Fetch business events during the same time windows
3. Compare business event volume during the problem vs a baseline period
4. Calculate the delta as "lost" transactions or revenue

Let's start by examining recent closed problems.

```dql
// Recent closed detected problems with duration
fetch dt.davis.problems, from:-7d
| filter event.status == "CLOSED"
| fieldsAdd duration_min = resolved_problem_duration / 1m
| fieldsKeep display_id, event.name, event.start, event.end, duration_min, event.category
| sort event.start desc
| limit 10
```

```dql
// Business event volume during problem periods vs overall
// Count concurrent open problems alongside business event volume per hour
fetch bizevents, from:-7d
| makeTimeseries biz_events = count(), interval:1h
```

<a id="revenue-during-incidents-vs-baseline"></a>

## 2. Revenue During Incidents vs Baseline

To measure impact, compare the business event rate during an incident against a "normal" baseline. The baseline can be:

- The same time period on the previous day
- The average for that hour of the week
- The 7-day rolling average

```dql
// Compare today's business event volume vs yesterday (hourly)
fetch bizevents, from:-24h
| summarize today_count = count()
| append [
    fetch bizevents, from:-48h, to:-24h
    | summarize yesterday_count = count()
  ]
```

```dql
// Hourly business event comparison — today vs same day last week
fetch bizevents, from:-24h
| fieldsAdd hour = formatTimestamp(timestamp, format: "HH")
| summarize today_events = count(), by:{hour}
| append [
    fetch bizevents, from:-192h, to:-168h
    | fieldsAdd hour = formatTimestamp(timestamp, format: "HH")
    | summarize last_week_events = count(), by:{hour}
  ]
| sort hour asc
```

<a id="calculating-revenue-loss"></a>

## 3. Calculating Revenue Loss

If your business events include an `amount` or `total` field, you can calculate actual revenue impact.

> **Note:** Replace `amount` with the actual field name in your business events that carries the transaction value.

```dql
// Total revenue from completed orders in the last 24 hours
fetch bizevents, from:-24h
| filter event.type == "com.myapp.order.completed"
| filter isNotNull(amount)
| summarize {total_revenue = sum(toDouble(amount)),
           order_count = count(),
           avg_order_value = avg(toDouble(amount))}
```

```dql
// Revenue per hour — detect dips that correlate with incidents
fetch bizevents, from:-24h
| filter event.type == "com.myapp.order.completed"
| filter isNotNull(amount)
| makeTimeseries hourly_revenue = sum(toDouble(amount)), interval:1h
```

```dql
// Estimate revenue loss: compare a specific incident window to the baseline
// Replace the timeframe values with your actual incident window
fetch bizevents, from:-24h
| filter event.type == "com.myapp.order.completed"
| filter isNotNull(amount)
| fieldsAdd hour = getHour(timestamp)
| summarize {hourly_revenue = sum(toDouble(amount)),
           order_count = count()}, by:{hour}
| sort hour asc
```

<a id="business-hours-filtering"></a>

## 4. Business Hours Filtering

Not all hours are equal. An incident at 3 AM on a Sunday has different business impact than one at 10 AM on a Tuesday. Filter analysis to business hours for more meaningful impact assessment.

```dql
// Business events during business hours only (Mon-Fri, 9 AM - 5 PM)
fetch bizevents, from:-7d
| fieldsAdd dow = getDayOfWeek(timestamp),
           hour = getHour(timestamp)
| filter dow >= 1 and dow <= 5 and hour >= 9 and hour < 17
| summarize business_hours_events = count(),
           by:{day = bin(timestamp, 1d)}
| sort day asc
```

```dql
// Compare business hours revenue vs after-hours revenue
fetch bizevents, from:-7d
| filter event.type == "com.myapp.order.completed"
| filter isNotNull(amount)
| fieldsAdd dow = getDayOfWeek(timestamp),
           hour = getHour(timestamp)
| fieldsAdd period = if(dow >= 1 and dow <= 5 and hour >= 9 and hour < 17,
                        then: "Business Hours",
                        else: "After Hours")
| summarize {total_revenue = sum(toDouble(amount)),
           order_count = count()}, by:{period}
```

<a id="day-over-day-business-comparison"></a>

## 5. Day-Over-Day Business Comparison

Compare today's business metrics against yesterday to quickly spot anomalies.

```dql
// Day-over-day comparison: today vs yesterday order counts
fetch bizevents, from:bin(now(), 24h)
| filter event.type == "com.myapp.order.completed"
| summarize today_orders = count()
| append [
    fetch bizevents, from:bin(now(), 24h) - 24h, to:bin(now(), 24h)
    | filter event.type == "com.myapp.order.completed"
    | summarize yesterday_orders = count()
  ]
```

```dql
// Week-over-week trend — daily order counts for the past 2 weeks
fetch bizevents, from:-14d
| filter event.type == "com.myapp.order.completed"
| fieldsAdd day = bin(timestamp, 1d)
| summarize daily_orders = count(), by:{day}
| sort day asc
```

<a id="sla-impact-measurement"></a>

## 6. SLA Impact Measurement

Service Level Agreements (SLAs) often include business availability targets. By correlating problem duration with business event volume, you can measure whether SLAs were met.

```dql
// Total problem duration this week (hours of impact)
fetch dt.davis.problems, from:-7d
| filter event.status == "CLOSED"
| filter dt.davis.is_duplicate == false
| fieldsAdd duration_hours = resolved_problem_duration / 1h
| summarize {total_problem_hours = sum(duration_hours),
           problem_count = count(),
           avg_mttr_hours = avg(duration_hours)}
```

```dql
// SLA calculation: percentage of time without problems in business hours
// Assumes 8 business hours * 5 days = 40 hours per week
fetch dt.davis.problems, from:-7d
| filter event.status == "CLOSED"
| filter dt.davis.is_duplicate == false
| fieldsAdd duration_hours = resolved_problem_duration / 1h
| summarize total_downtime_hours = sum(duration_hours)
| fieldsAdd total_business_hours = 40.0
| fieldsAdd availability_pct = round((total_business_hours - total_downtime_hours) / total_business_hours * 100, decimals: 2)
```

<a id="building-an-impact-timeline"></a>

## 7. Building an Impact Timeline

An impact timeline overlays business event volume with detected problems to visualize how incidents affect business operations.

```dql
// Business event volume over the past 7 days
// Overlay with problem occurrences to see impact
fetch bizevents, from:-7d
| filter event.type == "com.myapp.order.completed"
| makeTimeseries orders = count(), interval:1h
```

```dql
// Concurrent open problems over time — overlay with order volume chart
fetch dt.davis.problems, from:-7d
| makeTimeseries open_problems = count(),
    spread: timeframe(from: event.start, to: coalesce(event.end, now()))
```

<a id="summary-and-next-steps"></a>

## 8. Summary and Next Steps

In this notebook, you learned:

- **Problem-to-business correlation** — Connecting detected problems to business event data
- **Revenue during incidents** — Comparing incident periods against baselines
- **Revenue loss estimation** — Using transaction amounts to quantify financial impact
- **Business hours filtering** — Scoping analysis to when it matters most
- **Day-over-day comparison** — Detecting business anomalies via historical comparison
- **SLA measurement** — Calculating availability from problem duration data
- **Impact timelines** — Overlaying problems with business metrics

### Next Steps

- **BIZEV-05: KPIs and Metrics** — Define ongoing business health indicators
- **BIZEV-06: Executive Reporting** — Build executive-ready reports combining these techniques

### References

- [Davis Problems app (DT docs)](https://docs.dynatrace.com/docs/dynatrace-intelligence/problems-app)
- [Dynatrace Business Analytics](https://docs.dynatrace.com/docs/observe/business-observability)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
