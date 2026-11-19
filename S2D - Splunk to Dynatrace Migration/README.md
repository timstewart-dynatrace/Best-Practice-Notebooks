# S2D - Splunk to Dynatrace Migration

Guide for migrating monitoring capabilities from Splunk to Dynatrace.

> **Recommended:** For the best experience, import the notebooks into your Dynatrace tenant. Each one is available as a Dynatrace notebook (JSON) file, and its DQL queries run against your own environment's data.

## Structure
- notebooks/ — Dynatrace notebook JSON files
- pdfs/ — Printable versions of each notebook
- markdown/ — Markdown exports of the notebooks

## Notebook Lineup
1. [Getting Started](markdown/-[S2D]-01-getting-started.md) — Migration overview and roadmap
2. [Locating Logs](markdown/-[S2D]-02-locating-logs.md) — Finding and verifying log data in Dynatrace
3. [SPL to DQL Translation](markdown/-[S2D]-03-spl-to-dql.md) — Converting Splunk queries to DQL
4. [Davis Anomaly Detectors](markdown/-[S2D]-04-anomaly-detectors.md) — Migrating alerts to continuous monitoring
5. [Workflow-Based Alerts](markdown/-[S2D]-05-workflow-alerts.md) — Migrating alerts to scheduled workflows
6. [ArrayMovingSum](markdown/-[S2D]-06-arraymovingsum.md) — Rolling sums for extended timeframes
7. [Metric Creation](markdown/-[S2D]-07-metric-creation.md) — Extracting metrics from logs via OpenPipeline
8. [Dashboard Migration](markdown/-[S2D]-08-dashboard-migration.md) — Converting Splunk dashboards to Dynatrace
9. [Naming Standards](markdown/-[S2D]-09-naming-standards.md) — Asset naming conventions and organization
99. [Best Practice Summary](markdown/-[S2D]-99-best-practice-summary.md) — Consolidated best practices from the S2D series

## Usage
1. Open a notebook from the lineup below. Each one is also available as a Dynatrace notebook (JSON) to import into your tenant, so its queries run against your own data, and as a PDF for print.
2. Follow the numbered sequence for a complete migration journey.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
