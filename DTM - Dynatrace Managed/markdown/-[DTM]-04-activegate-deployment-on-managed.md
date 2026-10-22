# DTM-04: ActiveGate Deployment on Managed

> **Series:** DTM — Dynatrace Managed | **Reference:** 04 — ActiveGate Deployment on Managed | **Created:** October 2026 | **Last Updated:** 10/21/2026

> **Dynatrace Classic only.** This series applies to **Dynatrace Managed**, the self-hosted, on-premises deployment. None of it applies to Dynatrace SaaS.

## Overview

A Managed deployment uses more kinds of ActiveGate than a SaaS one, and each kind has its own job, its own update mechanism and its own limits. This notebook covers which ActiveGate to deploy for which job, where to put them, how many, how network zones and load balancers change their routing, when one ActiveGate can serve several environments, and how each kind is kept up to date. The run-state side — the Cluster ActiveGate traffic path, load-balancer rules, certificates and health — is in DTM-14.

**What you'll learn:**

- What each ActiveGate type does on Managed, and which purposes can't be combined
- Why internal OneAgents should reach an Environment ActiveGate
- How many ActiveGates to run, and how to size them
- How network zones and load balancers change ActiveGate routing
- What a multi-environment ActiveGate can't do
- How Environment and Cluster ActiveGates are updated

---

## Table of Contents

