# WEBRUM-03: Core Web Vitals

> **Series:** WEBRUM — Web Real User Monitoring | **Notebook:** 3 of 10 | **Created:** March 2026 | **Last Updated:** 10/05/2026

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
//   action.type == "Load"              -> characteristics.has_page_summary == true
//                                        (was characteristics.classifier — "not intended for query usage", SD 1.349)
//   action.type                        -> user_action.type      (hard_navigation | soft_navigation | same_view | api)
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
| filter characteristics.has_page_summary == true
| filter isNotNull(lcp.start_time)
| summarize {avg_lcp = avg(lcp.start_time),
    p75_lcp = percentile(lcp.start_time, 75),
    p95_lcp = percentile(lcp.start_time, 95),
    sample_size = count()},
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
| filter characteristics.has_page_summary == true
| filter isNotNull(lcp.start_time)
| fieldsAdd lcp_ms = lcp.start_time
| fieldsAdd lcp_category = if(lcp_ms <= 2500, "Good",
    else: if(lcp_ms <= 4000, "Needs Improvement",
    else: "Poor"))
| summarize action_count = count(), by:{lcp_category}
| summarize {rows = collectArray(record(lcp_category, action_count)), total = sum(action_count)}
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

### How INP Appears in New RUM

INP lives on page summary events as two fields:

- **`web_vitals.interaction_to_next_paint`** — the INP value itself, a duration (*"The Interaction to Next Paint (INP) value"*). Compare its p75 against Google's 200 ms / 500 ms thresholds.
- **`inp.status`** — why a value is or is not present. `below_threshold` means *"INP is not reported because the value is below the threshold of 40 milliseconds"*; `not_reported` means *"INP is not reported because no relevant user interaction happened"*; `reported` means a value was captured.

`reported` therefore does **not** mean "slow": a 60 ms interaction is reported and is Good against Google's 200 ms threshold. Score INP on the numeric value, and count `inp.status` on page summaries only — the field also appears on view summaries and user actions, so counting it across all events counts each page several times.

