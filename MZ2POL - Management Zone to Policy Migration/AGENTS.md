# AGENTS.md — MZ2POL: Management Zone Migration: Segments, Policies, Workflows

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

15 numbered notebooks (00–14), a LAB and a summary — 17 in all — on migrating off Management Zones. All three replacements below
read a field on the data, so the series starts from **tagging at source**:
notebook 10 (the model change, enrichment ladder and readiness gate) and
notebook 11 (translating each MZ rule into a target field); notebook 12 is the
IAM reference architecture for the access job; notebook 13 covers Spaces
(upcoming, not yet documented as available); 14 is a disposition workshop
and 95 a hands-on LAB. Read 10 and 11 after
03 and before 04/05/09. Management Zones do three distinct jobs and they
migrate to different places:

- **Access control** (who may read what) becomes IAM policies, boundaries,
  and `dt.security_context` — notebooks 01-04, 06-08 and 12.
- **Filtering** (what a user sees) becomes Segments — notebook 05, which is
  self-contained and readable without the access-control notebooks.
- **Alerting** (who gets paged) becomes problem-triggered workflows filtered
  by affected-entity tags — notebook 09, also self-contained. **Not** Segments:
  there is no segment field on a workflow trigger, and management-zone
  filtering does not carry over — the trigger filters on the problem itself.

The series covers an SDK analysis tool for the existing MZ estate, the ABAC
model, assessment and planning, policy/boundary design, the MZ-to-Segment
migration scenarios, phased execution with rollback, validation, and
templated policies for bulk migration. There is **no automatic MZ-to-Segment
conversion** — every segment is hand-authored. For greenfield IAM (no MZs to
migrate), use the IAM series instead; for segment mechanics in general, use
ORGNZ.


**Reading paths** (route a question to the lane first):

| Path | Use it when | Read in this order |
|---|---|---|
| **Filtering only** | Your zones only scope views and dashboards | 00 → 03 → 10 → 11 → 05 → 06 → 07 |
| **Access control** | Your zones restrict who may read what | 00 → 03 → 10 → 11 → 12 → 04 → 08 → 06 → 07 |
| **Alerting** | Your zones scope alerting profiles | 00 → 03 → 10 → 11 → 09 → 06 |
| **Full migration** | Your zones do several of these jobs | 01 → 02 → 00 → 03 → 14 → 10 → 11 → 12 → 04 → 08 → 05 → 09 → 06 → 07 → 95 → 99 |
| **Decision makers** | You need scope, approach and what is not ready yet | 01 → 02 → 13 → 99 |

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
| Replacing MZ-based **alerting**: why alerting profiles become problem-triggered workflows rather than segments, converting MZ rules into routing tags, mapping `delayInMinutes` onto the trigger's fixed Minimum duration values, connectors with no native equivalent (Trello/VictorOps/xMatters; Opsgenie moves to the JSM connector), problem visibility via `dt.security_context`, and the undocumented MZ-deletion failure mode | `-[MZ2POL]-09-alerting-and-notification-migration.md` |
| Why management zones and auto-tags don't reach Grail data, what replaces auto-tagging (built-in fields, `primary_tags.*`, special fields), why a primary tag is not an IAM boundary (write `dt.security_context` too), the enrichment ladder and precedence (`DT_TAGS` › host › central rule; OpenPipeline tag rules overwrite unless guarded with `isNull`), how built-in propagation replaces the eight MZ propagation flags, silent limits (forward-only, 20-tag cap, 24 h dropdown lag), and DQL that audits classic-tag vs primary-tag parity, per-zone acceptance, coverage by host group, value drift and derived-data reach before anything is built on the tags | `-[MZ2POL]-10-tagging-at-source-foundation.md` |
| Turning an MZ inventory into target fields one rule at a time: `ME` / `DIMENSION` / `SELECTOR` rule anatomy and AND/OR logic, the five-step decision path (built-in field → at source → central enrichment rule → OpenPipeline tag rule → explicit membership tag), property-family catalog with which fields IAM can use (`storage:dt.host_group.id`, `storage:k8s.namespace.name`, `storage:log.source` — no `primary_tags`), where each MZ operator still works (regex, numeric, IP-range and exclusions do not on segments or record-level IAM), pushing `NOT` into the enrichment condition, dimensional and entity-selector rules, six worked translations, and verification DQL | `-[MZ2POL]-11-translating-mz-rules-to-source-attributes.md` |
| Designing IAM to replace MZ-based access: groups × default policies × one boundary per team covering every domain (storage, settings, and `environment:management-zone` while classic pages remain), the documented evaluation rules (union across bindings, conditions apply only to their own service, boundaries computed separately, one unscoped grant wins, `MATCH` for arrays), the zone-only-boundary-on-a-default-policy trap that grants all Grail data, the scoped-settings breaking change (MZ conditions and MZ-derived contexts stop matching), six personas, classic `environment:roles:*` → Grail mapping, transition states, the effective-permissions API, and DQL checks for array-typed and missing security contexts | `-[MZ2POL]-12-iam-reference-architecture.md` |
| Whether to wait for Spaces when leaving management zones: the one published source (`dt.space`, `experimental`, tagged `permission` + `primary-field`), what is not documented (no docs page, no IAM condition, dropped from the SaaS 1.348 notes, absent from the validation tenant), where a record-level permission field would fit beside `dt.security_context` / segments / workflows, no-regret preparation (team values at source, one boundary per team, segments per dimension, ownership tags, an inventory of delegated responsibility), a zero-byte tenant check, and the triggers that would change the guidance | `-[MZ2POL]-13-spaces-what-is-coming.md` |
| Running the decisions of an MZ migration with the people who own them: attendees and what each decides, two zero-byte pre-work queries (classic tag keys; zones vs host groups), six sessions each ending in a table (triage keep/delete, jobs, dimension and allowed-value list, per-rule translation worksheet, group × policy × boundary with every domain covered, alerting routing table and batch plan), stop conditions, and the traps to name out loud (zone-only boundaries, second boundaries, unguarded OpenPipeline backstops, admin testing, waiting for Spaces) | `-[MZ2POL]-14-[WORKSHOP]-zone-disposition.md` |
| A worked end-to-end migration of a fictional estate (eight zones, three teams, shared platform): job classification and MZ2POL-11 rule translation as runnable `data record` DQL, the actual enrichment config (central rules, `DT_TAGS`, namespace annotations, the exclusion pushed into enrichment), readiness checks that catch planted traps (unenriched host and API integration, team-value drift, array security context), a who-sees-what simulation contrasting `MATCH` with `IN`, segments, the replacement workflow, real-tenant checks, and a batch cutover plan | `-[MZ2POL]-95-[LAB]-end-to-end-migration.md` |
| Consolidated checklist across the whole series — assessment, tagging at source, rule translation, policies and boundaries (one boundary covering every domain, `MATCH` on arrays, no zone-only boundaries), settings access, segments, buckets, alerting, execution states and record-level validation, and Spaces — with each row pointing to its source notebook | `-[MZ2POL]-99-best-practice-summary.md` |

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
