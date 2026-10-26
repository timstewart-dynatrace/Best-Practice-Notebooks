# OPIPE - OpenPipeline Beyond Logs

Advanced OpenPipeline patterns for spans, metrics, business events, and security events — extending observability beyond log processing.

> **Recommended:** Import the JSON files from notebooks/ into a Dynatrace tenant for the best experience. These notebooks contain interactive DQL queries that execute against your environment's data.

## Structure
- notebooks/ — Dynatrace notebook JSON files
- pdfs/ — Printable versions of the notebooks
- markdown/ — Markdown exports of the notebooks

## Notebook Lineup
1. [OpenPipeline as a Multi-Scope Platform](markdown/-[OPIPE]-01-multi-scope-platform.md) — Beyond logs: processing spans, metrics, and events at ingestion
2. [Span Processing & Enrichment](markdown/-[OPIPE]-02-span-processing-and-enrichment.md) — Filtering, enriching, and routing distributed traces at ingestion
3. [Sampling-Aware Metrics](markdown/-[OPIPE]-03-sampling-aware-metrics.md) — Extracting accurate metrics from sampled trace data
4. [Cardinality Management](markdown/-[OPIPE]-04-cardinality-management.md) — Controlling dimension explosion across all scopes
5. [Business & Security Event Pipelines](markdown/-[OPIPE]-05-business-and-security-event-pipelines.md) — Processing business transactions and security events at ingestion
6. [Cross-Scope Design Patterns](markdown/-[OPIPE]-06-cross-scope-design-patterns.md) — Correlating logs, spans, metrics, and events across scopes
7. [Pipeline Group Governance](markdown/-[OPIPE]-07-pipeline-group-governance.md) — Central control and team autonomy with pipeline groups: ownership, mandated stages, execution order, routing, limits and gating
99. [Best Practice Summary](markdown/-[OPIPE]-99-best-practice-summary.md) — Consolidated best practices from the OPIPE series

## Usage
1. Choose a format: import JSON from notebooks/, read pdfs/ for print, or view markdown/ for lightweight browsing.
2. Start with OpenPipeline as a Multi-Scope Platform before progressing to advanced patterns.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
