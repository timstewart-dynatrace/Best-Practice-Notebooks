# NR2DT-04: Step 4 — Translate

> **Series:** NR2DT — New Relic to Dynatrace Migration Steps | **Notebook:** 4 of 10 | **Created:** April 2026 | **Last Updated:** 10/05/2026

## Overview

**Goal of this step:** convert every NRQL query in the inventory to DQL. Auto-translate HIGH; review MEDIUM; hand-translate LOW. Resolve every gap before any artifact migrates.

Procedural — see **NRLC-02** for the compiler architecture, the 292 patterns, and confidence theory.

---

## Table of Contents

1. [What You'll Produce](#outputs)
2. [Run the Translator](#translate)
3. [Review MEDIUM Confidence](#medium)
4. [Hand-Translate LOW Confidence](#low)
5. [Validate Translated DQL](#validate)
6. [Step Exit Criteria](#gate)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Audience** | Migration lead + assigned engineer for this step |
| **Completed** | NR2DT-03 — Design |
| **Format** | Procedural step — use as a runbook; defer to NRLC for depth |
| **NRLC deep dives** | NRLC-02 (NRQL→DQL Translation deep dive) |

<a id="outputs"></a>
## 1. What You'll Produce

| Artifact | Format | Purpose |
|----------|--------|---------|
| `translated-queries.csv` | CSV | All queries with final DQL + confidence |
| `manual-translations.md` | Markdown | LOW-confidence hand-translations with rationale |
| `dql-validation-report.md` | Markdown | Tier 1 syntax validation results |
| `translation-coverage.md` | Markdown | % of queries translated; remaining gaps |

<a id="translate"></a>
## 2. Run the Translator

```bash
# Re-run translation against the latest inventory (Step 1 produced inventory/all-nrql.csv)
python3 migrate.py batch --file inventory/all-nrql.csv --output translated-queries.csv
```

Output columns — exactly four: `nrql`, `dql`, `confidence` (HIGH / MEDIUM / LOW, or `FAILED` when the query did not compile, in which case the `dql` cell holds the error), `warnings`. Use `batch` for the CSV: `compile` has no `--report` option (it accepts only `--interactive`, `--file`, `--output` and `--validate`) and writes DQL text, not a CSV.

> <sub>**Sources:** [migrate.py @ 78cbfce (tool repo, GitHub)](https://github.com/timstewart-dynatrace/NewRelic-to-Dynatrace-Migration-Utilities/blob/78cbfcec7cab6564103fbd6c23b915b46599dc56/migrate.py), read 10/05/2026.</sub>

**Sort by confidence ascending** — attack LOW first, then MEDIUM, save HIGH for sample-validate.

<a id="medium"></a>
## 3. Review MEDIUM Confidence

MEDIUM means the translation is structurally correct but has edge cases worth verifying. Common review checklist:

| Pattern | What to verify |
|---------|----------------|
| `COMPARE WITH` | Time alignment correct; append produces the expected shape |
| Subquery via `lookup` | Lookup target exists as queryable Grail data |
| `SLIDE BY` | Sliding window semantics preserved |
| Nested `if()` in `FACET CASES` | Nested DQL emits the right structure |
| Custom event types | Source mapping (FROM → fetch) correct |

**Process:** mark each MEDIUM as APPROVED or NEEDS-WORK in the CSV. NEEDS-WORK becomes a hand-translation.

<a id="low"></a>
## 4. Hand-Translate LOW Confidence

LOW translations contain TODO markers or are missing entirely. Common causes:

| Cause | Action |
|-------|--------|
| `apdex(field, t:N)` | Configure DT Apdex via custom DQL or use Dynatrace Intelligence SLO |
| `funnel(...)` | Reformulate as nested `append`/`lookup` chain |
| `cohort(...)` | No DT equivalent — reformulate the requirement |
| Custom event type with no source mapping | Add ingest mapping via OpenPipeline first |

Document each hand-translation in `manual-translations.md` with the source NRQL, the new DQL, and the rationale.

If a query can't be translated and the requirement can't be reformulated, document it as **deferred** — it joins the gap analysis from Step 1.

<a id="validate"></a>
## 5. Validate Translated DQL

Tier 1 syntax validation — confirm every final DQL query parses against the target tenant.

The tool helps only partly. `compile --file` takes the **NRQL** list (one query per line), not the CSV of translated DQL — fed the CSV, it compiles the header and every DQL row as if they were NRQL:

```bash
python3 migrate.py compile --file inventory/all-nrql.txt --output translated.dql
```

It prints a succeeded / failed count and writes each query as `-- Original: <nrql>` followed by its DQL (or `-- Error:`). It does **not** live-validate in file mode — `--validate` is honoured only for a single query — and there is no auto-fixer on this path. Check one query against the live tenant with:

```bash
python3 migrate.py compile --validate "SELECT count(*) FROM Transaction"
```

For the full set, run each final DQL query (including reviewed-MEDIUM and hand-translated ones from the CSV) in the target tenant — in a notebook, or with `dtctl verify query` — and record pass / fail in `dql-validation-report.md`.

> <sub>**Sources:** [migrate.py @ 78cbfce (tool repo, GitHub)](https://github.com/timstewart-dynatrace/NewRelic-to-Dynatrace-Migration-Utilities/blob/78cbfcec7cab6564103fbd6c23b915b46599dc56/migrate.py), read 10/05/2026.</sub>

**Tier 2 tenant validation** (referenced metrics/entities/buckets exist) runs in Step 8 before each component imports.

<a id="gate"></a>
## 6. Step Exit Criteria

**S4 — Translation Complete**

- [ ] Every query has a final DQL (auto, reviewed-MEDIUM, or hand-translated)
- [ ] All translations pass Tier 1 syntax validation
- [ ] LOW-confidence count is 0 (all triaged)
- [ ] `manual-translations.md` documents every LOW resolution
- [ ] Deferred queries documented with deferred-rationale

**Next step:** **NR2DT-05 — Migrate Dashboards & Alerts**.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources, including the open-source [NewRelic-to-Dynatrace-Migration-Utilities](https://github.com/timstewart-dynatrace/NewRelic-to-Dynatrace-Migration-Utilities), [nrql-engine](https://github.com/timstewart-dynatrace/nrql-engine), and [nrql-translator](https://github.com/timstewart-dynatrace/nrql-translator) projects. This notebook series is not officially supported by Dynatrace or New Relic. Always verify information against the official [Dynatrace documentation](https://docs.dynatrace.com/docs) and [New Relic documentation](https://docs.newrelic.com).*</sub>
