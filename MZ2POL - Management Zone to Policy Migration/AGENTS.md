# AGENTS.md — MZ2POL: Management Zone to Policy Migration

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

11 notebooks on migrating off Management Zones. Management Zones do three
distinct jobs and they migrate to different places:

- **Access control** (who may read what) becomes IAM policies, boundaries,
  and `dt.security_context` — notebooks 01-04 and 06-08.
- **Filtering** (what a user sees) becomes Segments — notebook 05, which is
  self-contained and readable without the access-control notebooks.
- **Alerting** (who gets paged) becomes problem-triggered workflows filtered
  by affected-entity tags — notebook 09, also self-contained. **Not** Segments:
  there is no segment field on a workflow trigger, and the Management Zone
  filter has no successor inside the alerting model.

The series covers an SDK analysis tool for the existing MZ estate, the ABAC
model, assessment and planning, policy/boundary design, the MZ-to-Segment
migration scenarios, phased execution with rollback, validation, and
templated policies for bulk migration. There is **no automatic MZ-to-Segment
conversion** — every segment is hand-authored. For greenfield IAM (no MZs to
migrate), use the IAM series instead; for segment mechanics in general, use
ORGNZ.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| Exporting and analyzing existing MZ configurations via the SDK/Settings API, rule-pattern analysis, `dt.security_context` coverage, migration-readiness scoring | `-[MZ2POL]-00-sdk-mz-analysis-tool.md` |
| Why Management Zones are deprecated, MZ-vs-new-model differences, migration urgency and timeline | `-[MZ2POL]-01-introduction-why-migrate.md` |
| The ABAC framework: how policies, boundaries, segments, and buckets relate, permission flow, MZ-concept mapping | `-[MZ2POL]-02-access-control-model.md` |
| MZ inventory and pattern classification, security-context and bucket planning, priority matrix, migration timeline and dependency checklist | `-[MZ2POL]-03-assessment-planning.md` |
| IAM policy statement and boundary syntax, default policies, the two-boundary (Gen3 + Gen2 transitional) pattern, group-policy-boundary structure for SAML/AD | `-[MZ2POL]-04-policies-and-boundaries.md` |
| Replacing MZ-based **filtering** with segments: the eight documented migration scenarios (host group, Kubernetes, host tags, the `Segment`-tag fallback, cloud-native), conversion blockers (no exclusions, derived-data tag propagation, entity operator limits, Davis problem includes), MZ-rule-to-filter mapping, DQL validation, classic-app coexistence, and the decision test that routes between the three jobs a Management Zone does: if removing it would let someone see data they are not allowed to see it is access control (→ MZ2POL-04); if it would only make dashboards noisy it is filtering (this notebook); if it decides who gets paged it is alerting (→ MZ2POL-09). Most estates are far more filtering than teams expect — MZ2POL-00 and MZ2POL-03 classify the estate | `-[MZ2POL]-05-segments-implementation.md` |
| Executing the migration phases: security-context assignment, parallel running, cutover, cleanup, rollback procedures | `-[MZ2POL]-06-migration-execution.md` |
| Validating the migration, troubleshooting access issues, diagnostic and health-monitoring queries, ongoing maintenance | `-[MZ2POL]-07-validation-troubleshooting.md` |
| `${bindParam:...}` policy templates for dozens of MZ-based teams, bulk binding via the IAM API, before/after access comparison | `-[MZ2POL]-08-templated-policies-migration.md` |
| Replacing MZ-based **alerting**: why alerting profiles become problem-triggered workflows rather than segments, converting MZ rules into routing tags, the `delayInMinutes` capability gap and why delay-dependent profiles migrate last, connectors with no native equivalent (Opsgenie/Trello/VictorOps/xMatters), problem visibility via `dt.security_context`, and the undocumented MZ-deletion failure mode | `-[MZ2POL]-09-alerting-and-notification-migration.md` |
| Consolidated best-practice checklist across assessment, policy/boundary/segment design, buckets, groups, execution, and validation | `-[MZ2POL]-99-best-practice-summary.md` |

If more than three rows match, start with
`-[MZ2POL]-99-best-practice-summary.md` and follow its pointers.

## Related series

- Greenfield IAM administration (groups, policies, personas — no MZ legacy): `../IAM - IAM Administration/`
- Grail buckets, segments, and security-context strategy in depth: `../ORGNZ - Organize Data: Buckets, Segments, Security/`
- Workflow triggers, routing patterns, and notification destinations: `../WFLOW - Workflows and Alert Notifications/`
- Alerting architecture, detection choice, and routing cost: `../ALERT - Alerting Strategy and Design/`
- Managing IAM groups/policies/bindings as code (Terraform): `../AUTOM - Dynatrace Automation/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "MZ2POL-04") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
