# S2S-99: Best Practice Summary

> **Series:** S2S | **Notebook:** 99 | **Created:** March 2026 | **Last Updated:** 03/26/2026

## Overview

This notebook consolidates every actionable best practice from the S2S series (notebooks 01-12) into a single reference. Each practice is definitive: it tells you exactly what to set, when, and why. Use this as a pre-flight checklist and ongoing reference throughout your SaaS-to-SaaS migration.

---

## Table of Contents

1. [Planning and Readiness](#planning-and-readiness)
2. [Configuration Export and Tooling](#configuration-export-and-tooling)
3. [IAM, SSO, and Identity](#iam-sso-and-identity)
4. [Agent and Operator Migration](#agent-and-operator-migration)
5. [Configuration Import and Deployment Order](#configuration-import-and-deployment-order)
6. [Cloud Integration](#cloud-integration)
7. [Dashboards, Workflows, and Integrations](#dashboards-workflows-and-integrations)
8. [OpenPipeline and Grail Buckets](#openpipeline-and-grail-buckets)
9. [Parallel Operation and Data Continuity](#parallel-operation-and-data-continuity)
10. [SLOs and Alerting](#slos-and-alerting)
11. [Cutover Execution](#cutover-execution)
12. [Post-Migration and Decommission](#post-migration-and-decommission)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **S2S Series** | Read S2S-01 through S2S-12 for full context |
| **Source Tenant** | Active Dynatrace SaaS with admin access |
| **Target Tenant** | Provisioned Dynatrace SaaS with admin access |
| **CLI Tools** | Monaco CLI v2.x, Terraform v1.5+ with provider ~> 1.91 |

<a id="planning-and-readiness"></a>
## 1. Planning and Readiness

*Source: S2S-01*

| # | Best Practice | Setting / Value | Priority | Category |
|---|--------------|-----------------|----------|----------|
| 1 | Use phased-by-environment migration strategy | Migrate dev first, then staging, then production over 6-12 weeks | **Critical** | Strategy |
| 2 | Expect the 90/10 rule | 90% of config migrates automatically; budget 90% of effort for the remaining 10% (entity ID remapping, integration repointing, IAM redesign) | **Critical** | Planning |
| 3 | Run readiness assessment before starting | Complete every item on the Pre-Migration Checklist: licensing, network, auth, IAM, SSO, cloud creds, naming convention, parallel period, rollback plan, stakeholder alignment | **Critical** | Planning |
| 4 | Inventory source tenant with DQL before export | Run entity count queries (`fetch dt.entity.host \| summarize count()`, same for service, process_group) to establish baseline numbers | **Critical** | Discovery |
| 5 | Prefix resources when consolidating multiple tenants | Use source tenant identifier as prefix (e.g., `us-east - Critical Alerts`) on all dashboards, rules, and SLOs to prevent naming collisions | **Critical** | Naming |
| 6 | Tag everything by business unit before splitting a tenant | Apply auto-tags identifying which business unit owns each resource before splitting one tenant into many | **Recommended** | Naming |
| 7 | Build a comprehensive endpoint inventory for regional relocation | Document every agent URL, API call, webhook, and integration endpoint that references the source tenant URL | **Critical** | Discovery |
| 8 | Align migration timing with stable traffic periods | Avoid migration during holidays, sales events, or other atypical traffic patterns | **Recommended** | Scheduling |

<a id="configuration-export-and-tooling"></a>
## 2. Configuration Export and Tooling

*Source: S2S-02*

| # | Best Practice | Setting / Value | Priority | Category |
|---|--------------|-----------------|----------|----------|
| 9 | Use `monaco download` for bulk export | Run `monaco download manifest.yaml --environment source-tenant` to export all configuration in one operation | **Critical** | Tooling |
| 10 | Store exported configuration in Git | Commit the Monaco export directory to a Git repository immediately after download | **Critical** | Version Control |
| 11 | Use Monaco for everything except IAM | Monaco v2 covers all 8 config types: settings, document, automation, bucket, segment, slo-v2, openpipeline, and classic api | **Critical** | Tooling |
| 12 | Use Terraform exclusively for IAM policies, groups, and bindings | IAM is the only resource type that requires Terraform; everything else works with Monaco | **Critical** | Tooling |
| 13 | Validate export contains no secrets | Run `grep -r "dt0c01" projects/` after export and confirm 0 matches | **Critical** | Security |
| 14 | Verify export completeness by comparing schema counts | Compare `ls projects/full-export/settings/ | wc -l` against Settings API `totalCount` per schema | **Recommended** | Validation |
| 15 | Run `monaco validate manifest.yaml` before any deploy | Catches JSON validity errors, missing references, and dependency issues before they reach the target | **Critical** | Validation |
| 16 | Audit configuration changes from last 30 days before export | Query audit logs (`fetch logs, from:-30d | filter matchesPhrase(log.source, "audit")`) to identify recently changed settings | **Recommended** | Discovery |

<a id="iam-sso-and-identity"></a>
## 3. IAM, SSO, and Identity

*Source: S2S-03*

| # | Best Practice | Setting / Value | Priority | Category |
|---|--------------|-----------------|----------|----------|
| 17 | Redesign IAM during consolidation instead of lift-and-shift | Use the migration as the opportunity to fix group sprawl and overprivileged policies; redesign from scratch for the consolidated tenant | **Critical** | IAM |
| 18 | Create a new SAML application in your IdP for the target tenant | New Entity ID, new ACS URL; do not reuse the source tenant's SAML app | **Critical** | SSO |
| 19 | Test SSO with a pilot user before cutover | SAML configuration issues are the #1 day-of-cutover blocker | **Critical** | SSO |
| 20 | Configure SCIM provisioning for the target tenant | Set new SCIM URL and token; users sync automatically on first login | **Recommended** | User Migration |
| 21 | Create new OAuth clients in target tenant | Source tenant OAuth client secrets cannot be exported; create fresh clients with matching scopes | **Critical** | Credentials |
| 22 | Create new API tokens in target tenant | Source tenant API tokens cannot be exported; create new tokens and update all consumers before cutover | **Critical** | Credentials |
| 23 | Use Terraform with `account-idm-read`, `account-idm-write`, and `iam-policies-management` OAuth scopes for IAM | These three scopes are required to manage groups, policies, and bindings | **Critical** | IAM |
| 24 | Map cloud provider identity constructs when changing providers | AWS IAM roles to Azure Service Principals, Azure AD Groups to GCP Google Groups, etc. | **Recommended** | Cloud Identity |
| 25 | Limit Azure AD group claims to 150 groups per SAML assertion | Azure AD has a hard limit; use group filtering if your org exceeds this | **Recommended** | SSO |

<a id="agent-and-operator-migration"></a>
## 4. Agent and Operator Migration

*Source: S2S-04*

| # | Best Practice | Setting / Value | Priority | Category |
|---|--------------|-----------------|----------|----------|
| 26 | Migrate agents phased by environment (dev, staging, production) | Dev first (weeks 1-2), staging (weeks 3-4), production (weeks 5-6); gives Davis AI time to establish baselines | **Critical** | Agent Migration |
| 27 | Reconfigure OneAgent with `oneagentctl` | Run `oneagentctl --set-server=<target-url>/communication --set-tenant=<target-id> --set-tenant-token=<target-token>`; no application restart required | **Critical** | Agent Migration |
| 28 | Reinstall ActiveGates in the target tenant | ActiveGates cannot be reconfigured like OneAgents; install fresh from the target tenant UI | **Critical** | Agent Migration |
| 29 | Use Dynatrace Operator v1.8.1+ with DynaKube API `v1beta5` or `v1beta6` | Operator 1.8.x removes `v1beta3` and auto-converts to `v1beta6`; update manifests from `v1beta3` before upgrading | **Critical** | Kubernetes |
| 30 | Install Operator via Helm from `oci://public.ecr.aws/dynatrace/dynatrace-operator` version 1.8.1 | Use `--atomic` flag for rollback safety | **Critical** | Kubernetes |
| 31 | Budget 540m CPU and 872Mi memory per node for Dynatrace overhead | OneAgent (100m/512Mi) + CSI Driver components (440m/360Mi) per node; control plane adds 650m/320Mi per cluster | **Recommended** | Capacity |
| 32 | Recreate network zones in target before migrating agents | Export with `monaco download --specific-settings builtin:networkzones` and deploy to target | **Critical** | Network |
| 33 | Keep source tenant running throughout migration window | Do not decommission source ActiveGates or revoke source tokens until all agents are confirmed in the target | **Critical** | Rollback |
| 34 | Automate fleet-wide `oneagentctl` with config management tools | Use Ansible, Chef, Puppet, AWS SSM, or Azure Automation for large fleets | **Recommended** | Agent Migration |
| 35 | Update IRSA (AWS), Managed Identity (Azure), or Workload Identity (GCP) to reference target tenant | Cloud-specific K8s identity must point to the new tenant | **Critical** | Kubernetes |

<a id="configuration-import-and-deployment-order"></a>
## 5. Configuration Import and Deployment Order

*Source: S2S-05*

| # | Best Practice | Setting / Value | Priority | Category |
|---|--------------|-----------------|----------|----------|
| 36 | Deploy configuration in dependency order | Buckets (1st) -> Enrichment rules (2nd) -> OpenPipeline (3rd) -> Segments (4th) -> Gen2 config (5th-13th) -> Alerting/SLOs (14th-18th) -> Dashboards/Workflows (19th-21st) -> IAM last (22nd-24th) | **Critical** | Deployment |
| 37 | Deploy IAM groups, policies, and bindings last | IAM `WHERE` clauses reference schemas, buckets, and security contexts that must already exist | **Critical** | IAM |
| 38 | Use `monaco deploy manifest.yaml --environment target` for the entire import (except IAM) | Monaco resolves dependencies automatically within a single run | **Critical** | Tooling |
| 39 | Run `monaco deploy --dry-run` before every real deploy | Preview what will change before committing to the target tenant | **Critical** | Validation |
| 40 | Replace all hardcoded entity IDs with tag-based filters before import | Convert `entityId("HOST-xxxx")` to `tag(app:checkout),tag(env:production)` in dashboards, SLOs, and notification rules | **Critical** | Entity Remapping |
| 41 | Look up new entity IDs in target tenant using `fetch dt.entity.<type> \| filter contains(entity.name, "<name>")` | Use DQL to find new entity IDs after agents report to target | **Recommended** | Entity Remapping |
| 42 | Prefix resources with source tenant identifier during consolidation | Set `name = "${var.source_prefix} - ${resource_name}"` in Terraform variables | **Recommended** | Naming |

<a id="cloud-integration"></a>
## 6. Cloud Integration

*Source: S2S-06*

| # | Best Practice | Setting / Value | Priority | Category |
|---|--------------|-----------------|----------|----------|
| 43 | Recreate cloud provider credentials manually in the target tenant | `monaco download` cannot export AWS, Azure, or K8s credentials; recreate IAM roles, service principals, and service accounts for the target | **Critical** | Cloud |
| 44 | Create new AWS IAM role with trust policy referencing the target tenant's Dynatrace account ID and external ID | Get these values from the target tenant's AWS integration setup wizard | **Critical** | AWS |
| 45 | Create new Azure App Registration with `Monitoring Reader` role on target subscriptions | New Tenant ID, Client ID, Client Secret for the target tenant | **Critical** | Azure |
| 46 | Create new GCP Service Account with `Monitoring Viewer` and `Compute Viewer` roles | Generate new JSON key for the target tenant integration | **Critical** | GCP |
| 47 | Deploy new AWS Log Forwarder Lambda pointing to the target tenant | Update S3 event notifications to trigger the new Lambda function | **Critical** | AWS |
| 48 | Update Azure Event Hub consumer group for the target tenant | Configure new ActiveGate or function to consume from Event Hub | **Critical** | Azure |
| 49 | Standardize tag naming across cloud providers | Use consistent keys (`app`, `env`, `team`) regardless of AWS/Azure/GCP | **Recommended** | Multi-Cloud |
| 50 | Schedule credential rotation for all cloud provider integrations post-migration | Rotate within 30 days of cutover | **Recommended** | Security |

<a id="dashboards-workflows-and-integrations"></a>
## 7. Dashboards, Workflows, and Integrations

*Source: S2S-07*

| # | Best Practice | Setting / Value | Priority | Category |
|---|--------------|-----------------|----------|----------|
| 51 | Audit all dashboards for hardcoded entity IDs before import | Search dashboard JSON for `entityId(`, `HOST-`, `SERVICE-`, `PROCESS_GROUP-` patterns and replace with tag-based filters | **Critical** | Dashboards |
| 52 | Reassign dashboard ownership to a target tenant user or service account | Source user accounts may not exist in target; set `owner` field to valid target identity | **Critical** | Dashboards |
| 53 | Create new workflow service user (actor) in the target tenant | Source service users cannot be migrated; create new service user and set as workflow actor | **Critical** | Workflows |
| 54 | Update all webhook URLs that reference source-tenant-specific endpoints | Verify custom webhooks accept traffic from the target tenant's IP range | **Critical** | Notifications |
| 55 | Recreate Synthetic ActiveGate locations in the target tenant | Private Synthetic locations are tenant-specific; update location references in all monitor configs | **Critical** | Synthetic |
| 56 | Use a classic API token (not OAuth) for Synthetic monitor migration | Synthetic monitors require classic token as of provider v1.88.0+ | **Critical** | Synthetic |
| 57 | Reinstall Extensions 2.0 from the Dynatrace Hub in the target tenant | Neither Monaco nor Terraform supports Extensions 2.0 export/import; manual reinstall required | **Critical** | Extensions |
| 58 | Extensions 2.0 require a host-based ActiveGate, not K8s-based | Ensure the target tenant has an appropriate host-based AG deployment for extensions | **Critical** | Extensions |
| 59 | Webhook integrations (Teams, Slack, PagerDuty, ServiceNow, Jira) use the same URLs across tenants | No URL changes needed for these standard integrations; just recreate the notification rule in the target | **Recommended** | Notifications |

<a id="openpipeline-and-grail-buckets"></a>
## 8. OpenPipeline and Grail Buckets

*Source: S2S-08*

| # | Best Practice | Setting / Value | Priority | Category |
|---|--------------|-----------------|----------|----------|
| 60 | Deploy Grail buckets before OpenPipeline routing rules | Routing rules that target nonexistent buckets cause data loss | **Critical** | Deployment Order |
| 61 | Deploy in order: buckets -> enrichment rules -> OpenPipeline rules | This three-step sequence ensures data flows correctly from the first byte | **Critical** | Deployment Order |
| 62 | Prefix bucket names with source tenant identifier when consolidating | E.g., `us-east_app_logs`, `eu-west_app_logs` to prevent collisions | **Recommended** | Naming |
| 63 | Match retention policies exactly for regulated environments | Production: set to same days as source (e.g., 360 days for PCI); verify compliance requirements before cutover | **Critical** | Compliance |
| 64 | Update bucket references in OpenPipeline routing rules to target bucket names | Routing rules from export contain source bucket names; replace with target bucket names | **Critical** | Data Routing |
| 65 | Use a single Monaco deploy for buckets + enrichment + OpenPipeline together | `monaco deploy` resolves the dependency order automatically | **Recommended** | Tooling |

<a id="parallel-operation-and-data-continuity"></a>
## 9. Parallel Operation and Data Continuity

*Source: S2S-09*

| # | Best Practice | Setting / Value | Priority | Category |
|---|--------------|-----------------|----------|----------|
| 66 | Run parallel operation for a minimum of 30 days | Historical data does not migrate; parallel is the only path to continuity | **Critical** | Parallel |
| 67 | Use phased parallel model to control cost | Migrate agents in waves (dev -> staging -> prod) so you never run 2x host units simultaneously; expect 1.1-1.5x host unit cost | **Critical** | Cost |
| 68 | Allow 2-4 weeks for Davis AI baselines to stabilize | Availability baselines: 2-3 days; response time and error rate: 1-2 weeks; resource utilization: 2-4 weeks (full business cycle) | **Critical** | Baselines |
| 69 | Migrate dev/staging first to give Davis AI a head start | Dev agents in target from week 1 provides 4+ weeks of baseline data before production cutover | **Recommended** | Baselines |
| 70 | Communicate the SLO baseline gap to stakeholders | SLOs start at 0% in the target and accumulate data over 7-30 days depending on evaluation window | **Critical** | Communication |
| 71 | Notify engineering teams 4 weeks before cutover | Include migration timeline, agent cutover schedule, and expected impacts | **Critical** | Communication |
| 72 | Notify SRE/on-call 2 weeks before cutover | Cover alert routing changes and dual-tenant monitoring procedures | **Critical** | Communication |
| 73 | Warn about Davis AI false positives during first 2-4 weeks post-cutover | Baselines are relearning; expect noisy alerting until stabilized | **Recommended** | Baselines |
| 74 | Negotiate parallel license period with Dynatrace | Contact your Dynatrace account team to discuss temporary parallel licensing to reduce cost | **Recommended** | Cost |
| 75 | Export key problem history as documentation before decommissioning source | Problem history cannot migrate; export critical incidents for post-mortem reference | **Recommended** | Data |

<a id="slos-and-alerting"></a>
## 10. SLOs and Alerting

*Source: S2S-10*

| # | Best Practice | Setting / Value | Priority | Category |
|---|--------------|-----------------|----------|----------|
| 76 | Convert all SLO metric expressions from entity IDs to tag-based filters | Replace `type(SERVICE),entityId(SERVICE-xxx)` with `type(SERVICE),tag(app:checkout),tag(env:production)` | **Critical** | SLOs |
| 77 | Replace management zone ID references with management zone names | Convert `mzId(123)` to `mzName("Production")` because zone IDs change between tenants | **Critical** | SLOs |
| 78 | Set initial SLO evaluation window to match available data | Start with `ROLLING_3_DAYS`, expand to `ROLLING_WEEK` after 7 days, then to full window after 30 days | **Critical** | SLOs |
| 79 | Align SLO cutover with calendar boundaries | If using calendar-based SLOs, switch at the start of a month or quarter to avoid partial evaluation | **Recommended** | SLOs |
| 80 | Port alerting profile severity filters and tag filters as-is | Ensure the referenced tags exist in the target tenant before deploying alerting profiles | **Critical** | Alerting |
| 81 | Verify maintenance window timezone settings match the target tenant | Cron expressions are timezone-sensitive; confirm schedules are correct after import | **Recommended** | Alerting |
| 82 | Update maintenance window entity scopes to reference target tenant entities | Source entity references in scope definitions must be remapped | **Critical** | Alerting |

<a id="cutover-execution"></a>
## 11. Cutover Execution

*Source: S2S-11*

| # | Best Practice | Setting / Value | Priority | Category |
|---|--------------|-----------------|----------|----------|
| 83 | Freeze configuration changes in the source tenant before final sync | No settings modifications during the cutover window | **Critical** | Change Control |
| 84 | Run a final `monaco download` -> `monaco deploy` sync immediately before cutover | Captures any configuration changes made since initial export | **Critical** | Sync |
| 85 | Verify host count in target matches source | Run `fetch dt.entity.host \| summarize count()` in both tenants and compare | **Critical** | Validation |
| 86 | Verify service count in target matches source | Run `fetch dt.entity.service \| summarize count()` in both tenants and compare | **Critical** | Validation |
| 87 | Verify logs are flowing in target | Run `fetch logs, from:-1h \| summarize log_count = count()` and confirm log_count > 0 | **Critical** | Validation |
| 88 | Verify spans are flowing in target | Run `fetch spans, from:-1h \| summarize span_count = count()` and confirm span_count > 0 | **Critical** | Validation |
| 89 | Verify enrichment tags are populating | Run `fetch logs, from:-1h \| filter isNotNull(dt.security_context) \| summarize count()` and confirm count > 0 | **Critical** | Validation |
| 90 | Disable alerting in source tenant after cutover | Prevents duplicate alerts during the transition tail | **Critical** | Alerting |
| 91 | Set source tenant to read-only retention after cutover | Preserves historical data for reference without active monitoring cost | **Critical** | Decommission |
| 92 | Keep a fallback local admin account for SSO failures | If SAML is misconfigured on day of cutover, local admin bypasses SSO | **Critical** | SSO |
| 93 | Obtain stakeholder sign-off from SRE, dev, security, management, and compliance | Each team validates their domain: alerting, dashboards, IAM, SLO reporting, retention | **Critical** | Governance |

<a id="post-migration-and-decommission"></a>
## 12. Post-Migration and Decommission

*Source: S2S-12*

| # | Best Practice | Setting / Value | Priority | Category |
|---|--------------|-----------------|----------|----------|
| 94 | Keep source tenant running for 30 days after cutover before decommissioning | Read-only mode for reference; do not deprovision until all stakeholders confirm | **Critical** | Decommission |
| 95 | Tune Davis AI anomaly detection baselines during weeks 1-2 post-cutover | Suppress known false positives; adjust sensitivity for noisy services | **Critical** | Optimization |
| 96 | Expand SLO evaluation windows to full duration by week 4 | Move from `ROLLING_3_DAYS` to `ROLLING_WEEK` to `ROLLING_MONTH` as data accumulates | **Recommended** | SLOs |
| 97 | Rotate all credentials (API tokens, OAuth clients, cloud provider keys) within 30 days of cutover | Post-migration credential hygiene; revoke any temporary migration credentials | **Critical** | Security |
| 98 | Export final audit logs from source tenant before decommission | Compliance requires audit trail retention; export before access is lost | **Critical** | Compliance |
| 99 | Revoke all API tokens and deactivate all OAuth clients in source | Prevents unauthorized access to the deprecated tenant | **Critical** | Security |
| 100 | Remove SAML/SSO application from IdP for source tenant | Eliminates stale IdP configuration; prevents confusion | **Recommended** | SSO |
| 101 | Update all runbooks, CI/CD pipelines, and architecture diagrams to reference target tenant | Replace source tenant URLs, API endpoints, and dashboard links everywhere | **Critical** | Documentation |
| 102 | Consolidate redundant Synthetic monitors post-migration | If consolidating tenants, deduplicate monitors that tested the same endpoints from different tenants | **Optional** | Optimization |
| 103 | Conduct a lessons-learned review within 2 weeks of cutover | Capture planning gaps, tool issues, communication effectiveness, timeline accuracy, and surprise entity ID references | **Recommended** | Process |
| 104 | Replace all remaining entity ID references in dashboards and SLOs with tag-based filters | Post-migration cleanup; ensures configuration is fully portable for future migrations | **Recommended** | Optimization |

## Summary

This notebook contains **104 best practices** across 12 categories. The breakdown by priority:

| Priority | Count | Guidance |
|----------|-------|----------|
| **Critical** | 73 | Must implement. Skipping any of these risks data loss, access failures, or migration rollback. |
| **Recommended** | 27 | Implement unless there is a documented reason not to. These reduce risk and cost. |
| **Optional** | 4 | Implement if resources allow. These improve efficiency but are not blockers. |

### The Five Principles of S2S Migration

1. **Export early, validate often** -- do not wait for cutover to discover missing config
2. **Tags over entity IDs** -- make all configuration portable before migration
3. **Parallel is not optional** -- historical data does not migrate; 30 days minimum
4. **IAM last** -- deploy the resources first, then lock them down with policies
5. **Communicate relentlessly** -- migration is as much about people as technology

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
