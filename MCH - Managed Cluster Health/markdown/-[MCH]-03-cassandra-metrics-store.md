# MCH-03: Cassandra Metrics Store Health

> **Series:** MCH — Managed Cluster Health | **Notebook:** 3 of 8 | **Created:** September 2026 | **Last Updated:** 10/02/2026

## Overview

Every Managed node runs Apache Cassandra, and together those nodes form the cluster's **metrics repository**, which the docs also call the Long-term Metrics Store. It keeps five years of timeseries data in three replicas. It is the store most likely to outgrow its disks, and a restore brings back less of it than most teams assume.

This notebook is layer 2 of the MCH-01 health model, for Cassandra. It covers what the store holds, how big it may get, how long metrics are kept and at what resolution, how to check Cassandra's health, what adding, removing and repairing nodes does to it, and what its daily backup does and does not protect.

**What you'll learn:**

- The per-node size ceilings, the events that report them, and the one documented way to stay under them
- Why metric retention can't be turned down to save space
- How to read `cassandra-nodetool.sh status` — `UN`, `UJ`, `Load` and `Owns`
- What adding or removing a node does to the data on disk, and when to run `cleanup`
- When Cassandra needs a repair, and when one runs automatically
- Exactly which metrics a Cassandra backup restores

---

## Table of Contents

