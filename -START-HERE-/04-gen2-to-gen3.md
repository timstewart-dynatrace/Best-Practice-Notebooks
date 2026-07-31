# Doorway 4 — Classic → Gen3 Platform

> **Purpose:** Reading order for existing Dynatrace SaaS customers moving off classic surfaces onto platform equivalents — management zones used as permissions, metric events, alerting profiles, classic dashboards, Classic Logs, USQL. Same tenant, same deployment model; what changes is which surface you operate.
> **Last Updated:** 07/31/2026

![Classic to Gen3 Migration Phases](images/04-gen2-to-gen3-phases.svg)

<!-- MARKDOWN_TABLE_ALTERNATIVE
| Phase | Classic surface | Gen3 surface |
|-------|-----------------|--------------|
| 0 | Classic UI, entity model | Grail, Smartscape, new navigation |
| 1 | Management zones as permissions | Policies, boundaries, security_context, buckets |
| 2 | Metric/entity selectors, USQL, Log Viewer | DQL, Smartscape queries, OpenPipeline |
| 3 | Classic dashboards, MZ filters | Gen3 dashboards, variables, segments |
| 4 | Metric events, alerting profiles, SLO Classic | Anomaly detectors, workflows, platform SLOs |
| 5 | Settings v1, manual config | Settings 2.0, Monaco, Terraform |
For environments where SVG doesn't render
-->

---

## Table of Contents

