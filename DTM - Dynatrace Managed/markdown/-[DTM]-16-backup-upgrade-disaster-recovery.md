# DTM-16: Backup, Upgrade, and Disaster Recovery

> **Series:** DTM — Dynatrace Managed | **Reference:** 16 — Backup, Upgrade, and Disaster Recovery | **Created:** September 2026 | **Last Updated:** 11/11/2026

> **Dynatrace Classic only.** This series applies to **Dynatrace Managed**, the self-hosted, on-premises deployment. None of it applies to Dynatrace SaaS.

## Overview

The last layer of the DTM-10 health model is **lifecycle**: whether the cluster can be recovered when something is lost, and whether it stays current. Both are easy to neglect because they don't fail loudly. A backup that stopped working weeks ago and a cluster that has drifted out of support look exactly like healthy ones until the day they're needed.

This notebook covers what a backup protects and what it doesn't, how to set backups up and keep them working, what a restore needs before it can succeed, how upgrades run and what slows them down, how long each version is supported, how Premium High Availability recovers from losing a data center, and how to pick the right recovery path for what was actually lost.

**What you'll learn:**

- Exactly which data a restore brings back, and which it can't
- The backup-storage mistake that can stop a node from starting
- The preconditions a restore needs, prepared before you need them
- The 24-hour rules, the skip rule and the maintenance window behind every upgrade
- How long a version is supported, and how failover works across two data centers

---

## Table of Contents