> <sub>**Sources:** [Navigation-related events — semantic dictionary (DT docs)](https://docs.dynatrace.com/docs/semantic-dictionary/model/rum/user-events/navigation-related) — *"INP is not reported because the value is below the threshold of 40 milliseconds."*; [Monitor web performance with DQL (DT docs)](https://docs.dynatrace.com/docs/observe/digital-experience/rum/analyze-and-alert/rum-dql-web-performance) (takes the p75 of `web_vitals.interaction_to_next_paint` on page summaries).</sub>

```dql
// INP corrected 10/05/2026. `web_vitals.interaction_to_next_paint` is the numeric INP (a
// duration) on page summaries; `inp.status` only says why it is or is not present:
// `below_threshold` = INP under Dynatrace's 40 ms reporting floor, `not_reported` = no relevant
// interaction, `reported` = a value was captured — NOT "slow". Score INP against Google's
// 200 / 500 ms on the value, and count page summaries only (inp.status also sits on view summaries
// and user actions, which counted each page about three times).
// Schema-verified, not execution-verified: the validation tenant holds only synthetic-monitor
// data, which never records an interaction, so the INP columns read null there.
// INP by application — measured coverage, p75 and poor share
fetch user.events, from:-24h
| filter characteristics.has_page_summary == true
| summarize {
    page_views     = count(),
    inp_measured   = countIf(isNotNull(web_vitals.interaction_to_next_paint)),
    p75_inp_ms     = percentile(web_vitals.interaction_to_next_paint, 75) / 1ms,
    inp_poor       = countIf(web_vitals.interaction_to_next_paint > 500ms),
    below_40ms     = countIf(inp.status == "below_threshold"),
    no_interaction = countIf(inp.status == "not_reported")
  }, by:{dt.rum.application.id}
| sort page_views desc
```

```dql
// INP corrected 10/05/2026. `web_vitals.interaction_to_next_paint` is the numeric INP (a
// duration) on page summaries; `inp.status` only says why it is or is not present:
// `below_threshold` = INP under Dynatrace's 40 ms reporting floor, `not_reported` = no relevant
// interaction, `reported` = a value was captured — NOT "slow". Score INP against Google's
// 200 / 500 ms on the value, and count page summaries only (inp.status also sits on view summaries
// and user actions, which counted each page about three times).
// Schema-verified, not execution-verified: the validation tenant holds only synthetic-monitor
// data, which never records an interaction, so the INP columns read null there.
// INP by page — worst p75 first
fetch user.events, from:-24h
| filter characteristics.has_page_summary == true
| summarize {
    page_views     = count(),
    inp_measured   = countIf(isNotNull(web_vitals.interaction_to_next_paint)),
    p75_inp_ms     = percentile(web_vitals.interaction_to_next_paint, 75) / 1ms,
    inp_poor       = countIf(web_vitals.interaction_to_next_paint > 500ms),
    below_40ms     = countIf(inp.status == "below_threshold"),
    no_interaction = countIf(inp.status == "not_reported")
  }, by:{page.detected_name}
| filter page_views > 10
| sort p75_inp_ms desc
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
| filter characteristics.has_page_summary == true
| filter isNotNull(cls.value)
| fieldsAdd cls_category = if(cls.value <= 0.1, "Good",
    else: if(cls.value <= 0.25, "Needs Improvement",
    else: "Poor"))
| summarize action_count = count(), by:{cls_category}
```

```dql
// Worst CLS pages — pages with the most layout shifting
fetch user.events, from:-24h
| filter characteristics.has_page_summary == true
| filter isNotNull(cls.value)
| summarize {avg_cls = avg(cls.value),
    p75_cls = percentile(cls.value, 75),
    action_count = count()},
    by:{page.detected_name}
| filter action_count > 10
| sort p75_cls desc
| limit 10
```

<a id="cwv-by-page"></a>

## 5. CWV by Page Group

Aggregate Core Web Vitals by page to identify which pages need optimization:

```dql
// INP corrected 10/05/2026: scored on the numeric `web_vitals.interaction_to_next_paint` p75
// against 200 / 500 ms — `inp.status == "reported"` only means the value cleared a 40 ms floor.
// A page with no measured value is "No data", not "Poor": a null p75 used to fall through the
// if-chain into the worst bucket (corrected 10/05/2026).
fetch user.events, from:-24h
| filter characteristics.has_page_summary == true
| summarize {
    p75_lcp_ms   = percentile(lcp.start_time, 75),
    p75_cls      = percentile(cls.value, 75),
    p75_inp_ms   = percentile(web_vitals.interaction_to_next_paint, 75) / 1ms,
    page_views   = count()
  }, by:{page.detected_name}
| filter page_views > 20
| fieldsAdd lcp_status = if(isNull(p75_lcp_ms), "No data", else: if(p75_lcp_ms <= 2500, "Good", else: if(p75_lcp_ms <= 4000, "NI", else: "Poor"))),
    cls_status = if(isNull(p75_cls), "No data", else: if(p75_cls <= 0.1, "Good", else: if(p75_cls <= 0.25, "NI", else: "Poor"))),
    inp_status = if(isNull(p75_inp_ms), "No data", else: if(p75_inp_ms <= 200, "Good", else: if(p75_inp_ms <= 500, "NI", else: "Poor")))
| sort page_views desc
| limit 15
```

<a id="cwv-trends"></a>

## 6. CWV Trends Over Time

Tracking CWV over time helps identify regressions after deployments, seasonal patterns, and the impact of optimizations.

```dql
// LCP trend over the last 7 days — hourly p75
fetch user.events, from:-7d
| filter characteristics.has_page_summary == true
| filter isNotNull(lcp.start_time)
| fieldsAdd lcp_ms = lcp.start_time
| makeTimeseries p75_lcp = percentile(lcp_ms, 75), interval:1h
```

```dql
// CLS trend over the last 7 days — daily p75
fetch user.events, from:-7d
| filter characteristics.has_page_summary == true
| filter isNotNull(cls.value)
| makeTimeseries p75_cls = percentile(cls.value, 75), interval:24h
```

<a id="cwv-scoring"></a>

## 7. CWV Scoring Dashboard

Create a single-query CWV scorecard showing, for each metric, the percentage of page loads in each category. Each percentage is taken over the page loads that measured that metric, so a page that never reported INP does not count as passing it:

```dql
// Corrected 10/05/2026: each "good %" / "poor %" divides by the page summaries that actually
// MEASURED that metric (lcp_n, cls_n, inp_n) — dividing by every page summary diluted the pass
// rates with pages that never reported the value. INP is scored on the numeric
// `web_vitals.interaction_to_next_paint` (≤ 200 ms good, > 500 ms poor), not on inp.status.
fetch user.events, from:-24h
| filter characteristics.has_page_summary == true
| summarize {
    total     = count(),
    lcp_n     = countIf(isNotNull(lcp.start_time)),
    lcp_good  = countIf(lcp.start_time <= 2500),
    lcp_poor  = countIf(lcp.start_time > 4000),
    cls_n     = countIf(isNotNull(cls.value)),
    cls_good  = countIf(cls.value <= 0.1),
    cls_poor  = countIf(cls.value > 0.25),
    inp_n     = countIf(isNotNull(web_vitals.interaction_to_next_paint)),
    inp_good  = countIf(web_vitals.interaction_to_next_paint <= 200ms),
    inp_poor  = countIf(web_vitals.interaction_to_next_paint > 500ms)
  }
| fieldsAdd lcp_good_pct = round(lcp_good * 100.0 / lcp_n, decimals: 1),
    lcp_poor_pct = round(lcp_poor * 100.0 / lcp_n, decimals: 1),
    cls_good_pct = round(cls_good * 100.0 / cls_n, decimals: 1),
    cls_poor_pct = round(cls_poor * 100.0 / cls_n, decimals: 1),
    inp_good_pct = round(inp_good * 100.0 / inp_n, decimals: 1),
    inp_poor_pct = round(inp_poor * 100.0 / inp_n, decimals: 1)
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

- [Google Core Web Vitals](https://web.dev/articles/vitals)
- [Experience Vitals (DT docs)](https://docs.dynatrace.com/docs/observe/digital-experience/rum/experience-vitals)
- [INP Documentation](https://web.dev/articles/inp)
- [User events — semantic dictionary (DT docs)](https://docs.dynatrace.com/docs/semantic-dictionary/model/rum/user-events) — *"Used for internal optimization when storing the data and not intended for query usage."*
- [Semantic Dictionary changelog 1.349 (DT docs)](https://docs.dynatrace.com/docs/semantic-dictionary/changelog/version-1-349)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
