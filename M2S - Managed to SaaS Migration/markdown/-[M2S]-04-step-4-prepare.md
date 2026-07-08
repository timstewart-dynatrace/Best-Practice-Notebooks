# M2S-04: Step 4 — Prepare: Readiness and Pre-Migration

> **Series:** M2S — Managed to SaaS Migration | **Notebook:** 4 of 9 | **Phase:** Upgrade | **Step:** Prepare | **Created:** March 2026 | **Last Updated:** 07/02/2026

With the target architecture designed, it is time to prepare everything needed for migration execution. This step ensures your SaaS tenant is provisioned, identity is configured, ActiveGates are deployed in parallel, and rollback procedures are tested—so that when you flip the switch in Step 5, there are no surprises.

> **M2S Migration Journey — 3 Phases / 9 Steps**
>
> **Plan:** 1. Discover | 2. Strategize | 3. Design
>
> **Upgrade:** **4. Prepare** | 5. Execute | 6. Integrate
>
> **Run:** 7. Expand | 8. Enable | 9. Optimize
---

## Table of Contents

1. [Licensing and Contract Alignment](#licensing-and-contract-alignment)
2. [SaaS Tenant Provisioning](#saas-tenant-provisioning)
3. [SSO and Identity Setup](#sso-and-identity-setup)
4. [ActiveGate Provisioning](#activegate-provisioning)
5. [SaaS Upgrade Assistant Setup](#saas-upgrade-assistant-setup)
6. [Network Zone Recreation](#network-zone-recreation)
7. [Configuration Freeze](#configuration-freeze)
8. [Rollback Procedure](#rollback-procedure)
9. [Readiness Checklist](#readiness-checklist)

---

## Prerequisites

Before starting this notebook, you should have:

| Requirement | Description |
|-------------|-------------|
| **Steps 1–3 completed** | Discovery inventory, migration strategy, and target architecture designed |
| **Dynatrace account team contact** | For licensing coordination and tenant provisioning |
| **Identity provider access** | Admin access to your SAML IdP (Azure Entra, Okta, etc.) |
| **Infrastructure access** | Ability to deploy new ActiveGate VMs/instances |
| **Network changes approved** | Firewall rules from Design step implemented or in progress |
| **Managed cluster v1.294+** | Required for SaaS Upgrade Assistant compatibility |

---

## Learning Objectives

By the end of this notebook, you will:

- Understand the licensing considerations for a dual-run migration period
- Provision and configure a SaaS tenant in the correct region
- Set up SAML SSO and initial IAM groups and policies
- Deploy SaaS-connected ActiveGates in parallel with Managed AGs
- Install and connect the SaaS Upgrade Assistant
- Recreate network zones in the SaaS tenant
- Implement a configuration freeze on the Managed environment
- Document and test a rollback procedure
- Complete the readiness checklist before proceeding to Execute

---

> **Order of Operations — You Are Here**
>
> This notebook covers operations **1-3** of the 11-step Order of Operations (see **M2S-02: Step 2 — Strategize**) defined in Step 2:
>
> | # | Operation | Status |
> |---|-----------|--------|
> | **1** | **Assess — Inventory current environment** | **This notebook** |
> | **2** | **Provision — SaaS tenant and access (SSO)** | **This notebook** |
> | **3** | **Install — New ActiveGates in parallel** | **This notebook** |
> | 4 | Migrate — Configuration and integrations | Step 5: Execute |
> | 5 | Rebuild — Dashboards, zones, alerts | Step 5: Execute |
> | 6 | Redirect — OneAgents to SaaS | Step 5: Execute |
> | 7 | Reconnect — Integrations and extensions | Step 6: Integrate |
> | 8 | Migrate — Remaining config and integrations | Step 6: Integrate |
> | 9 | Validate — Data flow and performance | Step 9: Optimize |
> | 10 | Cutover — Full switch to SaaS | Step 9: Optimize |
> | 11 | Decommission — Managed environment | Step 9: Optimize |

<a id="licensing-and-contract-alignment"></a>
## 1. Licensing and Contract Alignment

Before any technical work begins, coordinate with your Dynatrace account team to ensure licensing covers the migration period.

### 1.1 Dual-Run Licensing

During migration, both the Managed environment and the SaaS tenant are active simultaneously. This **dual-run period** typically lasts 2–4 weeks, depending on the size and complexity of your environment.

| Consideration | Detail |
|---------------|--------|
| **Overlap period** | Plan for 2–4 weeks of parallel operation |
| **DPS model** | SaaS uses Dynatrace Platform Subscription (DPS) |
| **Managed licensing** | Existing Managed license remains active during migration |
| **Budget impact** | Account for additional cost during overlap |
| **Account team** | Coordinate timing so both licenses are active simultaneously |

> **Important:** Do not let the Managed license expire before all agents are migrated. If the Managed license lapses during migration, OneAgents still connected to Managed will stop reporting.

### 1.2 DPS (Dynatrace Platform Subscription)

SaaS uses the DPS consumption model. Key differences from Managed licensing:

| Aspect | Managed | SaaS (DPS) |
|--------|---------|------------|
| **Billing unit** | Host units (HU) | DPS credits |
| **Consumption model** | Fixed capacity | Consumption-based |
| **Data retention** | Managed by customer | Configurable per data type |
| **Grail storage** | N/A (Managed uses Cassandra) | Included in DPS |

### 1.3 Timeline Coordination

| Milestone | Action |
|-----------|--------|
| T-4 weeks | Confirm dual-run licensing with account team |
| T-2 weeks | SaaS tenant provisioned and accessible |
| T-1 week | All preparation complete (this notebook) |
| T-0 | Begin agent migration (Step 5) |
| T+2–4 weeks | Complete migration, decommission Managed |

---

<a id="saas-tenant-provisioning"></a>
## 2. SaaS Tenant Provisioning

Your Dynatrace account team provisions the SaaS tenant. You must provide key decisions before provisioning begins.

### 2.1 Provisioning Decisions

| Decision | Options | Impact |
|----------|---------|--------|
| **Cloud provider** | AWS or Azure | Determines underlying infrastructure |
| **Data residency region** | US, EU, APAC (specific regions within) | **Cannot change after provisioning** |
| **Azure Native** | Azure Native Dynatrace Service (if applicable) | Simplified billing via Azure Marketplace |

> **Critical:** The data residency region is permanent. Confirm your choice with compliance, legal, and security teams before requesting provisioning. Moving to a different region later requires a new tenant and full re-migration.

### 2.2 Azure Native Dynatrace Service

If your organization uses Azure, consider the Azure Native Dynatrace Service:

| Benefit | Detail |
|---------|--------|
| **Marketplace billing** | Dynatrace costs appear on your Azure invoice |
| **Simplified provisioning** | Create Dynatrace resources directly in Azure portal |
| **Native integration** | Automatic log and metric forwarding from Azure resources |
| **Single sign-on** | Leverage existing Azure Entra ID without separate SAML config |

### 2.3 Initial Tenant Configuration

After the tenant is provisioned, configure these baseline settings:

| Setting | Location | Value |
|---------|----------|-------|
| **Timezone** | Settings > Preferences | Match your primary operations timezone |
| **Currency** | Settings > Preferences | Match your reporting currency |
| **Data retention** | Settings > Data privacy > Data retention | Set per data type (logs, metrics, spans) |
| **Session timeout** | Account Management > Security | Align with your organization's policy |

### 2.4 Verify Tenant Access

Confirm all migration team members can access the new tenant:

| Endpoint | Test |
|----------|------|
| `https://{tenant-id}.live.dynatrace.com` | Browser login succeeds |
| `https://{tenant-id}.apps.dynatrace.com` | Apps platform loads |
| `https://{tenant-id}.live.dynatrace.com/api/v1/time` | API responds with server time |

```bash
# Verify API access
curl -s "https://{tenant-id}.live.dynatrace.com/api/v1/time" \
  -H "Authorization: Api-Token {token}" | python3 -m json.tool
```

---

<a id="sso-and-identity-setup"></a>
## 3. SSO and Identity Setup

SaaS requires SAML 2.0 for single sign-on. If your Managed environment uses LDAP, this is the point where you transition to SAML.

### 3.1 SAML 2.0 Configuration Requirements

| Requirement | Detail |
|-------------|--------|
| **Protocol** | SAML 2.0 |
| **IdP signing** | Must sign the **entire SAML message** (not just the assertion) |
| **Name ID** | Email address format recommended |
| **Attributes** | First name, last name, email, groups |
| **SP Entity ID** | Provided by Dynatrace during SAML setup |
| **ACS URL** | Provided by Dynatrace during SAML setup |

> **Critical:** If the IdP signs only the SAML assertion (not the full message), authentication will fail silently. Azure Entra ID signs the full message by default. For other IdPs (Okta, PingFederate, ADFS), verify this setting explicitly.

### 3.2 Azure Entra ID Configuration

For organizations using Azure Entra ID as their IdP:

| Step | Action |
|------|--------|
| 1 | Create Enterprise Application for Dynatrace SSO |
| 2 | Configure SAML settings with SP Entity ID and ACS URL |
| 3 | Map attributes: UPN, email, first name, last name |
| 4 | **Filter group claims** to Dynatrace-related groups only |
| 5 | Assign users and groups to the application |
| 6 | Test SSO login |

> **Warning:** Azure Entra has a **150-group limit** per SAML token. If a user belongs to more than 150 groups, the token will contain a group overage claim instead of individual group IDs. Always filter group claims to include only Dynatrace-relevant groups.

### 3.3 DNS Domain Validation

Dynatrace requires domain validation for SSO:

| Step | Action |
|------|--------|
| 1 | Add your email domain in Dynatrace Account Management |
| 2 | Add the provided TXT record to your DNS |
| 3 | Wait for DNS propagation (up to 48 hours) |
| 4 | Verify domain in Dynatrace Account Management |

### 3.4 Test SSO Login

Before proceeding, verify SSO works end-to-end:

1. Open an incognito/private browser window
2. Navigate to `https://{tenant-id}.apps.dynatrace.com`
3. Click "Sign in with SSO"
4. Authenticate via your IdP
5. Verify you land in the Dynatrace tenant with correct permissions

> **Tip:** Test with at least one user from each IAM group to confirm group-based policy assignment works correctly.

### 3.5 Initial IAM Groups and Policies

Create the IAM structure designed in Step 3. At minimum:

| Group | Policy | Purpose |
|-------|--------|---------|
| `migration-admins` | Account admin + Environment admin | Migration team full access |
| `platform-admins` | Environment admin | Day-to-day administration |
| `sre-team` | Environment editor | SRE operations access |
| `viewers` | Environment viewer | Read-only stakeholders |

> **Note:** These are starting groups. Refine IAM after migration is complete in Step 6 (Integrate). The goal here is to have enough access for migration activities.

---

<a id="activegate-provisioning"></a>
## 4. ActiveGate Provisioning

Deploy new SaaS-connected ActiveGates **in parallel** with your existing Managed ActiveGates. This ensures continuous monitoring during migration and provides an easy rollback path.

### 4.1 Parallel Deployment Strategy

| Phase | Managed AGs | SaaS AGs | OneAgents Point To |
|-------|-------------|----------|--------------------|
| **Pre-migration (now)** | Running | Deploying | Managed AGs |
| **During migration** | Running | Running | Gradually shifting to SaaS AGs |
| **Post-migration** | Decommissioning | Running | SaaS AGs |

### 4.2 ActiveGate Installation

Install ActiveGates connected to the SaaS tenant. Use the sizing from your Design step.

```bash
# Download ActiveGate installer from SaaS tenant
curl -o Dynatrace-ActiveGate-Linux.sh \
  "https://{tenant-id}.live.dynatrace.com/api/v1/deployment/installer/gateway/unix/latest" \
  -H "Authorization: Api-Token {paas-token}"

# Install with network zone assignment
sudo /bin/sh Dynatrace-ActiveGate-Linux.sh \
  --set-network-zone={zone-name}
```

> **Deprecation note (SaaS 1.343, July 2026):** this classic ActiveGate deployment API (`/api/v1/deployment/installer/gateway/...`) is **deprecated** in favor of the Latest Dynatrace deployment REST API (platform-token support with fine-grained OAuth scopes; GA/enabled-by-default status arrives with the staged SaaS 1.343 tenant rollout from mid-July 2026 — verify in your tenant). The classic endpoint still works during the deprecation period — use it for in-flight migrations, but point new automation at the platform API.

### 4.3 Network Zone Assignment

Assign each AG to the correct network zone during installation:

```bash
# Assign during installation (recommended)
sudo /bin/sh Dynatrace-ActiveGate-Linux.sh --set-network-zone=datacenter-east

# Or assign after installation via custom.properties
# Edit /var/lib/dynatrace/gateway/config/custom.properties
# Add: networkzone = datacenter-east
# Then restart: sudo systemctl restart dynatracegateway
```

### 4.4 ActiveGate Groups

Create AG groups as designed in Step 3:

| Group | Purpose | Configuration |
|-------|---------|---------------|
| `production-routing` | Route production OneAgent traffic | Default group for production zones |
| `nonprod-routing` | Route non-production traffic | Default group for non-production zones |
| `extensions` | Extensions 2.0 execution | Host-based AGs only |
| `synthetic` | Private synthetic monitoring | Dedicated synthetic AGs |

> **Reminder:** Extensions 2.0 require **host-based** ActiveGates. Kubernetes-based ActiveGates do not support Extensions 2.0.

### 4.5 Validate AG Connectivity

After deploying each ActiveGate, verify it appears in the SaaS tenant and is healthy.

```dql
// Verify ActiveGates connected to SaaS tenant
fetch dt.entity.active_gate
| fieldsAdd entity.name, version = toString(softwareVersion)
| fields entity.name, version, networkZone
| sort entity.name asc

// Note: smartscapeNodes ACTIVE_GATE is not yet available on Grail
// Continue using fetch dt.entity.active_gate until Smartscape coverage expands
```

```dql
// Verify ActiveGate distribution across network zones
fetch dt.entity.active_gate
| summarize agCount = count(), by:{networkZone}
| sort agCount desc

// Note: smartscapeNodes ACTIVE_GATE is not yet available on Grail
// Continue using fetch dt.entity.active_gate until Smartscape coverage expands
```

### 4.6 Connectivity Validation Checklist

| Check | Command | Expected Result |
|-------|---------|----------------|
| AG → SaaS | `curl -v https://{tenant-id}.live.dynatrace.com` from AG host | HTTP 200 |
| AG appears in tenant | DQL query above returns the AG | AG listed with correct zone |
| AG version current | Compare version to latest available | Within 1–2 minor versions |
| AG health | Dynatrace UI > Deployment status | Green / Healthy |

---

<a id="saas-upgrade-assistant-setup"></a>
## 5. SaaS Upgrade Assistant Setup

The [SaaS Upgrade Assistant](https://docs.dynatrace.com/managed/upgrade/saas-upgrade-assistant/) is a Dynatrace app that automates configuration migration from Managed to SaaS. Install and connect it now so it is ready for Step 5 (Execute).

### 5.1 Prerequisites

| Requirement | Detail |
|-------------|--------|
| **Managed cluster version** | v1.294 or later |
| **Version alignment** | Same major version on Managed and SaaS recommended |
| **IAM policy** | `upgrade-assistant:environments:write` assigned to migration users |
| **Token scopes** | `Read network zones`, `Write network zones`, `Capture request data` |

### 5.2 Installation Steps

| Step | Action |
|------|--------|
| 1 | Open the Dynatrace Hub in your **SaaS tenant** |
| 2 | Search for "SaaS Upgrade Assistant" |
| 3 | Install the app |
| 4 | Grant the required IAM policy to migration users |
| 5 | Open the app and follow the connection wizard |

### 5.3 Connect to Managed Source

| Step | Action |
|------|--------|
| 1 | In the SaaS Upgrade Assistant, select "Connect Source Environment" |
| 2 | Enter your Managed cluster URL and environment ID |
| 3 | Provide an API token from the Managed environment with `Read configuration` scope |
| 4 | Test the connection |
| 5 | Run the initial configuration scan |

### 5.4 Verify Connection

After connecting, the SaaS Upgrade Assistant should display:

| Item | Expected |
|------|----------|
| **Connection status** | Connected (green) |
| **Configuration count** | Total number of exportable configurations |
| **Environment name** | Your Managed environment name |
| **Version** | Managed cluster version |

> **Tip:** Run the initial scan now but do **not** deploy any configurations yet. Configuration migration happens in Step 5. The goal here is to verify the connection works and review the scope of what will be migrated.

### 5.5 Review Configuration Scope

The SaaS Upgrade Assistant groups configurations by type. Review the counts to validate against your Discovery inventory from Step 1:

| Configuration Type | Expected Count (from Step 1) | SUA Count | Match? |
|--------------------|------------------------------|-----------|--------|
| Management zones | | | [ ] |
| Auto-tagging rules | | | [ ] |
| Dashboards | | | [ ] |
| Alerting profiles | | | [ ] |
| Request attributes | | | [ ] |
| Service detection rules | | | [ ] |
| SLOs | | | [ ] |

---

<a id="network-zone-recreation"></a>
## 6. Network Zone Recreation

Network zones do **not** transfer from Managed to SaaS. You must recreate them in the SaaS tenant before migrating any OneAgents.

### 6.1 Why Zones Must Be Recreated

| Reason | Detail |
|--------|--------|
| **Separate tenants** | Managed and SaaS are independent environments |
| **Zone IDs differ** | SaaS generates new internal IDs |
| **AG assignment** | New SaaS AGs must be assigned to SaaS zones |
| **OneAgent routing** | Agents need zones defined before they can route correctly |

### 6.2 Create Zones in SaaS

Recreate each zone from your Design step. Use the Settings UI or API:

**Via Settings UI:**

Settings > Network zones > Add network zone

> **Sprint 1.339 deprecation (May 2026):** The dedicated `/api/v2/networkZones` Configuration API endpoint is deprecated in favor of the Settings 2.0 schema `builtin:networkzones.zones`. Prefer the Settings 2.0 form below for new automation; the legacy endpoint still works during the deprecation window.

**Via Settings 2.0 API (recommended):**

```bash
# Create a network zone via Settings 2.0
curl -X POST "https://{tenant-id}.live.dynatrace.com/api/v2/settings/objects" \
  -H "Authorization: Api-Token {token}" \
  -H "Content-Type: application/json" \
  -d '[{
    "schemaId": "builtin:networkzones.zones",
    "scope": "environment",
    "value": {
      "name": "datacenter-east",
      "description": "Primary datacenter in East region",
      "alternativeZones": ["datacenter-west"]
    }
  }]'
```

**Via legacy Configuration API (deprecated, still functional during deprecation window):**

```bash
# Create a network zone
curl -X POST "https://{tenant-id}.live.dynatrace.com/api/v2/networkZones" \
  -H "Authorization: Api-Token {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "id": "datacenter-east",
    "description": "Primary datacenter in East region",
    "alternativeZones": ["datacenter-west"]
  }'
```

### 6.3 Assign ActiveGates to Zones

If you deployed AGs with `--set-network-zone` during installation (Section 4), they are already assigned. Otherwise, update the assignment:

```bash
# Update AG network zone after installation
# Edit /var/lib/dynatrace/gateway/config/custom.properties
# Add: networkzone = datacenter-east
sudo systemctl restart dynatracegateway
```

### 6.4 Verify Zone Topology

```dql
// Verify network zone configuration and AG assignment
fetch dt.entity.active_gate
| fieldsAdd entity.name, zone = networkZone
| fields entity.name, zone
| sort zone asc, entity.name asc

// Note: smartscapeNodes ACTIVE_GATE is not yet available on Grail
// Continue using fetch dt.entity.active_gate until Smartscape coverage expands
```

### 6.5 Zone Topology Validation

Compare the SaaS zone topology against your Design step:

| Zone Name | Alternative Zone | AGs Assigned | Matches Design? |
|-----------|------------------|--------------|-----------------|
| | | | [ ] |
| | | | [ ] |
| | | | [ ] |
| | | | [ ] |

> **Important:** Every network zone must have at least 2 ActiveGates for high availability. Verify this before proceeding.

---

<a id="configuration-freeze"></a>
## 7. Configuration Freeze

Announce and enforce a configuration freeze on the Managed environment. This prevents configuration drift between the source (Managed) and target (SaaS) during migration.

### 7.1 What to Freeze

| Category | Freeze Scope |
|----------|-------------|
| **Dashboards** | No new dashboards or modifications to existing ones |
| **Alerting** | No new alerting profiles, metric events, or notification rules |
| **Settings** | No changes to management zones, auto-tagging, or service detection |
| **Monitoring config** | No changes to OneAgent features, deep monitoring, or anomaly detection |
| **RUM/Mobile** | No new web or mobile application definitions |

### 7.2 Exception Process

Some changes may be unavoidable during the freeze period:

| Exception Type | Approval Required | Documentation |
|----------------|-------------------|---------------|
| **Critical fix** | Migration lead | Document change and apply to both Managed and SaaS |
| **Security patch** | Security team + Migration lead | Apply to both environments |
| **New deployment** | Migration lead | Ensure monitoring config exists in both environments |

### 7.3 Communication Template

Send to all Dynatrace users and configuration owners:

> **Subject:** Dynatrace Configuration Freeze — Effective {date}
>
> As part of the Managed-to-SaaS migration, a configuration freeze is now in effect on the Managed environment. **Do not create or modify** dashboards, alerting rules, management zones, or any other configuration in the Managed tenant.
>
> All configuration changes during this period require approval from the migration lead. Contact {migration-lead-email} for exception requests.
>
> The freeze will remain in effect until the migration is complete (estimated {end-date}).

### 7.4 Freeze Verification

Monitor for unauthorized changes during the freeze period:

```dql
// Monitor for configuration changes during freeze period
// Run this on the Managed environment to detect unauthorized changes
fetch events, from:-24h
| filter event.kind == "CONFIG"
| fields timestamp, event.type, user = dt.user, description = event.description
| sort timestamp desc
| limit 50
```

---

<a id="rollback-procedure"></a>
## 8. Rollback Procedure

Before executing the migration, document and test a rollback procedure. If issues arise during Step 5, you need to be able to revert OneAgents back to Managed quickly.

### 8.1 Rollback Approach

Rollback redirects OneAgents from the SaaS tenant back to the Managed environment:

| Component | Rollback Action |
|-----------|----------------|
| **OneAgent** | Redirect to Managed server URL |
| **ActiveGate** | No change needed (Managed AGs still running during dual-run) |
| **Configuration** | No change needed (Managed config is frozen, not deleted) |
| **Data** | Managed retains all historical data until decommissioned |

### 8.2 OneAgent Rollback Command

The key rollback command redirects the OneAgent back to Managed:

```bash
# Linux rollback
sudo /opt/dynatrace/oneagent/agent/tools/oneagentctl \
  --set-server="https://{managed-cluster}/communication"

# Windows rollback
"C:\Program Files\dynatrace\oneagent\agent\tools\oneagentctl.exe" \
  --set-server="https://{managed-cluster}/communication"
```

### 8.3 Pre-Migration Rollback Test

Test rollback on a non-production host **before** the production migration:

| Step | Action | Validation |
|------|--------|------------|
| 1 | Select a non-production host currently reporting to Managed | Host visible in Managed UI |
| 2 | Redirect OneAgent to SaaS: `oneagentctl --set-server="https://{tenant-id}.live.dynatrace.com/communication"` | Host appears in SaaS UI within 5 minutes |
| 3 | Verify monitoring data flows to SaaS | Metrics, logs, spans visible in SaaS |
| 4 | Execute rollback: `oneagentctl --set-server="https://{managed-cluster}/communication"` | Host reappears in Managed UI |
| 5 | Verify monitoring data resumes in Managed | Metrics, logs, spans visible in Managed |

> **Important:** Do not proceed to Step 5 until you have successfully tested rollback on at least one host. This validates that the Managed environment is still fully operational and ready to accept agents back if needed.

### 8.4 Document Rollback Details

Record these values for use during migration:

| Parameter | Value |
|-----------|-------|
| **Managed server URL** | `https://{managed-cluster}/communication` |
| **Managed tenant token** | (retrieve from Managed UI > Deployment > OneAgent) |
| **SaaS server URL** | `https://{tenant-id}.live.dynatrace.com/communication` |
| **SaaS tenant token** | (retrieve from SaaS UI > Deployment > OneAgent) |
| **Rollback tested on** | {hostname}, {date} |
| **Rollback time** | ~5 minutes for agent to reconnect |

---

<a id="readiness-checklist"></a>
## 9. Readiness Checklist

Complete every checkpoint before proceeding to Step 5 (Execute). Each item corresponds to a section in this notebook.

### Licensing

| Checkpoint | Status |
|------------|--------|
| Dual-run licensing confirmed with Dynatrace account team | [ ] |
| DPS consumption model understood and budgeted | [ ] |
| Migration timeline communicated to account team | [ ] |

### SaaS Tenant

| Checkpoint | Status |
|------------|--------|
| SaaS tenant provisioned in correct data residency region | [ ] |
| Tenant accessible at `{tenant-id}.live.dynatrace.com` | [ ] |
| Tenant accessible at `{tenant-id}.apps.dynatrace.com` | [ ] |
| Baseline settings configured (timezone, currency, retention) | [ ] |
| API access verified | [ ] |

### SSO and Identity

| Checkpoint | Status |
|------------|--------|
| SAML SSO configured and tested | [ ] |
| IdP signs full SAML message (not just assertion) | [ ] |
| Azure Entra group claims filtered (if applicable) | [ ] |
| DNS domain validated | [ ] |
| Initial IAM groups and policies created | [ ] |
| SSO tested with users from each IAM group | [ ] |

### ActiveGates

| Checkpoint | Status |
|------------|--------|
| New SaaS-connected AGs deployed per Design step sizing | [ ] |
| Each AG assigned to correct network zone | [ ] |
| Minimum 2 AGs per network zone for HA | [ ] |
| AG groups created (routing, extensions, synthetic) | [ ] |
| AG connectivity to SaaS validated | [ ] |
| AG health confirmed in Dynatrace UI | [ ] |

### SaaS Upgrade Assistant

| Checkpoint | Status |
|------------|--------|
| App installed in SaaS tenant | [ ] |
| `upgrade-assistant:environments:write` IAM policy granted | [ ] |
| Connected to Managed source environment | [ ] |
| Initial configuration scan completed | [ ] |
| Configuration counts match Discovery inventory | [ ] |

### Network Zones

| Checkpoint | Status |
|------------|--------|
| All zones recreated in SaaS | [ ] |
| Alternative zones configured for failover | [ ] |
| AGs assigned to correct zones | [ ] |
| Zone topology matches Design step | [ ] |

### Configuration Freeze

| Checkpoint | Status |
|------------|--------|
| Configuration freeze announced to all stakeholders | [ ] |
| Exception process documented and communicated | [ ] |
| Monitoring for unauthorized changes in place | [ ] |

### Rollback

| Checkpoint | Status |
|------------|--------|
| Managed server URL and tenant token documented | [ ] |
| SaaS server URL and tenant token documented | [ ] |
| Rollback procedure documented | [ ] |
| Rollback tested on non-production host | [ ] |
| Rollback test successful (agent reconnected to Managed) | [ ] |

### Communication

| Checkpoint | Status |
|------------|--------|
| Migration schedule communicated to stakeholders | [ ] |
| Support team briefed on rollback procedure | [ ] |
| Escalation contacts identified for migration window | [ ] |

---

## Next Step

> **→ M2S-05: Step 5 — Execute** — Migrate configurations using the SaaS Upgrade Assistant and redirect OneAgents from Managed to SaaS.

### Continue the Series

| Next Notebook | Focus |
|---------------|-------|
| **M2S-05: Step 5 — Execute** | Configuration migration and agent redirection |

### Preparation Resources

- [SaaS Upgrade Assistant Documentation](https://docs.dynatrace.com/managed/upgrade/saas-upgrade-assistant/)
- [SaaS Upgrade Assistant on Dynatrace Hub](https://www.dynatrace.com/hub/detail/saas-upgrade-assistant/)
- [ActiveGate Installation](https://docs.dynatrace.com/docs/ingest-from/dynatrace-activegate/installation)
- [Network Zones Configuration](https://docs.dynatrace.com/docs/manage/network-zones)
- [SAML SSO Setup](https://docs.dynatrace.com/docs/manage/identity-access-management/single-sign-on)
- [IAM Documentation](https://docs.dynatrace.com/docs/manage/identity-access-management)
- [OneAgent Communication](https://docs.dynatrace.com/docs/setup-and-configuration/dynatrace-oneagent/oneagent-configuration/network-connectivity)
- [Azure Native Dynatrace Service](https://docs.dynatrace.com/docs/setup-and-configuration/setup-on-cloud-platforms/microsoft-azure-services/azure-native-integration)

---

## Summary

In this notebook, you prepared:

- **Licensing** — Dual-run licensing coordinated with Dynatrace account team for the overlap period
- **SaaS tenant** — Provisioned in the correct data residency region with baseline configuration
- **SSO and identity** — SAML 2.0 configured, DNS domain validated, initial IAM groups created
- **ActiveGates** — New SaaS-connected AGs deployed in parallel, assigned to network zones
- **SaaS Upgrade Assistant** — Installed, connected to Managed, initial scan completed
- **Network zones** — All zones recreated in SaaS matching the Design step topology
- **Configuration freeze** — Announced and enforced on the Managed environment
- **Rollback** — Procedure documented and tested on a non-production host

> **Key Takeaway:** Preparation is the most critical step in a smooth migration. Every item in the readiness checklist reduces risk during execution. Do not proceed to Step 5 until every checkbox is complete—especially the rollback test. A tested rollback gives confidence to move forward.

---

*Continue to **M2S-05: Step 5 — Execute** to begin migrating configurations and redirecting agents.*

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
