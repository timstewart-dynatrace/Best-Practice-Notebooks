# ONBRD-99: Best Practice Summary

> **Series:** ONBRD — Dynatrace Onboarding | **Reference:** 99 — Best Practice Summary | **Created:** March 2026 | **Last Updated:** 07/01/2026

## Overview

Reference card for an architect or tenant lead onboarding a new 2026 Dynatrace tenant: recommended defaults, the key decision matrix, validation queries, anti-patterns, and the cross-series next-steps map.

For the **sequence + dependencies + per-step decisions**, see [ONBRD-00 Architect's Sequence & Dependency Runbook](-[ONBRD]-00-architect-sequence.ipynb).

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
| Used alongside | [ONBRD-00](-[ONBRD]-00-architect-sequence.ipynb) for sequence + dependencies |

<a id="recommended-defaults"></a>
## 1. Recommended Defaults (2026)

A new 2026 tenant should default to these choices unless there is a specific reason not to.

| Area | Default | Why |
|---|---|---|
| API token | **Platform Token** (`dt0s16`) with `Authorization: Bearer` | Sprint-1.337 default; aligns with Gen3 IAM model. Classic API Tokens (`dt0c01`, `Authorization: Api-Token`) only for legacy paths. |
| Configuration | **Settings v2** / Configuration as Code (Terraform `dynatrace_settings`, Monaco v2) | Sprint-1.337 announced Configuration API endpoints have Settings v2 equivalents. Plan automation around Settings v2. |
| Extensions | **Extensions 3rd-gen** (managed via Dynatrace API Application → Extensions) | Only recommended path for new customers. |
| Tagging at source | **Primary fields/tags at OneAgent install** via `oneagentctl --set-host-tag="primary_tags.<key>=<value>"` and `--set-host-tag="dt.security_context=<value>"` (single tags-hub form, June 2026; prefix written explicitly) | OneAgent attribute enrichment (1.331+) emits these on every signal at ingest. |
| Boundary field | **`dt.security_context`** for data + IAM scoping | Gen3 standard; segments + this field replace legacy Management Zones. |
| IAM policies | **Parameterized policies** bound to groups via binding parameters | Avoids N-copies-of-similar-policy maintenance burden. |
| K8s deployment | **Dynatrace Operator + Cloud Native FullStack** | Classic FullStack deprecated for new deployments. |
| K8s CRD baseline | **DynaKube v1beta5 baseline; v1beta6 canary** | v1beta5 is field-tested baseline; v1beta6 for newer features (DB extensions preview) once canary-validated. |
| Alerting | **Workflows + Davis Anomaly Detectors** | Alerting Profiles + metric-events being phased out. |
| Logs | **OpenPipeline** | Future extension versions install ingest assets requiring updated configuration API (Nov 2025+). |
| Windows OneAgent | **Npcap** for network insight (sprint-1.338+) | Replaces legacy WinPcap. |

<a id="decision-matrix"></a>
## 2. Decision Matrix

Decisions an architect makes during onboarding, with default and when to deviate.

| Decision | Default | When to deviate |
|---|---|---|
| ActiveGate yes/no | Yes if >500 hosts, hybrid/on-prem, or cloud-API polling at scale | Pure SaaS-only with no on-prem footprint can defer until cloud-API polling demands it |
| ActiveGate sizing | 10–20 GB; 2–3 AGs per zone | Larger only when load testing shows it is needed |
| Per-cloud integration | **Clouds app** for AWS (GA), Azure (preview); legacy GCP integration via AG (GCP Clouds-app new connections will follow soon) | If existing CloudWatch / Azure Monitor pipelines exist, evaluate migration effort vs leaving in place |
| OneAgent rollout sequence | Pilot 5–10 hosts → expand by host group → full | Skip pilot if you have onboarded the same workload class on another tenant |
| Tag taxonomy | env / team / app / `dt.security_context` / `dt.cost.costcenter` | Add compliance dimensions (`*-pci`, `*-pii`) only if a hard audit boundary exists |
| Host group naming | `<env>-<app>` or `<env>-<workload-type>` per FAQ-01 | Customer-specific patterns acceptable; avoid `all-hosts` / `default` / hardware-trait names |
| Bucket strategy | `dt.security_context` for general data access; **buckets only for compliance / retention / hard cost / hostile multi-tenancy** | See FAQ-02 for the buckets-vs-`dt.security_context` decision |
| OTel coexistence | OneAgent for instrumentation; OTel for app-code spans / business events / serverless | Per FAQ-03 OneAgent vs OpenTelemetry decision framework |
| Workflow trigger | Davis problems (default); custom DQL-event triggers (advanced) | Custom DQL triggers when business-event-driven alerting is required |
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

Workflows running; Davis surfacing problems with the expected scopes:

```dql
// Davis problems detected (last 7 days)
fetch dt.davis.problems, from:-7d
| filter event.status == "ACTIVE" or event.status == "CLOSED"
| summarize { c = count() }, by:{event.status}
```

<a id="anti-patterns"></a>
## 4. Anti-Patterns to Avoid

| Anti-pattern | Why it bites later |
|---|---|
| Single host group (`all-hosts` / `default`) | Per-host-group thresholds, alerting profiles, and IAM policies become impossible to scope |
| Setting tags retroactively (after install) | Primary-tag-at-install propagation gets skipped; signals before the retroactive setting do not carry the tag |
| Many copies of similar IAM policies (one per group with hardcoded scope) | Maintenance grows with org churn; use parameterized policies bound via binding parameters |
| Legacy Management Zones for new tenant | MZ-on-calculated-metrics is on the May-2026 deprecation list; segments + `dt.security_context` are the modern path |
| Auto-tagging rules as the primary tagging strategy | Computed at view time, not at ingest; doesn't propagate to logs / business events uniformly |
| `fetch dt.entity.*` queries in new content | Modern path is `smartscapeNodes "<TYPE>"`; `dt.entity.*` works but is deprecated |
| Buckets used as the primary access-control mechanism | Buckets are for compliance / retention / hard cost / hostile multi-tenancy; default for general data access is `dt.security_context` |
| Classic FullStack on Kubernetes | Deprecated for new deployments; use Cloud Native FullStack |
| Configuration API for new automation | Settings v2 equivalents exist as of sprint-1.337; plan new pipelines on Settings v2 / Terraform / Monaco v2 |
| Naming host groups by hardware traits (`4cpu-hosts`, `region-only`) | Encodes properties that change; rename burden as workloads scale |

<a id="where-to-go-deeper"></a>
## 5. Where to Go Deeper — Topic Series Map

ONBRD covers the foundation. These topic series cover each domain in depth:

| Domain | Topic Series | Notebooks |
|--------|--------------|-----------|
| **IAM administration** | IAM | 13 |
| **Data organization (buckets, segments, security context)** | ORGNZ | 11 |
| **Frequently asked questions** | FAQ | growing collection |
| **OpenPipeline log processing** | OPLOGS | 9 |
| **Classic Logs → OpenPipeline migration** | OPMIG | 10 |
| **OpenPipeline beyond logs (spans, metrics, events, bizevents)** | OPIPE | 7 |
| **Distributed tracing & spans** | SPANS | 9 |
| **Kubernetes monitoring & DynaKube** | K8S | 15 |
| **Cloud integration deep dives (AWS, Azure, GCP)** | CLOUD | 9 |
| **OpenTelemetry integration** | OTEL | 9 |
| **Workflows & alert notifications** | WFLOW | 10 |
| **Dynatrace Intelligence (Causal/Predictive/Generative AI, Davis)** | AIOPS | 8 |
| **Dashboard strategy & executive reporting** | DASH | 8 |
| **Configuration automation & GitOps** | AUTOM | 11 |
| **Management Zone → Policy migration** | MZ2POL | 10 |
| **Web Real User Monitoring** | WEBRUM | 9 |
| **Native mobile monitoring** | MOBL | 13 |
| **Synthetic monitoring** | SYNTH | 7 |
| **Business events & funnel analytics** | BIZEV | 7 |
| **Database monitoring** | DBMON | 7 |
| **Platform maturity & adoption roadmap** | ADOPT | 6 |
| **Managed → SaaS migration** | M2S | 10 |
| **SaaS → SaaS migration** | S2S | 11 |
| **New Relic → Dynatrace (procedural runbook)** | NR2DT | 11 |
| **New Relic → Dynatrace (component deep dives)** | NRLC | 9 |
| **Sumo Logic → Dynatrace** | SL2DT | 10 |
| **Splunk → Dynatrace** | S2D | 10 |

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against the current [Dynatrace documentation](https://docs.dynatrace.com/docs).*</sub>
