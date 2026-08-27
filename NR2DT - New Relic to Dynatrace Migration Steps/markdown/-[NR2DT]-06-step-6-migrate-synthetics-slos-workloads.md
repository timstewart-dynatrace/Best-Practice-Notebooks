# NR2DT-06: Step 6 — Migrate Synthetics, SLOs & Workloads

> **Series:** NR2DT — New Relic to Dynatrace Migration Steps | **Notebook:** 6 of 10 | **Created:** April 2026 | **Last Updated:** 08/27/2026

## Overview

**Goal of this step:** complete Wave 2 (synthetics) and Wave 4 (SLOs + workloads). Synthetics are external-facing and visible quickly; SLOs need a 7-day delta-validation window before the math is trusted.

Procedural — see **NRLC-05** (Synthetic Migration) and **NRLC-06** (SLO & Workload Migration) for component depth.

---

## Table of Contents

1. [Wave 2 — Synthetics](#wave2)
2. [Wave 4 — SLOs & Workloads](#wave4)
3. [Step Exit Criteria](#gate)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Audience** | Migration lead + assigned engineer for this step |
| **Completed** | NR2DT-05 — Migrate Dashboards & Alerts |
| **Format** | Procedural step — use as a runbook; defer to NRLC for depth |
| **NRLC deep dives** | NRLC-05, NRLC-06 |

<a id="wave2"></a>
## 1. Wave 2 — Synthetics

Order: HTTP first (lowest risk), then Browser, then API multi-step. Scripted browsers are out of band — they get rebuilt manually.

```bash
# Inventory + transform
python3 migrate.py migrate --diff --components synthetics

# Import (auto-converts HTTP, Browser, API; stubs scripted)
python3 migrate.py migrate --import-only --components synthetics
```

**Re-enter secrets** in the DT credentials vault for any monitor that uses bearer tokens, basic auth, or OAuth.

**Scripted browser monitors:**

1. The transformer emits a stub Browser Monitor with the target URL
2. The original NR JS script is attached as a tag for reference
3. An engineer rebuilds the user journey using DT's clickpath recorder or DSL
4. Validation: confirm the new clickpath catches the same regressions during a 1-week dual-run

**G2 — Synthetics SLA:** dual-run for ≥ 1 week. DT availability % matches NR within ±0.5%. If delta exceeds, investigate location coverage and credential validity first.

<a id="wave4"></a>
## 2. Wave 4 — SLOs & Workloads

SLOs depend on metric expressions being mathematically equivalent. Workloads become OpenPipeline enrichments + IAM bucket conditions (Gen3 pattern).

```bash
python3 migrate.py migrate --diff --components slos,workloads
python3 migrate.py migrate --import-only --components slos,workloads
```

**Run the SLO auditor** (the most important validation in this wave):

```bash
python3 migrate.py audit-slos
```

> The audit window and tolerance are set inside the auditor configuration, not via CLI flags. Target your 7-day / 0.5%-tolerance policy in the project's audit config.

Output per SLO:

```
SLO: Checkout Availability
  NR SLI:  99.94%
  DT SLI:  99.92%
  Delta:    0.02%  PASS (within configured tolerance)
```

Any SLO with delta > 0.5% needs investigation:

1. Check unit conversion (ms vs. s vs. ns)
2. Check entity scope — does the bucket / OpenPipeline enrichment filter capture the same entities as NR's workload?
3. Check time alignment — NR's `SINCE 7 days ago` may bin differently than DT's `from:-7d`

**G4 — SLO delta:** all SLOs report value within tolerance for ≥ 7 days.

<a id="gate"></a>
## 3. Step Exit Criteria

**G6 — Synthetics + SLOs Migrated**

- [ ] All synthetics migrated; G2 dual-run availability ±0.5%
- [ ] Scripted browsers rebuilt and validated
- [ ] All synthetic credentials re-entered in DT vault
- [ ] All SLOs migrated; G4 7-day delta ≤ 0.5%
- [ ] Workload identifiers visible as OpenPipeline-enriched attributes in DQL queries

**Next step:** **NR2DT-07 — Migrate Logs, Tags & Drop Rules**.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources, including the open-source [NewRelic-to-Dynatrace-Migration-Utilities](https://github.com/timstewart-dynatrace/NewRelic-to-Dynatrace-Migration-Utilities), [nrql-engine](https://github.com/timstewart-dynatrace/nrql-engine), and [nrql-translator](https://github.com/timstewart-dynatrace/nrql-translator) projects. This notebook series is not officially supported by Dynatrace or New Relic. Always verify information against the official [Dynatrace documentation](https://docs.dynatrace.com/docs) and [New Relic documentation](https://docs.newrelic.com).*</sub>
