# AGENTS.md — APPSEC: Application Security

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

10 notebooks on Dynatrace Application Security in Gen3 SaaS: the three pillars
— Runtime Vulnerability Analytics (RVA), Runtime Application Protection (RAP),
and Security Posture Management (SPM) — plus Kubernetes/container security,
Security Investigator and Davis CoPilot, workflow remediation, the dual-surface
IAM model, and governance dashboards. All findings live in Grail
`security.events` and the `vulnerability-service`.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| What RVA/RAP/SPM are, Grail `security.events` vs `vulnerability-service`, deployment-mode coverage (Full-Stack vs Infrastructure/Discovery), Dynatrace Security Score (DSS) as a per-vulnerability score | `-[APPSEC]-01-fundamentals.md` |
| Third-party library CVEs, reachability/exposure signals, triage DQL over `security.events` (dedup to the latest snapshot — current open vulnerabilities), acknowledge/mute/exception lifecycle | `-[APPSEC]-02-runtime-vulnerability-analytics.md` |
| First-party code vulnerabilities, supported runtimes (Java/.NET/Go), `vulnerability.stack == "CODE"` query, vulnerable-function information, process-group scoping | `-[APPSEC]-03-code-level-vulnerability-analytics.md` |
| Attack detection and blocking (RAP): SQL/command injection, SSRF, JNDI, `DETECTION_FINDING` (RAP) queries, Monitor→Block promotion per technology and custom rule | `-[APPSEC]-04-runtime-application-protection.md` |
| Compliance standards (CIS, PCI DSS, NIST, DORA, …), KSPM/CSPM/VSPM scope, currently-failing `COMPLIANCE_FINDING` query, finding lifecycle | `-[APPSEC]-05-security-posture-management.md` |
| Kubernetes/container security: DynaKube `applicationMonitoring`, image vulnerabilities, cluster SPM findings, namespace-scoped access | `-[APPSEC]-06-kubernetes-and-container-security.md` |
| Investigating a security problem: Security Investigator entity pivots, Davis CoPilot NL-to-DQL, investigation log as audit artifact | `-[APPSEC]-07-investigator-and-davis-copilot.md` |
| Routing findings to Jira/ServiceNow/PagerDuty/Slack, Event trigger on `security.events` (not the Problem trigger), ack loopback, SLA and backlog burn-rate alerts | `-[APPSEC]-08-workflows-notifications-remediation.md` |
| AppSec permissions: `environment:roles:view-security-problems`, `storage:security.events:read`, `storage:buckets:read`, persona policies, boundaries, `view-sensitive-request-data` | `-[APPSEC]-09-iam-and-gen3-permissions.md` |
| Executive dashboards: open vulnerabilities by DSS level (trend), severity mix, MTTR by team, compliance coverage, governance cadence, DPS cost (RVA/RAP billed per GiB-hour) | `-[APPSEC]-10-dashboards-reporting-governance.md` |

## Related series

- Kubernetes rollout mechanics (Operator, DynaKube CR, namespaces): `../K8S - Kubernetes Monitoring/`
- IAM policy DSL, group/policy design beyond AppSec: `../IAM - IAM Administration/`
- Workflow building and notification plumbing: `../WFLOW - Workflows and Alert Notifications/`
- Dashboard design for stakeholder audiences: `../DASH - Dashboard Design & Building/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "APPSEC-02") and mention that the matching JSON in
  `NOTEBOOKS/` can be imported into a Dynatrace tenant for interactive use.
