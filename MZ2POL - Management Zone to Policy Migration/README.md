# MZ2POL - Management Zone to Policy Migration

Tools and guidance for migrating off Management Zones in Dynatrace.

Management Zones do three different jobs, and they migrate to three different places. **Access control** — who is allowed to read what — becomes IAM policies and boundaries (notebooks 01–04, 06–08). **Filtering** — what a user sees in apps and dashboards — becomes Segments (notebook 05). **Alerting** — who gets paged — becomes problem-triggered workflows (notebook 09), *not* Segments. Most estates use Management Zones more for filtering than for permissions, and notebooks 05 and 09 each read on their own if that is your situation.

There is no automatic conversion in either direction: every policy and every segment is authored by hand. Notebook 00 inventories and classifies the estate so you know how much work each zone represents.

> **Recommended:** Import the JSON files from notebooks/ into a Dynatrace tenant for the best experience. These notebooks contain interactive DQL queries that execute against your environment's data.

## Structure
- notebooks/ — Dynatrace notebook JSON files
- pdfs/ — Printable versions of each notebook
- markdown/ — Markdown exports of the notebooks

## Notebook Lineup
0. [SDK MZ Analysis Tool](markdown/-[MZ2POL]-00-sdk-mz-analysis-tool.md) — Analyze existing management zones
1. [Introduction: Why Migrate](markdown/-[MZ2POL]-01-introduction-why-migrate.md) — Benefits and overview
2. [Access Control Model](markdown/-[MZ2POL]-02-access-control-model.md) — Policies and access control concepts
3. [Assessment & Planning](markdown/-[MZ2POL]-03-assessment-planning.md) — Migration assessment and planning
4. [Policies and Boundaries](markdown/-[MZ2POL]-04-policies-and-boundaries.md) — Defining policies and boundaries
5. [Migrating Management Zone Filtering to Segments](markdown/-[MZ2POL]-05-segments-implementation.md) — Migration scenarios, conversion blockers, and rule mapping
6. [Migration Execution](markdown/-[MZ2POL]-06-migration-execution.md) — Executing the migration
7. [Validation & Troubleshooting](markdown/-[MZ2POL]-07-validation-troubleshooting.md) — Validating and resolving issues
8. [Templated Policies Migration](markdown/-[MZ2POL]-08-templated-policies-migration.md) — Policy templates for bulk MZ migration
9. [Alerting and Notification Migration](markdown/-[MZ2POL]-09-alerting-and-notification-migration.md) — MZ-scoped alerting profiles become problem-triggered workflows; the tag prerequisite, capability regressions, and the deletion test
99. [Best Practice Summary](markdown/-[MZ2POL]-99-best-practice-summary.md) — Consolidated best practices from the MZ2POL series

## Usage
1. Choose a format: import JSON from notebooks/, read pdfs/ for print, or view markdown/ for lightweight browsing.
2. Use the SDK analysis tool first, then follow the numbered migration path.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
