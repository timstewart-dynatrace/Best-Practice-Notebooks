# Doorway 2 — Expanding or Consolidating

> **Purpose:** Reading order for existing Dynatrace customers adding scope or pulling data from another tool. Skips most Foundation reading; focuses on the specific expansion or consolidation pattern.
> **Last Updated:** 07/23/2026

![Expanding or Consolidating Sub-Paths](images/02-expand-subpaths.svg)

---

## Table of Contents

1. [You Are Here If…](#you-are-here-if)
2. [Pick Your Sub-Path](#pick-your-sub-path)
3. [Sub-Path A — Adding a New Domain](#sub-path-a--adding-a-new-domain)
4. [Sub-Path B — Log Consolidation from Splunk or Sumo Logic](#sub-path-b--log-consolidation-from-splunk-or-sumo-logic)
5. [Sub-Path C — APM Consolidation from New Relic](#sub-path-c--apm-consolidation-from-new-relic)
6. [Sub-Path D — Maturing Operations](#sub-path-d--maturing-operations)
7. [Foundation Refresh Checkpoints](#foundation-refresh-checkpoints)
8. [Where to Next](#where-to-next)

---

## You Are Here If…

- You already have a working Dynatrace tenant
- You are extending its scope or consolidating data from another tool into it
- Examples: adding Kubernetes coverage, replacing Splunk for a subset of log sources, replacing New Relic APM for one application portfolio, building dashboards and automation for a maturing practice

If you are net-new to Dynatrace (no tenant yet), see [Doorway 1 — Net New](01-net-new.md). If you are migrating an existing Dynatrace deployment between deployment models, see [Doorway 3 — Deployment Migration](03-deployment-migration.md). If your tenant still runs classic constructs — management zones as permissions, metric events, alerting profiles, Classic Logs — see [Doorway 4 — Classic → Gen3 Platform](04-gen2-to-gen3.md); that migration is worth completing before layering new scope on top of it.

---

## Pick Your Sub-Path

| If you are doing… | Go to… |
|---|---|
| Adding a new observability domain (K8s, Mobile, RUM, DBMon, Security, etc.) | [Sub-Path A — Adding a New Domain](#sub-path-a--adding-a-new-domain) |
| Pulling logs in from Splunk or Sumo Logic without replacing them everywhere | [Sub-Path B — Log Consolidation](#sub-path-b--log-consolidation-from-splunk-or-sumo-logic) |
| Adding APM data from New Relic for some applications without full replacement | [Sub-Path C — APM Consolidation](#sub-path-c--apm-consolidation-from-new-relic) |
| Maturing operations (dashboards, alerting, SLOs, automation, AI) | [Sub-Path D — Maturing Operations](#sub-path-d--maturing-operations) |

Multiple sub-paths can run in parallel.

---

## Sub-Path A — Adding a New Domain

| Step | Reading |
|---|---|
| 1. Pick the domain | See [Domain Enablement Module](06-domain-enablement.md) for the recommended first-read in each domain |
| 2. Confirm prerequisites | Domain Enablement lists what Foundation pieces each domain needs (most assume basic [ONBRD](../ONBRD%20-%20Dynatrace%20Onboarding/) + [ORGNZ](../ORGNZ%20-%20Organize%20Data:%20Buckets,%20Segments,%20Security/) already in place) |
| 3. Read the domain series | Direct entry into the relevant series — [K8S](../K8S%20-%20Kubernetes%20Monitoring/), [CLOUD](../CLOUD%20-%20Cloud%20Provider%20Integrations/), [SPANS](../SPANS%20-%20Distributed%20Tracing%20and%20Spans/), [WEBRUM](../WEBRUM%20-%20Web%20Real%20User%20Monitoring/), [MOBL](../MOBL%20-%20Mobile%20Monitoring/), [DBMON](../DBMON%20-%20Database%20Monitoring/), [BIZEV](../BIZEV%20-%20Business%20Events%20&%20Funnel%20Analysis/), [SYNTH](../SYNTH%20-%20Synthetic%20Monitoring/), [APPSEC](../APPSEC%20—%20Application%20Security/) |
| 4. Foundation refresh if needed | See [Foundation Refresh Checkpoints](#foundation-refresh-checkpoints) below — Gen3 changes to ORGNZ and IAM may apply if your tenant predates them |
| 5. Operationalize the new domain | Add dashboards ([DASH](../DASH%20-%20Dashboard%20Design%20&%20Building/)) and alerts ([WFLOW](../WFLOW%20-%20Workflows%20and%20Alert%20Notifications/)) for the new domain |

Time per domain: 1–3 weeks for a small team. Mobile (SDK rollout to apps) and full Kubernetes coverage typically take longer.

---

## Sub-Path B — Log Consolidation from Splunk or Sumo Logic

You are pulling some or all log sources from Splunk or Sumo Logic into Dynatrace, often in parallel with the legacy tool for a transition period.

| Step | Reading | Notes |
|---|---|---|
| 1. Inventory | [S2D](../S2D%20-%20Splunk%20to%20Dynatrace%20Migration/) notebook 02 (locating logs) for Splunk; [SL2DT](../SL2DT%20-%20Sumo%20Logic%20to%20Dynatrace/) notebook 02 (assessment and inventory) for Sumo Logic | Catalog source-types/categories, retention requirements, query patterns |
| 2. Routing & ingestion design | [OPLOGS](../OPLOGS%20-%20OpenPipeline%20Logs/) — notebooks 01–04 (fundamentals, migration, pipeline, buckets); [OPIPE](../OPIPE%20-%20OpenPipeline%20Beyond%20Logs/) — notebook 01 | This is where Splunk source-types or Sumo `_sourceCategory` map to Dynatrace bucket and security_context strategy |
| 3. ORGNZ refresh | [ORGNZ](../ORGNZ%20-%20Organize%20Data:%20Buckets,%20Segments,%20Security/) — notebooks 02 (buckets), 03 (bucket strategy), 06 (security_context) | Critical if your tenant is Gen2-era or has not implemented bucket-based scoping |
| 4. Translation | [S2D](../S2D%20-%20Splunk%20to%20Dynatrace%20Migration/) notebook 03 (SPL → DQL) or [SL2DT](../SL2DT%20-%20Sumo%20Logic%20to%20Dynatrace/) notebook 04 (SumoQL → DQL) | Translate dashboards, monitors, scheduled searches |
| 5. Dashboard & monitor migration | [S2D](../S2D%20-%20Splunk%20to%20Dynatrace%20Migration/) notebooks 04–08 or [SL2DT](../SL2DT%20-%20Sumo%20Logic%20to%20Dynatrace/) notebooks 05–06 | Build Dynatrace equivalents; flag static-threshold monitors that should become Davis Anomaly Detectors |
| 6. Validation | Run side-by-side for 2–4 weeks; compare key queries | |
| 7. Cutover | Decommission source forwarders for migrated sources; legacy tool stays for non-migrated sources | |

Skip: [ONBRD](../ONBRD%20-%20Dynatrace%20Onboarding/) basics, [IAM](../IAM%20-%20IAM%20Administration/) notebooks 01–03 (assume tenant already operational and access already configured).

---

## Sub-Path C — APM Consolidation from New Relic

You are migrating APM coverage for one or more application portfolios from New Relic to Dynatrace, while leaving other tools in place.

| Step | Reading | Notes |
|---|---|---|
| 1. Component inventory | [NRLC](../NRLC%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Deep%20Dives/) notebook 01 (platform comparison) | Identify which NR features are in use for the portfolio (APM, browser, mobile, synthetics, alerts, dashboards, logs) |
| 2. Translation primer | [NRLC](../NRLC%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Deep%20Dives/) notebook 02 (NRQL → DQL) | Sets expectations for what translates cleanly vs. needs redesign |
| 3. Component migration (per app) | [NRLC](../NRLC%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Deep%20Dives/) notebooks 03–07 — dashboards, alerts and workflows, synthetics, SLOs and workloads, logs and tags | Pick the components in scope; skip what does not apply |
| 4. Validation | [NRLC](../NRLC%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Deep%20Dives/) notebook 08 (validation, diff, rollback) | Side-by-side comparison; rollback plan in case of issues |
| 5. Cutover & decommission | NR agent removal; license adjustment | |
| 6. Toolchain reference | [NRLC](../NRLC%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Deep%20Dives/) notebook 09 (toolchain reference) | For ongoing tooling: NRQL → DQL translator, asset inventory tools |

Skip: [NR2DT](../NR2DT%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Steps/) procedural runbook (that's for full-tenant migrations); [ONBRD](../ONBRD%20-%20Dynatrace%20Onboarding/) notebooks 01–05 (assume tenant operational).

---

## Sub-Path D — Maturing Operations

Your tenant is operational; you are improving the depth and quality of dashboards, alerting, automation, and AI-driven analysis.

| Step | Reading |
|---|---|
| 1. Dashboard maturity | [DASH](../DASH%20-%20Dashboard%20Design%20&%20Building/) — full series; especially notebook 02 (hierarchy) for organization across teams |
| 2. Alert routing maturity | [ALERT](../ALERT%20-%20Alerting%20Strategy%20and%20Design/) — full series for end-to-end alerting architecture; then [WFLOW](../WFLOW%20-%20Workflows%20and%20Alert%20Notifications/) — full series; especially notebooks 04 (notification routing), 05 (incident management), 09 (governance) |
| 3. Reliability targets (optional) | [SLO](../SLO%20-%20Service%20Level%20Objectives/) — full series if building formal SLOs |
| 4. Configuration automation | [AUTOM](../AUTOM%20-%20Dynatrace%20Automation/) — notebooks 01–04 are the foundation (Settings API, Monaco, Terraform); 05–08 build on that (workflows-as-code, SDKs, CI/CD, migration automation) |
| 4. Davis intelligence | [AIOPS](../AIOPS%20-%20Dynatrace%20Intelligence/) — full series; especially notebook 02 (anomaly detection), 03 (Davis problems and root cause), 06 (integrations and agentic workflows) |
| 5. Continuous improvement | [Maturity Module](08-maturity.md) → [ADOPT](../ADOPT%20-%20Observability%20Adoption%20&%20Maturity/) — ongoing |

See [Operationalize Module](07-operationalize.md) for the recommended order of these series and the reasoning behind it.

---

## Foundation Refresh Checkpoints

Even existing customers may need to refresh on Foundation topics, particularly if your tenant predates Gen3 or you are touching a previously-unaddressed area.

If several rows below apply at once, you are not doing a refresh — you are doing a surface migration. Read [Doorway 4 — Classic → Gen3 Platform](04-gen2-to-gen3.md) instead, which sequences these into phases rather than treating them as independent catch-ups.

| Topic | Why refresh | Reading |
|---|---|---|
| Gen3 IAM (security_context, parameterized policies) | Gen2 IAM does not exist in Gen3 the same way; if you have Gen2 management zones, see MZ2POL | [IAM](../IAM%20-%20IAM%20Administration/) — notebooks 04 (policy authoring), 05 (boundary design), 10 (parameterized assignments); [MZ2POL](../MZ2POL%20-%20Management%20Zone%20to%20Policy%20Migration/) full series if migrating Gen2 permissions, or notebook 05 alone if the MZs are used for filtering rather than permissions |
| Bucket strategy | Gen2-era tenants often have not implemented bucket-based scoping; bucket choices have downstream effects on cost, retention, and access | [ORGNZ](../ORGNZ%20-%20Organize%20Data:%20Buckets,%20Segments,%20Security/) — notebooks 02 (buckets), 03 (bucket strategy), 05 (bucket-level access control) |
| Tagging strategy | Tagging conventions established years ago may need rationalization | [FAQ](../FAQ%20-%20Frequently%20Asked%20Questions/) — entry 02 (tagging sources, standards, strategy); [ORGNZ](../ORGNZ%20-%20Organize%20Data:%20Buckets,%20Segments,%20Security/) — notebook 01 |
| security_context as universal scope field | If you have not adopted security_context, IAM boundaries are limited | [ORGNZ](../ORGNZ%20-%20Organize%20Data:%20Buckets,%20Segments,%20Security/) — notebook 06 |
| Classic Logs → OpenPipeline | If you are still on Classic Logs, OpenPipeline migration is a separate workstream | [OPMIG](../OPMIG%20-%20OpenPipeline%20Migration/) — full series |

These are not blockers — refresh as needed, in parallel with your sub-path.

---

## Where to Next

- [Operationalize Module](07-operationalize.md) — for ongoing operations work
- [Maturity Module](08-maturity.md) — for continuous improvement framing
- [Overlap Map](09-overlap-map.md) — when you find the same topic covered in multiple series
- [Foundation Module](05-foundation.md) — full Foundation reading order if you decide to do a comprehensive refresh

---

> *This playbook was AI-generated from community-submitted and publicly available sources. It is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*
