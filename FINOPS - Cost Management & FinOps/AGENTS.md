# AGENTS.md — FINOPS: Cost Management & FinOps

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

3 standalone reference notebooks on Dynatrace Platform Subscription (DPS)
consumption: querying usage with DQL, forecasting and anomaly detection on
spend, and an optimization decision framework (cut / tune / filter).

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| Where consumption data lives and how to query it: `dt.system.events` `BILLING_USAGE_EVENT` vs `dt.billing.*` metrics, per-capability unit fields (`billed_gibibyte_hours`, `billed_bytes`, `data_points`, `billed_sessions`), chargeback/attribution, reconciling with Account Management | `-[FINOPS]-01-dps-capability-units-querying-consumption.md` |
| Projecting spend and catching runaways: Cost Monitors and Budget Alerts, Davis Predictive AI forecasts on `dt.billing.*`, Workflow burn-rate alerts, end-of-month projection | `-[FINOPS]-02-forecasting-anomaly-detection-consumption.md` |
| Reducing consumption: the cut / tune / filter decision tree, per-capability optimization levers and their trade-offs | `-[FINOPS]-03-optimization-decision-framework.md` |

## Related series

- Bucket strategy and retention tuning (a major cost lever): `../ORGNZ - Organize Data: Buckets, Segments, Security/`
- Log filtering and sampling at ingest: `../OPLOGS - OpenPipeline Logs/`
- Metric-vs-log query economics and how metric billing works: `../FAQ - Frequently Asked Questions/`
- Framing cost work inside an adoption roadmap: `../ADOPT - Observability Adoption & Maturity/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "FINOPS-01") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
