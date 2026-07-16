# AGENTS.md — ALERT: Alerting Strategy and Design

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

5 notebooks on end-to-end alerting strategy and design. This series is the
cross-cutting doorway: it draws the whole board — telemetry → detection → one
enriched Davis problem → routing → destination — and orchestrates the series
that own each piece (AIOPS for detection, SLO for reliability targets, WFLOW
for routing) rather than re-documenting their mechanics.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| The end-to-end alerting architecture — what to set up where, the anti-noise funnel, the enrich-upstream-or-you-cannot-route rule, how detection/routing/reliability connect | `-[ALERT]-01-end-to-end-architecture.md` |
| Choosing between the four detection mechanisms — OOTB Davis vs custom anomaly detector vs OpenPipeline-derived metric vs SLO burn-rate — and the anti-patterns that cause noise | `-[ALERT]-02-choosing-and-building-detection.md` |
| Simple vs multi-step workflow billing (workflow-hours), the destination landscape, legacy alerting profiles, xMatters-is-legacy, cost discipline | `-[ALERT]-03-routing-destinations-cost.md` |
| ServiceNow / ITSM integration — the maturity ladder from HTTP `POST /api/now/table/incident` to native connector to bi-directional ITOM sync, what fields ServiceNow needs | `-[ALERT]-04-servicenow-integration.md` |
| Principles on one page, the complete end-to-end setup checklist, the ongoing audit loop, the cross-series build map | `-[ALERT]-99-best-practice-summary.md` |

If more than three rows match, start with `-[ALERT]-99-best-practice-summary.md`
and follow its pointers.

## Related series

- Detector build mechanics — anomaly detector types, Davis problems/RCA, CoPilot: go to `../AIOPS - Dynatrace Intelligence/` instead when the question is *how to build or tune detection*, not which mechanism to pick.
- Reliability targets — SLIs, error budgets, burn-rate math and alert config: go to `../SLO - Service Level Objectives/` instead when the question is the SLO itself.
- Notification plumbing — triggers, connections, templates, JavaScript actions: go to `../WFLOW - Workflows and Alert Notifications/` instead for routing implementation depth.
- Extracting log/span signals to alertable metrics at ingest: `../OPIPE - OpenPipeline Beyond Logs/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "ALERT-04") and mention that the matching JSON in
  `NOTEBOOKS/` can be imported into a Dynatrace tenant for interactive use.
