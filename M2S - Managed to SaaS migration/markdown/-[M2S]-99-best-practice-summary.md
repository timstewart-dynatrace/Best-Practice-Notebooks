# M2S-99: Best Practice Summary

> **Series:** M2S | **Notebook:** 99 | **Created:** March 2026 | **Last Updated:** 03/26/2026

A definitive, actionable reference of every best practice extracted from the M2S Managed-to-SaaS Migration series (notebooks 01–08). Each practice specifies the exact setting or action, its priority, and its category. No hedging—follow these and your migration succeeds.

---

## Table of Contents

1. [Planning and Assessment](#planning-and-assessment)
2. [Network and Architecture](#network-and-architecture)
3. [ActiveGate Deployment](#activegate-deployment)
4. [Configuration Migration](#configuration-migration)
5. [OneAgent Migration](#oneagent-migration)
6. [Security and Access Control](#security-and-access-control)
7. [Privacy and Data Protection](#privacy-and-data-protection)
8. [Validation and Post-Migration](#validation-and-post-migration)
9. [Optimization and Feature Adoption](#optimization-and-feature-adoption)

---

<a id="planning-and-assessment"></a>
## 1. Planning and Assessment

| Practice | Setting / Action | Priority |
|----------|-----------------|----------|
| Enforce host limit per tenant | Maximum **25,000 monitored hosts** per SaaS tenant. Split into multiple tenants if consolidating Managed clusters exceeds this. | Critical |
| Align Managed and SaaS versions | Both environments must be on the **same major version** (e.g., both 1.294.x) before using SaaS Upgrade Assistant. | Critical |
| Require Managed cluster v1.294+ | SaaS Upgrade Assistant requires **Dynatrace Managed version 1.294 or later**. Upgrade the Managed cluster first if needed. | Critical |
| Complete entity discovery before strategy | Run Entities API v2 inventory: `type(HOST)`, `type(SERVICE)`, `type(APPLICATION)`, `type(SYNTHETIC_TEST)`. Document counts in a spreadsheet. | Critical |
| Choose one migration tool | Pick **one** primary tool: SaaS Upgrade Assistant (recommended), Monaco, Terraform, or Settings API. Never mix approaches—Monaco YAML conflicts with the SaaS Upgrade Assistant. | Critical |
| Use SaaS Upgrade Assistant for most migrations | Install the app on the target SaaS tenant. Assign `upgrade-assistant:environments:write` IAM policy to migration users. | Recommended |
| Use Monaco instead for tenant consolidation | When consolidating multiple Managed tenants with **identical configuration names**, use Monaco—the SaaS Upgrade Assistant cannot resolve name conflicts. | Recommended |
| Plan for historic data loss | Historic metrics, logs, traces, problems, and session data **cannot be migrated**. Export critical reports before shutdown. Run dual environments to build SaaS baselines. | Critical |
| Coordinate licensing for dual-run | Work with Dynatrace account team to arrange temporary overlap licensing for the period both Managed and SaaS run simultaneously. | Recommended |
| Define measurable success criteria | Set targets: 100% host coverage, 100% service discovery, <15 min data gaps, 100% alert delivery, 100% dashboard availability, 100% integration success. | Critical |
| Choose migration approach by size | <500 hosts: Big Bang. 500–2,000 hosts: Phased by environment. >2,000 hosts or complex integrations: Phased by region/application. | Recommended |
| Engage Dynatrace Professional Services early | Involve account team and PS during the Plan phase, not during execution. | Recommended |
| Budget time for non-portable items | 90% of configs migrate automatically; the remaining 10% (credentials, webhooks, custom scripts) takes 90% of the manual effort. | Recommended |

<a id="network-and-architecture"></a>
## 2. Network and Architecture

| Practice | Setting / Action | Priority |
|----------|-----------------|----------|
| Open outbound 443 to SaaS endpoints | Allow HTTPS (port **443**) from all monitored hosts and ActiveGates to `{tenant-id}.live.dynatrace.com` and `{tenant-id}.apps.dynatrace.com`. | Critical |
| Use ActiveGate routing for restricted networks | If hosts cannot reach the internet directly, deploy Environment ActiveGates. OneAgents connect to AG on port **9999**; AG connects outbound on **443**. | Critical |
| Ensure DNS resolves SaaS domains | Verify DNS resolution for `{tenant-id}.live.dynatrace.com`, `{tenant-id}.apps.dynatrace.com`, and `*.dynatrace.com`. | Critical |
| Test connectivity before migration | Run `curl -v https://{tenant-id}.live.dynatrace.com/api/v1/time` from monitored hosts and ActiveGate servers before scheduling the migration window. | Critical |
| Recreate Network Zones in SaaS before migrating agents | Network Zone configuration does **not** transfer automatically. Create zones in SaaS via Settings > Network zones or the API, then assign AGs and OneAgents. | Critical |
| Match zones to physical network topology | Name zones by datacenter or region (e.g., `datacenter-east`, `aws-us-east-1`). Define alternative zones for failover. | Recommended |
| Deploy minimum 2 ActiveGates per network zone | Provides high availability within each zone. OneAgent auto-discovers available AGs. | Recommended |
| Enforce TLS 1.2+ for all traffic | Dynatrace SaaS enforces TLS 1.2 minimum. Verify no legacy TLS 1.0/1.1 configurations on proxies or load balancers. | Critical |

<a id="activegate-deployment"></a>
## 3. ActiveGate Deployment

| Practice | Setting / Action | Priority |
|----------|-----------------|----------|
| Deploy new SaaS ActiveGates in parallel | Install new SaaS-connected AGs alongside existing Managed AGs **before** migrating OneAgents. Validates connectivity without impacting monitored hosts. | Critical |
| Size ActiveGates by host count | Up to 500 hosts: **2 CPU / 4 GB RAM / 20 GB disk**. 500–1,500: **4 CPU / 8 GB / 40 GB**. 1,500–5,000: **8 CPU / 16 GB / 80 GB**. >5,000: scale horizontally with multiple AGs. | Recommended |
| Place ActiveGates close to monitored hosts | Minimize latency by deploying AGs in the same datacenter or cloud region as the hosts they serve. | Recommended |
| Use ActiveGate groups for role separation | Create groups: `production-routing`, `nonprod-routing`, `extensions`, `synthetic`. Assign AGs to groups based on function. | Recommended |
| Assign network zone during AG installation | Use `--set-network-zone=<zone>` flag during installation. Post-install: add `networkzone=<zone>` to `config.properties`. | Recommended |
| Deploy dedicated AGs for Extensions 2.0 | Extensions 2.0 require **host-based** ActiveGate (not K8s-based). Deploy a separate AG group for extension execution. | Recommended |

<a id="configuration-migration"></a>
## 4. Configuration Migration

| Practice | Setting / Action | Priority |
|----------|-----------------|----------|
| Migrate configurations BEFORE redirecting OneAgents | Deploy foundational settings (management zones, tags, service detection rules) to SaaS before any agents start reporting. | Critical |
| Follow the 4-wave deployment order | **Wave 1:** Management zones, auto-tagging rules, host groups. **Wave 2:** Service detection, request attributes, deep monitoring. **Wave 3:** Alerting profiles, anomaly detection, SLOs. **Wave 4:** Dashboards, remaining configs. | Critical |
| Use SaaS Upgrade Assistant selective import | Deploy configuration types incrementally using the Smart Selective Import. Smart dependency management auto-adds required configs. | Recommended |
| Preview all changes before deploying | Use the SaaS Upgrade Assistant **Preview Changes** feature to verify edits. The app retains original values for rollback reference. | Recommended |
| Download deploy result CSVs for audit | After each wave, download the deployment result CSV from SaaS Upgrade Assistant. Archive these for compliance and troubleshooting. | Recommended |
| Manually recreate Credentials Vault entries | Cannot be exported. Re-enter all credentials (cloud platform integrations, extension credentials) directly in SaaS. | Critical |
| Recreate problem notification webhooks | Endpoint URLs change for SaaS. Create new Slack, Teams, PagerDuty, ServiceNow, and custom webhook integrations with SaaS URLs. | Critical |
| Generate new API tokens in SaaS | Managed tokens do not work in SaaS. Create new tokens with **minimal required scopes** for each use case. | Critical |
| Remove hardcoded entity IDs from dashboards | Before importing dashboards, replace hardcoded entity IDs with entity selectors. Update management zone IDs and dashboard owners. | Recommended |
| Recreate synthetic private locations | Deploy new Synthetic ActiveGates in SaaS. Private locations are infrastructure-specific and do not transfer. | Recommended |
| Update all API scripts with new base URL | Change from `{managed-url}/e/{env-id}` to `{tenant}.live.dynatrace.com`. Update tokens in all CI/CD pipelines and automation scripts. | Critical |
| Recreate automations in the Workflows app | Managed problem notifications become workflow triggers in SaaS. Custom webhooks become HTTP Request actions. Document existing logic first, then rebuild. | Recommended |

<a id="oneagent-migration"></a>
## 5. OneAgent Migration

| Practice | Setting / Action | Priority |
|----------|-----------------|----------|
| Use `oneagentctl --set-server` to reconfigure (not reinstall) | Run: `oneagentctl --set-server="https://{tenant}.live.dynatrace.com:443/communication"` then `--set-tenant-token="{token}"` then `systemctl restart oneagent`. Preserves host identity and custom metadata. | Critical |
| Migrate non-production first, then production | Validate data flow on non-prod hosts before touching production. Apply lessons learned to each subsequent wave. | Critical |
| Restart application processes after OneAgent migration | Full-stack monitoring requires process restarts—Java, .NET, Node.js, PHP all inject instrumentation at process startup. Without restart: host metrics flow but **no distributed tracing, no service detection, no code-level visibility**. | Critical |
| Verify OneAgent versions are within support window | OneAgent support window is **9–12 months**. Check oldest deployed versions before migration. Upgrade outdated agents first. | Recommended |
| Use Ansible/automation for bulk reconfiguration | For large environments, use configuration management (Ansible, Puppet, Chef) to execute `oneagentctl` commands and rolling application restarts across all hosts. | Recommended |
| Choose parallel install for zero-downtime requirements | Install SaaS OneAgent alongside Managed temporarily. Higher resource overhead but eliminates monitoring gap. Use only when zero-gap is mandatory. | Optional |
| Configure OneAgent network zone assignment | Set during reconfiguration: `oneagentctl --set-network-zone=<zone>`. Must match the Network Zones created in SaaS. | Recommended |
| Prepare rollback procedure | Document the Managed server URL and token. Rollback: `oneagentctl --set-server="https://{managed-cluster}/communication"` + restart. Test on one host before bulk migration. | Critical |

<a id="security-and-access-control"></a>
## 6. Security and Access Control

| Practice | Setting / Action | Priority |
|----------|-----------------|----------|
| Configure SAML SSO with full message signing | IdP must sign the **entire SAML message**, not just the assertion. Azure AD meets this requirement by default. Failure causes authentication errors. | Critical |
| Filter SAML group claims to Dynatrace groups only | Azure Entra limit: **150 groups** per user in SAML claim. Filter to Dynatrace-related groups to stay under the limit. | Critical |
| Replace LDAP with SAML | SaaS does not support direct LDAP authentication. Migrate to SAML 2.0 SSO through your IdP. | Critical |
| Map Managed roles to SaaS IAM policies | Cluster admin → Account admin. Environment admin → Environment admin. Monitor user → Viewer + specific policies. Custom roles → Custom IAM policies. | Critical |
| Create purpose-specific API tokens with minimal scopes | Separate tokens per use case: installation (`InstallerDownload`), config migration (`settings.read/write`), queries (`entities.read`, `metrics.read`, `logs.read`), automation (`automation.read/write`). | Critical |
| Set API token expiration dates | Set expiration on all tokens. Store in secrets management (HashiCorp Vault, AWS Secrets Manager). Rotate regularly. | Recommended |
| Request SOC 2 Type II report from Dynatrace | Obtain SOC 2 Type II, ISO 27001 certificate, penetration test summaries, and Data Processing Addendum (DPA) for your security review. | Recommended |
| Confirm data residency region at provisioning | Select the correct region (US, EU, APAC) during tenant provisioning. Cannot be changed after provisioning. | Critical |
| Restrict egress to Dynatrace domains only | Firewall egress rules should allow only `*.dynatrace.com` on port 443. Block all other outbound from OneAgent and ActiveGate servers. | Recommended |

<a id="privacy-and-data-protection"></a>
## 7. Privacy and Data Protection

| Practice | Setting / Action | Priority |
|----------|-----------------|----------|
| Enable PII masking for request attributes | Settings → Request attributes: enable masking on all attributes capturing user input, account numbers, or personal data. | Critical |
| Configure session replay masking | RUM settings: mask all form inputs, text fields, and sensitive UI elements in session recordings. | Critical |
| Enable IP anonymization | RUM settings: enable IP anonymization to prevent storage of end-user IP addresses. | Recommended |
| Use OpenPipeline for log masking | Configure log processing rules in OpenPipeline to redact PII (SSNs, credit card numbers, email addresses) before storage. | Recommended |
| Exclude sensitive URL parameters | RUM settings: configure URL exclusion rules to strip query string parameters containing sensitive data (tokens, session IDs, PII). | Recommended |
| Disable session replay for internal apps | If internal apps do not require session replay, disable it to minimize data collection. | Optional |
| Obtain Data Processing Addendum for GDPR | Request the DPA from Dynatrace. Ensure it covers data subject access requests, right to erasure, and data portability. | Recommended |

<a id="validation-and-post-migration"></a>
## 8. Validation and Post-Migration

| Practice | Setting / Action | Priority |
|----------|-----------------|----------|
| Compare entity counts: Managed inventory vs SaaS | Run `fetch dt.entity.host \| summarize count()` (and service, application, process_group, synthetic_test). Counts must match the planning-phase inventory. | Critical |
| Verify metrics flow with no gaps >15 minutes | Run `timeseries avg(dt.host.cpu.usage), from:-1h, by:{dt.entity.host}` and check for nulls. Any host returning null has a data gap. | Critical |
| Validate log ingestion is continuous | Run `fetch logs, from:-1h \| summarize count(), by:{bin(timestamp, 5m)}` and confirm no 5-minute buckets with zero count. | Critical |
| Confirm distributed traces are flowing | Run `fetch spans, from:-1h \| summarize count(), by:{bin(timestamp, 5m)}`. Zero spans after agent migration means application processes need restart. | Critical |
| Review SaaS Upgrade Assistant deploy results | Download the CSV from each deployment wave. Filter for failed configurations. Fix or exclude with documented justification. | Critical |
| Test alert delivery end-to-end | Trigger a test alert and confirm delivery through every notification channel (email, Slack, Teams, PagerDuty, ServiceNow). | Critical |
| Validate all dashboard tiles show data | Open every migrated dashboard. Tiles with "No data" indicate missing entity mappings or incorrect management zone IDs. | Recommended |
| Get stakeholder sign-off | Obtain written approval from: Platform Team Lead, Security Team, Application Teams, Executive Sponsor. | Critical |
| Keep Managed running 2–4 weeks post-migration | Maintain access for historical reference and comparison. Decommission only after full validation period. | Recommended |
| Archive migration artifacts | Save SaaS Upgrade Assistant deploy result CSVs, entity inventory spreadsheets, and configuration mapping documents before decommissioning Managed. | Recommended |

<a id="optimization-and-feature-adoption"></a>
## 9. Optimization and Feature Adoption

| Practice | Setting / Action | Priority |
|----------|-----------------|----------|
| Allow 7–14 days for Davis AI baselines | Response time and error rate baselines: **2–7 days**. Resource usage and traffic patterns: **7–14 days**. Expect higher alert volume during this period—do not disable alerting. | Critical |
| Tune alert thresholds after baseline period | After 2 weeks: raise thresholds on noisy alerts, add time-based conditions, lower thresholds on missing alerts. | Recommended |
| Optimize DQL queries on dashboards | Add explicit `from:` time filters to every query. Use `summarize` instead of fetching raw rows. Apply `filter` before aggregation. Use `limit` on all exploration queries. | Recommended |
| Adopt Notebooks for interactive analysis | Replace ad-hoc USQL queries with Notebook-based DQL analysis. Share notebooks across teams for collaboration. Priority: **weeks 3–4** post-migration. | Recommended |
| Implement Workflows for automation | Replace Managed problem notifications with SaaS Workflow triggers + HTTP Request actions. Priority: **weeks 5–6** post-migration. | Recommended |
| Explore Davis Copilot for AI-assisted analysis | Enable Davis Copilot for natural language querying and root cause analysis. Priority: **weeks 7–8** post-migration. | Optional |
| Right-size data retention for cost optimization | Configure Grail bucket retention by data type. Logs: set retention based on compliance needs. Metrics: leverage the 5-year aggregated retention. Use `bucket:` targeting in queries to limit scan scope. | Recommended |
| Update all runbooks and documentation | Replace Managed URLs, procedures, and screenshots with SaaS equivalents. Update architecture diagrams, onboarding guides, and disaster recovery plans. | Recommended |

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
