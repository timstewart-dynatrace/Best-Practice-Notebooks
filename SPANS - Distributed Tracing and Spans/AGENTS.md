# AGENTS.md — SPANS: Distributed Tracing and Spans

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

9 notebooks on working with distributed traces and spans in Dynatrace:
tracing fundamentals, span DQL querying, trace-based troubleshooting, service
dependency mapping, time-series analytics, span security, Grail buckets and
OpenPipeline for spans, query cost optimization, and a best-practice summary.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| What spans/traces are, span anatomy, span kinds (server/client/producer/consumer), trace structure, first `fetch spans` query | `-[SPANS]-01-fundamentals.md` |
| Filtering spans by service/kind/operation, string matching, finding a trace by `trace.id`, HTTP and database span queries, NULL handling, DQL-is-not-SQL pitfalls | `-[SPANS]-02-querying.md` |
| Root cause analysis: finding error spans, error patterns, latency analysis, slow requests, reconstructing full traces, downstream dependency and DB query troubleshooting | `-[SPANS]-03-troubleshooting.md` |
| Service topology from spans: discovering services, mapping dependencies, client-server call patterns, async messaging flows, critical path analysis, `dt.service.name` / OTel `service.name` enrichment | `-[SPANS]-04-topology.md` |
| `makeTimeseries` on spans, trend detection, complex aggregations, comparison queries, dashboard-ready span queries | `-[SPANS]-05-analytics.md` |
| Sensitive data in spans (URLs, headers, DB queries), PII audits, auth-failure detection, anomalous traffic, masking spans via OpenPipeline, compliance | `-[SPANS]-06-security.md` |
| Grail buckets for spans, querying specific buckets, OpenPipeline span filtering/enrichment/routing, sampling strategies, access control, OneAgent attribute enrichment (primary fields/tags) | `-[SPANS]-07-buckets-pipeline.md` |
| Query cost (DPS/DDU), filter-early pattern, indexed fields, field selection, time-range optimization, high-cardinality grouping, performance checklist | `-[SPANS]-08-cost-optimization.md` |
| Consolidated checklist: span DQL syntax rules, indexed-field filtering, trace-analysis patterns, OpenPipeline span config, bucket/sampling settings | `-[SPANS]-99-best-practice-summary.md` |

If more than three rows match, start with `-[SPANS]-99-best-practice-summary.md`
and follow its pointers.

## Related series

- Processing spans at ingestion — filtering, enrichment, sampling-aware metrics (before they land in Grail): `../OPIPE - OpenPipeline Beyond Logs/`
- OpenTelemetry instrumentation, collectors, and OTLP trace ingest: `../OTEL - OpenTelemetry Integration/`
- Tenant-wide query and ingest cost analysis beyond span queries: `../FINOPS - Cost Management & FinOps/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "SPANS-03") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
