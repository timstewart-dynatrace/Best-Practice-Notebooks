# DTM - Dynatrace Managed

> **Dynatrace Classic only.** This series applies to **Dynatrace Managed**, the self-hosted, on-premises deployment. None of it applies to Dynatrace SaaS.

Best practices specific to a self-hosted Dynatrace Managed cluster: designing and configuring it, and keeping it healthy — its nodes, data stores, capacity, connectivity to Mission Control, backups and upgrades.

> **Formerly MCH — Managed Cluster Health.** DTM-10 to DTM-16 and DTM-99 were published as MCH-01 to MCH-07 and MCH-99 until October 2026. DTM-00 maps every old notebook to its new number.

> **Note:** This series has no Dynatrace notebook JSON. Dynatrace Managed has no Grail, so there are no DQL queries to run — every check is a Cluster Management Console view, a cluster event, or a Cluster API call. Read it as Markdown or PDF.

## Structure
- pdfs/ — Printable versions of each notebook
- markdown/ — Markdown exports of the notebooks

## Notebook Lineup
0. [The Dynatrace Managed Journey](markdown/-[DTM]-00-dynatrace-managed-journey.md) — Series map: what the series applies to, where to start by situation, the numbering bands, and where each former MCH notebook went
1. [Cluster Design Best Practices](markdown/-[DTM]-01-cluster-design-best-practices.md) — Sizing for growth, node rules and placement, choosing single-cluster HA, rack awareness or Premium HA, storage layout, and host, operating system and cloud choices
2. [Cluster Configuration Best Practices](markdown/-[DTM]-02-cluster-configuration-best-practices.md) — Custom properties that survive updates, reliable encrypted email, the Mission Control and remote-access trade-offs, internet proxy, sign-in and session security, privilege elevation, node traffic roles, and backups from day one
3. [Environment Best Practices](markdown/-[DTM]-03-environment-best-practices.md) — Environment or management zone as the boundary, per-environment limits, choosing LDAP, SAML or OpenID Connect, the break-glass administrator account, keeping cluster, environment and zone permissions narrow, and additive IAM policies
4. [ActiveGate Deployment on Managed](markdown/-[DTM]-04-activegate-deployment-on-managed.md) — ActiveGate roles and purposes, placement so OneAgents never fall back uncompressed, two per data center and sizing, network zones and load balancers, multi-environment limits and dedicated extension ActiveGates, and update modes
5. [OneAgent Deployment on Managed](markdown/-[DTM]-05-oneagent-deployment-on-managed.md) — How OneAgent reaches the cluster, host groups and network zones decided before rollout, network-zone fallback modes, update policy with rings and restarts, and the x86-64-v2 CPU baseline
1. [Managed Cluster Architecture and Health Model](markdown/-[DTM]-10-managed-cluster-architecture-health-model.md) — What runs on a cluster node, what a node failure costs, the five-layer health model, and where each health signal is surfaced
2. [Server Nodes and Processes](markdown/-[DTM]-11-server-nodes-and-processes.md) — Node inventory and operation state, on-node process checks, the gated service start order, safe restarts, which node events email you, memory, time, inter-node network, and triage
3. [Cassandra Metrics Store Health](markdown/-[DTM]-12-cassandra-metrics-store.md) — The metrics repository: replication, size ceilings and their events, 5-year retention by resolution, nodetool health checks, adding and removing nodes, repair, what the daily backup restores, rack awareness, and Cassandra versions
4. [Elasticsearch Store Health](markdown/-[DTM]-13-elasticsearch-store.md) — User sessions (3 copies) and log events (2 copies), sizing, retention as the size lever, _cluster/health and events, log ingest pressure, storage location, the incremental backup, failover and versions
5. [Cluster ActiveGates and Mission Control Connectivity](markdown/-[DTM]-14-activegates-mission-control-connectivity.md) — The agent traffic path and ActiveGate hierarchy, load balancers, endpoints and DNS, certificates, the Mission Control link and what breaks without it, offline clusters, remote access, and Cluster ActiveGate health
6. [Capacity and Scaling](markdown/-[DTM]-15-capacity-and-scaling.md) — The sizing model (host units, peak user actions, Premium HA), scaling out, up and storage, one-third headroom, capacity signals and which email you, environment quotas and overload prevention, and adding or removing nodes
7. [Backup, Upgrade, and Disaster Recovery](markdown/-[DTM]-16-backup-upgrade-disaster-recovery.md) — What a backup protects and what it doesn't, setting up backups (and the boot-time mount risk), restore preconditions and order, upgrade paths and their 24-hour rules, version support, Premium HA failover, and choosing a recovery path
99. [Best Practice Summary and Health-Check Checklist](markdown/-[DTM]-99-best-practice-summary.md) — The series on one page: a five-layer checklist, every signal that never emails you, a review cadence, the numbers worth knowing, anti-patterns, and the documentation's gaps and conflicts

## Usage
1. Choose a format: read markdown/ for browsing or pdfs/ for print.
2. Start with DTM-00 for the series map, DTM-10 for the health model, then use the notebook for whichever layer is failing.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
