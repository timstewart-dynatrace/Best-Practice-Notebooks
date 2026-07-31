# ADOPT-02: Platform Health Assessment

> **Series:** ADOPT — Observability Adoption & Maturity | **Notebook:** 2 of 6 | **Created:** March 2026 | **Last Updated:** 07/30/2026

## Overview

A healthy observability platform is the foundation for everything else — alerting, troubleshooting, capacity planning, and automation all depend on complete and accurate data. This notebook provides a structured approach to assessing Dynatrace platform health: OneAgent deployment coverage, data ingestion rates, entity monitoring completeness, ActiveGate status, and license consumption. The result is a platform health scorecard you can review regularly.

---

## Table of Contents

1. [OneAgent Deployment Coverage](#oneagent-coverage)
2. [Host Monitoring Health](#host-monitoring)
3. [Service and Process Discovery](#service-process-discovery)
4. [Data Ingestion Rates](#data-ingestion)
5. [ActiveGate Health](#activegate-health)
6. [License and Consumption Tracking](#license-consumption)
7. [Building a Platform Health Scorecard](#health-scorecard)
8. [Summary and Next Steps](#summary)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Dynatrace Environment** | SaaS or Managed with Grail enabled |
| **Permissions** | `storage:entities:read`, `storage:metrics:read`, `storage:logs:read`, `storage:events:read` |
| **Data** | At least 24 hours of OneAgent data |
| **Audience** | Platform engineers, Dynatrace administrators |

<a id="oneagent-coverage"></a>

## 1. OneAgent Deployment Coverage

OneAgent coverage is the most fundamental health indicator. Gaps in deployment mean gaps in visibility. The goal is to understand how many hosts are monitored and whether any are running outdated agent versions.

### 1.1 Total Monitored Hosts

```dql
// Count all monitored hosts and break down by monitoring mode
fetch dt.entity.host
| summarize total_hosts = count()

// Smartscape equivalent (dt.entity.* is deprecated but still functional):
//   smartscapeNodes "HOST"
//   | summarize total_hosts = count()
// Caveat: Smartscape reflects CURRENT live topology and can report fewer entities than
// the classic entity store; for a pre-migration discovery inventory keep the classic query above.
```

### 1.2 Hosts by Monitoring Mode

Dynatrace supports multiple monitoring modes. Full-stack provides the deepest visibility, while infrastructure-only and cloud-only modes have limited capabilities.

```dql
// Break down hosts by monitoring mode
fetch dt.entity.host
| summarize host_count = count(), by:{monitoringMode}
| sort host_count desc

// No Smartscape equivalent: monitoringMode is a classic host property, not a Smartscape node
// field. Keep the classic query above for monitoring-mode breakdowns.
```

### 1.3 Agent Version Distribution

Running outdated OneAgent versions can lead to missed features, security vulnerabilities, and compatibility issues. This query identifies the distribution of agent versions across your environment.

```dql
// Identify OneAgent version distribution across hosts
fetch dt.entity.host
| summarize host_count = count(), by:{agentVersion}
| sort host_count desc
| limit 20

// No Smartscape equivalent: agentVersion / installerVersion (OneAgent version) is not a
// Smartscape node field. Keep the classic query above for agent-version distribution.
```

> **Tip:** If you see more than 3-4 distinct agent versions, consider enabling auto-update or scheduling a coordinated update. A single major version across the fleet reduces support complexity.

<a id="host-monitoring"></a>

## 2. Host Monitoring Health

Beyond simple counts, we need to verify that monitored hosts are actively reporting data. A host entity may exist but stop sending metrics if the agent crashes or the host is decommissioned.

### 2.1 Host CPU Utilization Overview

This query provides a quick health check — if hosts are reporting CPU metrics, the agent is functional.

```dql
// Top 10 hosts by average CPU usage over last 1 hour
timeseries avgCpu = avg(dt.host.cpu.usage), from:-1h, by:{dt.entity.host}
| fieldsAdd avgCpuValue = arrayAvg(avgCpu)
| sort avgCpuValue desc
| limit 10
```

### 2.2 Host Memory Utilization

```dql
// Top 10 hosts by average memory usage over last 1 hour
timeseries avgMem = avg(dt.host.memory.usage), from:-1h, by:{dt.entity.host}
| fieldsAdd avgMemValue = arrayAvg(avgMem)
| sort avgMemValue desc
| limit 10
```

<a id="service-process-discovery"></a>

## 3. Service and Process Discovery

Auto-discovered services and process groups indicate the depth of application-level monitoring. A healthy deployment should show services that match your known application inventory.

### 3.1 Service Count and Types

```dql
// Count services by service type
fetch dt.entity.service
| summarize service_count = count(), by:{serviceType}
| sort service_count desc

// Smartscape equivalent (dt.entity.* is deprecated but still functional):
//   smartscapeNodes "SERVICE"
//   | summarize service_count = count(), by:{dt.service.sdv1_type}
//   | sort service_count desc
// Caveat: Smartscape reflects CURRENT live topology and can report fewer entities than
// the classic entity store; for a pre-migration discovery inventory keep the classic query above.
// Field maps: serviceType -> dt.service.sdv1_type.
```

### 3.2 Process Group Inventory

```dql
// Count process groups by technology type
fetch dt.entity.process_group
| summarize pg_count = count(), by:{softwareTechnologies}
| sort pg_count desc
| limit 15

// Smartscape note (dt.entity.* is deprecated but still functional): Smartscape models
// individual processes, not process GROUPS — smartscapeNodes "PROCESS" is a different
// granularity (process instances), so its results are not comparable to a process-group
// query. Keep the classic dt.entity.process_group query above.
```

<a id="data-ingestion"></a>

## 4. Data Ingestion Rates

Understanding how much data flows into Dynatrace helps with capacity planning, cost management, and identifying gaps. A sudden drop in ingestion often signals an agent outage or configuration change.

### 4.1 Log Ingestion Volume Over Time

```dql
// Log ingestion trend over the last 24 hours by hour
fetch logs, from:-24h
| makeTimeseries log_count = count(), interval:1h
```

### 4.2 Log Ingestion by Source

```dql
// Top 10 log sources by volume in the last 24 hours
fetch logs, from:-24h
| summarize log_count = count(), by:{log.source}
| sort log_count desc
| limit 10
```

### 4.3 Span Ingestion Volume

```dql
// Span ingestion trend over the last 24 hours
fetch spans, from:-24h
| makeTimeseries span_count = count(), interval:1h
```

<a id="activegate-health"></a>

## 5. ActiveGate Health

ActiveGates serve as routing, monitoring extension, and API endpoints. Their health directly impacts data collection reliability.

### 5.1 ActiveGate Inventory

```dql
// List all ActiveGates with their properties
smartscapeNodes "ACTIVEGATE"
| fields name, version = dt.active_gate.version, group = dt.active_gate.group.name, zone = dt.network_zone.id, is_containerized, modules
| sort name asc

// Correction (verified 07/2026): this cell previously ran `fetch dt.entity.active_gate` and
// carried a note claiming Smartscape had no ActiveGate node. Both were wrong. There is NO classic
// ActiveGate entity type in any spelling (active_gate, environment_active_gate,
// environment_activegate), so the classic query returned zero rows in every tenant —
// indistinguishable from "no ActiveGates deployed". `smartscapeNodes "ACTIVEGATE"` (no
// underscore) is the working path, and it works on tenants today.
// Field maps: entity.name → name, softwareVersion → dt.active_gate.version,
// networkZone → dt.network_zone.id.
// The node also makes this cell's stated intent achievable: the previous version could only count,
// because the classic entity exposed no properties — and in fact returned nothing at all.
// ActiveGate 1.343 (published 07/15/2026, staged tenant rollout from 07/28/2026) deprecates
// GET /api/v2/activeGates, /api/v2/activeGates/{agId} and /api/v2/activeGates/groups in favour of
// this same Smartscape node — the classic entity and the classic REST endpoints were retired as one
// move. If you need a REST surface in the meantime, the classic Entities API v2 selector
// (GET /api/v2/entities?entitySelector=type("ENVIRONMENT_ACTIVE_GATE")) is a different surface and
// may still respond; the DQL `fetch dt.entity.*` form does not.
```

### 5.2 ActiveGate Metric Health

Self-monitoring metrics confirm ActiveGates are processing data. A flat or zero metric suggests the ActiveGate is unhealthy.

```dql
// Connected agent modules per ActiveGate over the last 1 hour.
// A healthy routing ActiveGate holds a steady, non-zero count; a flat zero means agents are not
// reaching it, and a sudden drop means they stopped.
//
// Grouped by dt.active_gate.id — the ActiveGate identity field, and the same field the ACTIVEGATE
// Smartscape node carries, so `smartscapeNodes "ACTIVEGATE"` resolves these hex ids (0xd54e5d57, …)
// to names. Live-verified 07/30/2026: 4 series returned, ids matching the node exactly.
//
// Two corrections here, both verified 07/30/2026:
//  1. The metric key was `dt.sfm.active_gate.connections`, which DOES NOT EXIST. No such key is in
//     the catalogue, and `timeseries` against a non-existent key returns an EMPTY RESULT rather
//     than an error — so the cell read as "no ActiveGate data" instead of "wrong metric name".
//  2. The grouping was by:{dt.entity.environment_active_gate}, an entity type that does not exist
//     in any spelling.
//
// Discover the real keys yourself rather than trusting a remembered name:
//   metrics | filter startsWith(metric.key, "dt.sfm.active_gate") | fields metric.key
// Note `metrics` takes `from:` with NO leading comma — `metrics from:now()-2h`. Writing
// `metrics, from:…` is a PARSE_ERROR, which is easy to misread as "no such metrics exist".
timeseries connected = avg(dt.sfm.active_gate.communication.agent_modules.connected), from:-1h, by:{dt.active_gate.id}
```

<a id="license-consumption"></a>

## 6. License and Consumption Tracking

Tracking license consumption helps prevent unexpected overages and ensures you are getting value from your investment.

### 6.1 Host Unit Estimate

Host units are a primary consumption metric. Each host consumes units based on its memory size and monitoring mode.

```dql
// Estimate host unit consumption by monitoring mode
fetch dt.entity.host
| summarize host_count = count(), by:{monitoringMode}
| fieldsAdd estimated_hu = if(monitoringMode == "FULL_STACK", then: host_count, else: host_count * 0)

// No Smartscape equivalent: monitoringMode is a classic host property, not a Smartscape node
// field. Keep the classic query above for monitoring-mode breakdowns.
```

### 6.2 Log Ingestion Volume for Cost Awareness

```dql
// Daily log record count over the last 7 days
fetch logs, from:-7d
| makeTimeseries daily_logs = count(), interval:1d
```

<a id="health-scorecard"></a>

## 7. Building a Platform Health Scorecard

Combine the results from the queries above into a regular health scorecard. Review this scorecard weekly or monthly to track platform health trends.

### Recommended Scorecard Metrics

| Metric | Target | How to Measure |
|--------|--------|----------------|
| **Host Coverage** | > 95% of known hosts monitored | Compare entity count vs CMDB |
| **Agent Version Currency** | All agents within 1 major version | Version distribution query |
| **Service Discovery** | All known services auto-detected | Compare service count vs app inventory |
| **Log Ingestion Stability** | < 10% daily variance | 7-day ingestion trend |
| **Span Ingestion Active** | > 0 spans per hour | Span count query |
| **ActiveGate Health** | All AGs reporting metrics | AG connection metrics |
| **Dynatrace Intelligence Active** | Problems detected in last 7 days | Problem count query |
| **Alert Noise Ratio** | < 20% duplicate/frequent | Noise ratio query from ADOPT-01 |

### Scoring Guide

| Score | Status | Action |
|-------|--------|--------|
| **8/8 metrics green** | Healthy | Continue regular monitoring |
| **6-7 green** | Minor gaps | Address within current sprint |
| **4-5 green** | Significant gaps | Prioritize remediation |
| **< 4 green** | Critical | Escalate to platform team |

<a id="summary"></a>

## 8. Summary and Next Steps

### Key Takeaways

- Platform health assessment should be a regular practice, not a one-time exercise
- OneAgent coverage and data ingestion stability are the two most critical health indicators
- A simple scorecard with 8-10 metrics provides actionable visibility into platform health
- Gaps discovered here directly inform your maturity roadmap from ADOPT-01

### Next Steps

- Proceed to **ADOPT-03: Success Metrics** to define MTTR, MTTD, and other outcome-based metrics
- Schedule a recurring health scorecard review (weekly recommended for new deployments)
- Compare discovered entities against your CMDB to identify coverage gaps

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
