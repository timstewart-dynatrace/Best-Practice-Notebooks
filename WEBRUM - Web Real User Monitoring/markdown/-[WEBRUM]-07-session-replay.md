# WEBRUM-07: Session Replay

> **Series:** WEBRUM — Web Real User Monitoring | **Notebook:** 7 of 10 | **Created:** March 2026 | **Last Updated:** 08/03/2026

## Overview

Session Replay allows you to visually replay a user's interaction with your web application — seeing exactly what the user saw, where they clicked, how they scrolled, and when errors occurred. Unlike traditional analytics that show numbers, session replay provides qualitative context for quantitative data.

---

## Table of Contents

1. [How Session Replay Works](#how-it-works)
2. [Data Privacy and Masking](#data-privacy)
3. [Session Replay Configuration](#configuration)
4. [Finding Replay-Eligible Sessions](#finding-replays)
    - [When a Session Has No Replay](#no-replay)
5. [Correlating Replay with Performance](#correlating-data)
6. [Identifying UX Issues via Replay](#ux-issues)
7. [Third-Party Replay Integration](#third-party)
8. [Summary and Next Steps](#summary)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Dynatrace Environment** | SaaS with Session Replay license |
| **Session Replay Enabled** | Configured for at least one web application |
| **Permissions** | `storage:events:read`, Session Replay viewer permission |
| **Previous Notebooks** | WEBRUM-01: RUM Fundamentals, WEBRUM-04: Session Analysis |

<a id="how-it-works"></a>

## 1. How Session Replay Works

Dynatrace Session Replay captures the visual state of the web page as the user interacts with it. It does **not** record video — instead, it records DOM mutations:

### Recording Mechanism

1. **Initial Snapshot** — When a page loads, the RUM agent captures the full DOM structure and inline styles
2. **Mutation Recording** — As the user interacts, DOM mutations (element changes, attribute modifications, text updates) are recorded incrementally
3. **User Events** — Mouse movements, clicks, scrolls, and keyboard inputs are captured with timestamps
4. **Beacon Delivery** — Recorded data is sent to Dynatrace in compressed beacons
5. **Server-Side Reconstruction** — Dynatrace reconstructs the visual experience from the DOM mutations and events

### Recording Modes

| Mode | Description | Data Volume | Privacy Impact |
|------|-------------|-------------|----------------|
| **Visual replay** | Full DOM + CSS reconstruction | High | Highest — captures visible content |
| **Resource capture** | Includes external resources (images, CSS files) | Very high | Captures all visual assets |
| **Minimal mode** | DOM structure only, no external resources | Low | Lower — placeholder images |

> **Important:** Session Replay data is stored separately from RUM metrics and has its own retention period (typically 35 days by default in the classic model). Replay storage counts against your session replay quota.

> **Update (SaaS 1.343, July 2026 — staged tenant rollout) — Session Replay reaches general availability on Grail.** Replay data is stored natively in the Grail data lakehouse (`fetch user.replays`), with the video player, timeline, and multi-platform support built on that storage. On Grail-based tenants, replay retention follows the Grail bucket configuration rather than the classic quota model above. The classic model remains current for tenants that have not yet received 1.343 or still store replay in the classic surface — verify which model your tenant uses before planning retention.

<a id="data-privacy"></a>

## 2. Data Privacy and Masking

Session Replay captures real user interactions, which may include sensitive data. Proper masking is essential for compliance with GDPR, CCPA, HIPAA, and other privacy regulations.

### Masking Levels

| Level | Behavior | Use Case |
|-------|----------|----------|
| **Open** | No masking — all content visible | Non-sensitive public pages only |
| **Mask user input** | Input field values replaced with asterisks | Default — protects typed data |
| **Mask all text** | All text content replaced with blocks | Strict privacy environments |
| **Block element** | Entire elements replaced with placeholders | Specific sensitive components |

### HTML Attributes for Granular Control

Apply masking directly in your HTML:

```html
<!-- Mask specific input -->
<input type="text" data-dtrum-mask="true" />

<!-- Block entire section from replay -->
<div data-dtrum-block="true">
  <p>Sensitive content here</p>
</div>

<!-- Allow recording of specific element (override page-level masking) -->
<span data-dtrum-unmask="true">Public content</span>
```

### CSS Selector Masking Rules

Configure in **Settings > Web and mobile monitoring > Session Replay > Data privacy**:

| Selector | Effect |
|----------|--------|
| `.credit-card-form input` | Mask all inputs in credit card forms |
| `[data-sensitive]` | Mask elements with custom attribute |
| `.pii-container` | Block entire PII containers |

> **Warning:** Always err on the side of over-masking. It is better to mask non-sensitive content than to accidentally capture PII. Conduct a privacy review before enabling Session Replay in production.

<a id="configuration"></a>

## 3. Session Replay Configuration

### Sampling Rate

Not every session needs to be recorded. Configure sampling to balance insight with cost:

| Scenario | Recommended Sample Rate |
|----------|------------------------|
| High-traffic production site | 1-5% |
| Internal application | 10-25% |
| Pre-production testing | 50-100% |
| Investigating specific issue | 100% (temporarily) |

### Conditional Capture

Use capture rules to record specific session types:

- **Error-triggered replay** — Automatically capture replay when JavaScript errors occur
- **Conversion-path replay** — Capture sessions that reach checkout pages
- **Frustration-triggered replay** — Capture sessions with rage clicks or exit intent

### Storage Considerations

| Factor | Impact on Storage |
|--------|-------------------|
| Page complexity (DOM size) | More complex pages = larger snapshots |
| Session duration | Longer sessions = more mutation data |
| SPA vs. traditional | SPAs generate more mutations from in-page updates |
| Resource capture enabled | Significantly increases storage (images, CSS) |

<a id="finding-replays"></a>

## 4. Finding Replay-Eligible Sessions

Use DQL to identify sessions that have replay data available, then navigate to them in the Dynatrace UI.

```dql
// Sessions with replay data available in the last 24 hours
fetch user.sessions, from:-24h
| filter userType == "REAL_USER"
| filter isNotNull(hasSessionReplay) and hasSessionReplay == true
| summarize replay_sessions = count(),
    avg_actions = avg(userActionCount),
    avg_errors = avg(totalErrorCount),
    by:{application}
| sort replay_sessions desc
```

```dql
// Find replay sessions with errors — prioritize these for review
fetch user.sessions, from:-24h
| filter userType == "REAL_USER"
| filter isNotNull(hasSessionReplay) and hasSessionReplay == true
| filter totalErrorCount > 0
| fieldsKeep sessionId, application, duration, userActionCount, totalErrorCount, country, browserFamily
| sort totalErrorCount desc
| limit 20
```

```dql
// Replay coverage — what percentage of sessions have replay?
fetch user.sessions, from:-24h
| filter userType == "REAL_USER"
| summarize total_sessions = count(),
    replay_sessions = countIf(isNotNull(hasSessionReplay) and hasSessionReplay == true),
    by:{application}
| fieldsAdd replay_coverage_pct = round(toDouble(replay_sessions) / toDouble(total_sessions) * 100.0, decimals: 1)
| sort total_sessions desc
```

<a id="no-replay"></a>

### When a Session Has No Replay

The queries above filter for `hasSessionReplay == true`. A `false` — or a null — is neither an error nor necessarily a misconfiguration. Replay is *conditional* by design, and several independent gates can each suppress it.

| Cause | What actually happened | Where to check |
|-------|------------------------|----------------|
| **Outside the sampling rate** | By far the commonest cause. At a 2% sample rate, 98% of sessions correctly have no replay | Session Replay sampling rate for the application (§3) |
| **Replay disabled for the application** | Session Replay is configured per application, not globally — a tenant with replay enabled can still have applications without it | Settings → Session Replay, per application |
| **Unsupported browser** | Replay depends on DOM-mutation capture the browser must support; older and niche browsers are excluded | `browserFamily` / `browserMajorVersion` on the session |
| **A page blocked capture** | Page- or element-level privacy rules (`data-dtrum-block`, CSS selector block rules) suppress recording — sometimes on exactly the pages you care about (§2) | Session Replay data-privacy configuration |
| **Quota exhausted** | Replay storage counts against a quota; once consumed, subsequent eligible sessions are not recorded until the period resets | Session Replay quota / consumption |

**Forthcoming — SaaS 1.344 (released 07/27/2026, staged tenant rollout from 07/29/2026): Session Replay reports the reason itself.** Users & Sessions gains specific information about why session recordings become unavailable, which collapses the elimination exercise below into a single read in the UI. Verify 1.344 has reached your tenant before relying on it; until then — and for any tenant still on an earlier sprint — the manual order below is the working path.

**Eliminate in this order.** The sequence is deliberate: cheapest and most likely first, so you rarely reach the bottom.

1. **Sampling rate.** Compute actual replay coverage (the third query in this section) and compare it against the configured rate. If coverage ≈ the configured rate, **nothing is broken** — you are looking at sampling, and the fix is to raise the rate, not to debug.
2. **Application enablement.** If coverage is ~0% for one application while others are healthy, replay is off for that application. The coverage query already groups `by:{application}`, so this shows up without extra work.
3. **Browser.** Compare `browserFamily` and `browserMajorVersion` between sessions that do and do not have replay. A cause concentrated in one browser family is a support gap, not a configuration error — and not something you can configure away.
4. **Quota.** Check replay consumption last. A quota that ran out mid-period has a distinctive signature: replay present in earlier sessions, absent in later ones, configuration unchanged throughout.

That order also sorts by what you can act on — 1 and 2 are configuration you control, 3 is a platform constraint, 4 is a budget decision.

> <sub>**Sources:** [Dynatrace SaaS release notes 1.344 (DT docs)](https://docs.dynatrace.com/docs/whats-new/saas/sprint-344) — Session Replay unavailability details in Users & Sessions; [Session Replay (DT docs)](https://docs.dynatrace.com/docs/observe/digital-experience/session-replay). **Derived:** the four-step elimination order combines the documented ineligibility causes with their relative likelihood and remediability — verify the coverage figures against your own configured sampling rate.</sub>

<a id="correlating-data"></a>

## 5. Correlating Replay with Performance

Session Replay is most powerful when correlated with performance data. Use DQL to find sessions that experienced specific performance issues, then review the replay to understand the visual impact.

```dql
// Slow sessions with replay — sessions with poor page load times
fetch user.sessions, from:-24h
| filter userType == "REAL_USER"
| filter isNotNull(hasSessionReplay) and hasSessionReplay == true
| fieldsAdd duration_sec = duration / 1s
| filter duration_sec > 120
| fieldsKeep sessionId, application, duration_sec, userActionCount, totalErrorCount, country
| sort duration_sec desc
| limit 10
```

```dql
// Sessions with rage clicks that have replay — top candidates for UX review
fetch user.events, from:-24h
| filter type == "RageClick"
| summarize rage_clicks = count(), by:{sessionId, application}
| sort rage_clicks desc
| limit 10
```

> **Tip:** Copy the `session.id` value from query results and paste it into the Dynatrace Session Replay search to jump directly to the replay.

<a id="ux-issues"></a>

## 6. Identifying UX Issues via Replay

Session Replay helps identify qualitative UX issues that metrics alone cannot reveal:

| UX Issue | What to Look For in Replay |
|----------|----------------------------|
| **Confusing navigation** | Users going back and forth, visiting the same pages repeatedly |
| **Broken forms** | Users attempting to submit forms multiple times |
| **Invisible errors** | JS errors that silently break functionality |
| **Misleading UI elements** | Users clicking on non-interactive elements |
| **Layout shifts** | Content jumping around during page load |
| **Slow feedback** | Long pauses after clicks before visual response |

### Replay Investigation Workflow

1. **Identify problematic sessions** via DQL (errors, rage clicks, abandonment)
2. **Watch the replay** in Dynatrace UI
3. **Correlate with waterfall** — Performance tab shows timing alongside replay
4. **Note the pattern** — Is this a one-off or a systematic issue?
5. **Quantify with DQL** — How many sessions are affected by this pattern?

```dql
// Abandoned sessions with replay — users who started but did not finish
fetch user.sessions, from:-24h
| filter userType == "REAL_USER"
| filter isNotNull(hasSessionReplay) and hasSessionReplay == true
| filter userActionCount >= 3 and userActionCount <= 5
| filter totalErrorCount > 0
| fieldsKeep sessionId, application, userActionCount, totalErrorCount, duration, country
| sort totalErrorCount desc
| limit 15
```

<a id="third-party"></a>

## 7. Third-Party Replay Integration

Some organizations use dedicated session replay tools alongside Dynatrace. Common integration patterns:

### Integration Approaches

| Tool | Integration Method | Use Case |
|------|-------------------|----------|
| **Dynatrace Session Replay** | Built-in — no integration needed | Full-stack correlation (backend + frontend) |
| **LogRocket** | Session ID correlation via session property | Product analytics focus |
| **FullStory** | Session URL in custom session property | UX research focus |
| **Hotjar** | Heatmaps + recordings complement RUM data | Heatmap analysis |

### Correlation Pattern

To link Dynatrace sessions with a third-party replay tool:

1. Capture the third-party session ID via the RUM JavaScript API:

```javascript
// Example: send LogRocket session URL to Dynatrace
LogRocket.getSessionURL(function(sessionURL) {
  dtrum.sendSessionProperties(
    null, null, 
    { "logrocket_url": sessionURL }
  );
});
```

2. Configure a custom string session property in Dynatrace for `logrocket_url`
3. Query sessions with the third-party URL for cross-referencing

> **Note:** Dynatrace's native Session Replay has the advantage of being fully integrated with the performance waterfall, backend traces, and Dynatrace Intelligence — third-party tools typically lack this correlation.

<a id="summary"></a>

## 8. Summary and Next Steps

In this notebook, we covered:

- **How Session Replay works** — DOM mutation recording, not video
- **Data privacy** — Masking levels, HTML attributes, CSS selector rules
- **Configuration** — Sampling rates, conditional capture, storage considerations
- **Finding replays** — DQL queries to locate replay-eligible sessions
- **When a session has no replay** — the five ineligibility causes, and the sampling → enablement → browser → quota elimination order (SaaS 1.344 surfaces the reason directly)
- **Performance correlation** — Linking slow/error sessions to replay
- **UX issue identification** — Patterns to look for in replay analysis
- **Third-party integration** — Connecting Dynatrace with external replay tools

### Next Steps

- **WEBRUM-08: Dashboards and Alerting** — Build RUM dashboards with Apdex and set up performance alerts

### References

- [Session Replay (DT docs)](https://docs.dynatrace.com/docs/observe/digital-experience/session-replay)
- [Session Replay with strong privacy requirements (DT docs)](https://docs.dynatrace.com/docs/observe/digital-experience/session-replay-latest/configure-session-replay-web/session-replay-strong-privacy-req-web)
- [Configure Session Replay for web applications (DT docs)](https://docs.dynatrace.com/docs/observe/digital-experience/session-replay/configure-session-replay-web)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
