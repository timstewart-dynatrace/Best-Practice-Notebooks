# SL2DT - Sumo Logic to Dynatrace

Step-by-step migration path from Sumo Logic to Dynatrace, from strategy and inventory through cutover and decommission.

> **Recommended:** Import the JSON files from notebooks/ into a Dynatrace tenant for the best experience. These notebooks contain interactive DQL queries that execute against your environment's data.

## Structure
- notebooks/ — Dynatrace notebook JSON files
- pdfs/ — Printable versions of each notebook
- markdown/ — Markdown exports of the notebooks

## Notebook Lineup
1. [Overview & Migration Strategy](markdown/-[SL2DT]-01-overview-and-migration-strategy.md) — Migration strategy, mental model, and the five-wave pattern
2. [Assessment & Inventory](markdown/-[SL2DT]-02-assessment-and-inventory.md) — Inventorying Sumo Logic and producing the cut-scope decision artifact
3. [Log Ingest Architecture](markdown/-[SL2DT]-03-log-ingest-architecture.md) — Designing Grail buckets and deploying OneAgent, OTel, and OpenPipeline
4. [SumoQL → DQL Translation](markdown/-[SL2DT]-04-sumoql-to-dql-translation.md) — Translating SumoQL queries, dashboards, and monitor conditions to DQL
5. [Monitor & Alert Conversion](markdown/-[SL2DT]-05-monitor-and-alert-conversion.md) — Rebuilding Sumo Monitors using Anomaly Detection, Workflows, or Metric Events
6. [Dashboard Conversion](markdown/-[SL2DT]-06-dashboard-conversion.md) — Rebuilding in-scope Sumo dashboards in Dynatrace Notebooks and Dashboards
7. [User Governance & Access](markdown/-[SL2DT]-07-user-governance-and-access.md) — Translating Sumo RBAC to Dynatrace Platform IAM groups and policies
8. [Automation & GitOps](markdown/-[SL2DT]-08-automation-and-gitops.md) — CI/CD promotion paths for all migrated configuration, split across Terraform and Monaco
9. [Cutover, Validation & Decommission](markdown/-[SL2DT]-09-cutover-validation-decommission.md) — Parallel validation, declaring cutover, and decommissioning Sumo
10. [Migrating Telegraf-Collected Metrics](markdown/-[SL2DT]-10-telegraf-metric-migration.md) — Repointing a Telegraf metric estate at Dynatrace: ingest-path choice, output-plugin mechanics, key reshaping, and parity validation
99. [Summary & Runbook Index](markdown/-[SL2DT]-99-summary-and-runbook-index.md) — Single-page index and reference card for the entire SL2DT series

## Usage
1. Choose a format: import JSON from notebooks/, read pdfs/ for print, or view markdown/ for lightweight browsing.
2. Follow the numbered sequence for a complete migration journey.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
