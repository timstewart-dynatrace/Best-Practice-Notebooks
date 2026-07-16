# AGENTS.md — BIZEV: Business Events & Funnel Analysis

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

8 notebooks on turning business transactions into observability data with
Dynatrace Business Events (`bizevents`): the data model, instrumentation,
conversion funnels, revenue impact during incidents, business KPIs, executive
reporting, Gen2/Managed adoption paths without a full Grail move, and a
best-practice summary.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| What business events are, `bizevents` vs `events`/`logs`, data model (`event.type`, `event.provider`, `event.category`), ingestion methods, first `fetch bizevents` queries | `-[BIZEV]-01-business-events-fundamentals.md` |
| Capturing events: OneAgent capture rules, Business Events API (`bizevents.ingest`), SDK integration, OTel span-to-bizevent mapping, event naming, payload/cardinality design | `-[BIZEV]-02-instrumentation.md` |
| Building multi-step conversion funnels, step-by-step conversion rates, drop-off points, time between steps, funnel segmentation and trends | `-[BIZEV]-03-funnel-analysis.md` |
| Quantifying revenue lost during a detected problem, incident-vs-baseline comparison, business-hours filtering, day-over-day comparison, SLA impact, impact timelines | `-[BIZEV]-04-revenue-impact.md` |
| Defining business KPIs (transaction volume, AOV, error rate), extracting metrics from bizevents via OpenPipeline, business health scores, trend analysis | `-[BIZEV]-05-kpis-and-metrics.md` |
| Executive-ready queries combining technical + business signals, business SLA/SLO tracking, scheduled report patterns, weekly business review queries | `-[BIZEV]-06-executive-reporting.md` |
| Running Managed or a classic-operated (Gen2) tenant: whether business events work without Grail, minimal-Gen3-footprint hybrid adoption, `bizevents.*` metric bridge to classic dashboards, classic toolchain (request attributes, USQL, log metrics), DDU vs DPS licensing | `-[BIZEV]-07-gen2-vs-gen3-adoption-paths.md` |
| Consolidated checklist: schema rules, ingestion, naming, funnel/KPI/reporting patterns, Gen2 vs Gen3 adoption practices | `-[BIZEV]-99-best-practice-summary.md` |

If more than three rows match, start with `-[BIZEV]-99-best-practice-summary.md`
and follow its pointers.

## Related series

- Ingestion-time processing of business events (enrichment, routing, metric extraction pipelines): `../OPIPE - OpenPipeline Beyond Logs/`
- Building the dashboards that surface these KPIs to stakeholders: `../DASH - Dashboard Design & Building/`
- Moving a Managed environment to SaaS/Grail (the prerequisite path from BIZEV-07): `../M2S - Managed to SaaS Migration/`
- Capturing user behavior on the frontend (RUM sessions, user actions): `../WEBRUM - Web Real User Monitoring/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "BIZEV-03") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
