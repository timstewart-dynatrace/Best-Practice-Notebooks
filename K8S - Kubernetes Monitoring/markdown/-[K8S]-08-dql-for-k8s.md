# K8S-08: DQL Queries for Kubernetes

> **Series:** K8S — Kubernetes Monitoring | **Notebook:** 8 of 13 | **Created:** January 2026 | **Last Updated:** 09/24/2026

## Advanced Query Patterns for Kubernetes Data
This notebook provides a comprehensive reference of DQL queries for Kubernetes monitoring. From basic entity queries to complex performance analysis, these patterns help you extract insights from your Kubernetes data.

---

## Table of Contents

1. [Entity Queries](#entity-queries)
2. [Metric Queries](#metric-queries)
3. [Log and Event Queries](#log-and-event-queries)
4. [Trace Queries](#trace-queries)
5. [Correlation Queries](#correlation-queries)
6. [Dashboard Queries](#dashboard-queries)
7. [Alerting Queries](#alerting-queries)
8. [Query Templates](#query-templates)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Dynatrace Environment** | SaaS with Kubernetes monitoring |
| **Permissions** | `metrics.read`, `entities.read`, `logs.read`, `events.read` |
| **Knowledge** | K8S-01 through K8S-07 |
| **Data** | Active Kubernetes cluster monitored |

<a id="entity-queries"></a>
## 1. Entity Queries
### Kubernetes Entity Types

| Entity Type | DQL Table | Description |
|-------------|-----------|-------------|
| Cluster | `dt.entity.kubernetes_cluster` | K8s clusters |
| Node | `dt.entity.kubernetes_node` | Worker nodes |
| Namespace | `dt.entity.cloud_application_namespace` | Namespaces |
| Workload | `dt.entity.cloud_application` | Deployments, StatefulSets |
| Container | `dt.entity.container_group_instance` | Container instances |
| Service | `dt.entity.service` | Detected services |

```dql
// List all Kubernetes clusters (smartscape topology)
smartscapeNodes "K8S_CLUSTER"
| fields entity.name = name, tags
| sort entity.name asc

// Legacy alternative (deprecated for new content):
// fetch dt.entity.kubernetes_cluster
// | fields entity.name, tags
// | sort entity.name asc

```

```dql
// List all Kubernetes nodes (smartscape topology)
smartscapeNodes "K8S_NODE"
| fields entity.name = name, tags
| sort entity.name asc

// Legacy alternative (deprecated for new content):
// fetch dt.entity.kubernetes_node
// | fields entity.name, tags
// | sort entity.name asc

```

```dql
// List all namespaces (smartscape topology)
smartscapeNodes "K8S_NAMESPACE"
| fields entity.name = name, tags
| sort entity.name asc

// Legacy alternative (deprecated for new content):
// fetch dt.entity.cloud_application_namespace
// | fields entity.name, tags
// | sort entity.name asc

```

```dql
// List all workloads — Deployments (smartscape topology)
smartscapeNodes "K8S_DEPLOYMENT"
| fields entity.name = name, tags
| sort entity.name asc
| limit 50

// For other workload types, substitute:
// smartscapeNodes "K8S_STATEFULSET"
// smartscapeNodes "K8S_DAEMONSET"

// Legacy alternative (deprecated for new content):
// fetch dt.entity.cloud_application
// | fields entity.name, tags
// | sort entity.name asc
// | limit 50

```

```dql
// Count Kubernetes nodes (smartscape topology)
smartscapeNodes "K8S_NODE"
| summarize nodeCount = count()
```

<a id="metric-queries"></a>
## 2. Metric Queries
### Common Kubernetes Metrics

The table below lists keys verified present on a live tenant (`yhu28601`, 08/11/2026). Enumerate your own before writing a query — this is what exists there, not a guarantee for every estate:

```dql
metrics
| filter startsWith(metric.key, "dt.kubernetes")
| summarize n = count(), by:{metric.key}
| sort metric.key asc
```

> Two mechanics that bite here. **`metrics` takes `from:` with no leading comma** — `metrics from:-2h | …`. Writing `metrics, from:-2h` is a `PARSE_ERROR`, and a parse error read only for its record count is indistinguishable from "this tenant carries no such metrics." And **`metrics` returns one row per dimension tuple**, not one per key, so `summarize by:{metric.key}` is what collapses the result into a key list.

| Metric | Description | Unit | Grain |
|--------|-------------|------|-------|
| `dt.kubernetes.container.cpu_usage` | Container CPU usage | Millicores | container |
| `dt.kubernetes.container.memory_working_set` | Container memory (active) | Bytes | container |
| `dt.kubernetes.container.requests_cpu` | CPU requests | Millicores | container |
| `dt.kubernetes.container.requests_memory` | Memory requests | Bytes | container |
| `dt.kubernetes.container.limits_cpu` | CPU limits | Millicores | container |
| `dt.kubernetes.container.limits_memory` | Memory limits | Bytes | container |
| `dt.kubernetes.container.cpu_throttled` | CPU throttling | — | container |
| `dt.kubernetes.container.restarts` | Restart count (delta) | Count | container |
| `dt.kubernetes.container.oom_kills` | OOMKilled count | Count | container |
| `dt.containers.cpu.throttled_time` | CPU throttle time | Nanoseconds | container |
| `dt.kubernetes.node.cpu_allocatable` | Node allocatable CPU | Millicores | node |
| `dt.kubernetes.node.memory_allocatable` | Node allocatable memory | Bytes | node |
| `dt.kubernetes.node.pods_allocatable` | Node allocatable pod slots | Count | node |
| `dt.kubernetes.node.conditions` | Node condition status | — | node |
| `dt.kubernetes.pod.containers_desired` | Desired containers per pod | Count | pod |
| `dt.kubernetes.pod.network_received_data` / `.network_transmitted_data` | Pod network throughput | Bytes | pod |
| `dt.kubernetes.workload.pods_desired` | Desired pods per workload | Count | workload |
| `dt.kubernetes.workload.conditions` | Workload condition status | — | workload |
| `dt.kubernetes.nodes` / `.pods` / `.containers` / `.workloads` / `.events` | Object counts | Count | cluster |

> **There is no node- or workload-grain *usage* or *requests* metric.** `dt.kubernetes.node.cpu_usage`, `dt.kubernetes.node.memory_usage`, `dt.kubernetes.workload.requests_cpu` and `dt.kubernetes.workload.requests_memory` **do not exist** — each returns zero rows from `metrics | filter metric.key == "…"` (checked 08/11/2026). Node and workload views are **derived**: sum the `dt.kubernetes.container.*` series across `k8s.node.name` or `k8s.workload.name`. Only the `*_allocatable` family is genuinely node-grain.
>
> This is worth stating plainly because a `timeseries` against a key that does not exist is syntactically valid, executes without error, and returns nothing — the same shape as a correct query against an idle cluster. §5 below shows the derivation for nodes.

> **Reading request and limit metrics across ActiveGate upgrades.** The CPU and memory *request* and *limit* metrics carry an **effective pod value**. Where a simple sum of container values would not match what Kubernetes enforces, Dynatrace adds a **compensating delta data point** to the same metric at ingest — in the docs' words, *"Dynatrace calculates the correct effective value at ingest time and adjusts the metric so that aggregated views across pods, workloads, namespaces, and clusters always reflect what Kubernetes actually enforces."* What that effective value covers grew in ActiveGate steps:
>
> | ActiveGate | What the request/limit metrics account for |
> |---|---|
> | **1.339+** | App containers and native sidecars (`restartPolicy: Always`) |
> | **1.343+** | Init containers — a delta where the init phase peaks above the running-phase sum |
> | **1.345+** (rollout from 08/25/2026) | Pod-level `spec.resources` (Kubernetes 1.34+) and RuntimeClass `overhead` — including a **negative** delta where a pod-level *limit* caps the total below the container sum |
> | **1.347** (pre-release; rollout planned from 09/22/2026) | Fix for a miscalculated effective value when a native sidecar precedes one or more non-restartable init containers |
>
> | Metric family | Affected by the effective-resource changes? |
> |---|---|
> | `dt.kubernetes.container.requests_cpu`, `.requests_memory`, `.limits_cpu`, `.limits_memory` | **Yes** — the effective value changes at each boundary above |
> | `dt.kubernetes.container.cpu_usage`, `.memory_working_set` | **No** — usage metrics are unchanged |
>
> Three practical consequences:
>
> 1. **A trend that spans any of these ActiveGate versions may carry a discontinuity** — up for init containers, up **or down** for pod-level limits, with no workload change at all. It appears only where the pod shape triggers it (an init container larger than the running-phase sum, a pod-level `spec.resources`, or an `overhead`), so it is not a uniform shift. Do not read a request/limit chart across a boundary as a single trend, and do not let a threshold alert on reserved CPU or memory fire on the discontinuity. Bound your comparison windows on one side of the upgrade or the other.
> 2. **Earlier baselines misstate reservations** relative to what Kubernetes enforces. The post-upgrade figures are the more faithful number — re-baseline rather than trying to reconcile the two.
> 3. **The delta belongs to no container.** The compensating data point has no `dt.smartscape.container` dimension (its `dt.smartscape_source.type` is `K8S_POD`), so a breakdown *by container* shows it as an unnamed row. Summing across it gives the enforced pod value; filtering out rows without a container quietly brings back the old arithmetic.
>
> Because node and namespace roll-ups are *derived* from these container-grain series (see the note above), a change propagates to every level you aggregate to — there is no unaffected roll-up to fall back on. One edge case the docs call out explicitly: *"Upgrading Istio to version 1.27+ (where native sidecars are the default) without also upgrading to ActiveGate version 1.339+ will cause sidecar resources to be missing from the metric."*
>
> Each version rolls out per ActiveGate, not per tenant, and ActiveGate fleets lag tenant version — so **verify the version of the ActiveGate carrying the `kubernetes-monitoring` capability for the cluster in question** before deciding which side of a boundary a data point sits on. Until a given version reaches that ActiveGate, the earlier reading is what your data shows, and everything above about baselines and alert thresholds still describes it correctly.
>
> <sub>Sources: [Effective pod resources (DT docs)](https://docs.dynatrace.com/docs/observe/infrastructure-observability/kubernetes-app/reference/effective-pod-resources), [ActiveGate 1.345 release notes (DT docs)](https://docs.dynatrace.com/docs/whats-new/activegate/sprint-345), [ActiveGate 1.347 release notes (DT docs)](https://docs.dynatrace.com/docs/whats-new/activegate/sprint-347) — read 09/18/2026.</sub>

```dql
// Container CPU usage - top consumers
timeseries avgCpuMillicores = avg(dt.kubernetes.container.cpu_usage), from:-1h, by:{dt.entity.container_group_instance}
| sort avgCpuMillicores desc
| limit 15
```

```dql
// Container memory usage approaching limits
timeseries avgMemBytes = avg(dt.kubernetes.container.memory_working_set), from:-1h, by:{dt.entity.container_group_instance}
| fieldsAdd avgMemBytesValue = arrayAvg(avgMemBytes)
| sort avgMemBytesValue desc
```

```dql
// CPU throttling detection
timeseries totalThrottle = sum(dt.containers.cpu.throttled_time), from:-1h, by:{dt.entity.container_group_instance}
| fieldsAdd totalThrottleValue = arrayAvg(totalThrottle)
| filter totalThrottleValue > 0
| sort totalThrottleValue desc
| limit 15
```

```dql
// Namespace-level resource usage — sum() of every container in the namespace
timeseries cpuMillicores = sum(dt.kubernetes.container.cpu_usage), from:-1h, by:{k8s.namespace.name}
| fieldsAdd avgCpuMillicores = arrayAvg(cpuMillicores)
| sort avgCpuMillicores desc
| limit 15
```

```dql
// Resource requests by namespace (capacity planning)
// Requests are container-grain; sum them to reach the namespace figure.
timeseries cpuReq = sum(dt.kubernetes.container.requests_cpu), from:-1h, by:{k8s.namespace.name}
| fieldsAdd avgReqMillicores = round(arrayAvg(cpuReq), decimals: 0)
| fields k8s.namespace.name, avgReqMillicores
| sort avgReqMillicores desc
| limit 15
```

<a id="log-and-event-queries"></a>
## 3. Log and Event Queries

**Logs and Kubernetes events are two different objects, and the split is not optional.** Application output written to stdout/stderr lands in `logs`. Kubernetes events — `FailedScheduling`, `BackOff`, `Unhealthy`, `Killing` — land in `events`, and are **not** searchable through `logs`.

Getting this wrong is expensive in both directions:

| Anti-pattern | Result on a live tenant (08/11/2026) |
|---|---|
| `fetch logs \| filter matchesPhrase(log.source, "kubernetes")` | **0 rows after scanning 46.5 GB.** Empty *and* billed. |
| `fetch events \| filter event.kind == "K8S_EVENT"` | **0 rows.** `event.kind` never takes that value — over 24 h the only values present were `DAVIS_EVENT`, `SYNTHETIC_EVENT`, `FLEET_EVENT`, `DAVIS_PROBLEM`. Kubernetes events carry `DAVIS_EVENT`, so *kind* cannot discriminate them. |
| `k8s.event.reason` | Field resolves, **null on every record**. |

**The working pair is `event.provider == "KUBERNETES_EVENT"` plus `dt.kubernetes.event.reason`.** Companion fields: `dt.kubernetes.event.message`, `.count`, `.first_seen`, `.last_seen`, `.involved_object.kind`, `.involved_object.name`, alongside `k8s.cluster.name` / `k8s.namespace.name` / `k8s.pod.name` / `k8s.workload.name` / `k8s.node.name`.

### Log Query Patterns

```dql
// Error logs by namespace
fetch logs, from:-1h
| filter loglevel == "ERROR"
| filter isNotNull(k8s.namespace.name)
| summarize errorCount = count(), by:{k8s.namespace.name}
| sort errorCount desc
| limit 15
```

```dql
// Kubernetes events — what your estate actually emits, by reason
fetch events, from:-1h
| filter event.provider == "KUBERNETES_EVENT"
| summarize eventCount = count(), by:{dt.kubernetes.event.reason}
| sort eventCount desc
| limit 30
```

```dql
// Pod crash / restart loops — the BackOff event carries the pod and the reason
fetch events, from:-1h
| filter event.provider == "KUBERNETES_EVENT"
| filter dt.kubernetes.event.reason == "BackOff"
| fields timestamp, k8s.cluster.name, k8s.namespace.name,
         dt.kubernetes.event.involved_object.name,
         dt.kubernetes.event.message
| sort timestamp desc
| limit 20
```

```dql
// OOM kills — a metric, not an event
// Dynatrace derives this counter from the container status and writes it only
// when at least one kill occurred, so any non-null value is a real OOM.
timeseries oom = sum(dt.kubernetes.container.oom_kills), from:-24h,
  by:{k8s.cluster.name, k8s.namespace.name, k8s.workload.name}
| fieldsAdd oomTotal = arraySum(oom)
| filter oomTotal > 0
| fields k8s.cluster.name, k8s.namespace.name, k8s.workload.name, oomTotal
| sort oomTotal desc
| limit 20
```

```dql
// Log volume trend by namespace
fetch logs, from: now() - 24h
| filter isNotNull(k8s.namespace.name)
| summarize log_count = count(), by:{k8s.namespace.name, time_bucket = bin(timestamp, 1h)}
| sort time_bucket asc
| limit 200
```

<a id="trace-queries"></a>
## 4. Trace Queries
### Span Queries for Kubernetes Services

> **`duration` is a duration type — compare it against a duration literal (`1s`, `500ms`), never a nanosecond integer.** `duration > 1000000000` is a type mismatch: it raises only a `DATATYPE_MISMATCH` warning, evaluates to null, and the filter silently drops every span. Spans also have no `timestamp` field — their time fields are `start_time` and `end_time`.

```dql
// Service response times in K8s
fetch spans, from:-1h
| filter span.kind == "server"
| filter isNotNull(k8s.namespace.name)
| summarize 
    p50 = percentile(duration, 50),
    p95 = percentile(duration, 95),
    p99 = percentile(duration, 99),
    by:{k8s.namespace.name, dt.entity.service}
| sort p99 desc
| limit 15
```

```dql
// Error rate by service
// Three things to get right here, each of which fails silently:
//   1. The field is span.status_code (stable). otel.status_code does not exist.
//   2. Values are LOWERCASE — "error", not "ERROR".
//   3. span.status_code is NULL on almost every successful span, so derive
//      successes as total - errors rather than counting a non-error value.
fetch spans, from:-1h
| filter span.kind == "server"
| filter isNotNull(k8s.namespace.name)
| summarize {
    total = count(),
    errors = countIf(span.status_code == "error")
  }, by:{k8s.namespace.name, dt.entity.service}
| fieldsAdd successes = total - errors
| fieldsAdd errorRate = round(100.0 * errors / total, decimals: 2)
| filter errors > 0
| sort errorRate desc
| limit 15
```

```dql
// Slow traces (>1 second)
fetch spans, from:-1h
| filter span.kind == "server" and duration > 1s
| filter isNotNull(k8s.namespace.name)
| fields start_time, trace.id, k8s.namespace.name, span.name, duration
| sort duration desc
| limit 20
```

```dql
// Request throughput by namespace
fetch spans, from:-1h
| filter span.kind == "server"
| filter isNotNull(k8s.namespace.name)
| summarize requestCount = count(), by:{k8s.namespace.name}
| sort requestCount desc
| limit 10
```

<a id="correlation-queries"></a>
## 5. Correlation Queries
### Joining Multiple Data Sources

```dql
// High CPU workloads with their names
timeseries avgCpuMillicores = avg(dt.kubernetes.container.cpu_usage), from:-1h, by:{dt.entity.cloud_application}
| fieldsAdd avgCpuMillicoresValue = arrayAvg(avgCpuMillicores)
| lookup [fetch dt.entity.cloud_application | fields id, entity.name], sourceField:dt.entity.cloud_application, lookupField:id
| fieldsRename workloadName = lookup.entity.name
| fields workloadName, avgCpuMillicoresValue
| sort avgCpuMillicoresValue desc
```

```dql
// Nodes with high utilization — derived, because no node-grain usage metric exists
// used  = container CPU summed across every container on the node
// alloc = the one genuinely node-grain family
timeseries {
    used = sum(dt.kubernetes.container.cpu_usage),
    allocatable = avg(dt.kubernetes.node.cpu_allocatable)
  }, from:-1h, by:{k8s.cluster.name, k8s.node.name}
| fieldsAdd cpuPercent = round(100 * arrayAvg(used) / arrayAvg(allocatable), decimals: 1)
| filter cpuPercent > 70
| fields k8s.cluster.name, k8s.node.name, cpuPercent
| sort cpuPercent desc
```

<a id="dashboard-queries"></a>
## 6. Dashboard Queries
### Queries Optimized for Dashboards

```dql
// Cluster summary tile (smartscape topology)
smartscapeNodes "K8S_CLUSTER"
| summarize clusterCount = count()
```

```dql
// Node count tile (smartscape topology)
smartscapeNodes "K8S_NODE"
| summarize nodeCount = count()
```

```dql
// Error rate trend (chart)
fetch logs, from: now() - 24h
| filter loglevel == "ERROR" and isNotNull(k8s.namespace.name)
| summarize errors = count(), by:{time_bucket = bin(timestamp, 1h)}
| sort time_bucket asc
```

```dql
// Top namespaces by CPU (bar chart) — sum() of every container in the namespace
timeseries cpuMillicores = sum(dt.kubernetes.container.cpu_usage), from:-1h, by:{k8s.namespace.name}
| fieldsAdd avgCpuMillicores = arrayAvg(cpuMillicores)
| sort avgCpuMillicores desc
| limit 10
```

<a id="alerting-queries"></a>
## 7. Alerting Queries
### Threshold-Based Alerting Patterns

```dql
// High memory usage alert query
timeseries avgMemBytes = avg(dt.kubernetes.container.memory_working_set), from:-1h, by:{dt.entity.container_group_instance}
| fieldsAdd avgMemBytesValue = arrayAvg(avgMemBytes)

```

```dql
// CrashLoop detection (last hour) — alerting condition
// The event reason is "BackOff"; "CrashLoopBackOff" appears in the message text,
// so filter on the reason and read the message for detail.
fetch events, from:-1h
| filter event.provider == "KUBERNETES_EVENT"
| filter dt.kubernetes.event.reason == "BackOff"
| summarize crashLoopEvents = count(),
    affectedPods = countDistinctExact(dt.kubernetes.event.involved_object.name)
```

```dql
// Error rate spike detection
fetch logs, from: now() - 1h
| filter loglevel == "ERROR" and isNotNull(k8s.namespace.name)
| summarize errors = count(), by:{k8s.namespace.name}
| filter errors > 100
| sort errors desc
```

<a id="query-templates"></a>
## 8. Query Templates
### Reusable Query Patterns

**Filter by Namespace:**
```dql
| filter k8s.namespace.name == "NAMESPACE_NAME"
```

**Filter by Cluster:**
```dql
| filter k8s.cluster.name == "CLUSTER_NAME"
```

**Time-based Analysis:**
```dql
fetch logs, from:-DURATION
| summarize log_count = count(), by:{time_bucket = bin(timestamp, INTERVAL)}
| sort time_bucket asc
```

**Top N Pattern:**
```dql
| sort METRIC desc
| limit N
```

---

## Summary

This notebook provided DQL query patterns for:

- Entity discovery and relationships
- Container and node metrics
- Log and event analysis
- Distributed tracing
- Multi-source correlation
- Dashboard visualizations
- Alerting conditions

---

## References

- [Dynatrace Query Language (DQL) reference (DT docs)](https://docs.dynatrace.com/docs/platform/grail/dynatrace-query-language)
- [smartscapeNodes command (DT docs)](https://docs.dynatrace.com/docs/platform/grail/dynatrace-query-language)
- [Set up Dynatrace on Kubernetes (DT docs)](https://docs.dynatrace.com/docs/ingest-from/setup-on-k8s)
- [Kubernetes app — workloads + namespaces views (DT docs)](https://docs.dynatrace.com/docs/observe/infrastructure-observability/kubernetes-app)
- [Davis Problems app (DT docs)](https://docs.dynatrace.com/docs/dynatrace-intelligence/problems-app)
- [Effective pod resources (DT docs)](https://docs.dynatrace.com/docs/observe/infrastructure-observability/kubernetes-app/reference/effective-pod-resources)
- [ActiveGate 1.345 release notes (DT docs)](https://docs.dynatrace.com/docs/whats-new/activegate/sprint-345)
- [ActiveGate 1.347 release notes (DT docs)](https://docs.dynatrace.com/docs/whats-new/activegate/sprint-347)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
