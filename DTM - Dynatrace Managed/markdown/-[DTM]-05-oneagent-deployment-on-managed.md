# DTM-05: OneAgent Deployment on Managed

> **Series:** DTM — Dynatrace Managed | **Reference:** 05 — OneAgent Deployment on Managed | **Created:** October 2026 | **Last Updated:** 10/21/2026

> **Dynatrace Classic only.** This series applies to **Dynatrace Managed**, the self-hosted, on-premises deployment. None of it applies to Dynatrace SaaS.

## Overview

OneAgent works the same way on Managed as on SaaS once it is running. What differs is how it reaches the cluster, the decisions baked into it at installation, and how its updates are governed per environment. This notebook covers those parts. Operating-system support, deployment methods and Kubernetes are covered for SaaS in ONBRD-05 and the K8S series; the concepts carry over, but the installer, tokens and addresses come from your own cluster.

**What you'll learn:**

- How OneAgent chooses its route to a Managed cluster
- Which decisions to make before the first OneAgent is installed
- How network-zone fallback modes change what happens in an outage
- How to set an update policy that doesn't surprise anyone
- The CPU baseline to check across the estate now

---

## Table of Contents

1. [Short Answer](#short-answer)
2. [How OneAgent Reaches the Cluster](#reaching-cluster)
3. [Decide Before the First Installation](#install-decisions)
4. [Network-Zone Fallback](#zone-fallback)
5. [Update Policy](#update-policy)
6. [Check the CPU Baseline](#cpu-baseline)
7. [What the Documentation Does Not Say](#doc-gaps)
8. [Recommended Approach](#recommendation)
9. [Summary and Next Steps](#summary)

---

## Prerequisites

| Requirement | Details |
|-------------|---------|
| **Dynatrace deployment** | A Dynatrace Managed cluster |
| **Access** | Environment *Download & install OneAgent* permission (DTM-03 §6); *Change monitoring settings* for update settings |
| **Query language** | None. Managed has no Grail, so this series has **no DQL cells** |
| **Version basis** | Written against the Managed documentation read 10/21/2026 |

<a id="short-answer"></a>
## 1. Short Answer

Give OneAgents an **Environment ActiveGate** in every segment, so they never fall back to a Cluster ActiveGate. Decide **network zones and host groups before rollout** and set them at installation: a host group changes how process groups are detected. Choose a **network-zone fallback mode** deliberately. Set **update policy per host group**, using **update windows**, and plan the **process restarts** that every update needs. Inventory **CPUs below x86-64-v2** now.

<a id="reaching-cluster"></a>
## 2. How OneAgent Reaches the Cluster

**The route is in the installer.** *"OneAgent uses a list of communication endpoints that are embedded in the installer to connect to the environment."* That is why the ActiveGate for a sealed segment has to exist before the installer is downloaded (DTM-04 §3).

**The preference order.** Within each network-zone group, *"Index 1—Environment ActiveGates Index 2—Cluster ActiveGates Index 3—Embedded ActiveGates"*. OneAgents spread over the best available tier: *"if several ActiveGates with the low index are available, OneAgents will try to switch between the available ActiveGates on a regular basis."*

**Without a zone, the choice is random.** *"Such OneAgents try to connect to any ActiveGate in a random order and choose the first responding Environment ActiveGate."* And on Managed, the last resort is costly: *"If no Environment ActiveGate can be reached, OneAgents connect to the first responding Cluster ActiveGate."* *"In that case, OneAgent traffic goes to the Dynatrace Managed cluster uncompressed."*

**Open the firewall to the addresses the cluster gives you.** *"The remote Dynatrace addresses to add to the allow list are given on the installation page for OneAgent."* And *"All hosts that are to be monitored need to be able to send data to the Dynatrace cluster."*, either directly or through an ActiveGate.

> <sub>**Sources:** [Reverse proxy or load balancer for OneAgent (DT docs)](https://docs.dynatrace.com/managed/ingest-from/dynatrace-activegate/configuration/set-up-reverse-proxy-for-oneagent), [OneAgent connectivity in network zones (DT docs)](https://docs.dynatrace.com/managed/manage/network-zones/oneagent-connectivity), [Install OneAgent on Linux (DT docs)](https://docs.dynatrace.com/managed/ingest-from/dynatrace-oneagent/installation-and-operation/linux/installation/install-oneagent-on-linux).</sub>

<a id="install-decisions"></a>
## 3. Decide Before the First Installation

Three things are set on the installation command line and are disruptive to change later.

**Host group — decide it first.** It does more than group hosts. *"When the same process is running in two different host groups, Dynatrace will create one process group for each host group."* Services follow: *"Consequently, services are also grouped per host group."* Moving hosts between groups later changes how their processes and services are grouped, and *"OneAgent will automatically restart after the changes are applied."* Set it with `--set-host-group`. The rules: alphanumerics, hyphens, underscores and periods; *"Must not start with dt."*; *"Maximum length is 100 characters."*; *"Each host belongs to at most one host group"*. FAQ-01 discusses naming strategies (written for SaaS; the host-group mechanics are the same).

**Network zone** — set at installation alongside the host group, so the first connection is already local (§4, DTM-04 §5).

**Privileges.** *"You only need root rights to start OneAgent installation."* OneAgent then runs in non-privileged mode where the system supports it; the documented opt-out is the `NON_ROOT_MODE=0` parameter. Installers are downloaded with a PaaS token, and the installing user needs the environment's *Download & install OneAgent* permission (DTM-03 §6).

> <sub>**Sources:** [Organize your environment using host groups (DT docs)](https://docs.dynatrace.com/managed/observe/infrastructure-observability/hosts/configuration/organize-your-environment-using-host-groups) — *"You can define alerting thresholds and OneAgent update settings on a per-host-group basis."*, [Install OneAgent on Linux (DT docs)](https://docs.dynatrace.com/managed/ingest-from/dynatrace-oneagent/installation-and-operation/linux/installation/install-oneagent-on-linux) — *"You need Download/install OneAgent permissions to download and install OneAgent."*</sub>

<a id="zone-fallback"></a>
## 4. Network-Zone Fallback

A zone tells OneAgent where to go first; the **fallback mode** decides how far it may go when its zone has no ActiveGate left:

| Fallback mode | OneAgent may use |
|---------------|------------------|
| *Any ActiveGate* (default) | Its zone, the alternative zones, the default zone, then any other ActiveGate |
| *Only DefaultZone* | Its zone, the alternative zones, and the default zone |
| *None (drop traffic)* | Its zone and the alternative zones only |

The docs map them to the priority groups: *"Any ActiveGate (default) corresponds to priority groups 1, 2, 3, and 4."*, *"None (drop traffic) corresponds to priority groups 1 and 2."*

**Best practice:** use *None* only where data must never leave a segment, and then give that segment enough ActiveGates that losing one doesn't drop its data (DTM-04 §4). Elsewhere, keep the default and rely on alternative zones to choose the next-best route. With Premium HA, the alternative zone is how OneAgents reach the other data center (DTM-04 §5).

A multi-environment ActiveGate has one zone for all its environments: *"Multi-environment ActiveGate can have only one zone that is valid across all connected environments."*

> <sub>**Sources:** [OneAgent connectivity in network zones (DT docs)](https://docs.dynatrace.com/managed/manage/network-zones/oneagent-connectivity) — *"Only DefaultZone corresponds to priority groups 1, 2, and 3."*</sub>

<a id="update-policy"></a>
## 5. Update Policy

OneAgent updates are configured per environment at three levels, and *"Host settings override host group settings, and host group settings override global settings."* Each level offers *Automatic updates at earliest convenience*, *Automatic updates during update windows*, or *No automatic updates* (host groups and hosts can also inherit).

Points that shape the policy:

- **Every update needs process restarts.** *"Following OneAgent automatic updates, you do need to restart all server processes."* Until then, *"these processes will continue to be monitored with the previous version of OneAgent."* Schedule updates where restarts can follow.
- **Turning automatic updates off is silent.** *"OneAgent doesn't update on its own. You aren't notified when instances are outdated."* If you choose it, watch the update statuses in **Settings > Deployment > OneAgent updates**.
- **Windows are shared with ActiveGates.** *"The same windows govern Environment ActiveGate updates"*, so plan one calendar for both (DTM-04 §7).
- **New hosts wait.** *"By default, Dynatrace waits 45 minutes before updating hosts (both EC2 and non-EC2)."* *Update now* skips the wait.
- **Installers are verified.** *"OneAgent installers are signed and the signature is verified for each downloaded installer package."*

**Best practice:** use host groups as update rings. In community practice, a small non-production group updates at earliest convenience and production follows in update windows a few days later — verify the gap against your own change process.

> <sub>**Sources:** [OneAgent update (DT docs)](https://docs.dynatrace.com/managed/ingest-from/dynatrace-oneagent/oneagent-update) — *"By default, global automatic OneAgent updates are turned on, but this is configurable at the global, host group, and host level."*</sub>

<a id="cpu-baseline"></a>
## 6. Check the CPU Baseline

OneAgent on x86-64 is moving to a minimum CPU level: *"Minimum Supported Microarchitecture Level: x86-64-v2"*. The enforcing version isn't set yet: *"The Dynatrace OneAgent version where this is enforced is will be announced in a future release."* When it arrives, *"Systems not meeting the x86-64-v2 baseline will not be able to run the Dynatrace OneAgent."*

The docs name the first generations that qualify: *"Intel Nehalem was the first Intel architecture to fully support the x86-64-v2 baseline."* and *"AMD Bulldozer and its successors support the x86-64-v2 baseline."* For virtual machines: *"Virtualized environments should ensure CPU feature passthrough is enabled to meet the microarchitecture requirements."*

**Best practice:** find hosts and VM templates below x86-64-v2 now, while the enforcing release is still unannounced. On-premises estates that run Managed are more likely than cloud estates to have old hardware or VMs with restricted CPU features.

> <sub>**Sources:** [OneAgent requirements (DT docs)](https://docs.dynatrace.com/managed/ingest-from/dynatrace-oneagent/oa-requirements) — *"x86-64-v2 is a baseline that includes support for SSE3, SSSE3, SSE4.1, SSE4.2, POPCNT, and CMPXCHG16B instructions."*. **Derived:** "more likely to have old hardware" is the notebook's reasoning about on-premises estates, not a documented statement.</sub>

<a id="doc-gaps"></a>
## 7. What the Documentation Does Not Say

**Not found**, in the Managed documentation read 10/21/2026:

| Gap | Working assumption in this notebook |
|-----|-------------------------------------|
| How a OneAgent's version relates to the cluster's version (whether a OneAgent can be newer than the cluster) | Update OneAgents from what the cluster offers in **OneAgent updates**; the ActiveGate update page describes its target versions as versions *"available on the cluster"* |
| A recommended gap between update rings | Set it from your own change process (§5) |
| The OneAgent version that will enforce x86-64-v2 | Not yet announced (§6) |

**An old host-group page.** The host-group page was last updated 08/21/2023; its install example uses OneAgent 1.177. The mechanics it describes are what this notebook relies on; check the installer options on your cluster's own installation page.

> <sub>**Sources:** [Update ActiveGate (DT docs)](https://docs.dynatrace.com/managed/ingest-from/dynatrace-activegate/operation/update-activegate) — *"The newest stable ActiveGate version available on the cluster."*, [Organize your environment using host groups (DT docs)](https://docs.dynatrace.com/managed/observe/infrastructure-observability/hosts/configuration/organize-your-environment-using-host-groups).</sub>

<a id="recommendation"></a>
## 8. Recommended Approach

1. **Deploy Environment ActiveGates before OneAgents** in every segment, so installers carry the right endpoints and no OneAgent falls back to a Cluster ActiveGate (§2, DTM-04 §3).
2. **Design host groups and network zones first**, and set both on the installation command line (§3).
3. **Choose each zone's fallback mode on purpose**; use *None* only where data must stay in the segment, with enough ActiveGates there (§4).
4. **Use host groups as update rings**, with update windows shared with the ActiveGates, and plan the process restarts each update needs (§5).
5. **If you turn automatic updates off, watch the update statuses**, because nothing tells you OneAgents are outdated (§5).
6. **Inventory CPUs and VM templates against x86-64-v2** now (§6).

<a id="summary"></a>
## 9. Summary and Next Steps

On Managed, OneAgent's route, its host group and its network zone are fixed at installation, and its update policy is set per environment, host group and host. Put Environment ActiveGates in place first, decide host groups and zones before rollout, and treat host groups as update rings with planned restarts.

**Next in the series:**

| Notebook | Covers |
|----------|--------|
| DTM-04: ActiveGate Deployment on Managed | The ActiveGates OneAgents connect to |
| DTM-99: Best Practice Summary and Health-Check Checklist | The design, configuration and health checklists |

**Series map:** DTM-00: The Dynatrace Managed Journey routes every notebook in this series.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official [Dynatrace documentation](https://docs.dynatrace.com/managed).*</sub>
