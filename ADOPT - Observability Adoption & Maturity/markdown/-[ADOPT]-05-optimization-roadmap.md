# ADOPT-05: Optimization Roadmap

> **Series:** ADOPT — Observability Adoption & Maturity | **Notebook:** 5 of 6 | **Created:** March 2026 | **Last Updated:** 10/05/2026

## Overview

An optimization roadmap translates maturity assessment results and success metrics into a prioritized plan of action. This notebook covers monthly milestone planning, the distinction between quick wins and strategic investments, cost optimization opportunities (bucket management, sampling, retention tuning), automation candidates, and methods for measuring the return on observability investment. The roadmap framework is designed to be adapted to any organization's size and pace.


> **OneAgent primary-field enrichment (OneAgent 1.333+):** OneAgent can enrich telemetry at the source — metrics, spans, logs, events and entities — with the primary Grail fields `dt.security_context`, `dt.cost.costcenter` and `dt.cost.product`, and with primary tags such as `primary_tags.environment`. The values travel with the data into OpenPipeline routing, bucket assignment and record-level permissions, with no processing rule needed. Set them at install time or on an existing host with `--set-host-tag` (for example `oneagentctl --set-host-tag="dt.cost.costcenter=12345"`), or per process with `DT_TAGS`. See [Primary Grail fields and tags enrichment through OneAgent (DT docs)](https://docs.dynatrace.com/docs/ingest-from/dynatrace-oneagent/oneagent-attribute-enrichment).

### Dynatrace Version Support Policy

| Component | Standard Support | Enterprise Support |
|-----------|-----------------|-------------------|
| **OneAgent** | 9 months | 12 months |
| **ActiveGate** | 9 months | 12 months |
| **Dynatrace Operator** | Independent release cycle — check [release notes](https://docs.dynatrace.com/docs/whats-new) |

Support periods start on each version's release date, so an agent fleet ages out of support even if nothing changes.

### Capability Support Statuses

The OneAgent platform and capability support matrix labels each capability with one of these statuses (quoted from the matrix legend):

| Status | Meaning |
|------|---------|
| **GA** | "Generally available and fully supported." |
| **Preview** | "Preview features aren't production-ready and they aren't officially supported." |
| **Future** | "A feature or technology support that is either on the roadmap or may be considered on-demand." |
| **Not planned** | "A feature or technology support that Dynatrace does not currently plan to pursue." |

Do not build a production roadmap item on a Preview capability.

### Third-Party Technology EOL Policy

"Dynatrace typically supports technologies and their versions six months longer than the vendor to give you enough time to upgrade your environment." "End of support announcements are provided six months in advance."

> **Reference:** [OneAgent platform and capability support matrix](https://docs.dynatrace.com/docs/ingest-from/technology-support/oneagent-platform-and-capability-support-matrix) | [End-of-Support Announcements](https://docs.dynatrace.com/docs/whats-new/technology/end-of-support-news) | [Support Policy](https://www.dynatrace.com/company/trust-center/support-policy/)

---

## Table of Contents

1. [Roadmap Planning Principles](#roadmap-principles)
2. [Quick Wins — Month 1](#quick-wins)
3. [Foundation Building — Months 2-3](#foundation-building)
4. [Strategic Investments — Months 4-6](#strategic-investments)
5. [Cost Optimization Opportunities](#cost-optimization)
6. [Automation Candidates](#automation-candidates)
7. [Measuring ROI of Observability](#measuring-roi)
8. [Summary and Series Conclusion](#summary)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Dynatrace Environment** | Dynatrace SaaS (Grail) on DPS for § 5.3. The DQL does not apply to Dynatrace Managed, which has no Grail. |
| **Permissions** | `storage:logs:read`, `storage:spans:read`, `storage:events:read`, `storage:entities:read`, `storage:system:read`, `storage:buckets:read` |
| **Context** | Completed ADOPT-01 through ADOPT-04 |
| **Audience** | Platform team leads, engineering directors, VP of Engineering, CTO |

<a id="roadmap-principles"></a>

## 1. Roadmap Planning Principles

An effective optimization roadmap follows these principles:

### Principle 1: Start with Impact, Not Complexity

Prioritize actions by their impact on reliability and team productivity, not by their technical sophistication. A simple alerting cleanup can reduce MTTR more than a complex automation workflow.

### Principle 2: Measure Before and After

Every initiative on the roadmap should have a measurable outcome tied to the success metrics defined in ADOPT-03 (MTTR, MTTD, noise ratio, etc.).

### Principle 3: Quick Wins Build Momentum

Early, visible wins build organizational trust in the observability investment. Prioritize 2-3 quick wins in the first month.

### Principle 4: Budget for Learning

Every initiative has a learning component. Allocate time for team enablement (ADOPT-04) alongside technical work.

### Roadmap Structure

| Timeframe | Focus | Effort Level |
|-----------|-------|--------------|
| Month 1 | Quick wins — immediate impact | Low effort, high visibility |
| Months 2-3 | Foundation — platform hygiene | Medium effort, foundational |
| Months 4-6 | Strategic — transformative initiatives | High effort, long-term value |

<a id="quick-wins"></a>

## 2. Quick Wins — Month 1

Quick wins require minimal effort but deliver visible results. They establish credibility for the broader optimization program.

### 2.1 Alert Noise Cleanup

**Goal:** Halve the share of problems that close within five minutes, and remove the top recurring problems from the notification path.

**Actions:**
- Identify the problem types that most often close within five minutes (query below)
- Set a **Minimum duration** on the problem-triggered workflows that notify people, so transient problems never page anyone (FAQ-21)
- Tune the detector behind each short-lived type (ALERT-02)
- Configure maintenance windows for planned work
- Document alerting ownership per service

Use this query to find the problem types worth tuning first:

```dql
// Top 10 problem types by volume, with the share that closed within 5 minutes, 30 days
fetch dt.davis.problems, from:-30d
| filter dt.davis.is_duplicate == false
| summarize {
    total = count(),
    closed_within_5min = countIf(resolved_problem_duration < 5m)
  }, by:{event.name}
| fieldsAdd short_lived_pct = round(toDouble(closed_within_5min) / toDouble(total) * 100, decimals: 1)
| sort total desc
| limit 10
```

### 2.2 Agent Version Standardization

**Goal:** Every host on a OneAgent release that is still inside its support window, with as few distinct releases as practical.

**Actions:**
- Identify hosts on releases older than the support window (see the version policy above)
- Enable auto-update for non-production hosts
- Schedule coordinated updates for production (FAQ-04)

```dql
// Agent version distribution — identify outdated releases on hosts seen in the last 24 hours.
// The host property is installerVersion; agentVersion exists only on process group instances.
fetch dt.entity.host, from:-24h
| summarize {host_count = count()}, by:{installerVersion}
| sort host_count desc
| limit 20

// No Smartscape equivalent: the OneAgent version is not a Smartscape HOST node field.
```

### 2.3 Dashboard Consolidation

**Goal:** Replace ad-hoc dashboards with standardized templates.

**Actions:**
- Audit existing dashboards for overlap and staleness
- Create 3 standard templates: Infrastructure Health, Application Health, Business KPIs
- Retire unused dashboards

<a id="foundation-building"></a>

## 3. Foundation Building — Months 2-3

Foundation work addresses platform hygiene and establishes the infrastructure for long-term success.

### 3.1 Complete Agent Deployment

**Goal:** Achieve > 95% host monitoring coverage.

**Actions:**
- Compare monitored hosts against CMDB inventory
- Deploy OneAgent to all unmonitored hosts
- Establish automated agent deployment for new hosts (cloud-init, Ansible, etc.)

### 3.2 Grail Bucket Organization

**Goal:** Implement a bucket strategy aligned with data ownership and retention requirements.

**Actions:**
- Audit current bucket usage and data distribution
- Design a bucket naming convention (e.g., `<team>_<datatype>_<env>`)
- Configure OpenPipeline routing to direct data to appropriate buckets
- Set retention policies per bucket based on compliance and cost requirements

```dql
// Analyze log volume by source to inform bucket strategy
fetch logs, from:-24h
| summarize {record_count = count()}, by:{log.source}
| sort record_count desc
| limit 20
```

### 3.3 SLO Definition

**Goal:** Define SLOs for the top 10 business-critical services.

**Actions:**
- Identify the 10 services with the highest business impact
- Define availability and latency SLOs for each
- Build burn-rate alerting for each SLO as described in SLO-04 — a native burn-rate alert announced for SaaS 1.347 was withdrawn from the release notes, so plan to build it
- Create an SLO dashboard for leadership visibility

### 3.4 IAM Policy Implementation

**Goal:** Move from shared admin accounts to role-based access.

**Actions:**
- Define IAM groups per team (see IAM series for detailed guidance)
- Implement least-privilege policies
- Scope data access with security context and policy boundaries (ORGNZ, IAM) — segments filter what a user sees on screen but do not restrict access to data (FAQ-24)

<a id="strategic-investments"></a>

## 4. Strategic Investments — Months 4-6

Strategic investments are higher-effort initiatives that deliver transformative value over time.

### 4.1 Workflow Automation

**Goal:** Automate response to the top 5 most frequent problem types.

**Actions:**
- Identify the 5 most frequent, well-understood problem types from ADOPT-03 data
- Design Dynatrace Workflows for automated triage and notification
- Progress to automated remediation for problems with known runbooks
- Measure MTTR improvement for automated vs manual resolution

### 4.2 Configuration as Code (Monaco or Terraform)

**Goal:** Manage Dynatrace configuration through version-controlled code.

**Actions:**
- Export current configuration with Monaco or the Terraform provider (AUTOM)
- Establish a Git repository for Dynatrace configuration
- Implement CI/CD pipeline for configuration deployment
- Check for drift on a schedule — Terraform's `plan` reports it natively; with Monaco it is a scheduled download compared against the repository

### 4.3 OpenTelemetry Integration

**Goal:** Extend observability to applications not covered by OneAgent.

**Actions:**
- Identify services that need custom instrumentation
- Implement OpenTelemetry SDKs for critical code paths
- Route OTel data through Dynatrace's OTLP endpoint
- Validate trace context propagation end-to-end

### 4.4 Business Event Correlation

**Goal:** Connect technical metrics to business outcomes.

**Actions:**
- Identify 3-5 key business transactions (checkout, login, search, etc.)
- Instrument business events for each transaction
- Build dashboards correlating technical health with business KPIs
- Establish business-context alerting (e.g., revenue impact of slowdowns)

<a id="cost-optimization"></a>

## 5. Cost Optimization Opportunities

Cost optimization is not about reducing observability — it is about ensuring every byte of data delivers value. The following queries help identify opportunities.

### 5.1 High-Volume Log Sources

Identify the sources generating the most log data. High-volume sources are candidates for filtering, sampling, or reduced retention.

```dql
// Top 15 log sources by record count in the last 7 days
fetch logs, from:-7d
| summarize {record_count = count()}, by:{log.source}
| sort record_count desc
| limit 15
```

### 5.2 Debug and Verbose Log Analysis

Debug-level logs are often the largest contributor to ingestion volume but rarely needed in production. This query quantifies the opportunity.

```dql
// Log record count by severity level over the last 24 hours
fetch logs, from:-24h
| summarize {record_count = count()}, by:{loglevel}
| sort record_count desc
```

> **Cost optimization actions for high-volume logs:**
> - **Drop at ingest:** Configure an OpenPipeline rule to drop DEBUG/TRACE records from production sources — or stop shipping them at the source
> - **Reduce retention:** Route verbose logs to buckets with shorter retention (7-14 days)
> - **Sample:** Use sampling for high-cardinality, low-value data
> - **Aggregate:** Replace raw log storage with metric extraction for patterns you only need to count

### 5.3 Entity Monitoring Mode Review

Not every host requires Full-Stack monitoring. Infrastructure Monitoring is billed per host-hour rather than per GiB-hour of host memory, and may be appropriate for hosts that run no application you need to trace. Read each host's mode from the billing events — the classic `monitoringMode` host property is empty for many hosts (ADOPT-02 § 1.2).

> **SaaS 1.347 — staged tenant rollout; the release notes are still marked pre-release:** billing usage events are moving their host ID from `dt.entity.host` to `dt.smartscape.host`. Verbatim: *"If you use custom DQL queries that reference entity ID attributes in billing usage events, review and update them to the new Smartscape attribute names."* The query below reads `coalesce(toString(dt.smartscape.host), dt.entity.host)`, so it returns the same hosts before and after the change reaches your tenant (on the validation tenant both fields were written, with identical values, on 10/05/2026). [What's new in SaaS 1.347 (DT docs)](https://docs.dynatrace.com/docs/whats-new/saas/sprint-347).

```dql
// Hosts by the capability they were billed under (DPS), last 24 hours
fetch dt.system.events, from:-24h
| filter event.kind == "BILLING_USAGE_EVENT"
// SaaS 1.347 moves the host ID to dt.smartscape.host; coalesce reads whichever field your tenant writes.
| fieldsAdd host = coalesce(toString(dt.smartscape.host), dt.entity.host)
| filter isNotNull(host)
| summarize {hosts = countDistinctExact(host)}, by:{billed_as = event.type}
| sort hosts desc
```

### 5.4 Span Volume Analysis

Distributed tracing can generate significant data volume. Identify services producing the most spans to evaluate whether head-based or tail-based sampling is appropriate.

```dql
// Top 10 services by span volume in the last 24 hours
fetch spans, from:-24h
| summarize {span_count = count()}, by:{dt.entity.service}
| sort span_count desc
| limit 10
```

### Cost Optimization Decision Matrix

A starting point from community practice — adjust each cell to your own value judgments and compliance requirements.

| Data Type | Low Value | Medium Value | High Value |
|-----------|-----------|-------------|------------|
| **Logs** | Drop DEBUG/TRACE at ingest | Short-retention bucket (7-14 days) | Default or longer retention (35+ days) |
| **Spans** | Sample at 10:1 | Sample at 2:1 | Full retention |
| **Metrics** | Drop unused dimensions; stop extracting unused metrics | Standard ingest | Standard ingest |
| **Events** | Filter noise | Standard retention | Full retention |

<a id="automation-candidates"></a>

## 6. Automation Candidates

Automation reduces toil and improves consistency. Prioritize automating tasks that are repetitive, well-defined, and currently manual.

### Automation Priority Matrix

| Task | Frequency | Manual Effort | Automation Tool | Priority |
|------|-----------|--------------|-----------------|----------|
| Alert triage and routing | Daily | 30 min/day | Dynatrace Workflows | High |
| Agent deployment on new hosts | Weekly | 1 hour/week | Cloud-init / Ansible | High |
| Dashboard creation for new services | Monthly | 2 hours/service | Monaco or Terraform templates | Medium |
| Configuration drift detection | Weekly | 1 hour/week | Terraform `plan`, or scheduled Monaco download + diff | Medium |
| Report generation for leadership | Weekly | 2 hours/week | Scheduled DQL notebooks | Medium |
| Incident postmortem data collection | Per incident | 1 hour/incident | Workflow + DQL | Low (high value) |
| Capacity forecasting | Monthly | 4 hours/month | Dynatrace Intelligence forecasting | Low (high value) |

### Quick-Start Automation: Transient-Problem Suppression

The simplest and highest-impact automation is keeping transient problems away from people. Use this query to identify candidates:

```dql
// Problems that closed within 5 minutes — candidates for a Minimum duration on the notifying workflow
fetch dt.davis.problems, from:-7d
| filter event.status == "CLOSED"
| filter dt.davis.is_duplicate == false
| fieldsAdd duration_minutes = resolved_problem_duration / 1m
| filter duration_minutes < 5
| summarize {auto_resolve_count = count()}, by:{event.name}
| sort auto_resolve_count desc
| limit 10
```

> **Interpretation:** Problems that consistently resolve within 5 minutes are strong candidates for:
> - A **Minimum duration** on the problem-triggered workflow that notifies people, so a problem that resolves inside the window never notifies (FAQ-21)
> - Workflow-based triage that only escalates if the problem persists
> - Detector tuning, when the type is never worth acting on (ALERT-02)

<a id="measuring-roi"></a>

## 7. Measuring ROI of Observability

Leadership will eventually ask: **"What is the return on our observability investment?"** This section provides a framework for answering that question.

### 7.1 Cost Avoidance

Calculate the cost of incidents prevented or shortened by improved observability.

| Metric | Formula | Example |
|--------|---------|----------|
| **Incident cost reduction** | (Old MTTR - New MTTR) x Avg incidents/month x Cost per hour of downtime | (2h - 0.5h) x 20 x $10,000 = $300,000/month |
| **Alert fatigue reduction** | (Old noise hours - New noise hours) x Avg engineer hourly rate | (20h - 5h) x $75 = $1,125/week |
| **Prevented outages** | Proactive detections x Estimated outage cost | 5 x $50,000 = $250,000/quarter |

### 7.2 Productivity Gains

| Metric | Formula | Example |
|--------|---------|----------|
| **Self-service investigation** | Escalation tickets eliminated x Avg ticket resolution time x Rate | 100 tickets x 2h x $75 = $15,000/month |
| **Onboarding acceleration** | Weeks saved per new hire x New hires per year x Weekly rate | 2 weeks x 20 hires x $3,750 = $150,000/year |

### 7.3 Business Value Metrics

| Metric | How to Measure |
|--------|---------------|
| **Revenue protected** | SLO compliance rate x Revenue at risk |
| **Customer satisfaction** | Correlation between uptime and NPS/CSAT |
| **Deployment velocity** | Change failure rate improvement enables faster releases |

### 7.4 Building the ROI Dashboard

Create a monthly ROI report combining:
1. MTTR/MTTD trends (from ADOPT-03)
2. Problem count trends (declining = improving)
3. Data volume and cost metrics (from this notebook)
4. Team enablement progress (from ADOPT-04)
5. Estimated cost avoidance (calculated from the formulas above)

<a id="summary"></a>

## 8. Summary and Next Steps

### Key Takeaways

- Start with quick wins in Month 1 to build organizational momentum
- Cost optimization is about value alignment, not cost cutting — every byte should earn its storage
- Automation candidates should be prioritized by frequency and manual effort
- ROI of observability can be quantified through cost avoidance, productivity gains, and business value
- The roadmap is a living document — review and adjust monthly

### ADOPT Series Summary

| Notebook | Focus | Key Outcome |
|----------|-------|-------------|
| **ADOPT-01** | Maturity Model | Understand where you are today |
| **ADOPT-02** | Platform Health | Assess deployment completeness |
| **ADOPT-03** | Success Metrics | Define MTTD, problem duration, alert quality, and baselines |
| **ADOPT-04** | Team Enablement | Build organizational capability |
| **ADOPT-05** | Optimization Roadmap | Plan the path forward |
| **ADOPT-06** | Coverage Audit and Staged Enablement | Close coverage gaps in waves, each proven at a value gate |
| **ADOPT-99** | Best Practice Summary | One checklist of every practice in the series |

### What Comes Next

Proceed to **ADOPT-06: Maximizing Platform Value** to turn the coverage gaps found in ADOPT-02 into a staged enablement program. Use the other notebook series in this repository (ONBRD, K8S, IAM, AUTOM, WFLOW, etc.) as practical guides for implementing each roadmap initiative.

## References

- [Support policy (Dynatrace)](https://www.dynatrace.com/company/trust-center/support-policy/) — OneAgents and ActiveGates: 9 months Standard Support, 12 months Enterprise Support; *"Timeframes indicated commence upon version release date."*
- [OneAgent platform and capability support matrix (DT docs)](https://docs.dynatrace.com/docs/ingest-from/technology-support/oneagent-platform-and-capability-support-matrix) — *"Preview features aren't production-ready and they aren't officially supported."*
- [End of support announcements (DT docs)](https://docs.dynatrace.com/docs/whats-new/technology/end-of-support-news) — *"End of support announcements are provided six months in advance."*
- [Primary Grail fields and tags enrichment through OneAgent (DT docs)](https://docs.dynatrace.com/docs/ingest-from/dynatrace-oneagent/oneagent-attribute-enrichment) — *"OneAgent version 1.333"*
- [Infrastructure Monitoring (DT docs)](https://docs.dynatrace.com/docs/license/capabilities/app-infra-observability/infrastructure-monitoring) — *"Infrastructure Monitoring consumption is measured in host hours"*

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
