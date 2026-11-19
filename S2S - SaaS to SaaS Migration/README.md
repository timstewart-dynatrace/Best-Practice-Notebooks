# S2S - SaaS to SaaS Migration

Comprehensive guide to migrating between Dynatrace SaaS environments, covering every phase from readiness assessment through post-migration optimization.

> **Recommended:** For the best experience, import the notebooks into your Dynatrace tenant. Each one is available as a Dynatrace notebook (JSON) file, and its DQL queries run against your own environment's data.

## Structure
- notebooks/ — Dynatrace notebook JSON files
- pdfs/ — Printable versions of each notebook
- markdown/ — Markdown exports of the notebooks

## Notebook Lineup
1. [Step 1 — Discover: Migration Scenarios and Inventory](markdown/-[S2S]-01-step-1-discover.md) — Understanding why you're migrating, inventorying your source environment, and identifying what migrates automatically
2. [Step 2 — Strategize: Define Your Migration Approach](markdown/-[S2S]-02-step-2-strategize.md) — Planning your migration strategy, phasing, and execution approach
3. [Step 3 — Design: Target Tenant Architecture](markdown/-[S2S]-03-step-3-design.md) — Designing your target SaaS architecture and tenant structure
4. [Step 4 — Prepare: Export and Pre-Stage](markdown/-[S2S]-04-step-4-prepare.md) — Exporting configurations and pre-staging for migration
5. [Step 5 — Execute: Configuration Import and Agent Cutover](markdown/-[S2S]-05-step-5-execute.md) — Importing configuration and redirecting agents to the target environment
6. [Step 6 — Integrate: Cloud, Dashboards, and Workflows](markdown/-[S2S]-06-step-6-integrate.md) — Migrating cloud integrations, dashboards, and workflows
7. [Step 7 — Expand: OpenPipeline, SLOs, and Alerting](markdown/-[S2S]-07-step-7-expand.md) — Configuring data pipelines, SLOs, and alerting rules
8. [Step 8 — Enable: Parallel Operation and Stakeholder Handover](markdown/-[S2S]-08-step-8-enable.md) — Running source and target in parallel and handing over to operations
9. [Step 9 — Optimize: Cutover Validation and Decommission](markdown/-[S2S]-09-step-9-optimize.md) — Validating the migration, optimizing the target, and decommissioning source
10. [Migration Scripts](markdown/-[S2S]-10-migration-scripts.md) — Reusable Bash and PowerShell scripts for Monaco export with a short-lived token, staged for direct `monaco deploy` to the target
94. [[LAB] Retiring AWS for Azure](markdown/-[S2S]-94-[LAB]-aws-to-azure-cloud-retirement.md) — Appendix lab: retiring AWS for Azure — moving the environment to an Azure-hosted cluster (standard SaaS region vs Azure Native) while workloads move in waves; dual-cloud connections, log/event cut-over, Smartscape validation queries, AWS and source decommission
99. [Best Practice Summary](markdown/-[S2S]-99-best-practice-summary.md) — Comprehensive reference of all best practices from the S2S series

## Usage
1. Open a notebook from the lineup below. Each one is also available as a Dynatrace notebook (JSON) to import into your tenant, so its queries run against your own data, and as a PDF for print.
2. Start with Migration Scenarios and Readiness Assessment for an overview, then follow the numbered sequence through each migration phase.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
