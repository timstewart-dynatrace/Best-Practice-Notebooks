# AGENTS.md — DASH: Dashboard Design & Building

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

8 notebooks on designing and building Dynatrace dashboards for different
stakeholder audiences: fundamentals and dashboard-vs-notebook choice, the
three-tier Executive/Operations/Engineering hierarchy, one notebook per tier,
variables and filters, sharing and scheduled reporting, and a best-practice
summary.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| Dashboards vs notebooks — which to use, tile/section/variable architecture, design principles, creation workflow, first dashboard query | `-[DASH]-01-dashboard-fundamentals.md` |
| Organizing a dashboard library: the three-tier Executive/Operations/Engineering model, audience, tile counts, information density, refresh cadence per tier | `-[DASH]-02-dashboard-hierarchy.md` |
| Leadership KPI dashboards: KPI selection (SMART filter), availability %, MTTR, problem trends, service health score, error budget tiles, storytelling | `-[DASH]-03-executive-dashboards.md` |
| Real-time SRE/NOC views: service response time (p50/p90/p95), error rate by service, log volume anomalies, active problems, infrastructure health, deployment correlation | `-[DASH]-04-operations-dashboards.md` |
| Developer deep-dive dashboards: span analysis, database performance, endpoint-level metrics, deployment impact analysis, code-level metrics | `-[DASH]-05-engineering-dashboards.md` |
| Variable types (entity selector, string, query-based, time range), variable-driven DQL, filter propagation across tiles, template dashboard patterns | `-[DASH]-06-variables-and-filters.md` |
| Permission models (viewer/editor/owner), sharing with teams, scheduled reports via Workflows, exporting snapshots, dashboards as code, version control | `-[DASH]-07-sharing-and-reporting.md` |
| Consolidated checklist: architecture, layout, tile config, per-tier standards, DQL query standards, refresh/performance, lifecycle | `-[DASH]-99-best-practice-summary.md` |

If more than three rows match, start with `-[DASH]-99-best-practice-summary.md`
and follow its pointers.

## Related series

- Business KPIs and funnel data that feed executive dashboards: `../BIZEV - Business Events & Funnel Analysis/`
- SLOs, error budgets, and burn-rate signals behind health tiles: `../SLO - Service Level Objectives/`
- Managing dashboards as code at scale (Terraform, Monaco, CI/CD): `../AUTOM - Dynatrace Automation/`
- Span queries that power operations/engineering tiles: `../SPANS - Distributed Tracing and Spans/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "DASH-03") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
