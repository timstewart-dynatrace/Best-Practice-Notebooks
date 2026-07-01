# ALERT-99: Best-Practice Summary and Setup Checklist

> **Series:** ALERT — Alerting Strategy and Design | **Notebook:** 99 of 05 | **Created:** June 2026 | **Last Updated:** 06/25/2026

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
8. **Audit on a cadence.** Building well is half the job. A standing review — classify every problem, trace each false positive to the config behind it, and track the trend per area — keeps the funnel honest as services change.

<a id="checklist"></a>
## 2. Complete Setup Checklist

**Detection**
- [ ] Reviewed OOTB Davis anomaly detection settings and tuned sensitivity
- [ ] Custom anomaly detectors only where Davis does not cover the condition (auto-adaptive/seasonal, not static-on-traffic)
- [ ] Log/span signals extracted to OpenPipeline metrics before alerting on them
- [ ] SLOs defined for critical user journeys with burn-rate alerts

**Enrichment**
- [ ] Detector event templates carry team / zone / service properties
- [ ] Entity tags and Smartscape ownership populated for OOTB problems
- [ ] `event.severity` used to drive priority

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

<a id="audit"></a>
## 3. The Ongoing Audit Loop

The setup checklist above is build-time. Noise creeps back as services, traffic, and ownership change, so a standing audit is what keeps the funnel honest. The point is not to *count* false positives — it is to find the repeating configuration patterns behind them, so the same noise is not regenerated next period.

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

**Cadence:**

| Stage | When | Focus |
|-------|------|-------|
| **Pre-go-live** | Before an area or service reaches production | Every new or migrated detector fires correctly in non-prod; nothing fires on no-data |
| **Early** (first ~3 months) | Monthly | Top-noise alerts reviewed with the owning team |
| **Steady state** | Quarterly | Cross-area review; retire detectors silent 90+ days or at a 100% silence rate |

Express these as recurring queries over the problem feed (`fetch events | filter event.kind == "DAVIS_PROBLEM"`); AIOPS-03 and ORGNZ-10 carry validated problem-feed query patterns to build on.

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
