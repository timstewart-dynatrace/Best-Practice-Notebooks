# AGENTS.md — FAQ: Frequently Asked Questions

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

14 standalone single-page reference entries, each answering one recurring
Dynatrace question in decision-support format. Entries are independent — there
is no reading order; match the question and read only that entry.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| Why host-group naming matters: ownership, access control, alert routing, automation scope, single-group risks | `-[FAQ]-01-host-group-naming-strategy.md` |
| Tagging: the four tag sources, primary tags/fields vs auto-tags, per-cloud specifics, taxonomy standards, anti-patterns | `-[FAQ]-02-tagging-sources-standards-strategy.md` |
| OneAgent vs OpenTelemetry: convert vs layer vs leave-alone vs greenfield, per-runtime coverage (Java/.NET/Node/Python/Go/PHP/Ruby), async context propagation | `-[FAQ]-03-oneagent-vs-otel-decision-framework.md` |
| OneAgent update modes, tenant/host-group/host precedence, update vs maintenance windows, DynaKube `autoUpdate` deprecation, rollback | `-[FAQ]-04-managing-oneagent-updates-saas.md` |
| ActiveGate updates: auto vs manual, ActiveGates-before-OneAgents sequencing, HA-pair rolling updates, role-specific validation | `-[FAQ]-05-managing-activegate-updates-saas.md` |
| Trusting Davis AI: data residency, model training boundary, hallucination controls, autonomy limits, audit trail, compliance posture | `-[FAQ]-06-can-we-trust-davis-ai.md` |
| Launcher pages / Launchpads: tenant default vs group vs personal precedence, admin mechanism, persona content examples | `-[FAQ]-07-launcher-page-setup.md` |
| Why a log file isn't being collected: auto-discovery three-gate rules, built-in include/exclude, custom log sources, scale limits | `-[FAQ]-08-oneagent-log-autodiscovery.md` |
| Metric vs raw-log queries: DPS query economics (log scans billed, metric reads included), recurring-vs-one-shot rule, extraction, cardinality | `-[FAQ]-09-metrics-instead-of-log-queries.md` |
| ActiveGate sizing: capability-to-dimension map, host/K8s/synthetic baselines, headroom and survivor-capacity math, `dt.sfm.active_gate.*` saturation signals, scale up vs out | `-[FAQ]-10-activegate-sizing-and-scaling.md` |
| How metrics work end to end: gauge/count data-point model, Metrics Classic vs Grail dual-write (`builtin:` vs `dt.*`), OTLP delta-only, rollup/retention, cardinality limits, DPS billing | `-[FAQ]-11-how-metrics-work.md` |
| Partial enablement after migrating from another tool: mode ladder (Discovery/Infrastructure/Full-Stack), dependency cascade, per-capability handicaps, coverage-audit DQL | `-[FAQ]-12-cost-of-coverage-gaps.md` |
| OpenShift pods rejected with `Forbidden: seccomp may not be set`: anyuid + Operator 1.9.0 default flip, SCC compatibility matrix, custom SCC fix, pre-upgrade audit | `-[FAQ]-13-openshift-scc-seccomp-injection.md` |
| Replacing custom SQL Server / Telegraf monitoring scripts with the Dynatrace extension: `sql-server.*` metric mapping, honest gaps (tempdb version store), EF 2.0 fallback | `-[FAQ]-14-sql-server-extension-vs-custom-scripts.md` |

## Related series

- Instrumentation depth behind FAQ-03: `../OTEL - OpenTelemetry Integration/`
- DPS consumption and cost questions beyond FAQ-09: `../FINOPS - Cost Management & FinOps/`
- Operator/DynaKube depth behind FAQ-04 and FAQ-13: `../K8S - Kubernetes Monitoring/`
- Database monitoring depth behind FAQ-14: `../DBMON - Database Monitoring/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "FAQ-09") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
