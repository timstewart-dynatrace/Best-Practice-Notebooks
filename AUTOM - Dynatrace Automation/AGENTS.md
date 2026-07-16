# AGENTS.md — AUTOM: Dynatrace Automation

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

14 notebooks on automating Dynatrace configuration and operations: the tool
landscape (Settings API, Monaco, Terraform, Workflows, SDKs), CI/CD and GitOps
pipelines, tenant-to-tenant migration, a Terraform GitOps bootstrap recipe,
four hands-on labs, and a best-practice checklist.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| Which automation tool to pick (Monaco vs Terraform vs API vs SDK), decision framework, zero-to-GitOps setup path, token-type overview | `-[AUTOM]-01-automation-landscape.md` |
| Settings 2.0 API — schemas, objects CRUD via `/api/v2/settings`, the schema catalog by domain (`builtin:*` IDs), classic Config API contrast | `-[AUTOM]-02-settings-api.md` |
| Monaco CLI — manifest.yaml, `monaco download`/`deploy`, YAML config structure, multi-environment projects | `-[AUTOM]-03-monaco.md` |
| Terraform provider — API Token vs Platform Token vs OAuth client auth, `DYNATRACE_HTTP_OAUTH_PREFERENCE`, resource catalog by domain, `-export -ref -id`, state leakage, mint-out-of-band | `-[AUTOM]-04-terraform.md` |
| Workflows as an automation option — components, event-driven actions, auto-remediation patterns (overview; depth lives in the WFLOW series) | `-[AUTOM]-05-workflows.md` |
| TypeScript (`@dynatrace-sdk`) and Python SDKs, programmatic queries/config, MCP server integration | `-[AUTOM]-06-sdks.md` |
| CI/CD pipelines — GitHub Actions, GitLab, Bitbucket, Bamboo, Azure DevOps, ArgoCD/Flux, Operator GitOps, Vault, drift detection, ephemeral Platform Token minting | `-[AUTOM]-07-cicd-integration.md` |
| Moving config between tenants — Monaco/Terraform export-import, Managed→SaaS, consolidation, validation | `-[AUTOM]-08-migration-automation.md` |
| Standing up a Terraform GitOps shop — repo layout, state backends (S3/GCS/Azure/HCP), promotion, `prevent_destroy` lifecycle protections, secrets end-to-end, team onboarding | `-[AUTOM]-09-terraform-gitops-setup-recipe.md` |
| Hands-on lab: account-level IAM via Terraform — OAuth client scopes, `dynatrace_iam_group`/`_policy`/`_bindings_v2`, permission DSL discovery, bindings re-assign-all gotcha | `-[AUTOM]-95-[LAB]-terraform-iam.md` |
| Hands-on lab: GitHub Actions pipeline — OIDC → AWS IAM → Vault → Platform Token, plan-comment PRs, environment-gated apply | `-[AUTOM]-96-[LAB]-cicd-github-actions.md` |
| Hands-on lab: Monaco walkthrough — install, token, manifest, download, deploy, delete, CI/CD | `-[AUTOM]-97-[LAB]-monaco.md` |
| Hands-on lab: Terraform provider basics — first resource, Settings 2.0, import, variables, state, drift | `-[AUTOM]-98-[LAB]-terraform.md` |
| Consolidated checklist: token management, per-tool best practices, governance, tool selection | `-[AUTOM]-99-best-practice-summary.md` |

If more than three rows match, start with `-[AUTOM]-99-best-practice-summary.md`
and follow its pointers.

## Related series

- Workflow authoring depth (triggers, notifications, JavaScript actions): `../WFLOW - Workflows and Alert Notifications/`
- IAM concepts behind the Terraform IAM lab (policies, groups, boundaries): `../IAM - IAM Administration/`
- GitOps for the DynaKube CR and Dynatrace Operator specifically: `../K8S - Kubernetes Monitoring/`
- SLOs as code (`dynatrace_slo_v2`, Monaco for SLOs): `../SLO - Service Level Objectives/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "AUTOM-04") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
