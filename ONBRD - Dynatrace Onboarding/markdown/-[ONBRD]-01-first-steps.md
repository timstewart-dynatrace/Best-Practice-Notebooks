# ONBRD-01: Getting Started: Your First Steps in Dynatrace

> **Series:** ONBRD — Dynatrace Onboarding | **Notebook:** 1 of 10 | **Created:** December 2025 | **Last Updated:** 10/02/2026

## Finding Your Way Around
Welcome to Dynatrace. This notebook helps you get oriented in your new environment—where to find things, how to navigate, and what to do first.

---

## Table of Contents

1. [Accessing Your Environment](#accessing-your-environment)
2. [Understanding the Navigation](#understanding-the-navigation)
3. [Key Areas to Know](#key-areas-to-know)
4. [Your Environment ID and URLs](#your-environment-id-and-urls)
5. [Checking What's Already There](#checking-whats-already-there)
6. [Next Steps](#next-steps)

---

## Prerequisites

- Access credentials for your Dynatrace tenant
- A modern web browser (Chrome, Firefox, Edge, Safari)

### 2026 Defaults for New Customers

Three platform changes establish what "new customer onboarding" should default to in 2026:

1. **Platform tokens (`dt0s16`) with `Authorization: Bearer …`** — recommended default for all new automation. Dynatrace's upgrade guide states that for classic access tokens, *"In Latest Dynatrace, this model is replaced with platform tokens"*. Classic `dt0c01` (`Authorization: Api-Token …`) still works on classic and hybrid environments for legacy paths but should not be the default for new pipelines. Wrong scheme returns `401 Unsupported authorization scheme` even when scopes are correct (covered in ONBRD-02 IAM and Authentication).
2. **Settings v2 (Environment API v2)** — new automation should target Settings v2 paths. SaaS 1.337 (April 2026) put deprecation notices on many Configuration API endpoints that Settings now covers. Plan onboarding tooling around Settings v2 (Terraform's per-schema `dynatrace_*` settings resources, Monaco v2 `settings` configs). See ONBRD-06 (Organizing Your Environment) and ONBRD-09 (Setting Up Alerts) for the schema-id patterns to use.
3. **Extensions 2.0** (managed in the **Extensions** app) — **the current framework for customers adopting custom Extensions.** Extensions Framework 1.0 reached end of support on 2025-03-31 (Python EF1.0: 2024-10-31); JMX and PMI EF1.0 are deprecated, supported past March 2025 on contacting Dynatrace, and reach end of support on **July 1, 2027** — advocate migrating any remaining EF1.0 extensions in onboarding conversations.

Also: **OneAgent primary fields/tags at the source** (OneAgent 1.333+, Latest Dynatrace) means new customers should be told to design their tag taxonomy with primary tags first-class — set during OneAgent install via `oneagentctl --set-host-tag="primary_tags.<key>=<value>"` — the `primary_tags.` prefix must be written explicitly. Covered in ONBRD-05 (Deploying OneAgent) and ONBRD-06 (Organizing Your Environment).

> <sub>**Sources:**</sub>
> - <sub>[Upgrade from access tokens classic (DT docs)](https://docs.dynatrace.com/docs/platform/upgrade/set-up-your-environment/upgrade-from-access-tokens-classic) — *"Classic access tokens don't exist in latest environments, and v2/apiTokens isn't available."*</sub>
> - <sub>[SaaS 1.337 release notes (DT docs)](https://docs.dynatrace.com/docs/whats-new/saas/sprint-337) — *"Many Configuration API endpoints are now covered by the Settings endpoints in the Environment API v2."*</sub>
> - <sub>[End-of-support news (DT docs)](https://docs.dynatrace.com/docs/whats-new/technology/end-of-support-news) — *"JMX and PMI EF1.0 will reach End of Support on July 1, 2027."*</sub>
> - <sub>[Manage extensions (DT docs)](https://docs.dynatrace.com/docs/ingest-from/extensions/manage-extensions), [Terraform provider resources (Dynatrace GitHub)](https://github.com/dynatrace-oss/terraform-provider-dynatrace/tree/main/docs/resources), [Primary Grail fields and tags enrichment (DT docs)](https://docs.dynatrace.com/docs/ingest-from/dynatrace-oneagent/oneagent-attribute-enrichment)</sub>

---

<a id="accessing-your-environment"></a>
## 1. Accessing Your Environment
Your Dynatrace environment is accessed via a URL specific to your tenant:

```
https://{tenant-id}.apps.dynatrace.com
```

### Finding Your Tenant ID

Your tenant ID is the first part of your Dynatrace URL. For example:
- URL: `https://abc12345.apps.dynatrace.com`
- Tenant ID: `abc12345`

**Write down your tenant ID**—you'll need it for API tokens, OneAgent deployment, and integrations.

### First Login

1. Navigate to your tenant URL
2. Enter your credentials (local user or SSO)
3. Complete any MFA requirements
4. You'll land on the default home screen

> **Note:** If your organization is setting up SAML/SSO, see **ONBRD-02: IAM and Authentication** before inviting additional users.

> **New tenant or trial?** QuickStart → **Add data** opens Auto Discovery — *"We recommend Auto Discovery, the default when you select Add data"* — where a CLI (`dtwiz`) *"Recommends the optimal ingestion method for your environment (ranked suggestions)"* and deploys it ([QuickStart guide (DT docs)](https://docs.dynatrace.com/docs/discover-dynatrace/get-started/quickstart-guide)). For production rollouts, follow ONBRD-03 to ONBRD-05.

<a id="understanding-the-navigation"></a>
## 2. Understanding the Navigation
Dynatrace uses a left-hand navigation menu organized by function. The platform is built around **Apps**—each capability is an app you can launch.

![Navigation Structure](images/01-navigation-structure.png)
<!-- MARKDOWN_TABLE_ALTERNATIVE
| Area | Description |
|------|-------------|
| Search (Cmd+K) | Quick access to anything |
| Apps Menu | Platform capabilities (Dashboards, Notebooks, Problems, etc.) |
| Main Content | Workspace for analysis and monitoring |
| Settings | Configuration options |
| Account Management | Users, tokens, permissions |
-->

### Quick Search

Press **Cmd+K** (Mac) or **Ctrl+K** (Windows/Linux) to open the quick search. Type anything:
- Entity names (hosts, services)
- App names
- Keywords like "logs" or "problems"

### The App Launcher

Click the grid icon to see all available apps. You can:
- Pin frequently used apps to your sidebar
- Discover new apps in the Dynatrace Hub
- Install apps from the Hub to extend functionality

<a id="key-areas-to-know"></a>
## 3. Key Areas to Know
### Infrastructure & Operations App

The Infrastructure & Operations app has an Explorer for hosts, containers, processes, network devices and technologies:
- Host health and resource utilization
- Running processes and services
- Host properties and metadata

### Problems App

Dynatrace Intelligence (Davis) automatically detects problems and correlates related events. This is where you'll see:
- Active issues requiring attention
- Root cause analysis
- Affected entities
- Problem timeline and resolution

### Logs App

Explore all log data ingested into Dynatrace (the Logs app replaces the classic Logs and Events screen):
- Full-text search across logs
- Filter by source, severity, content
- Correlate logs with traces and metrics

### Notebooks App

Interactive analysis environment (you're using one now!):
- Write and execute DQL queries
- Document investigations
- Share findings with your team

### Workflows App

Automation and alerting for the modern platform:
- Create automated responses to problems
- Configure notifications (Slack, email, PagerDuty, etc.)
- Build custom automation logic

> <sub>**Sources:** [Infrastructure & Operations (DT docs)](https://docs.dynatrace.com/docs/observe/infrastructure-observability/infrastructure-and-operations), [Logs app (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/logs/lma-logs-app) — *"The application replaces the Logs and Events screen"*.</sub>

<a id="your-environment-id-and-urls"></a>
## 4. Your Environment ID and URLs
Several URLs are important to bookmark:

| Purpose | URL Pattern |
|---------|-------------|
| **Main UI** | `https://{tenant-id}.apps.dynatrace.com` |
| **Platform API** | `https://{tenant-id}.apps.dynatrace.com/platform/` |
| **Account Management** | `https://myaccount.dynatrace.com` |

### API Access

Modern Dynatrace platform access uses three credential types — choose based on the integration:

| Token Type | Prefix | When to Use |
|------------|--------|-------------|
| **Platform Token** *(recommended default)* | `dt0s16` | New automation, MCP integrations, OpenPipeline configuration, installer downloads |
| **OAuth Client** | (client ID + secret) | External SaaS integrations, account-admin automation |
| **Classic API Token** *(legacy phase-out)* | `dt0c01` | Existing scripts; migrate to Platform Token where possible |

Workflows don't use a token: each workflow runs as its **actor** (a user or service user), so grant permissions to the actor.

For OneAgent and ActiveGate deployment, installer downloads work with a **platform token** carrying `fleet-management:oneagents:download` (OneAgent) or `fleet-management:activegates:download` (ActiveGate). On classic and hybrid environments, a classic access token with the `InstallerDownload` scope (from the **Access Tokens** app) also works; latest environments have no classic access tokens. Token management depth lives in **ONBRD-02**.

> <sub>**Sources:** [Download latest OneAgent installer (DT docs)](https://docs.dynatrace.com/docs/dynatrace-api/environment-api/deployment/oneagent/download-oneagent-latest) — *"Platform Token / OAuth: Required scope: fleet-management:oneagents:download"*; [Workflow security (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/workflows/security) — *"Every execution of a workflow task is performed in the context of a user."*; [Account Management (DT docs)](https://docs.dynatrace.com/docs/manage/account-management).</sub>

<a id="checking-whats-already-there"></a>
## 5. Checking What's Already There
Before deploying OneAgent, check if any data is already flowing. Run these queries to see what exists in your environment.

```dql
// Count entities by type - see what's been discovered
fetch dt.entity.host
| summarize host_count = count()

// Smartscape equivalent (dt.entity.* is deprecated but still functional):
//   smartscapeNodes "HOST"
//   | summarize host_count = count()
// Caveat: Smartscape reflects CURRENT live topology and can report fewer entities
// than the classic entity store; for a pre-migration discovery inventory keep the
// classic query above.
```

```dql
// List all discovered hosts
fetch dt.entity.host
| fields entity.name, state
| sort entity.name
| limit 50

// Smartscape note (dt.entity.* is deprecated but still functional): this query uses the
// classic-only field state, which has NO Smartscape node equivalent
// (Smartscape expresses liveness via node lifetime, not a state field). Keep the classic
// query above for state detail. For monitoring mode, do not use monitoringMode — it is
// empty on Kubernetes and Fargate hosts; read billing events instead (ONBRD-05 § 6).
// Other fields do map: entity.name -> name.
```

```dql
// Check for any services
fetch dt.entity.service
| fields entity.name, serviceType
| sort entity.name
| limit 50

// Smartscape equivalent (dt.entity.* is deprecated but still functional):
//   smartscapeNodes "SERVICE"
//   | fields name, dt.service_detection.version, dt.service.sdv1_type
//   | sort name
//   | limit 50
// Caveat: Smartscape reflects CURRENT live topology and can report fewer entities
// than the classic entity store; for a pre-migration discovery inventory keep the
// classic query above.
// Field maps: serviceType -> dt.service.sdv1_type (SDv1 services only; null on SDv2
// services); entity.name -> name.
```

```dql
// Check for recent log data
fetch logs, from: now() - 1h
| summarize log_count = count()
```

```dql
// Check for recent problems
fetch dt.davis.problems, from: now() - 7d
| fields timestamp, display_id, event.name, event.status
| sort timestamp desc
| limit 10
```

### Interpreting Results

| Result | What It Means | Next Step |
|--------|--------------|----------|
| **Hosts found** | OneAgent or cloud integration active | Explore the Infrastructure & Operations app |
| **No hosts** | No monitoring deployed yet | Deploy OneAgent (ONBRD-05) |
| **Services found** | Application-level monitoring working | Review service mapping |
| **Logs found** | Log ingestion configured | Explore the Logs app |
| **Problems found** | Dynatrace Intelligence is detecting issues | Review problem details |

<a id="next-steps"></a>
## 6. Next Steps

Now that you're oriented in the Dynatrace UI, proceed based on your priorities:

### Recommended Path

1. **ONBRD-02: IAM and Authentication** - Set up SAML/SSO, Platform Tokens, and user permissions before inviting your team
2. **ONBRD-03: Deploying ActiveGate** - Set up network routing (if needed)
3. **ONBRD-04: Cloud & SaaS Integrations** - Connect AWS / Azure / GCP and SaaS data sources
4. **ONBRD-05: Deploying OneAgent** - Start getting infrastructure and application data
5. **ONBRD-06: Organizing Your Environment** - Set up tags, segments, and naming conventions

### Migrating from Another Platform?

If you're migrating from another APM tool, the deep-dive translation series cover concept mapping, query translation, and cutover patterns:

- **NRLC** (NR → Dynatrace component deep dives) + **NR2DT** (procedural runbook) for New Relic
- **SL2DT** for Sumo Logic logs/dashboards/monitors
- **S2D** for Splunk
- **M2S** for Managed-to-SaaS Dynatrace migrations

Pair the migration series with this **ONBRD** series for the platform-fundamentals foundation.

### Key Tasks Before Moving On

- [ ] Bookmark your tenant URL
- [ ] Note your tenant ID
- [ ] Explore the App Launcher
- [ ] Open the Infrastructure & Operations app (even if empty)
- [ ] Find Account Management for tokens and users

---

## Summary

In this notebook, you learned:

- How to access your Dynatrace environment
- The app-based navigation structure
- Key apps: Infrastructure & Operations, Problems, Logs, Notebooks, Workflows
- Important URLs for your tenant
- How to check what data already exists

---

## References

- [Get Started with Dynatrace (DT docs)](https://docs.dynatrace.com/docs/discover-dynatrace/get-started)
- [QuickStart guide (DT docs)](https://docs.dynatrace.com/docs/discover-dynatrace/get-started/quickstart-guide)
- [Navigate the Dynatrace Platform (DT docs)](https://docs.dynatrace.com/docs/discover-dynatrace/get-started/dynatrace-ui)
- [Dynatrace Community - Start with Dynatrace](https://community.dynatrace.com/t5/Start-with-Dynatrace/bd-p/GetStarted)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
