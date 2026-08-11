# K8S-13: Kafka Monitoring with Kpow

> **Series:** K8S — Kubernetes Monitoring | **Notebook:** 13 of 13 | **Created:** February 2026 | **Last Updated:** 08/11/2026

## Overview

Kpow (by Factor House) is an all-in-one toolkit for managing, monitoring, and learning about Apache Kafka clusters. When deployed on Kubernetes alongside Dynatrace, you get deep Kafka observability by combining Kpow's calculated telemetry with Dynatrace's infrastructure monitoring and alerting capabilities.

This notebook covers deploying Kpow on Kubernetes, scraping its Prometheus metrics into Dynatrace, writing DQL queries against Kafka health data, and building alerts for consumer lag and broker issues.

---

## Table of Contents

1. [Kpow Architecture and Metrics](#kpow-architecture-and-metrics)
2. [Deploying Kpow on Kubernetes](#deploying-kpow-on-kubernetes)
3. [Configuring Dynatrace Prometheus Scraping](#configuring-dynatrace-prometheus-scraping)
4. [Querying Kpow Metrics with DQL](#querying-kpow-metrics-with-dql)
5. [Monitoring Kpow as a Kubernetes Workload](#monitoring-kpow-as-a-kubernetes-workload)
6. [Alerting on Kafka Health](#alerting-on-kafka-health)
7. [Combining with Native Kafka Monitoring](#combining-with-native-kafka-monitoring)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Dynatrace Environment** | SaaS with Kubernetes monitoring enabled |
| **Kubernetes Cluster** | Dynatrace Operator v1.0+ with ActiveGate |
| **Apache Kafka** | Running Kafka cluster accessible from K8s |
| **Kpow License** | Community or Enterprise license from Factor House |
| **Helm** | v3+ for Kpow deployment |
| **Knowledge** | Completed K8S-01, K8S-05 (Workload Monitoring) |

<a id="kpow-architecture-and-metrics"></a>
## 1. Kpow Architecture and Metrics

### What Kpow Provides

Kpow calculates its own telemetry about your Kafka cluster independently of Kafka's built-in JMX metrics. This means:

- **No JMX dependency** — frictionless installation, no broker-side configuration
- **Calculated metrics** — consumer group lag, topic throughput, offset deltas
- **Multi-resource support** — brokers, topics, consumer groups, Kafka Connect, Schema Registry, ksqlDB

### Prometheus Metric Endpoints

Kpow exposes OpenMetrics-compatible endpoints when `PROMETHEUS_EGRESS=true`:

| Endpoint | Description |
|----------|-------------|
| `GET /metrics/v1` | All metrics across the system |
| `GET /metrics/v1/cluster/:cluster-id` | Cluster-specific metrics |
| `GET /metrics/v1/connect/:connect-id` | Kafka Connect metrics |
| `GET /metrics/v1/schema/:schema-id` | Schema Registry metrics |
| `GET /metrics/v1/ksqldb/:ksqldb-id` | ksqlDB metrics |
| `GET /offsets/v1` | Topic offset data |
| `GET /group-offsets/v1` | Consumer group offset data |
| `GET /streams/v1` | Kafka Streams metrics |

### Key Metrics

| Metric | Type | Description |
|--------|------|-------------|
| `broker_count` | Gauge | Number of brokers in the Kafka cluster |
| `topic_count` | Gauge | Number of topics in the cluster |
| `group_count` | Gauge | Number of consumer groups |
| `group_offset_lag` | Histogram | Consumer group lag across assignments |
| `group_state` | Gauge | Consumer group state (0=DEAD, 1=EMPTY, 2=UNKNOWN, 4=STABLE) |
| `broker_end_delta` | Histogram | Production rate per broker |
| `topic_urp_total` | Meter | Under-replicated partitions |
| `acl_count` | Gauge | Number of ACLs in the cluster |
| `connect_connector_task_state` | Gauge | Connector task state (1=RUNNING) |
| `connect_connector_running_total` | Gauge | Number of active connectors |
| `partition_end` | Gauge | Topic partition end offset |
| `group_assignment_offset` | Gauge | Current consumer position |

### Metric Label Structure

All Kpow metrics use consistent labels:

| Label | Description | Example |
|-------|-------------|----------|
| `domain` | Category of metric | `cluster`, `connect`, `streams` |
| `id` | Unique identifier | Kafka Cluster ID |
| `target` | Resource identifier | Consumer group name, topic name |
| `env` | Environment name | `Production`, `Staging` |

> **Note:** Non-compliant characters in Kafka resource names (topics, groups) are converted to underscores to meet Prometheus naming standards.

<a id="deploying-kpow-on-kubernetes"></a>
## 2. Deploying Kpow on Kubernetes

### Helm Chart Installation

Factor House provides an official Helm chart for Kubernetes deployment.

**Step 1: Add the Helm repository**

```bash
helm repo add factorhouse https://charts.factorhouse.io
helm repo update
```

**Step 2: Create the namespace and license secret**

```bash
kubectl create namespace kpow

# Create a secret for your Kpow license
kubectl create secret generic kpow-license \
  --namespace kpow \
  --from-literal=LICENSE_ID=your-license-id \
  --from-literal=LICENSE_CODE=your-license-code
```

**Step 3: Create a `values.yaml` with Prometheus enabled and Dynatrace annotations**

```yaml
# kpow-values.yaml
replicaCount: 1

env:
  # Kafka cluster connection
  BOOTSTRAP: "kafka-broker-0.kafka:9092,kafka-broker-1.kafka:9092"
  
  # Enable Prometheus metrics export
  PROMETHEUS_EGRESS: "true"
  
  # Optional: Basic auth for metrics endpoints
  # PROMETHEUS_USERNAME: "metrics"
  # PROMETHEUS_PASSWORD: "<from-secret>"

# Kpow serves on port 3000 by default
service:
  port: 3000

resources:
  requests:
    cpu: 500m
    memory: 1Gi
  limits:
    cpu: 2000m
    memory: 2Gi

# Dynatrace Prometheus scrape annotations
podAnnotations:
  metrics.dynatrace.com/scrape: "true"
  metrics.dynatrace.com/port: "3000"
  metrics.dynatrace.com/path: "/metrics/v1"
```

**Step 4: Install Kpow**

```bash
helm install kpow factorhouse/kpow \
  --namespace kpow \
  --values kpow-values.yaml
```

**Step 5: Verify deployment**

```bash
# Check pods are running
kubectl get pods -n kpow

# Verify Prometheus endpoints are accessible
kubectl port-forward svc/kpow 3000:3000 -n kpow
curl http://localhost:3000/metrics/v1
```

### Resource Sizing Guidelines

| Kafka Cluster Size | Kpow CPU | Kpow Memory | Notes |
|--------------------|----------|-------------|-------|
| Small (< 10 topics) | 500m | 1Gi | Dev/test environments |
| Medium (10-100 topics) | 1000m | 2Gi | Standard production |
| Large (100+ topics) | 2000m | 4Gi | High-throughput clusters |
| Multi-cluster | 2000m | 4Gi | Monitoring multiple clusters |

<a id="configuring-dynatrace-prometheus-scraping"></a>
## 3. Configuring Dynatrace Prometheus Scraping

### How Dynatrace Scrapes Prometheus Metrics

Dynatrace collects metrics from any Kubernetes pod annotated with `metrics.dynatrace.com/scrape: "true"`. The Dynatrace Operator's ActiveGate connects directly to annotated pods and scrapes their Prometheus endpoints, enriching the metrics with Kubernetes topology information.

> **Important:** The ActiveGate must be deployed **inside** the monitored Kubernetes cluster for annotation-based scraping to work. An ActiveGate running outside the cluster cannot reach pod endpoints that require RBAC or network-level access.

### Required Pod Annotations

| Annotation | Required | Default | Description |
|------------|----------|---------|-------------|
| `metrics.dynatrace.com/scrape` | Yes | - | Set to `"true"` to enable scraping |
| `metrics.dynatrace.com/port` | No | First TCP port | Port where metrics are exposed |
| `metrics.dynatrace.com/path` | No | `/metrics` | Path to the metrics endpoint |
| `metrics.dynatrace.com/secure` | No | `"false"` | Use HTTPS for scraping |
| `metrics.dynatrace.com/insecure_skip_verify` | No | `"false"` | Skip TLS certificate verification (self-signed certs) |
| `metrics.dynatrace.com/filter` | No | - | Include/exclude metrics matching pattern (supports `*` wildcard) |

### Applying Annotations to Kpow

If you deployed Kpow via Helm with `podAnnotations` (as shown above), the annotations are already applied. To verify:

```bash
kubectl get pods -n kpow -o jsonpath='{.items[0].metadata.annotations}' | python3 -m json.tool
```

To add annotations to an existing Kpow deployment:

```bash
kubectl annotate pods -n kpow -l app=kpow \
  metrics.dynatrace.com/scrape="true" \
  metrics.dynatrace.com/port="3000" \
  metrics.dynatrace.com/path="/metrics/v1"
```

### Scraping Multiple Endpoints

Kpow exposes several metric endpoints. The primary `/metrics/v1` endpoint includes all cluster metrics. For offset-specific data, you can deploy a sidecar or use additional scrape targets.

To scrape the offsets endpoint separately, add a second annotated service:

```yaml
# Additional Service for offset metrics
apiVersion: v1
kind: Service
metadata:
  name: kpow-offsets
  namespace: kpow
  annotations:
    metrics.dynatrace.com/scrape: "true"
    metrics.dynatrace.com/port: "3000"
    metrics.dynatrace.com/path: "/offsets/v1"
spec:
  selector:
    app: kpow
  ports:
    - port: 3000
      targetPort: 3000
```

### Enabling Prometheus Monitoring in Dynatrace

Ensure Prometheus scraping is enabled in your Dynatrace environment:

1. Navigate to **Settings > Cloud and virtualization > Kubernetes**
2. Select your cluster connection
3. Enable **Monitor annotated Prometheus exporters**
4. Metrics will appear within minutes of annotation application

> **Note:** Dynatrace ingests Prometheus Counter, Gauge, Histogram, and Summary metric types. All Kpow metrics are compatible.

```dql
// Verify Kpow metrics are being ingested
timeseries brokerCount = avg(broker_count), from:-1h
| fieldsAdd avgBrokers = arrayAvg(brokerCount)
```

<a id="querying-kpow-metrics-with-dql"></a>
## 4. Querying Kpow Metrics with DQL

Once Dynatrace scrapes Kpow's Prometheus endpoints, the metrics are available in Grail for DQL queries. Prometheus metrics are queryable using the `timeseries` command.

### Cluster Health Overview

```dql
// Kafka cluster overview: brokers, topics, consumer groups
timeseries brokers = avg(broker_count), from:-1h
| append [
    timeseries topics = avg(topic_count), from:-1h
  ]
| append [
    timeseries groups = avg(group_count), from:-1h
  ]
```

### Consumer Group Lag

```dql
// Consumer group lag over time
timeseries lag = avg(group_offset_lag), from:-1h, by:{target}
| fieldsAdd avgLag = arrayAvg(lag)
| sort avgLag desc
| limit 10
```

### Consumer Group State

```dql
// Consumer group state: 0=DEAD, 1=EMPTY, 2=UNKNOWN, 4=STABLE
timeseries state = avg(group_state), from:-1h, by:{target}
| fieldsAdd currentState = arrayLast(state)
| fieldsAdd stateLabel = if(currentState == 0, then: "DEAD",
    else: if(currentState == 1, then: "EMPTY",
    else: if(currentState == 2, then: "UNKNOWN",
    else: if(currentState == 4, then: "STABLE", else: "OTHER"))))
| fields target, currentState, stateLabel
| sort stateLabel asc
```

### Under-Replicated Partitions

```dql
// Under-replicated partitions trend (critical health indicator)
timeseries urp = avg(topic_urp_total), from:-6h
| fieldsAdd avgUrp = arrayAvg(urp)
| filter avgUrp > 0
```

### Broker Production Rate

```dql
// Broker production rate (messages produced per broker)
timeseries delta = avg(broker_end_delta), from:-1h, by:{target}
| fieldsAdd avgDelta = arrayAvg(delta)
| sort avgDelta desc
| limit 10
```

### Kafka Connect Health

```dql
// Kafka Connect: connector task states (1=RUNNING)
timeseries taskState = avg(connect_connector_task_state), from:-1h, by:{target}
| fieldsAdd currentTaskState = arrayLast(taskState)
| fieldsAdd healthy = if(currentTaskState == 1, then: "RUNNING", else: "NOT RUNNING")
| fields target, healthy, currentTaskState
```

<a id="monitoring-kpow-as-a-kubernetes-workload"></a>
## 5. Monitoring Kpow as a Kubernetes Workload

Beyond the Kafka metrics Kpow exposes, you should also monitor Kpow itself as a Kubernetes workload. Kpow runs as a JVM process, so Dynatrace OneAgent provides:

| Signal | What It Shows |
|--------|---------------|
| **Pod health** | Restarts, OOMKilled events, scheduling issues |
| **CPU/Memory** | Resource consumption of the Kpow process |
| **JVM metrics** | Heap usage, garbage collection, threads |
| **Logs** | Kpow application logs for troubleshooting |

### Kpow Pod Resource Usage

```dql
// Kpow pod CPU and memory usage
timeseries cpuUsage = avg(dt.kubernetes.container.cpu_usage), from:-1h,
  by:{k8s.namespace.name, k8s.pod.name},
  filter:{k8s.namespace.name == "kpow"}
| fieldsAdd avgCpu = arrayAvg(cpuUsage)
| sort avgCpu desc
```

```dql
// Kpow pod memory working set
timeseries memUsage = avg(dt.kubernetes.container.memory_working_set), from:-1h,
  by:{k8s.namespace.name, k8s.pod.name},
  filter:{k8s.namespace.name == "kpow"}
| fieldsAdd avgMemMB = arrayAvg(memUsage) / 1048576
| sort avgMemMB desc
```

### Kpow Application Logs

```dql
// Kpow error logs
fetch logs, from:-1h
| filter k8s.namespace.name == "kpow"
| filter loglevel == "ERROR" or loglevel == "WARN"
| fields timestamp, loglevel, content, k8s.pod.name
| sort timestamp desc
| limit 50
```

### Kpow Pod Restart History

```dql
// Kpow pod restarts and events
fetch events, from:-24h
| filter contains(toString(affected_entity_ids), "kpow")
| fields timestamp, event.name, event.kind
| sort timestamp desc
| limit 20
```

<a id="alerting-on-kafka-health"></a>
## 6. Alerting on Kafka Health

Kpow does not provide its own alerting — it delegates to external tools. With Dynatrace ingesting Kpow metrics, you can use Dynatrace metric events, Workflows, and the new Dynatrace Intelligence Agents for alerting and automated response.

### Recommended Alert Thresholds

| Alert | Metric | Condition | Severity |
|-------|--------|-----------|----------|
| **Consumer lag spike** | `group_offset_lag` | Lag > 10,000 for 5 min | Warning |
| **Consumer lag critical** | `group_offset_lag` | Lag > 100,000 for 5 min | Critical |
| **Consumer group dead** | `group_state` | State == 0 (DEAD) | Critical |
| **Under-replicated partitions** | `topic_urp_total` | URP > 0 for 5 min | Critical |
| **Broker count drop** | `broker_count` | Count < expected | Critical |
| **Connector task failure** | `connect_connector_task_state` | State != 1 | Warning |
| **Kpow pod unhealthy** | K8s pod status | Restart count > 3 | Warning |

### Setting Up Metric Events

Create metric events in Dynatrace for automated alerting:

1. Navigate to **Settings > Anomaly detection > Metric events**
2. Click **Add metric event**
3. Configure:

| Field | Consumer Lag Example |
|-------|---------------------|
| **Summary** | Kafka consumer lag critical |
| **Metric key** | `group_offset_lag` |
| **Aggregation** | Average |
| **Threshold** | 100,000 |
| **Violating samples** | 3 of 5 (sliding window) |
| **Event type** | Custom alert |
| **Severity** | Error |

### Workflow Integration

Connect metric events to Dynatrace Workflows for automated response:

| Trigger | Action |
|---------|--------|
| Consumer lag > threshold | Notify Slack channel, create Jira ticket |
| Broker count drops | Page on-call, trigger runbook |
| URP > 0 sustained | Alert Kafka admin team |
| Connector task fails | Auto-restart connector via API |

### Dynatrace Intelligence Agents

Announced at Perform 2026, Dynatrace Intelligence Agents can automate incident response by reasoning over real-time causal context. For Kafka scenarios, this means:

- **SRE Agents** can correlate Kpow consumer lag spikes with upstream service errors detected by OneAgent
- **Automated triage** links Kafka health metrics to Dynatrace Intelligence root-cause analysis
- **Cross-domain correlation** connects Kafka platform metrics (via Kpow) with application traces and Kubernetes events

> **Tip:** See **WFLOW-01** through **WFLOW-09** for comprehensive Dynatrace Workflow configuration patterns.

<a id="combining-with-native-kafka-monitoring"></a>
## 7. Combining with Native Kafka Monitoring

### Dynatrace's Built-in Kafka Support

Dynatrace provides native Kafka monitoring through OneAgent, which automatically detects Kafka broker processes and collects JMX-based metrics.

| Capability | OneAgent (Native) | Kpow |
|------------|-------------------|------|
| Broker process metrics | Yes | No |
| JMX metrics | Yes | No |
| Consumer group lag | Limited | Yes (detailed) |
| Topic offset tracking | No | Yes |
| Kafka Connect monitoring | No | Yes |
| Schema Registry monitoring | No | Yes |
| ksqlDB monitoring | No | Yes |
| Distributed traces | Yes | No |
| Code-level visibility | Yes | No |

### Recommended Combination Strategy

Use both together for comprehensive Kafka observability:

| Layer | Tool | What It Provides |
|-------|------|------------------|
| **Infrastructure** | Dynatrace OneAgent | Broker process health, JVM metrics, host metrics |
| **Application** | Dynatrace OneAgent | Distributed traces through Kafka producers/consumers |
| **Kafka Platform** | Kpow → Dynatrace | Consumer lag, topic throughput, Connect/Schema/ksqlDB |
| **Kubernetes** | Dynatrace Operator | Pod health, resource usage, events for all components |

### Dynatrace Apache Kafka Extension

For environments where Kpow is not available, Dynatrace also offers the **Apache Kafka extension** on Dynatrace Hub. This extension provides:

- Broker-level metrics via JMX
- Topic and partition statistics
- Consumer group metrics
- Pre-built dashboards

Install from: **Dynatrace Hub > Apache Kafka**

```dql
// Combine: Kpow consumer lag with OneAgent service traces
// Step 1: Check service-level error rates for Kafka consumers
//
// Field notes — all three fail silently if you get them wrong:
//   span.status_code is the real field (otel.status_code does not exist),
//   its values are lowercase ("error", never "ERROR"),
//   and it is null on successful spans, so derive successes as total - errors.
//
// If this returns nothing, check span.kind before assuming a healthy fleet:
//   fetch spans, from:-1h | summarize c = count(), by:{span.kind}
// A cluster whose Kafka clients are not instrumented emits no "consumer" spans at
// all — which is a coverage gap, not a clean bill of health.
fetch spans, from:-1h
| filter span.kind == "consumer"
| summarize {
    total = count(),
    errors = countIf(span.status_code == "error")
  }, by:{service.name}
| fieldsAdd successes = total - errors
| fieldsAdd errorRate = round(100.0 * errors / total, decimals: 2)
| sort errorRate desc
| limit 10
```

```dql
// Check Kafka broker processes detected by OneAgent
fetch dt.entity.process_group_instance
| filter contains(entity.name, "kafka") or contains(entity.name, "Kafka")
| fields entity.name, tags
| sort entity.name asc
| limit 20

// Smartscape equivalent (dt.entity.* is deprecated but still functional):
//   smartscapeNodes "PROCESS"
//   | filter contains(name, "kafka") or contains(name, "Kafka")
//   | fields name, tags
//   | sort name asc
//   | limit 20
// Caveat: Smartscape reflects CURRENT live topology and can report fewer entities than
// the classic entity store; for a pre-migration inventory keep the classic query above.
// Note: entity tags are not a flat "tags" field on Smartscape (resolve via getNodeField).
```

## Summary

In this notebook, you learned:

- **Kpow architecture** and the Prometheus metrics it exposes for Kafka clusters
- **Kubernetes deployment** of Kpow using the official Helm chart
- **Dynatrace Prometheus scraping** via pod annotations for automatic metric ingestion
- **DQL queries** for consumer lag, group state, under-replicated partitions, and broker production rates
- **Workload monitoring** of the Kpow pod itself (CPU, memory, logs, restarts)
- **Alerting patterns** using Dynatrace metric events and Workflows
- **Combining Kpow with native Kafka monitoring** for comprehensive observability

---

## Next Steps

| Next Notebook | Topic |
|---------------|-------|
| **K8S-11: Multi-Tool Coexistence** | Running Dynatrace alongside other monitoring tools |
| **K8S-05: Workload Monitoring** | Deep dive into Kubernetes workload observability |
| **WFLOW-01: Workflow Fundamentals** | Setting up automated alerting workflows |

---

## References

- [Kpow documentation (Factor House)](https://docs.factorhouse.io)
- [Kpow Prometheus integration (Factor House)](https://docs.factorhouse.io/kpow-ee/features/prometheus/)
- [Kpow metrics glossary (Factor House)](https://docs.factorhouse.io/kpow/integration/prometheus/metrics-glossary)
- [Kpow Helm charts (Factor House)](https://charts.factorhouse.io)
- [Monitor Prometheus metrics on K8s (DT docs)](https://docs.dynatrace.com/docs/observe/infrastructure-observability/container-platform-monitoring/kubernetes-monitoring/monitor-prometheus-metrics)
- [Apache Kafka technology support (DT docs)](https://docs.dynatrace.com/docs/ingest-from/technology-support/dynatrace-extensions/supported-out-of-the-box/kafka)
- [Apache Kafka extension (Dynatrace Hub)](https://www.dynatrace.com/hub/detail/apache-kafka/)
- [Set up Dynatrace on Kubernetes (DT docs)](https://docs.dynatrace.com/docs/ingest-from/setup-on-k8s)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