1. [You Are Here If…](#you-are-here-if)
2. [Why This Is Its Own Doorway](#why-this-is-its-own-doorway)
3. [Phase Overview](#phase-overview)
4. [The Readiness Scan](#the-readiness-scan)
5. [Phase 0 — Understand the Shift](#phase-0--understand-the-shift)
6. [Phase 1 — Access and Data Architecture](#phase-1--access-and-data-architecture)
7. [Phase 2 — Query Surfaces](#phase-2--query-surfaces)
8. [Phase 3 — Dashboards and App Surfaces](#phase-3--dashboards-and-app-surfaces)
9. [Phase 4 — Alerting and SLOs](#phase-4--alerting-and-slos)
10. [Phase 5 — Config-as-Code and Go-Live](#phase-5--config-as-code-and-go-live)
11. [Sequencing Rules](#sequencing-rules)
12. [Where to Next](#where-to-next)

---

<a id="you-are-here-if"></a>
## You Are Here If…

- You already have a working Dynatrace SaaS tenant, and you are **not** changing where it runs
- Your tenant still operates one or more classic constructs:
  - Management zones used for **access control** rather than filtering
  - **Metric events** and **alerting profiles** rather than anomaly detectors and workflows
  - **Classic dashboards** rather than Gen3 dashboards or notebooks
  - **Classic Logs** rather than OpenPipeline
  - **USQL** for session analysis, or classic **metric/entity selectors** rather than DQL
- You want the platform capabilities that only exist on the Gen3 side — Grail retention economics, Davis CoPilot, Workflows, segments, bucket-level access control

If you have no tenant yet, see [Doorway 1 — Net New](01-net-new.md). If you are adding scope to a tenant that is already Gen3-native, see [Doorway 2 — Expanding or Consolidating](02-expand-consolidate.md). If you are changing deployment model — Managed → SaaS or SaaS → SaaS — see [Doorway 3 — Deployment Migration](03-deployment-migration.md) first; it lands you on a Gen3 tenant, after which this doorway covers the surface-by-surface work.

---

<a id="why-this-is-its-own-doorway"></a>
## Why This Is Its Own Doorway

Doorways 1–3 are each keyed to a **change in circumstance** — a new tenant, new scope, a new deployment model. This one is keyed to a change in **surface**: your environment stays where it is, but the constructs you operate it with are replaced.

That distinction matters for sequencing. A deployment migration has a natural cutover date that forces the work into a schedule. A surface migration has none, so it tends to stall halfway — a tenant ends up with policies *and* management zones, workflows *and* alerting profiles, running in parallel indefinitely. Partial migration is more expensive than either end state, because every construct has to be maintained twice and every troubleshooting session starts by asking which surface owns the answer.

**One thing does move, though: your component fleet.** Several platform capabilities carry minimum OneAgent, ActiveGate, and Dynatrace Operator versions, and a deployment mode or image source that is merely legacy today can be a blocker tomorrow. These are not a phase — they gate every phase, and a tenant can work through all six and still not be upgradeable because its agents are below the floor. Phase 0 therefore opens with the fleet, not with the surfaces.

[FAQ](../FAQ%20-%20Frequently%20Asked%20Questions/) — entry 12 (coverage gaps from partial enablement) describes the same failure shape from the enablement angle and is worth reading before you plan the phases.

---

<a id="phase-overview"></a>
## Phase Overview

| Phase | What changes | Time |
|---|---|---|
| [0 — Understand the Shift](#phase-0--understand-the-shift) | Readiness scan; component version floors; Grail, Smartscape, navigation | 1 week to assess; fleet upgrades run longer, in parallel |
| [1 — Access and Data Architecture](#phase-1--access-and-data-architecture) | MZs → policies, boundaries, segments; tags; security_context; buckets | 4–8 weeks |
| [2 — Query Surfaces](#phase-2--query-surfaces) | Selectors and USQL → DQL; Classic Logs → OpenPipeline | 3–6 weeks |
| [3 — Dashboards and App Surfaces](#phase-3--dashboards-and-app-surfaces) | Classic dashboards → Gen3; MZ filters → variables and segments; classic domain apps → their Gen3 equivalents | 2–4 weeks |
| [4 — Alerting and SLOs](#phase-4--alerting-and-slos) | Metric events → anomaly detectors; alerting profiles → workflows; SLO Classic → platform SLOs | 3–6 weeks |
| [5 — Config-as-Code and Go-Live](#phase-5--config-as-code-and-go-live) | Settings 2.0, Monaco, Terraform, cutover | 2–4 weeks |

Estimates are calendar weeks for a small to mid-sized team, assuming the migration runs alongside normal operations rather than as a dedicated project. Phases 2–4 can overlap once Phase 1 is settled; Phase 1 cannot be parallelized with the rest — see [Sequencing Rules](#sequencing-rules).

---

<a id="the-readiness-scan"></a>
## The Readiness Scan

Dynatrace ships a ready-made dashboard, **Check your upgrade readiness**, to tenants on the latest SaaS. It is read-only, curated and auto-distributed by Dynatrace, and it scores your tenant across nineteen classic-surface domains plus a licensing gate. Open it from the ready-made documents in the Dashboards app.

Run it before you plan the phases. It replaces the guesswork in the inventory this doorway used to ask you to assemble by hand — but it answers *what* is outstanding, not *in what order* or *whether it is worth fixing at all*. That is what the rest of this page is for.

### Two different questions about a classic construct

The scan and most public documentation answer different questions, and a construct can fail one while passing the other:

| Question | What it means | Where it is answered |
|---|---|---|
| Is it **deprecated**? | Dynatrace has named a successor. An end-of-life date may or may not be published, and the construct keeps working meanwhile. | Product documentation, release notes |
| Is it **blocked at upgrade**? | The schema or endpoint stops answering once the tenant moves to the latest Dynatrace. | The readiness scan |

Several constructs are the second without being the first. Public documentation still describes some of them as classic-but-working, and that is accurate on the deprecation axis — so a tenant reading only the docs can conclude there is no deadline, while the scan flags the same construct as removed at upgrade.

**Plan against the stricter signal.** Where the two disagree, treat the scan as the constraint and the documentation as the explanation. Blocking is triggered by *your* tenant's upgrade rather than a published calendar date, so the timing is yours to choose — the scope is not.

### Read it in this order, not the order it renders

The scan leads with its cheapest signals — deprecated REST calls and classic-entity DQL — because those are the easiest to compute and the most visible. That is not the order to fix them in.

Working the scan top-to-bottom means rewriting your queries twice: once against today's field names, and again after bucket assignment and `security_context` land and change what those queries can see. The phase order on this page already encodes the dependency. Use the scan to populate the phases, not to sequence them.

### Where each finding goes

| Scan section | What a red light means | Phase | Start reading |
|---|---|---|---|
| DPS license | Not on the subscription the latest Dynatrace requires — everything else is moot until this is resolved | — | [FINOPS](../FINOPS%20-%20Cost%20Management%20&%20FinOps/) |
| Deprecated REST API & settings | A token, workflow, dashboard or detector calls something that stops answering | 5 | [AUTOM](../AUTOM%20-%20Dynatrace%20Automation/), [IAM](../IAM%20-%20IAM%20Administration/) |
| Classic entity model in DQL | A named document runs classic-entity DQL | 2 | [FAQ](../FAQ%20-%20Frequently%20Asked%20Questions/) — entry 16 |
| Management zone usage | The zone is actively queried, so it is real work rather than a stale object | 1 | [MZ2POL](../MZ2POL%20-%20Management%20Zone%20to%20Policy%20Migration/) |
| Classic RBAC roles | Groups still receive permissions through classic roles | 1 | [IAM](../IAM%20-%20IAM%20Administration/), [MZ2POL](../MZ2POL%20-%20Management%20Zone%20to%20Policy%20Migration/) |
| Classic app usage | Named users depend on an app that is being removed | 3 | [DASH](../DASH%20-%20Dashboard%20Design%20&%20Building/) |
| OpenPipeline adoption | Log or business-event volume still on the classic pipeline | 2 | [OPMIG](../OPMIG%20-%20OpenPipeline%20Migration/), [OPIPE](../OPIPE%20-%20OpenPipeline%20Beyond%20Logs/) |
| Log Classic | Ingest still landing in the classic store | 2 | [OPMIG](../OPMIG%20-%20OpenPipeline%20Migration/), [ORGNZ](../ORGNZ%20-%20Organize%20Data:%20Buckets,%20Segments,%20Security/) |
| Alerting | Metric-event alerts and classic disk detection to recreate | 4 | [ALERT](../ALERT%20-%20Alerting%20Strategy%20and%20Design/), [AIOPS](../AIOPS%20-%20Dynatrace%20Intelligence/) |
| Calculated service metrics | A metric key with no successor concept — needs a replacement plan, not a port | 2 | [OPIPE](../OPIPE%20-%20OpenPipeline%20Beyond%20Logs/), [BIZEV](../BIZEV%20-%20Business%20Events%20&%20Funnel%20Analysis/) |
| Service detection & rule settings | A rule scoped by management zone, service tag or non-primary process-group tag | 4 | [SPANS](../SPANS%20-%20Distributed%20Tracing%20and%20Spans/) |
| Classic cloud integrations | Classic AWS/Azure connections and the automation driving them | 1 | [CLOUD](../CLOUD%20-%20Cloud%20Provider%20Integrations/) |
| Cloud telemetry enrichment | Enrichment embedded in a connection rather than configured centrally | 1 | [CLOUD](../CLOUD%20-%20Cloud%20Provider%20Integrations/) |
| Infrastructure readiness | Operator, DynaKube mode, image source or agent version below the floor | 0 | [K8S](../K8S%20-%20Kubernetes%20Monitoring/), [ONBRD](../ONBRD%20-%20Dynatrace%20Onboarding/) |
| ActiveGate & network readiness | ActiveGate below the floor, bad auth token, multi-environment mode, or zones that will misroute | 0 | [ONBRD](../ONBRD%20-%20Dynatrace%20Onboarding/), [FAQ](../FAQ%20-%20Frequently%20Asked%20Questions/) — entry 10 |
| Cloud automations | A guardian objective pointing at a classic SLO, or a classic issue-tracking integration | 4 | [SLO](../SLO%20-%20Service%20Level%20Objectives/), [WFLOW](../WFLOW%20-%20Workflows%20and%20Alert%20Notifications/) |
| Digital experience | A frontend not yet on the Grail RUM experience | 3 | [WEBRUM](../WEBRUM%20-%20Web%20Real%20User%20Monitoring/), [MOBL](../MOBL%20-%20Mobile%20Monitoring/) |
| Synthetic monitoring | Private locations to redeploy, third-party monitors, unsupported maintenance windows | 3 | [SYNTH](../SYNTH%20-%20Synthetic%20Monitoring/) |
| Application Security | The new monitoring rules are off, so the current Vulnerabilities experience is unavailable | 4 | [APPSEC](../APPSEC%20—%20Application%20Security/) |
| Problem & event processing | Events that will not correlate, plus classic alerting objects still in place | 4 | [ALERT](../ALERT%20-%20Alerting%20Strategy%20and%20Design/), [WFLOW](../WFLOW%20-%20Workflows%20and%20Alert%20Notifications/) |

### Two findings that mean "retire", not "migrate"

Two of the scan's signals are usage counts rather than configuration checks, and they are the most valuable numbers on the page:

- **Management zone usage** counts how often each zone is actually queried. A zone nobody queries does not need a policy, a boundary, or a segment — it needs deleting.
- **Classic app usage** counts views *per user*. A classic dashboard with two viewers is a conversation; one with two hundred is a rebuild.

The scan reports both and draws no conclusion from either. Deciding retire-versus-migrate before you start is the single largest cost lever in this doorway, and Phase 0's deliverable is where it belongs.

### What the scan does not check

It reads configuration and usage, so it cannot see intent, ownership, or whether a construct is still needed. It is also read-only, which means you cannot diff its findings against your own configuration-as-code repository — for that, export your Monaco or Terraform inventory and compare by hand.

It does not cover the tagging and naming rationalization in Phase 1, the query-translation work in Phase 2 beyond flagging classic-entity DQL, or anything about whether your rebuilt dashboards and detectors are *good*. A clean scan means nothing is blocked. It does not mean the migration is finished.

---

<a id="phase-0--understand-the-shift"></a>
## Phase 0 — Understand the Shift

Before changing anything, find out where your fleet and your tenant actually stand, then understand what Grail changes about storage, retention, and access, and what Smartscape changes about the entity model.

| Step | Reading | Notes |
|---|---|---|
| 1. Run the readiness scan | [The Readiness Scan](#the-readiness-scan) | Produces the inventory the rest of this phase interprets |
| 2. Component fleet floor | [K8S](../K8S%20-%20Kubernetes%20Monitoring/), [ONBRD](../ONBRD%20-%20Dynatrace%20Onboarding/); [FAQ](../FAQ%20-%20Frequently%20Asked%20Questions/) — entries 04 and 05 (managing OneAgent and ActiveGate updates) | Agent, ActiveGate and Operator versions, DynaKube deployment mode, and image source. A blocker here outranks every surface below |
| 3. Storage and retention model | [ORGNZ](../ORGNZ%20-%20Organize%20Data:%20Buckets,%20Segments,%20Security/) — notebook 02 (understanding Grail buckets) | Buckets, retention, and what they cost |
| 4. Metrics mechanics | [FAQ](../FAQ%20-%20Frequently%20Asked%20Questions/) — entry 11 (how metrics work) | Covers the Classic ↔ Grail dual-write, the `builtin:` ↔ `dt.*` split, and selector → DQL conversion |
| 5. Partial-enablement risk | [FAQ](../FAQ%20-%20Frequently%20Asked%20Questions/) — entry 12 (coverage gaps from partial enablement) | Why a half-migrated tenant costs more than either end state |
| 6. Navigation and launchers | [FAQ](../FAQ%20-%20Frequently%20Asked%20Questions/) — entry 07 (launcher page setup) | The Gen3 app model and how users find things |
| 7. Where you are today | [ADOPT](../ADOPT%20-%20Observability%20Adoption%20&%20Maturity/) — notebook 02 (platform health assessment) | Baseline before you start moving surfaces |

Deliverable: the scan output, annotated. For every finding, record a decision — **migrate**, **retire**, or **not applicable** — and for anything you intend to migrate, the phase it belongs to. The scan tells you what is outstanding; this annotation is what turns it into a plan.

Most tenants over-estimate the work at this point. Many management zones turn out to be filtering conveniences rather than permission boundaries, which changes the Phase 1 path substantially, and a meaningful share of flagged classic dashboards and zones have no active users at all. Resolve those against the usage counts before you scope anything.

---

<a id="phase-1--access-and-data-architecture"></a>
## Phase 1 — Access and Data Architecture

The largest phase, and the one everything else depends on. Management zones split into three separate Gen3 concepts depending on the job they were doing.

| Step | Reading | Notes |
|---|---|---|
| 1. Understand the split | [MZ2POL](../MZ2POL%20-%20Management%20Zone%20to%20Policy%20Migration/) — notebooks 01 (why migrate), 02 (access control model) | Management zones did three jobs; Gen3 separates them |
| 2. Assess your MZs | [MZ2POL](../MZ2POL%20-%20Management%20Zone%20to%20Policy%20Migration/) — notebook 03 (assessment and planning) | Classify each MZ by job: access, filtering, or alerting scope |
| 3a. MZs used for **access** | [MZ2POL](../MZ2POL%20-%20Management%20Zone%20to%20Policy%20Migration/) — notebooks 04 (policies and boundaries), 08 (templated policies); [IAM](../IAM%20-%20IAM%20Administration/) — notebooks 04 (policy authoring), 05 (boundary design), 10 (templated policy assignments) | The bulk of the work for most tenants |
| 3b. MZs used for **filtering** | [MZ2POL](../MZ2POL%20-%20Management%20Zone%20to%20Policy%20Migration/) — notebook 05 (segments); [ORGNZ](../ORGNZ%20-%20Organize%20Data:%20Buckets,%20Segments,%20Security/) — notebooks 08 (Grail segments), 10 (advanced segment definitions) | If this is all your MZs do, notebook 05 alone may be the whole migration |
| 3c. MZs used for **alerting scope** | [MZ2POL](../MZ2POL%20-%20Management%20Zone%20to%20Policy%20Migration/) — notebook 09 (alerting and notification migration) | Hands off to Phase 4 |
| 4. Tagging rationalization | [FAQ](../FAQ%20-%20Frequently%20Asked%20Questions/) — entry 02 (tagging sources, standards, strategy); [ORGNZ](../ORGNZ%20-%20Organize%20Data:%20Buckets,%20Segments,%20Security/) — notebook 01 | Gen3 access control leans on tags far more than Gen2 did |
| 5. security_context | [ORGNZ](../ORGNZ%20-%20Organize%20Data:%20Buckets,%20Segments,%20Security/) — notebook 06 | The universal scope field; without it, IAM boundaries are limited |
| 6. Bucket strategy | [ORGNZ](../ORGNZ%20-%20Organize%20Data:%20Buckets,%20Segments,%20Security/) — notebooks 03 (bucket strategy and design), 05 (bucket-level access control) | Determines retention, cost, and what any given policy can see |
| 7. Cloud connections and enrichment | [CLOUD](../CLOUD%20-%20Cloud%20Provider%20Integrations/) — notebooks 01 (fundamentals), 02 (AWS), 03 (Azure) | Classic AWS/Azure connections are recreated rather than converted, and enrichment embedded in a connection moves to the central ingest-time configuration |
| 8. Execute and validate | [MZ2POL](../MZ2POL%20-%20Management%20Zone%20to%20Policy%20Migration/) — notebooks 06 (migration execution), 07 (validation and troubleshooting) | Run policies and MZs in parallel, then retire the MZs |
| Reference | [MZ2POL](../MZ2POL%20-%20Management%20Zone%20to%20Policy%20Migration/) — notebook 99; [ORGNZ](../ORGNZ%20-%20Organize%20Data:%20Buckets,%20Segments,%20Security/) — notebook 99 | One-page syntheses |

---

<a id="phase-2--query-surfaces"></a>
## Phase 2 — Query Surfaces

Every classic query language and selector has a DQL equivalent. This phase is mostly mechanical translation, with one structural workstream (Classic Logs).

| Step | Reading | Notes |
|---|---|---|
| 1. DQL grounding | [ORGNZ](../ORGNZ%20-%20Organize%20Data:%20Buckets,%20Segments,%20Security/) — notebook 99 (DQL reference); [SPANS](../SPANS%20-%20Distributed%20Tracing%20and%20Spans/) — query reference | Official Dynatrace DQL documentation is authoritative |
| 2. Metric selectors → `timeseries` | [FAQ](../FAQ%20-%20Frequently%20Asked%20Questions/) — entry 11 (how metrics work) | Covers selector → DQL conversion and the `builtin:` ↔ `dt.*` mapping |
| 3. Entity selectors → Smartscape | [FAQ](../FAQ%20-%20Frequently%20Asked%20Questions/) — entry 16 (migrating classic entity selectors to Smartscape) | `dt.entity.*` is deprecated in favour of `dt.smartscape.*` and `smartscapeNodes`; legacy form still runs |
| 4. USQL → DQL for RUM | [WEBRUM](../WEBRUM%20-%20Web%20Real%20User%20Monitoring/) — notebook 09 (migrating USQL to DQL) | Grammar and field mapping, plus the classic-RUM-on-Grail vs New RUM split that decides which field names apply |
| 5. Classic Logs → OpenPipeline | [OPMIG](../OPMIG%20-%20OpenPipeline%20Migration/) — full series; [OPLOGS](../OPLOGS%20-%20OpenPipeline%20Logs/) — full series | The one genuinely structural workstream here; treat as its own project |
| 6. Log metric extraction | [OPMIG](../OPMIG%20-%20OpenPipeline%20Migration/) — notebook 07 (metric event extraction) | Classic log metrics map onto OpenPipeline extraction |
| 7. Pipelines beyond logs | [OPIPE](../OPIPE%20-%20OpenPipeline%20Beyond%20Logs/) — full series; [BIZEV](../BIZEV%20-%20Business%20Events%20&%20Funnel%20Analysis/) — notebook 07 (Gen2 vs Gen3 adoption paths) | Business events, spans, metrics and events have their own classic processing surfaces, on the same removal path as the log ones |
| 8. Calculated service metrics | [OPIPE](../OPIPE%20-%20OpenPipeline%20Beyond%20Logs/) — notebook 01 (multi-scope platform) | These have no Gen3 equivalent concept — each one in use needs a replacement built on pipeline extraction, not a port |

---

<a id="phase-3--dashboards-and-app-surfaces"></a>
## Phase 3 — Dashboards and App Surfaces

Classic dashboards do not convert automatically. Treat this as a rebuild against a rationalized list, not a port — most tenants find a large fraction of classic dashboards are unused. The same phase covers the classic domain apps, which fold into their Gen3 equivalents rather than being rebuilt.

| Step | Reading | Notes |
|---|---|---|
| 1. Inventory and prune | [The Readiness Scan](#the-readiness-scan) — classic app usage, reported per dashboard and per user | The scan supplies the usage counts; retire anything with no active viewers rather than porting it |
| 2. Rebuild | [DASH](../DASH%20-%20Dashboard%20Design%20&%20Building/) — notebooks 01 (fundamentals), 02 (dashboard hierarchy), 03–05 (executive, operations, engineering) | Built on the Phase 2 DQL. Notebook 02 is the target structure to rebuild *into*, not an inventory method |
| 3. MZ filters → variables and segments | [DASH](../DASH%20-%20Dashboard%20Design%20&%20Building/) — notebook 06 (variables and filters); [ORGNZ](../ORGNZ%20-%20Organize%20Data:%20Buckets,%20Segments,%20Security/) — notebook 08 (Grail segments) | The dashboard-side half of Phase 1 step 3b |
| 4. Notebooks for investigation | [DASH](../DASH%20-%20Dashboard%20Design%20&%20Building/) — notebook 01; [AIOPS](../AIOPS%20-%20Dynatrace%20Intelligence/) — notebook 04 (Davis CoPilot and Dynatrace Assist) | Notebooks replace much of what classic ad-hoc dashboards were used for |
| 5. Distribution | [DASH](../DASH%20-%20Dashboard%20Design%20&%20Building/) — notebook 07 (sharing and reporting) | |
| 6. Frontend surfaces | [WEBRUM](../WEBRUM%20-%20Web%20Real%20User%20Monitoring/) — notebook 09 (migrating USQL to DQL); [MOBL](../MOBL%20-%20Mobile%20Monitoring/) — notebook 01 | Web and mobile frontends move onto the Grail RUM experience; the classic session apps fold into Users & Sessions. Custom application frontends have no Grail path yet — leave them where they are |
| 7. Synthetic surfaces | [SYNTH](../SYNTH%20-%20Synthetic%20Monitoring/) — notebooks 01 (fundamentals), 04 (private locations) | Private locations on Kubernetes are redeployed rather than upgraded in place; third-party monitors and alerting-suppression maintenance windows need checking separately |
| 8. Database surfaces | [DBMON](../DBMON%20-%20Database%20Monitoring/) — full series | Database Services Classic folds into the Databases app |

---

<a id="phase-4--alerting-and-slos"></a>
## Phase 4 — Alerting and SLOs

Metric events and alerting profiles map onto anomaly detectors and workflows, but rarely one-to-one — this is the phase where a lift-and-shift produces the worst results.

| Step | Reading | Notes |
|---|---|---|
| 1. Target architecture | [ALERT](../ALERT%20-%20Alerting%20Strategy%20and%20Design/) — notebook 01 (end-to-end architecture) | Read before migrating any individual alert |
| 2. Metric events → detection | [ALERT](../ALERT%20-%20Alerting%20Strategy%20and%20Design/) — notebook 02 (choosing and building detection); [AIOPS](../AIOPS%20-%20Dynatrace%20Intelligence/) — notebook 02 (anomaly detection) | Static thresholds ported as-is are the main source of Gen3 alert noise; many should become baselines |
| 3. Alerting profiles → workflows | [MZ2POL](../MZ2POL%20-%20Management%20Zone%20to%20Policy%20Migration/) — notebook 09; [WFLOW](../WFLOW%20-%20Workflows%20and%20Alert%20Notifications/) — notebooks 04 (notification routing), 05 (incident management) | MZ-scoped alerting profiles become problem-triggered workflows |
| 4. Routing and cost | [ALERT](../ALERT%20-%20Alerting%20Strategy%20and%20Design/) — notebook 03 (routing, destinations, cost) | Simple vs multi-step workflow billing differs — check before fanning out |
| 5. ITSM integration | [ALERT](../ALERT%20-%20Alerting%20Strategy%20and%20Design/) — notebook 04 (ServiceNow integration) | If alerting profiles fed ServiceNow, rebuild the payload rather than translating it |
| 6. SLO Classic → platform SLOs | [SLO](../SLO%20-%20Service%20Level%20Objectives/) — notebooks 01 (fundamentals), 05 (SLOs as code) | |
| 7. Application Security rules | [APPSEC](../APPSEC%20—%20Application%20Security/) — notebooks 01 (fundamentals), 02 (runtime vulnerability analytics) | The current Vulnerabilities experience depends on the newer monitoring rules being enabled. Enabling them turns the classic rules off, so treat it as a cutover with a rollback rather than an additive setting |
| 8. Auto-remediation | [WFLOW](../WFLOW%20-%20Workflows%20and%20Alert%20Notifications/) — full series; [AIOPS](../AIOPS%20-%20Dynatrace%20Intelligence/) — notebook 06 (integrations and agentic workflows) | Only available on the Gen3 side; a reason to finish the phase rather than stall in parallel |
| Reference | [ALERT](../ALERT%20-%20Alerting%20Strategy%20and%20Design/) — notebook 99 (checklist and cross-series map) | |

---

<a id="phase-5--config-as-code-and-go-live"></a>
## Phase 5 — Config-as-Code and Go-Live

Put the new configuration under version control before you retire the old surfaces, so the migration end state is reproducible.

| Step | Reading | Notes |
|---|---|---|
| 1. Automation landscape | [AUTOM](../AUTOM%20-%20Dynatrace%20Automation/) — notebook 01 | Which tool for which config surface |
| 2. Settings v1 → Settings 2.0 | [AUTOM](../AUTOM%20-%20Dynatrace%20Automation/) — notebook 02 (Settings API) | |
| 3. Monaco or Terraform | [AUTOM](../AUTOM%20-%20Dynatrace%20Automation/) — notebooks 03 (Monaco), 04 (Terraform), 09 (Terraform GitOps setup recipe) | |
| 4. IAM as code | [IAM](../IAM%20-%20IAM%20Administration/) — appendix LABs 95 (Terraform provisioning), 96 (Python provisioning) | Locks in the Phase 1 policy model |
| 5. Retire classic surfaces | *No cross-journey checklist yet* | Per-journey checklists exist in [M2S](../M2S%20-%20Managed%20to%20SaaS%20Migration/) — notebook 99 and [MZ2POL](../MZ2POL%20-%20Management%20Zone%20to%20Policy%20Migration/) — notebook 06 |
| 6. Optimize | [FINOPS](../FINOPS%20-%20Cost%20Management%20&%20FinOps/) — full series; [ADOPT](../ADOPT%20-%20Observability%20Adoption%20&%20Maturity/) — notebook 06 (staged enablement) | Grail retention and bucket choices are the main cost levers post-migration |

---

<a id="sequencing-rules"></a>
## Sequencing Rules

Four ordering constraints are worth respecting; the rest of the phases are flexible.

| Rule | Why |
|---|---|
| **Component versions before everything** | Agent, ActiveGate and Operator floors are a prerequisite rather than a phase. Fleet upgrades run on their own change-management calendar and are the longest-lead item here, so start them in Phase 0 and let the surface work proceed in parallel. |
| **Phase 1 before Phase 2** | Bucket assignment and `security_context` determine what any DQL query can see. Writing queries first means rewriting them after the boundaries land. |
| **Phase 2 before Phases 3 and 4** | Dashboards and detection are both built on queries. A dashboard rebuilt against pre-migration field names is rework. |
| **Phase 5 last, but started early** | Put configuration under version control as each phase completes rather than at the end — otherwise the migration end state exists only as manual tenant state. |

Phases 3 and 4 are independent of each other and can run in parallel by different teams.

**On running both surfaces in parallel:** a parallel window is correct and expected during each phase — it is the validation mechanism. What causes trouble is a parallel window with no end date. Set a retirement date for the classic construct when you start each phase, and treat the phase as incomplete until the old surface is switched off.

---

<a id="where-to-next"></a>
## Where to Next

- [Operationalize Module](07-operationalize.md) — once the surfaces are migrated, for the ongoing operations sequence
- [Maturity Module](08-maturity.md) — continuous improvement framing via [ADOPT](../ADOPT%20-%20Observability%20Adoption%20&%20Maturity/)
- [Domain Enablement Module](06-domain-enablement.md) — to add domains that were deferred during the migration
- [Overlap Map](09-overlap-map.md) — when the same topic appears in multiple series
- [Foundation Module](05-foundation.md) — if the assessment in Phase 0 turns up Foundation gaps rather than migration work

---

> *This playbook was AI-generated from community-submitted and publicly available sources. It is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*
