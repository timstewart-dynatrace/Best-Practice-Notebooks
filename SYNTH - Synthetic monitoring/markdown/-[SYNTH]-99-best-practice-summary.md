# SYNTH-99: Best Practice Summary

> **Series:** SYNTH | **Notebook:** 99 | **Created:** March 2026 | **Last Updated:** 03/26/2026

## Overview

This notebook consolidates every actionable best practice from the SYNTH series (notebooks 01-06) into a single definitive reference. Each practice specifies the exact setting or value to use, its priority level, and category. Use this as a checklist when deploying or auditing Dynatrace synthetic monitoring.

---

## Table of Contents

1. [Monitor Selection](#monitor-selection)
2. [HTTP Monitor Configuration](#http-monitor-configuration)
3. [Browser Monitor Configuration](#browser-monitor-configuration)
4. [Network Monitor Configuration](#network-monitor-configuration)
5. [Scheduling and Frequency](#scheduling-and-frequency)
6. [Location Strategy](#location-strategy)
7. [Private Locations and ActiveGate](#private-locations-and-activegate)
8. [Validation and Assertions](#validation-and-assertions)
9. [Authentication and Security](#authentication-and-security)
10. [Performance Thresholds](#performance-thresholds)
11. [Alerting Configuration](#alerting-configuration)
12. [SLOs and Error Budgets](#slos-and-error-budgets)
13. [Dashboard and Reporting](#dashboard-and-reporting)
14. [Operational Hygiene](#operational-hygiene)

---

<a id="monitor-selection"></a>
## 1. Monitor Selection

Choose the right monitor type for each use case.

| Practice | Setting | Priority |
|----------|---------|----------|
| Use **HTTP monitors** for API health checks and REST endpoints | Monitor type: HTTP single request | Critical |
| Use **HTTP multi-step** for auth-token-then-API workflows | Monitor type: HTTP multi-step with variable extraction | Critical |
| Use **Single-URL browser monitors** for homepage/landing page availability | Monitor type: Browser single-URL | Critical |
| Use **Browser clickpath monitors** for multi-page user journeys (login, checkout) | Monitor type: Browser clickpath | Critical |
| Use **ICMP monitors** for host reachability and latency baseline | Monitor type: Network availability - ICMP | Recommended |
| Use **DNS monitors** for name resolution validation | Monitor type: Network availability - DNS | Recommended |
| Use **TCP port monitors** for service port connectivity (databases, caches) | Monitor type: Network availability - TCP | Recommended |
| Layer monitors: DNS + ICMP + TCP + HTTP for full-stack coverage of critical services | Deploy one monitor per OSI layer for each critical path | Recommended |
| Prefer HTTP monitors over browser monitors when JavaScript rendering is not needed | Saves cost and runs faster (< 1s vs 3-30s) | Recommended |

<a id="http-monitor-configuration"></a>
## 2. HTTP Monitor Configuration

| Practice | Setting | Priority |
|----------|---------|----------|
| Set explicit request timeout | Timeout: **30 seconds** (default) | Critical |
| Set `Content-Type` and `Accept` headers on every request | `Content-Type: application/json`, `Accept: application/json` | Critical |
| Add User-Agent header for identification | `User-Agent: Dynatrace Synthetic` | Recommended |
| Enable SSL certificate monitoring on every HTTPS endpoint | SSL check: enabled (automatic on HTTP monitors) | Critical |
| Use JSON path assertions for API response validation | Assert `$.status == "success"` or equivalent | Critical |
| Extract variables between steps using JSON path | Variable extraction source: `$.data.token` | Critical |
| Reference extracted variables in subsequent steps with `${variableName}` syntax | URL: `https://api.example.com/users/${userId}` | Critical |
| Validate HTTP status codes fail on 4xx/5xx | Default behavior: fail on 400-599 | Critical |
| Add content-present assertion for positive match | Validation: contains `"status": "ok"` | Recommended |
| Add content-absent assertion to catch error states | Validation: does not contain `"error"` | Recommended |

<a id="browser-monitor-configuration"></a>
## 3. Browser Monitor Configuration

| Practice | Setting | Priority |
|----------|---------|----------|
| Set viewport to match target audience device | Desktop: **1920x1080**, Laptop: **1366x768**, Tablet: **768x1024**, Mobile: **375x667** | Critical |
| Use CSS selectors as the primary element selector type | Selector: `#login-btn` or `[data-testid='login']` | Critical |
| Use `data-testid` attributes for automation-stable selectors | Selector: `[data-testid='submit']` | Recommended |
| Avoid XPath selectors unless DOM structure requires it | Use CSS first; XPath only for complex structures | Recommended |
| Add content validation on every clickpath step | Validate expected text/element exists after each action | Critical |
| Record clickpaths with the Dynatrace Synthetic Recorder Chrome extension | Recorder captures reliable events and selectors | Recommended |
| Enable automatic screenshots on success and failure | Screenshots: enabled (default) | Critical |
| Keep clickpath steps minimal (only critical path) | Fewer steps = faster execution and fewer false positives | Recommended |
| Do NOT rely on session persistence between executions | Cookies are maintained within a single execution only | Critical |

<a id="network-monitor-configuration"></a>
## 4. Network Monitor Configuration

| Practice | Setting | Priority |
|----------|---------|----------|
| Set ICMP packet count to 3-10 per execution | Packet count: **5** (balanced accuracy vs speed) | Recommended |
| Set ICMP timeout to 5 seconds per packet | Timeout: **5 seconds** | Recommended |
| Configure DNS monitors with expected IP validation | Expected IP: set to known-good resolved address | Critical |
| Monitor DNS record type A for primary resolution | Record type: **A** | Critical |
| Set DNS query timeout to 10 seconds | Timeout: **10 seconds** | Recommended |
| Monitor TCP ports for every critical service: 443 (HTTPS), 5432 (PostgreSQL), 6379 (Redis), 3306 (MySQL), 27017 (MongoDB) | TCP monitor per port per service | Critical |
| Set TCP connection timeout to 10 seconds | Timeout: **10 seconds** | Recommended |
| Deploy multi-protocol monitors for critical infrastructure: DNS + ICMP + TCP in sequence | One monitor per protocol per target | Recommended |

<a id="scheduling-and-frequency"></a>
## 5. Scheduling and Frequency

| Practice | Setting | Priority |
|----------|---------|----------|
| Run HTTP monitors at **1-5 minute** intervals for critical APIs | Frequency: **1 min** (critical), **5 min** (standard) | Critical |
| Run browser monitors at **5-15 minute** intervals | Frequency: **5 min** (critical), **15 min** (standard) | Critical |
| Run network monitors at **1-5 minute** intervals | Frequency: **1 min** (infrastructure), **5 min** (standard) | Recommended |
| Use **5-minute** frequency as the default for all non-critical monitors | Frequency: **5 minutes** | Recommended |
| Never run browser monitors more frequently than every 5 minutes | Minimum interval: **5 min** (browser) | Critical |
| Match frequency to SLA detection requirements: 1 min frequency detects outages within 2-3 min | Frequency = desired detection time / consecutive-failure count | Recommended |

<a id="location-strategy"></a>
## 6. Location Strategy

| Practice | Setting | Priority |
|----------|---------|----------|
| Assign a minimum of **3 public locations** per external-facing monitor | Locations: 3+ geographically distributed | Critical |
| Select locations that match your user base geography | Choose regions where real users access the app | Critical |
| Use public locations for external-facing applications only | Location type: Public (Dynatrace-hosted) | Critical |
| Use private locations for internal applications, VPN-only apps, and pre-production | Location type: Private (ActiveGate) | Critical |
| Require failures from **2+ locations** before raising an outage alert | Outage detection: multi-location confirmation | Critical |
| Include at least one location per major region (Americas, EMEA, APAC) for global apps | 1+ location per continent where users exist | Recommended |

<a id="private-locations-and-activegate"></a>
## 7. Private Locations and ActiveGate

| Practice | Setting | Priority |
|----------|---------|----------|
| Deploy ActiveGate with `--enable-synthetic` capability | Capability: **synthetic** | Critical |
| Provision ActiveGate with minimum **4 CPU cores, 8 GB RAM, 50 GB disk** | Resources: 4 cores / 8 GB / 50 GB (recommended) | Critical |
| Deploy **2+ ActiveGates** per private location for high availability | Nodes per location: **2+** | Critical |
| Distribute ActiveGates across availability zones | AZ distribution: one AG per AZ minimum | Recommended |
| Install Chrome/Chromium and a display server on ActiveGates that run browser monitors | Required: Chrome + X11 or headless display | Critical |
| Monitor ActiveGate CPU < 80%, memory < 80%, disk < 80% | Alert thresholds: **80%** sustained for CPU/memory/disk | Critical |
| Monitor ActiveGate execution queue depth | Alert threshold: queue > **100** pending executions | Recommended |
| Use Kubernetes DynaKube CRD with `capabilities: [synthetic-monitoring]` for K8s deployments | DynaKube spec: `activeGate.capabilities: ["synthetic-monitoring"]` | Recommended |
| Ensure outbound-only connectivity (no inbound firewall rules required) | Network: HTTPS outbound to Dynatrace cluster only | Critical |

<a id="validation-and-assertions"></a>
## 8. Validation and Assertions

| Practice | Setting | Priority |
|----------|---------|----------|
| Add at least one content validation rule to every monitor | Validation: text-present or JSON-path assertion | Critical |
| Validate positive content ("Welcome", "status: ok") rather than only checking for absence of errors | Validation type: **Text Present** | Critical |
| Use regex validation for dynamic content (e.g., order numbers) | Regex: `Order #\d{6}` | Recommended |
| Validate JSON responses with JSON path assertions | Assert: `$.status == "success"`, `$.data.users.length > 0` | Critical |
| Verify element presence in browser monitors for SPA pages | Selector: `#success-message` must exist | Recommended |
| Enable HTTP status code validation (fail on 4xx/5xx) on all monitors | Default: fail on status 400-599 | Critical |
| Add content-absent checks for known error strings | Validation: does not contain `"error"`, `"exception"` | Recommended |

<a id="authentication-and-security"></a>
## 9. Authentication and Security

| Practice | Setting | Priority |
|----------|---------|----------|
| Store all credentials in the Dynatrace Credential Vault | Path: Settings > Integration > Credential vault | Critical |
| Reference credentials with `${credentials.vault.myCredential}` syntax | Never hardcode tokens or passwords in monitor config | Critical |
| Use OAuth2 client_credentials flow for API authentication | Step 1: POST to token endpoint, Step 2: Bearer token in header | Recommended |
| Use Bearer token auth (not Basic auth) for modern APIs | Header: `Authorization: Bearer ${token}` | Recommended |
| Use client certificates (mTLS) for high-security API endpoints | Credential type: certificate in vault | Optional |
| Set SSL certificate expiration warning at **30 days** | Alert: warning at 30 days before expiry | Critical |
| Set SSL certificate expiration critical alert at **14 days** | Alert: critical at 14 days before expiry | Critical |
| Set SSL certificate expiration emergency alert at **7 days** | Alert: emergency at 7 days before expiry | Critical |
| Validate SSL certificate chain, hostname match, and trusted CA on every HTTPS monitor | SSL checks: validity + chain + hostname + trust (all enabled) | Critical |

<a id="performance-thresholds"></a>
## 10. Performance Thresholds

Set definitive thresholds for synthetic response time and web vitals.

| Practice | Setting | Priority |
|----------|---------|----------|
| HTTP monitor response time target | Good: **< 2s**, Warning: **2-5s**, Critical: **> 5s** | Critical |
| DNS resolution time target | Good: **< 50ms**, Warning: **50-200ms**, Critical: **> 200ms** | Recommended |
| TCP connect time target | Good: **< 100ms**, Warning: **100-300ms**, Critical: **> 300ms** | Recommended |
| Time to First Byte (TTFB) target | Good: **< 500ms**, Warning: **500ms-1s**, Critical: **> 1s** | Critical |
| First Contentful Paint (FCP) target | Good: **< 1.8s**, Critical: **> 3.0s** | Critical |
| Largest Contentful Paint (LCP) target | Good: **< 2.5s**, Critical: **> 4.0s** | Critical |
| Time to Interactive (TTI) target | Good: **< 3.8s**, Critical: **> 7.3s** | Recommended |
| Total Blocking Time (TBT) target | Good: **< 200ms**, Critical: **> 600ms** | Recommended |
| Cumulative Layout Shift (CLS) target | Good: **< 0.1**, Critical: **> 0.25** | Recommended |
| Speed Index target | Good: **< 3.4s**, Critical: **> 5.8s** | Optional |
| Flag performance anomalies when max response time > **2x** the average | Deviation factor threshold: **2.0** | Recommended |

<a id="alerting-configuration"></a>
## 11. Alerting Configuration

| Practice | Setting | Priority |
|----------|---------|----------|
| Require **2-3 consecutive failures** before triggering an availability alert | Consecutive failures: **3** (recommended) | Critical |
| Require failures from **2+ locations** to confirm a global outage | Location threshold: **2** | Critical |
| Set alert delay to **5-10 minutes** to absorb transient failures | Alert delay: **5 min** (HTTP), **10 min** (browser) | Critical |
| Auto-resolve alerts after **2 consecutive successes** | Auto-close: 2 successes | Critical |
| Configure availability alerts for outage detection | Alert type: availability, trigger on consecutive failures | Critical |
| Configure performance alerts for degradation detection | Alert type: P95 response time > threshold | Recommended |
| Configure SLO-based alerts for error budget consumption | Alert type: error budget burn rate | Recommended |
| Configure SSL certificate expiration alerts at 30/14/7 day thresholds | Three alert tiers: warning/critical/emergency | Critical |
| Use multi-location failure detection to classify local vs global outages | Severity: WARNING at 2 locations, CRITICAL at 3+ locations | Recommended |

<a id="slos-and-error-budgets"></a>
## 12. SLOs and Error Budgets

| Practice | Setting | Priority |
|----------|---------|----------|
| Define an availability SLO for every critical synthetic monitor | SLO target: **99.9%** (8.76 hours downtime/year) | Critical |
| Define a performance SLO for P95 response time | SLO: **95%** of executions < **3000ms** | Recommended |
| Calculate error budget as `total_executions * (1 - SLO_target)` | Budget: 0.1% of executions for 99.9% SLO | Critical |
| Track error budget burn rate daily | Daily query: failures / daily budget allocation | Recommended |
| Use a **30-day rolling window** for SLO evaluation | SLO timeframe: **30 days** | Critical |
| Alert when error budget is **50% consumed** in less than half the window | Burn rate alert: 2x normal consumption rate | Recommended |
| Map SLA tiers to availability targets: 99.9% = 8.76h/yr, 99.5% = 43.8h/yr, 99.0% = 87.6h/yr | Set SLO target to match contractual SLA | Critical |

<a id="dashboard-and-reporting"></a>
## 13. Dashboard and Reporting

| Practice | Setting | Priority |
|----------|---------|----------|
| Build a synthetic dashboard with these 6 tiles: Overall Availability (single value), Availability Trend (line chart), P95 Response Time by Monitor (bar chart), Location Map, Failures Table, SLO Status (gauge) | Dashboard layout: KPI row + trend row + detail row + table row | Recommended |
| Show overall availability as a single-value tile | Tile: `countIf(available) * 100 / count()` over 24h | Recommended |
| Show hourly availability trend as a line chart over 24 hours | Tile: availability_pct by 1h buckets | Recommended |
| Show P95 response time by monitor as a bar chart, sorted descending | Tile: `percentile(response_time, 95)` by monitor, top 10 | Recommended |
| Show recent failures table with timestamp, monitor, location, error | Tile: last 20 failures, sorted by timestamp desc | Recommended |
| Include a week-over-week performance comparison | Query: this week avg/P95 vs last week avg/P95 | Optional |
| Generate a daily availability report for the last 30 days | Query: daily availability_pct, sorted by day | Recommended |

<a id="operational-hygiene"></a>
## 14. Operational Hygiene

| Practice | Setting | Priority |
|----------|---------|----------|
| Name monitors descriptively: `[Type] Target - Purpose` | Example: `[HTTP] api.example.com/health - API Health` | Recommended |
| Name private locations with a consistent convention that distinguishes them from public locations | Example: `PRIVATE-DC1-US-EAST` | Recommended |
| Pair every synthetic monitor with a corresponding RUM configuration for the same application | Synthetic = baseline/SLA; RUM = actual user experience | Recommended |
| Review and disable stale monitors that no longer have valid targets | Audit: quarterly review of all enabled monitors | Recommended |
| Use the timing breakdown (DNS, connect, SSL, TTFB, download) to isolate root cause of slowness | Query: avg per phase by monitor | Critical |
| Check ActiveGate logs at `/var/log/dynatrace/gateway/synthetic.log` when private location tests fail | Log: `synthetic.log` for execution errors | Critical |
| Test network connectivity from ActiveGate to targets with `curl -v` and `nslookup` before configuring monitors | Pre-check: verify DNS resolution and HTTPS connectivity | Critical |
| Use `bizevents` with `event.provider == "dynatrace.synthetic"` as the primary data source for DQL analysis | Data source: bizevents, not raw metrics | Recommended |

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
