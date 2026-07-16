# AGENTS.md — NR2DT: New Relic to Dynatrace Migration Steps

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

11 notebooks forming a procedural step-by-step runbook (Step 0 prerequisites
through Step 9 cutover, plus a 99 summary) for migrating from New Relic to
Dynatrace. Each step is a runbook with artifacts and exit gates; component
depth (NRQL translation theory, transformer internals) is deliberately
deferred to the companion NRLC series.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| Tenant readiness before migration: bucket strategy, host groups, `dt.security_context`, IAM baseline, access and tooling that must exist first | `-[NR2DT]-00-step-0-prerequisites.md` |
| Inventorying the New Relic account, translation-confidence report, effort estimation | `-[NR2DT]-01-step-1-discover.md` |
| Wave planning, in/out/deferred scope decisions, validation-gate definitions, stakeholder sign-off | `-[NR2DT]-02-step-2-strategize.md` |
| Designing the target architecture: bucket design, host-group taxonomy, IAM model, OpenPipeline layout, Terraform skeleton | `-[NR2DT]-03-step-3-design.md` |
| Running the NRQL→DQL translator (`migrate.py compile`), reviewing MEDIUM confidence, hand-translating LOW confidence | `-[NR2DT]-04-step-4-translate.md` |
| Applying Wave 0 foundations, importing dashboards (Wave 1), migrating alerts with the 1–2 week dual-alert window (Wave 3) | `-[NR2DT]-05-step-5-migrate-dashboards-alerts.md` |
| Migrating synthetic monitors incl. scripted-browser rebuilds (Wave 2), SLOs with 7-day delta validation, and workloads (Wave 4) | `-[NR2DT]-06-step-6-migrate-synthetics-slos-workloads.md` |
| Reconfiguring log forwarders, applying OpenPipeline drop/parse/tag rules, verifying tag enrichment (Wave 5) | `-[NR2DT]-07-step-7-migrate-logs-tags-drops.md` |
| Running the three-tier validation pass (syntax / tenant / behavioral), diffing against live config, quality report | `-[NR2DT]-08-step-8-validate.md` |
| Per-component cutover sequencing, rollback plan, decommissioning New Relic | `-[NR2DT]-09-step-9-cutover-rollback-decommission.md` |
| Best-practice checklist by step, common pitfalls, map of which NRLC deep dive covers what | `-[NR2DT]-99-best-practice-summary.md` |

If more than three rows match, start with
`-[NR2DT]-99-best-practice-summary.md` and follow its pointers. For an
end-to-end migration, start at `-[NR2DT]-00-step-0-prerequisites.md` and
follow the numbered steps in order.

## Related series

- Component-level reference — "how do I translate this NRQL query / dashboard / alert / synthetic / SLO?" — go there instead of this series when the question is about a specific artifact rather than the migration process: `../NRLC - New Relic to Dynatrace Migration Deep Dives/`
- Grail buckets, segments, and security-context design behind Step 3: `../ORGNZ - Organize Data: Buckets, Segments, Security/`
- IAM policy design behind the Step 0/Step 3 access model: `../IAM - IAM Administration/`
- Ongoing OpenPipeline log parsing/routing after Step 7: `../OPLOGS - OpenPipeline Logs/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "NR2DT-05") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