1. [Short Answer](#short-answer)
2. [What Cassandra Holds and How It Is Replicated](#what-it-holds)
3. [Size Ceilings](#size-ceilings)
4. [Retention and Resolution](#retention)
5. [Health Checks](#health-checks)
6. [Disk and Storage Location](#disk)
7. [Adding and Removing Nodes](#add-remove)
8. [Repair](#repair)
9. [Backup — the Cassandra Part](#backup)
10. [Rack Awareness and Cassandra Versions](#racks-versions)
11. [What the Documentation Does Not Say](#doc-gaps)
12. [Recommended Approach](#recommendation)
13. [Summary and Next Steps](#summary)

---

## Prerequisites

| Requirement | Details |
|-------------|---------|
| **Dynatrace deployment** | A self-hosted **Dynatrace Managed** cluster. Nothing in this series applies to Dynatrace SaaS. |
| **Access** | Cluster Management Console (CMC) administrator access; root (or `sudo`) on the cluster nodes |
| **Query language** | None. Managed has no Grail, so this series has **no DQL cells**. Every check is a CMC view, a cluster event, or a node command |
| **Read first** | MCH-01 (health model) and MCH-02 (node processes and the service start order) |
| **Version basis** | Written against the Managed documentation read 09/28/2026. Cassandra version notes are labeled with the Managed release that introduced them |

<a id="short-answer"></a>
## 1. Short Answer

- **Cassandra is the metrics repository.** It keeps three replicas, holds five years of metrics, and runs on every node.
- **Keep it under 2 TB per node, and never past 4 TB.** Only the 4 TB event can email you. The 2 TB warning goes to Mission Control and the CMC Events list only.
- **You can't shrink retention to save space.** Metric retention is five years, with no documented setting, and adaptive data retention never touches metrics. The documented remedies are *another node* or *fewer metrics*.
- **Healthy means every node shows `UN` and `Load` is roughly even.** `cassandra-nodetool.sh status` is the check.
- **Adding a node doesn't shrink the others.** Run `cleanup` on the existing nodes, and only after the last new node has joined.
- **A daily backup is not a full copy.** It excludes 1-minute and 5-minute data and the most frequently changing column families (~80% of storage). Plan restores around that.

> <sub>**Sources:** [Data retention periods (DT docs)](https://docs.dynatrace.com/managed/manage/data-privacy-and-security/data-privacy/data-retention-periods), [Hardware requirements (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/installation/managed-hardware-requirements), [Configure Cluster event notifications (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/configuration/configure-cluster-event-notifications), [Backup and restore a cluster (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/back-up-and-restore-a-cluster).</sub>

<a id="what-it-holds"></a>
## 2. What Cassandra Holds and How It Is Replicated

*"Data is stored in Metrics repository at DATASTORE_PATH/cassandra. Data is replicated across Managed Cluster nodes. Replication factor is set to three."* The hardware guide says the same thing in sizing terms: *"In multi-node installations, Dynatrace Managed stores three copies of the metrics store."*

Besides metrics, the docs name two more things kept in Cassandra:

- **Configuration.** The backup docs file configuration under the same store, in a section titled *Metrics and configuration storage*, and restore it with the Cassandra restore script (§9).
- **OneAgent and ActiveGate support archives.** *"Dynatrace OneAgent or Dynatrace ActiveGate creates support archives and keeps them in Cassandra, where Dynatrace automatically deletes them after 30 days."*

Some pages call it *"Cassandra-based Hypercube storage"*. That is the same store under another name, and it uses ports `7000`, `7001` and `9042` between nodes, plus `7199` for JMX (MCH-02 §8).

**What three replicas buy.** With a replication factor of three, each piece of metrics data lives on three nodes. That is why one node can fail without data loss, and why losing two can make some data unavailable (MCH-01 §4). It is also the reason the docs give for spacing node removals: *"it takes up to 24 hours for the long-term metrics replicas to be automatically redistributed on the remaining nodes"* (§7.2).

> <sub>**Sources:** [Data retention periods (DT docs)](https://docs.dynatrace.com/managed/manage/data-privacy-and-security/data-privacy/data-retention-periods), [Hardware requirements (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/installation/managed-hardware-requirements), [Backup and restore a cluster (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/back-up-and-restore-a-cluster), [Install a Managed Cluster (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/installation/install-managed-cluster), [Cluster node ports (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/installation/cluster-node-ports), [Remove a cluster node (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/remove-a-cluster-node).</sub>

<a id="size-ceilings"></a>
## 3. Size Ceilings

*"Keep the Long-term Metrics Store below 2 TB per node. Dynatrace supports stability and operational resilience up to 4 TB per node."* Growing past them has a cost beyond disk space: *"Persistent storage growth degrades Managed Cluster performance, resilience, and availability, and causes issues when adding nodes."*

The planned Long-term Metrics Store per node, by node size:

| Node size | Metrics store per node |
|-----------|------------------------|
| Micro | 100 GB |
| Small | 500 GB |
| Medium | 1 TB |
| Large | 2 TB |
| XLarge | 4 TB |

An XLarge node is sized at the supported ceiling from day one, so it has no headroom above it.

**The two events, and who hears them:**

| Event | Emailed |
|-------|---------|
| *"Long-term Metrics Store size exceeds recommended 2 TB on %s."* | **No** — Mission Control and CMC Events only |
| *"Long-term Metrics Store size exceeds the maximum acceptable 4 TB on %s."* | Configurable (on by default) |

By the time an email arrives, the store is already past the supported limit. Watch for the 2 TB event in CMC Events, or measure the store yourself (§6).

**What to do about it.** The docs give two remedies. The hardware guide: *"If you need more storage, add another node to reduce the per-node requirement."* The event description: *"you need to review your monitoring settings or add additional nodes to the Managed Cluster."* Adding a node works because the three replicas are spread over more nodes. With more than three nodes, each owns `(3 / number_of_nodes) × 100%` of the data (§5). Add nodes before the store crosses the ceiling, not after: growth *"causes issues when adding nodes"*.

**Compression (Managed 1.314+).** Managed 1.314 changed how Cassandra compresses metrics: *"the used compression format of timeseries data in Cassandra will be switched from LZ4 to ZSTD"*. The saving builds up slowly: *"reduction in disk usage will not be instant, but rather will happen gradually over the course of one month"*. Any cluster on 1.314 or later already has it, so it is not a lever left to pull.

> <sub>**Sources:** [Hardware requirements (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/installation/managed-hardware-requirements), [Configure Cluster event notifications (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/configuration/configure-cluster-event-notifications), [Add a cluster node (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/installation/add-cluster-node), [Managed 1.314 release notes (DT docs)](https://docs.dynatrace.com/managed/whats-new/managed/sprint-314). **Derived:** "add nodes before the ceiling" combines the growth warning with the add-node remedy.</sub>

<a id="retention"></a>
## 4. Retention and Resolution

Managed keeps metrics for **5 years**. How finely you can look at them depends on their age. *"The following interval granularity levels are available for dashboarding and API access:"*

| Data age | Finest resolution |
|----------|-------------------|
| 0–14 days | 1 minute |
| 14–28 days | 5 minutes |
| 28–400 days | 1 hour |
| 400 days–5 years | 1 day |

![Metric Resolution by Age — and What a Backup Restores](images/03-metric-resolution-and-backup.png)

<!-- MARKDOWN_TABLE_ALTERNATIVE
| Data age | Resolution | In the daily Cassandra backup? |
|----------|------------|--------------------------------|
| 0–14 days | 1 minute | No — 1-minute data excluded |
| 14–28 days | 5 minutes | No — 5-minute data excluded |
| 28–400 days | 1 hour | Partly — most frequently changing column families (~80% of storage) excluded |
| 400 days–5 years | 1 day | Partly — same exclusion |
Retention is 5 years with no documented setting; adaptive data retention does not cover metrics.
For environments where SVG doesn't render
-->

**Retention is not a size lever.** Other rows on the retention page say *Configurable*; the metrics row doesn't, and no metric-retention setting is documented. Adaptive data retention, which shortens retention automatically when disk runs short, applies only to *"transaction storage, Session Replay storage, and Log Monitoring data"*. For metrics, disk pressure has to be solved with nodes or with fewer metrics (§3), not with a shorter look-back.

> <sub>**Sources:** [Data retention periods (DT docs)](https://docs.dynatrace.com/managed/manage/data-privacy-and-security/data-privacy/data-retention-periods), [Adaptive data retention (DT docs)](https://docs.dynatrace.com/managed/manage/data-privacy-and-security/data-privacy/adaptive-data-retention), [Backup and restore a cluster (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/back-up-and-restore-a-cluster). **Observed 09/28/2026:** the retention page's Metrics row reads *5 years* where other rows read *Configurable*, and no metric-retention setting appears in the 140 pages under `/managed/managed-cluster/` and `/managed/dynatrace-api/cluster-api/`.</sub>

<a id="health-checks"></a>
## 5. Health Checks

### 5.1 `cassandra-nodetool.sh status`

The single check the docs rely on, from any node:

```bash
sudo /opt/dynatrace-managed/utils/cassandra-nodetool.sh status
```

Its output starts with a legend and one line per node, as in the docs' example:

```text
Status=Up/Down |/ State=Normal/Leaving/Joining/Moving
--  Address   Load        Tokens  Owns (effective)  Host ID   Rack
UN  1.6.1.6   349.88 GiB  256     100.0%            aaaa...   rack1
UJ  1.6.3.9   278.75 GiB  256     ?                 qqqq...   rack1
```

How to read it:

| Column | Healthy looks like |
|--------|--------------------|
| First two letters | `UN`: Up and Normal. *"Make sure that all nodes display UN before proceeding to the next step."* `UJ` means joining: *"Where UJ marks the node as joining."* |
| `Load` | Roughly even across nodes. *"The Load value should not differ significantly between the nodes and Status should display UN on all nodes."* |
| `Owns (effective)` | `100.0%` on clusters of up to three nodes. Above that, *"the percentage is calculated as follows: (3/number_of_nodes)*100%"*. *"While a node is joining, the Owns (effective) column shows a question mark (?)."* |

Anything other than `UN` on a node that isn't deliberately joining or leaving is a problem. So is one node carrying far more `Load` than the rest.

### 5.2 Cassandra events

None of these email you. They appear in CMC Events and Mission Control only:

| Event | Severity |
|-------|----------|
| *"Cassandra node connection lost ( %d times in the last hour)."* | WARNING |
| *"Cassandra backup problem."* | SEVERE |
| *"Internal: Cassandra has old files."* | INFO |

The Cassandra *process* is covered by MCH-02: `dynatrace-cassandra.service` in `dynatrace.sh status`, port `9042` in `dynatrace.sh check`, and the generic *"<component name> is down."* event. Mission Control also receives Cassandra details in the cluster heartbeat, including *"Cassandra version, Cassandra data files, Cassandra partitions"*.

> <sub>**Sources:** [Add a cluster node (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/installation/add-cluster-node), [Start/stop/restart a cluster (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/start-stop-restart-cluster), [Migrate a cluster (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/migrate-a-cluster), [Configure Cluster event notifications (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/configuration/configure-cluster-event-notifications), [Mission Control data exchange (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/basics/mission-control-data-exchange).</sub>

<a id="disk"></a>
## 6. Disk and Storage Location

**Where it lives.** By default, `/var/opt/dynatrace-managed/cassandra`, set at install time with `--cas-datastore-dir` (*"Full path to the Dynatrace metrics repository directory"*). The storage-location table asks for **25 GB** free on that path to install and **1 GB** to upgrade. That is only the minimum to install; plan capacity from the size table in §3.

**How big it is now.** The docs use plain `du` to measure it:

```bash
du -sh /var/opt/dynatrace-managed/cassandra/
```

Run it on each node and compare against the 2 TB line. That catches growth before the non-emailed 2 TB event does.

**Leave room for compaction.** A Dynatrace community KB article explains why the disk needs headroom beyond the store itself: the metrics store can undergo compaction phases, during which it can grow to twice its size, and when free space runs short transaction storage is deleted first to make room ([reserve disk space consumed (Dynatrace community)](https://community.dynatrace.com/t5/Troubleshooting/Why-is-my-reserve-disk-space-in-managed-node-is-getting-consumed/ta-p/204801), 2023). In community practice, that is the reason to keep well clear of a full disk even when the store is under its ceiling — verify the margin against your own compaction peaks.

**Layout rules:**

- Give Cassandra its own volume. The rack-awareness guide says so directly: *"Keep Cassandra data on a separate volume to avoid disk space issues caused by other data types."*
- Don't nest stores: *"you can't nest datastores within each other. For example, Cassandra storage can't be a subdirectory of session storage."*
- No network file systems for live data: *"High-latency remote volumes such as NFS or CIFS aren't supported for primary storage; NFS is sufficient for backups only."*

**Moving it.** The documented procedure, per node, one node at a time (MCH-02 §5):

1. Stop the node: `sudo /opt/dynatrace-managed/launcher/dynatrace.sh stop`
2. Copy the data, keeping permissions: `cp -pR /old_location/cassandra/* /new_location/cassandra`
3. Fix ownership: `chown -R dynatrace:dynatrace /new_location`
4. Point `CASSANDRA_DATASTORE_PATH` at the new location in `/etc/dynatrace.conf`
5. Apply it: `nohup <PRODUCT_PATH>/installer/reconfigure.sh --no-start &`
6. Start the node: `sudo /opt/dynatrace-managed/launcher/dynatrace.sh start`

On a 3+ node cluster, step 1 stops a node with `dynatrace.sh`, which MCH-02 §5 otherwise reserves for smaller clusters. The storage-move page documents it this way anyway. Confirm every other node is `UN` before you start.

> <sub>**Sources:** [Change storage location (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/change-storage-location), [Customize Managed Cluster installation (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/installation/customize-managed-cluster-install), [Estimating cluster backup size (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/estimating-cluster-backup-size), [Rack awareness — replication (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/high-availability/rack-aware-replication), [Hardware requirements (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/installation/managed-hardware-requirements).</sub>

<a id="add-remove"></a>
## 7. Adding and Removing Nodes

### 7.1 Adding a node

A new node streams its share of the data from the others. *"Full data synchronization can take a couple of hours."* During that time it shows as `UJ`, and *"You can't add additional cluster nodes until synchronization completes."*

The existing nodes don't shrink by themselves. *"Cassandra doesn't automatically balance the data on disk in the entire cluster after a node is added."* And *"Relatedly, data on disk (Load) isn't automatically reduced on all cluster nodes."* To reclaim that space, run `cleanup` on each existing node. It's a node-local operation, and it's slow: *"This command can run for several hours"*.

```bash
sudo /opt/dynatrace-managed/utils/cassandra-nodetool.sh cleanup
```

Time it right. The docs say *"run the cleanup only after you add all desired nodes"*. For a scale-out from three to six nodes, that means running it once the sixth node has joined, on the five nodes that were there before it.

**Why this matters for health.** If the store is near its ceiling, adding a node relieves nothing until `cleanup` has run. Budget the hours for it as part of the scale-out.

### 7.2 Removing a node

*"Remove no more than one node at a time. To avoid data loss, allow 24 hours before removing any subsequent nodes. This is because it takes up to 24 hours for the long-term metrics replicas to be automatically redistributed on the remaining nodes."* The survivors grow to absorb the removed node's share, so check that they have the disk for it before you start.

> <sub>**Sources:** [Add a cluster node (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/installation/add-cluster-node), [Remove a cluster node (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/remove-a-cluster-node). **Derived:** "survivors grow" and "nothing is relieved until cleanup" follow from the redistribution and no-automatic-rebalance statements.</sub>

<a id="repair"></a>
## 8. Repair

A repair makes the replicas consistent with each other again, for example after a node has been down or data has been restored.

**Manual repair.** The backup-and-restore procedure has it as an optional step: *"You can run the repair only on clusters with more than one node."* Run it on one node after another, never in parallel:

```bash
sudo /opt/dynatrace-managed/utils/repair-cassandra-data.sh
```

*"This is for ensuring data consistency between the nodes. This step may take several hours to complete."* Since Managed 1.318, a manual repair is gentler on the cluster: *"When a Cassandra repair operation is executed manually for some reason, we now run it table by table in order to avoid causing too much overhead on the overall cluster."*

**Automatic repair — multi-data-center clusters only.** On Premium High Availability clusters, *"If Cassandra was down for 3 hours or more, Nodekeepers also run Cassandra repairs, one by one, on all nodes in the unhealthy DC."* It gets one attempt: *"The repair process runs only once, even if it fails. You should manually run the repair process on nodes where it automatically triggered repair failure."* The logs to check are `nodekeeper.0.0.log`, `nodekeeper-healthcheck.0.log` and `repair-cassandra-data.log`.

**Single data center.** No automatic repair is documented for a single data center (observed 09/28/2026). In community practice, a manual repair is planned once a node that has been down for an extended period is back and `UN` — confirm with Dynatrace support first.

> <sub>**Sources:** [Backup and restore a cluster (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/back-up-and-restore-a-cluster), [Multi-data center failover (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/high-availability/failover), [Managed 1.318 release notes (DT docs)](https://docs.dynatrace.com/managed/whats-new/managed/sprint-318).</sub>

<a id="backup"></a>
## 9. Backup — the Cassandra Part

MCH-07 covers backup and disaster recovery as a whole. This section covers what matters specifically about the metrics store.

**What's in it.** *"The snapshot is performed daily."* It is not a full copy: *"Dynatrace excludes the most frequently changing column families (excluded column families comprise about 80% of total storage) in addition to 1-minute and 5-minute resolution data."* It also stores every replica: *"Any data that's replicated between nodes is also stored in the backup (there is no deduplication)."* So the 1-minute and 5-minute history (up to 28 days, §4) is not restored. Longer-term metrics and configuration are, from up to a day ago: *"potentially setting your data back by up to 24 hours."*

**How much space.** Size the metrics backup from the store itself: *"Typically, it's 20% of the sum of the metrics storage on all nodes."* The previous backup is kept until the next one finishes (*"Dynatrace keeps the previous backup until a new one is completed."*), so *"we recommend that you double the estimated space you need for metrics storage backup."* A configuration-only backup (*"Exclude timeseries metric data from the backup if your historical data isn't relevant and you only want to retain configuration data."*) *"Takes up to 10% of metrics storage instead of 20%"*.

**Where it goes.** Backups need a shared file system, and *"the shared file system needs to be mounted at the same shared directory on each node."* A missing mount is a node-health problem: *"If the shared file system mount point isn't available on system boot, Dynatrace won't start on that node."*

**When it fails.** *"Cassandra backup problem."* is SEVERE, but **not emailed**. Check CMC **Home**, which shows the last backup's date, and CMC Events.

**Restoring.** Cassandra is restored node by node, starting with the seed node, from each node's archive:

```bash
sudo /opt/dynatrace-managed/utils/restore-cassandra-data.sh <path-to-backup>/<UUID>/node_<node-id>/files/<backup-version-number>/backup-001.tar
```

Then wait for every node to show *Status = Up / State = Normal* in `cassandra-nodetool.sh status`, and optionally run the repair (§8).

> <sub>**Sources:**</sub>
> - <sub>[Backup and restore a cluster (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/back-up-and-restore-a-cluster), [Estimating cluster backup size (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/estimating-cluster-backup-size)</sub>
> - <sub>[Migrate a cluster (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/migrate-a-cluster), [Configure Cluster event notifications (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/configuration/configure-cluster-event-notifications)</sub>
> - <sub>**Derived:** "the 1-/5-minute history is not restored" applies the backup exclusion to the resolution tiers in §4</sub>

<a id="racks-versions"></a>
## 10. Rack Awareness and Cassandra Versions

### 10.1 Rack awareness

*"Rack awareness ensures that no replica is stored redundantly inside a single rack, so replicas are spread across all racks."* It exists for Cassandra's sake, and its requirements follow from the replication factor: *"The final number of racks is three, corresponding to the replication factor of Dynatrace data storage."* *"Each rack holds at least three nodes. Dynatrace doesn't enforce this"*. Racks must also be in the same low-latency network.

Converting an existing cluster is a large Cassandra operation:

- *"Rack-aware conversion using restore suits a metric storage (Cassandra database) per node of more than 1 TB."*
- The replication method needs room: *"The disk size should be at least double the combined Cassandra storage of all existing nodes."* It also takes time: *"Cassandra bootstrapping can take several days."*
- *"Run the reconfigure.sh script sequentially, because it triggers process restarts and there's a risk of Cassandra database downtime if it's run on multiple nodes in parallel."*

### 10.2 Which Cassandra you are running

Cassandra is upgraded as part of normal Managed updates, so your Cassandra version follows your Managed version:

| Managed release | Cassandra |
|-----------------|-----------|
| 1.338 | 4.1.11 |
| 1.344 and 1.346 | 4.1.12 — *"The Cassandra nodes are upgraded to version 4.1.12, delivering critical bug and security fixes."* |

In both cases *"No manual user intervention or downtime is required"*: the upgrade happens through rolling updates. Keeping Managed within its supported releases (MCH-07) is how you keep Cassandra patched.

> <sub>**Sources:** [Rack awareness (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/high-availability/rack-awareness), [Rack awareness — replication (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/high-availability/rack-aware-replication), [Managed 1.338 (DT docs)](https://docs.dynatrace.com/managed/whats-new/managed/sprint-338), [Managed 1.344 (DT docs)](https://docs.dynatrace.com/managed/whats-new/managed/sprint-344) release notes.</sub>

<a id="doc-gaps"></a>
## 11. What the Documentation Does Not Say

Searched for in the Managed documentation on 09/28/2026 and **not found**:

| Gap | Working assumption in this notebook |
|-----|-------------------------------------|
| What to do about status codes other than `UN` and `UJ` — the printed `nodetool` legend (*Status=Up/Down*, *State=Normal/Leaving/Joining/Moving*) names them, but no page explains them | Treat anything that isn't `UN` as unhealthy unless it's a planned join or removal |
| A `dsfm:` self-monitoring metric for Cassandra or the metrics store | Measure with `du` and `nodetool` on the node (§5, §6) |
| A setting for metric retention | Retention is not a size lever (§4) |
| A procedure for recovering Cassandra on a single failed node | Replace the node, then run a manual repair (§8) |
| An explicit statement in the docs that configuration lives in Cassandra | Inferred from the backup docs' *Metrics and configuration storage* section (§2); a Dynatrace product manager's 2019 forum answer says the same ([what are the different type of data (Dynatrace community)](https://community.dynatrace.com/t5/Alerting/what-are-the-different-type-of-data/td-p/122675)) |

**A naming oddity.** The restore procedure's configuration-only option runs `repair-cassandra-data.sh 1`, not `restore-cassandra-data.sh`. It is quoted exactly as the docs print it. Confirm with Dynatrace support before relying on it.

> <sub>**Sources:** [Backup and restore a cluster (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/back-up-and-restore-a-cluster), [Add a cluster node (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/installation/add-cluster-node). **Observed 09/28/2026:** none of the five gaps is filled in the 140 pages under `/managed/managed-cluster/` and `/managed/dynatrace-api/cluster-api/`, the 23 Managed release-note pages linked from the release-notes index and the data-privacy pages (crawled and searched for `nodetool` status codes, `dsfm:` keys, metric-retention settings and Cassandra recovery procedures). The rest of `/managed/` was not searched.</sub>

<a id="recommendation"></a>
## 12. Recommended Approach

1. **Measure the store weekly on every node** with `du -sh /var/opt/dynatrace-managed/cassandra/`. Trend it against the 2 TB line, since the 2 TB event never emails.
2. **Check `cassandra-nodetool.sh status` as part of every node check.** Every node `UN`, and `Load` roughly even.
3. **Scale out before 2 TB, not after 4 TB.** Budget the synchronization time, the one-node-at-a-time rule, and the `cleanup` hours after the last node joins.
4. **Keep Cassandra on its own local volume**, never NFS, with room for growth.
5. **Know what a restore gives back.** It returns configuration and longer-term metrics from up to a day ago, and none of the last 28 days' 1-minute and 5-minute data. Size the backup target at twice 20% of the total store.
6. **Plan a manual repair after any extended node outage** on a single-DC cluster. Nothing runs one for you.

<a id="summary"></a>
## 13. Summary and Next Steps

Cassandra is the cluster's metrics repository: three replicas, five years of data, one instance per node. It stays healthy when every node is `UN` with even load, when it stays under 2 TB per node, and when it gets the nodes it needs before it gets there. Retention can't be turned down to save space, a new node relieves nothing until `cleanup` runs, and the daily backup restores long-term metrics and configuration, not recent high-resolution data.

**Next in the series:**

| Notebook | Layer |
|----------|-------|
| MCH-04: Elasticsearch Store Health | 2 — Storage |
| MCH-06: Capacity and Scaling | 3 — Capacity (node sizing and scale-out) |
| MCH-07: Backup, Upgrade, and Disaster Recovery | 5 — Lifecycle (full backup and restore) |

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official [Dynatrace documentation](https://docs.dynatrace.com/managed).*</sub>
