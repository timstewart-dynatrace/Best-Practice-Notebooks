# AGENTS.md — WFLOW: Workflows and Alert Notifications

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

12 notebooks on the Dynatrace Workflows (AutomationEngine) app: triggers,
notification channels and routing, ITSM integration, templates, remediation,
JavaScript/HTTP actions, production governance, two hands-on labs, and a
best-practice checklist. This series owns the notification *plumbing* — how a
problem reaches a channel — not detection or alerting strategy.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| What Workflows are, execution model, simple vs standard workflows and workflow-hour billing, live vs draft (only deployed workflows trigger), the actor / service user, permissions (`automation:workflows:*`), documented limits, building/running a first workflow, execution history | `-[WFLOW]-01-fundamentals.md` |
| When a workflow fires — Detected Problem trigger, Davis event trigger (per-alert; thresholds live in the detector), cron schedules, on-demand, custom/business event triggers, the Davis problem payload (`event()` fields — `event.name`, `event.status`, `event.category`, `event.severity`, `{{ problem_link() }}`, and the old-template field mapping); and how the entity fields reach that payload — `affected_entity_ids`, `root_cause_entity_id` and `smartscape.affected_entities` come from the Davis events Dynatrace grouped, so a problem built from mis-attributed events arrives with them populated but wrong; custom *Davis* events go to `/api/v2/events/ingest` with an `entitySelector`, which is a different API from custom events | `-[WFLOW]-02-triggers.md` |
| Sending Slack (bot token, External requests allowlist), Microsoft Teams (Power Automate Workflows webhooks — O365 connectors retired May 2026), or email notifications — connections, notification tasks, message formatting, severity names | `-[WFLOW]-03-notification-basics.md` |
| Conditional routing — per-task conditions (predecessor states, one custom expression, Else Skip/Stop), severity levels 1 Critical / 2 Major / 3 Minor and the trigger Severity filter, team/service ownership (Ownership *Get owners* action, `primary_tags.team`, entity tags), the `event.severity` 1–5 scale and the SaaS 1.348 no-default change, time of day with an explicit time zone, escalation (still-active check or a Minimum duration second workflow), multi-channel strategy | `-[WFLOW]-04-notification-routing.md` |
| PagerDuty and ServiceNow integration — setup, workflow tasks, search-before-create de-duplication, resolve by incident number with resolution notes/code, problem-comment link-back, incident lifecycle, production hardening (retry/backoff by failure class, dead-letter queue, link-back pattern, monitoring the integration itself) | `-[WFLOW]-05-incident-management.md` |
| Rich message templates — Jinja2 expressions, Slack Block Kit, Teams Adaptive Cards, data enrichment | `-[WFLOW]-06-custom-templates.md` |
| Auto-remediation — root-cause gating (most problems name no root cause), a fail-closed non-production gate on primary tags, rate-limit guardrails, Kubernetes and AWS connectors, runbooks, approval workflows | `-[WFLOW]-07-remediation.md` |
| Custom code — JavaScript actions, `fetch()` HTTP requests with the External requests allowlist and EdgeConnect for private hosts, SDK usage (`monitoredEntitiesClient`, `settingsObjectsClient`), error handling, `result()`/`withItems` data passing, task timeouts, workflow-level failure branching (predecessor-state conditions, Skip/Stop else-behavior, custom Jinja conditions) | `-[WFLOW]-08-javascript-http.md` |
| Production hardening — secrets/Credential Vault, connections as settings objects, RBAC (`automation:workflows:*`, `automation:workflow-type`), service-user actors and Workflow admin mode, workflow observability with deduplicated execution history, alerting on workflows, change management and version-history restore | `-[WFLOW]-09-governance.md` |
| Hands-on lab: static egress IP for IP-allow-listed connector targets — EdgeConnect on AWS ECS Fargate behind a NAT Gateway (Snowflake worked example) | `-[WFLOW]-94-[LAB]-edgeconnect-static-egress-snowflake.md` |
| Hands-on lab: CMDB-driven bulk host tagging — lookup tables, `set hostTag` via the OneAgent Remote Configuration Management API (platform auth as the actor first, classic token as the classic/hybrid alternative), `failedEntities` checks, dry-run/idempotent pattern | `-[WFLOW]-95-[LAB]-cmdb-host-tag-enrichment.md` |
| Consolidated checklist: design, triggers, connections, channels, routing, remediation, security, operations | `-[WFLOW]-99-best-practice-summary.md` |

If more than three rows match, start with `-[WFLOW]-99-best-practice-summary.md`
and follow its pointers.

## Related series

- Overall alerting strategy — what to alert on, end-to-end design, ServiceNow maturity ladder: go to `../ALERT - Alerting Strategy and Design/` instead when the question is *design*, not plumbing.
- What fires the problem in the first place — anomaly detectors, Davis root cause: go to `../AIOPS - Dynatrace Intelligence/` instead for detection mechanics.
- Reliability targets — error budgets and the burn-rate alerts you route here: go to `../SLO - Service Level Objectives/` instead for defining them.
- Provisioning workflows as code and the wider automation toolchain: `../AUTOM - Dynatrace Automation/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "WFLOW-04") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
