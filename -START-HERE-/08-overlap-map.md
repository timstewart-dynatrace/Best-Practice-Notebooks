# Overlap Map

> **Purpose:** Catalog of where multiple series cover the same ground, with recommendations for which is canonical and the suggested reading order. Use this when you find yourself reading similar material across two or more series and want to dedupe.
> **Last Updated:** 07/15/2026

---

## Table of Contents

1. [How to Use This Map](#how-to-use-this-map)
2. [Foundation Overlaps](#foundation-overlaps)
3. [Migration Overlaps](#migration-overlaps)
4. [Ingestion Overlaps](#ingestion-overlaps)
5. [Operationalize Overlaps](#operationalize-overlaps)
6. [Domain Overlaps](#domain-overlaps)
7. [Cross-Category Overlaps](#cross-category-overlaps)
8. [Anti-Patterns to Avoid](#anti-patterns-to-avoid)

---

## How to Use This Map

When you encounter the same topic in multiple series, the table below tells you:

- **Canonical** — the series that goes deepest on the topic; treat as the source of truth
- **Also covers** — series that touch on the topic with lighter coverage; useful for context but not depth
- **Recommended reading order** — when more than one applies to your situation

This is reference material, not a curriculum. Use it as a lookup when reading rather than a sequence to follow end-to-end.

---

## Foundation Overlaps

| Topic | Canonical | Also covers | Recommended order |
|---|---|---|---|
| First-user setup and authentication | [IAM](../IAM%20-%20IAM%20Administration/) — notebooks 02 (SSO), 03 (groups) | [ONBRD](../ONBRD%20-%20Dynatrace%20Onboarding/) — notebook 02 (light intro) | ONBRD-02 first for orientation; IAM-02..03 for depth |
| Tagging strategy | [FAQ](../FAQ%20-%20Frequently%20Asked%20Questions/) — entry 02 (sources, standards, strategy) for the framework; [ORGNZ](../ORGNZ%20-%20Organize%20Data:%20Buckets,%20Segments,%20Security/) — notebook 01 for organizational concepts | [ONBRD](../ONBRD%20-%20Dynatrace%20Onboarding/) — notebook 06 (basic tags); host-group naming in [FAQ](../FAQ%20-%20Frequently%20Asked%20Questions/) entry 01 | FAQ-02 first for the framework; then ORGNZ-01; reference ONBRD-06 only if you need the basics |
| Bucket basics | [ORGNZ](../ORGNZ%20-%20Organize%20Data:%20Buckets,%20Segments,%20Security/) — notebooks 02 (understanding), 03 (strategy) | [ONBRD](../ONBRD%20-%20Dynatrace%20Onboarding/) — notebook 06 (mention only) | ORGNZ-02..03 always — ONBRD's coverage is too light |
| Bucket access control | [ORGNZ](../ORGNZ%20-%20Organize%20Data:%20Buckets,%20Segments,%20Security/) — notebook 05 (bucket-level access control) | [IAM](../IAM%20-%20IAM%20Administration/) — notebook 05 (boundary design references it) | Read ORGNZ-05 only if scenario applies (compliance, retention, hard cost separation) |
| security_context | [ORGNZ](../ORGNZ%20-%20Organize%20Data:%20Buckets,%20Segments,%20Security/) — notebook 06 | [IAM](../IAM%20-%20IAM%20Administration/) — notebooks 04, 05 use it for boundaries | ORGNZ-06 first — IAM boundary work depends on it |
| Policy authoring | [IAM](../IAM%20-%20IAM%20Administration/) — notebooks 04 (policy authoring), 05 (boundary design), 10 (parameterized) | [ONBRD](../ONBRD%20-%20Dynatrace%20Onboarding/) — notebook 02 (basic policy intro) | IAM-04..05 always; ONBRD-02 only for orientation |
| Segments | [ORGNZ](../ORGNZ%20-%20Organize%20Data:%20Buckets,%20Segments,%20Security/) — notebooks 08 (fundamentals, limits), 10 (filter syntax, variables, visibility, segments-as-code) | [MZ2POL](../MZ2POL%20-%20Management%20Zone%20to%20Policy%20Migration/) — notebook 05 (migration mapping only); [ONBRD](../ONBRD%20-%20Dynatrace%20Onboarding/) — notebook 06 (basic intro); [AUTOM](../AUTOM%20-%20Dynatrace%20Automation/) — notebook 04 (Terraform) | ORGNZ-08 then ORGNZ-10 for how segments work; add MZ2POL-05 only if you are converting existing Management Zones |
| Management Zones → Segments (filtering) | [MZ2POL](../MZ2POL%20-%20Management%20Zone%20to%20Policy%20Migration/) — notebook 05 | [ORGNZ](../ORGNZ%20-%20Organize%20Data:%20Buckets,%20Segments,%20Security/) — notebooks 08, 10 for the underlying mechanics | MZ2POL-05 is readable on its own — you do **not** need the rest of MZ2POL if you are only replacing MZ-based filtering, not MZ-based access control |
| Gen2 → Gen3 access control | [MZ2POL](../MZ2POL%20-%20Management%20Zone%20to%20Policy%20Migration/) — full series | (overview only in [ONBRD](../ONBRD%20-%20Dynatrace%20Onboarding/), [IAM](../IAM%20-%20IAM%20Administration/)) | MZ2POL's access-control path applies only if migrating Gen2 permissions. If your MZs are used for filtering rather than permissions, read MZ2POL-05 alone (see the row above) rather than skipping the series |
| Audit log queries | [IAM](../IAM%20-%20IAM%20Administration/) — notebook 07 (audit and compliance) | [ORGNZ](../ORGNZ%20-%20Organize%20Data:%20Buckets,%20Segments,%20Security/) — notebook 07 (advanced patterns) | IAM-07 for compliance lens; ORGNZ-07 for general patterns |

---

## Migration Overlaps

| Topic | Canonical | Also covers | Recommended order |
|---|---|---|---|
| NR component-level translation | [NRLC](../NRLC%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Deep%20Dives/) — full series | [NR2DT](../NR2DT%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Steps/) references NRLC at each step | NRLC for depth; NR2DT for procedural sequencing |
| NR procedural runbook | [NR2DT](../NR2DT%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Steps/) — 9-step runbook | (none) | NR2DT is the only procedural source |
| NRQL → DQL translation | [NRLC](../NRLC%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Deep%20Dives/) — notebook 02 | [NR2DT](../NR2DT%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Steps/) — notebook 04 (translate) | NRLC-02 for the mappings; NR2DT-04 for engagement-level translation work |
| Splunk SPL → DQL | [S2D](../S2D%20-%20Splunk%20to%20Dynatrace%20Migration/) — notebook 03 | (none) | S2D-03 only |
| SumoQL → DQL | [SL2DT](../SL2DT%20-%20Sumo%20Logic%20to%20Dynatrace/) — notebook 04 | (none) | SL2DT-04 only |
| Logs migration architecture | [OPLOGS](../OPLOGS%20-%20OpenPipeline%20Logs/) — notebooks 02 (migration), 03 (pipeline); [OPIPE](../OPIPE%20-%20OpenPipeline%20Beyond%20Logs/) — notebook 01 | [S2D](../S2D%20-%20Splunk%20to%20Dynatrace%20Migration/) — notebook 02 (Splunk locating); [SL2DT](../SL2DT%20-%20Sumo%20Logic%20to%20Dynatrace/) — notebook 03 (Sumo ingest) | Source-tool series first for inventory; OPLOGS for ingestion design |
| Classic Logs → OpenPipeline | [OPMIG](../OPMIG%20-%20OpenPipeline%20Migration/) — full series | [OPLOGS](../OPLOGS%20-%20OpenPipeline%20Logs/) — notebook 02 (migration) | OPMIG for migration-specific; OPLOGS-02 if general |
| Static-threshold to anomaly detection | [AIOPS](../AIOPS%20-%20Dynatrace%20Intelligence/) — notebook 02 (anomaly detection) | [S2D](../S2D%20-%20Splunk%20to%20Dynatrace%20Migration/) — notebook 04 (anomaly detectors); [SL2DT](../SL2DT%20-%20Sumo%20Logic%20to%20Dynatrace/) — notebook 05 | AIOPS-02 for the framework; S2D and SL2DT for tool-specific translation patterns |

---

## Ingestion Overlaps

| Topic | Canonical | Also covers | Recommended order |
|---|---|---|---|
| OTel collector deployment | [OTEL](../OTEL%20-%20OpenTelemetry%20Integration/) — notebooks 02 (architecture), 03 (deployment) | [K8S](../K8S%20-%20Kubernetes%20Monitoring/) — chapters in 04, 05; [CLOUD](../CLOUD%20-%20Cloud%20Provider%20Integrations/) — chapters in cloud-specific notebooks | OTEL-02..03 first; domain-specific chapters for context |
| OTel trace instrumentation | [OTEL](../OTEL%20-%20OpenTelemetry%20Integration/) — notebook 04 | [SPANS](../SPANS%20-%20Distributed%20Tracing%20and%20Spans/) — notebook 01 mentions OTel; [NRLC](../NRLC%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Deep%20Dives/) — for migration context | OTEL-04 always; SPANS-01 for the data model afterwards |
| Log pipeline processing | [OPLOGS](../OPLOGS%20-%20OpenPipeline%20Logs/) — notebook 03 | [OPMIG](../OPMIG%20-%20OpenPipeline%20Migration/) — notebook 06 (processing/parsing); [OPIPE](../OPIPE%20-%20OpenPipeline%20Beyond%20Logs/) — notebook 01 | OPLOGS-03 first; OPMIG-06 for migration-specific transformations |
| Bucket-routing decisions | [ORGNZ](../ORGNZ%20-%20Organize%20Data:%20Buckets,%20Segments,%20Security/) — notebooks 02, 03 | [OPLOGS](../OPLOGS%20-%20OpenPipeline%20Logs/) — notebook 04 (buckets governance); [OPMIG](../OPMIG%20-%20OpenPipeline%20Migration/) — notebook 05 (routing buckets) | ORGNZ-02..03 first; OPLOGS-04 / OPMIG-05 for ingestion-specific routing |
| Cardinality management | [OPIPE](../OPIPE%20-%20OpenPipeline%20Beyond%20Logs/) — notebook 04 | [SPANS](../SPANS%20-%20Distributed%20Tracing%20and%20Spans/) — notebook 08 (cost optimization) | OPIPE-04 for the mechanism; SPANS-08 for span-specific cost work |
| Span pipelines | [OPIPE](../OPIPE%20-%20OpenPipeline%20Beyond%20Logs/) — notebook 02 | [SPANS](../SPANS%20-%20Distributed%20Tracing%20and%20Spans/) — notebook 07 (buckets and pipeline) | OPIPE-02 first; SPANS-07 for span-specific patterns |

---

## Operationalize Overlaps

| Topic | Canonical | Also covers | Recommended order |
|---|---|---|---|
| Alerting architecture | [ALERT](../ALERT%20-%20Alerting%20Strategy%20and%20Design/) — notebooks 01 (architecture), 02 (detection choice), 03 (routing) | [AIOPS](../AIOPS%20-%20Dynatrace%20Intelligence/) — notebook 02 (detection methods); [WFLOW](../WFLOW%20-%20Workflows%20and%20Alert%20Notifications/) — notebooks 04 (routing), 09 (governance) | ALERT-01..03 first for the framework; domain series for implementation |
| Detection mechanisms | [ALERT](../ALERT%20-%20Alerting%20Strategy%20and%20Design/) — notebook 02 (4-mechanism decision framework) | [AIOPS](../AIOPS%20-%20Dynatrace%20Intelligence/) — notebook 02 (anomaly detection); [SLO](../SLO%20-%20Service%20Level%20Objectives/) — notebook 04 (burn-rate alerting) | ALERT-02 for decision; AIOPS-02 for anomaly depth; SLO-04 for reliability targets |
| Davis problems and root cause | [AIOPS](../AIOPS%20-%20Dynatrace%20Intelligence/) — notebook 03 | [ALERT](../ALERT%20-%20Alerting%20Strategy%20and%20Design/) — notebook 01 (framed in end-to-end architecture); [WFLOW](../WFLOW%20-%20Workflows%20and%20Alert%20Notifications/) — notebook 04 (alert routing on problems) | ALERT-01 for architecture context; AIOPS-03 for depth; WFLOW-04 for routing |
| Anomaly detection | [AIOPS](../AIOPS%20-%20Dynatrace%20Intelligence/) — notebook 02 | [ALERT](../ALERT%20-%20Alerting%20Strategy%20and%20Design/) — notebook 02 (decision framework); [SYNTH](../SYNTH%20-%20Synthetic%20Monitoring/) — notebook 02 (anomaly on synthetics); [DBMON](../DBMON%20-%20Database%20Monitoring/) — notebook 06 (anomaly on DB metrics) | ALERT-02 for the decision framework; AIOPS-02 for the model; domain series for domain-specific applications |
| Service level objectives (SLOs) | [SLO](../SLO%20-%20Service%20Level%20Objectives/) — full series | [ALERT](../ALERT%20-%20Alerting%20Strategy%20and%20Design/) — notebook 01 (SLO-driven alerting in end-to-end architecture); [WFLOW](../WFLOW%20-%20Workflows%20and%20Alert%20Notifications/) — notebook 04 (burn-rate alert routing) | SLO-01..02 first for fundamentals; ALERT for architecture context; WFLOW-04 for routing |
| Burn-rate alerting | [SLO](../SLO%20-%20Service%20Level%20Objectives/) — notebook 04 | [ALERT](../ALERT%20-%20Alerting%20Strategy%20and%20Design/) — notebook 02 (part of detection mechanisms); [WFLOW](../WFLOW%20-%20Workflows%20and%20Alert%20Notifications/) — notebooks 02 (triggers), 04 (routing) | SLO-04 for the concept; ALERT-02 for context; WFLOW for implementation |
| Workflow as code | [WFLOW](../WFLOW%20-%20Workflows%20and%20Alert%20Notifications/) — full series for concepts | [AUTOM](../AUTOM%20-%20Dynatrace%20Automation/) — notebook 05 (workflows-as-code) | WFLOW for design; AUTOM-05 for IaC delivery |
| Dashboard automation | [DASH](../DASH%20-%20Dashboard%20Design%20&%20Building/) — notebook 07 (sharing and reporting) | [AUTOM](../AUTOM%20-%20Dynatrace%20Automation/) — notebooks 02–04 (Settings API, Monaco, Terraform with dashboard examples) | DASH first; AUTOM-02..04 once dashboards stabilize |
| Settings API | [AUTOM](../AUTOM%20-%20Dynatrace%20Automation/) — notebook 02 | [IAM](../IAM%20-%20IAM%20Administration/) — notebook 12 (API provisioning) | AUTOM-02 for general; IAM-12 for IAM-specific |
| GitOps for Dynatrace | [AUTOM](../AUTOM%20-%20Dynatrace%20Automation/) — notebooks 03 (Monaco), 04 (Terraform), 07 (CI/CD) | [K8S](../K8S%20-%20Kubernetes%20Monitoring/) — notebook 03 (GitOps DynaKube) | AUTOM for general patterns; K8S-03 for DynaKube-specific |

---

## Domain Overlaps

| Topic | Canonical | Also covers | Recommended order |
|---|---|---|---|
| Frontend-to-backend tracing | [SPANS](../SPANS%20-%20Distributed%20Tracing%20and%20Spans/) — full series for span concepts | [WEBRUM](../WEBRUM%20-%20Web%20Real%20User%20Monitoring/) — notebook 04 (session analysis); [MOBL](../MOBL%20-%20Mobile%20Monitoring/) — notebook 07 (network requests) | SPANS-01..02 first; frontend domain for domain-specific |
| K8s logs | [K8S](../K8S%20-%20Kubernetes%20Monitoring/) — notebook 07 (events and logs) | [OPLOGS](../OPLOGS%20-%20OpenPipeline%20Logs/) — for log processing | K8S-07 for K8s-specific; OPLOGS for ingestion patterns |
| Kubernetes monitoring | [K8S](../K8S%20-%20Kubernetes%20Monitoring/) — full series | [CLOUD](../CLOUD%20-%20Cloud%20Provider%20Integrations/) — notebook 03 (AWS EKS) | K8S full series first; CLOUD-03 for managed-K8s context |
| Database call tracing | [DBMON](../DBMON%20-%20Database%20Monitoring/) — notebook 05 (query analysis) | [SPANS](../SPANS%20-%20Distributed%20Tracing%20and%20Spans/) — notebook 02 (querying) | DBMON-05 for DB-specific; SPANS-02 for the underlying span queries |
| Synthetic alerting | [WFLOW](../WFLOW%20-%20Workflows%20and%20Alert%20Notifications/) — notebook 04 | [SYNTH](../SYNTH%20-%20Synthetic%20Monitoring/) — notebook 02 (browser monitors) and 06 (analytics) | SYNTH for what to alert on; WFLOW for routing |
| RUM dashboards | [DASH](../DASH%20-%20Dashboard%20Design%20&%20Building/) — notebook 04 (operations) | [WEBRUM](../WEBRUM%20-%20Web%20Real%20User%20Monitoring/) — notebook 08 (dashboards and alerting) | DASH for general dashboard design; WEBRUM-08 for RUM-specific layouts |

---

## Cross-Category Overlaps

| Topic | Canonical | Also covers | Recommended order |
|---|---|---|---|
| Alerting strategy | [ALERT](../ALERT%20-%20Alerting%20Strategy%20and%20Design/) — full series | [WFLOW](../WFLOW%20-%20Workflows%20and%20Alert%20Notifications/) — notebooks 04 (routing emphasis); [AIOPS](../AIOPS%20-%20Dynatrace%20Intelligence/) — notebooks 02–03 (detection and root-cause focus) | ALERT for end-to-end; domain series for their pillar |
| Cost optimization and FinOps | [FINOPS](../FINOPS%20-%20Cost%20Management%20&%20FinOps/) — full series | [ADOPT](../ADOPT%20-%20Observability%20Adoption%20&%20Maturity/) — notebook 05 (optimization roadmap); [ORGNZ](../ORGNZ%20-%20Organize%20Data:%20Buckets,%20Segments,%20Security/) — notebooks 02, 03 (bucket strategy); [SPANS](../SPANS%20-%20Distributed%20Tracing%20and%20Spans/) — notebook 08 (span cost); [OPIPE](../OPIPE%20-%20OpenPipeline%20Beyond%20Logs/) — notebook 04 (cardinality) | FINOPS for dedicated depth; ADOPT-05 for framework context; signal-specific series for tactics |
| Service reliability and SLOs | [SLO](../SLO%20-%20Service%20Level%20Objectives/) — full series | [ALERT](../ALERT%20-%20Alerting%20Strategy%20and%20Design/) — notebook 01 (SLO-driven alerting); [WFLOW](../WFLOW%20-%20Workflows%20and%20Alert%20Notifications/) — notebooks 02, 04 (burn-rate routing) | SLO for concepts; ALERT/WFLOW for operationalization |
| Application security posture | [APPSEC](../APPSEC%20—%20Application%20Security/) — full series | [IAM](../IAM%20-%20IAM%20Administration/) — notebook 05 (boundary design); [ORGNZ](../ORGNZ%20-%20Organize%20Data:%20Buckets,%20Segments,%20Security/) — notebook 09 (enterprise patterns) | APPSEC for security observability; IAM for access control; ORGNZ for multi-tenant context |
| Davis intelligence and AI | [AIOPS](../AIOPS%20-%20Dynatrace%20Intelligence/) — full series | [ALERT](../ALERT%20-%20Alerting%20Strategy%20and%20Design/) — notebook 01 (Davis problems in architecture); [WFLOW](../WFLOW%20-%20Workflows%20and%20Alert%20Notifications/) — notebook 04 (problem routing) | AIOPS for depth; ALERT for alerting context; WFLOW for routing |
| Tagging governance | [FAQ](../FAQ%20-%20Frequently%20Asked%20Questions/) — entry 02 (tagging sources, standards, strategy) | [ORGNZ](../ORGNZ%20-%20Organize%20Data:%20Buckets,%20Segments,%20Security/) — notebook 01 (introduction); [ONBRD](../ONBRD%20-%20Dynatrace%20Onboarding/) — notebook 06 | FAQ-02 first for the framework; ORGNZ-01 for organizational concepts |
| Multi-tenant patterns | [IAM](../IAM%20-%20IAM%20Administration/) — notebook 08 (multi-environment) | [ORGNZ](../ORGNZ%20-%20Organize%20Data:%20Buckets,%20Segments,%20Security/) — notebook 09 (enterprise patterns); [APPSEC](../APPSEC%20—%20Application%20Security/) — notebook 09 (multi-tenant governance) | IAM-08 for access; ORGNZ-09 for organizational; APPSEC-09 for security governance |
| Migration validation | [NRLC](../NRLC%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Deep%20Dives/) — notebook 08; [NR2DT](../NR2DT%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Steps/) — notebook 08 (validate) | [SL2DT](../SL2DT%20-%20Sumo%20Logic%20to%20Dynatrace/) — notebook 09 (cutover validation); [S2D](../S2D%20-%20Splunk%20to%20Dynatrace%20Migration/) — implicit in notebook 09 | Tool-specific validation per migration series |

---

## Anti-Patterns to Avoid

| Anti-Pattern | What's wrong | What to do instead |
|---|---|---|
| Reading [ONBRD](../ONBRD%20-%20Dynatrace%20Onboarding/) notebook 09 (alerts) AND [WFLOW](../WFLOW%20-%20Workflows%20and%20Alert%20Notifications/) full series | ONBRD-09 is an intro that doesn't add to WFLOW | Skip ONBRD-09 if you'll read WFLOW |
| Reading [ONBRD](../ONBRD%20-%20Dynatrace%20Onboarding/) notebook 10 (dashboards) AND [DASH](../DASH%20-%20Dashboard%20Design%20&%20Building/) full series | ONBRD-10 is an intro that doesn't add to DASH | Skip ONBRD-10 if you'll read DASH |
| Treating bucket-level access control as the default | Buckets are immutable, capped at 80, not universal — they're for specific scenarios | Default to security_context (ORGNZ-06); use buckets only for compliance, retention, or hard cost separation |
| Reading [NRLC](../NRLC%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Deep%20Dives/) and [NR2DT](../NR2DT%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Steps/) sequentially as if they're a single series | They're paired references — NRLC for component depth, NR2DT for procedural sequencing | Read in parallel: open NR2DT for the step you're on, dive into NRLC for the components touched in that step |
| Migrating multiple source tools simultaneously | Halves your team's focus, doubles the risk of either failing | Migrate in waves; one source tool at a time |
| Using [ONBRD](../ONBRD%20-%20Dynatrace%20Onboarding/) as the definitive source for IAM, tagging, or buckets | ONBRD's coverage is intentionally light to keep the onboarding scope narrow | Use ONBRD for orientation; dive into IAM, ORGNZ, or FAQ for depth |

---

## Maintenance Note

**This overlap map MUST be updated whenever new series are added to `topics/` or existing series are significantly expanded:**
- Add new series to the relevant overlap tables (Operationalize, Domain, Cross-Category, etc.)
- When a new series introduces overlaps with existing series, add rows to the appropriate table
- Remove or update anti-patterns if they become outdated due to new series
- Update series count references if the total count changes (currently 32 topic series as of July 15, 2026)
- Update Last Updated date to current date

The overlap map is a deduplication guide; stale maps create confusion when readers encounter the same topic in unexpected places. Keep it current alongside new series releases.

---

> *This playbook was AI-generated from community-submitted and publicly available sources. It is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*
