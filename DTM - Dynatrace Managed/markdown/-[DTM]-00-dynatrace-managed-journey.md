# DTM-00: The Dynatrace Managed Journey

> **Series:** DTM — Dynatrace Managed | **Reference:** 00 — The Dynatrace Managed Journey | **Created:** October 2026 | **Last Updated:** 10/29/2026

> **Dynatrace Classic only.** This series applies to **Dynatrace Managed**, the self-hosted, on-premises deployment. None of it applies to Dynatrace SaaS.

## Overview

This series covers what is specific to running **Dynatrace Managed**, the self-hosted deployment, as opposed to Dynatrace SaaS. This notebook is its map: what the series covers, which notebook to open for a given question, how the notebooks are numbered, and where to go when the question is about leaving Managed for SaaS.

**What you'll find here:**

- What this series does and does not apply to
- A reading order by situation
- The numbering bands and what each holds
- Where the notebooks of the former MCH series went

---

## Table of Contents

1. [Scope](#scope)
2. [Where to Start](#where-to-start)
3. [How the Series Is Numbered](#numbering)
4. [Formerly the MCH Series](#formerly-mch)
5. [Summary and Next Steps](#summary)

---

## Prerequisites

| Requirement | Details |
|-------------|---------|
| **Dynatrace deployment** | A self-hosted **Dynatrace Managed** cluster, or one being planned. Nothing in this series applies to Dynatrace SaaS. |
| **Query language** | None. Managed has no Grail, so this series has **no DQL cells** |

<a id="scope"></a>
## 1. Scope

Every notebook in this series applies to **Dynatrace Managed only**. Managed has no Grail: Dynatrace describes Grail as *"available only in Dynatrace deployed as SaaS."* So the series contains no DQL and no importable notebook JSON. Every check is a Cluster Management Console (CMC) view, a cluster event, or a Cluster API call, and the series is published as markdown and PDF.

Advice written for SaaS elsewhere in this collection (Grail buckets, OpenPipeline, Workflows, segments) does not carry over to a Managed cluster.

> <sub>**Sources:** [Unavailable in Dynatrace Managed (DT docs)](https://docs.dynatrace.com/managed/upgrade/unavailable-in-managed) — *"Some innovations leverage the power of Grail, which is available only in Dynatrace deployed as SaaS."*</sub>

<a id="where-to-start"></a>
## 2. Where to Start

| Your situation | Start with | Then |
|----------------|------------|------|
| You run a Managed cluster and want to know what to check, and how often | DTM-99: Best Practice Summary and Health-Check Checklist | The notebook for whichever layer a check fails |
| You are designing a new cluster, or reviewing an existing one's layout | DTM-01: Cluster Design Best Practices | DTM-15 for sizing |
| You want to check the cluster's settings against best practice | DTM-02: Cluster Configuration Best Practices | DTM-10 §7 for notifications |
| You are deciding how to split the cluster into environments, or who can sign in | DTM-03: Environment Best Practices | DTM-15 §6 for quotas |
| You are placing or updating ActiveGates | DTM-04: ActiveGate Deployment on Managed | DTM-14 for the traffic path and certificates |
| You are rolling out OneAgents or setting their update policy | DTM-05: OneAgent Deployment on Managed | DTM-04 first |
| You are reviewing a cluster once — before a purchase, a renewal, a migration or after a run of incidents — and need a rated result | DTM-17: Running a Cluster Health Assessment | The notebook each failing check points to |
| You are new to how a Managed cluster works | DTM-10: Managed Cluster Architecture and Health Model | DTM-11 to DTM-16, one per health layer |
| A node, service or restart question | DTM-11: Server Nodes and Processes | — |
| A data store question (metrics, user sessions, logs) | DTM-12: Cassandra Metrics Store Health, DTM-13: Elasticsearch Store Health | DTM-15 for sizing |
| Agents or Mission Control cannot reach the cluster | DTM-14: Cluster ActiveGates and Mission Control Connectivity | — |
| The cluster is running out of headroom | DTM-15: Capacity and Scaling | — |
| Backups, upgrades, version support or disaster recovery | DTM-16: Backup, Upgrade, and Disaster Recovery | — |
| You need business analytics on Managed, which has no business events | BIZEV-07: Gen2 vs Gen3 — Business Events Without (or Before) the Full Move (Path B, the classic toolchain) | — |
| You are planning to move from Managed to SaaS | DTM-99's checklist, to confirm the source cluster is healthy | The M2S series (Managed to SaaS Migration) |

<a id="numbering"></a>
## 3. How the Series Is Numbered

| Numbers | Band | Holds |
|---------|------|-------|
| **00** | Map | This notebook |
| **01–09** | Design and configure | Best practices for designing and configuring a cluster, its environments, ActiveGates and OneAgents (DTM-01 to DTM-05) |
| **10–19** | Operate | Keeping a running cluster healthy: the health model, one notebook per layer (DTM-10 to DTM-16), and a one-off assessment method (DTM-17) |
| **99** | Summary | The health checklist, the design and configuration review, and the review cadence for the whole series |

Each band has room for more notebooks, so a new topic goes into its band instead of shifting the numbers after it.

<a id="formerly-mch"></a>
## 4. Formerly the MCH Series

The operate band was published as its own series, **MCH — Managed Cluster Health**, until October 2026. Its notebooks moved here with new numbers and image names. If you followed an old link or citation:

| Was | Now | Title |
|-----|-----|-------|
| MCH-01 | DTM-10 | Managed Cluster Architecture and Health Model |
| MCH-02 | DTM-11 | Server Nodes and Processes |
| MCH-03 | DTM-12 | Cassandra Metrics Store Health |
| MCH-04 | DTM-13 | Elasticsearch Store Health |
| MCH-05 | DTM-14 | Cluster ActiveGates and Mission Control Connectivity |
| MCH-06 | DTM-15 | Capacity and Scaling |
| MCH-07 | DTM-16 | Backup, Upgrade, and Disaster Recovery |
| MCH-99 | DTM-99 | Best Practice Summary and Health-Check Checklist |

<a id="summary"></a>
## 5. Summary and Next Steps

This series is the home for what is specific to Dynatrace Managed. For a running cluster, start with the DTM-99 checklist and follow its pointers into DTM-10 to DTM-16. For a move to SaaS, check the source cluster with DTM-99 first, then follow the M2S series.

**Related series:** M2S (Managed to SaaS Migration) covers moving a Managed cluster to SaaS.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official [Dynatrace documentation](https://docs.dynatrace.com/managed).*</sub>
