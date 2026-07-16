# AGENTS.md — NRLC: New Relic to Dynatrace Migration Deep Dives

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

9 standalone component-reference notebooks — the technical companion to the
NR2DT step-by-step runbook. Each covers one migration artifact class in depth:
platform concepts, NRQL→DQL compilation, dashboards, alerts, synthetics,
SLOs/workloads, logs/tags/drops, validation, and the open-source toolchain
(`Dynatrace-NewRelic` / `nrql-engine` / `nrql-translator`).

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| New Relic vs Dynatrace concepts: architecture, ingest/storage model, NRQL-vs-DQL philosophy, entity hierarchy, IAM, pricing, concept mapping | `-[NRLC]-01-platform-comparison.md` |
| Translating a specific NRQL query to DQL: the 292-pattern compiler, `FROM`-clause source mapping, aggregation-function and time-expression mapping, confidence scoring, unsupported NRQL | `-[NRLC]-02-nrql-to-dql-translation.md` |
| Rebuilding NR dashboards: widget-level translation, `DashboardTransformer`, multi-page dashboards, variables/dynamic content, layout preservation | `-[NRLC]-03-dashboard-migration.md` |
| Migrating NR alert policies: NRQL conditions → metric events, APM conditions → adaptive baselines, notification channels → Workflow tasks, mute rules/maintenance windows, the dual-alert window | `-[NRLC]-04-alert-workflow-migration.md` |
| Synthetic monitor type mapping (Ping / Simple Browser / API / Scripted Browser / Certificate Check / Broken Links), public/private location mapping, credential re-entry | `-[NRLC]-05-synthetic-monitor-migration.md` |
| SLO metric-expression migration, SLI math equivalence auditing, NR workloads → OpenPipeline enrichment + segments + bucket-scoped IAM, entity-selector translation | `-[NRLC]-06-slo-workload-migration.md` |
| Log forwarding patterns, NR drop rules → OpenPipeline filters, Grok → DPL parsing, tag taxonomy migration, cost optimization | `-[NRLC]-07-logs-tags-drops.md` |
| Three-tier validation (syntax / tenant / behavioral), diff against live config, rollback manifests, conversion-quality reports | `-[NRLC]-08-validation-diff-rollback.md` |
| Operating the toolchain: `migrate.py` CLI, component selection, Monaco/Terraform export formats, the end-to-end runbook, CI/CD integration | `-[NRLC]-09-toolchain-reference.md` |

Each notebook is self-contained — route by artifact class; there is no
required reading order.

## Related series

- Process runbook — "how do we run the migration end to end?" (waves, gates, cutover sequencing) — go there instead of this series when the question is about the migration process rather than translating a specific component: `../NR2DT - New Relic to Dynatrace Migration Steps/`
- Terraform resource catalog and Settings API depth behind NRLC-09's exports: `../AUTOM - Dynatrace Automation/`
- Workflow and notification-routing depth behind NRLC-04: `../WFLOW - Workflows and Alert Notifications/`
- SLO fundamentals and burn-rate alerting beyond migration parity: `../SLO - Service Level Objectives/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "NRLC-02") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
