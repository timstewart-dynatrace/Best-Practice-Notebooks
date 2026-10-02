# ONBRD-99: Best Practice Summary

> **Series:** ONBRD — Dynatrace Onboarding | **Reference:** 99 — Best Practice Summary | **Created:** March 2026 | **Last Updated:** 10/02/2026

## Overview

Reference card for an architect or tenant lead onboarding a new 2026 Dynatrace tenant: recommended defaults, the key decision matrix, validation queries, anti-patterns, and the cross-series next-steps map.

For the **sequence + dependencies + per-step decisions**, see **ONBRD-00: Architect's Sequence & Dependency Runbook**.

---

## Table of Contents

1. [Recommended Defaults (2026)](#recommended-defaults)
2. [Decision Matrix](#decision-matrix)
3. [Validation Queries](#validation-queries)
4. [Anti-Patterns to Avoid](#anti-patterns)
5. [Where to Go Deeper — Topic Series Map](#where-to-go-deeper)

---

## Prerequisites

| Requirement | Details |
|---|---|
| Audience | Architect or tenant lead onboarding a new 2026 Dynatrace tenant |
| Used alongside | **ONBRD-00: Architect's Sequence & Dependency Runbook** for sequence + dependencies |

<a id="recommended-defaults"></a>
## 1. Recommended Defaults (2026)

A new 2026 tenant should default to these choices unless there is a specific reason not to.

| Area | Default | Why |
|---|---|---|
| API token | **Platform Token** (`dt0s16`) with `Authorization: Bearer` | Latest environments have platform tokens only: *"Classic access tokens don't exist in latest environments, and v2/apiTokens isn't available."* Classic access tokens (`dt0c01`, `Authorization: Api-Token`) exist only in Classic and hybrid environments. |
| Configuration | **Settings v2** / Configuration as Code (Terraform `dynatrace_settings`, Monaco v2) | SaaS 1.337: *"Many Configuration API endpoints are now covered by the Settings endpoints in the Environment API v2."* The affected Configuration API endpoints are deprecated. Plan automation around Settings v2. |
| Extensions | **Extensions 2.0** | Current extensions framework; EF1.0 end of support 2025-03-31. JMX and PMI EF1 extensions were kept past that date but are deprecated: *"As of July 1, 2027, all Extension Framework 1.0 JMX and PMI extensions will be out of support for SaaS Environments."* Migrate them during onboarding. |
| Tagging at source | **Primary fields/tags at OneAgent install** via `oneagentctl --set-host-tag="primary_tags.<key>=<value>"` and `--set-host-tag="dt.security_context=<value>"` (single tags-hub form, June 2026; prefix written explicitly) | OneAgent attribute enrichment (OneAgent 1.333+) emits these on every signal at ingest. |
| Boundary field | **`dt.security_context`** for data + IAM scoping | Gen3 standard; segments + this field replace legacy Management Zones. |
| IAM policies | **Parameterized policies** bound to groups via binding parameters | Avoids N-copies-of-similar-policy maintenance burden. |
| K8s deployment | **Dynatrace Operator + Cloud Native FullStack** | *"Classic Full-Stack mode is not supported when using a platform token"*, the default credential above; ONBRD-05 covers the mode choice. |
| K8s CRD baseline | **DynaKube `v1beta6`** for new DynaKubes | Operator 1.11.0 removes `v1beta4` and says *"Before upgrading, update all DynaKube manifests to v1beta6."* `v1beta5` is still served by 1.11.0 but marked deprecated in its CRD (ONBRD-05). |
| Alerting | **Workflows** (simple workflows for notification) + **custom alerts** for metric conditions | Alerting profiles and problem notifications are Dynatrace Classic: *"They continue to work and are not being removed on a published schedule, but they'll not receive new capabilities."* Build new alerting on workflows (ONBRD-09). |
| Logs | **OpenPipeline** | The Latest Dynatrace path for log processing; OPMIG covers moving off classic log processing, OPLOGS the pipelines themselves. |
| Windows OneAgent | **Npcap** for network insight (OneAgent 1.337+) | Required for Network Agent metrics; WinPcap is no longer supported. |

> <sub>**Sources:**</sub>
> - <sub>[Authentication (DT docs)](https://docs.dynatrace.com/docs/dynatrace-api/basics/dynatrace-api-authentication) — *"dt0s16 Platform Token enabling programmatic access to Dynatrace platform services."*; [Upgrade from classic access tokens (DT docs)](https://docs.dynatrace.com/docs/platform/upgrade/set-up-your-environment/upgrade-from-access-tokens-classic)</sub>
> - <sub>[SaaS 1.337 (DT docs)](https://docs.dynatrace.com/docs/whats-new/saas/sprint-337)</sub>
> - <sub>[EF1 JMX and PMI extensions end of support (DT docs)](https://docs.dynatrace.com/docs/ingest-from/extensions/end-of-support/jmx-pmi-ef1-deprecation); [End-of-support news (DT docs)](https://docs.dynatrace.com/docs/whats-new/technology/end-of-support-news) — *"Note that JMX and PMI Extensions Framework 1.0 are supported past March 2025 but are deprecated."*</sub>
> - <sub>[Classic Full-Stack monitoring (DT docs)](https://docs.dynatrace.com/docs/ingest-from/setup-on-k8s/how-it-works/other-deployment-modes/classic-fullstack); [Operator 1.11.0 release notes (DT docs)](https://docs.dynatrace.com/docs/whats-new/dynatrace-operator/dto-fix-1-11-0); `v1beta5` `deprecated: true` read from the DynaKube CRD in the Operator v1.11.0 `kubernetes.yaml` ([Operator releases (Dynatrace GitHub)](https://github.com/Dynatrace/dynatrace-operator/releases)), 10/02/2026</sub>
> - <sub>[Upgrade guide: alert notifications (DT docs)](https://docs.dynatrace.com/docs/platform/upgrade/keep-problems-and-alerting-working/upgrade-guide-alert-notification)</sub>

<a id="decision-matrix"></a>
## 2. Decision Matrix

Decisions an architect makes during onboarding, with default and when to deviate.

| Decision | Default | When to deviate |
|---|---|---|
| ActiveGate yes/no | Recommended if >500 hosts (ONBRD-03); needed for hybrid/on-prem or cloud-API polling at scale | Pure SaaS-only with no on-prem footprint can defer until cloud-API polling demands it |
| ActiveGate sizing | 10–20 GB **disk** per ActiveGate (ONBRD-03); CPU and memory per FAQ-10; 2–3 AGs per network zone | Larger only when load testing shows it is needed |
| Per-cloud integration | **Clouds app** for AWS (GA) and Azure (SaaS 1.337+, no AG for metric polling); GCP Clouds-app connection in **Preview** — verify it has reached your tenant; the classic AG-based GCP integration remains the working path until then | If existing CloudWatch / Azure Monitor pipelines exist, evaluate migration effort vs leaving in place |
| OneAgent rollout sequence | Pilot 2–5 non-critical hosts (ONBRD-05) → expand by host group → full | Skip pilot if you have onboarded the same workload class on another tenant |
| Tag taxonomy | env / team / app / `dt.security_context` / `dt.cost.costcenter` | Add compliance dimensions (`*-pci`, `*-pii`) only if a hard audit boundary exists |
| Host group naming | `<env>-<app>` or `<env>-<workload-type>` per FAQ-01 | Customer-specific patterns acceptable; avoid `all-hosts` / `default` / hardware-trait names |
| Bucket strategy | `dt.security_context` for general data access; **buckets only for compliance / retention / hard cost / hostile multi-tenancy** | See ORGNZ-02 (Grail buckets) and ORGNZ-99 for the buckets-vs-`dt.security_context` decision |
| OTel coexistence | OneAgent for instrumentation; OTel for app-code spans / business events / serverless | Per FAQ-03 OneAgent vs OpenTelemetry decision framework |
| Workflow trigger | Problem trigger (default); Event trigger with a DQL filter (advanced) | Custom DQL triggers when business-event-driven alerting is required |
| Dashboard tier | Operations + Executive (mandatory); Engineering (recommended) | Skip Executive if there is no exec stakeholder for this tenant |
| Notification channel for new tenant | Slack / email / PagerDuty silent channel for first 1–2 weeks | Switch to live channels only after dual-alert window validates volume parity |

<a id="validation-queries"></a>
## 3. Validation Queries

A small DQL set that verifies each phase complete. Save these as a notebook in the new tenant — they are the architect's smoke-test for the deployment.

### Phase 1 — Foundation complete

Hosts reporting (Gate G1):

```dql
// Hosts reporting (Gate G1 — Foundation -> Organize)
smartscapeNodes "HOST"
| summarize host_count = count()

// Caveat: Smartscape HOST nodes can leave out PaaS-only hosts. On a validation tenant
// (10/02/2026) this returned 7 while fetch dt.entity.host returned 15; the 8 missing were
// Kubernetes application-only and AWS Fargate hosts (paasVendorType KUBERNETES /
// AWS_ECS_FARGATE). For those rollouts, count with the classic entity instead:
//   fetch dt.entity.host
//   | summarize host_count = count(), by: {paasVendorType}
```

```dql
// Services discovered
smartscapeNodes "SERVICE"
| summarize service_count = count()
```

### Phase 2 — Organize complete

Primary fields propagating; `dt.security_context` populated (Gate G2):

```dql
// dt.security_context populated on logs (Gate G2 — Organize -> Operate)
fetch logs, from:-1h
| summarize { c = count() }, by:{dt.security_context}
| sort c desc
```

```dql
// Primary tags propagating
fetch logs, from:-1h
| filter isNotNull(primary_tags.environment)
| summarize { c = count() }, by:{primary_tags.environment}
```

### Phase 3 — Operate complete

Davis surfacing problems, and workflows running:

```dql
// Davis problems detected (last 7 days)
fetch dt.davis.problems, from:-7d
| summarize { c = count() }, by:{event.status}
```

```dql
// Workflow runs by final state (last 7 days)
// Each run also writes a RUNNING record when it starts; filter to final states to count runs once
fetch dt.system.events, from:-7d
| filter event.kind == "WORKFLOW_EVENT" and event.type == "WORKFLOW_EXECUTION"
| filter in(dt.automation_engine.state, {"SUCCESS", "ERROR"})
| summarize { runs = count() }, by:{dt.automation_engine.workflow.title, dt.automation_engine.state}
| sort runs desc
```

<a id="anti-patterns"></a>
## 4. Anti-Patterns to Avoid

| Anti-pattern | Why it bites later |
|---|---|
| Single host group (`all-hosts` / `default`) | Per-host-group thresholds, alerting profiles, and IAM policies become impossible to scope |
| Setting tags retroactively (after install) | Primary-tag-at-install propagation gets skipped; signals before the retroactive setting do not carry the tag |
| Many copies of similar IAM policies (one per group with hardcoded scope) | Maintenance grows with org churn; use parameterized policies bound via binding parameters |
| Legacy Management Zones for new tenant | API 1.337 marks the `managementZone` (and `serviceTag`) properties of calculated service metrics deprecated; segments + `dt.security_context` + policies are the Latest Dynatrace path (MZ2POL) |
| Auto-tagging rules as the primary tagging strategy | Applied to entities by periodic background rule evaluation, which can lag (*"the application of all tags might be delayed!"*), not written onto records at ingest; doesn't propagate to logs / business events uniformly |
| `fetch dt.entity.*` queries in new content | Modern path is `smartscapeNodes "<TYPE>"`; `dt.entity.*` works but is deprecated |
| Buckets used as the primary access-control mechanism | Buckets are for compliance / retention / hard cost / hostile multi-tenancy; default for general data access is `dt.security_context` |
| Classic FullStack on Kubernetes | Not supported with platform tokens; use Cloud Native FullStack |
| Configuration API for new automation | Many Configuration API endpoints are deprecated in favour of Settings (SaaS 1.337); plan new pipelines on Settings v2 / Terraform / Monaco v2 |
| Naming host groups by hardware traits (`4cpu-hosts`, `region-only`) | Encodes properties that change; rename burden as workloads scale |

> <sub>**Sources:** [Dynatrace API 1.337 (DT docs)](https://docs.dynatrace.com/docs/whats-new/dynatrace-api/sprint-337) — `POST /calculatedMetrics/service`: *"Changed property managementZone Deprecated changed to true"*; [Settings API: automatically applied tags schema (DT docs)](https://docs.dynatrace.com/docs/dynatrace-api/environment-api/settings/schemas/builtin-tags-auto-tagging) — *"Tagging rules are executed periodically in the background, for a limited timeframe."*; [Classic Full-Stack monitoring (DT docs)](https://docs.dynatrace.com/docs/ingest-from/setup-on-k8s/how-it-works/other-deployment-modes/classic-fullstack); [SaaS 1.337 (DT docs)](https://docs.dynatrace.com/docs/whats-new/saas/sprint-337).</sub>

<a id="where-to-go-deeper"></a>
## 5. Where to Go Deeper — Topic Series Map

ONBRD covers the foundation. These topic series cover each domain in depth:

| Domain | Topic Series | Notebooks |
|--------|--------------|-----------|
| **IAM administration** | IAM | 15 |
| **Data organization (buckets, segments, security context)** | ORGNZ | 11 |
| **Frequently asked questions** | FAQ | growing collection |
| **OpenPipeline log processing** | OPLOGS | 9 |
| **Classic Logs → OpenPipeline migration** | OPMIG | 10 |
| **OpenPipeline beyond logs (spans, metrics, events, bizevents)** | OPIPE | 7 |
| **Distributed tracing & spans** | SPANS | 9 |
| **Kubernetes monitoring & DynaKube** | K8S | 15 |
| **Cloud integration deep dives (AWS, Azure, GCP)** | CLOUD | 9 |
| **OpenTelemetry integration** | OTEL | 9 |
| **Workflows & alert notifications** | WFLOW | 12 |
| **Alerting strategy & design** | ALERT | 5 |
| **Service level objectives** | SLO | 6 |
| **Dynatrace Intelligence (Causal/Predictive/Generative AI, Davis)** | AIOPS | 8 |
| **Dashboard strategy & executive reporting** | DASH | 8 |
| **Configuration automation & GitOps** | AUTOM | 14 |
| **Management Zone → Policy migration** | MZ2POL | 11 |
| **Web Real User Monitoring** | WEBRUM | 10 |
| **Native mobile monitoring** | MOBL | 13 |
| **Synthetic monitoring** | SYNTH | 7 |
| **Business events & funnel analytics** | BIZEV | 8 |
| **Database monitoring** | DBMON | 7 |
| **Application security** | APPSEC | 10 |
| **Cost management & FinOps** | FINOPS | 3 |
| **Platform maturity & adoption roadmap** | ADOPT | 7 |
| **Managed → SaaS migration** | M2S | 11 |
| **SaaS → SaaS migration** | S2S | 12 |
| **New Relic → Dynatrace (procedural runbook)** | NR2DT | 11 |
| **New Relic → Dynatrace (component deep dives)** | NRLC | 9 |
| **Sumo Logic → Dynatrace** | SL2DT | 11 |
| **Splunk → Dynatrace** | S2D | 10 |

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against the current [Dynatrace documentation](https://docs.dynatrace.com/docs).*</sub>