1. [Short Answer](#short-answer)
2. [The Three ActiveGate Types on Managed](#ag-types)
3. [Where to Place Them](#placement)
4. [How Many, and How Big](#redundancy-sizing)
5. [Network Zones and Load Balancers](#zones-lbs)
6. [Multi-Environment ActiveGates and Extensions](#multi-environment)
7. [Keeping ActiveGates Updated](#updates)
8. [What the Documentation Does Not Say](#doc-gaps)
9. [Recommended Approach](#recommendation)
10. [Summary and Next Steps](#summary)

---

## Prerequisites

| Requirement | Details |
|-------------|---------|
| **Dynatrace deployment** | A Dynatrace Managed cluster |
| **Access** | Cluster Management Console (CMC) cluster administrator for Cluster ActiveGates; environment *Change monitoring settings* for Environment ActiveGate settings |
| **Query language** | None. Managed has no Grail, so this series has **no DQL cells** |
| **Version basis** | Written against the Managed documentation read 10/21/2026 |

<a id="short-answer"></a>
## 1. Short Answer

Use **Cluster ActiveGates** as the public entry point and **Environment ActiveGates** next to your OneAgents. Run **at least two of each kind per data center**, sized so the survivors can take the load. Give synthetic monitoring its **own** ActiveGates. Use **network zones** to keep traffic local, and set `dnsEntryPoint` when a load balancer sits in front of an ActiveGate. Use a **multi-environment** ActiveGate only for routing; extensions need a **dedicated Environment ActiveGate per environment** in its own group. Choose an **update mode and target version** for Environment ActiveGates, and keep Cluster ActiveGates within five versions of current.

<a id="ag-types"></a>
## 2. The Three ActiveGate Types on Managed

The docs arrange ActiveGates in a hierarchy, and *"ActiveGates can only send data to higher hierarchy levels."*

| Level | Type | Job on Managed |
|-------|------|----------------|
| 1 | **Environment ActiveGate** | Belongs to one environment (or several, §6). Routes local OneAgent traffic, runs extensions, monitors clouds and remote technologies, or runs private synthetic monitors |
| 2 | **Cluster ActiveGate** | Belongs to the cluster and is managed in the CMC. The public entry point for external OneAgents, Real User Monitoring beacons and synthetic traffic (DTM-14 §2) |
| 3 | **Embedded ActiveGate** | Runs on every cluster node; *"ActiveGates embedded within cluster nodes"* |

Both installable types matter: *"Dynatrace Managed deployments typically require both ActiveGate types, though the most important type for Dynatrace Managed deployments is the Cluster ActiveGate ."*

**Pick one purpose per ActiveGate.** An ActiveGate is installed for one purpose — route traffic and monitor, run private synthetic monitors, or run the zRemote module — and *"Modules should not be mixed between purposes—such re-configuration is not supported."* The synthetic purpose takes the whole ActiveGate: *"Installing ActiveGate with this purpose disables all other ActiveGate features, including communication with OneAgents."* Plan synthetic locations on their own ActiveGates and don't count them as routing capacity.

> <sub>**Sources:** [Supported connectivity schemes for ActiveGates (DT docs)](https://docs.dynatrace.com/managed/ingest-from/dynatrace-activegate/supported-connectivity-schemes-for-activegates), [Dynatrace ActiveGate (DT docs)](https://docs.dynatrace.com/managed/ingest-from/dynatrace-activegate), [ActiveGate purposes and functionality (DT docs)](https://docs.dynatrace.com/managed/ingest-from/dynatrace-activegate/capabilities), [Install a Cluster ActiveGate (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/installation/install-cluster-activegate).</sub>

<a id="placement"></a>
## 3. Where to Place Them

**Environment ActiveGates next to the OneAgents they serve.** For a network segment whose hosts can't reach the cluster, the docs' advice is one exit: *"you should install an ActiveGate to serve as a single access point, rather than opening the firewall for multiple hosts running OneAgents."*

**Don't let internal OneAgents fall back to a Cluster ActiveGate.** OneAgents prefer an Environment ActiveGate, then a Cluster ActiveGate, then the cluster itself (DTM-14 §2). The fallback costs compression: when no Environment ActiveGate can be reached and OneAgents use a Cluster ActiveGate, *"In that case, OneAgent traffic goes to the Dynatrace Managed cluster uncompressed."* An Environment ActiveGate *"collects messages from OneAgent instances and builds bulks, which are then sent in compressed form"*.

**Decide how each Environment ActiveGate reaches the cluster.** By default it connects straight to the cluster nodes: *"Environment ActiveGates, by default, connect directly to the Dynatrace Cluster (unless custom network zones are used—see below)."* Route it through a Cluster ActiveGate only where that's needed: *"Connecting through the Cluster ActiveGate needs to be arranged, if the Dynatrace Cluster is not directly reachable. For example, if the Environment ActiveGate is in a different network or different data center."*

**Install the ActiveGate before the OneAgents behind it.** For a segment with no other route, order matters: *"first install the ActiveGate and then download the OneAgent installer."* An installer downloaded earlier doesn't know about the ActiveGate, and the OneAgents have to be installed again.

**Keep ActiveGates on dedicated hosts.** *"For optimal performance and enhanced security, we recommend installing and running ActiveGate on a dedicated system."*

> <sub>**Sources:**</sub>
> - <sub>[Route OneAgent traffic (DT docs)](https://docs.dynatrace.com/managed/ingest-from/dynatrace-activegate/capabilities/routing-monitoring-purpose)</sub>
> - <sub>[OneAgent connectivity in network zones (DT docs)](https://docs.dynatrace.com/managed/manage/network-zones/oneagent-connectivity)</sub>
> - <sub>[Supported connectivity schemes for ActiveGates (DT docs)](https://docs.dynatrace.com/managed/ingest-from/dynatrace-activegate/supported-connectivity-schemes-for-activegates)</sub>
> - <sub>[Install a Cluster ActiveGate (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/installation/install-cluster-activegate) — *"In some cases, however, the installation order matters, because the OneAgent installer needs to know about your ActiveGate installation before the OneAgent can be installed."*</sub>
> - <sub>[Linux ActiveGate hardware and system requirements (DT docs)](https://docs.dynatrace.com/managed/ingest-from/dynatrace-activegate/installation/linux/linux-activegate-hardware-and-system-requirements)</sub>

<a id="redundancy-sizing"></a>
## 4. How Many, and How Big

**Two per data center, at least.** The Managed upgrade guide states the rule for the ActiveGate fleet: *"Dynatrace best practices dictate at least two ActiveGates per data center, with additional instances added based on capacity requirements."* Apply it per job: two Cluster ActiveGates behind the load balancer, and two Environment ActiveGates per network segment they serve.

**Size for one being down.** The requirements page says *"It is recommended that the machine on which ActiveGate is running should not exceed 50% CPU and 80% memory."* It also assumes outages: *"it must be assumed that ActiveGates may be inoperable during updates, restarts or short communication problems. In order to ensure high availability, operating ActiveGates should be able to takeover traffic of the unavailable ActiveGates"*.

**The capacity table is the same as for SaaS.** The Managed requirements page lists the same estimated host counts per machine size (for example about 800 hosts on 2 vCPUs and 3.75 GiB, about 1,800 on 4 vCPUs and 7.5 GiB), with the same caveat: *"The real number of hosts may be different depending on the monitored technologies in your environment."* The tables and the survivor arithmetic are worked through in ONBRD-03 §3 and FAQ-10, which are written for SaaS but use these figures.

> <sub>**Sources:** [Plan the upgrade (DT docs)](https://docs.dynatrace.com/managed/upgrade/up-plan) — *"When sizing the ActiveGates, please ensure that you have sufficient capacity for failure events."*, [Linux ActiveGate hardware and system requirements (DT docs)](https://docs.dynatrace.com/managed/ingest-from/dynatrace-activegate/installation/linux/linux-activegate-hardware-and-system-requirements).</sub>

<a id="zones-lbs"></a>
## 5. Network Zones and Load Balancers

**Zones belong to the cluster.** *"Network zones are organized per Managed cluster."* They are switched on per environment (**Settings > Preferences > Network zones**, *"Turn on Enable network zones in this environment"*). They keep OneAgents and ActiveGates on local routes and fall back elsewhere: *"You can still use ActiveGates in other zones as a backup option in case primary ActiveGates becomes unavailable for some reason."*

**Environment ActiveGates have their own priority order.** Within a zone, an Environment ActiveGate prefers the embedded ActiveGates on the cluster nodes, then Cluster ActiveGates: *"Index 1—Embedded ActiveGates Index 2—Cluster ActiveGates"*. And it spreads the load: *"For load balancing, an Environment ActiveGate rotates through all available endpoints of ActiveGates with the lowest possible index."* How OneAgents choose between ActiveGates is in DTM-05 §2.

**With Premium HA, zones are how traffic stays local.** *"PHA uses network zones to route telemetry to Cluster nodes in different DCs. Network zones let OneAgents and ActiveGates prefer local routes while retaining the ability to fail over to another DC during an outage."* Design the zones together with the data centers (DTM-01 §4).

**Tell the ActiveGate about a load balancer in front of it.** Components discover each other's addresses automatically, *"unless there are networking devices (proxies, load balancers) in your environment, which should be taken into account, and of which Dynatrace is not aware."* For a load balancer between OneAgents and an ActiveGate, *"you should specify the load balancer's address as the dnsEntryPoint property in the ActiveGate configuration."* OneAgents need nothing: *"There is no need to configure OneAgent to use a reverse proxy."* For a load balancer between an ActiveGate and its next hop, the docs name `seedServerUrl` and `ignoreClusterRuntimeInfo`.

> <sub>**Sources:**</sub>
> - <sub>[Get started with network zones (DT docs)](https://docs.dynatrace.com/managed/manage/network-zones/network-zones-basic-info)</sub>
> - <sub>[ActiveGate connectivity in network zones (DT docs)](https://docs.dynatrace.com/managed/manage/network-zones/activegate-connectivity) — *"The preferable ActiveGate is the one with lowest possible index."*</sub>
> - <sub>[Multi-data center high availability (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/high-availability/multi-data-centers)</sub>
> - <sub>[Supported connectivity schemes for ActiveGates (DT docs)](https://docs.dynatrace.com/managed/ingest-from/dynatrace-activegate/supported-connectivity-schemes-for-activegates), [Reverse proxy or load balancer for OneAgent (DT docs)](https://docs.dynatrace.com/managed/ingest-from/dynatrace-activegate/configuration/set-up-reverse-proxy-for-oneagent)</sub>

<a id="multi-environment"></a>
## 6. Multi-Environment ActiveGates and Extensions

One Environment ActiveGate can serve several environments of the same cluster, which *"significantly reduces the maintenance and setup overhead."* It is a routing ActiveGate only. The documented limits:

| Can't be used to | |
|------------------|--|
| *"Connect to environments from different clusters"* | |
| *"Install the zRemote module for z/OS monitoring"* | |
| *"Monitor remote technologies with the Extensions framework"* | |
| *"Execute monitors from private Synthetic locations"* | |
| *"Run Database insights"* | |

**Extensions need their own ActiveGates.** *"Extensions are not supported on multi-environment ActiveGates or Cluster ActiveGates. To run Extensions, deploy a dedicated Environment ActiveGate for each environment and enable the Extension Execution Controller (EEC) ."* Assign it to an ActiveGate group, because *"Dynatrace uses a group to instruct the extension where it should run."* *"If you plan to use a single ActiveGate to run a remote extension, assign it to a dedicated group containing only that ActiveGate."* Set the group at installation with `--set-group`, which *"you can't use"* during an update.

**Get the main environment right.** The multi-environment configuration names one environment as `mainTenant`, and a mistake there takes the ActiveGate out everywhere: *"Incorrect configuration of the main environment will result in the rejection of the ActiveGate in all configured environments"*. Check the ActiveGate log for the *"Working mode is set to MULTITENANT"* line after every change.

> <sub>**Sources:** [Multi-environment deployment of ActiveGate (DT docs)](https://docs.dynatrace.com/managed/ingest-from/dynatrace-activegate/configuration/configure-an-environment-activegate-for-multi-environment-support), [ActiveGate group (DT docs)](https://docs.dynatrace.com/managed/ingest-from/dynatrace-activegate/activegate-group) — *"An ActiveGate can belong to only one group."*</sub>

<a id="updates"></a>
## 7. Keeping ActiveGates Updated

The two installable types are updated in different places.

**Environment ActiveGates** — per environment, under **Settings > Deployment > ActiveGate updates**:

| Setting | Options |
|---------|---------|
| Update mode | *Automatic (earliest convenience)*, *Automatic (during update window)*, *No automatic updates* |
| Target version | *Latest stable*, *Previous stable*, *Older stable*, or a specific main version |

Update windows are shared with OneAgent: *"The same update windows govern both OneAgent and ActiveGate"*. Pinning is coarse: *"Unlike OneAgent, ActiveGate does not expose a sub-version selector; the latest sub-version of the chosen main version is always applied."*

**Cluster ActiveGates** — in the CMC under **Deployment Status > ActiveGates**, with an *Automatic updates* toggle per ActiveGate; turned off, *Update* applies a new version immediately. Keep them current: the yellow icon means *"your ActiveGate is behind by more than five versions. You should update these ActiveGates as soon as possible."* Back up the configuration first: settings are kept in `custom.properties` and `launcheruserconfig.conf`, *"but it's good practice to back them up before updating the ActiveGate."*

**Containerized ActiveGates don't auto-update.** *"Auto-update and the manual Update now action only support host-based deployments installed via the installer."* Their versions come from the deployment tooling.

**Best practice:** put Environment ActiveGates on *Automatic (during update window)*, using the same windows as the OneAgents they serve (DTM-05 §5). In community practice, two ActiveGates in a pair are kept in different windows, so one is always up — verify against your window layout.

> <sub>**Sources:** [Update ActiveGate (DT docs)](https://docs.dynatrace.com/managed/ingest-from/dynatrace-activegate/operation/update-activegate), [Update Cluster ActiveGate (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/update-dynatrace-managed-activegate) — *"The availability check is performed at 30-minute intervals."*</sub>

<a id="doc-gaps"></a>
## 8. What the Documentation Does Not Say

**Not found**, in the Managed documentation read 10/21/2026:

| Gap | Working assumption in this notebook |
|-----|-------------------------------------|
| Sizing figures for Cluster ActiveGates specifically | Use the routing table and the 50% CPU / 80% memory rule (§4); DTM-14 §9.4 has more |
| Whether Cluster ActiveGates follow the Environment ActiveGate update modes and target version | Treat them separately: the toggle in Deployment Status (§7) |

**Pages of different ages.** The Cluster ActiveGate update page was published 06/03/2020 and describes a single *Automatic updates* toggle. The ActiveGate update page, updated 07/14/2026, describes update modes, target versions and windows for Environment ActiveGates. This notebook follows each page for its own ActiveGate type.

**A port that two pages state differently** — 9999 or 443 for the Cluster ActiveGate's inbound hop — is recorded in DTM-14 §10.

> <sub>**Sources:** [Update Cluster ActiveGate (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/update-dynatrace-managed-activegate), [Update ActiveGate (DT docs)](https://docs.dynatrace.com/managed/ingest-from/dynatrace-activegate/operation/update-activegate).</sub>

<a id="recommendation"></a>
## 9. Recommended Approach

1. **Assign each ActiveGate one purpose**: Cluster ActiveGates for external traffic, Environment ActiveGates for local routing and monitoring, separate ActiveGates for private synthetic monitors (§2).
2. **Put Environment ActiveGates in every segment with OneAgents**, so no internal OneAgent falls back to a Cluster ActiveGate and sends uncompressed (§3).
3. **Install the ActiveGate before the OneAgents behind it** (§3).
4. **Run at least two per data center for each job**, sized at 50% CPU and 80% memory with one down (§4).
5. **Design network zones with the data centers**, and set `dnsEntryPoint` for every load balancer in front of an ActiveGate (§5).
6. **Use multi-environment ActiveGates for routing only**; give extensions a dedicated Environment ActiveGate per environment in its own group (§6).
7. **Set an update mode and target version** for Environment ActiveGates, and update Cluster ActiveGates before they fall five versions behind (§7).

<a id="summary"></a>
## 10. Summary and Next Steps

On Managed, ActiveGates come in three levels. Cluster ActiveGates face outward, Environment ActiveGates face the OneAgents, and embedded ActiveGates run on the nodes. Give each kind one purpose, place Environment ActiveGates close to the agents, run them in pairs per data center, and keep both kinds within reach of current.

**Next in the series:**

| Notebook | Covers |
|----------|--------|
| DTM-05: OneAgent Deployment on Managed | How OneAgents choose a route, install-time decisions and update policy |
| DTM-14: Cluster ActiveGates and Mission Control Connectivity | The traffic path, load-balancer rules, certificates and health |

**Series map:** DTM-00: The Dynatrace Managed Journey routes every notebook in this series.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official [Dynatrace documentation](https://docs.dynatrace.com/managed).*</sub>
