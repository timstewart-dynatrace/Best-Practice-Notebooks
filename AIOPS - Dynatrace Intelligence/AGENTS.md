# AGENTS.md — AIOPS: Dynatrace Intelligence

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

8 notebooks on the platform's AI capabilities — Causal AI (Davis problems and
root cause), Predictive AI (anomaly detection, baselines, forecasts), and
Generative AI (Davis CoPilot / Dynatrace Assist) — plus the model inventory,
integration surfaces (Workflow AI tasks, MCP server, agentic patterns), and a
composed Detect → Investigate → Remediate operational pattern. This series owns
*detection and AI mechanics* — not alerting strategy or notification routing.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| What "Dynatrace Intelligence" is — the Causal/Predictive/Generative categories, where AI surfaces in the product, the Smartscape + Grail foundation | `-[AIOPS]-01-dynatrace-intelligence-overview.md` |
| Building/choosing anomaly detectors — static threshold vs auto-adaptive vs seasonal baseline vs multi-dimensional vs novelty/forecast, metric events, Settings 2.0 schemas, testing with Davis analyzers | `-[AIOPS]-02-anomaly-detection.md` |
| Davis problems and root cause — how Causal AI groups signals via Smartscape, `dt.davis.problems` vs `dt.davis.events` DQL, active problem feed, MTTR reporting, severity rollups | `-[AIOPS]-03-davis-ai-problems-and-root-cause.md` |
| Davis CoPilot / Dynatrace Assist — natural-language-to-DQL, explain-query (DQL2NL), problem summarization, limitations, data privacy | `-[AIOPS]-04-davis-copilot-dynatrace-assist.md` |
| Which model does what — causal correlation, predictive baselines/forecasts, generative LLMs, where each runs, cost, bring-your-own-model status | `-[AIOPS]-05-ai-models.md` |
| Connecting AI out — Workflow AI tasks (DQL cost optimization, problem summaries, forecasts), the Dynatrace MCP server for IDEs/agents, AutomationEngine agentic/approval patterns | `-[AIOPS]-06-ai-integrations-and-agentic-workflows.md` |
| Composing it all — the Detect → Investigate → Remediate loop with capacity, incident-response, and quality-regression scenarios, AIOps health dashboard | `-[AIOPS]-07-putting-it-together.md` |
| Series recap — canonical DQL query index, cross-series pointers, next steps | `-[AIOPS]-99-series-summary.md` |

If more than three rows match, start with `-[AIOPS]-99-series-summary.md`
and follow its pointers.

## Related series

- Overall alerting strategy — which detection mechanism to pick, the end-to-end anti-noise design: go to `../ALERT - Alerting Strategy and Design/` instead when the question is strategy, not detector mechanics.
- Notification plumbing — getting a Davis problem to Slack/PagerDuty/ServiceNow: go to `../WFLOW - Workflows and Alert Notifications/` instead for routing and connections.
- Reliability targets — SLIs, error budgets, burn-rate alerting: go to `../SLO - Service Level Objectives/` instead when the signal is a reliability promise.
- Extracting log/span signals into cheap alertable metrics at ingest: `../OPIPE - OpenPipeline Beyond Logs/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "AIOPS-03") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
