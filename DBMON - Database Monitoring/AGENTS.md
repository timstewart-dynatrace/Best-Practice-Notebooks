# AGENTS.md — DBMON: Database Monitoring

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

7 notebooks on monitoring databases with Dynatrace: span-based query analysis
for SQL and NoSQL platforms, cache and message broker monitoring, ActiveGate
extensions for server-side metrics, query anti-pattern detection, database
dashboards/alerting, and the Databases app's Health Score and remediation
suggestions as an analysis layer over both data sources.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| How database monitoring works: OneAgent span capture vs ActiveGate extensions, database span anatomy (`db.*` attributes), entity model, discovering database services; the dedicated Databases app as a third analysis layer over both data sources — five analysis areas (configuration, schema, query, execution plan, and a rolled-up 0–100 **Health Score**) plus AI-generated remediation suggestions (GA August 2026 for PostgreSQL and MySQL, more technologies planned) | `-[DBMON]-01-database-monitoring-fundamentals.md` |
| PostgreSQL, MySQL/MariaDB, SQL Server, or Oracle: slow query detection, operation breakdown by table, connection monitoring, SQL Server ActiveGate extension (`sql-server.*` metrics, Always On, job logs) | `-[DBMON]-02-sql-databases.md` |
| MongoDB, DynamoDB, Cassandra, or Cosmos DB: operation performance, read/write ratios, Request Units, cross-database comparison | `-[DBMON]-03-nosql-databases.md` |
| Redis, Memcached, Kafka, RabbitMQ, or Elasticsearch: cache hit/eviction, consumer/producer throughput, queue health, search query latency | `-[DBMON]-04-cache-and-messaging.md` |
| Query-level diagnostics: N+1 anti-pattern detection, frequency/duration distributions, missing-index signals, connection pool exhaustion, query normalization | `-[DBMON]-05-query-analysis.md` |
| Dashboards and alerts: database KPIs (P95 response time, error rate, throughput), slow-query and engine-internals alerting, database SLO definitions; using the Databases app **Health Score** as a first-pass triage signal alongside — not instead of — the span-based tiles, thresholds and SLOs the app's UI doesn't expose | `-[DBMON]-06-dashboards-and-alerting.md` |
| Consolidated checklist: instrumentation coverage, per-technology thresholds, alerting values, SLO targets, DQL standards, Databases app triage practices (rows 13b/13c — Health Score first-pass, execution-plan visualization as a first pass not a replacement) | `-[DBMON]-99-best-practice-summary.md` |

If more than three rows match, start with `-[DBMON]-99-best-practice-summary.md`
and follow its pointers.

## Related series

- Span/trace mechanics underneath every database query here: `../SPANS - Distributed Tracing and Spans/`
- The OTel `db.*` semantic conventions these spans follow: `../OTEL - OpenTelemetry Integration/`
- Managed cloud databases (RDS, Cosmos DB, Cloud SQL) via provider integrations: `../CLOUD - Cloud Provider Integrations/`
- Turning the alert-ready queries into detectors and notifications: `../ALERT - Alerting Strategy and Design/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "DBMON-05") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
