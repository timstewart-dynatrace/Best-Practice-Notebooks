# SLO-05: SLOs as Code

> **Series:** SLO — Service Level Objectives | **Notebook:** 5 of 6 | **Created:** June 2026 | **Last Updated:** 08/31/2026

## Overview

A validated SLO that lives only in the UI is one accidental click from gone, and impossible to reproduce across environments. This notebook covers promoting SLOs into version control: the `dynatrace_platform_slo` Terraform resource and SLO Service Public API for the modern app, the classic `dynatrace_slo_v2`/`builtin:monitoring.slo` path it replaces (blocked at upgrade), the Monaco alternative, and the API path — with a deliberate emphasis on verifying field names at the source rather than copying a payload that may be stale.

---

## Table of Contents

1. [Why Version SLOs](#why)
2. [The Schema and Terraform Resource](#schema)
3. [Terraform Worked Example](#terraform)
4. [Monaco Alternative](#monaco)
5. [API and CI/CD](#api)

---

## Prerequisites

| Requirement | Details |
|-------------|---------|
| **Dynatrace Environment** | SaaS Gen3 with the SLO app |
| **Tooling** | Terraform with the `dynatrace-oss/dynatrace` provider, or Monaco |
| **Auth** | Platform Token or OAuth client per the AUTOM-04 auth-scheme guidance |
| **Prior reading** | SLO-02 (the SLI query you will codify), AUTOM-04 / AUTOM-07 (provider auth, CI/CD) |

<a id="why"></a>
## 1. Why Version SLOs

SLOs are configuration, and configuration that matters belongs in source control:

- **Reproducible across environments** — the same SLO in dev, staging, and prod from one definition.
- **Reviewable** — a target change goes through a PR, not a quiet UI edit.
- **Recoverable** — an accidental deletion is a `terraform apply` away from restored.

The workflow mirrors AIOPS-02's stance on detectors: **prototype in the app, promote to code once it matters.**

![SLOs as code](images/05-slo-as-code-flow_930x500.png)

<!-- MARKDOWN_TABLE_ALTERNATIVE
| Step | Action |
|------|--------|
| 1 Prototype | Validate the SLI DQL in the SLO app + a notebook |
| 2 Codify | dynatrace_slo_v2 or builtin:slo (Monaco) |
| 3 Review & apply | PR + plan in CI, terraform apply |
| 4 Live SLO | Versioned, reproducible per environment |
For environments where SVG doesn't render
-->

<a id="schema"></a>
## 2. The Schema and Terraform Resource

| Surface | Modern SLO app | SLO Classic |
|---------|----------------|-------------|
| API | **SLO Service Public API** | Service-level Objectives API classic (`/api/v2/slo`) — **blocked at upgrade** |
| Terraform resource | **`dynatrace_platform_slo`** (provider v1.78.0+; OAuth client with `slo:slos:read` / `slo:slos:write`) | `dynatrace_slo_v2` (classic `slo.*` + `settings.*` token scopes) |
| Monaco | supported since Monaco v2.22 | `builtin:monitoring.slo` via `--settings-schema` — **blocked at upgrade** |
| Settings 2.0 schema | none — the SLO service is the store | `builtin:monitoring.slo` |

> **What "blocked at upgrade" rests on (checked 08/28/2026).** The phrase is the **tenant upgrade readiness scan's** classification, read 07/31/2026 — not documentation. No Dynatrace page restates it; the published upgrade guidance is softer: *"An automated upgrade flow is under consideration; however, due to the highly customized nature of SLOs, a manual review is expected to deliver the best results."* A readiness scan reporting what a tenant will not carry across and a docs page describing a manual migration are answering different questions, so they can both be right — but they imply different urgency. **Run the scan against your own tenant** before planning the work.

> **Corrected 07/31/2026.** This section previously presented `builtin:monitoring.slo` / `dynatrace_slo_v2` as *the* config-as-code path. They are the **classic** generation. `dynatrace_slo_v2` authenticates with classic `slo.read`/`slo.write` plus `settings.read`/`settings.write` and writes through `builtin:monitoring.slo`; the modern app has its own SLO Service Public API, reached by `dynatrace_platform_slo` with OAuth `slo:slos:*` scopes. The readiness scan flags both classic surfaces as blocked at upgrade, so codifying new SLOs as `dynatrace_slo_v2` puts them on a path that stops working. Verify argument names at the provider docs before writing HCL — the note below applies to `dynatrace_platform_slo` too.

**The DQL-SLI gap was a property of the classic resource, and `dynatrace_platform_slo` closes it.** Read the paragraph below as the reason to move rather than as a live constraint: the `dynatrace_slo_v2` resource's SLI field (`metric_expression`) takes a **classic metric-selector expression** — e.g. `100*(builtin:service.requestCount.server:splitBy())/(...)` — not the DQL query with an `sli` field that a Custom SLO in the app uses (SLO-01 §3, SLO-02). If your SLI is built from a pre-aggregated metric, this maps over directly, and Dynatrace's own docs recommend metric-based SLOs for performance and cost where a suitable metric exists. If your SLI is genuinely DQL-only — a span-ratio or log/bizevent-derived `sli`, as in SLO-02's latency and custom examples — there is no documented Terraform field for it today; codify those via the export-a-known-good-object path in Section 5 instead of hand-writing HCL.

> **Verify the exact argument names against the provider docs before you write HCL.** The provider schema evolves between releases, and copying a stale payload is precisely how the field-authored alerting document ended up with an SLO API body that would not apply. The example below reflects the resource's documented schema as fetched at source on 07/01/2026 — treat the [resource docs](https://registry.terraform.io/providers/dynatrace-oss/dynatrace/latest/docs/resources/slo_v2) as authoritative going forward, since the schema can still change between provider releases.

```terraform
# dynatrace_slo_v2 — schema verified at the provider registry 07/01/2026
# Metric-based SLI (classic metric-selector syntax) — see the gap callout in Section 2
# regarding DQL-only SLIs, which this resource does not accept directly.
resource "dynatrace_slo_v2" "web_availability_30d" {
  name               = "Web Service Availability - 30d"
  custom_description = "Request success ratio for the critical web service, rolling 30 days"
  enabled            = true
  evaluation_type    = "AGGREGATE"

  # rolling 30-day window
  evaluation_window = "-30d"

  # entitySelector syntax (not a DQL filter) — scopes the SLO to web services
  filter = "type(SERVICE),serviceType(WEB_SERVICE,WEB_REQUEST_SERVICE)"

  # classic metric-selector expression — good / total as a percentage
  metric_expression = "100*(builtin:service.successCount:splitBy())/(builtin:service.requestCount:splitBy())"
  metric_name       = "web_availability_30d"

  target_success = 99.5
  target_warning = 99.9

  # required block — burn-rate visualization + optional fast-burn threshold (SLO-04)
  error_budget_burn_rate {
    burn_rate_visualization_enabled = true
    fast_burn_threshold             = 14
  }
}
```

<a id="terraform"></a>
## 3. Terraform Worked Example — Notes

A few things the example encodes:

- **`target_success` and `target_warning`** are the goal and the early-warning line from SLO-01 §4 (note the argument names — not the shorter `target`/`warning` an earlier draft of this notebook assumed). Warning above target surfaces "getting close" before an actual breach.
- **`metric_expression`** is a classic metric-selector string, not a DQL query — see the Section 2 gap callout. It is the codified form of a good/total ratio built from a pre-aggregated metric, analogous in intent to the SLO-02 DQL SLI but not the same syntax.
- **`filter`** uses entitySelector syntax (`type(SERVICE),serviceType(...)`), not a DQL filter clause and not quoted the way a DQL string literal would be. An unscoped SLO measures the whole environment, which is rarely what you want.
- **`error_budget_burn_rate` is a required block**, not optional — even if you don't need the visualization, the block itself must be present. `fast_burn_threshold` is optional within it and pairs with the SLO-04 burn-rate alerting recipe.
- **State and auth** follow the same rules as every other Dynatrace Terraform resource — see AUTOM-04 for Platform-Token vs classic-token routing and AUTOM-09 for state-backend setup.

<a id="monaco"></a>
## 4. Monaco Alternative

If your shop standardises on Monaco rather than Terraform, note first that **Monaco has supported the modern SLO Service Public API since v2.22** — prefer that over the classic settings object below, which targets the blocked `builtin:monitoring.slo` schema. The classic form is retained here because it is what existing Monaco projects contain, and because `monaco download` against it is how you inventory what needs moving:

```yaml
configs:
  - id: web-availability-30d
    type:
      settings:
        schema: builtin:monitoring.slo
        scope: environment
    config:
      template: web-availability-30d.json
      skip: false
```

The JSON template carries the SLI, target, warning, and window — and, unlike the Terraform resource, the Settings API payload behind a DQL-based Custom SLO (SLO-01 §3) is a first-class fit here, since Monaco pushes whatever JSON shape the schema accepts rather than mapping through the narrower Terraform resource attributes. As with Terraform, validate the SLI query in a notebook first, then export the working SLO from the app with `monaco download` to capture the exact current JSON shape rather than hand-writing it.

<a id="api"></a>
## 5. API and CI/CD

For direct API automation, use the **SLO Service Public API** — the modern app's own endpoint. This is not `/api/v2/slo`, which is the Service-level Objectives API *classic* and is flagged blocked at upgrade by the readiness scan; nor the older v1 SLO API. Rather than reproduce a request body that may drift, generate it from a working SLO:

1. Create and validate one SLO in the app.
2. Read it back via the API (or `monaco download` / `terraform-provider-dynatrace -export`) to capture the exact current schema.
   - **`dynatrace_platform_slo` will not appear in a default export.** Its resource documentation states: *"This resource is excluded by default in the export utility, please explicitly specify the resource to retrieve existing configuration."* Name it explicitly — `terraform-provider-dynatrace -export dynatrace_platform_slo` — or the step above returns nothing for exactly the resource you are trying to capture, which reads as "no SLOs are codified" rather than "this resource is opt-in".
3. Template that shape for the rest.

This "export a known-good object" approach is the antidote to stale-payload errors, and it is also the path of least resistance for a DQL-only SLI, since the Terraform resource's `metric_expression` field cannot carry one (Section 2). For the full CI/CD pattern — plan on PR, gated apply, three-way validation — see AUTOM-07 (pipelines) and AUTOM-96 (GitHub Actions LAB); the SLO resource slots into exactly the same pipeline as any other Dynatrace config.

A codified SLO and a codified deployment gate are two different resources answering two different questions — see **SLO-06** for the `dynatrace_site_reliability_guardian` counterpart to everything in this notebook, including a guardian objective that references an SLO directly rather than duplicating its query.

> <sub>**Sources:** [Service-Level Objectives (DT docs)](https://docs.dynatrace.com/docs/deliver/service-level-objectives), [dynatrace_slo_v2 resource (Dynatrace provider docs)](https://registry.terraform.io/providers/dynatrace-oss/dynatrace/latest/docs/resources/slo_v2), [dynatrace_platform_slo resource (Dynatrace provider docs)](https://registry.terraform.io/providers/dynatrace-oss/dynatrace/latest/docs/resources/platform_slo) — the modern resource this section recommends: *"covers configuration for platform service-level objectives"*, Dynatrace SaaS only, OAuth **View SLOs** (`slo:slos:read`) / **Create and edit SLOs** (`slo:slos:write`), and *"This resource is excluded by default in the export utility, please explicitly specify the resource to retrieve existing configuration."*, [platform_slo.md (Dynatrace GitHub)](https://github.com/dynatrace-oss/terraform-provider-dynatrace/blob/main/docs/resources/platform_slo.md) — the same resource doc at its source; the registry page renders client-side, so it is the repository copy that makes the two quotes above machine-verifiable. [SLO Service Public API (DT docs)](https://docs.dynatrace.com/docs/dynatrace-api/environment-api/service-level-objectives) — endpoints are `/slos` and `/objective-templates`; the classic surface is documented separately as *Service-level Objectives API classic*, [Upgrade from Service-Level Objectives Classic (DT docs)](https://docs.dynatrace.com/docs/deliver/service-level-objectives-classic/service-level-objective-upgrade-classic) — the version gates in §2, verbatim: *"Configuration as Code via Terraform overview support the SLO Service Public API since v1.78.0"* and *"Configuration as Code via Monaco overview supports the SLO Service Public API since v2.22."*, [Create service-level objectives (DT docs)](https://docs.dynatrace.com/docs/deliver/service-level-objectives/create-slo), [Monaco configuration (DT docs)](https://docs.dynatrace.com/docs/deliver/configuration-as-code/monaco). Terraform schema (`builtin:monitoring.slo`, `evaluation_window`, `target_success`/`target_warning`, required `error_budget_burn_rate` block) verified at the provider registry 07/01/2026; `dynatrace_platform_slo`'s schema, OAuth scopes and export exclusion re-verified at source 08/28/2026. **Readiness scan:** the "blocked at upgrade" classification is from the tenant upgrade readiness scan, read 07/31/2026; re-checked 08/28/2026 against both upgrade pages and the API references, where it is not restated. **Derived:** the metric-expression-vs-DQL-sli gap in Section 2 combines the provider's resource schema with the Create-SLO docs' DQL `sli`-field description — neither source states the gap explicitly.</sub>

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
