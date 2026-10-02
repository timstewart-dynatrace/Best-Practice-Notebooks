# S2S-08: Step 8 — Enable: Parallel Operation and Stakeholder Handover

> **Series:** S2S — SaaS to SaaS Migration | **Notebook:** 8 of 9 | **Phase:** Run | **Step:** Enable | **Created:** March 2026 | **Last Updated:** 10/01/2026

## Overview

With data pipelines, SLOs, and alerting configured in the target tenant (Step 7), this step focuses on the human side of migration: managing parallel operation, communicating changes to stakeholders, establishing Dynatrace Intelligence baselines, and preparing users for the cutover. Parallel operation is the most expensive phase of the migration — you are paying for two tenants — so the goal is to keep it as short as possible while ensuring confidence in the target tenant.

> **S2S Migration Journey — 3 Phases / 9 Steps**
>
> **Plan:** 1. Discover | 2. Strategize | 3. Design
>
> **Upgrade:** 4. Prepare | 5. Execute | 6. Integrate
>
> **Run:** 7. Expand | **8. Enable** | 9. Optimize

---

## Table of Contents

1. [Parallel Operation Strategy](#parallel-operation-strategy)
2. [Dynatrace Intelligence Baseline Establishment](#davis-ai-baseline-establishment)
3. [SLO Continuity](#slo-continuity)
4. [Stakeholder Communication](#stakeholder-communication)
5. [Cost Management During Parallel Period](#cost-management-during-parallel-period)
6. [User Training and Documentation](#user-training-and-documentation)
7. [Step Completion Checklist](#step-completion-checklist)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Step 7 Complete** | OpenPipeline rules, SLOs, and alerting deployed in target tenant |
| **Agents Reporting** | OneAgents and DynaKube operators sending data to target tenant |
| **Both Tenants Active** | Source and target tenants both receiving data for validation |
| **Stakeholder List** | Identified audiences for migration communication |
| **Training Environment** | Target tenant accessible for user training and exploration |

<a id="parallel-operation-strategy"></a>

## 1. Parallel Operation Strategy

During parallel operation, both the source and target tenants are live. Each host's OneAgent reports to exactly one of them — a single OneAgent has one destination — so hosts move from source to target in waves, while synthetics and log forwarders can feed both tenants at once. Cloud connections are the exception to plan around: for Azure, the docs warn against monitoring one subscription from multiple connections (**S2S-06** §1), so Azure subscriptions move per subscription rather than running in both. This period exists to validate the target tenant, build Dynatrace Intelligence baselines, and give users confidence before cutover.

### Parallel Operation Timeline

| Week | Activity | Key Milestones |
|------|----------|----------------|
| **Week 1** | Data validation | Confirm hosts, services, logs, and spans in target match source counts |
| **Week 2** | Dynatrace Intelligence learning | Baselines begin forming; initial anomaly detection active |
| **Week 3** | SLO evaluation | 7-day rolling SLOs have full data; compare source vs target values |
| **Week 4** | User acceptance | Key users validate dashboards, workflows, and alerts in target |
| **Weeks 5–6** | Cutover preparation | Go/no-go decision, communication, maintenance window scheduled |
| **Weeks 7–8** | Post-cutover buffer | Source tenant no longer ingesting; data stays queryable for its bucket retention |
| **Week 9+** | Decommission | Source tenant deactivated and deprovisioned |

### Operation Models

| Model | Description | Best For | Duration |
|-------|-------------|----------|----------|
| **Full Parallel** | Both tenants live; each host reports to exactly one — pending waves to source, migrated waves to target. Synthetics and log forwarders can feed both; Azure subscriptions switch per subscription rather than feeding both (see S2S-06 §1) | High-risk environments, regulatory requirements | 4–6 weeks |
| **Phased** | Migrate groups of hosts/services incrementally | Large environments (1000+ hosts) | 6–10 weeks |
| **Reference** | Source receives data, target receives from a representative subset only | Cost-constrained, low-risk migrations | 2–4 weeks |

> **Cost impact:** overlap cost comes from the integrations, synthetics and forwarders you point at both tenants, plus the per-wave window — not from agents, because a host reports to one tenant at a time. Work with your Dynatrace account team to negotiate temporary parallel licensing.
>
> **Agents cannot dual-report.** Do not plan for a host to send to both tenants: a single OneAgent per host is required, and one OneAgent has one destination (**S2S-02**, and **FAQ-25** §6).

### Validate Host Count Parity

Run this query in both source and target tenants. The host counts should match (allowing for normal churn):

```dql
// Host count validation — run in both source and target tenants (Smartscape)
smartscapeNodes "HOST", from:-24h
| fieldsAdd provider = coalesce(cloud.provider, "none (on-premises or undetected)")
| summarize hosts = count(), by:{provider}
| sort hosts desc

// cloud.provider is aws / azure / gcp on cloud-hosted hosts. Summed across both tenants, the
// per-provider totals should match the pre-migration baseline (allowing for churn), because each
// host reports to exactly one tenant. Classic fallback: fetch dt.entity.host — but its cloud-tag
// fields (awsNameTag / azureResourceGroupName) do not exist on the Smartscape node, and mixing
// classic and Smartscape counts produces false gaps.
```

### Validate Log Continuity

Compare log volumes between source and target to ensure data pipelines are operating at parity:

```dql
// Log volume over the last 24 hours by loglevel
// Run in both source and target — compare totals
fetch logs, from:-24h
| summarize record_count = count(), by:{loglevel}
| sort record_count desc
```

<a id="davis-ai-baseline-establishment"></a>

## 2. Dynatrace Intelligence Baseline Establishment

Dynatrace Intelligence requires historical data to establish baselines for anomaly detection. In the target tenant, Dynatrace Intelligence starts from zero — there is no way to transfer learned baselines.

### Baseline Types and Learning Times

In community practice, baselines become trustworthy on roughly these timescales — planning estimates, not documented figures. Replace them with a measurement after each wave: **FAQ-25** §5 has a per-host history-depth query that shows which migrated hosts have enough data yet.

| Baseline Type | Learning Period | What It Detects |
|--------------|----------------|------------------|
| **Response time** | 1–2 weeks | Service slowdowns |
| **Error rate** | 1–2 weeks | Error rate increases |
| **Throughput** | 2–4 weeks | Traffic anomalies (requires weekly patterns) |
| **Infrastructure** | 1 week | CPU, memory, disk anomalies |
| **Custom events** | 2–4 weeks | Depends on event frequency |

### Accelerating Baseline Learning

You cannot directly speed up Dynatrace Intelligence learning, but you can reduce noise:

| Action | Impact |
|--------|--------|
| Use wave-scoped maintenance windows during each cutover | Keeps the cutover itself from raising problems |
| Ensure consistent traffic patterns | Avoid load testing or unusual deployments during baseline period |
| Configure sensitivity thresholds | Set anomaly detection sensitivity to match source tenant settings |
| Migrate anomaly detection settings first | Ensures Dynatrace Intelligence uses the same thresholds from day one |

### Expected Behavior During Baseline Period

| Week | Dynatrace Intelligence Behavior | Action Required |
|------|---------------|------------------|
| Week 1 | Many false positives (no baseline context) | Route to a staging alert channel — do not suppress or retune yet |
| Week 2 | Infrastructure baselines forming, fewer false positives | Monitor and triage manually |
| Week 3 | Service baselines forming, anomaly detection improving | Compare detected problems with source tenant |
| Week 4+ | Baselines stable for most metrics | Ready for production alerting |

### Monitor Detected Problem Trends

Track detected problem volume in the target tenant to see baselines stabilize over time:

```dql
fetch dt.davis.problems, from:-7d
| summarize {problem_count = count()}, by:{day = bin(event.start, 24h)}
| sort day asc

// Bucketed by event.start (when the problem began) — timestamp on a problem record is its
// last update. 24h rather than 1d: calendar durations are rewritten with an INFO notification.
```

```dql
// detected problems by category — identify which areas are generating noise
fetch dt.davis.problems, from:-7d
| filter event.status == "ACTIVE" or event.status == "CLOSED"
| summarize problem_count = count(), by:{event.category}
| sort problem_count desc
```

<a id="slo-continuity"></a>

## 3. SLO Continuity

SLOs with rolling evaluation windows will show incomplete or misleading values during the transition period. This is expected and must be communicated to stakeholders.

### Impact During Transition

| SLO Window | Impact | Duration of Impact |
|-----------|--------|-------------------|
| 7-day rolling | Partial data for first 7 days | 1 week |
| 30-day rolling | Severely incomplete for first 30 days | 4 weeks |
| Calendar month | No data until month boundary | Up to 30 days |

### Mitigation Strategies

| Strategy | Implementation | When to Use |
|----------|---------------|-------------|
| **Temporary short window** | Change 30-day SLOs to 7-day during transition | When stakeholders need early visibility |
| **Dual reporting** | Report SLOs from both tenants until target window fills | When SLO values are reported externally |
| **SLO holiday** | Suspend SLO reporting during transition with documented justification | When stakeholders accept a gap |
| **Timestamp shift** | Start SLO tracking at beginning of evaluation window | For calendar-based SLOs |

<a id="stakeholder-communication"></a>

## 4. Stakeholder Communication

Migration success depends as much on communication as on technical execution. Different audiences need different messages.

### Communication Plan by Audience

| Audience | What They Need to Know | When | Channel |
|----------|----------------------|------|----------|
| **Executive leadership** | Timeline, risk summary, cost impact, go/no-go decision | Week 1 + weekly status | Email + steering committee |
| **SRE / Platform team** | Technical details, runbooks, rollback procedures | Throughout | Slack + wiki |
| **Application developers** | New tenant URL, updated API tokens, dashboard locations | 2 weeks before cutover | Email + team meeting |
| **Service desk** | Updated escalation paths, known issues during transition | 1 week before cutover | Runbook update |
| **External stakeholders** | Expected alert changes, SLO reporting gaps | 2 weeks before cutover | Email + SLA documentation |

### Expected Impact Summary

Include this table in your stakeholder communication:

| Area | Impact | Duration | Mitigation |
|------|--------|----------|------------|
| **Dynatrace Intelligence** | Increased false positives in target tenant | Until migrated hosts have enough history (measured per host — **FAQ-25** §5) | Alerts routed to staging channel |
| **SLOs** | Incomplete evaluation in target tenant | 1–4 weeks (depends on window) | Dual reporting from both tenants |
| **Historical data** | Not available in target tenant | Permanent | Source tenant stays queryable during the buffer period (no new ingest) |
| **Dashboards** | New URLs, possible layout differences | One-time | User training and URL redirect documentation |
| **API tokens** | All tokens regenerated | One-time | Token distribution before cutover |

<a id="cost-management-during-parallel-period"></a>

## 5. Cost Management During Parallel Period

Parallel operation is expensive — but only for what actually reaches both tenants. A host reports to one tenant at a time, so agent-sourced data is not doubled; integrations, synthetics and forwarders that you deliberately point at both tenants are.

### Dual Licensing Impact

| DPS Capability | Impact During Parallel | Optimization |
|---------------|----------------------|---------------|
| **Full-Stack (GiB-hours) / Infrastructure (host-hours) Monitoring** | 1x per host — each host reports to one tenant | Keep wave windows short |
| **Log Management & Analytics (ingest & retain)** | 1x for agent-collected logs; 2x only for forwarders you deliberately dual-feed | Dual-feed only the sources you must compare |
| **Traces** | 1x per host (agent-sourced) | — |
| **Metrics** | 1x for agent metrics; 2x for cloud-integration and API-ingested metrics sent to both | Enable cloud integrations in the target per wave |
| **Real User Monitoring / Synthetic Monitoring** | 2x only for synthetics and RUM you run in both tenants | Disable synthetic in source once target is validated |

See **FINOPS-01** for the current DPS capability names and how to query consumption per tenant.

### Cost Optimization Strategies

| Strategy | Savings | Risk |
|----------|---------|------|
| Shorten parallel period (2 weeks vs 4 weeks) | 50% reduction | Less baseline time for Dynatrace Intelligence |
| Phased migration (migrate by group) | Dual-fed integrations are enabled per wave, not all at once | Longer total timeline |
| Reduce source tenant data collection | 20–40% on source during parallel | Reduced source visibility |
| Negotiate temporary parallel licensing | Cost-neutral | Requires account team engagement |

> **Recommendation:** Negotiate parallel licensing with your Dynatrace account team **before** starting the parallel phase. Most account teams can provide temporary licensing for migration periods.

<a id="user-training-and-documentation"></a>

## 6. User Training and Documentation

Users need to know what changes, what stays the same, and where to find things in the target tenant.

### Persona-Based Training

| Persona | Training Focus | Format | Duration |
|---------|---------------|--------|----------|
| **Platform admin** | IAM, OpenPipeline, bucket management, API token generation | Hands-on workshop | 2 hours |
| **SRE** | Dashboards, SLOs, alerting, Dynatrace Intelligence differences | Live demo + Q&A | 1 hour |
| **Developer** | New tenant URL, notebook access, DQL querying | Self-service guide + office hours | 30 min |
| **Service desk** | Escalation paths, known issues, FAQ | Runbook update | 30 min |

### Documentation Updates Required

| Document | Update Required |
|----------|----------------|
| Runbooks | New tenant URLs, API endpoints, token references |
| Wiki / knowledge base | Dashboard links, notebook links, SLO references |
| CI/CD pipelines | API token environment variables, deployment scripts |
| Incident response playbooks | Alert routing, escalation contacts |
| Onboarding documentation | New user access procedures, IAM group requests |

### FAQ Template

| Question | Answer |
|----------|--------|
| Why are my dashboards showing less data? | The target tenant does not have historical data. Data accumulates from the migration date forward. |
| Why am I getting more alerts than usual? | Dynatrace Intelligence is establishing baselines from the target's own data. Expect more noise for recently migrated hosts until they have built up history — the platform team tracks this per host. |
| Where is my old dashboard? | Dashboards have been migrated. Open the Dashboards app in the target tenant and search by name. |
| Do I need a new API token? | Yes. All API tokens must be regenerated in the target tenant. |
| When will SLOs be accurate? | SLOs need one full evaluation window of data. 7-day SLOs: 1 week. 30-day SLOs: 4 weeks. |

<a id="step-completion-checklist"></a>

## 7. Step Completion Checklist

Before proceeding to **Step 9 — Optimize**, confirm that you have completed each item:

| Checkpoint | Status |
|-----------|--------|
| Parallel operation model selected (full, phased, or reference) | [ ] |
| Host count parity validated between source and target | [ ] |
| Log and span volumes validated between source and target | [ ] |
| Dynatrace Intelligence problem trend monitored — false positives decreasing | [ ] |
| SLO continuity strategy documented (temporary window, dual reporting, or holiday) | [ ] |
| Stakeholder communication sent to all audiences | [ ] |
| Cost optimization negotiated with account team | [ ] |
| User training completed for all personas | [ ] |
| Documentation updated (runbooks, wiki, CI/CD, playbooks) | [ ] |
| Go/no-go criteria defined for cutover decision | [ ] |

---

## Next Step

> **S2S-09: Step 9 — Optimize** — Execute the final cutover, validate migration completeness, decommission the source tenant, and capture lessons learned.

| Completed | Next | Remaining |
|-----------|------|-----------|
| ~~1. Discover~~ → ~~2. Strategize~~ → ~~3. Design~~ → ~~4. Prepare~~ → ~~5. Execute~~ → ~~6. Integrate~~ → ~~7. Expand~~ → ~~8. Enable~~ | **9. Optimize** | — |

---

## Summary

In Step 8, you:

- Established a parallel operation strategy with a clear timeline (weeks 1–9+)
- Monitored Dynatrace Intelligence baseline establishment and tracked false positive trends
- Documented SLO continuity strategy for the transition period
- Communicated migration impact to all stakeholder audiences
- Optimized cost during the dual-tenant parallel period
- Trained users by persona and updated all supporting documentation

### Additional Resources

- [Dynatrace Intelligence Anomaly Detection](https://docs.dynatrace.com/docs/dynatrace-intelligence/anomaly-detection)
- [Service-level objectives (DT docs)](https://docs.dynatrace.com/docs/deliver/service-level-objectives)
- [Dynatrace Notifications and Alerting](https://docs.dynatrace.com/docs/analyze-explore-automate/notifications-and-alerting)
- [Dynatrace Platform Subscription (DT docs)](https://docs.dynatrace.com/docs/license)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
