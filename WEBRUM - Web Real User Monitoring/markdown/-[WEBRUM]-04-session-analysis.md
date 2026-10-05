# WEBRUM-04: Session Analysis

> **Series:** WEBRUM — Web Real User Monitoring | **Notebook:** 4 of 10 | **Created:** March 2026 | **Last Updated:** 10/05/2026

## Overview

User session analysis is the foundation of understanding how real users interact with your web applications. A session represents a complete visit — from the first page load to the last interaction before timeout. By analyzing sessions, you can identify user journey patterns, measure engagement, track conversions, calculate bounce rates, and understand how geographic location and device type impact the user experience.

---

## Table of Contents

1. [Session Properties](#session-properties)
2. [Session Segmentation](#session-segmentation)
3. [User Journey Mapping](#user-journey-mapping)
4. [Conversion Tracking](#conversion-tracking)
5. [Bounce Rate Analysis](#bounce-rate)
6. [Geographic Analysis](#geographic-analysis)
7. [Device and Browser Analysis](#device-analysis)
8. [Summary and Next Steps](#summary)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Dynatrace Environment** | SaaS with Grail enabled |
| **RUM Enabled** | Web applications with session capture active |
| **Session Properties** | Custom session properties configured (optional but recommended) |
| **Permissions** | `storage:events:read` |
| **Previous Notebook** | WEBRUM-01: RUM Fundamentals |

<a id="session-properties"></a>

## 1. Session Properties

Dynatrace automatically captures standard session properties and allows you to define custom ones:

### Standard Properties

| Property | Description | Example Values |
|----------|-------------|----------------|
| `dt.rum.session.id` | Unique session identifier | `23626166142035610_1-0` |
| `dt.rum.user_type` | Session classification | `real_user`, `robot`, `synthetic` |
| `start_time` / `end_time` | Session start and end — `user.sessions` has no populated `timestamp` | `2026-10-05T15:18:09Z` |
| `duration` | Total session duration | `300000000000` (nanoseconds) |
| `user_action_count` | Number of user actions | `15` |
| `navigation_count` | Number of navigations | `4` |
| `error.count` | Number of errors in session | `2` |
| `geo.country.iso_code` | User's country | `US` |
| `os.name` | Operating system | `Windows`, `macOS`, `iOS` |
| `browser.name` | Browser | `Chrome`, `Firefox`, `Safari` |
| `browser.window.width` | Browser window width in pixels | `1920` |
| `browser.window.height` | Browser window height in pixels | `1080` |

These are the New RUM names the queries below use. The classic names (`sessionId`, `userType`, `userActionCount`, `totalErrorCount`, `osFamily`, `browserFamily`) read null on New RUM data, so a query copied with them returns nothing and raises no error. Web RUM has no connection-type field: `network.connection.type` is documented as *"Only supported by OneAgent for Mobile."*

> <sub>**Sources:** [User events — semantic dictionary (DT docs)](https://docs.dynatrace.com/docs/semantic-dictionary/model/rum/user-events) — *"The internet connection type. Only supported by OneAgent for Mobile."* **Dictionary:** model `rum.user_session` lists `start_time`, `end_time`, `duration`, `user_action_count`, `navigation_count`, `error.count`, `dt.rum.session.id`, `dt.rum.user_type`, `geo.country.iso_code`, `os.name`, `browser.name`, `browser.window.width` / `.height` and no `timestamp`, read 10/05/2026.</sub>

### Custom Session Properties

Define custom properties via **Settings > Web and mobile monitoring > Session and user action properties**. Common examples:

- `session.user_id` — Authenticated user identifier
- `session.plan_type` — Subscription tier (free, pro, enterprise)
- `session.cart_value` — Shopping cart total
- `session.ab_variant` — A/B test variant

```dql
// Session/performance field vocabulary corrected 08/12/2026 (New RUM). Classic camelCase RUM
// names are null on New RUM data and fail silently. Verified against 3,261 user.sessions:
//   userType -> dt.rum.user_type      userActionCount -> user_action_count
//   totalErrorCount -> error.count    sessionId -> dt.rum.session.id
//   hasSessionReplay -> characteristics.has_replay
//   browserFamily -> browser.name     osFamily -> os.name
//   application -> primary_tags.application
//   country/city/continent -> geo.country.name / geo.city.name / geo.continent.name
//   screen.width|height -> browser.window.width|height
//   dom.interactive.time -> performance.dom_interactive
//   load.event.time -> performance.load_event_end
//   server.time -> ttfb.value (page summaries; web_vitals.time_to_first_byte where populated) —
//                  NOT ttfb.waiting_duration, which is the pre-request wait/redirect phase
// TWO TENANT CAVEATS on the validation tenant, both of which leave a CORRECT query empty:
//   * every session is dt.rum.user_type == "synthetic", so a real-user filter matches nothing —
//     the documented values are "real_user" / "robot" / "synthetic" (lowercase);
//   * geo.* is 0-populated, because synthetic traffic carries no geolocation.
// Explore session data — view available fields
fetch user.sessions, from:-1h
| filter dt.rum.user_type == "real_user"
| limit 5
```

<a id="session-segmentation"></a>

## 2. Session Segmentation

Segmenting sessions helps identify patterns across different user groups. Common segmentation dimensions include engagement level, error impact, and return frequency.

```dql
// Engagement segmentation — bucket sessions by number of actions. Sessions with zero user actions
// get their own bucket; they used to fall into "Low (2-3 actions)" (corrected 10/05/2026).
fetch user.sessions, from:-24h
| filter dt.rum.user_type == "real_user"
| fieldsAdd engagement = if(user_action_count == 0, "No actions",
    else: if(user_action_count == 1, "Bounce",
    else: if(user_action_count <= 3, "Low (2-3 actions)",
    else: if(user_action_count <= 10, "Medium (4-10 actions)",
    else: "High (10+ actions)"))))
| summarize {session_count = count(),
    avg_duration = avg(duration),
    avg_errors = avg(error.count)},
    by:{engagement}
| sort session_count desc
```

```dql
// Error-impacted sessions — how many sessions had errors?
fetch user.sessions, from:-24h
| filter dt.rum.user_type == "real_user"
| summarize {total_sessions = count(),
    error_sessions = countIf(error.count > 0)},
    by:{primary_tags.application}
| fieldsAdd error_session_pct = round(toDouble(error_sessions) / toDouble(total_sessions) * 100.0, decimals: 1)
| sort error_session_pct desc
```

<a id="user-journey-mapping"></a>

## 3. User Journey Mapping

Understanding the sequence of actions within sessions reveals common navigation paths, dead ends, and drop-off points.

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
// Top entry pages — where do users start their journey?
fetch user.events, from:-24h
| filter characteristics.has_page_summary == true
| summarize first_action = takeFirst(page.detected_name), by:{dt.rum.session.id}
| summarize entry_count = count(), by:{first_action}
| sort entry_count desc
| limit 10
```

```dql
// Top exit pages — where do users leave?
fetch user.events, from:-24h
| filter characteristics.has_page_summary == true
| summarize last_action = takeLast(page.detected_name), by:{dt.rum.session.id}
| summarize exit_count = count(), by:{last_action}
| sort exit_count desc
| limit 10
```

```dql
// Session duration distribution by hour of day — when are users most active?
fetch user.sessions, from:-7d
| filter dt.rum.user_type == "real_user"
| fieldsAdd hour = getHour(start_time)   // user.sessions has no populated timestamp
| summarize {session_count = count(),
    avg_duration_sec = avg(duration / 1s)},
    by:{hour}
| sort hour asc
```

<a id="conversion-tracking"></a>

## 4. Conversion Tracking

Conversion tracking identifies which sessions completed a desired business action (purchase, signup, form submission). This requires either:

- **Session properties** marking conversion events
- **Specific user actions** that indicate conversion (e.g., a "Thank You" page load)

### Conversion via Action Name Detection

If your conversion page has a recognizable URL pattern, you can detect conversions from user actions:

```dql
// Conversion rate — share of real-user sessions that reached a checkout/confirmation page.
// Adapt the page-name tests to match your conversion page URL pattern.
// Computed from the events side alone (corrected 10/05/2026): the previous version appended two
// unrelated row sets and never produced a rate.
fetch user.events, from:-24h
| filter dt.rum.user_type == "real_user"
| filter characteristics.has_page_summary == true
| fieldsAdd is_conversion_page = contains(page.detected_name, "confirmation") or contains(page.detected_name, "thank-you") or contains(page.detected_name, "checkout-success")
| summarize {conversion_views = countIf(is_conversion_page)}, by:{dt.rum.session.id}
| summarize {sessions = count(), converted_sessions = countIf(conversion_views > 0)}
| fieldsAdd conversion_rate_pct = round(toDouble(converted_sessions) / toDouble(sessions) * 100.0, decimals: 2)
```

> **Tip:** For accurate conversion tracking, use Dynatrace session properties to flag conversion events. This avoids reliance on URL pattern matching, which can be fragile.

<a id="bounce-rate"></a>

## 5. Bounce Rate Analysis

A "bounce" is a session with only one user action — the user loaded one page and left. High bounce rates may indicate poor landing page relevance, slow performance, or broken functionality.

Dynatrace's own definition: *"A session with only one user action or navigation is tagged as bounced."* The queries in this series count a bounce as `user_action_count == 1`, the same rule in WEBRUM-04, -05, -08 and -09. Sessions with zero user actions are kept out of the bounce count and shown separately in the engagement query above. Dynatrace's definition also counts a session whose only activity is a single navigation, so compare the result with the **Bounced Sessions** filter in Users & Sessions on your tenant before treating the two as identical.

> <sub>**Sources:** [User sessions in web frontends (DT docs)](https://docs.dynatrace.com/docs/observe/digital-experience/rum/web-frontends/concepts/user-sessions-web) — *"A session with only one user action or navigation is tagged as bounced."*</sub>

```dql
// Bounce rate by application
fetch user.sessions, from:-24h
| filter dt.rum.user_type == "real_user"
| summarize {total_sessions = count(),
    bounced_sessions = countIf(user_action_count == 1)},
    by:{primary_tags.application}
| fieldsAdd bounce_rate_pct = round(toDouble(bounced_sessions) / toDouble(total_sessions) * 100.0, decimals: 1)
| sort bounce_rate_pct desc
```

```dql
// Bounce rate trend over 7 days — track improvement over time
fetch user.sessions, from:-7d
| filter dt.rum.user_type == "real_user"
| fieldsAdd is_bounce = if(user_action_count == 1, 1, else: 0)
| makeTimeseries {total = count(), bounces = sum(is_bounce)}, interval:24h
| fieldsAdd bounce_rate = bounces[] * 100.0 / total[]   // per-day rate, not one collapsed number
```

<a id="geographic-analysis"></a>

## 6. Geographic Analysis

RUM data includes geographic information derived from the user's IP address. This helps identify regional performance differences and target optimization efforts.

```dql
// Sessions by country — top 15 countries by volume
fetch user.sessions, from:-24h
| filter dt.rum.user_type == "real_user"
| filter isNotNull(geo.country.name)
| summarize {session_count = count(),
    avg_actions = avg(user_action_count),
    avg_errors = avg(error.count),
    avg_duration_sec = avg(duration / 1s)},
    by:{geo.country.name}
| sort session_count desc
| limit 15
```

```dql
// Top 10 cities by session count with average performance
fetch user.sessions, from:-24h
| filter dt.rum.user_type == "real_user"
| filter isNotNull(geo.city.name) and isNotNull(geo.country.name)
| summarize session_count = count(), by:{geo.city.name, geo.country.name}
| sort session_count desc
| limit 10
```

<a id="device-analysis"></a>

## 7. Device and Browser Analysis

Understanding the device and browser mix helps prioritize testing and optimization efforts.

```dql
// Session distribution by browser
fetch user.sessions, from:-24h
| filter dt.rum.user_type == "real_user"
| filter isNotNull(browser.name)
| summarize {session_count = count(),
    avg_errors = avg(error.count)},
    by:{browser.name}
| sort session_count desc
| limit 10
```

```dql
// Session distribution by operating system
fetch user.sessions, from:-24h
| filter dt.rum.user_type == "real_user"
| filter isNotNull(os.name)
| summarize {session_count = count(),
    avg_duration_sec = avg(duration / 1s)},
    by:{os.name}
| sort session_count desc
```

```dql
// Screen resolution distribution — identify common viewport sizes
fetch user.sessions, from:-24h
| filter dt.rum.user_type == "real_user"
| filter isNotNull(browser.window.width) and isNotNull(browser.window.height)
| fieldsAdd resolution = concat(toString(browser.window.width), "x", toString(browser.window.height))
| summarize session_count = count(), by:{resolution}
| sort session_count desc
| limit 10
```

<a id="summary"></a>

## 8. Summary and Next Steps

In this notebook, we covered:

- **Session properties** — Standard and custom properties for segmentation
- **Engagement segmentation** — Bucketing sessions by interaction depth
- **User journey mapping** — Entry/exit pages and activity by time of day
- **Conversion tracking** — Identifying sessions that completed business goals
- **Bounce rate analysis** — Measuring and trending single-action sessions
- **Geographic analysis** — Session distribution by country and city
- **Device/browser analysis** — Understanding the technology mix of your users

### Next Steps

- **WEBRUM-05: Error Analysis** — JavaScript error tracking and impact analysis
- **WEBRUM-06: Performance Analysis** — Deep dive into page load waterfall timings

### References

- [Define user action and session properties (DT docs)](https://docs.dynatrace.com/docs/observe/digital-experience/rum-classic/web-applications/additional-configuration/define-user-action-and-session-properties)
- [USQL — custom session queries (DT docs)](https://docs.dynatrace.com/docs/observe/digital-experience/rum-classic/session-segmentation/custom-queries-segmentation-and-aggregation-of-session-data)
- [User events — semantic dictionary (DT docs)](https://docs.dynatrace.com/docs/semantic-dictionary/model/rum/user-events) — *"Used for internal optimization when storing the data and not intended for query usage."*
- [Semantic Dictionary changelog 1.349 (DT docs)](https://docs.dynatrace.com/docs/semantic-dictionary/changelog/version-1-349)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
