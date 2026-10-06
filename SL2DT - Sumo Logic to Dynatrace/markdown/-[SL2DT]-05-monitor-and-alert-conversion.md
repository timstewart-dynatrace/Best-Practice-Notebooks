# SL2DT-05: Monitor & Alert Conversion

> **Series:** SL2DT — Sumo Logic to Dynatrace | **Notebook:** 5 of 11 | **Created:** April 2026 | **Last Updated:** 10/06/2026

## Overview

**Goal of this step:** rebuild Sumo Monitors in Dynatrace, choosing the right target for each (Anomaly Detection, Workflow with DQL threshold, or Metric Event). This is the notebook where the biggest fidelity-vs-better-approach decisions happen.

The anti-pattern to avoid: 1:1 static-threshold lift-and-shift. It produces noisy alerts, misses real anomalies, and under-uses Dynatrace's capabilities. The pattern to prefer: Anomaly Detection for metric-backed alerts, Workflows for complex multi-condition logic, and Metric Events for simple static thresholds that genuinely are static.

---

## Table of Contents

1. [What You'll Produce](#outputs)
2. [The Monitor Conversion Decision Framework](#framework)
3. [Anomaly Detection — When & How](#davis)
4. [Metric Events — Simple Static Thresholds](#metric-events)
5. [Workflows — Complex or Multi-Condition Alerts](#workflows)
6. [Rebuilding Notification Actions](#actions)
7. [Handling Rare Alert Classes](#rare)
8. [Tuning & Noise Reduction](#tuning)
9. [Step Exit Criteria](#gate)
10. [References](#references)

---

## Prerequisites

| Requirement | Details |
|-------------|---------|
| **Audience** | Core migration engineers + app-team reviewers |
| **Inputs** | `inventory/monitors.json`, translations from SL2DT-04 |
| **Dynatrace access** | Platform Token with `settings:objects:write`, `automation:workflows:write`, `davis:anomaly-detectors:write` |
| **Prior reading** | SL2DT-04 for query translations; OPLOGS-09 for log-based alert patterns |

<a id="outputs"></a>
## 1. What You'll Produce

| Artifact | Purpose |
|----------|---------|
| `monitors/decision-matrix.csv` | Per-monitor: Dynatrace Intelligence vs Workflow vs Metric Event decision |
| `monitors/configs/` | Terraform + JSON for Dynatrace Intelligence detectors, Workflows, Metric Events |
| `notifications/action-map.md` | Sumo webhook/email → DT notification target map |
| `monitor-rebuild-report.md` | Progress per team |

<a id="framework"></a>
## 2. The Monitor Conversion Decision Framework

For each Sumo monitor, pick one of three targets:

### Decision Rules

| Sumo Monitor Type | Target | Why |
|-------------------|--------|-----|
| Outlier / anomaly / seasonality-based | **anomaly detection** | Native baseline learning, no tuning |
| Static threshold on metric (CPU > 90%) | **Metric Event** | Lightweight, tenant-wide config |
| Static threshold on log count | **Workflow with DQL** | Query runs on schedule, threshold in task condition |
| Missing data alert | **Workflow** | Use `isNull` / `count == 0` checks |
| Multi-condition (X AND Y but not Z) | **Workflow** | Multiple DQL tasks + branching |
| SLO-backed alert | **SLO burn-rate alert** (if Dynatrace SLO) | Native SLO product |

![Monitor Conversion Decision Tree](images/05-monitor-decision-tree_930x500.png)

<!-- MARKDOWN_TABLE_ALTERNATIVE
| Input | Output | Rationale |
|-------|--------|-----------|
| Outlier/anomaly monitor | anomaly detection | Native baseline |
| Metric static threshold | Metric Event | Tenant-wide |
| Log count static threshold | Workflow + DQL | Scheduled query |
| Missing data | Workflow | isNull/count check |
| Multi-condition | Workflow | Branching logic |
| SLO-backed | SLO burn-rate | Native product |
For environments where SVG doesn't render
-->

<a id="davis"></a>
## 3. Anomaly Detection — When & How

Dynatrace Intelligence learns the baseline of a metric and alerts when values deviate. Best for:

- Any Sumo outlier monitor (`outlier _count window=N threshold=K`)
- Static thresholds where the "normal" value drifts seasonally (weekly cycle, business hours)
- Static thresholds that fire too often in Sumo (false positives from workload swings)

### Configuration

anomaly detection is configured via the Settings API (`builtin:anomaly-detection.metric-events`) or via the UI. This is the **metric-event** schema, which evaluates Metrics Classic keys. An adaptive or seasonal model needs a `METRIC_SELECTOR` query definition — *"Metric key based query definitions only support static thresholds"* — and every model needs its full set of sample counts plus an event template. For programmatic setup:

```json
{
  "schemaId": "builtin:anomaly-detection.metric-events",
  "scope": "environment",
  "value": {
    "enabled": true,
    "summary": "API error rate anomaly",
    "queryDefinition": {
      "type": "METRIC_SELECTOR",
      "metricSelector": "custom.metric.api_error_rate:splitBy(\"dt.entity.host\"):avg"
    },
    "modelProperties": {
      "type": "AUTO_ADAPTIVE_THRESHOLD",
      "signalFluctuation": 1.0,
      "tolerance": 4.0,
      "alertCondition": "ABOVE",
      "alertOnNoData": false,
      "samples": 5,
      "violatingSamples": 3,
      "dealertingSamples": 5
    },
    "eventTemplate": {
      "title": "API error rate anomaly",
      "description": "The API error rate is above its learned baseline.",
      "eventType": "ERROR",
      "davisMerge": true,
      "metadata": []
    }
  }
}
```

> <sub>**Sources:** [builtin:anomaly-detection.metric-events schema (DT docs)](https://docs.dynatrace.com/docs/dynatrace-api/environment-api/settings/schemas/builtin-anomaly-detection-metric-events).</sub>

### Sumo outlier → anomaly detection example

**Sumo:**
```
_sourceCategory=prod/api | timeslice 1m | count | outlier _count window=10 threshold=3
```

**Dynatrace Intelligence:**
1. Create a custom metric from the DQL equivalent (via metric extraction in OpenPipeline, or via `makeTimeseries` into an extracted metric).
2. Configure anomaly detection on that metric with `AUTO_ADAPTIVE_THRESHOLD` mode.
3. Attach the Sumo monitor's notification channel to the detected problem.

### Metric Extraction from Logs

If the Sumo monitor was log-count-based, extract a metric first:

```json
{
  "schemaId": "builtin:logmonitoring.log-custom-metric",
  "scope": "environment",
  "value": {
    "key": "custom.metric.api_error_count",
    "enabled": true,
    "query": "sumo.source_category = \"prod/api\" AND status = \"ERROR\"",
    "measure": {"type": "OCCURRENCE"},
    "dimensions": ["dt.entity.host"]
  }
}
```

Then apply anomaly detection to the extracted metric.

### Validation

```dql
// Confirm metric extraction is producing data
timeseries c = avg(custom.metric.api_error_count), from:-1h, by:{dt.entity.host}
| fieldsAdd c_max = arrayMax(c)
| filter c_max > 0

```

<a id="metric-events"></a>
## 4. Metric Events — Simple Static Thresholds

When the threshold truly is static and the underlying metric is well-defined:

- CPU > 90%
- Memory available < 5%
- Disk full
- Certificate expiry < 30 days

Use Metric Events via `builtin:anomaly-detection.metric-events`:

```json
{
  "schemaId": "builtin:anomaly-detection.metric-events",
  "scope": "environment",
  "value": {
    "enabled": true,
    "summary": "Host CPU > 90%",
    "queryDefinition": {
      "type": "METRIC_KEY",
      "metricKey": "builtin:host.cpu.usage",
      "aggregation": "AVG",
      "entityFilter": {
        "dimensionKey": "dt.entity.host",
        "conditions": [
          {"type": "HOST_GROUP_NAME", "operator": "EQUALS", "value": "prod-web"}
        ]
      }
    },
    "modelProperties": {
      "type": "STATIC_THRESHOLD",
      "threshold": 90.0,
      "alertCondition": "ABOVE",
      "alertOnNoData": false,
      "samples": 5,
      "violatingSamples": 3,
      "dealertingSamples": 5
    },
    "eventTemplate": {
      "title": "Host CPU > 90%",
      "description": "CPU usage has been above 90% for 3 of the last 5 minutes.",
      "eventType": "RESOURCE",
      "davisMerge": true,
      "metadata": []
    }
  }
}
```

The schema's only scope is `environment`; narrow the alert to particular hosts with the `entityFilter` (here a host group), not with an entity ID as the scope. Metric events read Metrics Classic, so the key is `builtin:host.cpu.usage` — the Grail key `dt.host.cpu.usage` is for DQL.

### When NOT to use Metric Events

- Threshold depends on workload (use Dynatrace Intelligence)
- Condition involves multiple metrics (use Workflow)
- Alert requires custom notification payload (use Workflow)

<a id="workflows"></a>
## 5. Workflows — Complex or Multi-Condition Alerts

Workflows are the equivalent of Sumo's more complex monitors. Use when:

- Alert logic needs multiple DQL queries
- Conditional branches required
- Custom notification payload (ServiceNow incident with specific fields, etc.)
- Scheduled evaluation (every N minutes)

### Workflow Structure

The schedule (cron `*/5 * * * *`, every 5 minutes) is the workflow's trigger, set in the editor rather than written into the task list. The HTTP task authenticates with a Credential Vault credential selected in its **Authentication** field.

```yaml
tasks:
  - name: check_error_rate
    action: dynatrace.automations:execute-dql-query
    input:
      query: |
        fetch logs, from:-5m
        | filter sumo.source_category == "prod/api"
        | summarize {total = count(), errors = countIf(contains(content, "error", caseSensitive:false))}
        | fieldsAdd error_pct = 100.0 * toDouble(errors) / toDouble(total)
  - name: check_threshold
    action: dynatrace.automations:http-function   # ServiceNow Table API — see §6
    predecessors: [check_error_rate]
    conditions:
      states:
        check_error_rate: SUCCESS
      custom: '{{ result("check_error_rate").records[0].error_pct > 5 }}'
    input:
      method: POST
      url: https://<instance>.service-now.com/api/now/table/incident
      headers:
        Content-Type: application/json
      payload: |
        {
          "short_description": "API error rate {{ result('check_error_rate').records[0].error_pct }}%",
          "assignment_group": "Payments Platform",
          "priority": "2"
        }
```

### Scheduled Search → Workflow

A Sumo scheduled search (query + schedule + action) maps directly:

| Sumo Scheduled Search Field | Workflow Equivalent |
|------------------------------|---------------------|
| Query | **Execute DQL Query** task (`dynatrace.automations:execute-dql-query`) |
| Schedule (run every N) | Schedule trigger with a cron rule, set in the editor |
| Alert condition | `conditions.custom` on the notification task (a Jinja expression) |
| Webhook action | HTTP Request task (`dynatrace.automations:http-function`) or a connector action |
| Email action | **Send email** task (`dynatrace.email:send-email`) |

### DQL in Workflow Tasks

The translated DQL from SL2DT-04 goes directly here. Wrap long queries in `|` to preserve formatting:

```yaml
tasks:
  - name: find_slow_transactions
    action: dynatrace.automations:execute-dql-query
    input:
      query: |
        fetch logs, from:-5m
        | filter sumo.source_category == "prod/api"
        | parse content, "LD? 'latency=' INT:latency"
        | filter latency > 2000
        | summarize c = count(), by:{http.path}
        | sort c desc
        | limit 20
```

<a id="actions"></a>
## 6. Rebuilding Notification Actions

Every Sumo monitor has one or more actions. Map each to a Dynatrace notification target.

| Sumo Action | Dynatrace Target | Notes |
|-------------|------------------|-------|
| Email | **Send email** task (`dynatrace.email:send-email`) | Same recipient list; at most ten per field |
| Webhook → ServiceNow | HTTP Request task to the Table API, or ServiceNow **Create Incident** (`dynatrace.servicenow:snow-create-incident`) | Rebuild incident payload |
| Webhook → Slack | Slack **Send message** task (`dynatrace.slack:slack-send-message`) | Rebuild message format |
| Webhook → PagerDuty | PagerDuty **Send event** task (`dynatrace.pagerduty:send-event`) | Rebuild event payload |
| Webhook → custom HTTP | HTTP Request task (`dynatrace.automations:http-function`) | |
| Mobile push | Workflow + Dynatrace Mobile app | |

### ServiceNow Integration — Specific Patterns

ServiceNow is the most common target and the highest-risk translation (wrong field mapping → misrouted incidents).

**Sumo webhook payload:**
```json
{
  "incident": {
    "short_description": "{{Name}}: {{Description}}",
    "u_affected_service": "{{ClusterName}}",
    "assignment_group": "{{Team}}",
    "priority": "{{Priority}}"
  }
}
```

**Dynatrace Workflow equivalent:**
```yaml
- name: create_servicenow_incident
  action: dynatrace.automations:http-function
  input:
    method: POST
    url: https://<instance>.service-now.com/api/now/table/incident
    # Authentication: select a Credential Vault credential (Basic) in the task's
    # Authentication field. Never put the credential in a static Authorization header.
    headers:
      Content-Type: application/json
    payload: |
      {
        "short_description": "API error rate {{ result('check_error_rate').records[0].error_pct }}%",
        "u_affected_service": "{{ result('check_error_rate').records[0]['dt.entity.service'] }}",
        "assignment_group": "Payments Platform",
        "priority": "2"
      }
```

The request body goes in `payload`, and dotted field names need bracket access (`records[0]['dt.entity.service']`). The HTTP route keeps custom fields such as `u_affected_service`. The ServiceNow Connector's **Create Incident** action (`dynatrace.servicenow:snow-create-incident`) is the alternative when the standard fields are enough. It takes named inputs (`shortDescription`, `description`, `impact`, `urgency`, `category`, `group`, `correlationId`, …) rather than a raw body. For the query above, `u_affected_service` stays empty unless the query groups by `dt.entity.service`.

> <sub>**Sources:** [HTTP request action (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/workflows/default-workflow-actions/http-request-workflow-action) — *"Payload : The payload of the HTTP request."*, *"We strictly advise against providing any static Authorization header and therefore, leak a secret."*; [wftpl_sample_servicenow_incident_man.yaml (Dynatrace GitHub)](https://raw.githubusercontent.com/Dynatrace/Dynatrace-workflow-samples/main/samples/Messaging%20and%20Incident%20Management/wftpl_sample_servicenow_incident_man.yaml) — *"action: dynatrace.servicenow:snow-create-incident"*.</sub>

**Verify** with a test incident before flipping production traffic.

<a id="rare"></a>
## 7. Handling Rare Alert Classes

### Missing Data Alerts

Sumo:
```
_sourceCategory=prod/heartbeat | count
| where _count == 0
```

Dynatrace Workflow:
```yaml
tasks:
  - name: check_heartbeat
    action: dynatrace.automations:execute-dql-query
    input:
      query: |
        fetch logs, from:-10m
        | filter sumo.source_category == "prod/heartbeat"
        | summarize c = count()
  - name: alert_if_missing
    action: dynatrace.email:send-email
    predecessors: [check_heartbeat]
    conditions:
      custom: '{{ result("check_heartbeat").records[0].c == 0 }}'
    input:
      to: ["oncall@example.com"]
      cc: []
      bcc: []
      subject: "No heartbeat from prod/heartbeat"
      content: "No heartbeat logs from prod/heartbeat in the last 10 minutes."
```

### Change Detection Alerts

Sumo:
```
_sourceCategory=audit | logcompare timeshift=24h
```

Dynatrace: Change detection or two-fetch comparison workflow. See OPLOGS-08.

### Multi-Metric Correlation

Sumo: two separate monitors with same target; Dynatrace: one Workflow with two DQL tasks + AND condition.

<a id="tuning"></a>
## 8. Tuning & Noise Reduction

Expect more alerts in the first week post-cutover — baselines aren't learned yet, and Workflow thresholds may be too aggressive.

### First-week discipline

1. Monitor alert volume daily. Dashboard the total count of detected problems + Workflow triggers.
2. For every alert, triage: real issue, false positive, or threshold too tight?
3. Adjust thresholds daily until noise drops below Sumo baseline.

### anomaly detection — let it learn

Dynatrace Intelligence needs ~2 weeks of data to build a stable baseline. During week 1–2, expect more sensitivity alerts. Keep them in a separate "tuning" severity class if possible; don't page on-call for them.

### Checking alert volume

```dql
// Alert volume over time — compare Sumo baseline
// dt.davis.problems holds one record per problem; `fetch events | filter event.kind == "DAVIS_PROBLEM"`
// returns every update of every problem and over-counts many times over.
fetch dt.davis.problems, from:-7d
| makeTimeseries problems = count(), interval:24h, time:event.start

```

### Silence during load tests

Workflow-based monitors should respect maintenance windows. Settings 2.0 schema `builtin:maintenance.general` applies tenant-wide; configure before load tests, CI/CD deployments, or controlled outages.

### Severity mapping

| Sumo Priority | DT Severity | Notification |
|---------------|-------------|--------------|
| Critical | ERROR / Dynatrace Intelligence P1 | PagerDuty + ServiceNow |
| High | Dynatrace Intelligence P2 | ServiceNow |
| Warning | Dynatrace Intelligence P3 | Email |
| Info | (no alert) | Logged only |

<a id="gate"></a>
## 9. Step Exit Criteria

**G5 — Monitors Rebuilt**

- [ ] Every Sumo monitor has a decision logged (Dynatrace Intelligence / Workflow / Metric Event / retire)
- [ ] ≥40% of new monitors use anomaly detection (not static thresholds) — track this metric
- [ ] Top-10 business-critical monitors validated end-to-end (fires on known-bad condition, doesn't fire on normal)
- [ ] Notification integrations (ServiceNow, Slack, PagerDuty) tested with live test incidents
- [ ] First-week alert volume ≤ Sumo baseline
- [ ] App teams trained on monitor rebuild (train-the-trainer — see SL2DT-01)

**Next step:** **SL2DT-06 — Dashboard Conversion** (rebuild dashboards using the translation output).

---

<a id="references"></a>
## 10. References

### Dynatrace alerting and intelligence
- [Anomaly detection (DT docs)](https://docs.dynatrace.com/docs/dynatrace-intelligence/anomaly-detection)
- [Workflows (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/workflows)
- [Notifications and alerting (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/notifications-and-alerting)
- [Problems app (DT docs)](https://docs.dynatrace.com/docs/dynatrace-intelligence/problems-app)
- [Maintenance windows (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/notifications-and-alerting/maintenance-windows)

### Sumo Logic monitor and alert reference (source)
- [Sumo Logic monitors (Sumo Logic docs)](https://www.sumologic.com:443/help/docs/alerts/monitors/)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace or Sumo Logic. Always verify information against the official [Dynatrace documentation](https://docs.dynatrace.com/docs) and [Sumo Logic documentation](https://www.sumologic.com:443/help/docs/).*</sub>
