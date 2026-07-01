# ALERT-03: Routing, Destinations, and Cost

> **Series:** ALERT — Alerting Strategy and Design | **Notebook:** 03 of 05 | **Created:** June 2026 | **Last Updated:** 06/25/2026

## Overview

Once a problem fires, getting it to the right people without waste is a routing problem with a cost dimension. This notebook covers the **simple vs multi-step workflow** decision (and its billing implications), the destination landscape, and where the legacy alerting-profile path still applies. It orchestrates the WFLOW series rather than repeating it.

---

## Table of Contents

1. [Simple vs Multi-Step Workflows](#simple)
2. [The Routing Pattern](#pattern)
3. [Destination Landscape](#destinations)
4. [The Legacy Path](#legacy)
5. [Cost Discipline](#cost)

---

## Prerequisites

| Requirement | Details |
|-------------|---------|
| **Dynatrace Environment** | SaaS Gen3 with AutomationEngine (Workflows) |
| **Prior reading** | ALERT-01; routing depth in WFLOW-03/04 |
| **Upstream** | Problems enriched with routing metadata (ALERT-01 §4) |

<a id="simple"></a>
## 1. Simple vs Multi-Step Workflows

This is the cost decision that the field most often gets wrong.

| | Simple workflow | Multi-step workflow |
|--|-----------------|---------------------|
| **Shape** | Problem trigger → one notification action | Trigger → conditions, multiple actions, enrichment, multiple teams |
| **Billing** | Not charged on run (query + send) | Charged (workflow-hours) |
| **Use when** | Filter problems and send to one channel | Routing to multiple teams, leaving comments, conditional logic, enrichment, centralised config |

**Default to simple workflows where possible.** This may mean several simple workflows instead of one big one — that is the cheaper shape, because their query and runs are not charged under workflow-hours. Reach for a multi-step workflow when you genuinely need conditional branching, multi-team routing, or a single centralised configuration; the capability is worth the cost when you need it, but do not pay it by default.

> Verify current billing specifics against the Workflows documentation for your DPS model — the simple-vs-billed boundary is the kind of detail that shifts, and it is exactly what the field-authored source document got partially right but did not fully pin down.

**A worked trap.** A frequent pattern is *one routing workflow per team or area* that catches every problem for that area and posts to the team's chat channel. It looks like simple fan-out — but the form of the delivery action decides the billing, not the apparent shape. A single built-in notification action keeps the workflow simple; a hand-rolled HTTP-function call or a run-JavaScript task to reach the same channel is what can tip it into the billed multi-step category. Choose the action type deliberately, and confirm which connectors count as simple notification actions for your DPS model.

<a id="pattern"></a>
## 2. The Routing Pattern

1. Create a workflow with a **Problem trigger**.
2. Configure the trigger to **filter the problems** relevant to this channel — filter on the metadata you enriched upstream (team, zone, service, severity).
3. Add the **notification action** for the channel.
4. Set up the **connection** to that channel if not already present.
5. Compose the message, using `{` to embed problem details (name, link, severity, affected entity).

Routing dimensions — severity, team/ownership, service, time of day — and escalation patterns are covered in depth in WFLOW-04. Sprint-1.337 made Smartscape ownership a first-class routing attribute, so workflows can read the owning team directly rather than maintaining a side-table.

<a id="destinations"></a>
## 3. Destination Landscape

| Destination | Path | Notebook |
|-------------|------|----------|
| Slack / Teams | Native workflow connector | WFLOW-03/04 |
| PagerDuty / on-call | Native connector — use for fast-burn pages | WFLOW-04, SLO-04 |
| Jira | Native connector — create/comment/assign issues | WFLOW-04 |
| ServiceNow | Native connector / HTTP Table API / ITOM app | ALERT-04 |
| Email | Native action | WFLOW-03 |
| xMatters | Legacy alerting-profile path (see below) | — |
| Anything else | HTTP action to a webhook | WFLOW-08 |

Match the destination to the urgency: fast-burn / acute → page (PagerDuty, on-call); slow-burn / steady → ticket (Jira, ServiceNow).

<a id="legacy"></a>
## 4. The Legacy Path

Some integrations predate workflows and still rely on **classic problem notifications** driven by alerting profiles rather than the AutomationEngine. **xMatters** is the canonical example: Dynatrace workflows are the preferred routing method, but they do not drive xMatters directly unless you POST the problem JSON to an xMatters endpoint via a workflow HTTP action.

If you have a classic problem-notification integration working, it is not deprecated — but new integrations should use workflows. When you find a destination that "only works the classic way," check whether it exposes a webhook a workflow HTTP action can target before settling for the alerting-profile path.

<a id="cost"></a>
## 5. Cost Discipline

- **Prefer simple workflows.** Several cheap simple workflows beat one billed multi-step workflow when no conditional logic is needed.
- **One problem, one notification.** Let the Davis problem group related signals; do not also alert on the underlying raw metrics, or you double-notify and double-spend.
- **Filter early in the trigger.** A trigger that matches every problem and decides relevance later still evaluates on every problem.
- **Centralise only when it pays.** A single multi-step routing workflow is easier to govern but is billed; weigh that against many simple ones.

> <sub>**Sources:** [Workflows (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/workflows), [Workflow actions — Jira / MS Teams (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/workflows/default-workflow-actions). **Softened:** exact simple-vs-billed boundaries follow your DPS model — verify against current Workflows billing docs.</sub>

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
