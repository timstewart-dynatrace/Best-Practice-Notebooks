# NR2DT - New Relic to Dynatrace Migration Steps

Step-by-step migration path from New Relic to Dynatrace, from discovery through cutover and decommission.

> **Recommended:** For the best experience, import the notebooks into your Dynatrace tenant. Each one is available as a Dynatrace notebook (JSON) file, and its DQL queries run against your own environment's data.

## Structure
- notebooks/ — Dynatrace notebook JSON files
- pdfs/ — Printable versions of each notebook
- markdown/ — Markdown exports of the notebooks

## Notebook Lineup
0. [Step 0 — Tenant Prerequisites](markdown/-[NR2DT]-00-step-0-prerequisites.md) — Readiness gate confirming the target Dynatrace tenant is fit to receive a New Relic migration
1. [Step 1 — Discover](markdown/-[NR2DT]-01-step-1-discover.md) — Inventorying the New Relic environment and understanding migration scope
2. [Step 2 — Strategize](markdown/-[NR2DT]-02-step-2-strategize.md) — Planning the migration strategy, phasing, and execution approach
3. [Step 3 — Design](markdown/-[NR2DT]-03-step-3-design.md) — Designing the target Dynatrace architecture and configuration
4. [Step 4 — Translate](markdown/-[NR2DT]-04-step-4-translate.md) — Translating NRQL queries, dashboards, and alerts to Dynatrace equivalents
5. [Step 5 — Migrate Dashboards & Alerts](markdown/-[NR2DT]-05-step-5-migrate-dashboards-alerts.md) — Porting dashboards and alerting rules
6. [Step 6 — Migrate Synthetics, SLOs & Workloads](markdown/-[NR2DT]-06-step-6-migrate-synthetics-slos-workloads.md) — Moving synthetic monitors, SLOs, and workload definitions
7. [Step 7 — Migrate Logs, Tags & Drop Rules](markdown/-[NR2DT]-07-step-7-migrate-logs-tags-drops.md) — Migrating log ingestion, tagging schemes, and drop rules
8. [Step 8 — Validate](markdown/-[NR2DT]-08-step-8-validate.md) — Parallel-run validation and diff checks between source and target
9. [Step 9 — Cutover, Rollback & Decommission](markdown/-[NR2DT]-09-step-9-cutover-rollback-decommission.md) — Executing cutover, preparing rollback plans, and decommissioning New Relic
99. [Best Practice Summary](markdown/-[NR2DT]-99-best-practice-summary.md) — Consolidated best practices from the NR2DT series

## Usage
1. Open a notebook from the lineup below. Each one is also available as a Dynatrace notebook (JSON) to import into your tenant, so its queries run against your own data, and as a PDF for print.
2. Follow the numbered sequence for a complete migration journey.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
