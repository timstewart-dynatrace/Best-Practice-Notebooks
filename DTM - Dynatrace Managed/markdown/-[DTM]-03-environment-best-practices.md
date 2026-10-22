# DTM-03: Environment Best Practices

> **Series:** DTM — Dynatrace Managed | **Reference:** 03 — Environment Best Practices | **Created:** October 2026 | **Last Updated:** 10/21/2026

> **Dynatrace Classic only.** This series applies to **Dynatrace Managed**, the self-hosted, on-premises deployment. None of it applies to Dynatrace SaaS.

## Overview

A Managed cluster hosts one or more **monitoring environments**, and it decides who may sign in to each. Those two choices — how the cluster is divided, and how people get in — are made once and lived with for years. This notebook covers when to create a separate environment and when a management zone is enough, how to set an environment's limits, how to choose an identity source, and how to keep permissions narrow.

**What you'll learn:**

- When an environment is the right boundary, and when a management zone is
- Which environment limits to set deliberately
- How LDAP, SAML and OpenID Connect change who can sign in
- Why the installation administrator account needs special care
- How cluster, environment and management-zone permissions combine
- How IAM policies interact with classic role permissions on Managed

---

## Table of Contents

1. [Short Answer](#short-answer)
2. [Environment or Management Zone?](#environment-or-zone)
3. [Set Each Environment's Limits Deliberately](#environment-limits)
4. [Choose an Identity Source](#identity-source)
5. [Protect the Installation Administrator Account](#admin-account)
6. [Keep Permissions Narrow](#permissions)
7. [IAM Policies on Managed](#iam-policies)
8. [What the Documentation Does Not Say](#doc-gaps)
9. [Recommended Approach](#recommendation)
10. [Summary and Next Steps](#summary)

---

## Prerequisites

| Requirement | Details |
|-------------|---------|
| **Dynatrace deployment** | A Dynatrace Managed cluster |
| **Access** | Cluster Management Console (CMC) cluster administrator |
| **Query language** | None. Managed has no Grail, so this series has **no DQL cells** |
| **Version basis** | Written against the Managed documentation read 10/21/2026 |

<a id="short-answer"></a>
## 1. Short Answer

Use a **separate environment** where you need separate limits, retention or settings, and **management zones** for team views inside one environment. Set every environment's **quotas and retention** on purpose. Sign users in through **LDAP, SAML or OpenID Connect**, map groups from the identity provider, and remember that **every user needs a group with at least one environment** to sign in. Treat the **installation administrator account** as a break-glass account. Keep the **cluster-permission group small**, grant sensitive permissions narrowly, and grant restricted access at the **management-zone** level, because environment-level permissions override it.

<a id="environment-or-zone"></a>
## 2. Environment or Management Zone?

The two are different kinds of boundary.

| | Environment | Management zone |
|--|-------------|-----------------|
| **What it is** | A monitoring environment in the cluster, created and configured in the CMC | A partition *within* an environment: *"Each customizable management zone comprises a set of monitored entities or dimensional data (such as logs and metrics) in your environment"* |
| **Limits and retention** | Its own quotas, storage and retention settings (§3) | None of its own; it shares the environment's |
| **Access** | Permissions assigned per environment to user groups (§6) | Permissions assigned per zone; *"Users may be granted access to entire environments, a specific management zone, or a subset of related management zones."* |
| **Cross-team problems** | — | *"when a problem spans multiple management zones, users see a full end-to-end view of the problem"*, limited to what they may see |

**Best practice:** create a separate environment when the separation must be hard — different quotas, different retention, different settings, or data that must never be visible together (for example production and non-production). Use management zones when teams share one estate and need their own views of it. In community practice, services that call each other are kept in the same environment, so their traces and problems stay connected — confirm against how your applications are split.

**On SaaS**, management zones are replaced by segments and IAM policies; the MZ2POL series covers that migration. On Managed, management zones remain the access mechanism inside an environment.

> <sub>**Sources:** [Management zones (Dynatrace Classic) (DT docs)](https://docs.dynatrace.com/managed/manage/identity-access-management-classic/permission-management/management-zones) — *"Management zones may overlap, just as team responsibilities might overlap."*, [Manage your monitoring environments (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/manage-your-monitoring-environments).</sub>

<a id="environment-limits"></a>
## 3. Set Each Environment's Limits Deliberately

Each environment's configuration page in the CMC (**Environments** → the environment) holds its limits:

| Kind | Settings |
|------|----------|
| **Total quotas** | Host units, custom metrics |
| **Monthly and annual quotas** | User sessions, synthetic monitors, average daily log volume |
| **Storage and retention** | Transaction storage, mobile symbol files, service request and code-level retention, Real User Monitoring retention, synthetic retention, Session Replay retention, Log Monitoring storage and retention |

They are hard limits: *"Once any one of these limits is consumed, the respective monitoring will no longer be available and you'll need to purchase more monitoring."* So set them from the cluster's capacity and the environment's purpose, not by leaving the defaults. A non-production environment rarely needs production retention.

**Two related decisions:**

- **Synthetic monitors** can be switched off when the environment is created: *"You can disable synthetic monitors by setting the Enable synthetic monitors switch to the off position."*
- **Overload prevention and the ingest limit** are on the same page and are covered in DTM-15 §6. Revisit them after every hardware change.

> <sub>**Sources:** [Manage your monitoring environments (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/manage-your-monitoring-environments) — *"On the configuration page, you can set the total as well as the monthly and annual quotas for your environment."*</sub>

<a id="identity-source"></a>
## 4. Choose an Identity Source

The cluster can authenticate users from its internal database, an LDAP directory, or a single sign-on provider over SAML 2.0 or OpenID Connect. DTM-02 §6 covers the password policy for internal accounts. What changes when you connect a directory:

**LDAP** (**User authentication > User repository**):

- **Local accounts stop working.** *"Local accounts (other than the administrator account) will stop working: it will be impossible to log in with a local account."*
- **Encrypt the connection.** *"To enable encrypted communication with the LDAP server, select Use SSL ."* The standard secure port is 636.
- **Use a read-only system account to bind.** *"Bind DN typically is a system user (not an actual person) used to connect to the LDAP server."* It *"does not need write access"*.
- **Groups are yours to create; users appear on sign-in.** *"Groups needs to be created manually by the admin. Users are created automatically, but only after a successful authentication attempt."*

**SAML 2.0 and OpenID Connect** (**User authentication > Single sign-on settings**):

- **Choose the login page.** *Standard + SSO* keeps the local sign-in alongside SSO. *SSO* *"skips the Dynatrace login page, so the user cannot sign in using a local user account"*. Decide which local accounts you still need before choosing *SSO* (§5).
- **Map groups from the identity provider, and spell them exactly.** With automatic assignment, *"any assignments made within Dynatrace Managed are overwritten by the list of groups sent in your IdP's authentication response"*. For SAML, *"Make sure group names exactly match existing Dynatrace user group names (case-sensitive, no extra spaces)."*
- **Every user needs a group with an environment.** *"Each Dynatrace Managed user must be assigned to at least one user group, with at least one associated monitoring environment ."* Without that, the user *"can't sign in to Dynatrace Managed"*.
- **OpenID Connect needs one redirect URI per environment, or a wildcard.** *"Configure one URI per environment or use a wildcard ( https://{dynatrace-server}/e/* ) to match all environment URIs."* A new environment without one can't be reached through SSO.

> <sub>**Sources:**</sub>
> - <sub>[Manage users and groups via LDAP (DT docs)](https://docs.dynatrace.com/managed/manage/identity-access-management/user-and-group-management/manage-users-and-groups-with-ldap)</sub>
> - <sub>[Manage users and groups with SAML in Dynatrace Managed (DT docs)](https://docs.dynatrace.com/managed/manage/identity-access-management/user-and-group-management/manage-users-and-groups-with-saml) — *"SAML can use either HTTP POST (preferred) or HTTP Redirect bindings."*</sub>
> - <sub>[Manage users and groups with OpenID in Dynatrace Managed (DT docs)](https://docs.dynatrace.com/managed/manage/identity-access-management/user-and-group-management/manage-users-and-groups-with-openid)</sub>

<a id="admin-account"></a>
## 5. Protect the Installation Administrator Account

One account outlives every identity decision. *"A default administrator account is created during Dynatrace Managed installation. This account exists regardless of the authentication type you select (internal or LDAP)."* The LDAP page says the same: *"The administrator account you created during installation will continue to work regardless of the selected authentication provider."* It *"has cluster permissions"*.

That makes it both the way back in when the directory or identity provider fails, and a standing privileged credential. Best practice:

- Give it a long, unique password that meets the policy in DTM-02 §6, stored where your organization keeps break-glass credentials.
- Don't use it for day-to-day administration; sign in through your identity source instead.
- If you choose the *SSO* login page (§4), confirm how the administrator account would sign in during an identity-provider outage before you rely on it. The docs don't describe that path (§8).

> <sub>**Sources:** [User groups, permissions, and policies (DT docs)](https://docs.dynatrace.com/managed/manage/identity-access-management/user-and-group-management/user-groups-and-permissions), [Manage users and groups via LDAP (DT docs)](https://docs.dynatrace.com/managed/manage/identity-access-management/user-and-group-management/manage-users-and-groups-with-ldap).</sub>

<a id="permissions"></a>
## 6. Keep Permissions Narrow

Permissions are granted to **groups**, at three levels.

**Cluster permissions — keep this group small.** *"Users assigned to groups with this permission are automatically given administrator access rights for all environments."* They can also *"Add new Dynatrace Server nodes"*, *"Upgrade Dynatrace Server"*, *"Manage Dynatrace Managed users and user groups"*, and *"Configure monitoring settings for any monitoring environment"*.

**Environment permissions — grant the sensitive ones narrowly.** *View environment* is the base; the others add to it. Three deserve particular care:

| Permission | Why |
|------------|-----|
| *View logs* | *"Allows access to sensitive log file data in the Logs tab."* It also *"bypasses any existing conditional access you may have defined in policy boundaries through the storage logs permission"* |
| *View sensitive request data* | *"Allows viewing of potentially personal data captured by Dynatrace , including downloading memory dumps."* |
| *Replay session data without masking* | Replays recorded user sessions without playback masking |

**Management-zone permissions — use them to restrict, because the environment level wins.** *"Any permission you provide at the environment level supersedes and adds to those at the management-zone level. In other words, management-zone permissions cannot be used to limit permissions already provided at the environment level."* So a group that should see only part of an environment gets its permissions on the zone, not on the environment. The docs recommend it directly: *"If you wish to provide a permission to users accessing a management zone, we recommend that you use the management-zone-level permissions ."*

**Group membership adds up.** *"Users can belong to more than one group and inherit the permissions of the groups that they belong to."* Review a user's effective access across all their groups, not one group at a time.

> <sub>**Sources:** [User groups, permissions, and policies (DT docs)](https://docs.dynatrace.com/managed/manage/identity-access-management/user-and-group-management/user-groups-and-permissions) — *"A group cannot be assigned if no permissions are specified for the group."*</sub>

<a id="iam-policies"></a>
## 7. IAM Policies on Managed

Managed also supports IAM policies alongside the role permissions: *"Starting with Dynatrace version 1.252+, Dynatrace supports defining user and group permissions using IAM policies alongside classic role-based access control."* Policies can scope a role to management zones, for example:

```text
ALLOW environment:roles:viewer WHERE environment:management-zone IN ("prod");
```

**They add; they don't subtract.** *"Policies do not restrict classic role-based permissions"*: *"role-based permissions and security policies are additive."* A user who still holds an environment permission through a classic group keeps access to the whole environment, whatever their policies say. When moving a group to policies, remove its classic environment permissions in the same change, or the narrower policy has no effect.

> <sub>**Sources:** [Upgrade role-based permissions to Dynatrace IAM policies (Dynatrace Classic) (DT docs)](https://docs.dynatrace.com/managed/manage/identity-access-management-classic/permission-management/manage-user-permissions-policies/advanced/migrate-roles-classic) — *"the environment permission still grants access to a whole environment, including all management zones."*</sub>

<a id="doc-gaps"></a>
## 8. What the Documentation Does Not Say

**Not found**, in the Managed documentation read 10/21/2026:

| Gap | Working assumption in this notebook |
|-----|-------------------------------------|
| A recommended number of environments, or criteria for splitting one | Split where quotas, retention, settings or visibility must differ (§2) |
| How the administrator account signs in when the login page is set to *SSO* and the identity provider is down | Confirm the recovery path before choosing *SSO* (§5) |
| Default values for an environment's quotas and retention | Set every one deliberately (§3) |

**Old pages.** The environment, user-group, LDAP, SAML and OpenID pages carry publication dates from 2017 and 2018 and no later update date. The IAM policy page was updated 10/08/2026. Check the CMC itself where a page and the console disagree.

> <sub>**Sources:** [Manage your monitoring environments (DT docs)](https://docs.dynatrace.com/managed/managed-cluster/operation/manage-your-monitoring-environments), [User groups, permissions, and policies (DT docs)](https://docs.dynatrace.com/managed/manage/identity-access-management/user-and-group-management/user-groups-and-permissions), [Upgrade role-based permissions to Dynatrace IAM policies (Dynatrace Classic) (DT docs)](https://docs.dynatrace.com/managed/manage/identity-access-management-classic/permission-management/manage-user-permissions-policies/advanced/migrate-roles-classic).</sub>

<a id="recommendation"></a>
## 9. Recommended Approach

1. **Draw the environment boundaries first**: separate environments where limits, retention, settings or visibility must differ; management zones for team views inside one (§2).
2. **Set each environment's quotas and retention on purpose**, and switch off synthetic monitors where they aren't wanted (§3).
3. **Connect LDAP, SAML or OpenID Connect.** Use SSL for LDAP and a read-only bind account; map groups automatically from the identity provider with exact names; give every user a group with at least one environment (§4).
4. **Store the installation administrator account as a break-glass credential** and keep it out of daily use (§5).
5. **Keep the cluster-permission group small**, grant *View logs*, *View sensitive request data* and unmasked replay narrowly, and restrict access on management zones rather than environments (§6).
6. **When adopting IAM policies, remove the classic environment permissions they replace** in the same change (§7).
7. **Review effective access per user** across all groups at least once a quarter (community practice).

<a id="summary"></a>
## 10. Summary and Next Steps

Environments are the hard boundary in a Managed cluster: their own limits, retention and settings. Management zones are the soft one, for team views inside an environment. Access comes from groups at three levels, and the broadest level always wins, so restriction has to be granted at the narrowest level and classic permissions have to go when policies replace them.

**Next in the series:**

| Notebook | Covers |
|----------|--------|
| DTM-02: Cluster Configuration Best Practices | Password, session and sign-out policy |
| DTM-15: Capacity and Scaling | Environment quotas, overload prevention and the ingest limit |

**Series map:** DTM-00: The Dynatrace Managed Journey routes every notebook in this series.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official [Dynatrace documentation](https://docs.dynatrace.com/managed).*</sub>