1. [Short Answer](#short-answer)
2. [What a Backup Protects — and What It Doesn't](#backup-scope)
3. [Setting Up Backups](#backup-setup)
4. [Restore Readiness](#restore)
5. [Upgrades](#upgrades)
6. [Version Support](#version-support)
7. [Premium High Availability and Data-Center Failover](#pha)
8. [Choosing a Recovery Path](#recovery-paths)
9. [What the Documentation Does Not Say](#doc-gaps)
10. [Recommended Approach](#recommendation)
11. [Summary and Next Steps](#summary)

---

## Prerequisites

| Requirement | Details |
|-------------|---------|
| **Dynatrace deployment** | A self-hosted **Dynatrace Managed** cluster. Nothing in this series applies to Dynatrace SaaS. |
| **Access** | Cluster Management Console (CMC) administrator access; root (or `sudo`) on the cluster nodes; for Premium HA recovery, a Cluster API token with the `ServiceProviderAPI` scope |
| **Shared storage** | A shared file system (for example NFS) mounted at the same path on every node, for backups |
| **Query language** | None. Managed has no Grail, so this series has **no DQL cells** |
| **Read first** | DTM-10 (health model), DTM-12 §9 and DTM-13 §8 (what each store's backup contains) |
| **Version basis** | Written against the Managed documentation read 10/01/2026. Support dates come from the Managed release-notes page as it stood that day |

<a id="short-answer"></a>
## 1. Short Answer

- **A backup is not a full copy.** It restores configuration, longer-term metrics and (unless excluded) user sessions. It never restores transaction storage, never restores recent 1-minute and 5-minute metrics, and restores log events only if you opted in.
- **A missing backup mount can stop a node from starting.** *"If the shared file system mount point is not available on system boot, Dynatrace services will not start on that node"*. Make the mount persistent.
- **Prepare the restore before you need it.** It needs the exact installer version from the backup, the same number of nodes (*"In exceptional cases"* up to two fewer), matching user IDs, and the old cluster stopped. Afterwards, backups are **off** until you turn them back on.
- **Upgrades have built-in waits:** 24 hours after a download, and 24 hours between updates. Clusters under three nodes are unavailable while they update.
- **Three releases are supported**, plus a fourth with Enterprise Success and Support. A new release arrives roughly every four weeks, so falling behind happens quickly.
- **Premium HA repairs a lost data center automatically for up to 72 hours — through Mission Control.** *"There is no manual or local failover option that bypasses MC."* After 72 hours, the data center has to be rebuilt.

> <sub>**Sources:** [Backup and restore a cluster (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/back-up-and-restore-a-cluster), [Update a cluster (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/update-cluster), [Dynatrace Managed release notes (DT docs)](https://docs.dynatrace.com/managed/whats-new/managed), [Mission Control proactive support (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/basics/mission-control-proactive-support), [Multi-data center failover (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/high-availability/failover), [Recover from another data center (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/high-availability/recover-from-data-center).</sub>

<a id="backup-scope"></a>
## 2. What a Backup Protects — and What It Doesn't

*"You can automatically back up Dynatrace Managed configuration data (naming rules, tags, management zones, alerting profiles, and more), time series metric data, and user sessions."* Each of those comes with conditions:

| Data | In the backup? | Details |
|------|----------------|---------|
| Configuration | **Yes** | Daily, with the metrics snapshot. A configuration-only backup is an option: *"Exclude time series metric data from the backup"* |
| Metrics | **Partly** | Daily; excludes the most frequently changing column families (about 80% of storage) and 1-minute and 5-minute data (DTM-12 §9) |
| User sessions | **Yes, unless excluded** | *"Exclude user sessions from the backup to remain compliant with the General Data Protection Regulation."* (DTM-13 §8) |
| Log Monitoring events | **Only if included** | *"Include backup of Log Monitoring events."* |
| Transaction storage (traces, code-level data) | **No** | *"Transaction storage data is not backed up, so after a restore you might see gaps in deep monitoring data (for example, distributed traces and code-level traces)."* |

The docs' reasoning for leaving transaction storage out: *"By default, Dynatrace retains transaction storage data for only 10 days, so including it in backups is not necessary for long-term retention."* The consequence is that after a restore, trace history starts again from the restore point.

**Recovery objectives.** The *Pre-upgrade information* page's Managed-vs-SaaS comparison gives Managed backup an *RPO = 24h; RTO = 24h*: up to a day of data lost, and up to a day to restore.

> <sub>**Sources:** [Backup and restore a cluster (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/back-up-and-restore-a-cluster), [Pre-upgrade information (DT docs)](https://docs.dynatrace.com/managed/upgrade/up-information).</sub>

<a id="backup-setup"></a>
## 3. Setting Up Backups

**Where.** In the CMC: *"Go to Settings > Backup."* Enable the backup, choose the scope (§2), give *"a full path to the mounted network file storage where backup archives are stored"*, and set a start time. Then keep a copy elsewhere: *"To maximize resilience, save your backup files to an off-site location."*

**The shared file system.** Every node writes to the same place: *"the shared file system needs to be mounted at the same shared directory on each node."* The docs' preference: *"NFSv4 is the recommended protocol. CIFS is not recommended, due to its low performance and resilience."* Test that the Dynatrace user can write there, on each node:

```bash
su - dynatrace -s /bin/bash -c "touch /nfs/dynatrace/backup/$(uname -n)"
```

**The mount that can stop a node.** This is the backup setting most likely to cause an outage:

> *"If the shared file system mount point is not available on system boot, Dynatrace services will not start on that node, which might make the Managed Cluster unavailable. You must disable backups manually to allow Dynatrace to start."*

So *"The shared file system mount must be available at system restart."* Make it persistent, for example in `fstab`. Then a reboot during an NFS outage won't take Dynatrace down with it.

If it happens anyway, the docs give the way out: *"Edit the <install-dir>/elasticsearch/config/elasticsearch.yml file. Remove the line with the path.repo: parameter."* Then start the node again (DTM-11 §3), and re-enable backups once the mount is back.

**How much space.** Plan for twice 20% of the metrics store plus the full Elasticsearch store (DTM-12 §9, DTM-13 §8).

**Watching it.** Neither *"Cassandra backup problem."* nor *"ElasticSearch backup problem."* is emailed (DTM-10 §7). CMC **Home** has a *Last backup* field, which *"Displays the date and storage location of the last backup."* Check it on a schedule. One backup event does email: *"Backup has been disabled because the configuration is no longer supported."*

> <sub>**Sources:** [Backup and restore a cluster (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/back-up-and-restore-a-cluster), [Estimating cluster backup size (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/estimating-cluster-backup-size), [Cluster Management Console (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/basics/cluster-management-console), [Event notification reference (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/configuration/cluster-event-notifications-reference).</sub>

<a id="restore"></a>
## 4. Restore Readiness

A restore is a reinstall from backup onto prepared machines. Most of what can make it fail is decided before it starts.

### 4.1 Preconditions

| Precondition | From the docs |
|--------------|---------------|
| Clean target hosts | *"To restore a Managed Cluster on the same host as the source Managed Cluster, make sure to uninstall it first."* |
| Comparable hardware | *"Make sure the machines prepared for the Managed Cluster restore have a similar hardware and disk layout as the original Managed Cluster and sufficient capacity to handle the load after the restore."* |
| Node count | *"Restore the Managed Cluster to the same number of nodes as the backed-up Managed Cluster whenever possible."* At most two fewer: *"In exceptional cases, you can restore to up to two nodes fewer, but a larger shortfall risks losing the Managed Cluster configuration."* |
| Old cluster stopped | *"Make sure the existing Managed Cluster is stopped to prevent two Managed Clusters with the same ID from connecting to Dynatrace Mission Control."* |
| Matching identities | *"Make sure that system users created for Dynatrace Managed have the same UID:GID identifiers on all nodes."* |
| The same installer | *"To restore the cluster, you need to use the exact same installer version as in the original one."* It is in the backup itself, under each node's `files/<backup-version-number>/` directory |

Before you start, also prepare an inventory: the node IDs (each backup sits in a `node_<node-id>` directory), the new IP address for each node, and which node will be the seed.

### 4.2 The restore order

1. **Run the installer in restore mode** on every node, in parallel, with `--restore`, `--cluster-ip`, `--cluster-nodes`, `--seed-ip` and `--backup-file`.
2. **Start the firewall, Cassandra and Elasticsearch** on each node, then verify both: Cassandra *Status = Up / State = Normal*, Elasticsearch `green` (DTM-12 §5, DTM-13 §5).
3. **Restore Elasticsearch once, on the seed node:** `restore-elasticsearch-data.sh <path-to-backup>/<UUID>`.
4. **Restore Cassandra node by node, seed first:** `restore-cassandra-data.sh`, using each node's backup file.
5. **Optionally repair Cassandra** (DTM-12 §8), then **start Dynatrace** on each node, starting with the seed node.
6. **Finish the restore.** Turn backups back on: *"the backup is automatically disabled after the restore. Once you have finished restoring, you should enable the backup again."* If you restored to fewer nodes, remove the nodes marked **Offline** in the CMC. If the cluster address changed, point OneAgents and Cluster ActiveGates at it (the page gives the `PUT /api/v1.0/onpremise/endpoint/publicIp/agents/<node-id>` call), or update DNS if they connect by name.

**After the restore (Managed 1.350).** Managed 1.350 shortened the time a restored cluster is only partly usable: *"Key monitoring features, such as the Hosts page and filtering in Data Explorer, are now automatically restored after a backup-and-restore migration of a large environment."* Before that release, *"these features could remain unavailable for up to five days after a restore."* A restore runs on the backup's own installer version (§4.1), so a cluster backed up on 1.348 or earlier comes back on that version, and the gap applies to it. The release note doesn't say whether updating such a cluster to 1.350 afterwards starts the recovery, so plan the restore, and what you tell users, around up to five days for a large environment.

### 4.3 Rehearse it

The docs don't prescribe a restore test. In community practice, a restore that has never been rehearsed is treated as unproven. Teams run it on spare hosts, with the production cluster left running and the rehearsal kept off the Mission Control network, so the two clusters never connect at once — verify the approach with Dynatrace support before relying on it.

> <sub>**Sources:** [Backup and restore a cluster (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/back-up-and-restore-a-cluster), [Managed 1.350 release notes (DT docs)](https://docs.dynatrace.com/managed/whats-new/managed/sprint-350). **Derived:** "comes back on that version" combines the exact-installer precondition with the 1.350 note.</sub>

<a id="upgrades"></a>
## 5. Upgrades

### 5.1 The three paths

| Path | How it works |
|------|--------------|
| **Automatic** (the docs' recommendation) | *"By default, installation packages are downloaded automatically from Mission Control as soon as they're available to your Managed Cluster."* The update runs in the maintenance window set under **Settings > Automatic update** |
| **Semi-automatic** | Without automatic downloads or a Mission Control connection, you get an email for each new package, download it, and upload it in the CMC |
| **Manual** | Run the update script on each node: *"it's crucial that you manually update one node at a time."* |

### 5.2 The rules that pace it

- **A day after download.** *"After an installation package is downloaded, a 24-hour waiting period is required before an automatic update can run."* An update downloaded today won't run in tonight's window.
- **A day between updates.** *"You wait 24 hours between cluster updates to ensure that all post-processing steps are completed before initiating another update."*
- **Disk.** *"You have at least 5 GB of available disk space on the partition where Dynatrace Managed is installed."*
- **Downtime below three nodes.** *"If your Managed Cluster consists of fewer than three nodes, Dynatrace Managed will not be accessible during the update process."*
- **Kubernetes workloads.** *"keep the Docker OneAgent installation package for every OneAgent version that's still in use, because without it the Managed Cluster fails to build the OneAgent image and your pods fail to pull it."*
- **The built-in container registry is deprecated (Managed 1.346).** The Kubernetes deployment form now offers *"Amazon ECR (new default)"*, Docker Hub, a private container registry, or *"Built-in registry (deprecated)"*, and *"You must manually update the image locations in the generated dynakube.yaml before deployment."* The deadline: *"The built-in cluster container registry will be shut down on January 1, 2028, and will no longer function because it doesn't support ARM architectures."* The action plan is *"Migrate to Amazon ECR or Docker Hub."*

### 5.3 Skipping versions

*"You can skip a Managed version only if its installed version is divisible by 4."* A cluster on 1.300 can skip 1.302 once 1.304 is released; a cluster on 1.302 can't skip. And skipping can become the only option. In the docs' own example, a cluster on 1.340 must update to 1.342 before 1.344 is released, *"because once version 1.344 is released, version 1.342 is no longer available for download"* — which matters most for offline and semi-automatic clusters. If you would rather never skip, since Managed 1.324 there's a switch for that: *"Turn on Run Dynatrace cluster updates sequentially without skipping a version."* For clusters that can skip, the docs advise: *"it's recommended to wait until the target version is released if you are unable to complete the update process within one month."*

### 5.4 Cadence

*"These software updates are mandatory and are typically released every four weeks."* With the 24-hour rules, a weekly maintenance window, and only some versions skippable, a cluster can only catch up so fast.

> <sub>**Sources:** [Update a cluster (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/update-cluster), [Managed version update logic (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/managed-version-update-logic), [Mission Control proactive support (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/basics/mission-control-proactive-support), [Managed 1.346 release notes (DT docs)](https://docs.dynatrace.com/managed/whats-new/managed/sprint-346). **Derived:** "can only catch up so fast" combines the four-week cadence with the 24-hour rules and the skip rule.</sub>

<a id="version-support"></a>
## 6. Version Support

*"New versions are rolled out within configurable maintenance windows."* The release-notes page lists which versions are currently supported. As it stood when read on 11/10/2026 (page updated Nov 06, 2026):

| Version | Rollout start | Currently supported |
|---------|---------------|---------------------|
| 1.350 | Nov 05, 2026 | Yes |
| 1.348 | Oct 27, 2026 | Yes |
| 1.346 | Aug 31, 2026 | Yes |
| 1.344 | Aug 03, 2026 | Only with Enterprise Success and Support |

1.342 (rolled out Jul 06, 2026) is in the previous-versions table: standard support ended Oct 27, 2026, and Enterprise Success and Support ended Nov 05, 2026.

The table of earlier versions shows the pattern. Standard support for a version ends when the release three versions later rolls out, and Enterprise Success and Support covers it for one more release. For example, 1.338 rolled out May 11, 2026; its standard support ended Aug 03, 2026 (when 1.344 rolled out), and Enterprise Success and Support ended Aug 31, 2026.

**The cluster flags it too (Managed 1.346+).** *"When you're using an unsupported version of Dynatrace Managed, you will be informed with a notification and a special tag on the Licensing page."* A cluster older than 1.346 shows neither, and the note doesn't say whether the tag distinguishes Enterprise Success and Support coverage, so the release-notes table stays the check.

**Managed's future.** No end-of-life has been announced. Dynatrace's annual report for fiscal 2026 still describes the offering: *"We also provide options to deploy our platform in customer-provisioned infrastructure, which we call Dynatrace Managed."* It also notes that *"The majority of our customers deploy Dynatrace as a Software-as-a-Service"*. For clusters planning that move, M2S is the series.

**What falling behind costs** goes beyond support. Several changes in this series only reach you through updates: Cassandra and Elasticsearch fixes (DTM-12 §10, DTM-13 §9), and changes to ActiveGate compatibility and Mission Control ciphers (DTM-14).

> <sub>**Sources:** [Dynatrace Managed release notes (DT docs)](https://docs.dynatrace.com/managed/whats-new/managed), [Managed 1.346 release notes (DT docs)](https://docs.dynatrace.com/managed/whats-new/managed/sprint-346). [Dynatrace FY2026 Form 10-K (Dynatrace IR)](https://ir.dynatrace.com/sec-filings/all-sec-filings/content/0001773383-26-000019/dt-20260331.htm). **Derived:** the "three versions later" pattern reads the rollout and support-end dates in the page's previous-versions table.</sub>

<a id="pha"></a>
## 7. Premium High Availability and Data-Center Failover

### 7.1 What it is, and what it needs

*"Premium High Availability (PHA) extends Dynatrace Managed high availability across two data centers (DC)."* *"PHA keeps a full, active copy of your data in the second DC."* Its requirements:

- *"Deploy at least six nodes, with three nodes per DC."* *"Size both DCs symmetrically."*
- *"PHA is available only for online Managed Clusters."*
- *"Keep round-trip network latency at 100 ms or less."*
- *"Encrypt the connections between nodes in different DCs. Dynatrace Managed doesn't create or install the required certificates."*
- It's a one-way decision: *"You cannot reverse a migration to PHA."*

### 7.2 How failover works

Mission Control drives it — and nothing else can: *"There is no manual or local failover option that bypasses MC."* *"Without access to MC, the secondary DC doesn't autonomously detect that it needs to take over."* *"If Mission Control (MC) detects that two or more Elasticsearch or Cassandra nodes in a data center (DC) are down for 15 minutes, MC automatically stops the server processes in that DC. MC then marks the DC as unhealthy."* The healthy data center takes over, and a healthy server sends an event and an email. When you start the failed processes again (*"After you start the Elasticsearch processes on the nodes that were down"* …), the servers come back. If Mission Control is unreachable, shut down any nodes still running in the failed data center yourself (§7.3). Where Cassandra was down for three hours or more, Cassandra repairs also run, and responsibility moves back only after they finish: *"Thirty minutes after all Cassandra nodes are repaired, Mission Control requests the Nodekeepers to change the responsibility override"* (DTM-12 §8).

Two conditions apply. *"To transfer responsibility to another DC, it must be healthy at the time of failover."* And the mechanism handles a window: *"PHA automatically repairs network disconnections of up to 72 hours between DCs."*

### 7.3 While a data center is down, and after 72 hours

- **During the outage:** *"To avoid data inconsistency while a DC is unavailable, shut down the Dynatrace Server service on all Cluster nodes in the affected DC. Start the services only when network connectivity is stable again."*
- **Past 72 hours:** *"After 72 hours, Mission Control marks the Managed Cluster as not repaired after failover. Such a Managed Cluster is not reliable."* The data center must then be rebuilt, either from the surviving data center (*Rebuild data center*) or from backup (*Recover from a backup*). Both are long, Cluster-API-driven procedures.

> <sub>**Sources:** [Multi-data center high availability (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/high-availability/multi-data-centers), [Multi-data center failover (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/high-availability/failover), [Recover from another data center (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/high-availability/recover-from-data-center), [Rebuild data center (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/high-availability/rebuild-data-center), [Recover from a backup (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/high-availability/recover-from-backup).</sub>

<a id="recovery-paths"></a>
## 8. Choosing a Recovery Path

What was lost decides the path:

![Choosing a Recovery Path](images/16-recovery-paths.png)

<!-- MARKDOWN_TABLE_ALTERNATIVE
| What was lost | What happens | What you do |
|---------------|--------------|-------------|
| One node (3+ node cluster) | Cluster keeps running on three copies; that node's transaction share is gone | Remove the node, install a replacement (DTM-12 §7) |
| The whole cluster | Restore from backup onto new hosts; recent 1-/5-min metrics and traces are not restored | Same node count (at most two fewer), same installer version, old cluster stopped |
| One data center, under 72 h (Premium HA) | After 15 minutes Mission Control fails over to the healthy DC; repaired when it returns (needs MC) | Stop the Server service in the affected DC; restart its services once the network is stable |
| One data center, over 72 h (Premium HA) | Marked "not repaired after failover"; the cluster is not reliable | Rebuild the DC from the surviving one, or restore it from backup |
For environments where SVG doesn't render
-->

A single node is the common case, and it needs no restore: *"The Managed Cluster maintains three copies of this data, so Dynatrace Managed continues to operate after the loss of one node."* A full restore from backup is for losing the cluster. Once the cluster's data is gone, the gaps in §2 become permanent.

### 8.1 A second, independent cluster

Some organizations keep a second Managed cluster as a standby. The documentation describes two recovery mechanisms, restore from backup and Premium HA, and no standby arrangement between independent clusters (§9). Nothing keeps the two in step for you. If a second cluster is your recovery path, four questions decide whether it works, and each needs an owner:

| Question | Why it matters | How to check |
|----------|----------------|--------------|
| Is its configuration the same? | Dashboards, alerting, management zones and settings exist only on the cluster where someone created them | Export each environment on both sides and compare, or deploy both from one source. Each environment in the CMC has an **Export configuration** action: *"To export the configuration of the selected Dynatrace environment"* |
| Does it hold enough history? | Each cluster keeps only the data sent to it, under its own retention settings | Compare retention per environment (DTM-13 §4) with the history the business needs after a failover |
| Can agents and users reach it? | OneAgents carry the endpoints from their installer (DTM-05 §2) | A documented switch: DNS names that can be repointed, or a reconfiguration procedure, and sign-in set up on both clusters |
| Can it carry the load? | After a failover it takes the primary's traffic on top of its own | Its headroom (DTM-15 §4.1) against the combined load |

The second cluster also has its own backups, versions and certificates, so the rest of this notebook applies to it unchanged. DTM-17 (D8) turns these questions into assessment checks.

> <sub>**Sources:** [Single-cluster high availability (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/high-availability/single-cluster-high-availability), [Backup and restore a cluster (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/back-up-and-restore-a-cluster), [Multi-data center failover (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/high-availability/failover), [Recover from another data center (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/high-availability/recover-from-data-center).</sub>

<a id="doc-gaps"></a>
## 9. What the Documentation Does Not Say

**Not found**, in the Managed documentation read 10/01/2026:

| Gap | Working assumption in this notebook |
|-----|-------------------------------------|
| A procedure for testing a restore without disturbing production | Rehearse on isolated hosts, kept off Mission Control (§4.3) |
| Whether Session Replay data is backed up | Assume it isn't. It isn't in the documented scope (§2) |
| A replacement procedure for a single failed node in a single-DC cluster | Remove the node and add a new one (DTM-12 §7) |
| A standby arrangement between two independent clusters, or a way to keep their configuration in step | Treat the second cluster as a cluster in its own right and check it as in §8.1 |

**One conflict.** The Elasticsearch snapshot interval: the backup page says *"every two hours"*, while the *Pre-upgrade information* comparison table says *"Elasticsearch snapshots every 2 days"*. This series follows the backup page (DTM-13 §10). Confirm in CMC **Settings > Backup**.

**One caution.** The backup-and-restore page was reworded on Oct 22, 2026 (*Updated on Oct 22, 2026*, read 11/10/2026). Read its procedure against your installed version.

> <sub>**Sources:** [Backup and restore a cluster (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/back-up-and-restore-a-cluster), [Pre-upgrade information (DT docs)](https://docs.dynatrace.com/managed/upgrade/up-information). **Observed 10/01/2026:** the first three gaps were searched for across the Managed cluster operation and high-availability pages; none is filled. The standby row was added 10/29/2026 after the same pages and the [Cluster Management Console (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/basics/cluster-management-console) page were re-read.</sub>

<a id="recommendation"></a>
## 10. Recommended Approach

1. **Enable backups with a deliberate scope.** Decide on user sessions (GDPR) and log events explicitly, and keep an off-site copy.
2. **Make the backup mount persistent.** A mount that's missing at boot stops Dynatrace starting on that node.
3. **Check CMC Home's *Last backup* weekly.** Backup *problems* are never emailed; only a backup disabled for an unsupported configuration is.
4. **Keep the restore kit ready:** node inventory, seed choice, matching UID:GID, and the installer version taken from the latest backup. After any restore, re-enable backups.
5. **Let automatic updates run** in a weekly maintenance window, and plan around the two 24-hour waits.
6. **Stay inside the supported releases.** Treat the "Currently supported" column as the deadline.
7. **On Premium HA, know your 72 hours.** Stop the Server service in a failed data center, bring it back while it's still inside 72 hours, and treat anything longer as a rebuild.

<a id="summary"></a>
## 11. Summary and Next Steps

Lifecycle health is the layer you only find out about when it's needed. Backups restore configuration, longer-term metrics and (by default) user sessions. They don't restore transaction storage, recent fine-grained metrics, or log events you didn't opt into, and a missing backup mount can keep a node from starting. A restore needs the exact installer version, a matching node count and the old cluster stopped. Upgrades are paced by two 24-hour waits and the skip rule, and a version is supported for about three releases. Premium HA repairs a lost data center by itself for 72 hours; after that it's a rebuild.

**Next in the series:**

| Notebook | Layer |
|----------|-------|
| DTM-17: Running a Cluster Health Assessment | A one-off rated review across all five layers |
| DTM-99: Best Practice Summary and Health-Check Checklist | All five layers on one page |

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official [Dynatrace documentation](https://docs.dynatrace.com/managed).*</sub>
