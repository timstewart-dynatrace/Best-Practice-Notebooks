# AGENTS.md — SL2DT: Sumo Logic to Dynatrace

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

10 notebooks forming a procedural runbook for migrating from Sumo Logic to
Dynatrace: strategy and wave planning, Sumo inventory and cut-scope, log
ingest architecture, SumoQL→DQL translation, monitor and dashboard
conversion, IAM governance, GitOps automation, and cutover/decommission.
Scope is logs, dashboards, monitors, and Field Extraction Rules — Cloud SIEM
is out of scope.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| Migration strategy, the five-wave pattern, MVP-vs-full-fidelity cut scope, train-the-trainer, anti-patterns to avoid | `-[SL2DT]-01-overview-and-migration-strategy.md` |
| Inventorying Sumo (dashboards, monitors, FERs, collectors, roles), audit-log-driven usage analysis, `_sourceCategory` taxonomy mapping, the cut-scope decision | `-[SL2DT]-02-assessment-and-inventory.md` |
| Grail bucket design, deploying OneAgent/OTel collectors, OpenPipeline pipeline setup, converting Field Extraction Rules to processors, ingest parity validation | `-[SL2DT]-03-log-ingest-architecture.md` |
| Translating SumoQL (`parse`, `where`, `count by`, scheduled searches) to DQL, batch translation, confidence scoring, LOW-confidence triage | `-[SL2DT]-04-sumoql-to-dql-translation.md` |
| Rebuilding Sumo Monitors: anomaly detection vs Metric Events vs Workflows decision, notification actions, alert-noise tuning | `-[SL2DT]-05-monitor-and-alert-conversion.md` |
| Dashboard rebuild: Notebooks vs Dashboards target choice, panel-to-section mapping, visualization types, dashboard variables → parameters | `-[SL2DT]-06-dashboard-conversion.md` |
| Translating Sumo RBAC (roles, search filters, capabilities) to IAM groups/policies, bucket-scoped policies, SSO and user provisioning | `-[SL2DT]-07-user-governance-and-access.md` |
| Terraform vs Monaco tool selection, CI/CD promotion pipelines, automating the Sumo-extract → Dynatrace-import flow | `-[SL2DT]-08-automation-and-gitops.md` |
| Parallel-run validation tiers, the cutover runbook, rollback plan, decommissioning Sumo | `-[SL2DT]-09-cutover-validation-decommission.md` |
| Series index, critical-decisions checklist, failure-mode reference, engagement timeline, DQL quick reference | `-[SL2DT]-99-summary-and-runbook-index.md` |

If more than three rows match, start with
`-[SL2DT]-99-summary-and-runbook-index.md` and follow its pointers. For an
end-to-end migration, read in numbered order starting at
`-[SL2DT]-01-overview-and-migration-strategy.md`.

## Related series

- Ongoing OpenPipeline log parsing/routing after the migration: `../OPLOGS - OpenPipeline Logs/`
- Platform IAM depth behind the SL2DT-07 governance step: `../IAM - IAM Administration/`
- Terraform/Monaco automation catalog behind SL2DT-08: `../AUTOM - Dynatrace Automation/`
- Analogous Splunk → Dynatrace migration runbook: `../S2D - Splunk to Dynatrace Migration/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "SL2DT-04") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
