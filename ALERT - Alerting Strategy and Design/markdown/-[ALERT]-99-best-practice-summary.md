# ALERT-99: Best-Practice Summary and Setup Checklist

> **Series:** ALERT — Alerting Strategy and Design | **Notebook:** 99 | **Created:** June 2026 | **Last Updated:** 08/27/2026

## Overview

The series on one page: the principles, a complete end-to-end setup checklist, and the cross-series map of where each piece is built. Use this as the standing reference once you have read ALERT-01 through ALERT-04.

---

## Table of Contents

1. [Principles](#principles)
2. [Complete Setup Checklist](#checklist)
3. [The Ongoing Audit Loop](#audit)
4. [Cross-Series Map](#map)

---

## Prerequisites

| Requirement | Details |
|-------------|---------|
| **Prior reading** | ALERT-01 through ALERT-04 |
| **Companion series** | AIOPS, SLO, WFLOW, OPIPE |

<a id="principles"></a>
## 1. Principles

1. **OOTB Davis first.** Tune before you build; descend the funnel only when the layer above cannot express the condition.
2. **One enriched problem is the hub.** Every mechanism converges on a Davis problem; everything downstream operates on it.
3. **Enrich upstream or you cannot route downstream.** Team/zone/severity belong on the problem before a workflow ever sees it.
4. **Burn-rate beats threshold-on-SLI.** Alert on how fast the budget drains, with multiwindow confirmation.
5. **Prefer simple workflows.** Pay for multi-step only when you need conditional/multi-team logic.
6. **Match destination to urgency.** Fast-burn → page; slow-burn → ticket.
7. **Validate every query before it becomes an alert.** A detector on a silent query is worse than no alert.
8. **Audit on a cadence.** Building well is half the job. A standing review — classify every problem, trace each false positive to the config behind it, and track the trend per area — keeps the funnel honest as services change. The 0.1%-of-observed-time yardstick in §3 is what turns “too noisy” from an opinion into a measurement.

<a id="checklist"></a>
## 2. Complete Setup Checklist

**Detection**
- [ ] Reviewed OOTB Davis anomaly detection settings and tuned sensitivity
- [ ] Custom anomaly detectors only where Davis does not cover the condition (auto-adaptive/seasonal, not static-on-traffic)
- [ ] Log/span signals extracted to OpenPipeline metrics before alerting on them
- [ ] SLOs defined for critical user journeys with burn-rate alerts
- [ ] New detectors deployed with relaxed thresholds first, tightened after 1–2 weeks of observation
- [ ] New detectors shadow-deployed as `CUSTOM_INFO` and measured before promotion to `CUSTOM_ALERT`
- [ ] Detectors alert on aggregated data, not split by high-cardinality dimensions (version, pod, status code)
- [ ] Sparse log/event conditions use records-based detectors with a large window and matching delay

**Enrichment**
- [ ] Detector event templates carry team / zone / service properties
- [ ] Entity tags and Smartscape ownership populated for OOTB problems
- [ ] `event.severity` used to drive priority
- [ ] Custom detector event templates set `dt.smartscape_source.id` to a real entity ID, so alerts correlate against the thing that broke instead of falling back to the environment entity and merging with unrelated alerts

**Routing**
- [ ] Problem-trigger workflows filter on enriched metadata
- [ ] Simple workflows used wherever conditional logic is not needed
- [ ] Fast-burn → page channel; slow-burn → ticket channel
- [ ] One notification per problem (no duplicate alerts on raw metrics)

**ITSM / destinations**
- [ ] ServiceNow incidents carry assignment group, priority, and a de-dup correlation key
- [ ] Legacy alerting-profile integrations identified; migration to workflows considered

**Governance**
- [ ] Production detectors/SLOs promoted to config-as-code
- [ ] Alerts that fire without action reviewed and tuned
- [ ] Quarterly removal pass run: never-fired detectors, orphaned entities, and problems open for weeks

<a id="audit"></a>
## 3. The Ongoing Audit Loop

The setup checklist above is build-time. Noise creeps back as services, traffic, and ownership change, so a standing audit is what keeps the funnel honest. The point is not to *count* false positives — it is to find the repeating configuration patterns behind them, so the same noise is not regenerated next period.

**The yardstick: 0.1% of observed time.** A healthy service should sit in an alert state no more than 0.1% of the time it is observed — about **1 to 1.5 minutes a day**, or roughly **43 minutes a month**. For a well-calibrated detector that means firing once or twice a *month*, not on a daily drumbeat.

The value of the number is not its precision. It is that it converts "this feels noisy" into something anyone can check: a detector firing several times a week is a tuning candidate by definition rather than by opinion, which is what lets you take the finding to an owning team without it becoming an argument. AIOPS-02 §8 carries the triage query that ranks detectors by firing count and shows, per detector, whether it is attributable and whether you can navigate to its configuration.

**Classify every problem.** Each problem in the audit window lands in exactly one bucket:

| Class | Meaning | Action |
|-------|---------|--------|
| **True Positive (TP)** | A real issue that required action | None — the detector is working |
| **False Positive (FP)** | Fired but needed no action — a threshold, scope, detection-method, or sliding-window issue | Trace it to the config and fix the source |
| **Under Review (UR)** | Not yet conclusive | Carry to the next period; gather more data points |

Every FP must trace back to a concrete cause — the analyzer type, the entity scope, the sliding window, or the event template. An FP with no configuration change behind it is just a number; an FP with an identified root cause is progress.

**Track one KPI:** the false-positive rate over time, per area, and whether it is trending down. That single trend is the primary output of every cycle.

**Signals to scan each cycle:**

| Signal | What it flags |
|--------|---------------|
| **Fire rate** | A detector firing many times a day is a tuning or de-dup candidate |
| **Silence rate** | Never fired in 90+ days → redundant, or watching a dead path |
| **Immediate-silence rate** | Acknowledged and closed with no action = noise |
| **Routing coverage** | Every alert should route to exactly one owner; an unrouted alert is invisible |
| **Runbook link** | An alert with no runbook attached has no possible action |
| **Owner / area property** | Present, so the alert maps to a team — also catches copy-paste clones that never set it |
| **Problem age** | A problem open for weeks is not a long incident — it is a misconfigured detector or an entity that no longer exists |
| **Correlation coverage** | Alerts with no `dt.smartscape_source.id` fall back to the environment entity, so they merge with unrelated alerts into problems with no usable root cause |

**Remove, do not only tune.** A review that only ever adjusts thresholds accumulates configuration. Each quarter, actively delete:

- Detectors firing more than a few times a week that nobody has acted on — noise carrying a maintenance cost.
- Detectors that have not fired in months. Either the condition cannot occur or something else is catching it; either way the configuration is dead weight, and its silence is indistinguishable from coverage.
- Alerts bound to orphaned entities — decommissioned services, retired application versions, hosts long gone.
- Detectors behind problems that have stayed open for weeks. Disabling one closes its problem and stops the wasted re-evaluation.

**Cadence:**

| Stage | When | Focus |
|-------|------|-------|
| **Pre-go-live** | Before an area or service reaches production | Every new or migrated detector fires correctly in non-prod; nothing fires on no-data |
| **Early** (first ~3 months) | Monthly | Top-noise alerts reviewed with the owning team |
| **Steady state** | Quarterly | Cross-area review; retire detectors silent 90+ days or at a 100% silence rate |

Express these as recurring queries, and pick the data object deliberately — there are **three** surfaces here and they answer different questions. Counts measured over the same 7 days on one tenant, 07/31/2026:

| Query | Returns | Use it for |
|---|---:|---|
| `fetch dt.davis.problems` | 3,318 | **Problem-level signals** — one row per problem. MTTR, severity rollups, problem age. |
| `fetch dt.davis.events` | 251,833 | **Detector-level noise ranking** — the firing stream (AIOPS-02 §8). |
| `fetch events \| filter event.kind == "DAVIS_PROBLEM"` | 2,480,783 | The **raw problem-event stream** — many records per problem as its state updates. Valid, but do not count these and call the result "problems": it over-counts by roughly 750× here. |

The trap worth naming: **`fetch dt.davis.events | filter event.kind == "DAVIS_PROBLEM"` returns exactly zero** — `dt.davis.events` carries only `DAVIS_EVENT` (AIOPS-03 §3). That is a filter on the wrong table, not a deprecated form, and it fails silently. The superficially similar `fetch events | filter event.kind == "DAVIS_PROBLEM"` is a different query against a different table and works fine. AIOPS-03 and ORGNZ-10 carry validated problem-feed patterns to build on.

Two of the signals above are worth turning into standing trend queries rather than one-off checks, both built from the two-data-object model in AIOPS-03 §3.

**Denoising ratio (events → problems).** How much is Causal AI's grouping actually consolidating? Divide problem-worthy events by the problems they produced, trended daily — a ratio sitting near 1 for a stretch means events are arriving as separate problems rather than merging. Read it alongside "Correlation coverage" above rather than as the same signal: poor correlation coverage pushes the ratio the *other* way, since environment-fallback events over-merge. As with the 0.1% yardstick, there's no universal healthy number here; trend it against your own baseline.

```dql
// Denoising ratio — problem-worthy events divided by resulting problems, trended daily
//
// Both streams must be confined to the SAME window. dt.davis.problems is fetched by
// record timestamp, but binned by event.start — so a problem that started before the
// window (or is still open from weeks ago) lands in a day bucket with zero events and
// reports denoising_ratio = 0. Verified 08/27/2026: without the event.start filter the
// first 12 rows were all such artifacts, dated back to 2026-06-01.
fetch dt.davis.events, from:-30d
| filter in(event.category, {"AVAILABILITY","ERROR","RESOURCE_CONTENTION","SLOWDOWN"})
| fieldsAdd kind = "event", day = bin(timestamp, 24h)
| append [
    fetch dt.davis.problems, from:-30d
    | filter not(dt.davis.is_duplicate)
    | filter event.start >= now() - 30d
    | fieldsAdd kind = "problem", day = bin(event.start, 24h)
  ]
| summarize {event_count = countIf(kind == "event"), problem_count = countIf(kind == "problem")}, by:{day}
| filter problem_count > 0 and event_count > 0
| fieldsAdd denoising_ratio = round(toDouble(event_count) / problem_count, decimals: 2)
| sort day asc
```

**Correlation coverage.** "Correlation coverage" in the table above names the mechanism: an event with no `dt.smartscape_source.id` is not unattributed but *mis*-attributed, falling back to the environment entity and merging with everything else that did the same (AIOPS-03 §1). Trended directly, it is the share of non-informational Davis events that carry the entity a meaningful chain needs:

```dql
// Correlation coverage — share of non-informational Davis events carrying a source-entity reference, trended daily
fetch dt.davis.events, from:-30d
| filter not(in(event.category, {"INFO","WARNING"}))
| fieldsAdd attributed_flag = if(isNotNull(dt.smartscape_source.id), 100.0, else: 0.0)
| makeTimeseries correlation_coverage_pct = avg(attributed_flag), interval:1d
```

**Reading the two together.** A falling correlation-coverage trend is a config problem — trace it to the event templates behind it (the Enrichment block of Section 2's checklist). A falling denoising ratio *despite* stable correlation coverage points the other way, at scoping or tuning rather than attribution.

> <sub>**Sources:** [Avoid overalerting (DT docs)](https://docs.dynatrace.com/docs/dynatrace-intelligence/use-cases/avoid-overalerting). **Derived:** the audit signals table extends the documented review targets with the correlation-coverage and problem-age checks implied by the correlation-key and long-running-alert guidance; the denoising-ratio and correlation-coverage trend queries apply the two-data-object model (AIOPS-03 §3) and the `dt.smartscape_source.id` correlation key (AIOPS-03 §1) to already-documented fields as standing metrics.</sub>

<a id="map"></a>
## 4. Cross-Series Map

| Piece | Built in |
|-------|----------|
| Anomaly detector mechanics (analyzer, tuning, event template) | AIOPS-02 |
| Metric events / detector schemas as code | AIOPS-02 §6, AUTOM-05/06 |
| OpenPipeline metric extraction | OPIPE, FAQ-09 |
| SLIs, error budgets, burn-rate alerting | SLO-02 / 03 / 04 |
| SLOs as code | SLO-05 |
| Routing, conditions, escalation | WFLOW-03 / 04 |
| HTTP / webhook actions | WFLOW-08 |
| ServiceNow incident integration | ALERT-04 |
| Cost / DPS framing | FINOPS, FAQ-09 |

> <sub>**Sources:** [Alerting and notifications (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/notifications-and-alerting), [Workflows (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/workflows). **Derived:** the checklist and funnel synthesise the ALERT, AIOPS, SLO, and WFLOW series into one operational sequence.</sub>

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
