# AGENTS.md — M2S: Managed to SaaS Migration

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

10 notebooks on migrating from Dynatrace Managed to Dynatrace SaaS: a
3-phase / 9-step journey (Plan → Upgrade → Run) covering discovery, strategy,
target architecture, the SaaS Upgrade Assistant, OneAgent/ActiveGate cutover,
integration reconnection, SaaS-capability adoption, user enablement, and
Managed decommissioning, plus a consolidated best-practice summary.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| Why move to SaaS, inventorying the Managed environment (configs, Credential Vault, tokens, extensions), what migrates automatically vs manually | `-[M2S]-01-step-1-discover.md` |
| Choosing a migration approach, the 11-step order of operations, risk assessment, success criteria, timeline, the 90/10 rule | `-[M2S]-02-step-2-strategize.md` |
| Target architecture: network connectivity/firewall rules, network zones (`builtin:networkzones.zones`), ActiveGate sizing and placement, SSO/IAM security, HA | `-[M2S]-03-step-3-design.md` |
| Pre-migration readiness: licensing for dual-run, tenant provisioning, SAML SSO setup, parallel ActiveGates, SaaS Upgrade Assistant install (Managed v1.294+), config freeze, rollback plan | `-[M2S]-04-step-4-prepare.md` |
| Running the migration: dependency-ordered configuration waves via the SaaS Upgrade Assistant, redirecting OneAgents to SaaS, per-wave validation | `-[M2S]-05-step-5-execute.md` |
| Reconnecting integrations: dashboards, alert/notification channels, CI/CD pipelines, ITSM, extensions, API scripts, synthetic monitors | `-[M2S]-06-step-6-integrate.md` |
| Adopting SaaS-only capabilities post-cutover: Grail, Notebooks, OpenPipeline, Workflows, Dynatrace Assist, platform apps, privacy controls | `-[M2S]-07-step-7-expand.md` |
| Communication plan, persona-based training, documentation updates, support channels, measuring enablement success | `-[M2S]-08-step-8-enable.md` |
| Final validation, performance/retention optimization, Davis baseline establishment, stakeholder sign-off, decommissioning the Managed cluster | `-[M2S]-09-step-9-optimize.md` |
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
