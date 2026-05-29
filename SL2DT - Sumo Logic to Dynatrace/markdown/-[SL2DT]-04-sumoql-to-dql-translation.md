# SL2DT-04: SumoQL → DQL Translation

> **Series:** SL2DT — Sumo Logic to Dynatrace | **Notebook:** 4 of 10 | **Created:** April 2026 | **Last Updated:** 05/15/2026

## Overview

**Goal of this step:** translate the SumoQL query surface (dashboard panels, scheduled searches, saved searches, monitor conditions) into DQL. Use the `sumoql-to-dql` skill as the canonical mapping reference. Track confidence per query; flag LOW-confidence queries for manual review before they ship.

This is the largest engineering effort in the migration — a typical customer has thousands of queries. Get the automation pipeline right before starting; batch-translate; review in rank order of confidence.

---

## Table of Contents

1. [What You'll Produce](#outputs)
2. [The Translation Engine — Overview](#engine)
3. [Critical Translation Rules](#rules)
4. [Running Batch Translation](#batch)
5. [Confidence Distribution Analysis](#confidence)
6. [Handling LOW-Confidence Queries](#low)
7. [Validating DQL Against Live Tenant](#validate)
8. [Worked Examples — 5 Common Patterns](#examples)
9. [Step Exit Criteria](#gate)

---

## Prerequisites

| Requirement | Details |
|-------------|---------|
| **Audience** | Core migration engineers + app-team reviewers |
| **Skill reference** | `sumoql-to-dql` SKILL.md + mapping-tables.md + examples.md |
| **Inputs** | Inventory from SL2DT-02 (dashboards.json, monitors.json, searches.json, fers.json) |
| **Dynatrace access** | Platform Token with `storage:logs:read` + `storage:events:read` + `storage:metrics:read` for validation |
| **Tools** | Python 3 for batch translation scripts; optionally the `Dynatrace-SumoLogic` migration tool when built (see `docs/AGENT-TASKS.md`) |

<a id="outputs"></a>
## 1. What You'll Produce

| Artifact | Purpose |
|----------|---------|
| `translations/all-queries.csv` | Every query — source SumoQL + translated DQL + confidence + notes |
| `translations/high-confidence.csv` | Auto-approved (confidence ≥80) |
| `translations/medium-confidence.csv` | Requires sample validation |
| `translations/low-confidence.csv` | Manual rewrite required |
| `translation-report.md` | Aggregate metrics + escalations |

Commit to migration repo. These gate the downstream monitor and dashboard rebuild work.

<a id="engine"></a>
## 2. The Translation Engine — Overview

The `sumoql-to-dql` skill is the engine's specification. It defines:

- **Pipeline mapping** — `| parse` / `| where` / `| count by` → DQL equivalents
- **Function mapping** — `pct` → `percentile`, `count_distinct` → `countDistinctExact`, `case` → nested `if`
- **Metadata mapping** — `_sourceCategory` → `dt.source_entity` (per your SL2DT-03 mapping)
- **Confidence scoring** — deductions for parse syntax conversion, transpose, outlier, etc.

The engine is consumed either:

1. **Manually** — human reads the skill, translates one query at a time. Use for LOW-confidence + complex queries.
2. **Programmatically** — via the `Dynatrace-SumoLogic` migration tool (see `docs/AGENT-TASKS.md`) when available. This tool batches the translation using the skill as its data source.

Until the tool exists, manual translation with skill reference is the path. The `translate` subcommand equivalent can also be implemented as a short Python script using the mapping tables — see SL2DT-08 for an automation sketch.

<a id="rules"></a>
## 3. Critical Translation Rules

Four rules must be applied to every translation — the validator and Dynatrace execution engine will reject queries that violate them.

### Rule 1 — Explicit Time Range on Every `fetch` / `timeseries`

Sumo inherits the time range from the dashboard or panel. DQL does not. Every `fetch logs`, `fetch spans`, `fetch events`, `fetch bizevents`, or `timeseries` must carry a `from:` value.

### Rule 2 — Alias Every Aggregation Referenced Downstream

Sumo gives you `_count` automatically. DQL requires explicit aliasing before sort or filter can reference the aggregation.

### Rule 3 — Reduce Arrays Before Filter/Sort on `timeseries` Output

`timeseries` returns arrays. You cannot `filter` or `sort` on the array directly — reduce with `arrayAvg`, `arraySum`, `arrayMax`, etc.

### Rule 4 — Parse Uses DPL Matchers, Not Sumo Wildcards

Sumo's `parse "user=* status=*"` uses `*` wildcards. DPL uses named matchers: `LD 'user=' DATA:user ' status=' DATA:status`.

### Example — All Four Rules Applied

```dql
// Translation of Sumo:
//   _sourceCategory=prod/api | parse "status=*" as status
//   | timeslice 1m | count by status, _timeslice
//   | where status >= 500 | sort by _count desc
fetch logs, from:-1h                                               // Rule 1: explicit from
| filter dt.source_entity == "prod/api"
| parse content, "LD 'status=' INT:status"                         // Rule 4: DPL matcher
| makeTimeseries c = count(), interval:1m, by:{status}             // Rule 2: aliased to c
| fieldsAdd c_max = arrayMax(c)                                    // Rule 3: reduce array
| filter status >= 500
| sort c_max desc

```

<a id="batch"></a>
## 4. Running Batch Translation

Pull the query surface from `inventory/` JSON files into a flat CSV for translation.

### Extract all SumoQL queries

```bash
# Dashboards
jq -r '[.panels[]? | {
  asset_id: .id,
  asset_type: "dashboard_panel",
  dashboard_id: .dashboardId,
  query: .query
}]' inventory/dashboards/*.json > queries-dashboards.json

# Monitors
jq -r '[.monitors[]? | {
  asset_id: .id,
  asset_type: "monitor",
  query: .queries[].query,
  threshold: .triggers[0].threshold
}]' inventory/monitors.json > queries-monitors.json

# Searches + scheduled searches
jq -r '...' inventory/searches.json > queries-searches.json

# Merge into one
jq -s 'add' queries-*.json > translations/input.json
```

### Run the translation pass

When the `Dynatrace-SumoLogic` migration tool exists:

```bash
migrate.py translate translations/input.json \
  --output translations/all-queries.csv \
  --confidence-threshold 80 \
  --sourcecategory-map taxonomy-map.md
```

Until then, iterate manually with the skill's mapping tables. For a typical 500–2000 query batch, this is 2–5 days of engineer time for core-team examples; app teams do the rest.

### CSV Output Schema

| Column | Description |
|--------|-------------|
| `asset_id` | Sumo asset ID (for round-trip correlation) |
| `asset_type` | dashboard_panel / monitor / search / fer |
| `sumo_query` | Original SumoQL |
| `dql_query` | Translated DQL |
| `confidence` | HIGH / MEDIUM / LOW |
| `confidence_score` | 0–100 |
| `deductions` | Reasons for score reduction (semicolon-separated) |
| `warnings` | Translator warnings (e.g., "transpose not supported") |
| `needs_human_review` | true/false |

<a id="confidence"></a>
## 5. Confidence Distribution Analysis

For a typical Sumo deployment, expect this distribution:

| Confidence | Typical share | Action |
|-----------|--------------|--------|
| HIGH (80–100) | 70–85% | Auto-use; sample-validate during dual-run |
| MEDIUM (50–79) | 15–25% | 100% review before shipping |
| LOW (<50) | 2–10% | Manual rewrite; flag for Dynatrace Intelligence redirect where applicable |

### Red flags

- **HIGH > 95%** — translator may be over-confident; do deeper sampling in validation
- **LOW > 15%** — engagement cost underestimated; re-plan with stakeholders
- **MEDIUM > 40%** — translator may need tuning; report recurring patterns back to the `sumoql-to-dql` skill maintainer

### Breakdown by pattern

Track which translation issues cause most deductions:

| Pattern | Impact | Example |
|---------|--------|---------|
| Parse syntax conversion | -10 | `parse "user=*"` → DPL `DATA:user` |
| Case → nested if | -15 | `case(...)` → `if(..., else:if(..., else:...))` |
| Transpose required | -20 | `transpose row x column y` |
| `outlier` / `predict` / `logreduce` | -30 | Redirect to Dynatrace Intelligence |
| Nested aggregation | -20 | `sum(avg(x))` — rejected |
| Unknown `_sourceCategory` | -10 | Taxonomy map incomplete |

This table feeds the translator improvement backlog.

<a id="low"></a>
## 6. Handling LOW-Confidence Queries

LOW-confidence queries fall into three buckets. Handle differently:

### 6.1 Redirect to Dynatrace Intelligence

Outlier / anomaly detection patterns should become anomaly detectors, not DQL queries.

**Sumo:**
```
_sourceCategory=prod/api | timeslice 1m | count | outlier _count window=10 threshold=3
```

**Do:** Configure a anomaly detector on a metric derived from the same log source. See SL2DT-05.

**Don't:** Try to recreate outlier math in DQL. The baseline is brittle and doesn't adapt to seasonality.

### 6.2 Manual Rewrite

Patterns that DQL supports but the translator couldn't match (transpose, complex join, custom functions).

Example — transpose:

```
_sourceCategory=prod/api | count by status, host
| transpose row host column status
```

No direct DQL. Use:

```dql
// Alternative: pivot via summarize + per-status aliases
fetch logs, from:-1h
| filter dt.source_entity == "prod/api"
| parse content, "LD 'status=' INT:status"
| summarize
    count_200 = countIf(status == 200),
    count_400 = countIf(status == 400),
    count_500 = countIf(status >= 500),
    by:{host.name}
| sort count_500 desc

```

### 6.3 Flag for Retirement

Some LOW-confidence queries serve no business purpose (old debugging queries, abandoned experiments). Review with the owner — they often say "yeah, cut it."

Goal: minimize the hand-rewrite load. For a 500-query batch, after Dynatrace Intelligence redirects and retirement, actual hand-rewrites should be under 30.

<a id="validate"></a>
## 7. Validating DQL Against Live Tenant

Every translated query must execute successfully against a live Dynatrace tenant before being added to a notebook, monitor, or dashboard.

### Validation via MCP

```
mcp__dynatrace__verify-dql  — syntax check
mcp__dynatrace__execute-dql — runs query; confirms results
```

### Validation via API

```bash
curl -X POST "$DT_TENANT/platform/storage/query/v1/query:execute" \
  -H "Authorization: Bearer $DT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"query": "fetch logs, from:-5m | limit 10"}'
```

### Sample Size

For a 500-query batch:

- HIGH confidence: validate 10% random sample
- MEDIUM confidence: validate 100%
- LOW confidence: validate 100% after rewrite

### Failure patterns

| Failure | Fix |
|---------|-----|
| `No such bucket` | Bucket typo or not yet created (SL2DT-03) |
| `Unknown field` | Field name wasn't renamed correctly (check `dt.source_entity` mapping) |
| `Invalid syntax` | Translator bug — report to skill maintainer, fix by hand |
| `Query timeout` | Query scans too much data — add `from:` narrower range or `samplingRatio:` |
| `Empty results` | Data isn't yet flowing for that scope (SL2DT-03 parity issue) |

<a id="examples"></a>
## 8. Worked Examples — 5 Common Patterns

Each example shows the source Sumo, target DQL, and confidence. These are representative; the skill's `references/examples.md` has 15 more.

### Example 1 — Top errors by host (HIGH 100%)

Sumo:
```
_sourceCategory=prod/api ERROR
| count by _sourceHost
| sort by _count desc
| limit 10
```

DQL:

```dql
// 8. Worked Examples — 5 Common Patterns
fetch logs, from:-1h
| filter dt.source_entity == "prod/api"
| filter contains(content, "ERROR")
| summarize c = count(), by:{host.name}
| sort c desc
| limit 10

```

### Example 2 — Latency percentiles by endpoint (HIGH 95%)

Sumo:
```
_sourceCategory=prod/api | parse regex "latency=(?<lat>\d+)"
| pct(lat, 50), pct(lat, 95), pct(lat, 99) by path
```

DQL:

```dql
// Example 2 — Latency percentiles by endpoint (HIGH 95%)
fetch logs, from:-1h
| filter dt.source_entity == "prod/api"
| parse content, "LD 'latency=' INT:latency"
| summarize p50 = percentile(latency, 50),
            p95 = percentile(latency, 95),
            p99 = percentile(latency, 99),
            by:{http.path}

```

### Example 3 — Error rate timeseries (MEDIUM 80%)

Sumo:
```
_sourceCategory=prod/api | timeslice 1m
| if(matches(_raw, "*ERROR*"), 1, 0) as is_error
| sum(is_error) as errors, count as total by _timeslice
| errors / total * 100 as error_pct
```

DQL:

```dql
// Example 3 — Error rate timeseries (MEDIUM 80%)
fetch logs, from:-1h
| filter dt.source_entity == "prod/api"
| fieldsAdd is_error = if(contains(content, "ERROR"), 1, else:0)
| makeTimeseries errors = sum(is_error), total = count(), interval:1m
| fieldsAdd error_pct = (100.0 * toDouble(arraySum(errors)) / toDouble(arraySum(total)))

```

### Example 4 — Lookup enrichment (MEDIUM 75%)

Sumo:
```
_sourceCategory=prod | count by _sourceHost
| lookup environment, owner from path://lookups/host_metadata on _sourceHost=hostname
```

DQL (requires entity attributes or a Grail lookup table):

```dql
// Example 4 — Lookup enrichment (MEDIUM 75%)
fetch logs, from:-1h
| filter startsWith(dt.source_entity, "prod/")
| summarize c = count(), by:{host.name}
| lookup [fetch dt.entity.host
         | fieldsAdd hostname = entity.name
         | fields hostname, environment = `dt.tags.environment`, owner = `dt.tags.team`],
    sourceField:host.name, lookupField:hostname,
    fields:{environment, owner}

```

### Example 5 — Outlier (LOW — Dynatrace Intelligence redirect)

Sumo:
```
_sourceCategory=prod/api | timeslice 1m | count | outlier _count window=10 threshold=3
```

**Action:** configure anomaly detector. DQL baseline is a fallback only if Dynatrace Intelligence can't be used (rare):

```dql
// LOW CONFIDENCE: prefer anomaly detector.
// Fallback baseline calc (for reference, not production):
fetch logs, from:-1h
| filter dt.source_entity == "prod/api"
| makeTimeseries c = count(), interval:1m
| fieldsAdd mean = arrayAvg(c), sd = arrayStdDev(c)
| fieldsAdd upper = mean + 3 * sd

```

<a id="gate"></a>
## 9. Step Exit Criteria

**G4 — Translation Complete**

- [ ] Every query from `inventory/` in `translations/all-queries.csv`
- [ ] HIGH-confidence queries: ≥75% of total
- [ ] MEDIUM-confidence queries: 100% reviewed
- [ ] LOW-confidence queries: classified (Dynatrace Intelligence / rewrite / retire)
- [ ] 10% random sample of HIGH queries validated against live tenant
- [ ] 100% of MEDIUM queries validated against live tenant
- [ ] Translation report delivered to stakeholders

**Next step:** **SL2DT-05 — Monitor/Alert Conversion** (apply translations + anomaly detection redirects to Monitor-class assets).

---

<a id="references"></a>
## 11. References

### Dynatrace query and pattern languages
- [DQL Reference (DT docs)](https://docs.dynatrace.com/docs/discover-dynatrace/references/dynatrace-query-language)
- [DPL Pattern Reference (DT docs)](https://docs.dynatrace.com/docs/discover-dynatrace/references/dynatrace-pattern-language)
- [Grail (DT docs)](https://docs.dynatrace.com/docs/discover-dynatrace/platform/grail)

### Sumo Logic source-language reference
- [Sumo Logic search operators (Sumo Logic docs)](https://help.sumologic.com/docs/search/search-query-language/search-operators/)
- [Sumo Logic parse operators (Sumo Logic docs)](https://help.sumologic.com/docs/search/search-query-language/parse-operators/)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace or Sumo Logic. Always verify information against the official [Dynatrace documentation](https://docs.dynatrace.com/docs) and [Sumo Logic documentation](https://help.sumologic.com/docs/).*</sub>
