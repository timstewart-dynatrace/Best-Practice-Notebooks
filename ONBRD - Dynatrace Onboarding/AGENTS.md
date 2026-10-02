# AGENTS.md — ONBRD: Dynatrace Onboarding

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

12 notebooks taking a new Dynatrace tenant from first login to working
dashboards: an architect's sequence runbook, IAM/authentication, ActiveGate and
OneAgent deployment, cloud integrations, environment organization, the data
model, first DQL queries, alerting, dashboards, and a best-practice summary
with 2026 recommended defaults.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| What order to onboard in: phases, dependencies, phase gates, parallel-able steps, skip rules | `-[ONBRD]-00-architect-sequence.md` |
| First login, tenant ID/URLs, navigating the UI (Logs, Infrastructure & Operations apps), platform tokens (`dt0s16`) vs classic access tokens (classic/hybrid environments only) | `-[ONBRD]-01-first-steps.md` |
| SAML/SCIM setup (fallback account, DNS domain verification, signed message, no assertion encryption), user groups and the Admin/Pro/Standard User policies, `storage:buckets:read` with table permissions, platform token vs OAuth vs classic token, why IAM comes before agents | `-[ONBRD]-02-iam-and-authentication.md` |
| When/where you need ActiveGate, how many, token scopes (platform `fleet-management:activegates:download` or classic), install on the `.live.` host (incl. Kubernetes sizing: separate monitoring and routing ActiveGates), `Dynatrace Gateway` service, AG troubleshooting | `-[ONBRD]-03-deploying-activegate.md` |
| Connecting AWS/Azure/GCP (classic GCP = `dynatrace-gcp-monitor`, no ActiveGate on SaaS), Clouds app, AWS IAM policy, Extensions (remote on ActiveGate, local on OneAgent), Dynatrace Hub, SaaS integrations | `-[ONBRD]-04-cloud-saas-integrations.md` |
| OneAgent rollout strategy, installer tokens (platform `fleet-management:oneagents:download` or classic `InstallerDownload`) on the `.live.` host, install methods, K8s deployment (`v1beta6` DynaKube, image mirroring), hosts not reporting | `-[ONBRD]-05-deploying-oneagent.md` |
| Tags, segments, naming conventions, primary tags/fields via `oneagentctl --set-host-tag`, cloud tag fields (`aws.tags.<key>` etc.) reaching signals only through enrichment, querying by tags | `-[ONBRD]-06-organizing-your-environment.md` |
| The Grail data model and default retention, data objects (`security.events`, `user.sessions`), Smartscape topology (`FRONTEND`, `PROCESS`), log-source health events, what OneAgent discovered | `-[ONBRD]-07-understanding-your-data.md` |
| DQL fundamentals: pipeline model, fetch/filter/fields/summarize/sort, time ranges, error logs via `status == "ERROR"`, common patterns | `-[ONBRD]-08-your-first-queries.md` |
| Davis problem detection, the Problem trigger (states, wait for root cause, minimum duration), simple vs standard workflows, Slack/PagerDuty connections, routing alerts to teams, custom metric alerts | `-[ONBRD]-09-setting-up-alerts.md` |
| Dashboards vs notebooks, Dashboards-app tile and visualization types, service tiles from `dt.service.request.*` metrics, variables, sharing and permissions | `-[ONBRD]-10-building-dashboards.md` |
| Recommended 2026 defaults, key decision matrix, validation DQL, onboarding anti-patterns, cross-series map | `-[ONBRD]-99-best-practice-summary.md` |

If more than three rows match, start with `-[ONBRD]-99-best-practice-summary.md`
and follow its pointers.

## Related series

- Deep IAM administration after the onboarding basics: `../IAM - IAM Administration/`
- Buckets, segments, and security context in depth: `../ORGNZ - Organize Data: Buckets, Segments, Security/`
- Kubernetes Operator/DynaKube deployment depth: `../K8S - Kubernetes Monitoring/`
- Post-onboarding maturity and adoption program: `../ADOPT - Observability Adoption & Maturity/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "ONBRD-03") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
