# BIZEV-03: Funnel Analysis

> **Series:** BIZEV — Business Events & Funnel Analysis | **Notebook:** 3 of 7 | **Created:** March 2026 | **Last Updated:** 08/04/2026

## Overview

Conversion funnel analysis is one of the most powerful applications of business events. By tracking users through a sequence of steps — browse, add to cart, checkout, purchase — you can identify where customers drop off, measure conversion rates, and quantify the business impact of friction points. This notebook demonstrates how to build multi-step funnels from business events using DQL, calculate step-by-step conversion rates, and analyze the time users spend between funnel stages.

---

## Table of Contents

1. [Funnel Analysis Concepts](#funnel-analysis-concepts)
2. [Building a Basic Funnel](#building-a-basic-funnel)
3. [Step-by-Step Conversion Rates](#step-by-step-conversion-rates)
4. [Identifying Drop-Off Points](#identifying-drop-off-points)
5. [Time Between Steps](#time-between-steps)
6. [Funnel Segmentation](#funnel-segmentation)
7. [Funnel Trends Over Time](#funnel-trends-over-time)
8. [Summary and Next Steps](#summary-and-next-steps)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Dynatrace Environment** | SaaS with Grail enabled |
| **Permissions** | `storage:bizevents:read` |
| **Data** | Business events with sequential event types (e.g., browse, cart, checkout, purchase) |
| **Knowledge** | BIZEV-01 fundamentals, DQL `summarize` and `countIf` |

<a id="funnel-analysis-concepts"></a>

## 1. Funnel Analysis Concepts

A **conversion funnel** models a sequence of steps that users take to complete a desired outcome. Each step represents a business event type.

### Example: E-Commerce Funnel

| Step | Event Type | Description |
|------|-----------|-------------|
| 1 | `com.myapp.product.viewed` | User views a product page |
| 2 | `com.myapp.cart.updated` | User adds item to cart |
| 3 | `com.myapp.checkout.started` | User begins checkout |
| 4 | `com.myapp.payment.processed` | Payment completed |
| 5 | `com.myapp.order.completed` | Order confirmed |

### Key Metrics

| Metric | Formula | Meaning |
|--------|---------|----------|
| **Step conversion rate** | Step N count / Step N-1 count | % proceeding to next step |
| **Overall conversion rate** | Last step / First step | End-to-end conversion |
| **Drop-off rate** | 1 - step conversion rate | % leaving at each step |
| **Time to convert** | Avg time from first to last step | Speed of conversion |

<a id="building-a-basic-funnel"></a>

## 2. Building a Basic Funnel

The simplest funnel counts distinct users (or sessions) at each step. We use `countIf` to count events matching each step condition in a single query.

> **Note:** Adapt the `event.type` values below to match your actual business event names. The examples use a generic e-commerce pattern.

```dql
// Basic funnel — count events at each step over the last 24 hours
fetch bizevents, from:-24h
| summarize {step1_view = countIf(event.type == "com.myapp.product.viewed"),
           step2_cart = countIf(event.type == "com.myapp.cart.updated"),
           step3_checkout = countIf(event.type == "com.myapp.checkout.started"),
           step4_payment = countIf(event.type == "com.myapp.payment.processed"),
           step5_complete = countIf(event.type == "com.myapp.order.completed")}
```

```dql
// Funnel with distinct user counts (more accurate for conversion)
// Uses countDistinctApprox for performance on large datasets
fetch bizevents, from:-24h
| filter isNotNull(user_id)
| summarize {step1_users = countDistinctApprox(if(event.type == "com.myapp.product.viewed", then: user_id)),
           step2_users = countDistinctApprox(if(event.type == "com.myapp.cart.updated", then: user_id)),
           step3_users = countDistinctApprox(if(event.type == "com.myapp.checkout.started", then: user_id)),
           step4_users = countDistinctApprox(if(event.type == "com.myapp.payment.processed", then: user_id)),
           step5_users = countDistinctApprox(if(event.type == "com.myapp.order.completed", then: user_id))}
```

<a id="step-by-step-conversion-rates"></a>

## 3. Step-by-Step Conversion Rates

Conversion rates reveal the percentage of users who proceed from one step to the next. This is the core metric for funnel optimization.

```dql
// Calculate conversion rates between each funnel step
fetch bizevents, from:-24h
| summarize {step1 = countIf(event.type == "com.myapp.product.viewed"),
           step2 = countIf(event.type == "com.myapp.cart.updated"),
           step3 = countIf(event.type == "com.myapp.checkout.started"),
           step4 = countIf(event.type == "com.myapp.payment.processed"),
           step5 = countIf(event.type == "com.myapp.order.completed")}
| fieldsAdd view_to_cart = round(toDouble(step2) / toDouble(step1) * 100, decimals: 1),
           cart_to_checkout = round(toDouble(step3) / toDouble(step2) * 100, decimals: 1),
           checkout_to_payment = round(toDouble(step4) / toDouble(step3) * 100, decimals: 1),
           payment_to_complete = round(toDouble(step5) / toDouble(step4) * 100, decimals: 1),
           overall_conversion = round(toDouble(step5) / toDouble(step1) * 100, decimals: 2)
```

<a id="identifying-drop-off-points"></a>

## 4. Identifying Drop-Off Points

Drop-off analysis highlights where users abandon the process. The step with the highest drop-off rate is your biggest optimization opportunity.

```dql
// Visualize funnel steps as a bar chart to see drop-off
// Each row represents one funnel step
fetch bizevents, from:-24h
| filter in(event.type, {"com.myapp.product.viewed", "com.myapp.cart.updated", "com.myapp.checkout.started", "com.myapp.payment.processed", "com.myapp.order.completed"})
| summarize event_count = count(), by:{event.type}
| sort event_count desc
```

```dql
// Identify checkout abandonment — users who started checkout but did not pay
// Requires a user_id or session_id field in your business events
fetch bizevents, from:-24h
| filter in(event.type, {"com.myapp.checkout.started", "com.myapp.payment.processed"})
| filter isNotNull(user_id)
| summarize {started_checkout = countIf(event.type == "com.myapp.checkout.started"),
           completed_payment = countIf(event.type == "com.myapp.payment.processed")}, by:{user_id}
| filter started_checkout > 0 and completed_payment == 0
| summarize abandoned_users = count()
```

<a id="time-between-steps"></a>

## 5. Time Between Steps

Understanding how long users take between funnel steps reveals friction points. A long delay between "add to cart" and "checkout" may indicate a confusing checkout flow or price comparison behavior.

```dql
// Average time between cart and checkout (per user)
// Requires a user_id or session identifier
fetch bizevents, from:-24h
| filter in(event.type, {"com.myapp.cart.updated", "com.myapp.checkout.started"})
| filter isNotNull(user_id)
| summarize {cart_time = min(if(event.type == "com.myapp.cart.updated", then: timestamp)),
           checkout_time = min(if(event.type == "com.myapp.checkout.started", then: timestamp))}, by:{user_id}
| filter isNotNull(cart_time) and isNotNull(checkout_time)
| fieldsAdd time_to_checkout_sec = (toDouble(unixMillisFromTimestamp(checkout_time)) - toDouble(unixMillisFromTimestamp(cart_time))) / 1000.0
| filter time_to_checkout_sec > 0
| summarize {avg_seconds = avg(time_to_checkout_sec),
           median_seconds = median(time_to_checkout_sec),
           p95_seconds = percentile(time_to_checkout_sec, 95)}
```

<a id="funnel-segmentation"></a>

## 6. Funnel Segmentation

Segmenting funnels by dimensions — device type, geographic region, customer tier — reveals which segments convert better and which need attention.

```dql
// Funnel conversion rates segmented by event provider
// Useful for comparing web vs mobile vs API channels
fetch bizevents, from:-24h
| summarize {step1 = countIf(event.type == "com.myapp.product.viewed"),
           step5 = countIf(event.type == "com.myapp.order.completed")}, by:{event.provider}
| filter step1 > 0
| fieldsAdd conversion_pct = round(toDouble(step5) / toDouble(step1) * 100, decimals: 1)
| sort conversion_pct desc
```

```dql
// Funnel by category — compare product categories or business lines
fetch bizevents, from:-24h
| summarize {views = countIf(event.type == "com.myapp.product.viewed"),
           purchases = countIf(event.type == "com.myapp.order.completed")}, by:{event.category}
| filter views > 0
| fieldsAdd conversion_pct = round(toDouble(purchases) / toDouble(views) * 100, decimals: 1)
| sort conversion_pct desc
```

<a id="funnel-trends-over-time"></a>

## 7. Funnel Trends Over Time

Track how conversion rates change over time to measure the impact of feature releases, marketing campaigns, or incidents.

```dql
// Funnel step volumes over time
fetch bizevents, from:-7d
| filter in(event.type, {"com.myapp.product.viewed", "com.myapp.cart.updated", "com.myapp.checkout.started", "com.myapp.order.completed"})
| makeTimeseries event_count = count(), by:{event.type}, interval:6h
```

```dql
// Daily overall conversion rate trend over the past week
fetch bizevents, from:-7d
| fieldsAdd day = bin(timestamp, 1d)
| summarize {views = countIf(event.type == "com.myapp.product.viewed"),
           orders = countIf(event.type == "com.myapp.order.completed")}, by:{day}
| filter views > 0
| fieldsAdd conversion_pct = round(toDouble(orders) / toDouble(views) * 100, decimals: 2)
| sort day asc
```

<a id="summary-and-next-steps"></a>

## 8. Summary and Next Steps

In this notebook, you learned:

- **Funnel concepts** — Steps, conversion rates, drop-off rates
- **Building funnels with DQL** — Using `countIf` to count each step in a single query
- **Conversion rate calculation** — Step-by-step and overall rates
- **Drop-off analysis** — Identifying abandoned sessions
- **Time between steps** — Measuring friction via inter-step timing
- **Segmentation** — Comparing funnels by provider, category, or other dimensions
- **Trend analysis** — Tracking conversion rates over time

### Next Steps

- **BIZEV-04: Revenue Impact** — Connect funnel drop-offs to revenue loss during incidents
- **BIZEV-05: KPIs and Metrics** — Define business KPIs from event data

### References

- [Dynatrace Business Analytics](https://docs.dynatrace.com/docs/observe/business-observability)
- [DQL structuring commands (DT docs)](https://docs.dynatrace.com/docs/platform/grail/dynatrace-query-language/commands/structuring-commands)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
