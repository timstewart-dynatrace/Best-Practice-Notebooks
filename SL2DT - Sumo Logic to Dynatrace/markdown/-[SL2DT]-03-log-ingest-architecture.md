# SL2DT-03: Log Ingest Architecture

> **Series:** SL2DT — Sumo Logic to Dynatrace | **Notebook:** 3 of 10 | **Created:** April 2026 | **Last Updated:** 06/23/2026

## Overview

**Goal of this step:** stand up Dynatrace ingest that produces parity with (or improvement over) the Sumo collector footprint. Design Grail buckets, deploy OneAgent + OTel + OpenPipeline pipelines, convert every Field Extraction Rule into an OpenPipeline processor.

This step is gating. Downstream monitors (SL2DT-05), dashboards (SL2DT-06), and the translation pass (SL2DT-04) all assume logs are flowing into the correct buckets with the expected fields. Don't start those until the ingest layer produces at least 5–10% of final production volume for validation.

---

## Table of Contents

1. [What You'll Produce](#outputs)
2. [Prerequisites & Access](#access)
3. [Grail Bucket Design](#buckets)
4. [`_sourceCategory` Mapping — Implementation](#sourcecat)
5. [Deploy OneAgent for Host Collectors](#oneagent)
6. [Deploy OTel Collector for Non-OneAgent Sources](#otel)
7. [Configure OpenPipeline Pipelines](#openpipeline)
8. [Convert FERs to OpenPipeline Processors](#fers)
9. [Validate Ingest Parity](#validate)
10. [Step Exit Criteria](#gate)

---

## Prerequisites

| Requirement | Details |
|-------------|---------|
| **Audience** | Platform engineering + ingest team |
| **Dynatrace tenant** | Gen3 SaaS, Platform Token with `storage:buckets:write`, `settings:objects:write`, `openpipeline:configurations:write` scopes |
| **Prior reading** | SL2DT-02 (taxonomy map + collector inventory + FER inventory); OPMIG-01 (OpenPipeline fundamentals); K8S-01 (if K8s sources) |
| **Tools** | Terraform + Monaco for config-as-code (recommended); kubectl for K8s; cURL for Platform Token validation |

<a id="outputs"></a>
## 1. What You'll Produce

| Artifact | Purpose |
|----------|---------|
| Grail bucket configs | One per retention/IAM tier from taxonomy map |
| OneAgent rollout config | Hosts auto-instrumented |
| OTel Collector manifests | Non-OneAgent sources (syslog, HTTP, cloud) |
| OpenPipeline pipeline configs | Per bucket + per `_sourceCategory` scope |
| OpenPipeline processors | FER equivalents (one per original FER) |
| Ingest validation report | Parity check: Sumo volume vs DT volume per scope |

Commit to `config/ingest/` as Terraform HCL + Monaco YAML.

<a id="access"></a>
## 2. Prerequisites & Access

### Platform Token

Platform Token (prefix `dt0s16.`) with these scopes:

```
storage:buckets:read, storage:buckets:write
storage:logs:read, storage:logs:write
settings:objects:read, settings:objects:write
openpipeline:configurations:read, openpipeline:configurations:write
iam:policies:read, iam:policies:write (for bucket-scoped policies)
```

**Critical:** Platform Token uses `Authorization: Bearer <token>`, not `Api-Token`. See user memory if needed.

```bash
export DT_TENANT="https://<env-id>.apps.dynatrace.com"
export DT_TOKEN="dt0s16.XXXXX..."
curl -H "Authorization: Bearer $DT_TOKEN" "$DT_TENANT/platform/classic/environment-api/v2/tokens/lookup" \
  -H "Content-Type: application/json" -d "{\"token\": \"$DT_TOKEN\"}"
```

### Terraform + Monaco

```bash
# Terraform
terraform init

# Monaco (for bulk config promotion)
monaco --version
```

<a id="buckets"></a>
## 3. Grail Bucket Design

From the taxonomy map (SL2DT-02 output), define buckets.

### Bucket Naming Convention

```
{env}_{purpose}[_{scope}]
```

Examples:
- `custom_logs_prod` — production logs, default
- `custom_logs_prod_db` — production database logs (separate retention)
- `custom_logs_preprod` — pre-production
- `custom_logs_dev` — development
- `audit_logs` — audit trail (compliance retention)

### Settings 2.0 Schema

Buckets are managed via the Dynatrace Settings API using schema `builtin:bucket-configuration`:

```json
{
  "schemaId": "builtin:bucket-configuration",
  "scope": "environment",
  "value": {
    "name": "custom_logs_prod",
    "displayName": "Production Logs",
    "retention": 90,
    "table": "logs"
  }
}
```

### Terraform Example

```hcl
resource "dynatrace_platform_bucket" "custom_logs_prod" {
  name         = "custom_logs_prod"
  display_name = "Production Logs"
  retention    = 90
  table        = "logs"
}

resource "dynatrace_platform_bucket" "custom_logs_prod_db" {
  name         = "custom_logs_prod_db"
  display_name = "Production Database Logs"
  retention    = 180
  table        = "logs"
}

resource "dynatrace_platform_bucket" "audit_logs" {
  name         = "audit_logs"
  display_name = "Audit Trail"
  retention    = 365
  table        = "logs"
}
```

### Validate buckets exist

```dql
// Confirm bucket inventory
fetch dt.system.buckets, from:-5m
| fields name = `name`, displayName = `displayName`, retention = `retention`
| sort name asc

```

<a id="sourcecat"></a>
## 4. `_sourceCategory` Mapping — Implementation

Two decisions come out of the taxonomy map (SL2DT-02): bucket routing and attribute preservation. Both get implemented in OpenPipeline.

### Pattern A — Bucket routing at ingest

Route based on the source identifier (hostname pattern, tag, container label, etc.):

```json
{
  "pipelineId": "pipeline_prod_routing",
  "matchers": [
    { "matcher": "k8s.namespace.name == 'prod'" },
    { "matcher": "log.source startsWith 'prod/'" }
  ],
  "targetBucket": "custom_logs_prod"
}
```

### Pattern B — Attribute preservation

Add a `dt.source_entity` field that mirrors the Sumo `_sourceCategory` value so queries can still group by it:

```json
{
  "processor": "add-field",
  "field": "dt.source_entity",
  "expression": "extract(log.source, 'regex_for_sourcecategory_equivalent')"
}
```

Most customers do both: bucket for isolation, attribute for query-time grouping.

### Query what you get

After OpenPipeline is live, this should return your taxonomy:

```dql
// Inspect the preserved _sourceCategory equivalent
fetch logs, from:-1h, bucket:"custom_logs_prod"
| summarize c = count(), by:{dt.source_entity}
| sort c desc
| limit 50

```

<a id="oneagent"></a>
## 5. Deploy OneAgent for Host Collectors

Every Sumo Installed Collector has a OneAgent equivalent. For hosts already running a Sumo Installed Collector:

### Linux / Windows Hosts

```bash
# Fetch installer from tenant UI or API, then:
/bin/sh Dynatrace-OneAgent-Linux-1.xxx.sh \
  --set-app-log-content-access=true \
  --set-host-group=prod-web

# Verify
oneagentctl --get-host-tag
oneagentctl --get-logmonitoring-enabled
```

### AWS / Azure / GCP Hosts

Use Dynatrace Clouds app (AWS) or cloud-native integrations. No manual OneAgent install.

### Kubernetes

Install the Dynatrace Operator, then configure DynaKube:

```yaml
apiVersion: dynatrace.com/v1beta6
kind: DynaKube
metadata:
  name: dynakube
  namespace: dynatrace
spec:
  apiUrl: https://<env-id>.apps.dynatrace.com/api
  tokens: tokens-secret
  oneAgent:
    cloudNativeFullStack:
      tolerations: []
      nodeSelector: {}
  activeGate:
    capabilities:
      - kubernetes-monitoring
      - routing
  logMonitoring:
    ingestRuleMatchers:
      - attribute: k8s.namespace.name
        operator: matches
        values: ["prod-*", "preprod-*"]
```

### Capture host groupings

Tag hosts per team/environment. These tags feed the bucket routing in OpenPipeline.

```bash
oneagentctl --set-host-tag="team=payments"
oneagentctl --set-host-tag="env=prod"
```

### Verify ingest

```dql
// Confirm OneAgent-instrumented hosts are sending logs
fetch logs, from:-15m
| filter isNotNull(dt.entity.host)
| summarize c = count(), by:{entityName(dt.entity.host)}
| sort c desc
| limit 20

```

<a id="otel"></a>
## 6. Deploy OTel Collector for Non-OneAgent Sources

For sources where OneAgent doesn't apply (syslog listener, HTTP upload, message-queue consumer, proprietary agents), deploy an OTel Collector.

### Minimal OTel Collector Config

```yaml
receivers:
  syslog:
    tcp:
      listen_address: 0.0.0.0:5514
    protocol: rfc5424
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317

processors:
  resourcedetection:
    detectors: [env, system]
  batch:
    send_batch_size: 1000
    timeout: 5s

exporters:
  otlphttp:
    endpoint: https://<env-id>.live.dynatrace.com/api/v2/otlp
    headers:
      Authorization: "Api-Token dt0c01.XXX"

service:
  pipelines:
    logs:
      receivers: [syslog, otlp]
      processors: [resourcedetection, batch]
      exporters: [otlphttp]
```

**Note on auth:** OTel HTTP exporter pairs with Classic API Token (`dt0c01.`) + `Api-Token` scheme for the `/api/v2/otlp` endpoint. Platform Token is not used for OTel ingest auth as of April 2026.

### HTTP Source Equivalent

Sumo "Hosted HTTP Source" → OpenPipeline HTTP endpoint (no OTel in between):

```bash
curl -X POST "$DT_TENANT/api/v2/logs/ingest" \
  -H "Authorization: Api-Token dt0c01.XXX" \
  -H "Content-Type: application/json" \
  -d '[{"content": "app log line", "log.source": "prod/api/web"}]'
```

Scripted ingestion from CloudWatch, EventHub, Pub/Sub — replace with Dynatrace cloud integrations (AWS Clouds app, Azure, GCP) rather than dual-writing via HTTP.

<a id="openpipeline"></a>
## 7. Configure OpenPipeline Pipelines

For each bucket, define a pipeline that:

1. **Matches** — which logs belong here
2. **Processes** — parse, enrich, redact
3. **Routes** — to the target bucket

### Example: Production API Logs Pipeline

```json
{
  "id": "pipeline_prod_api",
  "displayName": "Production API Logs",
  "matchers": [
    { "attribute": "k8s.namespace.name", "operator": "matches", "value": "prod-api-*" }
  ],
  "processors": [
    {
      "type": "parse",
      "pattern": "LD 'method=' WORD:http.method ' path=' DATA:http.path ' status=' INT:http.status"
    },
    {
      "type": "add-field",
      "field": "dt.source_entity",
      "expression": "concat('prod/api/', k8s.deployment.name)"
    },
    {
      "type": "redact",
      "field": "content",
      "pattern": "\\b\\d{16}\\b",
      "replacement": "[REDACTED_CCN]"
    }
  ],
  "targetBucket": "custom_logs_prod"
}
```

### Validation Queries

```dql
// Check that parsing produces the expected fields
fetch logs, from:-15m, bucket:"custom_logs_prod"
| filter dt.source_entity == "prod/api/payments"
| filter isNotNull(http.status)
| summarize c = count(), by:{http.method, http.status}
| sort c desc

```

<a id="fers"></a>
## 8. Convert FERs to OpenPipeline Processors

Every Sumo FER becomes one OpenPipeline parse processor. The logic is the same; only the pattern syntax changes.

### Sumo FER Example

```
# FER name: api-parse-req
# Scope: _sourceCategory=prod/api/*
# Parse expression:
"method=*" "path=*" "status=*"
```

Fields extracted: `method`, `path`, `status`.

### Dynatrace OpenPipeline Equivalent

```json
{
  "type": "parse",
  "matchers": [{ "attribute": "dt.source_entity", "operator": "startsWith", "value": "prod/api/" }],
  "pattern": "LD 'method=' DATA:http.method ' path=' DATA:http.path ' status=' INT:http.status"
}
```

### DPL Pattern Cheat Sheet

| Sumo regex | DPL matcher |
|------------|-------------|
| `.+` (greedy) | `DATA` (lazy, default) |
| `\w+` (word chars) | `WORD` |
| `\d+` (digits) | `INT` |
| `\d+\.\d+` (float) | `DOUBLE` |
| `\S+` (non-space) | `LD` |
| `\d+\.\d+\.\d+\.\d+` (IPv4) | `IPADDR` |
| ISO 8601 timestamp | `TIMESTAMP` |
| Quoted string | `QUOTED_STRING` |
| `\{.*\}` (JSON blob) | `JSON` |

### FER Conversion Workflow

1. For each FER in `inventory/fers.json`, identify the scope.
2. Map the parse expression to DPL matchers.
3. Create the OpenPipeline processor on the matching pipeline.
4. Validate extraction via a DQL query.

### Check: Parsed fields are available

```dql
// After FER conversion, confirm extraction is working
fetch logs, from:-15m, bucket:"custom_logs_prod"
| filter startsWith(dt.source_entity, "prod/api/")
| filter isNotNull(http.method)
| limit 10
| fields content, http.method, http.path, http.status

```

<a id="validate"></a>
## 9. Validate Ingest Parity

Parity = volume per scope per hour in DT is within 5% of Sumo volume for the same scope.

### Sumo-side baseline
```sumoql
*
| count by _sourceCategory
| where _count > 0
```

### Dynatrace-side comparison

```dql
// Count per scope in DT — compare to Sumo baseline
fetch logs, from:-1h
| summarize c = count(), by:{dt.source_entity}
| sort c desc
| limit 50

```

### Parity report

| `_sourceCategory` | Sumo count/hr | DT count/hr | % diff | Status |
|--------------------|----------------|---------------|--------|--------|
| prod/api/payments | 18,452 | 18,110 | -1.9% | ✅ |
| prod/web/frontend | 9,300 | 9,044 | -2.8% | ✅ |
| prod/db/primary | 2,103 | 1,845 | -12.3% | ⚠️ investigate |
| ... | ... | ... | ... | ... |

**Target:** ≥95% of scopes within 5% diff. Flag any scope with >10% diff for investigation before proceeding.

### Common Parity Gaps

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| DT count much lower | OneAgent or OTel not deployed to all sources | Check deployment inventory |
| DT count much higher | Duplicate ingest paths (Sumo still writing + DT ingesting same source twice) | Audit collector config |
| DT missing entire scope | `_sourceCategory` pattern didn't match any OpenPipeline matcher | Adjust matcher |
| Parsed fields missing | FER processor not applied to the pipeline | Re-check processor scope |

<a id="gate"></a>
## 10. Step Exit Criteria

**G3 — Ingest Parity Achieved**

- [ ] All planned buckets exist and retention configured
- [ ] OneAgent deployed to 100% of host-based collectors
- [ ] OTel Collector deployed for non-OneAgent sources
- [ ] OpenPipeline pipelines configured per bucket
- [ ] Every FER has a corresponding OpenPipeline processor
- [ ] `dt.source_entity` attribute populated on all logs
- [ ] Parity report: ≥95% of scopes within 5% of Sumo volume
- [ ] No scope with >10% volume gap uninvestigated

**Next step:** **SL2DT-04 — SumoQL → DQL Translation** (translate the query surface with the `sumoql-to-dql` skill).

---

<a id="references"></a>
## 11. References

### Log ingest paths
- [Ingest from (DT docs)](https://docs.dynatrace.com/docs/ingest-from)
- [Dynatrace OneAgent (DT docs)](https://docs.dynatrace.com/docs/ingest-from/dynatrace-oneagent)
- [OpenTelemetry ingest (DT docs)](https://docs.dynatrace.com/docs/ingest-from/opentelemetry)
- [Logs (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/logs)
- [Dynatrace API (DT docs)](https://docs.dynatrace.com/docs/dynatrace-api)

### Routing, parsing, and storage
- [OpenPipeline (DT docs)](https://docs.dynatrace.com/docs/platform/openpipeline)
- [DPL Pattern Reference (DT docs)](https://docs.dynatrace.com/docs/platform/grail/dynatrace-pattern-language)
- [Grail (DT docs)](https://docs.dynatrace.com/docs/platform/grail)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace or Sumo Logic. Always verify information against the official [Dynatrace documentation](https://docs.dynatrace.com/docs) and [Sumo Logic documentation](https://help.sumologic.com/docs/).*</sub>
