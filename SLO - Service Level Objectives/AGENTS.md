# AGENTS.md — SLO: Service Level Objectives

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

5 notebooks on defining and operating SLOs in Dynatrace: SLI/SLO/error-budget
fundamentals and the modern DQL-based SLO app, the four SLI query shapes,
composition and burn-rate math, multiwindow burn-rate alerting, and SLOs as
code. This series owns *reliability targets and error budgets* — not detector
mechanics or notification routing.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| What SLI/SLO/error budget mean, SLO vs SLA, the Dynatrace SLO app vs SLO Classic, gating IAM permissions (`slo:slos:*`), choosing your first SLOs | `-[SLO]-01-fundamentals.md` |
| Writing the SLI query — availability, latency (percentile-threshold metric vs good/total span ratio), error-rate, and custom/business SLIs as good ÷ total DQL | `-[SLO]-02-defining-slis.md` |
| Error-budget math (allowed downtime per target), burn-rate calculation, composite/weighted-global SLOs, rolling vs calendar windows | `-[SLO]-03-composition-and-error-budgets.md` |
| Alerting on SLOs — why burn-rate beats threshold-on-SLI, fast-burn/slow-burn multiwindow alerts, configuring SLO alerts, routing breaches, avoiding fatigue | `-[SLO]-04-alerting.md` |
| Provisioning SLOs from version control — the `builtin:monitoring.slo` schema, the `dynatrace_slo_v2` Terraform resource, Monaco, the export-known-good API/CI-CD path | `-[SLO]-05-slos-as-code.md` |

## Related series

- Overall alerting strategy — where SLO burn-rate fits among the four detection mechanisms: go to `../ALERT - Alerting Strategy and Design/` instead for the end-to-end design.
- Anomaly detectors and Davis problems — signals that are not reliability promises: go to `../AIOPS - Dynatrace Intelligence/` instead for detection mechanics.
- Delivering a burn-rate alert to a channel or ITSM tool: go to `../WFLOW - Workflows and Alert Notifications/` instead for notification plumbing.
- Terraform/Monaco/CI-CD tooling depth behind SLO-05: `../AUTOM - Dynatrace Automation/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "SLO-04") and mention that the matching JSON in
  `NOTEBOOKS/` can be imported into a Dynatrace tenant for interactive use.
