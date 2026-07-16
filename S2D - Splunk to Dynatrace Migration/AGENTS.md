# AGENTS.md — S2D: Splunk to Dynatrace Migration

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

10 notebooks on migrating monitoring from Splunk to Dynatrace: platform
differences and planning, verifying log availability, SPL→DQL translation,
alert migration to Davis anomaly detectors and scheduled workflows, extended
timeframes via `arrayMovingSum`, log-to-metric extraction with OpenPipeline,
dashboard conversion, and naming standards, plus a best-practice summary.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| Splunk-vs-Dynatrace platform differences (indexes vs Grail buckets), migration planning, the 5-phase roadmap, first log-source verification | `-[S2D]-01-getting-started.md` |
| Confirming Splunk index data exists in Dynatrace: scoping, identifying required log sources, DQL verification queries, log-ingest configuration, resolving missing-data discrepancies | `-[S2D]-02-locating-logs.md` |
| Translating SPL queries to DQL: `search`/`stats`/`eval`/`table` → `fetch`/`summarize`/`fieldsAdd`/`fields`, filtering, field extraction, time-series patterns, syntax differences | `-[S2D]-03-spl-to-dql.md` |
| Migrating scheduled Splunk alerts to Davis anomaly detectors: continuous vs scheduled evaluation, the sliding-window concept, threshold translation formula, decision tree | `-[S2D]-04-anomaly-detectors.md` |
| Scheduled workflow-based alerts when detectors don't fit: workflow structure, event-creation JavaScript, cron schedules, drawbacks and license cost | `-[S2D]-05-workflow-alerts.md` |
| Alerting or reporting over windows longer than the 60-minute detector limit: `arrayMovingSum` rolling aggregations and other array functions | `-[S2D]-06-arraymovingsum.md` |
| Extracting metrics from logs via OpenPipeline: when extraction pays off, counter vs value metric types, dimension design, the request checklist | `-[S2D]-07-metric-creation.md` |
| Converting Splunk dashboards: panel/visualization type mapping, query translation examples, variables for filtering, the log-searcher dashboard pattern | `-[S2D]-08-dashboard-migration.md` |
| Naming conventions for migrated dashboards, alerts, reports, lookup tables, metrics, and workflows | `-[S2D]-09-naming-standards.md` |
| Consolidated best practices for every migration phase, including DQL performance and syntax rules | `-[S2D]-99-best-practice-summary.md` |

If more than three rows match, start with `-[S2D]-99-best-practice-summary.md`
and follow its pointers.

## Related series

- Log parsing, routing, and processing depth on the Dynatrace side: `../OPLOGS - OpenPipeline Logs/`
- Dashboard design for stakeholder audiences: `../DASH - Dashboard Design & Building/`
- Anomaly-detection mechanisms and Davis AI depth: `../AIOPS - Dynatrace Intelligence/`
- Workflow building and notification routing: `../WFLOW - Workflows and Alert Notifications/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "S2D-03") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
