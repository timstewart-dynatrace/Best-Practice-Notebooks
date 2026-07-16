# AGENTS.md — OTEL: OpenTelemetry Integration

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

9 notebooks on integrating OpenTelemetry with Dynatrace: Collector
architecture and deployment patterns, trace/metric/log instrumentation, OTLP
endpoint and authentication setup, OneAgent coexistence, and pipeline
troubleshooting.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| OTel concepts: signals, semantic conventions, OTel vs OneAgent trade-offs, when to use which, OpenTelemetry licensing/DPS | `-[OTEL]-01-fundamentals.md` |
| Collector internals: receivers/processors/exporters/connectors, `batch` + `memory_limiter` config, collector-side masking (`redaction`/`transform`), distributions | `-[OTEL]-02-collector-architecture.md` |
| Running collectors: agent (DaemonSet/sidecar) vs gateway mode, Kubernetes/Helm/Operator CRD deployment, tiered Prometheus scraping with the Target Allocator, HA, sizing, security | `-[OTEL]-03-collector-deployment.md` |
| Trace instrumentation: zero-code vs manual, context propagation, span attributes/events, head vs tail sampling (`tail_sampling`), Istio/Envoy | `-[OTEL]-04-trace-instrumentation.md` |
| Metric instrumentation: counter/histogram/gauge selection, views and aggregation, histogram bucket boundaries, Prometheus scrape-to-OTLP bridging | `-[OTEL]-05-metrics-instrumentation.md` |
| Log instrumentation: log bridge pattern, trace-log correlation (`trace_id`/`span_id`), `file_log` receiver, structured logging | `-[OTEL]-06-logs-instrumentation.md` |
| Sending data to Dynatrace: OTLP endpoints and tokens, exporter config, `service.name`/`dt.service.name` entity enrichment, metric dimensions, OneAgent + DynaKube coexistence | `-[OTEL]-07-dynatrace-integration.md` |
| Data not arriving or wrong: SDK/collector/connectivity diagnostics, data quality and performance issues, `otelcol_*` self-metrics, common fixes | `-[OTEL]-08-troubleshooting.md` |
| Consolidated checklist: collector config values, deployment sizing, OTLP integration settings, semantic-convention rules | `-[OTEL]-99-best-practice-summary.md` |

If more than three rows match, start with `-[OTEL]-99-best-practice-summary.md`
and follow its pointers.

## Related series

- OTel collectors alongside OneAgent/DynaKube in Kubernetes: `../K8S - Kubernetes Monitoring/`
- Analyzing the ingested traces and spans with DQL: `../SPANS - Distributed Tracing and Spans/`
- Processing OTLP spans/metrics/events at ingest: `../OPIPE - OpenPipeline Beyond Logs/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "OTEL-07") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
