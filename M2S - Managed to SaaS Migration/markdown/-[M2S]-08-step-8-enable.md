# M2S-08: Step 8 — Enable: User Enablement and Communication

> **Series:** M2S — Managed to SaaS Migration | **Notebook:** 8 of 9 | **Phase:** Run | **Step:** Enable | **Created:** March 2026 | **Last Updated:** 07/17/2026

A successful migration is measured not by the technical cutover but by whether every team in the organization can use the new platform effectively. Step 8 focuses on communication, training, documentation, and establishing the support structures that ensure adoption. Without deliberate enablement, teams will struggle with new URLs, unfamiliar interfaces, and unanswered questions — undermining the value of the migration.

> **M2S Migration Journey — 3 Phases / 9 Steps**
>
> **Plan:** 1. Discover | 2. Strategize | 3. Design
>
> **Upgrade:** 4. Prepare | 5. Execute | 6. Integrate
>
> **Run:** 7. Expand | **8. Enable** | 9. Optimize

---

## Table of Contents

1. [Communication Plan](#communication-plan)
2. [Persona-Based Training](#persona-based-training)
3. [Documentation Updates](#documentation-updates)
4. [Self-Service Resources](#self-service-resources)
5. [Support Channels](#support-channels)
6. [Measuring Enablement Success](#measuring-enablement-success)
7. [Step Completion Checklist](#step-completion-checklist)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Steps 1–7 Complete** | Discovery, strategy, design, preparation, execution, integration, and expansion phases finished |
| **SaaS Tenant Stable** | All OneAgents reporting, integrations reconnected, SaaS-exclusive features enabled |
| **IAM Configured** | User accounts, groups, and policies set up in SaaS |
| **Stakeholder List** | Complete inventory of Dynatrace users by role and team |
| **Communication Channels** | Access to organizational email, Slack/Teams, intranet |

<a id="communication-plan"></a>

## 1. Communication Plan

The migration affects every Dynatrace user in the organization. A structured communication plan ensures no team is surprised, confused, or left behind. Tailor the message to the audience — executives need a different message than SRE engineers.

### Communication Timeline

| When | Audience | Message | Channel |
|------|---------|---------|----------|
| **Pre-migration (2–4 weeks before)** | All stakeholders | Migration timeline, what changes, what stays the same, how to prepare | Email + town hall |
| **During migration** | Platform team | Wave status updates, issues found, resolution progress | Slack/Teams war room |
| **Day of cutover** | All users | New URLs, login process, where to get help | Email + Slack/Teams announcement |
| **Post-migration (+1 week)** | All users | Feature highlights, training schedule, FAQ link | Email + intranet |
| **Post-migration (+2 weeks)** | All users | Adoption metrics, success stories, remaining training sessions | Email |
| **Post-migration (+30 days)** | Management | Migration summary, adoption dashboard, next steps (Step 9) | Executive briefing |

### Audience-Specific Messaging

Each audience cares about different things. Craft messages that answer their specific questions:

| Audience | Key Questions | Message Focus |
|---------|--------------|----------------|
| **Executives** | What’s the ROI? Is it done? | Business value, cost savings, reduced operational burden |
| **Platform Admins** | How do I manage it? What’s different? | New admin workflows, IAM policies, Settings 2.0, Grail buckets |
| **SRE/Operations** | Where are my alerts? How do I troubleshoot? | New URLs, Notebooks replacing USQL, Workflow-based alerting |
| **Developers** | Does my pipeline still work? Where’s my data? | API endpoint changes, tracing, log analysis, deployment markers |
| **Security Team** | Is it compliant? Where are the audit logs? | Application security, audit logging, compliance dashboards |

### Communication Templates

#### Pre-Migration Email (Example)

```
Subject: Dynatrace Platform Migration — Moving to SaaS [Timeline]

Team,

We are migrating our Dynatrace monitoring platform from Managed (self-hosted)
to SaaS (cloud-hosted). This migration brings improved performance, automatic
updates, and access to new capabilities including Grail data lakehouse,
Notebooks, and the AutomationEngine.

What changes:
- New URL: https://{tenant}.apps.dynatrace.com (replaces your current bookmark)
- Login: SSO via your corporate identity provider
- Query language: DQL replaces USQL for advanced queries

What stays the same:
- Your dashboards, management zones, and alerting rules are migrated
- OneAgent monitoring continues without interruption
- All historical data is preserved during the transition period

Timeline: [Insert dates]
Training: [Insert schedule link]
Questions: [Insert support channel]
```

#### Post-Migration Email (Example)

```
Subject: Dynatrace Migration Complete — Your New Platform is Live

Team,

The Dynatrace migration to SaaS is complete. Your new environment is live at:

  https://{tenant}.apps.dynatrace.com

Action required:
1. Update your bookmarks to the new URL
2. Log in using SSO (same corporate credentials)
3. Verify your dashboards and alerts are working

Resources:
- Quick Start Guide: [link]
- FAQ: [link]
- Support channel: #dynatrace-migration-support
- Training schedule: [link]

If anything looks wrong, report it in #dynatrace-migration-support immediately.
```

### Verify User Access to SaaS

Before sending the post-migration announcement, confirm that users can actually log in and see data:

```dql
// Verify IAM groups are configured — users need group membership to access SaaS
fetch dt.entity.user_session_synthetic, from:-24h
| summarize sessionCount = count()
| fieldsAdd status = if(sessionCount > 0, then: "User sessions detected", else: "No user sessions — verify IAM configuration")

```

<a id="persona-based-training"></a>

## 2. Persona-Based Training

One-size-fits-all training does not work. Each persona interacts with Dynatrace differently, so training must be tailored to their workflows. Deliver training within the first two weeks of migration to prevent users from forming bad habits or giving up on the platform.

### Training Matrix

| Persona | Training Focus | Key Topics | Duration | Format |
|---------|---------------|------------|----------|--------|
| **Platform Admins** | SaaS administration | IAM policies, Settings 2.0, Grail bucket management, OpenPipeline configuration, token management | 2 hours | Hands-on workshop |
| **SRE/Operations** | Monitoring and alerting | Notebooks, DQL queries, Workflow-based alerting, problem analysis, SLO management | 2 hours | Hands-on workshop |
| **Developers** | Observability integration | Distributed tracing in Notebooks, log analysis with DQL, deployment markers, release tracking | 1.5 hours | Hands-on workshop |
| **Management** | Executive dashboards | Dashboard navigation, SLO reporting, business analytics, sharing and scheduling | 1 hour | Demo + walkthrough |
| **Security Team** | Security features | Application Security module, vulnerability analysis, audit log queries, compliance dashboards | 1.5 hours | Hands-on workshop |

### What Changed: Managed vs. SaaS Key Differences

Training must address the specific changes users will encounter:

| Feature | Managed Experience | SaaS Experience |
|---------|-------------------|------------------|
| **URL** | `https://{managed-server}/e/{env-id}/` | `https://{tenant}.apps.dynatrace.com` |
| **Login** | Local users or LDAP | SSO via identity provider |
| **Navigation** | Classic UI | Unified platform UI with app launcher |
| **Query language** | USQL (limited) | DQL (full Grail access) |
| **Data analysis** | Custom charts, data explorer | Notebooks (interactive, shareable) |
| **Alerting** | Notification rules + alerting profiles | Workflows (AutomationEngine) |
| **Configuration** | Settings v1 | Settings 2.0 (schema-based) |
| **Data storage** | Elasticsearch + Cassandra | Grail data lakehouse |
| **Updates** | Manual cluster updates (downtime) | Automatic, zero-downtime updates |

### DQL Transition Guide

The biggest workflow change for power users is the transition from USQL to DQL. Include these common translations in training:

| USQL Pattern | DQL Equivalent |
|-------------|----------------|
| `SELECT * FROM usersession` | `fetch logs, from:-1h` |
| `WHERE country = 'US'` | `\| filter geo.country.name == "US"` |
| `GROUP BY application` | `\| summarize count = count(), by:{dt.entity.application}` |
| `ORDER BY duration DESC` | `\| sort duration desc` |
| `TOP 10` | `\| limit 10` |

> **Tip:** DQL is far more powerful than USQL. Frame the transition as an upgrade, not a disruption. Teams that learn DQL gain access to all Grail data sources (logs, spans, events, metrics, entities) through a single query language.

### Hands-On Training: First DQL Queries

Use these queries in training sessions to give users immediate hands-on experience with the SaaS platform:

```dql
// Training query 1: Explore recent log data (replaces USQL session queries)
fetch logs, from:-1h
| summarize logCount = count(), by:{loglevel}
| sort logCount desc
```

```dql
// Training query 2: View active detected problems
fetch dt.davis.problems, from:-24h
| filter event.status == "ACTIVE"
| fieldsKeep display_id, event.name, event.category, timestamp
| sort timestamp desc
| limit 10
```

```dql
// Training query 3: Host CPU overview (replaces Managed data explorer)
timeseries avgCpu = avg(dt.host.cpu.usage), from:-1h, by:{dt.entity.host}
| fieldsAdd avgCpuValue = arrayAvg(avgCpu)
| sort avgCpuValue desc
| limit 10
```

```dql
// Training query 4: Service response time analysis (distributed tracing)
fetch spans, from:-1h
| filter span.kind == "server"
| summarize avgDuration = avg(duration), p95Duration = percentile(duration, 95), requestCount = count(), by:{dt.entity.service}
| sort p95Duration desc
| limit 10
```

<a id="documentation-updates"></a>

## 3. Documentation Updates

Every internal document that references Dynatrace must be updated. Stale documentation with Managed URLs, old login procedures, or outdated screenshots will confuse users and generate unnecessary support requests.

### Documentation Inventory

| Document | Updates Required |
|----------|------------------|
| **Runbooks** | New SaaS URLs, updated troubleshooting procedures, revised escalation paths |
| **Architecture diagrams** | New network topology showing SaaS endpoints, ActiveGate placement, data flow |
| **Onboarding guides** | SaaS-specific login instructions, navigation walkthrough, first-day tasks |
| **Disaster recovery plans** | Updated recovery procedures (SaaS manages infrastructure; focus shifts to data and configuration backup) |
| **API documentation** | New base URLs (`{tenant}.live.dynatrace.com`), token management procedures |
| **Migration records** | Archive SaaS Upgrade Assistant deploy result CSVs for audit trail |
| **Security documentation** | Updated SSO configuration, token policies, audit log access |
| **ITSM integration guides** | Updated webhook URLs, ServiceNow/Jira connector configuration |

### URL Replacement Checklist

Search all documentation for these patterns and replace:

| Find | Replace With |
|------|--------------|
| `https://{managed-server}/e/{env-id}/` | `https://{tenant}.apps.dynatrace.com/` |
| `https://{managed-server}/e/{env-id}/api/v2/` | `https://{tenant}.live.dynatrace.com/api/v2/` |
| `https://{managed-server}/e/{env-id}/api/config/v1/` | `https://{tenant}.live.dynatrace.com/api/config/v1/` |
| `https://{managed-server}/api/cluster/v2/` | N/A — rewrite to use Account Management API |
| References to "Managed cluster" | "SaaS environment" |
| References to "cluster node" | Remove (SaaS manages infrastructure) |

> **Important:** Do not just replace URLs. Review the surrounding context. Some procedures (e.g., cluster restarts, Cassandra maintenance) no longer apply in SaaS and should be removed entirely.

<a id="self-service-resources"></a>

## 4. Self-Service Resources

Not every user will attend training. Self-service resources let users learn at their own pace and serve as a reference after training is complete. Create and share the following:

### Quick Start Guide

A one-page guide covering the first five minutes on the new platform:

1. **Log in** — Navigate to `https://{tenant}.apps.dynatrace.com` and authenticate via SSO
2. **Navigate** — Use the app launcher (left sidebar) to find Dashboards, Notebooks, Problems, Services
3. **Run your first query** — Open Notebooks > New Notebook > Add DQL cell > paste:
   ```
   fetch logs, from:-1h | summarize count = count(), by:{loglevel} | sort count desc
   ```
4. **Find your dashboard** — Go to Dashboards and search by name
5. **Check alerts** — Go to Problems to see active Dynatrace-detected issues

### DQL Cheat Sheet

Common queries that replace everyday Managed workflows:

| Task | DQL Query |
|------|-----------|
| Find error logs | `fetch logs, from:-1h \| filter loglevel == "ERROR" \| limit 50` |
| List active problems | `fetch dt.davis.problems, from:-24h \| filter event.status == "ACTIVE"` |
| Check host CPU | `timeseries avg(dt.host.cpu.usage), from:-1h, by:{dt.entity.host}` |
| View service requests | `fetch spans, from:-1h \| filter span.kind == "server" \| summarize count(), by:{dt.entity.service}` |
| Count log volume | `fetch logs, from:-24h \| summarize count = count(), by:{log.source} \| sort count desc` |
| Find slow transactions | `fetch spans, from:-1h \| filter span.kind == "server" \| filter duration > 5000000000 \| limit 20` |

### Frequently Asked Questions

Publish answers to the questions you know users will ask:

| Question | Answer |
|----------|--------|
| Where do I log in? | `https://{tenant}.apps.dynatrace.com` — use your corporate SSO credentials |
| What happened to my dashboards? | Migrated via SaaS Upgrade Assistant — find them in Dashboards (use search) |
| How do I query data now? | DQL in Notebooks replaces USQL. See the DQL Cheat Sheet above |
| Where are my management zones? | Migrated — find them in Settings > Ownership and Permissions > Management Zones |
| Why do I see fewer problems? | Dynatrace Intelligence is re-establishing baselines on the new SaaS environment. This takes 7–14 days. Problem detection will normalize |
| Can I still use the old Managed URL? | During the transition period, yes (read-only). After decommission, no |
| Where did USQL go? | USQL is not available in SaaS. Use DQL in Notebooks — it is more powerful and covers all data sources |
| How do I create a new API token? | Go to Account Management > Identity & Access Management > OAuth clients, or use Settings > Access Tokens for environment tokens |
| Why does the UI look different? | SaaS uses the unified platform UI. The app launcher on the left provides access to all features |
| Who do I contact for help? | Use the #dynatrace-migration-support channel (Slack/Teams) or attend weekly office hours |

### Video Walkthroughs

Record short (5–10 minute) screen recordings covering:

| Topic | Audience | Content |
|-------|---------|----------|
| SaaS login and navigation | All users | SSO login, app launcher, finding dashboards |
| Creating your first Notebook | SRE/Developers | New notebook, DQL cells, visualization, sharing |
| Workflow-based alerting | SRE/Operations | Creating a workflow trigger, conditional routing, Slack notification |
| IAM and token management | Platform Admins | Groups, policies, environment tokens, OAuth clients |
| Dashboard management | Management | Finding dashboards, sharing, scheduling PDF exports |

<a id="support-channels"></a>

## 5. Support Channels

Establish clear support channels before the migration goes live. Users need to know exactly where to go when they have questions. Ambiguity leads to frustration and shadow IT workarounds.

### Support Channel Matrix

| Channel | Purpose | Owner | Response Time |
|---------|---------|-------|---------------|
| **Slack/Teams channel** (`#dynatrace-migration-support`) | Day-to-day questions, quick troubleshooting | Platform team | < 2 hours during business hours |
| **Office hours** (weekly 30-min drop-in) | Live Q&A, screen sharing, hands-on help | Platform team | Scheduled |
| **Jira/ServiceNow** | Formal requests: access issues, configuration changes, feature requests | SRE team | Per SLA |
| **Dynatrace University** | Self-paced learning, certifications | Individual | Self-service |
| **Dynatrace Community** | Peer support, best practices, shared queries | Community | Best-effort |
| **Dynatrace Support** | Platform issues, bugs, outage investigation | Dynatrace | Per contract SLA |

### Escalation Path

Define a clear escalation path so support requests do not get stuck:

```
User question
  → Slack/Teams channel (platform team triages)
    → If access issue: IAM admin resolves
    → If configuration issue: Platform team resolves
    → If platform bug: Dynatrace Support ticket
    → If feature request: Backlog (Jira/ServiceNow)
```

### Transition Support Timeline

Support intensity should decrease over time as users become self-sufficient:

| Period | Support Level | Activities |
|--------|-------------|-------------|
| **Week 1–2** | High (hyper-care) | Daily office hours, immediate Slack response, proactive check-ins |
| **Week 3–4** | Medium | Bi-weekly office hours, same-day Slack response |
| **Month 2–3** | Standard | Weekly office hours, standard SLA response |
| **Month 4+** | Business as usual | Monthly office hours, standard support channels |

<a id="measuring-enablement-success"></a>

## 6. Measuring Enablement Success

Enablement is not complete when training is delivered — it is complete when users are actively and effectively using the platform. Measure adoption with concrete metrics and set targets that demonstrate real usage.

### Enablement Metrics

| Metric | Target | How to Measure |
|--------|--------|----------------|
| **Active users (weekly)** | ≥ Managed baseline within 30 days | IAM audit logs |
| **Notebooks created** | 10+ in the first month | Notebook listing / document count |
| **DQL queries executed** | Increasing week-over-week trend | Query audit (if enabled) |
| **Support tickets** | Decreasing trend after week 2 | ITSM system |
| **Training attendance** | 80%+ of target personas | Training records |
| **Dashboard views** | ≥ Managed baseline | Dashboard analytics |
| **Workflow automations created** | At least 5 within first month | Workflow listing |
| **Managed environment logins** | Declining to zero | Managed access logs |

### Tracking Active Users

Monitor platform adoption using audit events. If audit logging is enabled, query for active user sessions:

```dql
// Track daily active users over the past 7 days (requires audit log access)
fetch events, from:-7d
| filter event.type == "AUDIT_LOG"
| summarize activeUsers = countDistinct(user), by:{day = bin(timestamp, 1d)}
| sort day desc
```

```dql
// Track audit events by category — shows which platform features are being used
fetch events, from:-7d
| filter event.type == "AUDIT_LOG"
| summarize eventCount = count(), by:{event.category}
| sort eventCount desc
| limit 15
```

### Monitoring Data Flow Health

Verify that the platform is healthy and data is flowing — users cannot adopt a platform that is not working correctly:

```dql
// Verify log ingestion is healthy — compare volume to expected baseline
fetch logs, from:-24h
| makeTimeseries logVolume = count(default: 0), interval: 1h
```

```dql
// Count monitored hosts — should match or exceed Managed baseline
fetch dt.entity.host
| summarize hostCount = count()

// Alternative: Smartscape on Grail (entity.name → name)
// smartscapeNodes HOST
// | summarize hostCount = count()

```

```dql
// Count monitored services — compare to Managed inventory
fetch dt.entity.service
| summarize serviceCount = count()

// Alternative: Smartscape on Grail (entity.name → name)
// smartscapeNodes SERVICE
// | summarize serviceCount = count()

```

### Adoption Dashboard

Build a migration adoption dashboard that tracks these metrics over time. Share it with management as evidence that the migration is succeeding. Key tiles:

| Tile | Visualization | Data Source |
|------|--------------|-------------|
| Active users per day | Line chart | Audit log events |
| Monitored hosts | Single value | Entity count |
| Monitored services | Single value | Entity count |
| Log ingestion volume | Line chart | Log count timeseries |
| Problems detected | Line chart | detected problems timeseries |
| Managed logins remaining | Line chart (target: zero) | Managed access logs |

<a id="step-completion-checklist"></a>

## 7. Step Completion Checklist

Do not proceed to Step 9 (Optimize) until all items are confirmed.

| Checkpoint | Status |
|-----------|--------|
| Pre-migration communication sent to all stakeholders | [ ] |
| Post-migration announcement sent with new URLs and login instructions | [ ] |
| Platform admin training delivered | [ ] |
| SRE/operations training delivered | [ ] |
| Developer training delivered | [ ] |
| Management dashboard walkthrough completed | [ ] |
| Security team training delivered | [ ] |
| All runbooks updated with SaaS URLs and procedures | [ ] |
| Architecture diagrams updated to reflect SaaS topology | [ ] |
| Onboarding guides updated for SaaS | [ ] |
| API documentation updated with new base URLs | [ ] |
| Disaster recovery plans revised for SaaS model | [ ] |
| Quick Start Guide published and shared | [ ] |
| DQL Cheat Sheet published and shared | [ ] |
| FAQ published and shared | [ ] |
| Support channels established and announced | [ ] |
| Escalation path documented and communicated | [ ] |
| Enablement success metrics baselined | [ ] |
| Active user count meets or exceeds Managed baseline | [ ] |

---

## Next Step

> **M2S-09: Step 9 — Optimize** — Validate the migration, optimize SaaS configuration, decommission the Managed environment, and establish ongoing operational excellence.

### Additional Resources

- [Dynatrace University](https://university.dynatrace.com/) — Self-paced learning and certifications
- [Dynatrace Community](https://community.dynatrace.com/) — Peer support and shared best practices
- [DQL Documentation](https://docs.dynatrace.com/docs/platform/grail/dynatrace-query-language) — Full DQL reference
- [Notebooks Documentation](https://docs.dynatrace.com/docs/observe-and-explore/dashboards-and-notebooks/notebooks) — Interactive data exploration
- [Workflows Documentation](https://docs.dynatrace.com/docs/platform-modules/automations) — AutomationEngine for alerting and remediation
- [IAM Documentation](https://docs.dynatrace.com/docs/manage/identity-access-management) — User and access management in SaaS

---

## Summary

In Step 8, you:

- Executed a structured communication plan targeting all stakeholders with audience-specific messaging
- Delivered persona-based training for platform admins, SRE/operations, developers, management, and security
- Updated all internal documentation (runbooks, architecture diagrams, onboarding guides, API docs) to reflect SaaS
- Published self-service resources including a Quick Start Guide, DQL Cheat Sheet, and FAQ
- Established support channels with clear escalation paths and a hyper-care schedule
- Defined and baselined enablement success metrics to track adoption

> **Key Takeaway:** The migration is only as successful as the enablement effort behind it. Technology changes are straightforward; people changes require communication, training, and ongoing support. Invest in enablement now to avoid months of low adoption, frustrated users, and shadow IT workarounds.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
