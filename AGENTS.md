# AGENTS.md — Best-Practice-Notebooks

Guidance for AI agents consuming this repository. Humans: see [README.md](README.md).

This is a **content-only** repository: 32 series of Dynatrace best-practice
notebooks (~307 documents, ~725k words). There is no build system, no tests,
and no application code.

## How to navigate (important — do not crawl)

The corpus is far too large to read wholesale. Route instead:

1. Match the user's question to a series in the map below.
2. Open that series' `AGENTS.md` — it contains a routing table mapping question
   types to individual notebooks.
3. Read only the notebook(s) it names. Each is self-contained (~2–5k words)
   with a table of contents, prerequisites, and fenced `dql` / `yaml` / `bash`
   blocks you can quote directly.
4. For broad "where do I start" / adoption-journey questions, use
   [`-START-HERE-/`](-START-HERE-/README.md) — a playbook that sequences series
   by scenario (net-new, expand/consolidate, deployment migration,
   classic → Gen3 platform) and includes a series overlap map.

## Formats — read only markdown

Every series has the same layout. Only one format is for you:

| Path | Purpose | Agents |
|---|---|---|
| `markdown/` | Canonical text of every notebook | **Read this** |
| `AGENTS.md` | Per-series routing table | **Read this** |
| `README.md` | Human overview + import instructions | Fallback index |
| `notebooks/` | Dynatrace notebook JSON for tenant import | Only when the user asks to import |
| `pdfs/`, `markdown/images/` | Print/visual duplicates | Ignore |

Subdirectory casing varies: three series (ALERT, APPSEC, SLO) use `NOTEBOOKS/`
and `PDFs/` instead of lowercase. `markdown/` — the one directory you read — is
lowercase in every series.

## Series map

Paths are the literal directory names (they contain spaces — quote them in shell).

**Foundations & Adoption**

| Prefix | Directory | Scope |
|---|---|---|
| ONBRD | `ONBRD - Dynatrace Onboarding` | Step-by-step onboarding for new Dynatrace users |
| ADOPT | `ADOPT - Observability Adoption & Maturity` | Maturity model, health assessment, success metrics, enablement |
| IAM | `IAM - IAM Administration` | Enterprise identity and access management administration |
| ORGNZ | `ORGNZ - Organize Data: Buckets, Segments, Security` | Grail buckets, segments, security context |
| FAQ | `FAQ - Frequently Asked Questions` | Cross-series Q&A |
| FINOPS | `FINOPS - Cost Management & FinOps` | Understanding, forecasting, optimizing DPS consumption |

**Data Sources & Instrumentation**

| Prefix | Directory | Scope |
|---|---|---|
| K8S | `K8S - Kubernetes Monitoring` | Operator/DynaKube deployment, GitOps, cluster/workload monitoring, K8s DQL, troubleshooting |
| CLOUD | `CLOUD - Cloud Provider Integrations` | AWS, Azure, GCP integrations |
| MOBL | `MOBL - Mobile Monitoring` | Mobile RUM: iOS, Android, cross-platform frameworks |
| WEBRUM | `WEBRUM - Web Real User Monitoring` | Web RUM, session replay, client-side performance |
| SYNTH | `SYNTH - Synthetic Monitoring` | Synthetic monitors: HTTP, browser, network |
| DBMON | `DBMON - Database Monitoring` | SQL, NoSQL, cache, and messaging platform monitoring |
| OTEL | `OTEL - OpenTelemetry Integration` | OTLP ingest, collectors, OneAgent coexistence |

**Data Processing & Analytics**

| Prefix | Directory | Scope |
|---|---|---|
| OPLOGS | `OPLOGS - OpenPipeline Logs` | Log ingest, parsing, routing with OpenPipeline |
| OPIPE | `OPIPE - OpenPipeline Beyond Logs` | OpenPipeline for spans, metrics, business/security events |
| SPANS | `SPANS - Distributed Tracing and Spans` | Working with traces and spans |
| BIZEV | `BIZEV - Business Events & Funnel Analysis` | Business events, funnels |
| DASH | `DASH - Dashboard Design & Building` | Dashboard design for stakeholder audiences |

**Automation & Workflows**

| Prefix | Directory | Scope |
|---|---|---|
| AUTOM | `AUTOM - Dynatrace Automation` | Configuration and operations automation |
| WFLOW | `WFLOW - Workflows and Alert Notifications` | Workflows, notification routing |
| AIOPS | `AIOPS - Dynatrace Intelligence` | Causal, predictive, and generative AI capabilities |
| ALERT | `ALERT - Alerting Strategy and Design` | End-to-end alerting design; orchestrates AIOPS + SLO + WFLOW |
| SLO | `SLO - Service Level Objectives` | SLIs, error budgets, burn-rate alerting, SLOs as code |

**Security**

| Prefix | Directory | Scope |
|---|---|---|
| APPSEC | `APPSEC — Application Security` | Runtime Vulnerability Analytics, Runtime Application Protection, posture management (note: em dash in directory name) |

**Migrations**

| Prefix | Directory | Scope |
|---|---|---|
| M2S | `M2S - Managed to SaaS Migration` | Dynatrace Managed → SaaS |
| S2S | `S2S - SaaS to SaaS Migration` | Between Dynatrace SaaS environments |
| S2D | `S2D - Splunk to Dynatrace Migration` | Splunk → Dynatrace |
| SL2DT | `SL2DT - Sumo Logic to Dynatrace` | Sumo Logic → Dynatrace |
| NR2DT | `NR2DT - New Relic to Dynatrace Migration Steps` | New Relic → Dynatrace: process, discovery → cutover |
| NRLC | `NRLC - New Relic to Dynatrace Migration Deep Dives` | NR2DT companion: query translation, dashboards, alerting, validation reference |
| OPMIG | `OPMIG - OpenPipeline Migration` | Classic log processing → OpenPipeline |
| MZ2POL | `MZ2POL - Management Zone to Policy Migration` | Management Zones → policy-based access control (notebooks 01-04, 06-08) and → segments for filtering (notebook 05) |

Disambiguation for the overlapping clusters:
- **OpenPipeline**: logs → OPLOGS; other signal types → OPIPE; migrating from classic pipelines → OPMIG.
- **New Relic**: "how do we migrate" (process) → NR2DT; "how do I translate this NRQL/dashboard/alert" (reference) → NRLC.
- **Alerting**: strategy/design → ALERT; anomaly detectors and AI → AIOPS; notification plumbing → WFLOW; reliability targets → SLO.

## Conventions and rules

- **Read-only.** Never edit, "fix", or reformat content files. If you find an
  error, report it to the user as a suggestion for the upstream author.
- Notebook filenames follow `-[PREFIX]-NN-name.md` — the literal brackets and
  leading dash require quoting in shell commands.
- Entity queries prefer `smartscapeNodes` over legacy `fetch dt.entity.*`;
  where both appear, the fetch form is the deprecated alternative. Preserve
  that preference when quoting queries.
- Each notebook header carries **Created / Last Updated** dates — surface them
  when currency matters.
- Cite sources by series and number (e.g. "per K8S-09 Troubleshooting") so
  users can open the full document or import the matching notebook JSON.
- Content is AI-generated from community-submitted and public sources and is
  **not officially supported by Dynatrace**. For version-sensitive claims
  (operator versions, GA dates, pricing), advise verifying against official
  Dynatrace documentation.
