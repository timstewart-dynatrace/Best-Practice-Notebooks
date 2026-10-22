# DTM-01: Cluster Design Best Practices

> **Series:** DTM — Dynatrace Managed | **Reference:** 01 — Cluster Design Best Practices | **Created:** October 2026 | **Last Updated:** 10/21/2026

> **Dynatrace Classic only.** This series applies to **Dynatrace Managed**, the self-hosted, on-premises deployment. None of it applies to Dynatrace SaaS.

## Overview

The design choices that are hardest to change later are made before the first node is installed: how many nodes, where they run, how the cluster survives a lost node or a lost site, and how storage is laid out. This notebook collects the documented constraints behind each choice and the best practice that follows from them. It does not repeat installation steps; the official procedures are linked.

**What you'll learn:**

- Why to size for growth rather than for today, and the node limits
- The rules every node in a multi-node cluster must follow
- How to choose between single-cluster high availability, rack awareness and Premium High Availability
- How to lay out storage so it can grow without a rebuild
- Which host, operating system and cloud choices to avoid for new nodes

---

## Table of Contents

1. [Short Answer](#short-answer)
2. [Size for Growth, Not for Today](#size-for-growth)
3. [Node Count and Placement](#node-placement)
4. [Choosing an Availability Model](#availability-model)
5. [Storage Layout](#storage-layout)
6. [Hosts, Operating Systems and Cloud](#hosts-os-cloud)
7. [What the Documentation Does Not Say](#doc-gaps)
8. [Recommended Approach](#recommendation)
9. [Summary and Next Steps](#summary)

---

## Prerequisites

| Requirement | Details |
|-------------|---------|
| **Dynatrace deployment** | A Dynatrace Managed cluster being designed, or an existing one being reviewed |
| **Query language** | None. Managed has no Grail, so this series has **no DQL cells** |
| **Version basis** | Written against the Managed documentation read 10/21/2026 |

<a id="short-answer"></a>
## 1. Short Answer

Build at least **three identical nodes**, each on its **own host**, on a private network with **10 ms or less** between them, and size them **one-third above** expected load. Pick the availability model by what you must survive: a lost node (single cluster), a lost rack or room (rack awareness, three fault domains), or a lost data center (Premium High Availability, six nodes or more). Put Dynatrace data on **separate, resizable volumes**, never on the root volume or a network file system. Plan to grow by adding nodes.

<a id="size-for-growth"></a>
## 2. Size for Growth, Not for Today

The docs are blunt about up-front sizing: *"Exact sizing of a Managed Cluster isn't always possible, especially in environments with growing traffic. While upfront analysis is useful, it's more important to be able to add capacity as your monitoring needs increase."* Three ways to grow are documented:

| Dimension | What it means | Limit or condition |
|-----------|---------------|--------------------|
| **Horizontal** | Add nodes | *"Dynatrace Managed supports up to 42 Managed Cluster nodes."* |
| **Vertical** | More RAM or CPU per node | *"When adding CPUs or RAM, keep all nodes equally sized."* |
| **Storage** | Larger disks | *"Use resizable disk volumes."* (§5) |

Two design consequences:

- **Growth has a cost in maintenance time.** *"Plan for somewhat longer maintenance windows as your Managed Cluster grows, and scale out to gain monitoring capacity rather than to speed up existing operations."*
- **Check both per-node limits.** Each node size has a host-unit limit and a Full-Stack OneAgent limit, and *"Both columns are limits, so check which one your environment reaches first."* Many small hosts reach the OneAgent limit first. The sizing table and worked examples are in DTM-15 §2.

> <sub>**Sources:** [Hardware requirements (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/installation/managed-hardware-requirements) — *"Plan to scale along these dimensions"*.</sub>

<a id="node-placement"></a>
## 3. Node Count and Placement

**Three nodes is the floor for production.** *"Production environments require a minimum 3-node Managed Cluster for reliability and data redundancy."* With three nodes, configuration, events, user sessions and metrics are held three times, so *"Dynatrace Managed continues to operate after the loss of one node."* What a lost node does cost (transaction storage is not replicated) is in DTM-10 §4.

**The rules every node follows.** The docs list them for multi-node clusters. All nodes must:

| Rule | Why it matters to the design |
|------|------------------------------|
| *"Have the same hardware configuration"* | A cluster runs at the speed of its smallest node when another fails |
| *"Synchronize with NTP"* and *"Be in the same time zone"* | Plan time sources for every host before installation |
| *"Communicate over a private network on the required ports"* | Node-to-node traffic stays off public networks |
| *"Have a network latency between nodes of 10 ms or less"* | Rules out stretching one cluster across distant sites (use Premium HA, §4) |
| *"Have matching Dynatrace Managed system user and group identifiers on all nodes"* | Create the service user with the same UID and GID everywhere, including future nodes and restore targets |

**One node per host.** *"To prevent loss of configuration, metrics, and log data, deploy each node on a separate host."* On virtualization platforms this also means keeping nodes off the same physical host. In community practice, that is enforced with anti-affinity rules — confirm what your platform offers.

**Dedicated hosts.** *"Dynatrace Managed requires a dedicated host."* The host *"must not run other services that are central processing unit (CPU)- or memory-intensive, or that use ports required by Dynatrace Managed."* The resources are reserved too: *"CPU and RAM must be exclusively available for Dynatrace."*

> <sub>**Sources:**</sub>
> - <sub>[Hardware requirements (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/installation/managed-hardware-requirements) — *"Turn off CPU power-saving mode."*</sub>
> - <sub>[Single-cluster high availability (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/high-availability/single-cluster-high-availability) — *"Deploy a minimum of three Cluster nodes."*</sub>
> - <sub>[Operating system requirements (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/installation/operating-system-requirements)</sub>

<a id="availability-model"></a>
## 4. Choosing an Availability Model

Choose by **what you must survive**. A single cluster *"tolerates the loss of individual nodes, but not the loss of an entire location."* Everything beyond that is a deliberate design decision made up front:

| Model | Survives | Minimum | Latency | Key conditions |
|-------|----------|---------|---------|----------------|
| **Single cluster** | One node | 3 nodes | *"Keep the latency between nodes at 10 ms or less."* | Identical hardware, separate hosts (§3) |
| **Rack awareness** | All nodes in one rack | 3 racks, 3+ nodes each | Same low-latency network; *"network latency between sites needs to stay below 10 ms"* | *"The final number of racks is three"*; racks must *"reflect the underlying physical deployment location of nodes"* |
| **Premium High Availability (PHA)** | A whole data center | 6 nodes, 3 per data center | *"Keep round-trip network latency at 100 ms or less."* | Online cluster only, 2 data centers, symmetric sizing, encrypted links between data centers |
| **PHA + rack awareness** | A data center **and** a rack in the surviving one | 9+ nodes per data center to gain from racks | As above | *"For the highest resilience, make every data center in a PHA deployment rack-aware."* |

**Decide before you install.** Rack membership can be set at installation (`--rack-dc`, `--rack-name`) or added later by a conversion. Moving to PHA is one-way: *"You cannot reverse a migration to PHA."* And PHA has conditions a design can't work around: *"PHA is available only for online Managed Clusters."* Offline clusters are covered in DTM-14 §7.

**Don't count the second data center as capacity.** *"For capacity planning in a PHA deployment, treat the nodes in the additional DC as redundant rather than as expanded capacity."* The per-node sizing figures for PHA are lower than for a single data center (DTM-15 §2).

**Plan headroom for a lost node, whatever the model.** *"Plan for a processing capacity one-third higher than typical utilization."* A cluster at full capacity *"can drop data during a node failure."*

**Rack awareness needs nodes, not just labels.** A six-node PHA cluster gains nothing from racks: *"In a rack-aware PHA deployment, six nodes leave one node per rack, where rack awareness gains no failover tolerance."* For each rack, *"Dynatrace doesn't enforce this, but fewer nodes per rack reduce the protection rack awareness provides."*

> <sub>**Sources:**</sub>
> - <sub>[Single-cluster high availability (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/high-availability/single-cluster-high-availability)</sub>
> - <sub>[Rack awareness (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/high-availability/rack-awareness) — *"Each rack holds at least three nodes."*</sub>
> - <sub>[Multi-data center high availability (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/high-availability/multi-data-centers) — *"Deploy at least six nodes, with three nodes per DC."*, *"Encrypt the connections between nodes in different DCs. Dynatrace Managed doesn't create or install the required certificates."*</sub>
> - <sub>**Derived:** the "9+ nodes per data center" figure combines rack awareness's three racks with its three-nodes-per-rack recommendation.</sub>

<a id="storage-layout"></a>
## 5. Storage Layout

Storage is the part of the design that is hardest to fix after data arrives. The documented recommendations, in the order to apply them:

1. **Separate binaries from data.** *"Store Dynatrace binaries and the data store on separate mount points so you can resize the data store independently."*
2. **Keep data off the root volume.** *"Don't store Dynatrace data on the root volume to avoid complexity when resizing later."*
3. **One volume per data type.** *"Mount different types of data storage on separate disk volumes for maximum flexibility and performance."* (The stores are described in DTM-12 and DTM-13.)
4. **Make volumes resizable.** *"Use resizable disk partitions, for example with Logical Volume Manager (LVM)."*
5. **Keep partitions identical.** *"Use the same partition size on all Managed Cluster nodes."* A node whose partition is smaller than the others' is stopped (DTM-10 §5).
6. **Exclude data paths from antivirus.** *"Exclude data storage paths from antivirus scanning to prevent data consistency issues."*

**What kind of storage.** *"Use fast local storage suitable for database workloads."* Encryption is fine when it is transparent: *"Dynatrace Managed also supports encrypted file systems when disk IOPS meet the requirements and encryption is transparent at the operating system level."* Network file systems are not: *"High-latency remote volumes such as NFS or CIFS aren't supported for primary storage; NFS is sufficient for backups only."* And on AWS: *"Amazon Elastic File System (EFS) isn't supported as primary storage for Elasticsearch, because it may lead to index corruption."*

**Grow with nodes, not ever-larger disks.** *"Keep the Long-term Metrics Store data below 2 TB per node."* When more storage is needed, *"add another node to reduce the per-node requirement."* The 2 TB and 4 TB events are covered in DTM-12 §3.

> <sub>**Sources:** [Hardware requirements (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/installation/managed-hardware-requirements) — *"Persistent storage growth degrades Managed Cluster performance, resilience, and availability, and causes issues when adding nodes."*</sub>

<a id="hosts-os-cloud"></a>
## 6. Hosts, Operating Systems and Cloud

**Host basics that rule out designs.** Managed *"supports installation on physical and virtualized hosts but not in containers."* It *"requires a fixed IPv4 address. It supports only IPv4 address spaces."* Set an English locale: *"Set the system locale to an English option, for example LANG=en_US.UTF-8"*.

**Don't build new nodes on an operating system that is about to leave support.** The operating system requirements page lists the supported distributions and announces these removals:

| Operating system | Managed support ends |
|------------------|----------------------|
| Red Hat Enterprise Linux 9.4, 9.7 | 11/01/2026 |
| Ubuntu 16.04 | 11/01/2026 |
| Amazon Linux 2 | 01/01/2027 |

The list moves with every release, so read the page when you provision, not this table. A cluster of identical nodes (§3) also means **one operating system version across all nodes**; in community practice, mixing versions makes later upgrades harder to plan.

**In the cloud,** use the memory-optimized families the docs list (AWS `r6i`, Azure `E…as v6`, Google Cloud `n4-highmem`) and keep the documented ratio: *"Maintain a vCPU to RAM (random access memory) ratio of 1:8 to optimize performance."* The node minimum still applies: *"Hosts must have at least 32 GB RAM."*

> <sub>**Sources:** [Operating system requirements (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/installation/operating-system-requirements) — *"The following operating systems will no longer be supported starting 01 November 2026"*, [Hardware requirements for cloud deployments (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/installation/managed-cloud-requirements), [Hardware requirements (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/installation/managed-hardware-requirements).</sub>

<a id="doc-gaps"></a>
## 7. What the Documentation Does Not Say

**Not found**, in the Managed documentation read 10/21/2026:

| Gap | Working assumption in this notebook |
|-----|-------------------------------------|
| How to keep nodes on separate physical hosts in a virtualized estate | Use the platform's anti-affinity rules (community practice, §3) |
| A recommended starting node count above the minimum of three | Start at three and grow by adding nodes (§2) |

**Conflicts on one page.** The multi-data-center page says, for a cluster spanning data centers, *"The Managed Cluster maintains two copies of this data"*, and later *"PHA stores three copies of all configuration data, metrics, and user sessions in each DC."* This notebook follows the second, more specific statement and the single-cluster page's three copies.

**An example that isn't on the supported list.** The same page's PHA requirements give *"for example Red Hat Enterprise Linux 7 or CentOS 7"* as operating systems that meet the cgroups and systemd condition. Neither appears on the current operating system requirements page. Choose from the supported list (§6).

> <sub>**Sources:** [Multi-data center high availability (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/high-availability/multi-data-centers), [Operating system requirements (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/installation/operating-system-requirements).</sub>

<a id="recommendation"></a>
## 8. Recommended Approach

1. **Decide what you must survive first**, then pick the model: single cluster, rack awareness, or Premium HA (§4). Moving to PHA can't be undone.
2. **Start at three identical nodes**, each on its own dedicated host, with matching UID and GID, NTP and one time zone (§3).
3. **Size one-third above expected load**, checking both the host-unit and the OneAgent limit per node (§2, DTM-15).
4. **Lay out storage before installing**: binaries and data on separate mount points, one volume per data type, LVM, the same sizes on every node, data paths excluded from antivirus, nothing on NFS, CIFS or EFS (§5).
5. **Pick a supported operating system with time left on it**, and use one version on every node (§6).
6. **Write the design down**: node count, racks or data centers, volumes and sizes. Every later change (adding a node, restoring a backup) has to match it.

<a id="summary"></a>
## 9. Summary and Next Steps

A Managed cluster's design comes down to four decisions: how many identical nodes, where they run, what failure they must survive, and how storage grows. Get those right and the cluster grows by adding nodes. Get them wrong and fixing them means rebuilding or migrating the cluster.

**Next in the series:**

| Notebook | Covers |
|----------|--------|
| DTM-02: Cluster Configuration Best Practices | The settings to get right once the cluster is running |
| DTM-15: Capacity and Scaling | Node sizes, scaling and capacity signals |
| DTM-16: Backup, Upgrade, and Disaster Recovery | Premium HA failover and recovery paths |

**Series map:** DTM-00: The Dynatrace Managed Journey routes every notebook in this series.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official [Dynatrace documentation](https://docs.dynatrace.com/managed).*</sub>
