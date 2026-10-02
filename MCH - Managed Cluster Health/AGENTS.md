# AGENTS.md — MCH: Managed Cluster Health

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

8 reference notebooks on operating a self-hosted **Dynatrace Managed** cluster
day to day: node architecture, a five-layer health model (node, storage,
capacity, connectivity, lifecycle), where each health signal is surfaced, and
one notebook per layer. Managed has no Grail, so the series contains **no DQL**
and ships no importable notebook JSON — markdown and PDF only.

All eight notebooks are complete. For "what should I check / how often / which signals
don't email" start with MCH-99; for the detail behind any line, follow its pointer to
MCH-01–07.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| What runs on a Managed cluster node (NGINX, Server, Cassandra, Elasticsearch, embedded ActiveGate), where each data store lives, what a node failure costs (three copies vs. unreplicated, never-backed-up transaction storage), the five-layer health model, where health is surfaced (CMC, cluster event notifications, local / hosted / private self-monitoring and their licensing limits, Cluster API v1 `onpremise/cluster`, Mission Control), and which cluster events to route on day one | `-[MCH]-01-managed-cluster-architecture-health-model.md` |
| Whether every node and service is up: node inventory via CMC and Cluster API v1 (`operationState`, `configuration`, `configuration/status`, maintenance), `dynatrace.sh status` / `check` / `pid` and the seven systemd units, the 12 `operationState` values a Dynatrace PM posted (community), the CMC per-node Restart recorded as a conflict with the docs, the gated service start order (firewall → Elasticsearch → Cassandra → Server → rest), the 3+ node cluster start/stop procedure vs. `dynatrace.sh` for smaller clusters, one-node-at-a-time rules for patches / updates / removals / IP changes, disabling a node's OneAgent traffic, which node and process events are emailed vs. Mission-Control-only, heap and NTP events, inter-node ports, a node triage order, and diagnostic archives | `-[MCH]-02-server-nodes-and-processes.md` |
| Health of the Cassandra metrics store (Long-term Metrics Store): replication factor 3, what else it holds (configuration, support archives), per-node size table and the 2 TB (not emailed) / 4 TB (configurable) events, why metric retention (5 years; 1-min to 14 d, 5-min to 28 d, 1-h to 400 d, 1-day after) can't be reduced, reading `cassandra-nodetool.sh status` (`UN` / `UJ`, `Load`, `Owns`), storage location and moving it, adding nodes and `cleanup`, 24 h redistribution on removal, manual vs PHA automatic repair, what the daily backup excludes (1-/5-min data, ~80% of column families) and its sizing, rack awareness, Cassandra 4.1.x by Managed release | `-[MCH]-03-cassandra-metrics-store.md` |
| Health of the Elasticsearch store: what it holds (RUM Classic user sessions, 3 copies; Log Monitoring events, **2 copies** — two node failures make some logs unavailable), per-node sizing for 35 days, RUM taking up to 99% of the store, Log Monitoring's 64 GB RAM requirement, no NFS/EFS, retention as the size lever (user sessions 1–90 d per environment, logs up to 90 d; adaptive retention trims logs, not sessions), `_cluster/health` green/yellow, per-node `du`, the MC-only overload / transient-settings / backup / log-pressure events, moving the store, the 2-hourly incremental backup (never delete its files; logs only if opted in), PHA failover, Elasticsearch 8.19 from 1.340 | `-[MCH]-04-elasticsearch-store.md` |
| Whether agents can reach the cluster and the cluster can reach Mission Control: the three-level ActiveGate hierarchy (Environment → Cluster → embedded) and OneAgent's fallback order, keeping the cluster internal-only behind Cluster ActiveGates (public IP, own domain, trusted certificate, no web UI proxying), per-node NGINX and external load-balancer rules (443 unaltered, ≥120 s timeout, `/rest/health`, update after node changes), Public endpoints and DNS, node web-UI/agent roles, cluster and Cluster ActiveGate certificates (expiry events are **not emailed**), the Mission Control link (outbound HTTPS/WSS 443, TLS 1.2, proxy needs WebSockets + SNI, 1.318 cipher restriction, exchange frequencies), what a lost link costs (14-day overage limit on classic licensing, DPS usage queued, PHA needs online), offline clusters and conversion (1.338+), support remote access, Cluster ActiveGate status and events, the documented `dsfm:active_gate.*` self-monitoring metrics, 1.332/1.344 ActiveGate changes | `-[MCH]-05-activegates-mission-control-connectivity.md` |
| Whether the cluster can keep up: the per-node sizing table (host units, peak user actions/min, spec, IOPS), host units = RAM (16 GB = 1 for full-stack), Premium HA's roughly halved per-node figures, cloud instance families and the 1:8 vCPU:RAM ratio, scaling out (≤30 nodes, equal hardware) vs up vs storage (LVM, separate mounts), the one-third headroom rule, adaptive load reduction (15-minute accuracy warning) and the capacity events (traffic control and heap pressure **not emailed**), the capacity-indicator dashboard and `dsfm:server.*` throughput metrics, environment quotas, Cluster overload prevention (entry-point traces 1,000 → 100,000) and adaptive capture control, updating the ingest limit after every hardware change, the new-node disk minimum formula | `-[MCH]-06-capacity-and-scaling.md` |
| Backup, restore, upgrades, version support and disaster recovery: backup scope (configuration yes; metrics partly; user sessions unless excluded; log events only if included; transaction storage **never**), CMC Settings > Backup, NFS at the same path (NFSv4, not CIFS), the boot-time mount that stops Dynatrace starting, backup failures not emailed, restore preconditions (same installer version, same node count or ≤ 2 fewer, old cluster stopped, matching UID:GID) and order (Elasticsearch on the seed, Cassandra seed-first), automatic / semi-automatic / manual updates, the 24-hour rules, 5 GB free, < 3-node downtime, the divisible-by-4 skip rule and the 1.324 sequential option, the supported-versions table and pattern, Premium HA requirements and failover (15 min, 72 h automatic repair, rebuild afterwards), choosing a recovery path | `-[MCH]-07-backup-upgrade-disaster-recovery.md` |
| The whole series on one page: the five-layer health-check checklist (check, how, what healthy looks like), **every cluster event that never emails** in one table, a daily / weekly / monthly / after-every-change review cadence, the numbers worth knowing (3, 30, 2/4 TB, ⅓, 10/100 ms, 15 min, 24 h, 14 days, 72 h, 5 GB, 4 weeks), anti-patterns with their documented consequences, the series-wide gaps and inter-page conflicts, and a "Beyond the Docs" list of Dynatrace blog/Hub pages, the `dtmgd` / `dynatrace-managed-mcp` OSS tools and community KB articles | `-[MCH]-99-best-practice-summary.md` |

## Related series

- Moving a Managed cluster to SaaS: `../M2S - Managed to SaaS Migration/`
- ActiveGate sizing concepts (SaaS-focused): `../FAQ - Frequently Asked Questions/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- This series applies to **Dynatrace Managed only**. Do not apply it to SaaS,
  and do not offer DQL for Managed cluster health — Managed has no Grail.
- There is no `notebooks/` JSON for this series; do not offer a tenant import.
  Cite by notebook ID (e.g. "MCH-01").
