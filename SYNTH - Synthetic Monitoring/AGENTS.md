# AGENTS.md — SYNTH: Synthetic Monitoring

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

7 notebooks on Dynatrace Synthetic Monitoring: browser, HTTP, and network
availability (multi-protocol) monitors, private location deployment on
ActiveGates, and synthetic analytics/alerting — queried through
`dt.synthetic.events` and the `dt.synthetic.*` metric families (Synthetic on
Grail).

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| Synthetic basics: monitor types, key concepts, where synthetic data lives in Grail (`fetch dt.synthetic.events`), first query | `-[SYNTH]-01-fundamentals.md` |
| Browser monitors: single-URL vs clickpaths, `dt.synthetic.browser.*` metrics, classic vs new browser experience (post-2026-01-26 tenants), validation and assertions | `-[SYNTH]-02-browser-monitors.md` |
| HTTP/API monitors: multi-step workflows with variable extraction, authentication, response validation, SSL certificate expiry (`result.statistics.peer_certificate_expiry_date`), DNS/TCP/TLS/TTFB timing breakdown | `-[SYNTH]-03-http-monitors.md` |
| Private locations: synthetic-enabled ActiveGate setup, deployment options, `METRIC_3RD_GEN_ENABLED` on K8s/OpenShift, location health, troubleshooting | `-[SYNTH]-04-private-locations.md` |
| ICMP/ping, DNS, or TCP port checks: network availability (multi-protocol) monitors, `dt.entity.multiprotocol_monitor`, `multiprotocol_monitor_execution` events, protocol field discovery | `-[SYNTH]-05-network-monitoring.md` |
| Analyzing results: availability/success rates, `result.state`, performance and location comparison, synthetic SLOs, alerting strategies, dashboards | `-[SYNTH]-06-analytics.md` |
| Consolidated checklist: monitor type selection, frequency/location strategy, thresholds, alerting config, operational hygiene | `-[SYNTH]-99-best-practice-summary.md` |

If more than three rows match, start with `-[SYNTH]-99-best-practice-summary.md`
and follow its pointers.

## Related series

- Real-user data that synthetic checks complement: `../WEBRUM - Web Real User Monitoring/`
- SLOs and error budgets built on synthetic availability: `../SLO - Service Level Objectives/`
- Routing synthetic-failure alerts to people and tools: `../WFLOW - Workflows and Alert Notifications/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "SYNTH-03") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
