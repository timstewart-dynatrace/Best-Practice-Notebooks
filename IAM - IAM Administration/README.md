# IAM - IAM Administration

Comprehensive guide to identity and access management in Dynatrace.

> **Recommended:** Import the JSON files from notebooks/ into a Dynatrace tenant for the best experience. These notebooks contain interactive DQL queries that execute against your environment's data.

## Structure
- notebooks/ — Dynatrace notebook JSON files
- pdfs/ — Printable versions of each notebook
- markdown/ — Markdown exports of the notebooks

## Notebook Lineup
1. [Governance Foundations](markdown/-[IAM]-01-governance-foundations.md) — IAM governance principles and strategy
2. [SSO & Authentication](markdown/-[IAM]-02-sso-authentication.md) — Single sign-on and authentication setup
3. [Group Architecture](markdown/-[IAM]-03-group-architecture.md) — Designing group structures
4. [Policy Authoring](markdown/-[IAM]-04-policy-authoring.md) — Writing and managing policies
5. [Boundary Design](markdown/-[IAM]-05-boundary-design.md) — Defining access boundaries
6. [User Lifecycle](markdown/-[IAM]-06-user-lifecycle.md) — Managing user provisioning and deprovisioning
7. [Audit & Compliance](markdown/-[IAM]-07-audit-compliance.md) — Auditing access and ensuring compliance using the `dt.system.events` audit trail (`AUDIT_EVENT`), with failed-access, off-hours, and compliance-report queries — and where account-level IAM changes are audited instead
8. [Multi-Environment](markdown/-[IAM]-08-multi-environment.md) — IAM across multiple environments
9. [Troubleshooting](markdown/-[IAM]-09-troubleshooting.md) — Diagnosing and resolving IAM issues
10. [Templated Policy Assignments](markdown/-[IAM]-10-templated-policy-assignments.md) — Policy templates and bulk assignments
11. [[WORKSHOP] Policy Persona Simulation](markdown/-[IAM]-11-[WORKSHOP]-policy-persona.md) — Interactive workshop: designing a persona-based permissions model (personas, IdP alignment, schema/app audits, data boundaries)
12. [API Provisioning & Validation](markdown/-[IAM]-12-api-provisioning-validation.md) — Scripts and DQL for provisioning via Account Management API (curl flavor)
95. [[LAB] Terraform IAM Provisioning](markdown/-[IAM]-95-[LAB]-terraform-iam-provisioning.md) — Hands-on lab: groups, policies, boundaries and bindings as code — a self-contained Terraform module set included in full (live-verified, with expected output)
96. [[LAB] Python IAM Provisioning](markdown/-[IAM]-96-[LAB]-python-iam-provisioning.md) — Hands-on lab: the group → policy → boundary → binding chain step by step against the raw Account Management API in Python (live-verified, with expected output)
97. [Permission Reference](markdown/-[IAM]-97-permission-reference.md) — What each permission in the Standard User policy (plus reader-persona extras) actually grants, with scope, risk, persona matrix and the basis for every claim
99. [Best Practice Summary](markdown/-[IAM]-99-best-practice-summary.md) — Consolidated best practices from the IAM series

## Usage
1. Choose a format: import JSON from notebooks/, read pdfs/ for print, or view markdown/ for lightweight browsing.
2. Start with Governance Foundations for an overview, then explore specific IAM topics as needed.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
