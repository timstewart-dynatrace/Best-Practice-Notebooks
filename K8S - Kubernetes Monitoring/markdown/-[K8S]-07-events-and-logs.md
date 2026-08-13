# K8S-07: Kubernetes Events and Log Ingestion

> **Series:** K8S — Kubernetes Monitoring | **Notebook:** 7 of 13 | **Created:** January 2026 | **Last Updated:** 08/12/2026

## Capturing and Analyzing Kubernetes Events and Logs
Kubernetes events and container logs provide crucial insights for debugging and operational awareness. This notebook covers event monitoring, log ingestion configuration, and analysis patterns in Dynatrace.

---

## Table of Contents

1. [Event Ingestion Configuration](#event-ingestion-configuration)
2. [Container Log Collection](#container-log-collection)
3. [OpenPipeline for K8s Logs](#openpipeline-for-k8s-logs)
4. [Event Analysis Patterns](#event-analysis-patterns)
5. [Log Analysis Patterns](#log-analysis-patterns)
6. [Alerting on Events and Logs](#alerting-on-events-and-logs)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Dynatrace Environment** | SaaS with log ingestion enabled |
| **DynaKube** | ActiveGate with `kubernetes-monitoring` |
| **Permissions** | `logs.read`, `logs.ingest`, `events.read` |
| **Knowledge** | K8S-01 Fundamentals |

## 1. Kubernetes Events Overview

### Event Types

Kubernetes events are first-class objects that record what happened in the cluster.

| Type | Description | Examples |
|------|-------------|----------|
| **Normal** | Routine operations | Scheduled, Pulled, Created |
| **Warning** | Potential issues | FailedScheduling, BackOff, Unhealthy |

### Event Sources

| Source | Events Generated |
|--------|------------------|
| **Scheduler** | Scheduling decisions, failures |
| **Kubelet** | Pod lifecycle, probe results |
| **Controller Manager** | ReplicaSet scaling, Deployment rollouts |
| **Custom Controllers** | CRD reconciliation events |

### Event Lifecycle

![Kubernetes Events & Logs Flow](images/07-k8s-event-log-flow.png)

<!-- MARKDOWN_TABLE_ALTERNATIVE
| Stage | Description |
|-------|-------------|
| Event Created | Kubernetes generates event |
| Stored in etcd | Event persisted temporarily |
| TTL expires (1h) | Event deleted from etcd |
| Dynatrace Ingests | Captured before deletion |
| Persisted in Grail | Long-term storage (35+ days) |

**Key:** K8s events have 1h TTL in etcd, but Dynatrace preserves them for 35+ days.
For environments where SVG doesn't render
-->

**Important:** Kubernetes events are short-lived. Dynatrace captures them for long-term storage and analysis.

<a id="event-ingestion-configuration"></a>
## 2. Event Ingestion Configuration
### ActiveGate Kubernetes Monitoring

Enable event collection via DynaKube:

```yaml
spec:
  activeGate:
    capabilities:
      - kubernetes-monitoring  # Enables event collection
      - routing
```

### Event Filtering

Configure which events to ingest:

| Setting | Location | Purpose |
|---------|----------|----------|
| Event types | Settings > Cloud and virtualization | Normal, Warning, or both |
| Namespace filter | DynaKube spec | Limit to specific namespaces |
| Event reasons | Settings | Include/exclude specific reasons |

### Recommended Configuration

| Use Case | Configuration |
|----------|---------------|
| **Full visibility** | All event types, all namespaces |
| **Warning focus** | Warning events only, all namespaces |
| **Cost optimization** | Warning events, specific namespaces |

```dql
// Recent Kubernetes events
// Data object corrected 08/12/2026. Kubernetes events are NOT logs. This cell scraped
// `fetch logs` for `log.source` containing "kubernetes" or for content substrings like "BackOff" —
// no log.source matches, and kubelet event text is not in the log stream, so it returned nothing
// while the cluster emitted 231,296 Kubernetes events in the same window.
// They arrive as `fetch events | filter event.provider == "KUBERNETES_EVENT"`, STRUCTURED:
//   dt.kubernetes.event.reason            Unhealthy · BackOff · Killing · FailedScheduling ·
//                                         FailedMount · BackoffLimitExceeded · EvictionThresholdMet …
//   dt.kubernetes.event.message           the human-readable text
//   dt.kubernetes.event.important         "true" marks the warning-class events
//   dt.kubernetes.event.involved_object.kind / .name
//   dt.kubernetes.event.count / .first_seen / .last_seen
//   plus k8s.cluster.name · k8s.namespace.name · k8s.pod.name · k8s.workload.name · k8s.node.name
// NOTE: event.type is CUSTOM_INFO on every one of these — it is NOT "Warning"; severity lives in
// dt.kubernetes.event.important. Enumerate reasons with:
//   fetch events, from:-24h | filter event.provider == "KUBERNETES_EVENT"
//   | summarize n = count(), by:{dt.kubernetes.event.reason} | sort n desc
fetch events, from:-1h
| filter event.provider == "KUBERNETES_EVENT"
| fields timestamp, k8s.cluster.name, k8s.namespace.name, dt.kubernetes.event.reason, dt.kubernetes.event.message
| sort timestamp desc
| limit 50
```

```dql
// Warning-class events only
// Data object corrected 08/12/2026. Kubernetes events are NOT logs. This cell scraped
// `fetch logs` for `log.source` containing "kubernetes" or for content substrings like "BackOff" —
// no log.source matches, and kubelet event text is not in the log stream, so it returned nothing
// while the cluster emitted 231,296 Kubernetes events in the same window.
// They arrive as `fetch events | filter event.provider == "KUBERNETES_EVENT"`, STRUCTURED:
//   dt.kubernetes.event.reason            Unhealthy · BackOff · Killing · FailedScheduling ·
//                                         FailedMount · BackoffLimitExceeded · EvictionThresholdMet …
//   dt.kubernetes.event.message           the human-readable text
//   dt.kubernetes.event.important         "true" marks the warning-class events
//   dt.kubernetes.event.involved_object.kind / .name
//   dt.kubernetes.event.count / .first_seen / .last_seen
//   plus k8s.cluster.name · k8s.namespace.name · k8s.pod.name · k8s.workload.name · k8s.node.name
// NOTE: event.type is CUSTOM_INFO on every one of these — it is NOT "Warning"; severity lives in
// dt.kubernetes.event.important. Enumerate reasons with:
//   fetch events, from:-24h | filter event.provider == "KUBERNETES_EVENT"
//   | summarize n = count(), by:{dt.kubernetes.event.reason} | sort n desc
fetch events, from:-1h
| filter event.provider == "KUBERNETES_EVENT"
| filter dt.kubernetes.event.important == "true"
| summarize events = count(), by:{dt.kubernetes.event.reason, k8s.namespace.name}
| sort events desc
| limit 25
```

<a id="container-log-collection"></a>
## 3. Container Log Collection
### Log Collection Methods

| Method | Source | Configuration |
|--------|--------|---------------|
| **OneAgent** | Container stdout/stderr | Automatic with OneAgent |
| **Log Monitoring** | Mounted log files | Custom paths in settings |
| **Fluentd/Fluent Bit** | External forwarder | OTLP endpoint |

### OneAgent Log Collection

OneAgent automatically collects:
- Container stdout logs
- Container stderr logs
- Process-generated log files (with configuration)

### Log Attributes

| Attribute | Source | Example |
|-----------|--------|----------|
| `k8s.namespace.name` | Container metadata | `checkout` |
| `k8s.pod.name` | Container metadata | `checkout-api-abc123` |
| `k8s.container.name` | Container metadata | `api` |
| `dt.entity.container_group_instance` | Entity relationship | `CONTAINER_GROUP_INSTANCE-XXX` |
| `loglevel` | Parsed from content | `ERROR`, `WARN`, `INFO` |

### DynaKube Log Configuration

```yaml
spec:
  oneAgent:
    cloudNativeFullStack:
      env:
        - name: ONEAGENT_ENABLE_LOG_ANALYTICS
          value: "true"
```

```dql
// Container logs by namespace
fetch logs, from:-1h
| filter isNotNull(k8s.namespace.name)
| summarize logCount = count(), by:{k8s.namespace.name}
| sort logCount desc
| limit 15
```

```dql
// Error logs with Kubernetes context
fetch logs, from:-1h
| filter loglevel == "ERROR" or loglevel == "SEVERE"
| filter isNotNull(k8s.namespace.name)
| fields timestamp, k8s.namespace.name, k8s.pod.name, content
| sort timestamp desc
| limit 30
```

<a id="openpipeline-for-k8s-logs"></a>
## 4. OpenPipeline for K8s Logs
### Log Processing Pipeline

OpenPipeline processes logs before storage:

```
Log Ingest → Routing → Processing → Storage
                ↓           ↓
         Match rules   Transform, enrich
```

### Common Processing Rules

| Rule Type | Use Case | Example |
|-----------|----------|----------|
| **Parse** | Extract fields | JSON parsing, regex |
| **Transform** | Modify content | Rename fields, mask data |
| **Filter** | Drop logs | Remove debug logs |
| **Route** | Direct to bucket | By namespace or app |

### Example: Parse JSON Logs

```yaml
# OpenPipeline configuration
pipelines:
  - name: k8s-json-logs
    routes:
      - match: k8s.namespace.name exists
    processing:
      - type: json
        source: content
```

### Example: Filter Debug Logs

```yaml
pipelines:
  - name: k8s-filter-debug
    routes:
      - match: k8s.namespace.name exists and loglevel == "DEBUG"
    processing:
      - type: drop
```

<a id="event-analysis-patterns"></a>
## 5. Event Analysis Patterns
### Key Event Reasons to Monitor

| Reason | Meaning | Action |
|--------|---------|--------|
| **FailedScheduling** | Pod can't be scheduled | Check resource availability |
| **FailedMount** | Volume mount failed | Check PV/PVC config |
| **BackOff** | Container restart backoff | Check logs, fix crash |
| **Unhealthy** | Probe failed | Check probe config, app health |
| **Evicted** | Pod evicted from node | Check node pressure |
| **OOMKilled** | Out of memory | Increase limits |

```dql
// Failed scheduling events
// Data object corrected 08/12/2026. Kubernetes events are NOT logs. This cell scraped
// `fetch logs` for `log.source` containing "kubernetes" or for content substrings like "BackOff" —
// no log.source matches, and kubelet event text is not in the log stream, so it returned nothing
// while the cluster emitted 231,296 Kubernetes events in the same window.
// They arrive as `fetch events | filter event.provider == "KUBERNETES_EVENT"`, STRUCTURED:
//   dt.kubernetes.event.reason            Unhealthy · BackOff · Killing · FailedScheduling ·
//                                         FailedMount · BackoffLimitExceeded · EvictionThresholdMet …
//   dt.kubernetes.event.message           the human-readable text
//   dt.kubernetes.event.important         "true" marks the warning-class events
//   dt.kubernetes.event.involved_object.kind / .name
//   dt.kubernetes.event.count / .first_seen / .last_seen
//   plus k8s.cluster.name · k8s.namespace.name · k8s.pod.name · k8s.workload.name · k8s.node.name
// NOTE: event.type is CUSTOM_INFO on every one of these — it is NOT "Warning"; severity lives in
// dt.kubernetes.event.important. Enumerate reasons with:
//   fetch events, from:-24h | filter event.provider == "KUBERNETES_EVENT"
//   | summarize n = count(), by:{dt.kubernetes.event.reason} | sort n desc
fetch events, from:-7d
| filter event.provider == "KUBERNETES_EVENT"
| filter dt.kubernetes.event.reason == "FailedScheduling"
| fields timestamp, k8s.namespace.name, dt.kubernetes.event.involved_object.name, dt.kubernetes.event.message
| sort timestamp desc
| limit 25
```

```dql
// Pod restart events (BackOff, CrashLoopBackOff)
// Data object corrected 08/12/2026. Kubernetes events are NOT logs. This cell scraped
// `fetch logs` for `log.source` containing "kubernetes" or for content substrings like "BackOff" —
// no log.source matches, and kubelet event text is not in the log stream, so it returned nothing
// while the cluster emitted 231,296 Kubernetes events in the same window.
// They arrive as `fetch events | filter event.provider == "KUBERNETES_EVENT"`, STRUCTURED:
//   dt.kubernetes.event.reason            Unhealthy · BackOff · Killing · FailedScheduling ·
//                                         FailedMount · BackoffLimitExceeded · EvictionThresholdMet …
//   dt.kubernetes.event.message           the human-readable text
//   dt.kubernetes.event.important         "true" marks the warning-class events
//   dt.kubernetes.event.involved_object.kind / .name
//   dt.kubernetes.event.count / .first_seen / .last_seen
//   plus k8s.cluster.name · k8s.namespace.name · k8s.pod.name · k8s.workload.name · k8s.node.name
// NOTE: event.type is CUSTOM_INFO on every one of these — it is NOT "Warning"; severity lives in
// dt.kubernetes.event.important. Enumerate reasons with:
//   fetch events, from:-24h | filter event.provider == "KUBERNETES_EVENT"
//   | summarize n = count(), by:{dt.kubernetes.event.reason} | sort n desc
fetch events, from:-24h
| filter event.provider == "KUBERNETES_EVENT"
| filter in(dt.kubernetes.event.reason, {"BackOff", "BackoffLimitExceeded"})
| summarize events = count(), by:{k8s.namespace.name, dt.kubernetes.event.involved_object.name}
| sort events desc
| limit 25
```

```dql
// Volume mount and storage failures
// Data object corrected 08/12/2026. Kubernetes events are NOT logs. This cell scraped
// `fetch logs` for `log.source` containing "kubernetes" or for content substrings like "BackOff" —
// no log.source matches, and kubelet event text is not in the log stream, so it returned nothing
// while the cluster emitted 231,296 Kubernetes events in the same window.
// They arrive as `fetch events | filter event.provider == "KUBERNETES_EVENT"`, STRUCTURED:
//   dt.kubernetes.event.reason            Unhealthy · BackOff · Killing · FailedScheduling ·
//                                         FailedMount · BackoffLimitExceeded · EvictionThresholdMet …
//   dt.kubernetes.event.message           the human-readable text
//   dt.kubernetes.event.important         "true" marks the warning-class events
//   dt.kubernetes.event.involved_object.kind / .name
//   dt.kubernetes.event.count / .first_seen / .last_seen
//   plus k8s.cluster.name · k8s.namespace.name · k8s.pod.name · k8s.workload.name · k8s.node.name
// NOTE: event.type is CUSTOM_INFO on every one of these — it is NOT "Warning"; severity lives in
// dt.kubernetes.event.important. Enumerate reasons with:
//   fetch events, from:-24h | filter event.provider == "KUBERNETES_EVENT"
//   | summarize n = count(), by:{dt.kubernetes.event.reason} | sort n desc
fetch events, from:-7d
| filter event.provider == "KUBERNETES_EVENT"
| filter in(dt.kubernetes.event.reason, {"FailedMount", "FailedAttachVolume", "FreeDiskSpaceFailed", "EvictionThresholdMet"})
| fields timestamp, k8s.namespace.name, dt.kubernetes.event.reason, dt.kubernetes.event.message
| sort timestamp desc
| limit 25
```

```dql
// Event frequency by reason (last 24h)
// Data object corrected 08/12/2026. Kubernetes events are NOT logs. This cell scraped
// `fetch logs` for `log.source` containing "kubernetes" or for content substrings like "BackOff" —
// no log.source matches, and kubelet event text is not in the log stream, so it returned nothing
// while the cluster emitted 231,296 Kubernetes events in the same window.
// They arrive as `fetch events | filter event.provider == "KUBERNETES_EVENT"`, STRUCTURED:
//   dt.kubernetes.event.reason            Unhealthy · BackOff · Killing · FailedScheduling ·
//                                         FailedMount · BackoffLimitExceeded · EvictionThresholdMet …
//   dt.kubernetes.event.message           the human-readable text
//   dt.kubernetes.event.important         "true" marks the warning-class events
//   dt.kubernetes.event.involved_object.kind / .name
//   dt.kubernetes.event.count / .first_seen / .last_seen
//   plus k8s.cluster.name · k8s.namespace.name · k8s.pod.name · k8s.workload.name · k8s.node.name
// NOTE: event.type is CUSTOM_INFO on every one of these — it is NOT "Warning"; severity lives in
// dt.kubernetes.event.important. Enumerate reasons with:
//   fetch events, from:-24h | filter event.provider == "KUBERNETES_EVENT"
//   | summarize n = count(), by:{dt.kubernetes.event.reason} | sort n desc
fetch events, from:-24h
| filter event.provider == "KUBERNETES_EVENT"
| summarize events = count(), by:{dt.kubernetes.event.reason, dt.kubernetes.event.important}
| sort events desc
| limit 25
```

<a id="log-analysis-patterns"></a>
## 6. Log Analysis Patterns
### Error Log Investigation

```dql
// Pattern: Find errors with full context
fetch logs
| filter loglevel == "ERROR"
| filter k8s.namespace.name == "checkout"
| fields timestamp, k8s.pod.name, content
| sort timestamp desc
| limit 50
```

### Log Volume Analysis

```dql
// Pattern: Identify noisy pods
fetch logs, from: now() - 1h
| summarize count = count(), by:{k8s.pod.name}
| sort count desc
| limit 10
```

### Exception Tracking

```dql
// Pattern: Find stack traces
fetch logs
| filter matchesPhrase(content, "Exception") or matchesPhrase(content, "Traceback")
| fields timestamp, k8s.namespace.name, content
| sort timestamp desc
```

```dql
// Log volume by pod (find noisy pods)
fetch logs, from: now() - 1h
| filter isNotNull(k8s.pod.name)
| summarize logCount = count(), by:{k8s.pod.name}
| sort logCount desc
| limit 15
```

```dql
// Exception and error messages
fetch logs, from:-1h
| filter matchesPhrase(content, "Exception") or matchesPhrase(content, "error") or matchesPhrase(content, "failed")
| filter isNotNull(k8s.namespace.name)
| fields timestamp, k8s.namespace.name, k8s.pod.name, content
| sort timestamp desc
| limit 30
```

```dql
// Log level distribution by namespace
fetch logs, from: now() - 1h
| filter isNotNull(k8s.namespace.name) and isNotNull(loglevel)
| summarize count = count(), by:{k8s.namespace.name, loglevel}
| sort count desc
| limit 30
```

<a id="alerting-on-events-and-logs"></a>
## 7. Alerting on Events and Logs
### Event-Based Alerts

| Alert | Condition | Severity |
|-------|-----------|----------|
| **Failed Scheduling** | FailedScheduling events > 5 in 10 min | Warning |
| **Crash Loop** | CrashLoopBackOff events | Warning |
| **OOM Kills** | OOMKilled events | Critical |
| **Volume Failures** | FailedMount events | Critical |

### Log-Based Alerts

| Alert | Condition | Severity |
|-------|-----------|----------|
| **Error Spike** | Error log count > baseline | Warning |
| **Critical Errors** | Specific error patterns | Critical |
| **No Logs** | Log volume drops to 0 | Warning |

### Custom Metric Events

Create metric events for log-based alerting:

1. Navigate to **Settings > Anomaly detection > Custom events**
2. Create event based on log count metric
3. Set thresholds and notification targets

## Next Steps

With event and log monitoring configured, proceed to:

| Next Notebook | Topic |
|---------------|-------|
| **K8S-08: DQL for Kubernetes** | Advanced query patterns |
| **K8S-09: Troubleshooting** | Debugging K8s monitoring |

---

## Summary

In this notebook, you learned:

- Kubernetes event types and sources
- Event ingestion configuration via DynaKube
- Container log collection methods
- OpenPipeline for log processing
- Event analysis patterns for common issues
- Log analysis patterns for debugging
- Alerting strategies for events and logs

---

## References

- [Kubernetes log monitoring (DT docs)](https://docs.dynatrace.com/docs/ingest-from/setup-on-k8s/deployment/k8s-log-monitoring)
- [Set up Dynatrace on Kubernetes (DT docs)](https://docs.dynatrace.com/docs/ingest-from/setup-on-k8s)
- [Logs (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/logs)
- [Log processing with OpenPipeline (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/logs/lma-log-processing/lma-openpipeline)
- [OpenPipeline (DT docs)](https://docs.dynatrace.com/docs/platform/openpipeline)
- [Kubernetes app — events view (DT docs)](https://docs.dynatrace.com/docs/observe/infrastructure-observability/kubernetes-app)
- [Dynatrace Operator releases (Dynatrace GitHub)](https://github.com/Dynatrace/dynatrace-operator/releases)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
