# SLO - Service Level Objectives

Defining and operating Service Level Objectives in Dynatrace — SLIs, error budgets, composition, burn-rate alerting, and SLOs as code. Every DQL query in the series was validated against a live tenant.

> **Recommended:** Import the JSON files from NOTEBOOKS/ into a Dynatrace tenant for the best experience. These notebooks contain interactive DQL queries that execute against your environment's data.

## Structure
- NOTEBOOKS/ — Dynatrace notebook JSON files
- PDFs/ — Printable versions of each notebook
- markdown/ — Markdown exports of the notebooks

## Notebook Lineup
1. [SLO and SLI Fundamentals](markdown/-[SLO]-01-fundamentals.md) — The three building blocks (SLI, SLO, error budget), the Dynatrace SLO model, and choosing your first SLOs
2. [Defining SLIs](markdown/-[SLO]-02-defining-slis.md) — The good ÷ total ratio as DQL: availability, latency, error-rate, and custom business SLIs
3. [Composition and Error Budgets](markdown/-[SLO]-03-composition-and-error-budgets.md) — Error-budget math and burn rate, composite and weighted-global SLOs, and rolling vs calendar windows
4. [SLO Alerting](markdown/-[SLO]-04-alerting.md) — Burn-rate alerting over threshold-on-SLI: fast-burn and slow-burn multiwindow alerts, surfacing, routing, and avoiding fatigue
5. [SLOs as Code](markdown/-[SLO]-05-slos-as-code.md) — Promoting SLOs into version control: the `dynatrace_platform_slo` Terraform resource and SLO Service Public API for the modern app, the classic `dynatrace_slo_v2` / `builtin:monitoring.slo` path it replaces, Monaco, and the export-known-good API pattern

## Usage
1. Choose a format: import JSON from NOTEBOOKS/, read PDFs/ for print, or view markdown/ for lightweight browsing.
2. Start with the Fundamentals for orientation, then follow the numbered sequence — each notebook builds on the previous.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
