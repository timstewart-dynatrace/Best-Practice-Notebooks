# AGENTS.md — OPLOGS: OpenPipeline Logs

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

9 notebooks on managing logs with Dynatrace OpenPipeline: architecture and
data-source fundamentals, migration from classic log processing, pipeline
processing stages (parse/enrich/mask/route), Grail bucket governance, DQL and
DPL querying, entity topology context, log analytics, security/masking, and a
best-practice summary.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| What OpenPipeline is, architecture, `dt.openpipeline.*` fields, ingestion sources (OneAgent/API/OTLP), classic-vs-OpenPipeline comparison | `-[OPLOGS]-01-fundamentals.md` |
| Assessing log sources/volumes before moving, migration paths, validation queries, migration checklist | `-[OPLOGS]-02-migration.md` |
| Configuring processors: DPL parsing, metric extraction from logs, attribute enrichment, event generation, bucket routing, drop/sampling filters | `-[OPLOGS]-03-pipeline-processing.md` |
| Bucket design, retention policies (`dt.system.bucket`), routing to buckets, access control, storage cost optimization | `-[OPLOGS]-04-buckets-governance.md` |
| Writing log DQL: filters, string matching, `parse` with DPL matchers (LD/INT/IPADDR/JSON), JSON logs, null handling | `-[OPLOGS]-05-querying-parsing.md` |
| Entity context on logs: `dt.entity.host`/`process_group`/`service`, `k8s.*` fields, cross-entity correlation, entity-ID lookups | `-[OPLOGS]-06-topology.md` |
| Aggregations, `makeTimeseries`, statistical/trend analysis, log pattern analysis, dashboard-ready log queries | `-[OPLOGS]-07-analytics.md` |
| Finding sensitive data (PII, credit cards, SSNs, tokens), masking configuration, audit log queries, compliance reporting | `-[OPLOGS]-08-security.md` |
| Consolidated settings checklist: pipeline stage order, retention values by log type, masking patterns, DQL cookbook | `-[OPLOGS]-99-best-practice-summary.md` |

If more than three rows match, start with `-[OPLOGS]-99-best-practice-summary.md`
and follow its pointers.

## Related series

- OpenPipeline for other signal types — spans, metrics, business/security events (this series is logs only): `../OPIPE - OpenPipeline Beyond Logs/`
- Migrating *from* classic log processing (processing rules, log metrics, log events → OpenPipeline): `../OPMIG - OpenPipeline Migration/`
- Grail buckets, segments, and security context beyond logs: `../ORGNZ - Organize Data: Buckets, Segments, Security/`
- OneAgent log collection and Kubernetes log ingestion upstream of the pipeline: `../K8S - Kubernetes Monitoring/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "OPLOGS-03") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
