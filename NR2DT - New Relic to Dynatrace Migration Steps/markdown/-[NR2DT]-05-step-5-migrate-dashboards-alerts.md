# NR2DT-05: Step 5 — Migrate Dashboards & Alerts

> **Series:** NR2DT — New Relic to Dynatrace Migration Steps | **Notebook:** 5 of 10 | **Created:** April 2026 | **Last Updated:** 08/27/2026

## Overview

**Goal of this step:** import dashboards (Wave 1, low risk) and alerts (Wave 3, HIGH risk). Dashboards land first as read-only signal; alerts go through a 1–2 week dual-alert window before NR alerts are silenced.

Procedural — see **NRLC-03** (Dashboard Migration) and **NRLC-04** (Alert & Workflow Migration) for component depth.

---

## Table of Contents

1. [Wave 0 — Apply Foundations](#wave0)
2. [Wave 1 — Dashboards](#wave1)
3. [Wave 3 — Alerts (Dual-Alert)](#wave3)
4. [Step Exit Criteria](#gate)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Audience** | Migration lead + assigned engineer for this step |
| **Completed** | NR2DT-04 — Translate |
| **Format** | Procedural step — use as a runbook; defer to NRLC for depth |
| **NRLC deep dives** | NRLC-03, NRLC-04 |

<a id="wave0"></a>
## 1. Wave 0 — Apply Foundations

Phase-1 Terraform from Step 3 lands first.

```bash
cd terraform
terraform apply -target=module.buckets -target=module.host_groups \
    -target=module.iam_groups -target=module.openpipeline_enrichments
```

**Wait ~45 minutes** for OpenPipeline enrichment propagation on EKS clusters before Wave 2 Terraform applies any routing rules.

**Validate G0:**

```
fetch logs, from:-15m
| summarize records = count(), by:{dt.system.bucket}
```

Every active bucket should show ingest. If anything lands in `default_logs` for a routed source, Wave 0 isn't done.

<a id="wave1"></a>
## 2. Wave 1 — Dashboards

Dashboards are low-risk because they're read-only — users can compare visualizations side-by-side during the parallel run.

```bash
# Diff first to avoid duplicates
python3 migrate.py migrate --diff --components dashboards

# Import
python3 migrate.py migrate --import-only --components dashboards
```

**G1 — Dashboard parity:**

Sample 10–20% of dashboards. Open each NR dashboard and the corresponding DT document side-by-side. Acceptable diff:

- Visual layout: minor variations OK
- Data values: ±5% on count widgets
- Time-series shape: matches

Document any dashboard with > 5% delta in `wave1-issues.md` for follow-up.

<a id="wave3"></a>
## 3. Wave 3 — Alerts (Dual-Alert)

**HIGH RISK.** Follow this exactly.

### Step 1: Import to silent test channels

```bash
python3 migrate.py migrate --import-only --components alerts,notifications
```

Routing in this step goes to a **silent test Slack channel** (or equivalent). Update Workflow tasks to point at the test channel before enabling.

### Step 2: Run dual-alert for 1–2 weeks

Both NR and DT raise alerts. Compare volume daily:

```
fetch events, from:-1d
| filter event.kind == "DAVIS_PROBLEM"
| summarize problems = count(), by:{event.name, event.category}
```

DT problem count should match NR alert count within ±10% per condition. Investigate any condition with > 25% delta.

### Step 3: Promote per policy

**One policy at a time.** When a policy's dual-alert volume aligns:

1. Update its Workflow tasks to point at the production channel (re-enter secrets in DT credentials vault)
2. Silence that policy in NR (set policy state to inactive)
3. Move to the next policy

Don't batch — sequential cutover lets you reverse one policy without affecting others.


![Dual-Alert Window Pattern](images/05-dual-alert-window_930x500.png)

<!-- MARKDOWN_TABLE_ALTERNATIVE
| Phase | Action |
|-------|--------|
| 1 | Import to silent test channel (Day 0) |
| 2 | Dual-alert for 1–2 weeks; compare volume daily |
| 3 | Promote per policy (one at a time); silence NR per policy |

Pass criteria: DT vs NR alert count within ±10% per condition.
For environments where SVG doesn't render
-->

<a id="gate"></a>
## 4. Step Exit Criteria

**G5 — Dashboards + Alerts Migrated**

- [ ] All Wave 0 Foundations applied; G0 validated
- [ ] All dashboards migrated; G1 visual-diff sample passed
- [ ] All alert policies migrated; dual-alert window completed (≥ 1 week)
- [ ] G3 dual-alert volume aligned within ±10%
- [ ] All notification secrets re-entered in DT credentials vault
- [ ] NR alert policies silenced for migrated-policies-only (NR ingest still on)

**Next step:** **NR2DT-06 — Migrate Synthetics, SLOs, Workloads**.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources, including the open-source [NewRelic-to-Dynatrace-Migration-Utilities](https://github.com/timstewart-dynatrace/NewRelic-to-Dynatrace-Migration-Utilities), [nrql-engine](https://github.com/timstewart-dynatrace/nrql-engine), and [nrql-translator](https://github.com/timstewart-dynatrace/nrql-translator) projects. This notebook series is not officially supported by Dynatrace or New Relic. Always verify information against the official [Dynatrace documentation](https://docs.dynatrace.com/docs) and [New Relic documentation](https://docs.newrelic.com).*</sub>
