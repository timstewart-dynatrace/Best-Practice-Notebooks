# SLO - Service Level Objectives

Defining and operating Service Level Objectives in Dynatrace — SLIs, error budgets, composition, burn-rate alerting, SLOs as code, Site Reliability Guardians for point-in-time deployment gating, and upgrading SLO Classic definitions to the SLO app. Every DQL query in the series was validated against a live tenant.

> **Recommended:** Import the JSON files from notebooks/ into a Dynatrace tenant for the best experience. These notebooks contain interactive DQL queries that execute against your environment's data.

## Structure
- notebooks/ — Dynatrace notebook JSON files
- pdfs/ — Printable versions of each notebook
- markdown/ — Markdown exports of the notebooks

## Notebook Lineup
1. [SLO and SLI Fundamentals](markdown/-[SLO]-01-fundamentals.md) — The three building blocks (SLI, SLO, error budget), the Dynatrace SLO model, and choosing your first SLOs
2. [Defining SLIs](markdown/-[SLO]-02-defining-slis.md) — The good ÷ total ratio as DQL: availability, latency, error-rate, and custom business SLIs, each with the SLO-ready `sli` array form the SLO app requires
3. [Composition and Error Budgets](markdown/-[SLO]-03-composition-and-error-budgets.md) — Error-budget math and burn rate, composite and weighted-global SLOs, and rolling vs calendar windows
4. [SLO Alerting](markdown/-[SLO]-04-alerting.md) — Burn-rate alerting over threshold-on-SLI: fast-burn and slow-burn multiwindow alerts, surfacing, routing, and avoiding fatigue
5. [SLOs as Code](markdown/-[SLO]-05-slos-as-code.md) — Promoting SLOs into version control: the `dynatrace_platform_slo` Terraform resource and SLO Service Public API for the modern app, the classic `dynatrace_slo_v2` / `builtin:monitoring.slo` path it replaces, Monaco `slo-v2`, and the export-known-good API pattern
6. [Site Reliability Guardians](markdown/-[SLO]-06-site-reliability-guardians.md) — Point-in-time deployment validation gates: guardian types, DQL objectives (reference-SLO is Classic-only and deprecated), the Pass/Warning/Fail/Error/Info result model, creating and triggering a guardian, provisioning as code (Terraform/Monaco), CI/CD pipeline integration, and how guardians complement SLOs
7. [Upgrading Classic SLOs](markdown/-[SLO]-07-upgrading-classic-slos.md) — Moving SLO Classic definitions to the Service-Level Objectives app: inventory, translating the SLI to DQL, proving the new query measures the same services, segments vs access, rebuilding alerts and guardian objectives, Terraform, and the parallel run before retirement

## Usage
1. Open a notebook from the lineup below. Each one is also available as a Dynatrace notebook (JSON) to import into your tenant, so its queries run against your own data, and as a PDF for print.
2. Start with the Fundamentals for orientation, then follow the numbered sequence — each notebook builds on the previous.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
