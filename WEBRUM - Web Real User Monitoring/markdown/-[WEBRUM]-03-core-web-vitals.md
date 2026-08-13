# WEBRUM-03: Core Web Vitals

> **Series:** WEBRUM — Web Real User Monitoring | **Notebook:** 3 of 10 | **Created:** March 2026 | **Last Updated:** 08/12/2026

## Overview

Core Web Vitals (CWV) are Google's standardized metrics for measuring real-world user experience on the web. They focus on three pillars: loading performance, interactivity, and visual stability. Dynatrace captures these metrics via the RUM JavaScript agent using the browser's PerformanceObserver API, making them available for DQL analysis in Grail.

---

## Table of Contents

1. [Understanding Core Web Vitals](#understanding-cwv)
2. [Largest Contentful Paint (LCP)](#lcp)
3. [Interaction to Next Paint (INP)](#inp)
4. [Cumulative Layout Shift (CLS)](#cls)
5. [CWV by Page Group](#cwv-by-page)
6. [CWV Trends Over Time](#cwv-trends)
7. [CWV Scoring Dashboard](#cwv-scoring)
8. [Summary and Next Steps](#summary)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Dynatrace Environment** | SaaS with Grail enabled |
| **RUM Enabled** | Web applications with Core Web Vitals capture enabled |
| **Browser Support** | CWV are Chromium-based only (Chrome, Edge); Safari/Firefox have partial support |
| **Permissions** | `storage:events:read`, `storage:metrics:read` |
| **Previous Notebook** | WEBRUM-01: RUM Fundamentals |

<a id="understanding-cwv"></a>

## 1. Understanding Core Web Vitals

Google defines three Core Web Vitals that reflect real user experience:

| Metric | Full Name | Measures | Good | Needs Improvement | Poor |
|--------|-----------|----------|------|-------------------|------|
| **LCP** | Largest Contentful Paint | Loading performance | ≤ 2.5s | 2.5s – 4.0s | > 4.0s |
| **INP** | Interaction to Next Paint | Interactivity responsiveness | ≤ 200ms | 200ms – 500ms | > 500ms |
| **CLS** | Cumulative Layout Shift | Visual stability | ≤ 0.1 | 0.1 – 0.25 | > 0.25 |

> **Note:** INP replaced FID (First Input Delay) as a Core Web Vital in March 2024. INP measures the latency of *all* interactions during a session, not just the first one.

### How Dynatrace Captures CWV

The RUM JavaScript agent uses the browser's `PerformanceObserver` API to capture:

- **LCP** — Observed via `largest-contentful-paint` entry type
- **INP** — Calculated from `event` entry types (click, keypress, pointerdown)
- **CLS** — Accumulated from `layout-shift` entries (excluding user-initiated shifts)

These values are reported as part of the user action data and are available as metrics in Grail.

<a id="lcp"></a>

## 2. Largest Contentful Paint (LCP)

LCP measures the time from when the user initiates navigation to when the largest content element (image, text block, video) is rendered in the viewport. It answers: **"How quickly does the main content appear?"**

### Common LCP Elements

| Element Type | Example |
|-------------|----------|
| `<img>` | Hero image, product photo |
| `<video>` poster | Video thumbnail |
| Block-level text | Heading, paragraph |
| CSS `background-image` | Banner background |

### Factors That Impact LCP

- Server response time (TTFB)
- Render-blocking resources (CSS, JS)
- Resource load time (images, fonts)
- Client-side rendering delays

```dql
// Field vocabulary corrected 08/12/2026 — this series targets **New RUM**, but was written
// against names that are null on New RUM data, so these cells returned nothing while erroring
// nowhere. Verified against 5,556,127 user.events records (schema 0.24.0, javascript agent):
//   action.type == "Load"              -> characteristics.classifier == "page_summary"
//   action.type                        -> user_action.type      (hard_navigation | same_view)
//   action.name                        -> page.detected_name
//   web_vitals.largest_contentful_paint-> lcp.start_time        (327,099 populated)
//   web_vitals.cumulative_layout_shift -> cls.value             (387,254 populated)
//   app.name                           -> dt.rum.application.id
// UNITS CHANGE WITH THE FIELD. web_vitals.* was a nanosecond DURATION, so `/ 1ms` was correct for
// it; lcp.start_time is a PLAIN NUMBER already in milliseconds, and dividing it by 1ms yields
// null. Compare it against the 2500/4000 ms thresholds directly.
// `web_vitals.*` does still exist in the same schema, but carried 16 records in 30 days against
// lcp.*'s 327,099 — it is not a different RUM generation, just a rarely-populated sibling.
// Average LCP across all applications in the last 24 hours
fetch user.events, from:-24h
| filter characteristics.classifier == "page_summary"
| filter isNotNull(lcp.start_time)
| summarize avg_lcp = avg(lcp.start_time),
    p75_lcp = percentile(lcp.start_time, 75),
    p95_lcp = percentile(lcp.start_time, 95),
    sample_size = count(),
    by:{dt.rum.application.id}
| sort avg_lcp desc
```

```dql
// LCP distribution — classify into Good / Needs Improvement / Poor
//
// Corrected 08/12/2026: `fieldsAdd total = sum(action_count)` put an AGGREGATION in a row-wise
// stage, which fails with "Aggregations aren't allowed here". A percentage-of-total needs the
// grand total on every row, which means collapsing to one row, keeping the per-category rows in an
// array, and expanding back out.
fetch user.events, from:-24h
| filter characteristics.classifier == "page_summary"
| filter isNotNull(lcp.start_time)
| fieldsAdd lcp_ms = lcp.start_time
| fieldsAdd lcp_category = if(lcp_ms <= 2500, "Good",
    else: if(lcp_ms <= 4000, "Needs Improvement",
    else: "Poor"))
| summarize action_count = count(), by:{lcp_category}
| summarize rows = collectArray(record(lcp_category, action_count)), total = sum(action_count)
| expand rows
| fieldsAdd lcp_category = rows[lcp_category], action_count = rows[action_count]
| fieldsAdd percentage = round(action_count * 100.0 / total, decimals: 1)
| fieldsRemove rows
| sort action_count desc
```

<a id="inp"></a>

## 3. Interaction to Next Paint (INP)

INP measures the time from when a user interacts (click, tap, keypress) to when the browser renders the next frame reflecting that interaction. Unlike FID (which only measured the *first* interaction), INP considers *all* interactions and reports the worst one (approximated by the 98th percentile).

### What INP Captures

1. **Input delay** — Time from user interaction to event handler execution
2. **Processing time** — Time to execute event handlers
3. **Presentation delay** — Time from handler completion to next paint

### Common INP Bottlenecks

| Bottleneck | Symptom | Fix |
|-----------|---------|-----|
| Long tasks on main thread | High input delay | Break up long JavaScript tasks |
| Expensive event handlers | High processing time | Debounce, use web workers |
| Forced layout/reflow | High presentation delay | Batch DOM reads/writes |

```dql
// INP reshaped 08/12/2026 — New RUM publishes NO numeric INP on this schema. The only INP field
// is `inp.status`, a category: `below_threshold` (the interaction was fast enough that no INP value
// is reported — the healthy case, 369,268 events), `reported` (an INP value was actually recorded,
// i.e. a slow interaction), `not_reported`. There is nothing to average or take a percentile of, so
// an INP query is a RATE over statuses, not a latency distribution. Confirm on your own tenant with:
//   fetch user.events, from:-24h | filter isNotNull(inp.status) | summarize n = count(), by:{inp.status}
fetch user.events, from:-24h
| filter isNotNull(inp.status)
| summarize {
    interactions          = count(),
    slow_interactions     = countIf(inp.status == "reported"),
    below_threshold       = countIf(inp.status == "below_threshold")
  }, by:{dt.rum.application.id}
| fieldsAdd slow_pct = round(slow_interactions * 100.0 / interactions, decimals: 3)
| sort slow_pct desc
```

```dql
// INP reshaped 08/12/2026 — New RUM publishes NO numeric INP on this schema. The only INP field
// is `inp.status`, a category: `below_threshold` (the interaction was fast enough that no INP value
// is reported — the healthy case, 369,268 events), `reported` (an INP value was actually recorded,
// i.e. a slow interaction), `not_reported`. There is nothing to average or take a percentile of, so
// an INP query is a RATE over statuses, not a latency distribution. Confirm on your own tenant with:
//   fetch user.events, from:-24h | filter isNotNull(inp.status) | summarize n = count(), by:{inp.status}
fetch user.events, from:-24h
| filter isNotNull(inp.status)
| summarize {
    interactions      = count(),
    slow_interactions = countIf(inp.status == "reported")
  }, by:{page.detected_name}
| filter interactions > 10
| fieldsAdd slow_pct = round(slow_interactions * 100.0 / interactions, decimals: 3)
| sort slow_pct desc
| limit 10
```

<a id="cls"></a>

## 4. Cumulative Layout Shift (CLS)

CLS measures unexpected layout shifts during the entire lifespan of a page. A layout shift occurs when a visible element changes position from one rendered frame to the next without user interaction.

### Common Causes of CLS

| Cause | Example | Fix |
|-------|---------|-----|
| Images without dimensions | Image loads and pushes content down | Set `width` and `height` attributes |
| Dynamically injected content | Cookie banner pushes page down | Reserve space with CSS |
| Web fonts | FOUT (flash of unstyled text) causes text reflow | Use `font-display: swap` with fallback metrics |
| Ads/embeds | Third-party content loads late | Use `<iframe>` with fixed dimensions |

```dql
// CLS distribution — classify into Good / Needs Improvement / Poor
fetch user.events, from:-24h
| filter characteristics.classifier == "page_summary"
| filter isNotNull(cls.value)
| fieldsAdd cls_category = if(cls.value <= 0.1, "Good",
    else: if(cls.value <= 0.25, "Needs Improvement",
    else: "Poor"))
| summarize action_count = count(), by:{cls_category}
```

```dql
// Worst CLS pages — pages with the most layout shifting
fetch user.events, from:-24h
| filter characteristics.classifier == "page_summary"
| filter isNotNull(cls.value)
| summarize avg_cls = avg(cls.value),
    p75_cls = percentile(cls.value, 75),
    action_count = count(),
    by:{page.detected_name}
| filter action_count > 10
| sort p75_cls desc
| limit 10
```

<a id="cwv-by-page"></a>

## 5. CWV by Page Group

Aggregate Core Web Vitals by page to identify which pages need optimization:

```dql
// INP note (08/12/2026): New RUM publishes no numeric INP on this schema — only `inp.status`
// (`below_threshold` = fast enough that no value is reported, the healthy case; `reported` = a slow
// interaction was actually measured). So INP enters a scorecard as a RATE, not a percentile.
fetch user.events, from:-24h
| filter characteristics.classifier == "page_summary"
| summarize {
    p75_lcp_ms   = percentile(lcp.start_time, 75),
    p75_cls      = percentile(cls.value, 75),
    slow_inp_pct = round(countIf(inp.status == "reported") * 100.0 / count(), decimals: 3),
    page_views   = count()
  }, by:{page.detected_name}
| filter page_views > 20
| fieldsAdd lcp_status = if(p75_lcp_ms <= 2500, "Good", else: if(p75_lcp_ms <= 4000, "NI", else: "Poor")),
    cls_status = if(p75_cls <= 0.1, "Good", else: if(p75_cls <= 0.25, "NI", else: "Poor")),
    inp_status = if(slow_inp_pct <= 1.0, "Good", else: if(slow_inp_pct <= 5.0, "NI", else: "Poor"))
| sort page_views desc
| limit 15
```

<a id="cwv-trends"></a>

## 6. CWV Trends Over Time

Tracking CWV over time helps identify regressions after deployments, seasonal patterns, and the impact of optimizations.

```dql
// LCP trend over the last 7 days — hourly p75
fetch user.events, from:-7d
| filter characteristics.classifier == "page_summary"
| filter isNotNull(lcp.start_time)
| fieldsAdd lcp_ms = lcp.start_time
| makeTimeseries p75_lcp = percentile(lcp_ms, 75), interval:1h
```

```dql
// CLS trend over the last 7 days — daily p75
fetch user.events, from:-7d
| filter characteristics.classifier == "page_summary"
| filter isNotNull(cls.value)
| makeTimeseries p75_cls = percentile(cls.value, 75), interval:1d
```

<a id="cwv-scoring"></a>

## 7. CWV Scoring Dashboard

Create a single-query CWV scorecard showing the percentage of page loads in each category:

```dql
// INP note (08/12/2026): New RUM publishes no numeric INP on this schema — only `inp.status`
// (`below_threshold` = fast enough that no value is reported, the healthy case; `reported` = a slow
// interaction was actually measured). So INP enters a scorecard as a RATE, not a percentile.
fetch user.events, from:-24h
| filter characteristics.classifier == "page_summary"
| summarize {
    total     = count(),
    lcp_good  = countIf(lcp.start_time <= 2500),
    lcp_poor  = countIf(lcp.start_time > 4000),
    cls_good  = countIf(cls.value <= 0.1),
    cls_poor  = countIf(cls.value > 0.25),
    inp_fast  = countIf(inp.status == "below_threshold"),
    inp_slow  = countIf(inp.status == "reported")
  }
| fieldsAdd lcp_good_pct = round(lcp_good * 100.0 / total, decimals: 1),
    lcp_poor_pct = round(lcp_poor * 100.0 / total, decimals: 1),
    cls_good_pct = round(cls_good * 100.0 / total, decimals: 1),
    cls_poor_pct = round(cls_poor * 100.0 / total, decimals: 1),
    inp_fast_pct = round(inp_fast * 100.0 / total, decimals: 1),
    inp_slow_pct = round(inp_slow * 100.0 / total, decimals: 3)
```

<a id="summary"></a>

## 8. Summary and Next Steps

In this notebook, we covered:

- **Core Web Vitals overview** — LCP, INP, and CLS with Google's thresholds
- **LCP analysis** — Loading performance measurement and distribution
- **INP analysis** — Interactivity measurement replacing FID
- **CLS analysis** — Visual stability scoring and worst pages
- **Page-level breakdown** — CWV per page group for targeted optimization
- **Trend tracking** — CWV over time for regression detection
- **Scoring dashboard** — Unified CWV scorecard query

### Next Steps

- **WEBRUM-04: Session Analysis** — User journey mapping and conversion tracking
- **WEBRUM-06: Performance Analysis** — Deeper performance waterfall analysis beyond CWV

### References

- [Google Core Web Vitals](https://web.dev/vitals/)
- [Experience Vitals (DT docs)](https://docs.dynatrace.com/docs/observe/digital-experience/new-rum-experience/experience-vitals)
- [INP Documentation](https://web.dev/inp/)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
