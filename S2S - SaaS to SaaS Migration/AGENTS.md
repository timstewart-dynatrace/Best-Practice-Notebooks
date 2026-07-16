# AGENTS.md — S2S: SaaS to SaaS Migration

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

11 notebooks on migrating between Dynatrace SaaS environments (consolidation,
relocation, split, or cloud transformation): the same 3-phase / 9-step journey
as M2S plus a scripts companion — Monaco export/import, Terraform for IAM,
OpenPipeline and bucket recreation, agent cutover, parallel operation, and
source-tenant decommission, capped by a 112-practice summary.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| Why migrate between tenants (consolidation / relocation / split), entity and configuration inventory, what migrates automatically, Monaco-vs-Terraform tool comparison | `-[S2S]-01-step-1-discover.md` |
| Choosing a migration approach, the S2S order of operations, risk assessment, success criteria, timeline, the 90/10 rule | `-[S2S]-02-step-2-strategize.md` |
| Target tenant design: IAM architecture, Grail bucket + retention layout, OpenPipeline layout, cloud-integration mapping, configuration deployment order, entity-ID remapping | `-[S2S]-03-step-3-design.md` |
| Pre-staging: target provisioning, SSO/IAM setup, Monaco bulk export, ActiveGate provisioning, Kubernetes operator prep, config freeze, rollback | `-[S2S]-04-step-4-prepare.md` |
| Executing the cutover: `monaco deploy` workflow, `terraform apply` for IAM, entity-ID remapping, OneAgent reconfiguration, DynaKube operator and ActiveGate migration, wave validation gates | `-[S2S]-05-step-5-execute.md` |
| Reconnecting AWS/Azure/GCP integrations, cloud-transformation scenarios, dashboards, workflows, notification channels, synthetic monitors, extensions | `-[S2S]-06-step-6-integrate.md` |
| OpenPipeline rule migration, Grail bucket configuration, SLO migration, alerting/notification migration, retention optimization | `-[S2S]-07-step-7-expand.md` |
| Parallel operation (dual-tenant cost control), Davis baseline establishment, SLO continuity, stakeholder communication, user training | `-[S2S]-08-step-8-enable.md` |
| Go/no-go checklist, cutover execution, post-cutover validation queries, rollback procedures, source-tenant decommission, lessons learned | `-[S2S]-09-step-9-optimize.md` |
| Ready-to-run Bash and PowerShell scripts: Monaco download/export and SaaS Upgrade Assistant `.tar.gz` + `exportMetadata.json` packaging (script/reference format) | `-[S2S]-10-migration-scripts.md` |
| Consolidated checklist of 112 best practices across the 9 steps | `-[S2S]-99-best-practice-summary.md` |

If more than three rows match, start with `-[S2S]-99-best-practice-summary.md`
and follow its pointers.

## Related series

- Migrating from Dynatrace Managed instead of another SaaS tenant: `../M2S - Managed to SaaS Migration/`
- Monaco / Terraform configuration-as-code depth: `../AUTOM - Dynatrace Automation/`
- Grail bucket, segment, and security-context design on the target: `../ORGNZ - Organize Data: Buckets, Segments, Security/`
- IAM group/policy design beyond the migration window: `../IAM - IAM Administration/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "S2S-05") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
