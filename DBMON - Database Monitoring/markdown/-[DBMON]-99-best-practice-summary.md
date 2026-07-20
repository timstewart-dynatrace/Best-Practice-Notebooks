# DBMON-99: Best Practice Summary

> **Series:** DBMON — Database Monitoring | **Notebook:** 7 of 7 | **Created:** March 2026 | **Last Updated:** 07/20/2026

## Overview

This notebook consolidates every actionable best practice for Dynatrace database monitoring extracted from the DBMON series (notebooks 01 through 06). It covers SQL databases, NoSQL databases, caches, message brokers, query analysis, dashboards, and alerting. Each best practice is definitive: it specifies exactly what to configure, what threshold to set, and what priority to assign.

---

## Table of Contents

1. [OneAgent Instrumentation](#oneagent-instrumentation)
2. [ActiveGate Extensions](#activegate-extensions)
3. [Span Attribute Coverage](#span-attribute-coverage)
4. [SQL Database Monitoring](#sql-database-monitoring)
5. [NoSQL Database Monitoring](#nosql-database-monitoring)
6. [Cache Monitoring (Redis / Memcached)](#cache-monitoring)
7. [Message Broker Monitoring (Kafka / RabbitMQ)](#message-broker-monitoring)
8. [Search Engine Monitoring (Elasticsearch)](#search-engine-monitoring)
9. [Query Analysis and Optimization](#query-analysis-and-optimization)
10. [Dashboards and KPIs](#dashboards-and-kpis)
11. [Alerting Thresholds](#alerting-thresholds)
12. [SLO Definitions](#slo-definitions)
13. [DQL Query Standards](#dql-query-standards)
14. [Where to Go Deeper — Topic Series Map](#see-also)

---

## Prerequisites

| Requirement | Details |
|-------------|--------|
| **Dynatrace Environment** | SaaS or Managed with Grail enabled |
| **OneAgent** | Deployed on all application hosts making database, cache, and messaging calls |
| **ActiveGate** | Host-based ActiveGate for Extensions 2.0 (remote DB metrics) |
| **Permissions** | `storage:spans:read`, `storage:entities:read`, `storage:metrics:read` |
| **Prior Reading** | DBMON-01 through DBMON-06 for full context |

<a id="oneagent-instrumentation"></a>

## 1. OneAgent Instrumentation

| # | Best Practice | Recommended Setting/Value | Priority | Category |
|---|--------------|----------------|----------|----------|
| 1 | Deploy OneAgent on every host that makes database calls | OneAgent installed on all application servers, not just database servers | **Critical** | Deployment |
| 2 | Enable deep code-level instrumentation | OneAgent auto-instruments JDBC, ADO.NET, and native database drivers by default; do not disable | **Critical** | Deployment |
| 3 | Verify `db.system` attribute is populated | Run `fetch spans, from:-1h \| filter isNotNull(db.system) \| summarize count(), by:{db.system}` — every expected technology must appear | **Critical** | Validation |
| 4 | Verify `db.statement` capture is enabled | Confirm normalized SQL/commands appear in `db.statement` field; if blank, check OneAgent deep monitoring settings | **Critical** | Validation |
| 5 | Confirm `db.namespace` is present | Run `filter isNotNull(db.namespace)` on spans; missing values indicate driver-level gaps | **Recommended** | Validation |
| 6 | Ensure `server.address` and `server.port` resolve | These fields must contain the actual database endpoint, not `localhost` or `127.0.0.1` when the DB is remote | **Recommended** | Validation |

<a id="activegate-extensions"></a>

## 2. ActiveGate Extensions

| # | Best Practice | Recommended Setting/Value | Priority | Category |
|---|--------------|----------------|----------|----------|
| 7 | Build new database integrations on Extensions 2.0 | Extensions 2.0 is the current extensions framework; Extensions Framework 1.0 reached end of support on 2025-03-31 (Python EF1.0: 2024-10-31); JMX and PMI EF1.0 are deprecated but supported past that date on request. Extensions 2.0 require **host-based ActiveGate** (K8s-based AG not supported) | **Critical** | Deployment |
| 7b | Extensions are YAML-defined with optional Python data sources | YAML package executed by EEC; built-in SQL/Prometheus/SNMP data sources cover most cases; Python data source is for custom logic only — not required by default | **Recommended** | Architecture |
| 7c | Route extension logs to the `default_database_monitoring` bucket | Default destination for extension log output; reference this bucket name in IAM policies for DB-team access scoping (see ORGNZ-02 + IAM-04/05) | **Recommended** | IAM/Bucket |
| 8 | Install PostgreSQL extension | Captures: connections, transactions/sec, tuple operations, table/index sizes, lock waits, replication lag | **Recommended** | Extension |
| 9 | Install MySQL extension | Captures: connections, queries/sec, buffer pool hit ratio, InnoDB row ops, slow queries | **Recommended** | Extension |
| 10 | Install MS SQL Server extension | Captures: batch requests/sec, page life expectancy, buffer cache hit ratio, deadlocks, wait stats | **Recommended** | Extension |
| 11 | Install Oracle extension | Captures: sessions, tablespace usage, SGA/PGA memory, redo log waits, library cache hit ratio | **Recommended** | Extension |
| 12 | Install MongoDB extension | Captures: connections, operations/sec, document metrics, replica set health, storage engine stats | **Recommended** | Extension |
| 13 | Set extension polling interval to 60 seconds | Default polling; do not exceed 300s or real-time visibility degrades | **Recommended** | Configuration |

<a id="span-attribute-coverage"></a>

## 3. Span Attribute Coverage

| # | Best Practice | Recommended Setting/Value | Priority | Category |
|---|--------------|----------------|----------|----------|
| 14 | Validate all six core DB span attributes | Every database span must carry: `db.system`, `db.statement`, `db.operation`, `db.namespace`, `server.address`, `server.port` | **Critical** | Data Quality |
| 15 | Check for NULL fields periodically | Run `filter isNotNull(db.operation)` — NULL means the driver is not reporting the operation type; investigate driver version | **Recommended** | Data Quality |
| 16 | Verify span.kind is CLIENT for outgoing DB calls | All database spans from the calling application must have `span.kind == "CLIENT"` | **Recommended** | Data Quality |
| 17 | For Kafka, verify `messaging.system` attribute | Kafka spans use `messaging.system` (not `db.system`); confirm `messaging.destination.name`, `messaging.operation`, and `messaging.kafka.consumer.group` are populated | **Critical** | Data Quality |
| 18 | For RabbitMQ, verify `messaging.system` attribute | RabbitMQ spans use `messaging.system` (not `db.system`); confirm `messaging.destination.name` is populated | **Critical** | Data Quality |

<a id="sql-database-monitoring"></a>

## 4. SQL Database Monitoring

| # | Best Practice | Recommended Setting/Value | Priority | Category |
|---|--------------|----------------|----------|----------|
| 19 | Set slow query threshold for SQL databases | **500ms** — any SQL query exceeding `duration > 500ms` (500ms in nanoseconds) is classified as slow | **Critical** | Threshold |
| 20 | Track read/write ratio | Classify operations: `SELECT` = READ, `INSERT/UPDATE/DELETE` = WRITE; monitor ratio shifts over time | **Recommended** | Analysis |
| 21 | Rank queries by total execution time, not average | Sort by `total_time_ms = sum(duration)` to find highest-impact patterns regardless of individual speed | **Critical** | Optimization |
| 22 | Monitor response time distribution using latency tiers | Bucket into: `<1ms`, `1-10ms`, `10-100ms`, `100ms-1s`, `>1s` — track tier distribution over time | **Recommended** | Analysis |
| 23 | Baseline P50, P95, P99 over 24 hours | Run hourly percentile query on `duration` for each `db.system`; this is your performance reference point | **Critical** | Baseline |
| 24 | Monitor vendor-specific patterns | PostgreSQL: vacuum/lock contention; MySQL: buffer pool/replication lag; MSSQL: page life expectancy/deadlocks; Oracle: tablespace/SGA | **Recommended** | Vendor |
| 25 | Track errors by `otel.status_message` | Group `otel.status_code == "ERROR"` by `otel.status_message` to distinguish connection timeouts, deadlocks, and constraint violations | **Critical** | Error Handling |


### Sprint-1.338 Update (May 2026)

| # | Best Practice | Recommended Setting/Value | Priority | Category |
|---|--------------|----------------|----------|----------|
| 25b | Use DQL `timeseries` for table/index metrics — classic Metrics API v2 path is blocked | The `postgres.tables.*`, `postgres.indexes.*`, `mysql.tables.*`, `mysql.indexes.*` metric keys are blocked from the classic Metrics API v2 path as of sprint-1.338 (May 2026). Query via DQL `timeseries` against Grail instead. | **Critical** | API/Migration |

<a id="nosql-database-monitoring"></a>

## 5. NoSQL Database Monitoring

| # | Best Practice | Recommended Setting/Value | Priority | Category |
|---|--------------|----------------|----------|----------|
| 26 | Set slow query threshold for NoSQL databases | **100ms** for MongoDB; **300ms** critical threshold for all NoSQL | **Critical** | Threshold |
| 27 | Monitor MongoDB by collection | Group by `db.mongodb.collection` in addition to `db.namespace` — collection-level granularity reveals hot spots | **Critical** | MongoDB |
| 28 | Track DynamoDB Scan-to-Query ratio | `Scan` operations read every item in the table; ratio must be Query-dominated; any high Scan count signals missing Global Secondary Indexes | **Critical** | DynamoDB |
| 29 | Set Cassandra slow threshold at 200ms | `filter duration > 200ms` — Cassandra operations exceeding 200ms indicate partition hotspots or cross-DC reads | **Recommended** | Cassandra |
| 30 | Monitor Cosmos DB by operation and error rate | Track `ReadItem`, `CreateItem`, `Query`, `ReplaceItem` separately; errors indicate RU throttling (HTTP 429) | **Recommended** | Cosmos DB |
| 31 | Classify NoSQL read/write operations correctly | READ: `find`, `Query`, `GetItem`, `SELECT`, `ReadItem`, `get`, `search`; everything else: WRITE | **Recommended** | Analysis |
| 32 | Run cross-database comparison | Compare `total_calls`, `avg_ms`, `p95_ms`, `error_rate_pct` across all NoSQL systems in a single query for unified health view | **Recommended** | Analysis |

<a id="cache-monitoring"></a>

## 6. Cache Monitoring (Redis / Memcached)

| # | Best Practice | Recommended Setting/Value | Priority | Category |
|---|--------------|----------------|----------|----------|
| 33 | Set Redis slow command threshold at 5ms | `filter duration > 5ms` — Redis operations should complete in microseconds; anything over 5ms is abnormal | **Critical** | Threshold |
| 34 | Monitor Redis read/write balance | READ: `GET`, `MGET`, `HGET`, `HGETALL`, `LRANGE`, `SMEMBERS`, `ZRANGE`; WRITE: `SET`, `MSET`, `HSET`, `LPUSH`, `RPUSH`, `SADD`, `ZADD`, `DEL` | **Recommended** | Analysis |
| 35 | Alert on Redis latency spikes | Track `p95_us` (microseconds) over time at 5m intervals; any sustained increase above 5000us (5ms) triggers investigation | **Critical** | Alerting |
| 36 | Watch for expensive Redis commands | `KEYS *`, `SMEMBERS` on large sets, `LRANGE` with large ranges cause latency spikes — flag any occurrence | **Critical** | Anti-Pattern |
| 37 | Report cache metrics in microseconds, not milliseconds | Redis and Memcached latency is sub-millisecond; use `avg(duration) / 1000.0` for microsecond precision | **Recommended** | Reporting |


### Sprint-1.337 Update — .NET Redis coverage

| # | Best Practice | Recommended Setting/Value | Priority | Category |
|---|--------------|----------------|----------|----------|
| 37b | Confirm StackExchange.Redis instrumentation on .NET workloads | OneAgent 1.337 added StackExchange.Redis coverage for .NET applications. Verify your .NET services produce `db.system == "redis"` spans. | **Recommended** | Coverage |

<a id="message-broker-monitoring"></a>

## 7. Message Broker Monitoring (Kafka / RabbitMQ)

| # | Best Practice | Recommended Setting/Value | Priority | Category |
|---|--------------|----------------|----------|----------|
| 38 | Track Kafka throughput by topic | Group by `messaging.destination.name` and `messaging.operation` (`publish` vs `process`) at 5m intervals | **Critical** | Kafka |
| 39 | Monitor Kafka consumer group processing latency | Group by `messaging.kafka.consumer.group` and `messaging.destination.name`; track `avg_ms` and `p95_ms` per group | **Critical** | Kafka |
| 40 | Detect Kafka consumer errors | Filter `otel.status_code == "ERROR"` on `messaging.operation == "process"` spans; any sustained errors indicate poisoned messages or deserialization failures | **Critical** | Kafka |
| 41 | Track RabbitMQ publish/consume rate per queue | Group by `messaging.destination.name` and `messaging.operation`; a publish rate exceeding consume rate signals backpressure | **Critical** | RabbitMQ |
| 42 | Monitor RabbitMQ message throughput trend | Use `makeTimeseries` at 5m intervals grouped by `messaging.operation` to detect throughput drops | **Recommended** | RabbitMQ |

<a id="search-engine-monitoring"></a>

## 8. Search Engine Monitoring (Elasticsearch)

| # | Best Practice | Recommended Setting/Value | Priority | Category |
|---|--------------|----------------|----------|----------|
| 43 | Set Elasticsearch slow query threshold at 200ms | `filter duration > 200ms` for search operations; indexing operations use a separate threshold | **Recommended** | Threshold |
| 44 | Break down Elasticsearch operations by type | Track `db.operation` (search, index, bulk, delete) separately; search latency and indexing throughput have different SLOs | **Recommended** | Analysis |
| 45 | Include OpenSearch in all Elasticsearch queries | Always filter `in(db.system, {"elasticsearch", "opensearch"})` — treat both identically | **Recommended** | Compatibility |

<a id="query-analysis-and-optimization"></a>

## 9. Query Analysis and Optimization

| # | Best Practice | Recommended Setting/Value | Priority | Category |
|---|--------------|----------------|----------|----------|
| 46 | Detect N+1 queries | Group spans by `trace.id` and `db.statement`; any pattern with `calls_per_trace >= 10` is an N+1 candidate; >= 50 is confirmed N+1 | **Critical** | Anti-Pattern |
| 47 | Fix N+1 by batching | Replace N individual queries with a single batch query using `IN` clauses or JOINs | **Critical** | Remediation |
| 48 | Detect missing indexes heuristically | SELECT queries with `call_count >= 50` AND `avg_ms > 10` AND high `stddev_ms / avg_ms` ratio are index candidates | **Recommended** | Optimization |
| 49 | Detect queries getting slower over time | Run 24h `makeTimeseries` of `avg(duration)` by `db.system` at 1h intervals; rising trend signals index degradation or table growth | **Recommended** | Trend |
| 50 | Flag dynamic SQL / poor parameterization | Count one-off query patterns (`call_count == 1`); high count per `db.system` means queries are not parameterized, preventing plan caching | **Recommended** | Anti-Pattern |
| 51 | Prioritize optimization by total time impact | Sort query patterns by `total_time_ms = sum(duration)`, not by `avg_ms` — a fast query called 1M times has more impact than a slow query called once | **Critical** | Optimization |
| 52 | Use the tail ratio to detect outliers | Calculate `p99_ms / p50_ms`; a ratio above 10 indicates severe tail latency requiring investigation | **Recommended** | Analysis |
| 53 | Monitor connection pool pressure | Track `countIf(otel.status_code == "ERROR")` grouped by `dt.entity.service` and `server.address`; connection refused or timeout errors indicate pool exhaustion | **Critical** | Connection Pool |
| 54 | Track error rate trend over 6h at 5m intervals | Use `makeTimeseries` with `total` and `errors` counts; a rising error trend within 30 minutes demands immediate action | **Critical** | Error Handling |

<a id="dashboards-and-kpis"></a>

## 10. Dashboards and KPIs

| # | Best Practice | Recommended Setting/Value | Priority | Category |
|---|--------------|----------------|----------|----------|
| 55 | Build a unified health overview tile | Single query: group by `db.system`, show `total_calls`, `avg_ms`, `p95_ms`, `error_rate_pct`, `slow_rate_pct` | **Critical** | Dashboard |
| 56 | Include a "total database calls" single-value tile | `fetch spans, from:-1h \| filter isNotNull(db.system) \| summarize total_db_calls = count()` | **Critical** | Dashboard |
| 57 | Add "top 10 heaviest services" table tile | Group by `dt.entity.service` with `call_count`, `avg_ms`, `error_count`; resolve names with `entityName()` | **Critical** | Dashboard |
| 58 | Show P50 vs P95 vs P99 trend chart | 6h timeseries at 5m intervals; three lines on one chart reveals tail latency divergence | **Critical** | Dashboard |
| 59 | Add read vs write ratio trend tile | Classify operations into READ/WRITE; 6h timeseries at 5m intervals | **Recommended** | Dashboard |
| 60 | Add queries-per-minute trend by db.system | 6h timeseries at 1m intervals; used for throughput anomaly detection | **Recommended** | Dashboard |
| 61 | Add today-vs-yesterday volume comparison | Use `append` pattern: `from:-24h` for today, `from:-48h, to:-24h` for yesterday | **Recommended** | Dashboard |
| 62 | Include slow query detail table | Last 15m, `duration > 500ms`, fields: timestamp, db.system, db.namespace, db.statement, duration_ms, service_name | **Critical** | Dashboard |

<a id="alerting-thresholds"></a>

## 11. Alerting Thresholds

### Error Rate Alerts

| # | Best Practice | Recommended Setting/Value | Priority | Category |
|---|--------------|----------------|----------|----------|
| 63 | Alert on error rate > 5% for 5 minutes | Severity: **Critical** — immediate page | **Critical** | Alert |
| 64 | Alert on error rate > 1% for 15 minutes | Severity: **Warning** — notification | **Critical** | Alert |
| 65 | Alert on any connection refused error | Severity: **Critical** — immediate page; zero tolerance for connection failures | **Critical** | Alert |
| 66 | Alert on any deadlock detection | Severity: **Warning** — notification | **Recommended** | Alert |

### Slow Query Alerts (P95 Thresholds)

| # | Best Practice | Recommended Setting/Value | Priority | Category |
|---|--------------|----------------|----------|----------|
| 67 | SQL Warning threshold | P95 > **200ms** over 5-minute window | **Critical** | Alert |
| 68 | SQL Critical threshold | P95 > **500ms** over 5-minute window | **Critical** | Alert |
| 69 | NoSQL Warning threshold | P95 > **100ms** over 5-minute window | **Critical** | Alert |
| 70 | NoSQL Critical threshold | P95 > **300ms** over 5-minute window | **Critical** | Alert |
| 71 | Cache (Redis/Memcached) Warning threshold | P95 > **5ms** over 5-minute window | **Critical** | Alert |
| 72 | Cache (Redis/Memcached) Critical threshold | P95 > **20ms** over 5-minute window | **Critical** | Alert |
| 73 | Elasticsearch Warning threshold | P95 > **500ms** over 5-minute window | **Recommended** | Alert |
| 74 | Elasticsearch Critical threshold | P95 > **2000ms** over 5-minute window | **Recommended** | Alert |

### Throughput Alerts

| # | Best Practice | Recommended Setting/Value | Priority | Category |
|---|--------------|----------------|----------|----------|
| 75 | Alert on throughput drop > 50% vs 7-day average | Compare current queries/min to 7-day baseline; sudden drop indicates upstream failure or connectivity loss | **Critical** | Alert |
| 76 | Alert on slow query count > 10 per 5-minute window | Sustained slow query volume indicates systemic degradation, not a one-off outlier | **Recommended** | Alert |

<a id="slo-definitions"></a>

## 12. SLO Definitions

| # | Best Practice | Recommended Setting/Value | Priority | Category |
|---|--------------|----------------|----------|----------|
| 77 | Define availability SLO | **99.9%** of database calls succeed (`otel.status_code != "ERROR"`) measured over 24h rolling window | **Critical** | SLO |
| 78 | Define latency SLO | **95%** of calls complete under the vendor-specific P95 threshold (SQL: 500ms, NoSQL: 300ms, Cache: 20ms, Search: 2000ms) | **Critical** | SLO |
| 79 | Define throughput SLO | No more than **50% drop** from 7-day baseline queries/min | **Recommended** | SLO |
| 80 | Measure SLO compliance hourly | Run `makeTimeseries` at 1h intervals with `total` and `errors`/`under_threshold` counts per `db.system` | **Critical** | SLO |
| 81 | Track SLO budget burn rate | If 24h availability drops below 99.9%, calculate remaining error budget: `(1 - 0.999) * total_calls - error_count` | **Recommended** | SLO |

<a id="dql-query-standards"></a>

## 13. DQL Query Standards

| # | Best Practice | Recommended Setting/Value | Priority | Category |
|---|--------------|----------------|----------|----------|
| 82 | Always specify a time range on fetch | `fetch spans, from:-1h` — never omit `from:` on span queries; default 2h scan is excessive | **Critical** | DQL |
| 83 | Filter on `isNotNull(db.system)` first | This is the primary filter to isolate database spans; apply immediately after `fetch` | **Critical** | DQL |
| 84 | Use named parameters in functions | `round(value, decimals: 2)`, `if(cond, then: "x", else: "y")` — positional parameters cause errors | **Critical** | DQL |
| 85 | Alias all aggregations | `summarize c = count()`, not `summarize count()` — unaliased aggregations cannot be used in `sort` or `fieldsAdd` | **Critical** | DQL |
| 86 | Use duration arithmetic (NOT nanosecond constants) | Write `duration / 1ms`, `duration / 1s`, `(t2-t1) / 1m` — divide a duration by another duration to get a unitless number. Never `duration / 1000000.0`; never `toLong(duration) / 1ms` (long/duration fails type-check). Per `dt-dql-essentials`. | **Critical** | DQL |
| 87 | Use `entityName()` (or `getNodeName()` on smartscape) to resolve service IDs | `fieldsAdd service_name = entityName(dt.entity.service, type:"dt.entity.service")` for the legacy form; `fieldsAdd service_name = getNodeName(dt.smartscape.service)` for the modern Smartscape form. Never show raw entity IDs in dashboards. | **Recommended** | DQL |
| 88 | Filter early, sort last | Apply `filter` immediately after `fetch`; apply `sort` only after `summarize`; never `sort` right after `fetch` | **Critical** | DQL |
| 89 | Use `countIf()` for conditional aggregation | `errors = countIf(otel.status_code == "ERROR")` inside `summarize` — do not use separate filter + count | **Recommended** | DQL |
| 90 | Calculate error rate with safe division | `error_rate_pct = round((toDouble(error_count) / toDouble(total_calls)) * 100, decimals: 2)` — use `toDouble()` to avoid integer truncation | **Recommended** | DQL |


### Modern DQL standards (added 2026-05-07)

| # | Best Practice | Recommended Setting/Value | Priority | Category |
|---|--------------|----------------|----------|----------|
| 91 | Use `event.status` / `event.start` / `event.end` on `dt.davis.problems` | NOT `status` / `end_time` (recurring authoring bug). Status values are `"ACTIVE"` and `"CLOSED"` (not `"OPEN"`). For duration: `(event.end - event.start) / 1m` (NOT `event.end - timestamp` — `timestamp` is record-write time, not problem-open time). | **Critical** | DQL |
| 92 | Prefer `smartscapeNodes "<TYPE>"` for new entity queries | Modern Smartscape topology query: `smartscapeNodes "SERVICE"`, `smartscapeNodes "HOST"`, etc. Legacy `fetch dt.entity.<type>` still works on hybrid tenants but is deprecated. | **Recommended** | DQL |
| 93 | Wrap multiple `summarize` aggregations in `{}` | When summarizing 2+ aggregations, wrap them: `summarize { a = count(), b = avg(x) }, by: {field}`. Tenant emits `PARAMETERS_SHOULD_BE_GROUPED` INFO if not wrapped (query still runs but is non-canonical). | **Recommended** | DQL |

<a id="see-also"></a>


DBMON covers database monitoring foundations. These topic series cover adjacent domains in depth:

| Domain | Topic Series | Notebooks |
|--------|--------------|-----------|
| **Distributed tracing & spans** (DB spans live here) | SPANS | 9 |
| **OpenPipeline log processing** | OPLOGS | 9 |
| **OpenPipeline beyond logs (metrics from spans)** | OPIPE | 7 |
| **Dynatrace Intelligence (Davis anomaly detection on DB metrics)** | AIOPS | 8 |
| **Workflows & alert notifications** (DB alert routing) | WFLOW | 10 |
| **Dashboard strategy & executive reporting** | DASH | 8 |
| **Bucket strategy** (`default_database_monitoring` and custom DB buckets) | ORGNZ | 11 |
| **IAM administration** (bucket-scoped policies for DB-team access) | IAM | 13 |
| **Tagging strategy** (host-group + ownership tagging for DBs) | FAQ-01, FAQ-02 | 2+ |
| **Configuration automation** (Monaco/Terraform for extension deployment) | AUTOM | 11 |
| **OpenTelemetry integration** (OTel-instrumented DBs) | OTEL | 9 |
| **Cloud DB integrations** (RDS, Aurora, Cosmos DB, Cloud SQL) | CLOUD | 9 |

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
