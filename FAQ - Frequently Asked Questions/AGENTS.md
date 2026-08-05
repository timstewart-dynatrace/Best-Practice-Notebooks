# AGENTS.md — FAQ: Frequently Asked Questions

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

22 standalone single-page reference entries, each answering one recurring
Dynatrace question in decision-support format. Entries are independent — there
is no reading order; match the question and read only that entry.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| Why host-group naming matters: ownership, access control, alert routing, automation scope, single-group risks | `-[FAQ]-01-host-group-naming-strategy.md` |
| Tagging: the four tag sources, primary tags/fields vs auto-tags, per-cloud specifics, taxonomy standards, anti-patterns | `-[FAQ]-02-tagging-sources-standards-strategy.md` |
| OneAgent vs OpenTelemetry: convert vs layer vs leave-alone vs greenfield, per-runtime coverage (Java/.NET/Node/Python/Go/PHP/Ruby), async context propagation | `-[FAQ]-03-oneagent-vs-otel-decision-framework.md` |
| OneAgent update modes, tenant/host-group/host precedence, update vs maintenance windows, DynaKube `autoUpdate` deprecation, rollback | `-[FAQ]-04-managing-oneagent-updates-saas.md` |
| ActiveGate updates: auto vs manual, ActiveGates-before-OneAgents sequencing, HA-pair rolling updates, role-specific validation | `-[FAQ]-05-managing-activegate-updates-saas.md` |
| Trusting Davis AI: data residency, model training boundary, hallucination controls, autonomy limits, audit trail, compliance posture | `-[FAQ]-06-can-we-trust-davis-ai.md` |
| Launcher pages / Launchpads: tenant default vs group vs personal precedence, admin mechanism, persona content examples | `-[FAQ]-07-launcher-page-setup.md` |
| Why a log file isn't being collected: auto-discovery three-gate rules, built-in include/exclude, custom log sources, scale limits | `-[FAQ]-08-oneagent-log-autodiscovery.md` |
| Metric vs raw-log queries: DPS query economics (log scans billed, metric reads included), recurring-vs-one-shot rule, extraction, cardinality | `-[FAQ]-09-metrics-instead-of-log-queries.md` |
| ActiveGate sizing: capability-to-dimension map, host/K8s/synthetic baselines, headroom and survivor-capacity math, `dt.sfm.active_gate.*` saturation signals, scale up vs out | `-[FAQ]-10-activegate-sizing-and-scaling.md` |
| How metrics work end to end: gauge/count data-point model, Metrics Classic vs Grail dual-write (`builtin:` vs `dt.*`), OTLP delta-only, rollup/retention, cardinality limits, DPS billing | `-[FAQ]-11-how-metrics-work.md` |
| Partial enablement after migrating from another tool: mode ladder (Discovery/Infrastructure/Full-Stack), dependency cascade, per-capability handicaps, coverage-audit DQL | `-[FAQ]-12-cost-of-coverage-gaps.md` |
| OpenShift pods rejected with `Forbidden: seccomp may not be set`: anyuid + Operator 1.9.0 default flip, SCC compatibility matrix, custom SCC fix, pre-upgrade audit | `-[FAQ]-13-openshift-scc-seccomp-injection.md` |
| Replacing custom SQL Server / Telegraf monitoring scripts with the Dynatrace extension: `sql-server.*` metric mapping, honest gaps (tempdb version store), EF 2.0 fallback | `-[FAQ]-14-sql-server-extension-vs-custom-scripts.md` |
| How DPL works: the pattern model, coming from regex (no backtracking, character classes as the only overlap), matcher catalog, why a pattern returns null or the wrong value, quantifiers, the five DQL surfaces (`parse` / `parseAll` / `matchesPattern` / `replacePattern` / `splitByPattern`), ingest-vs-query placement, failure diagnosis | `-[FAQ]-15-how-dpl-works.md` |
| Migrating classic entity queries to Smartscape: classifying the query first (mass-data filter vs entity list), `dt.entity.*` → `smartscapeNodes` type mapping, `classicEntitySelector`/`entityName`/`entityAttr` construct migration, `traverse` and edge discovery, host/process/container groups as fields rather than nodes, and the gotchas (`getNodeField` null inside `smartscapeNodes`, uppercase node vs lowercase edge types, `id_classic` as the id bridge) | `-[FAQ]-16-entity-selectors-to-smartscape.md` |
| Planning a migration cutover across any migration type: the eight cutover invariants (Go/No-Go gate, three-tier validation ladder, parallel-run window with an end date, T-minus runbook, rollback triggers-before-procedures, decommission gated on stabilization, stabilization window, lessons-learned), which series documents each best (SL2DT-09, S2S-09, NR2DT-08, OPMIG-09, MZ2POL-06), and live-verified entity-parity checks (`smartscapeNodes` inventory, `lifetime[end]` staleness) — orchestrates rather than restates | `-[FAQ]-17-planning-a-migration-cutover.md` |
| Monitoring AEM as a Cloud Service (AEMaaCS): the Dynatrace-published OneAgent full-stack integration on Adobe-operated containers — enablement via Adobe customer-care ticket (not self-service) and the exact payload it requires (environment URL, `PaaS integration - Installer download` token, ActiveGate port for Managed, AEM environment IDs); the Adobe/customer control boundary and which corpus guidance it invalidates (FAQ-04 agent updates, FAQ-01 host groups) versus what still applies (ORGNZ, IAM, ALERT, SLO); per-environment licensing figures; and the rule that enabling Dynatrace stops data flowing to a previously enabled APM tool, which removes the parallel-run window from FAQ-17's cutover model | `-[FAQ]-18-monitoring-aem-cloud-service.md` |
| Bringing a third-party SaaS platform's telemetry into Dynatrace (the reusable pattern): the three signal classes and which questions each answers, choosing an ingestion route from the vendor's transport (HTTPS direct vs collector hop vs API pull vs multi-homing an existing pipeline), signal-first extract-then-discard and the `Drop record`-vs-`No storage assignment` stage trap that silently defeats it, modeling vendor objects as `CUSTOM_*` Smartscape nodes and edges via OpenPipeline, bucket isolation, and dimension normalization | `-[FAQ]-19-third-party-saas-telemetry-integration.md` |
| Monitoring Zscaler (ZIA / ZPA / ZDX) with Dynatrace: the absence of any Dynatrace Hub listing and what owning the integration means, the three data planes and which questions each can and cannot answer (ZIA/ZPA logs are not a substitute for ZDX), the transport split that breaks a single-route design (ZIA Cloud NSS pushes HTTPS, ZPA LSS is raw TCP/TLS and needs a collector hop, ZDX is OAuth pull with a two-hour report window), direct-vs-multi-home architectures, extract/discard posture, a five-node topology model, field mapping and metric naming, and when synthetics add anything over ZDX | `-[FAQ]-20-monitoring-zscaler.md` |
| Getting the right alerts to the right people: the four axes an alerting design conflates — what **fires** (anomaly detector, scoped by segment), who gets **paged** (problem-triggered workflow trigger), who may **see** the problem (IAM policy boundary on `dt.security_context`, on an Event Read permission), what a user **filters** in the UI (segments); **Ownership is the built-in routing mechanism** — `owner` and `dt.owner` are default tag keys in every environment, stored as tags on Smartscape nodes, so the trigger's affected-entity tag filter routes on ownership with no DQL; carrying `dt.owner` on the event instead needs an explicit, **non-retroactive** Problem fields mapping, while record-permission fields like `dt.security_context` map onto problems automatically; the Ownership app's `get_owners` returns team contact details so one workflow can route dynamically; the caveat that a key being *available* is not *populated* (verified 0 of 8 hosts tagged); the documented problem-trigger option list including the **Delay** option for duration suppression (`dt.duration_marker`) and its unresolved conflict with the upgrade guide; and three anti-patterns — segments-scope-notification, routing-replaces-visibility, route-before-enrich | `-[FAQ]-21-alert-notification-routing.md` |
| What happened to classic PurePath timings in the Grail span model: which sub-timings survived (`span.timing.cpu` and `span.timing.cpu_self`, both `stable`) and which were never modeled at all (IO time, disk IO time, lock/sync time, suspension) — an absence in `dt.semantic_dictionary.fields` rather than a rollout gap, which is what makes it permanent rather than pending; why `db.result.fetch_size` is not classic fetch count; why an external web request is a client span with no `endpoint.name`, no `request.is_failed`, and no `dt.service.request.*` series, since endpoints are exclusively detected on request root spans; `request_attribute.<name>` vs `captured_attribute.<name>` — both `stable`, both **arrays** (so `==` matches nothing, silently), request-scoped reconciled vs span-scoped raw; the stability ladder for failure detection (`span.status_code` stable, `request.is_failed` deprecated, `dt.failure_detection.*` experimental); and the four-case dictionary test for answering any "does Grail have field X" question at zero billable bytes | `-[FAQ]-22-purepath-timings-in-grail.md` |

## Related series

- Instrumentation depth behind FAQ-03: `../OTEL - OpenTelemetry Integration/`
- DPS consumption and cost questions beyond FAQ-09: `../FINOPS - Cost Management & FinOps/`
- Operator/DynaKube depth behind FAQ-04 and FAQ-13: `../K8S - Kubernetes Monitoring/`
- Database monitoring depth behind FAQ-14: `../DBMON - Database Monitoring/`
- OpenPipeline depth behind FAQ-19 and FAQ-20: `../OPIPE - OpenPipeline Beyond Logs/`, `../OPLOGS - OpenPipeline Logs/`
- Bucket and retention design behind FAQ-19 § 6: `../ORGNZ - Organize Data: Buckets, Segments, Security/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "FAQ-09") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
