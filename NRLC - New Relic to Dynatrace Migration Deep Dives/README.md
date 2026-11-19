# NRLC - New Relic to Dynatrace Migration Deep Dives

Technical deep-dive companion to the NR2DT series — focused reference notebooks for query translation, dashboard migration, alerting, synthetics, SLOs, logs, and validation.

> **Recommended:** Import the JSON files from notebooks/ into a Dynatrace tenant for the best experience. These notebooks contain interactive DQL queries that execute against your environment's data.

## Structure
- notebooks/ — Dynatrace notebook JSON files
- pdfs/ — Printable versions of each notebook
- markdown/ — Markdown exports of the notebooks

## Notebook Lineup
1. [Platform Comparison](markdown/-[NRLC]-01-platform-comparison.md) — Side-by-side comparison of New Relic and Dynatrace concepts
2. [NRQL → DQL Translation](markdown/-[NRLC]-02-nrql-to-dql-translation.md) — Translating NRQL queries into DQL with worked examples
3. [Dashboard Migration](markdown/-[NRLC]-03-dashboard-migration.md) — Rebuilding New Relic dashboards in Dynatrace
4. [Alert & Workflow Migration](markdown/-[NRLC]-04-alert-workflow-migration.md) — Migrating alert conditions and notification workflows
5. [Synthetic Monitor Migration](markdown/-[NRLC]-05-synthetic-monitor-migration.md) — Moving synthetic monitors and private locations
6. [SLO & Workload Migration](markdown/-[NRLC]-06-slo-workload-migration.md) — Porting SLOs and workload definitions
7. [Logs, Tags & Drop Rules](markdown/-[NRLC]-07-logs-tags-drops.md) — Log pipelines, tagging strategies, and drop-rule equivalents
8. [Validation, Diff & Rollback](markdown/-[NRLC]-08-validation-diff-rollback.md) — Parallel-run validation, diff checks, and rollback strategy
9. [Toolchain Reference & End-to-End Runbook](markdown/-[NRLC]-09-toolchain-reference.md) — Tooling reference and end-to-end migration runbook

## Usage
1. Open a notebook from the lineup below. Each one is also available as a Dynatrace notebook (JSON) to import into your tenant, so its queries run against your own data, and as a PDF for print.
2. Use alongside the NR2DT step-by-step series for full migration context.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
