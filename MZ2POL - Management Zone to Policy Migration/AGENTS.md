# AGENTS.md — MZ2POL: Management Zone to Policy Migration

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

10 notebooks on migrating from Management Zones to policy-based access
control (IAM policies + boundaries + segments): an SDK analysis tool for the
existing MZ estate, the ABAC model, assessment and planning, policy/boundary
and segment implementation, phased execution with rollback, validation, and
templated policies for bulk migration. For greenfield IAM (no MZs to
migrate), use the IAM series instead.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| Exporting and analyzing existing MZ configurations via the SDK/Settings API, rule-pattern analysis, `dt.security_context` coverage, migration-readiness scoring | `-[MZ2POL]-00-sdk-mz-analysis-tool.md` |
| Why Management Zones are deprecated, MZ-vs-new-model differences, migration urgency and timeline | `-[MZ2POL]-01-introduction-why-migrate.md` |
| The ABAC framework: how policies, boundaries, segments, and buckets relate, permission flow, MZ-concept mapping | `-[MZ2POL]-02-access-control-model.md` |
| MZ inventory and pattern classification, security-context and bucket planning, priority matrix, migration timeline and dependency checklist | `-[MZ2POL]-03-assessment-planning.md` |
| IAM policy statement and boundary syntax, default policies, the two-boundary (Gen3 + Gen2 transitional) pattern, group-policy-boundary structure for SAML/AD | `-[MZ2POL]-04-policies-and-boundaries.md` |
| Segments: creating DQL-powered filters, variables, testing with DQL, mapping MZ rules to segment conditions, sharing and permissions | `-[MZ2POL]-05-segments-implementation.md` |
| Executing the migration phases: security-context assignment, parallel running, cutover, cleanup, rollback procedures | `-[MZ2POL]-06-migration-execution.md` |
| Validating the migration, troubleshooting access issues, diagnostic and health-monitoring queries, ongoing maintenance | `-[MZ2POL]-07-validation-troubleshooting.md` |
| `${bindParam:...}` policy templates for dozens of MZ-based teams, bulk binding via the IAM API, before/after access comparison | `-[MZ2POL]-08-templated-policies-migration.md` |
| Consolidated best-practice checklist across assessment, policy/boundary/segment design, buckets, groups, execution, and validation | `-[MZ2POL]-99-best-practice-summary.md` |

If more than three rows match, start with
`-[MZ2POL]-99-best-practice-summary.md` and follow its pointers.

## Related series

- Greenfield IAM administration (groups, policies, personas — no MZ legacy): `../IAM - IAM Administration/`
- Grail buckets, segments, and security-context strategy in depth: `../ORGNZ - Organize Data: Buckets, Segments, Security/`
- Managing IAM groups/policies/bindings as code (Terraform): `../AUTOM - Dynatrace Automation/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "MZ2POL-04") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
