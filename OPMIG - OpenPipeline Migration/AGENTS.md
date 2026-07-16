# AGENTS.md — OPMIG: OpenPipeline Migration

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

10 notebooks on migrating from classic log processing to OpenPipeline:
why-migrate and readiness assessment, architecture and DPL, wave planning,
pipeline configuration, routing and bucket strategy, parsing (incl.
Grok→DPL), metric/event extraction, masking and compliance, troubleshooting,
and a best-practice summary.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| Why migrate off classic log processing, classic-vs-OpenPipeline differences, API endpoint compatibility, readiness assessment, headline limits | `-[OPMIG]-01-introduction-why-migrate.md` |
| Data-flow architecture, processing-stage execution order, pipeline/processor types, DPL fundamentals, the complete limits reference | `-[OPMIG]-02-architecture-key-concepts.md` |
| Discovering the current log landscape, volume analysis, source inventory, priority scoring matrix, defining migration waves, sensitive-data assessment | `-[OPMIG]-03-migration-assessment-planning.md` |
| Creating pipelines step-by-step (UI and API), configuring processors, dynamic routing setup, testing with sample data | `-[OPMIG]-04-pipeline-configuration.md` |
| Routing strategies, Grail bucket design, multi-tier bucket architectures, cost-optimization ROI, bucket governance and access control | `-[OPMIG]-05-routing-buckets.md` |
| Parsing and transformation: DPL patterns, Apache/Nginx/JSON log formats, Grok→DPL (ELK/Logstash) conversion, drop processors, DPL Architect | `-[OPMIG]-06-processing-parsing.md` |
| Extracting metrics and events from logs: RED SLI metrics, business events and KPIs, matching conditions, cardinality-aware metric design | `-[OPMIG]-07-metric-event-extraction.md` |
| Masking and PII protection, built-in vs custom `replacePattern` masking, GDPR/HIPAA/PCI-DSS/SOC 2 checklists, field removal | `-[OPMIG]-08-security-masking.md` |
| Diagnosing pipeline issues via decision trees, pipeline health monitoring, parsing/volume validation, emergency rollback, performance tuning | `-[OPMIG]-09-troubleshooting-validation.md` |
| Consolidated best-practice settings tables and the DQL cookbook for the whole migration | `-[OPMIG]-99-best-practice-summary.md` |

If more than three rows match, start with
`-[OPMIG]-99-best-practice-summary.md` and follow its pointers.

## Related series

- Ongoing OpenPipeline **log** work after (or independent of) the migration — ingest, parsing, routing as day-2 operations: `../OPLOGS - OpenPipeline Logs/`
- OpenPipeline for **non-log signal types** (spans, metrics, business/security events): `../OPIPE - OpenPipeline Beyond Logs/`
- Grail bucket, segment, and security-context strategy behind the routing decisions: `../ORGNZ - Organize Data: Buckets, Segments, Security/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "OPMIG-06") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
