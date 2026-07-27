# AGENTS.md — M2S: Managed to SaaS Migration

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

11 notebooks on migrating from Dynatrace Managed to Dynatrace SaaS: a
3-phase / 9-step journey (Plan → Upgrade → Run) covering discovery, strategy,
target architecture, the SaaS Upgrade Assistant, Grail bucket planning before
first ingest, OneAgent/ActiveGate/Kubernetes/serverless cutover, integration
reconnection, SaaS-capability adoption, user enablement, and Managed
decommissioning, plus a consolidated best-practice summary and an appendix LAB
on running the migration with the Terraform provider.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| Why move to SaaS, inventorying the Managed environment (configs, Credential Vault, tokens, extensions), what migrates automatically vs manually | `-[M2S]-01-step-1-discover.md` |
| Choosing a migration approach (host-count bands plus cloud-native and serverless sizing axes for estates with few VMs), the 11-step order of operations, risk assessment, success criteria, timeline, the 90/10 rule | `-[M2S]-02-step-2-strategize.md` |
| Target architecture: network connectivity/firewall rules, network zones (`builtin:networkzones.zones`), ActiveGate sizing and placement, SSO/IAM security, HA | `-[M2S]-03-step-3-design.md` |
| Pre-migration readiness: licensing for dual-run, tenant provisioning, SAML SSO setup, parallel ActiveGates, SaaS Upgrade Assistant install (align to the target tenant's major version — no fixed minimum), Grail bucket strategy before first ingest, config freeze, rollback plan | `-[M2S]-04-step-4-prepare.md` |
| Running the migration: dependency-ordered configuration waves via the SaaS Upgrade Assistant, redirecting each workload surface to SaaS (OneAgent via `oneagentctl`, Kubernetes DynaKube recreate, Cloud Run image rebuild, OTLP endpoint repoint), Grail bucket routing before high-volume log redirect, per-wave validation | `-[M2S]-05-step-5-execute.md` |
| Reconnecting integrations: dashboards, alert/notification channels, CI/CD pipelines, ITSM, extensions, API scripts, synthetic monitors | `-[M2S]-06-step-6-integrate.md` |
| Communication plan, persona-based training, documentation updates, support channels, measuring enablement success | `-[M2S]-07-step-7-enable.md` |
| Adopting SaaS-only capabilities post-cutover: Grail, Notebooks, OpenPipeline, Workflows, Dynatrace Assist, platform apps, privacy controls | `-[M2S]-08-step-8-expand.md` |
| Final validation, performance/retention optimization, Davis baseline establishment, stakeholder sign-off, decommissioning the Managed cluster | `-[M2S]-09-step-9-optimize.md` |
| Migrating configuration with **Terraform** instead of (or alongside) the SaaS Upgrade Assistant: bulk vs iterative export (`-migrate`, `-datasources`, duplicate handling), Managed-source URL form / token type / required scopes, what never exports, `.flawed` and `.requires_attention` triage, entity-ID preservation via `oneagentctl`, wave-ordered apply, post-cutover `-import-state` adoption and drift detection | `-[M2S]-95-[LAB]-terraform-migration.md` |
| Consolidated checklist of every M2S best practice across the 9 steps and the 11-step order of operations | `-[M2S]-99-best-practice-summary.md` |

If more than three rows match, start with `-[M2S]-99-best-practice-summary.md`
and follow its pointers.

## Related series

- Migrating between two SaaS tenants instead: `../S2S - SaaS to SaaS Migration/`
- Monaco / Terraform configuration-as-code used in migration tooling: `../AUTOM - Dynatrace Automation/`
- Fresh OneAgent/ActiveGate deployment fundamentals: `../ONBRD - Dynatrace Onboarding/`
- Retiring Management Zones for policy-based access on the SaaS side: `../MZ2POL - Management Zone to Policy Migration/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "M2S-05") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
