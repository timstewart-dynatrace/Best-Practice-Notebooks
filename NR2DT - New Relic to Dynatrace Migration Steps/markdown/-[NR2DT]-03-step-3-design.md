# NR2DT-03: Step 3 — Design

> **Series:** NR2DT — New Relic to Dynatrace Migration Steps | **Notebook:** 3 of 10 | **Created:** April 2026 | **Last Updated:** 08/27/2026

## Overview

**Goal of this step:** lock in the target Dynatrace architecture before any artifact migrates. Bucket strategy, host group taxonomy, IAM model, and OpenPipeline routing are all decided here.

Procedural runbook — see **FAQ-01** (host group naming strategy) and **ORGNZ-02 / ORGNZ-99** (Grail bucket strategy) for the recommended design patterns this step uses.

---

## Table of Contents

1. [What You'll Produce](#outputs)
2. [Bucket Design](#buckets)
3. [Host Group Design](#hostgroups)
4. [IAM Model](#iam)
5. [OpenPipeline Layout](#openpipeline)
6. [Step Exit Criteria](#gate)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Audience** | Migration lead + assigned engineer for this step |
| **Completed** | NR2DT-02 — Strategize |
| **Format** | Procedural step — use as a runbook; defer to NRLC for depth |
| **Topic deep dives** | FAQ-01 (host group naming), ORGNZ-02 / ORGNZ-99 (bucket strategy), IAM-04/05 (policy design), IAM-11 (WORKSHOP) |

<a id="outputs"></a>
## 1. What You'll Produce

| Artifact | Format | Purpose |
|----------|--------|---------|
| `bucket-design.md` | Markdown | Bucket names, retention, pricing model |
| `host-group-design.md` | Markdown | Host group taxonomy and naming |
| `iam-model.md` | Markdown | Groups, policies, scopes |
| `openpipeline-design.md` | Markdown | Enrichment + routing + filter rules |
| `terraform/` | HCL | Phase 1 Terraform module skeleton |

<a id="buckets"></a>
## 2. Bucket Design

Use the recommended pattern from **ORGNZ-02 — Understanding Grail Buckets** (with the consolidated reference in **ORGNZ-99**) as your starting template:

| Bucket | Retention | Pricing | Purpose |
|--------|-----------|---------|---------|
| `<org>_app_logs` | 30 days | Retain w/ Included Queries | Application logs |
| `<org>_infra_logs` | 14 days | Retain w/ Included Queries | Infrastructure / K8s |
| `<org>_security_logs` | 365 days | Usage-based | Audit trail / PCI |
| `<org>_spans` | 14 days | Retain w/ Included Queries | Distributed traces |
| `<org>_events` | 90 days | Retain w/ Included Queries | Platform events |
| `<org>_bizevents` | 90 days | Retain w/ Included Queries | Business events |

Use `<org>_` as the prefix — it lets IAM policies scope with one `STARTSWITH "<org>_"` condition.

Document any deviations from this template (e.g., per-app buckets) and why.

<a id="hostgroups"></a>
## 3. Host Group Design

Per **FAQ-01 — Host Group Naming Strategy**, group hosts by ownership boundary, not by team alone. Common dimensions:

- Environment (`prod-app`, `nonprod-app`, `prod-infra`)
- Compliance scope (`payment-pci`)
- Workload tier (`prod-batch`, `prod-stream`)

Plan a host group per (environment × ownership) combination. Avoid a single all-hosts group — it breaks per-environment thresholds, IAM scoping, and automation safety.

<a id="iam"></a>
## 4. IAM Model

Recommended starting groups:

| Group | Bucket Access | Policy Condition |
|-------|---------------|------------------|
| `<org>-admins` | All | `STARTSWITH "<org>_"` |
| `<org>-sre` | All operational | `bucket-name IN ('<org>_app_logs', '<org>_infra_logs', '<org>_events', '<org>_spans')` |
| `<org>-app-eng` | App logs only | `bucket-name == '<org>_app_logs'` |
| `<org>-readonly` | All except security | `STARTSWITH "<org>_" AND bucket-name != '<org>_security_logs'` |

Always include an explicit DENY on `<org>_security_logs` for non-admin / non-audit groups.

<a id="openpipeline"></a>
## 5. OpenPipeline Layout

Routing rules in priority order (first match wins):

| Priority | Matcher | Bucket |
|----------|---------|--------|
| 1 | `matchesValue(log.source, "*audit*") OR loglevel == "CRITICAL"` | `<org>_security_logs` |
| 2 | `in(k8s.namespace.name, {"kube-system", "kube-public", ...})` | `<org>_infra_logs` |
| 3 | App namespace match | `<org>_app_logs` |
| 4 | catch-all | `default_logs` (review weekly) |

**Enrichment rules** for any NR workload identifier (replaces NR's tag/workload model):

```yaml
matcher: contains(k8s.namespace.name, "prod")
field: environment
value: prod
```

**Drop rules** for cost optimization (decide which apply now vs. Wave 5):

- Drop health-check logs (`contains(content, "GET /health")`)
- Drop DEBUG logs in production
- Sample sidecar proxy logs

<a id="gate"></a>
## 6. Step Exit Criteria

**G3 — Architecture Locked**

- [ ] `bucket-design.md` final; bucket names + retention + pricing model documented
- [ ] `host-group-design.md` final; taxonomy understood by SRE + service owners
- [ ] `iam-model.md` final; security stakeholder signed off
- [ ] `openpipeline-design.md` final; routing rules sequenced
- [ ] Phase 1 Terraform module skeleton committed (will be applied in Wave 0 of Step 5)

**Next step:** **NR2DT-04 — Translate** (NRQL→DQL run; manual triage).

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources, including the open-source [NewRelic-to-Dynatrace-Migration-Utilities](https://github.com/timstewart-dynatrace/NewRelic-to-Dynatrace-Migration-Utilities), [nrql-engine](https://github.com/timstewart-dynatrace/nrql-engine), and [nrql-translator](https://github.com/timstewart-dynatrace/nrql-translator) projects. This notebook series is not officially supported by Dynatrace or New Relic. Always verify information against the official [Dynatrace documentation](https://docs.dynatrace.com/docs) and [New Relic documentation](https://docs.newrelic.com).*</sub>
