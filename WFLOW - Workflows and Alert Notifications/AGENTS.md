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
| What Workflows are, execution model, permissions (`automation:workflows:*`), building/running a first workflow, execution history | `-[WFLOW]-01-fundamentals.md` |
| When a workflow fires — Detected Problem trigger, metric-event trigger, cron schedules, on-demand, custom/business event triggers, the Davis problem payload (`event()` fields) | `-[WFLOW]-02-triggers.md` |
| Sending Slack, Microsoft Teams, or email notifications — connections, notification tasks, message formatting | `-[WFLOW]-03-notification-basics.md` |
| Conditional routing — by severity, team/service ownership (Smartscape `ownership.team`), time of day, escalation, multi-channel strategy | `-[WFLOW]-04-notification-routing.md` |
| PagerDuty and ServiceNow integration — setup, workflow tasks, bi-directional sync, incident lifecycle | `-[WFLOW]-05-incident-management.md` |
| Rich message templates — Jinja2 expressions, Slack Block Kit, Teams Adaptive Cards, data enrichment | `-[WFLOW]-06-custom-templates.md` |
| Auto-remediation — safety guardrails, Kubernetes/cloud resource remediation, runbooks, approval workflows | `-[WFLOW]-07-remediation.md` |
| Custom code — JavaScript actions, `fetch()` HTTP requests, SDK usage, error handling, `result()`/`withItems` data passing, task timeouts | `-[WFLOW]-08-javascript-http.md` |
| Production hardening — secrets/Credential Vault, third-party connections, RBAC, workflow observability, alerting on workflows, change management | `-[WFLOW]-09-governance.md` |
| Hands-on lab: static egress IP for IP-allow-listed connector targets — EdgeConnect on AWS ECS Fargate behind a NAT Gateway (Snowflake worked example) | `-[WFLOW]-94-[LAB]-edgeconnect-static-egress-snowflake.md` |
| Hands-on lab: CMDB-driven bulk host tagging — lookup tables, `set hostTag` via the OneAgent Remote Configuration Management API, dry-run/idempotent pattern | `-[WFLOW]-95-[LAB]-cmdb-host-tag-enrichment.md` |
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
