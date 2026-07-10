# FAQ - Frequently Asked Questions

Frequently asked questions and answers across the Dynatrace Best-Practice Notebooks series. Each entry consolidates guidance, trade-offs, and design principles for a single recurring question.

> **Recommended:** Import the JSON files from notebooks/ into a Dynatrace tenant for the best experience. These notebooks contain interactive DQL queries that execute against your environment's data.

## Structure
- notebooks/ — Dynatrace notebook JSON files
- pdfs/ — Printable versions of each notebook
- markdown/ — Markdown exports of the notebooks

## FAQ Entries
1. [Why you need a good Host Group naming strategy](markdown/-[FAQ]-01-host-group-naming-strategy.md) — How host group naming influences access control, alerting, automation, and tenant maintainability
2. [Tagging — Sources, Standards, and Strategy](markdown/-[FAQ]-02-tagging-sources-standards-strategy.md) — The four tag sources, primary tags vs. ordinary tags, and the standards that turn tag sprawl into a managed asset
3. [OneAgent vs OpenTelemetry — A Decision Framework](markdown/-[FAQ]-03-oneagent-vs-otel-decision-framework.md) — Cross-runtime decision framework for convert vs. layer vs. leave-alone vs. greenfield, covering Java, .NET, Node.js, Python, Go, PHP, Ruby, with async context propagation edge cases and JVM dynamic-attach guidance
4. [Managing OneAgent Updates on SaaS](markdown/-[FAQ]-04-managing-oneagent-updates-saas.md) — Update mechanism, the three modes, tenant/host-group/host precedence, update vs. maintenance windows, the DynaKube `autoUpdate` deprecation, validation, rollback, and common pitfalls
5. [Managing ActiveGate Updates on SaaS](markdown/-[FAQ]-05-managing-activegate-updates-saas.md) — Auto-update vs manual decision, ActiveGates-before-OneAgents sequencing, HA-pair rolling updates, role-specific considerations (routing, synthetic, EF 2.0, cloud), validation, rollback, and common pitfalls
6. [Can We Trust Davis AI? A Risk and Controls Walkthrough](markdown/-[FAQ]-06-can-we-trust-davis-ai.md) — Four-surface taxonomy (Causal, Predictive, Generative/CoPilot, AI Observability) with data residency, training boundary, hallucination controls, autonomy limits, audit trail, IAM, compliance posture, and a decision framework for when to lean in vs hold back
7. [How Do I Set Up a Launcher Page? (Default + Persona-Based)](markdown/-[FAQ]-07-launcher-page-setup.md) — Default launcher configuration plus persona-based launcher patterns for tailoring the entry experience by role
8. [How Does OneAgent Decide Which Logs to Collect?](markdown/-[FAQ]-08-oneagent-log-autodiscovery.md) — Log auto-discovery rules, what gets picked up automatically vs what needs a custom log source, host scanning, include/exclude patterns, and verification checklist
9. [When Should I Query a Metric Instead of Raw Logs?](markdown/-[FAQ]-09-metrics-instead-of-log-queries.md) — Decision framework for replacing recurring log aggregations with pre-extracted metrics; covers Grail query economics, OOTB metrics, cardinality, and extraction trade-offs
10. [How Do I Size and Scale ActiveGates?](markdown/-[FAQ]-10-activegate-sizing-and-scaling.md) — Capability-to-sizing-dimension map, host-based and containerized K8s sizing tables, synthetic node sizing, HA/network-zone survivor-capacity math, and saturation detection via `dt.sfm.active_gate.*` Grail metrics
11. [How Do Metrics Work in Dynatrace?](markdown/-[FAQ]-11-how-metrics-work.md) — End-to-end metric mechanics: data-point model (gauge summaries + count deltas), Classic vs Grail dual-write backends, all ingest paths (OneAgent, API v2, OTLP, OpenPipeline), storage/rollup/retention, cardinality enforcement, timeseries querying, and DPS billing
12. [Coming from Another Tool — How Partial Enablement Handicaps Your Dynatrace Coverage](markdown/-[FAQ]-12-cost-of-coverage-gaps.md) — Per-capability habit-to-handicap matrix for migration customers who carry partial-enablement patterns from their previous tool; what each gap costs, when reduced coverage is right, and objection handling

## Usage
1. Choose a format: import JSON from notebooks/, read pdfs/ for print, or view markdown/ for lightweight browsing.
2. FAQ entries are independent — read only the entries relevant to your current question.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
