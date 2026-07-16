# AGENTS.md — OPIPE: OpenPipeline Beyond Logs

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

7 notebooks on OpenPipeline's non-log scopes: the six-scope architecture,
span filtering/enrichment/routing at ingestion, sampling-aware metric
extraction, cardinality control, business and security event pipelines,
cross-scope correlation patterns, and a best-practice summary.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| The six OpenPipeline scopes, default-pipeline anti-pattern, processing groups, security context, ingestion-time vs query-time processing, primary fields/tags as routing keys | `-[OPIPE]-01-multi-scope-platform.md` |
| Dropping health-check/noise spans at ingestion, span attribute enrichment, span-to-log/event generation, routing spans to buckets, span pipeline monitoring | `-[OPIPE]-02-span-processing-and-enrichment.md` |
| Extracting accurate RED metrics from sampled traces, why naive `count()` undercounts, sampling-ratio multiplication, standard vs sampling-aware metric comparison | `-[OPIPE]-03-sampling-aware-metrics.md` |
| Dimension explosion, detecting high-cardinality fields, reduction strategies (attribute removal, value normalization, bucketing, hashing), metric dimension guardrails | `-[OPIPE]-04-cardinality-management.md` |
| Business event enrichment/metric extraction (`event.type`, `event.provider`), security event routing, compliance buckets, event-to-metric KPIs | `-[OPIPE]-05-business-and-security-event-pipelines.md` |
| Correlating logs/spans/metrics/events via shared dimensions, cascade processing, unified bucket families, when NOT to use OpenPipeline, production readiness checklist | `-[OPIPE]-06-cross-scope-design-patterns.md` |
| Consolidated settings checklist: pipeline-per-scope rules, processing group limits, span drop filters, cardinality/security-context settings | `-[OPIPE]-99-best-practice-summary.md` |

If more than three rows match, start with `-[OPIPE]-99-best-practice-summary.md`
and follow its pointers.

## Related series

- OpenPipeline for logs — parsing, log buckets, log masking (this series covers the other five scopes): `../OPLOGS - OpenPipeline Logs/`
- Migrating from classic log processing to OpenPipeline (go there for the classic → OpenPipeline conversion path, not new pipeline design): `../OPMIG - OpenPipeline Migration/`
- Querying and analyzing spans *after* ingestion processing: `../SPANS - Distributed Tracing and Spans/`
- Business event analytics — funnels, revenue, KPIs (this series covers only their ingestion pipelines): `../BIZEV - Business Events & Funnel Analysis/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "OPIPE-03") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
