# DTM-02: Cluster Configuration Best Practices

> **Series:** DTM — Dynatrace Managed | **Reference:** 02 — Cluster Configuration Best Practices | **Created:** October 2026 | **Last Updated:** 10/21/2026

> **Dynatrace Classic only.** This series applies to **Dynatrace Managed**, the self-hosted, on-premises deployment. None of it applies to Dynatrace SaaS.

## Overview

A new Managed cluster runs on defaults, and several of them are worth changing before anyone depends on the cluster. This notebook covers the cluster-level settings in the Cluster Management Console (CMC) and on the nodes that shape how the cluster is supported, how it reaches people, who can sign in, and how it survives its own updates. Where another notebook in the series already covers a setting in depth, this one states the decision and points there.

**What you'll learn:**

- How to keep custom properties through updates
- How to make cluster email reliable and encrypted
- What the Mission Control preferences and remote-access settings trade off
- Which sign-in and session settings to tighten
- How the cluster gains root privileges, and what breaks it
- When to dedicate nodes to UI or OneAgent traffic

---

## Table of Contents

1. [Short Answer](#short-answer)
2. [Keep Custom Properties Through Updates](#custom-properties)
3. [Make Cluster Email Reliable](#email)
4. [The Mission Control Relationship](#mission-control)
5. [The Internet Proxy](#internet-proxy)
6. [Sign-in and Session Security](#sign-in)
7. [Elevated Operating System Permissions](#elevated-permissions)
8. [Node Traffic Roles](#node-roles)
9. [Backups From Day One](#backups)
10. [What the Documentation Does Not Say](#doc-gaps)
11. [Recommended Approach](#recommendation)
12. [Summary and Next Steps](#summary)

---

## Prerequisites

| Requirement | Details |
|-------------|---------|
| **Dynatrace deployment** | A running Dynatrace Managed cluster |
| **Access** | CMC cluster administrator; shell access to the nodes for §2 and §7 |
| **Query language** | None. Managed has no Grail, so this series has **no DQL cells** |
| **Version basis** | Written against the Managed documentation read 10/21/2026 |

<a id="short-answer"></a>
## 1. Short Answer

Put every node-level property change in **`custom.settings`**, not `config.properties`. Use **your own SMTP server with an encrypted connection** and Mission Control as the fallback, and fill in the **emergency contacts**. Choose the **remote-access scope** and the **Mission Control reporting** settings deliberately. Prefer **LDAP or SSO** for users; for any local accounts keep a **12-character minimum** and **limit concurrent sessions**. Watch the **Dynatrace service user's password expiry**, which can break privilege elevation. Turn **backups** on before you need them.

<a id="custom-properties"></a>
## 2. Keep Custom Properties Through Updates

Node-level properties live in `/opt/dynatrace-managed/server/conf/config.properties`, but that file is the wrong place to change them: *"The file is overwritten during an update."* So *"your custom configuration won't be preserved during an update."*

The documented way is a `custom.settings` file in `/opt/dynatrace-managed/installer/` on **each node**: *"During an upgrade, the installer reads custom.settings and modifies config.properties accordingly."* It is re-applied on restarts too: *"Dynatrace Managed also executes this custom configuration action with each restart of the Dynatrace service."*

**Best practice:** keep the same `custom.settings` on every node, and keep a copy under change control. A new node or a restored cluster needs it too, and the file is easy to forget.

> <sub>**Sources:** [Configure Cluster properties (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/configuration/configure-cluster-properties) — *"Place this custom.settings file on all Managed Cluster nodes in the /opt/dynatrace-managed/installer directory."*</sub>

<a id="email"></a>
## 3. Make Cluster Email Reliable

Cluster event notifications are only as good as the mail path. Three delivery methods are documented under **Settings > Emails > SMTP server**:

| Method | Behavior | When to use it |
|--------|----------|----------------|
| **SMTP server with Mission Control fallback** (default) | Your server sends; *"If your SMTP server becomes unavailable and MC email fallback is available for your cluster, MC attempts to deliver the email."* | Most clusters |
| **SMTP server** | *"If your SMTP server becomes unavailable, your email won't be sent."* | When mail must never leave through Mission Control |
| **Mission Control** | Mission Control sends all email | Clusters with no internal mail server |

**Encrypt the connection.** The default is unencrypted: *"The default is No encryption on port 25."* Two options are labelled *Secure*: **SSL/TLS required** and **STARTTLS required**. **STARTTLS optional** is labelled *Not secure*: *"If a STARTTLS connection can't be established, send email without encryption."*

**Prove it works.** Send the test message and check the inbox; the page describes what to expect: *"Check the inbox of the Email test address account to make sure it received a "Dynatrace Managed: Mail server setup" message with the expected sender email address."*

**Fill in both recipient lists.** Under **Settings > Emails > Email notifications**, set the notification recipients to a monitored, shared mailbox, and set the **emergency contacts**, who *"receive notifications from Dynatrace product experts in the event of catastrophic Managed Cluster states or critical security vulnerabilities that must be addressed."* Which cluster events to route, and which never email, is in DTM-10 §6–§7.

> <sub>**Sources:** [Configure SMTP server connection (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/configuration/configure-smtp-server-connection), [Configure Cluster event notifications (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/configuration/configure-cluster-event-notifications) — *"Optional: Turn on Also notify all active users with cluster admin rights ."*</sub>

<a id="mission-control"></a>
## 4. The Mission Control Relationship

**What always goes to Mission Control.** Two reports can't be switched off. Usage and billing data *"can't be changed to ensure proper billing and license verification of your deployment."* Cluster health reporting *"can't be changed to ensure Dynatrace support of your deployment."*

**What is your choice, and what it costs.**

| Preference (Settings > Preferences) | If you turn it off |
|-------------------------------------|---------------------|
| Report cluster and OneAgent events to Mission Control | *"When turned off, your organization is responsible for monitoring system events and collecting log files necessary for problem resolution prior to contacting Dynatrace product experts."* |
| Send information about monitored technologies and feature usage | You lose the proactive alerts it feeds: *"Dynatrace proactively sends alerts for incompatibilities or technology-specific risks related to your environment."* |

**Remote access: choose the scope and the role.** The scopes are described in DTM-14 §8. The trade-off in one line each:

- **All** gives *"the full power of proactive support"*.
- **Read-only access to all** *"significantly limits the level of proactive support"*, and *"only the Viewer role is available for a remote-access user."*
- **Approved** *"gives you maximum control over who can access your Managed Cluster, but it significantly impacts the Dynatrace product expert team's ability to provide you with proactive support."*

Each remote-access user gets the **Admin**, **User** or **Viewer** role. Review what the role allows before granting it: the permissions include *"Run diagnostic operations"*, which *"Allows the user to run diagnostic operations such as service restarts, run diagnostic scripts on Managed Cluster node hosts, and access the database."*

> <sub>**Sources:** [Configure Cluster preferences (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/configuration/configure-cluster-preferences) — *"All cluster preferences settings are turned on by default."*, [Configure Cluster remote access (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/configuration/configure-cluster-remote-access) — *"Your cluster administrators receive email notifications about pending remote access requests."*</sub>

<a id="internet-proxy"></a>
## 5. The Internet Proxy

The proxy carries more than Mission Control traffic. It is configured *"to allow Mission Control (MC) proactive support , receive MC updates, and use external problem notifications via tools such as ServiceNow, Jira, and webhooks."* What the proxy itself must support (WebSockets, SNI) is in DTM-14 §6.

Three settings decide whether it keeps working:

- **Authentication.** *"Configure Basic authentication together with TLS."* NTLMv1 is listed as *Deprecated*.
- **Exclusions for internal targets.** When problem notifications go to internal systems, *"add them to the exclusion list."* Otherwise internal webhooks are sent to the proxy.
- **Inspecting proxies are supported** if the cluster trusts the proxy's root certificate: *"Configure Dynatrace Managed to trust a root certificate whose private key is known to the proxy."* When that certificate changes, the cluster has to be told (the page gives the `reconfigure.sh --update-cert` command).

> <sub>**Sources:** [Configure internet proxy (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/configuration/configure-internet-proxy) — *"Yes, Dynatrace supports transparent proxy configuration."*</sub>

<a id="sign-in"></a>
## 6. Sign-in and Session Security

**Use a central directory where you can.** The password page's own advice: *"If possible, use your organization’s central user repository service (LDAP or SSO)."* The password policy only covers local accounts: *"The password complexity rules apply only to the embedded administrator account and internal user accounts"*.

**Check the minimum length on older clusters.** The default depends on when the cluster was installed: 8 characters *"for existing Managed Clusters installed before 1.206"*, 12 for newer ones, and 12 is the recommended value. A cluster that has been upgraded for years may still be on 8. A policy change takes effect at the next sign-in: *"When the password policy changes, Dynatrace prompts all users to update their passwords the next time they sign in."*

**Limit sessions.** *"Limit concurrent sessions per user account to the minimum required to perform job duties."* Limits can differ for regular and admin users, under **User authentication > User sessions**. The same view lists active sessions by sign-in type, including *"DEVOPSTOKEN - Sign-in from Mission Control proactive support"*, and lets you end any of them.

**Set an automatic sign-out.** There is none by default for pages that refresh themselves: *"By default, there's no automatic sign-out for users who stay on pages that automatically refresh."* It is set through the REST API.

> <sub>**Sources:** [Password complexity rules (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/configuration/password-complexity-rules), [Manage user sessions (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/configuration/manage-user-sessions) — *"Use the REST API to update the automatic sign-out policy."*</sub>

<a id="elevated-permissions"></a>
## 7. Elevated Operating System Permissions

Maintenance needs root. The Dynatrace service user needs elevated permissions to *"Run the installation or reconfiguration script"*, *"Add or remove a Managed Cluster node"*, and *"Start, stop, restart, or check the status of services"*. It gets them through the `dtrun` wrapper, which uses `sudo` by default or the alternative chosen at installation (for example `pbrun`). `dtrun` can only run what `/opt/dtrun/dtrun.conf` lists, and *"The root user owns the /opt/dtrun directory and all files inside it for security reasons."*

**Best practices:**

- **Agree the elevation method with your security team before installing.** Changing it later means running the reconfiguration script on every node.
- **Watch the service user's password expiry.** The troubleshooting checks include `chage -l dynatrace`, because *"Password expiration can cause issues with sudo access."* An expired password can block node operations at the worst moment, such as during an update.
- **Keep `dtrun.conf` as installed.** It is the allow-list of what the cluster may run as root.

> <sub>**Sources:** [Configure elevated permissions (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/configuration/configure-elevated-permissions) — *"By default, Dynatrace Managed uses sudo to elevate permissions for maintenance operations."*</sub>

<a id="node-roles"></a>
## 8. Node Traffic Roles

Each node can serve web UI traffic, OneAgent traffic, or both. Splitting them is a documented option for larger clusters: *"By configuring one or the other of these options, you can achieve higher stability, better cache behavior, and faster UI response times."* Large clusters already split by default: *"starting with Dynatrace Managed 1.214, web UI traffic is by default turned off when installing the 13th and subsequent nodes (or 7th and subsequent nodes in a data center for Premium HA)."*

For a small cluster, leave every node serving both: a three-node cluster that dedicates a node to the UI has fewer nodes left to absorb a failure. In community practice, splitting roles starts to pay off only on larger clusters — verify against your own UI and ingest load. Disabling OneAgent traffic is also the first step before removing a node (DTM-11 §5).

> <sub>**Sources:** [Configure Cluster capabilities (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/configuration/configure-cluster-capabilities) — *"Certain deployments may require that specific nodes only serve UI traffic and not process OneAgent data."*</sub>

<a id="backups"></a>
## 9. Backups From Day One

Backups are configured in the CMC (**Settings > Backup**), and they are not something to add after the first incident. The decisions to make at configuration time:

- **Scope.** *"User sessions may contain sensitive information. Exclude user sessions from the backup to remain compliant with GDPR ."* Log Monitoring events are included only if you choose them.
- **Location.** *"To maximize resilience, save your backup files to an off-site location."* Use NFSv4; *"Due to low performance and resilience, we don't recommend CIFS."*
- **The boot-time mount.** *"If the shared file system mount point isn't available on system boot, Dynatrace won't start on that node."* Make the mount persistent.

What a backup does and doesn't protect, sizing, and restore are in DTM-16 §2–§4.

> <sub>**Sources:** [Backup and restore a cluster (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/back-up-and-restore-a-cluster) — *"The shared file system mount must be available at system restart."*</sub>

<a id="doc-gaps"></a>
## 10. What the Documentation Does Not Say

**Not found**, in the Managed documentation read 10/21/2026:

| Gap | Working assumption in this notebook |
|-----|-------------------------------------|
| A recommended automatic sign-out value | Set one that matches your organization's session policy (§6) |
| When to start splitting node traffic roles below the 13-node default | Keep both roles on every node in small clusters (§8) |

**A path that doesn't match the UI it names.** The preferences page says remote-access settings are under *"Settings > Remote access permissions > SMTP server"*. The remote-access page gives **Settings > Remote access permissions**. This notebook follows the remote-access page.

> <sub>**Sources:** [Configure Cluster preferences (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/configuration/configure-cluster-preferences), [Configure Cluster remote access (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/configuration/configure-cluster-remote-access).</sub>

<a id="recommendation"></a>
## 11. Recommended Approach

1. **Create `custom.settings` on every node** before the first property change, and keep a copy under change control (§2).
2. **Configure your SMTP server with SSL/TLS required or STARTTLS required**, keep Mission Control as the fallback, send the test message, and set both the notification recipients and the emergency contacts (§3).
3. **Decide the Mission Control preferences and the remote-access scope and role on purpose**, and record why (§4).
4. **Set up the internet proxy** with Basic authentication over TLS, and add internal webhook targets to the exclusion list (§5).
5. **Connect LDAP or SSO.** For the remaining local accounts, set a 12-character minimum, limit concurrent sessions, and set an automatic sign-out (§6).
6. **Agree the privilege-elevation method before installing**, and monitor the service user's password expiry (§7).
7. **Leave node traffic roles alone on small clusters** (§8).
8. **Turn on backups** to an off-site NFSv4 location with a persistent mount, excluding user sessions if GDPR applies (§9).

<a id="summary"></a>
## 12. Summary and Next Steps

Most of a Managed cluster's configuration risk sits in a few defaults: unencrypted email, a short password minimum on old clusters, no automatic sign-out, property changes that vanish on update, and backups that were never turned on. Each takes minutes to fix and is expensive to discover during an incident.

**Next in the series:**

| Notebook | Covers |
|----------|--------|
| DTM-10: Managed Cluster Architecture and Health Model | Which cluster events to route on day one |
| DTM-14: Cluster ActiveGates and Mission Control Connectivity | The Mission Control link, proxies and remote-access scopes |
| DTM-16: Backup, Upgrade, and Disaster Recovery | Backup scope, restore and update policy |

**Series map:** DTM-00: The Dynatrace Managed Journey routes every notebook in this series.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official [Dynatrace documentation](https://docs.dynatrace.com/managed).*</sub>
