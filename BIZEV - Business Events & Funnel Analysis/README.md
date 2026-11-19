# BIZEV - Business Events & Funnel Analysis

Turning business transactions into first-class observability data with Dynatrace Business Events.

> **Recommended:** Import the JSON files from notebooks/ into a Dynatrace tenant for the best experience. These notebooks contain interactive DQL queries that execute against your environment's data.

## Structure
- notebooks/ — Dynatrace notebook JSON files
- pdfs/ — Printable versions of each notebook
- markdown/ — Markdown exports of the notebooks

## Notebook Lineup
1. [Business Events Fundamentals](markdown/-[BIZEV]-01-business-events-fundamentals.md) — Business events data model, ingestion methods, and DQL patterns
2. [Instrumentation](markdown/-[BIZEV]-02-instrumentation.md) — Capturing transactional events through OneAgent capture rules, the ingest API, RUM/mobile APIs, and OpenPipeline extraction
3. [Funnel Analysis](markdown/-[BIZEV]-03-funnel-analysis.md) — Analyzing conversion flows, identifying drop-off points, and measuring user journeys
4. [Revenue Impact Analysis](markdown/-[BIZEV]-04-revenue-impact.md) — Correlating business events with financial outcomes and quantifying impact
5. [KPIs and Metrics](markdown/-[BIZEV]-05-kpis-and-metrics.md) — Defining business metrics, calculating KPIs, and creating metric events
6. [Executive Reporting](markdown/-[BIZEV]-06-executive-reporting.md) — Building actionable business dashboards and executive reports
7. [Gen2 vs Gen3 — Business Events Without (or Before) the Full Move](markdown/-[BIZEV]-07-gen2-vs-gen3-adoption-paths.md) — Decision framework for Managed and classic-operated tenants: what works without Grail, adoption paths, and the migration sequence when you're ready to move
99. [Best Practice Summary](markdown/-[BIZEV]-99-best-practice-summary.md) — Consolidated best practices from the BIZEV series

## Usage
1. Open a notebook from the lineup below. Each one is also available as a Dynatrace notebook (JSON) to import into your tenant, so its queries run against your own data, and as a PDF for print.
2. Start with Business Events Fundamentals to understand the data model and ingestion approach.
3. Instrument your applications to capture meaningful business transactions.
4. Create funnels and dashboards to measure business outcomes and impact.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
