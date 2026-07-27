# M2S - Managed to SaaS Migration

Guide for migrating from Dynatrace Managed to Dynatrace SaaS.

> **Recommended:** Import the JSON files from notebooks/ into a Dynatrace tenant for the best experience. These notebooks contain interactive DQL queries that execute against your environment's data.

## Structure
- notebooks/ — Dynatrace notebook JSON files
- pdfs/ — Printable versions of each notebook
- markdown/ — Markdown exports of the notebooks

## Notebook Lineup
1. [Step 1 — Discover: Understand SaaS Differences](markdown/-[M2S]-01-step-1-discover.md) — Understanding why you're migrating, the benefits of SaaS, and inventorying your Managed environment
2. [Step 2 — Strategize: Define Your Migration Approach](markdown/-[M2S]-02-step-2-strategize.md) — Planning your migration strategy, phasing, and execution approach
3. [Step 3 — Design: Create Target Architecture](markdown/-[M2S]-03-step-3-design.md) — Designing your target SaaS architecture and configuration
4. [Step 4 — Prepare: Readiness and Pre-Migration](markdown/-[M2S]-04-step-4-prepare.md) — Assessing readiness and preparing for migration
5. [Step 5 — Execute: Migrate Configuration and Agents](markdown/-[M2S]-05-step-5-execute.md) — Migrating configurations and redirecting agents to SaaS
6. [Step 6 — Integrate: Reconnect Integrations](markdown/-[M2S]-06-step-6-integrate.md) — Reconnecting cloud integrations and third-party tools
7. [Step 7 — Enable: User Enablement and Communication](markdown/-[M2S]-07-step-7-enable.md) — User training, support, and operations handover
8. [Step 8 — Expand: Adopt New SaaS Capabilities](markdown/-[M2S]-08-step-8-expand.md) — Discovering and adopting new SaaS-only capabilities
9. [Step 9 — Optimize: Validate, Optimize, and Decommission](markdown/-[M2S]-09-step-9-optimize.md) — Validating the migration, optimizing the environment, and decommissioning Managed
95. [[LAB] Terraform for Managed-to-SaaS Migration](markdown/-[M2S]-95-[LAB]-terraform-migration.md) — Appendix lab: migrating with the Terraform provider — bulk vs iterative export, Managed-source auth and scopes, entity-ID preservation via `oneagentctl`, wave-ordered apply
99. [Best Practice Summary](markdown/-[M2S]-99-best-practice-summary.md) — Definitive reference of all best practices from the M2S series

## Usage
1. Choose a format: import JSON from notebooks/, read pdfs/ for print, or view markdown/ for lightweight browsing.
2. Follow the numbered sequence for a complete migration journey.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
