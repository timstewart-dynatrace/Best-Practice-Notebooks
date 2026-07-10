# Best-Practice-Notebooks

Dynatrace best-practice notebooks with matching PDF and Markdown exports. These assets are not officially supported by Dynatrace.

> **Recommended:** Import the JSON files from notebooks/ into a Dynatrace tenant for the best experience. These notebooks contain interactive DQL queries that execute against your environment's data.



[![Start Here — Find Your Path](-START-HERE-/images/00-readme-banner.svg)](-START-HERE-/README.md)


> **👉 New here?** Open [`-START-HERE-/`](-START-HERE-/README.md) — a navigational playbook that picks an entry path based on your situation (net-new, expand/consolidate, deployment migration), then sequences the relevant topic series in order.
> Already know what you need? Skip the playbook and **[browse all 32 topic series ↓](#all-series-az)**.

## Layout

Each topic follows the same structure:
- notebooks/ — Dynatrace notebook JSON files (import into Dynatrace)
- pdfs/ — Printable/exported versions of the notebooks
- markdown/ — Markdown exports of the same content
- README.md — Topic overview and usage guide

## Categories

Series are grouped into six categories for navigation. An alphabetical index follows below.

- **Foundations & Adoption** — getting started, maturity, access, data organization, cost management
  [ONBRD](#onbrd---dynatrace-onboarding) · [ADOPT](#adopt---observability-adoption--maturity) · [IAM](#iam---iam-administration) · [ORGNZ](#orgnz---organize-data-buckets-segments-security) · [FAQ](#faq---frequently-asked-questions) · [FINOPS](#finops---cost-management--finops)
- **Data Sources & Instrumentation** — what you monitor and how to instrument it
  [K8S](#k8s---kubernetes-monitoring) · [CLOUD](#cloud---cloud-provider-integrations) · [MOBL](#mobl---mobile-monitoring) · [WEBRUM](#webrum---web-real-user-monitoring) · [SYNTH](#synth---synthetic-monitoring) · [DBMON](#dbmon---database-monitoring) · [OTEL](#otel---opentelemetry-integration)
- **Data Processing & Analytics** — ingestion, shaping, querying, dashboards
  [OPLOGS](#oplogs---openpipeline-logs) · [OPIPE](#opipe---openpipeline-beyond-logs) · [SPANS](#spans---distributed-tracing-and-spans) · [BIZEV](#bizev---business-events--funnel-analysis) · [DASH](#dash---dashboard-design--building)
- **Automation & Workflows** — configuration-as-code, alerting, reliability, and operational workflows
  [AUTOM](#autom---dynatrace-automation) · [WFLOW](#wflow---workflows-and-alert-notifications) · [AIOPS](#aiops---dynatrace-intelligence) · [ALERT](#alert---alerting-strategy-and-design) · [SLO](#slo---service-level-objectives)
- **Security** — runtime vulnerability analytics, application protection, and security posture management
  [APPSEC](#appsec---application-security)
- **Migrations** — moving to Dynatrace or between Dynatrace environments
  [M2S](#m2s---managed-to-saas-migration) · [S2S](#s2s---saas-to-saas-migration) · [S2D](#s2d---splunk-to-dynatrace-migration) · [SL2DT](#sl2dt---sumo-logic-to-dynatrace) · [NR2DT](#nr2dt---new-relic-to-dynatrace-migration-steps) · [NRLC](#nrlc---new-relic-to-dynatrace-migration-deep-dives) · [OPMIG](#opmig---openpipeline-migration) · [MZ2POL](#mz2pol---management-zone-to-policy-migration)

## All Series (A–Z)

### [ADOPT - Observability Adoption & Maturity](ADOPT%20-%20Observability%20Adoption%20%26%20Maturity/README.md)
Framework for assessing and advancing Dynatrace observability adoption across the organization.
- [ADOPT-01: Observability Maturity Model](ADOPT%20-%20Observability%20Adoption%20%26%20Maturity/markdown/-%5BADOPT%5D-01-maturity-model.md) — Five-level maturity framework
- [ADOPT-02: Platform Health Assessment](ADOPT%20-%20Observability%20Adoption%20%26%20Maturity/markdown/-%5BADOPT%5D-02-platform-health-assessment.md) — Evaluating current state and readiness
- [ADOPT-03: Success Metrics](ADOPT%20-%20Observability%20Adoption%20%26%20Maturity/markdown/-%5BADOPT%5D-03-success-metrics.md) — Defining KPIs and measuring adoption
- [ADOPT-04: Team Enablement](ADOPT%20-%20Observability%20Adoption%20%26%20Maturity/markdown/-%5BADOPT%5D-04-team-enablement.md) — Training and organization strategies
- [ADOPT-05: Optimization Roadmap](ADOPT%20-%20Observability%20Adoption%20%26%20Maturity/markdown/-%5BADOPT%5D-05-optimization-roadmap.md) — Building your adoption roadmap
- [ADOPT-06: Maximizing Platform Value — Coverage Audit and Staged Enablement](ADOPT%20-%20Observability%20Adoption%20%26%20Maturity/markdown/-%5BADOPT%5D-06-maximizing-value-staged-enablement.md) — Coverage audit workflow, staged enablement waves, and value gates for closing partial-coverage gaps
- [ADOPT-99: Best Practice Summary](ADOPT%20-%20Observability%20Adoption%20%26%20Maturity/markdown/-%5BADOPT%5D-99-best-practice-summary.md) — Consolidated best practices from the ADOPT series

### [AIOPS - Dynatrace Intelligence](AIOPS%20-%20Dynatrace%20Intelligence/README.md)
Comprehensive guide to AIOps and Dynatrace Intelligence — Causal AI, Predictive AI, and Generative AI capabilities across the platform.
- [AIOPS-01: Dynatrace Intelligence Overview](AIOPS%20-%20Dynatrace%20Intelligence/markdown/-%5BAIOPS%5D-01-dynatrace-intelligence-overview.md) — Causal, Predictive, and Generative AI categories and where they appear in the product
- [AIOPS-02: Anomaly Detection](AIOPS%20-%20Dynatrace%20Intelligence/markdown/-%5BAIOPS%5D-02-anomaly-detection.md) — Static thresholds, auto-adaptive, seasonal baselines, and multi-dimensional detection
- [AIOPS-03: Davis AI — Problems and Root Cause Analysis](AIOPS%20-%20Dynatrace%20Intelligence/markdown/-%5BAIOPS%5D-03-davis-ai-problems-and-root-cause.md) — Causal AI engine, problem grouping via Smartscape, and DQL for querying problems
- [AIOPS-04: Davis CoPilot — Dynatrace Assist for Investigation](AIOPS%20-%20Dynatrace%20Intelligence/markdown/-%5BAIOPS%5D-04-davis-copilot-dynatrace-assist.md) — Generative AI surfaces, NL-to-DQL, problem summaries, and investigative patterns
- [AIOPS-05: AI Models — Causal, Predictive, and Generative](AIOPS%20-%20Dynatrace%20Intelligence/markdown/-%5BAIOPS%5D-05-ai-models.md) — Model inventory, responsibilities, costs, and governance considerations
- [AIOPS-06: AI Integrations and Agentic Workflows](AIOPS%20-%20Dynatrace%20Intelligence/markdown/-%5BAIOPS%5D-06-ai-integrations-and-agentic-workflows.md) — AI Workflow tasks, Dynatrace MCP server, and external agent integration
- [AIOPS-07: Putting It Together — Detect, Investigate, Remediate](AIOPS%20-%20Dynatrace%20Intelligence/markdown/-%5BAIOPS%5D-07-putting-it-together.md) — Composing all AI categories into the full operational AIOps pattern
- [AIOPS-99: Series Summary](AIOPS%20-%20Dynatrace%20Intelligence/markdown/-%5BAIOPS%5D-99-series-summary.md) — DQL query index, cross-series pointers, and next steps for an AIOps initiative

### [ALERT - Alerting Strategy and Design](ALERT%20-%20Alerting%20Strategy%20and%20Design/README.md)
Designing alerting end to end in Dynatrace — detection, routing, reliability targets, and ITSM integration — orchestrating the AIOPS, SLO, and WFLOW series into one operational pattern.
- [ALERT-01: End-to-End Alerting Architecture](ALERT%20-%20Alerting%20Strategy%20and%20Design/markdown/-%5BALERT%5D-01-end-to-end-architecture.md) — The whole alerting board on one canvas: what to set up, where, and how detection, routing, and reliability connect
- [ALERT-02: Choosing and Building Detection](ALERT%20-%20Alerting%20Strategy%20and%20Design/markdown/-%5BALERT%5D-02-choosing-and-building-detection.md) — Decision framework for the four detection mechanisms — which to use for which signal, and the anti-patterns that cause noise
- [ALERT-03: Routing, Destinations, and Cost](ALERT%20-%20Alerting%20Strategy%20and%20Design/markdown/-%5BALERT%5D-03-routing-destinations-cost.md) — Simple vs multi-step workflow routing, the destination landscape, the legacy alerting-profile path, and cost discipline
- [ALERT-04: ITSM Integration: ServiceNow](ALERT%20-%20Alerting%20Strategy%20and%20Design/markdown/-%5BALERT%5D-04-servicenow-integration.md) — Integration maturity ladder from one-way incident creation to bi-directional sync, with the worked ServiceNow Table API path
- [ALERT-99: Best-Practice Summary and Setup Checklist](ALERT%20-%20Alerting%20Strategy%20and%20Design/markdown/-%5BALERT%5D-99-best-practice-summary.md) — The series on one page: principles, a complete end-to-end setup checklist, and the cross-series build map

### [APPSEC - Application Security](APPSEC%20%E2%80%94%20Application%20Security/README.md)
End-to-end application security observability with Dynatrace — covering Runtime Vulnerability Analytics, Runtime Application Protection, Security Posture Management, and the governance patterns to operationalize AppSec at scale.
- [APPSEC-01: Fundamentals and the Three Pillars of Application Security](APPSEC%20%E2%80%94%20Application%20Security/markdown/-%5BAPPSEC%5D-01-fundamentals.md) — RVA, RAP, and SPM architecture; the Gen3 Grail foundation for security findings
- [APPSEC-02: Runtime Vulnerability Analytics](APPSEC%20%E2%80%94%20Application%20Security/markdown/-%5BAPPSEC%5D-02-runtime-vulnerability-analytics.md) — Third-party library CVE detection, reachability analysis, and risk prioritization
- [APPSEC-03: Code-Level Vulnerability Analytics](APPSEC%20%E2%80%94%20Application%20Security/markdown/-%5BAPPSEC%5D-03-code-level-vulnerability-analytics.md) — First-party code vulnerability detection: patterns, taint analysis, and remediation guidance
- [APPSEC-04: Runtime Application Protection](APPSEC%20%E2%80%94%20Application%20Security/markdown/-%5BAPPSEC%5D-04-runtime-application-protection.md) — Detection-to-blocking promotion, protection rules, and attack class coverage
- [APPSEC-05: Security Posture Management](APPSEC%20%E2%80%94%20Application%20Security/markdown/-%5BAPPSEC%5D-05-security-posture-management.md) — Compliance baseline evaluation (CIS, PCI, NIST) and misconfiguration findings
- [APPSEC-06: Kubernetes and Container Security](APPSEC%20%E2%80%94%20Application%20Security/markdown/-%5BAPPSEC%5D-06-kubernetes-and-container-security.md) — Cross-pillar AppSec coverage for Kubernetes workloads and container images
- [APPSEC-07: Security Investigator and Davis CoPilot for Security](APPSEC%20%E2%80%94%20Application%20Security/markdown/-%5BAPPSEC%5D-07-investigator-and-davis-copilot.md) — Visual entity pivoting and NL-to-DQL for security investigation workflows
- [APPSEC-08: Workflows, Notifications and Remediation](APPSEC%20%E2%80%94%20Application%20Security/markdown/-%5BAPPSEC%5D-08-workflows-notifications-remediation.md) — Routing findings to SOC, AppDev, and platform teams via Dynatrace Workflows
- [APPSEC-09: IAM and Gen3 Permissions for AppSec](APPSEC%20%E2%80%94%20Application%20Security/markdown/-%5BAPPSEC%5D-09-iam-and-gen3-permissions.md) — Permission catalog, persona policies, boundary patterns, and sensitive-data access controls
- [APPSEC-10: Dashboards, Reporting and Governance](APPSEC%20%E2%80%94%20Application%20Security/markdown/-%5BAPPSEC%5D-10-dashboards-reporting-governance.md) — Executive dashboard composition, governance cadence, and FinOps angle for AppSec DPS billing

### [AUTOM - Dynatrace Automation](AUTOM%20-%20Dynatrace%20Automation/README.md)
Comprehensive guide to automating Dynatrace configuration and operations.
- [AUTOM-01: Automation Landscape](AUTOM%20-%20Dynatrace%20Automation/markdown/-%5BAUTOM%5D-01-automation-landscape.md) — Overview of Dynatrace automation options
- [AUTOM-02: Settings API](AUTOM%20-%20Dynatrace%20Automation/markdown/-%5BAUTOM%5D-02-settings-api.md) — Working with the Settings 2.0 API
- [AUTOM-03: Monaco](AUTOM%20-%20Dynatrace%20Automation/markdown/-%5BAUTOM%5D-03-monaco.md) — Configuration as code with Monaco
- [AUTOM-04: Terraform](AUTOM%20-%20Dynatrace%20Automation/markdown/-%5BAUTOM%5D-04-terraform.md) — Infrastructure as code with Terraform provider
- [AUTOM-05: Workflows](AUTOM%20-%20Dynatrace%20Automation/markdown/-%5BAUTOM%5D-05-workflows.md) — Dynatrace Workflows automation
- [AUTOM-06: SDKs](AUTOM%20-%20Dynatrace%20Automation/markdown/-%5BAUTOM%5D-06-sdks.md) — Using Dynatrace SDKs for automation
- [AUTOM-07: CI/CD Integration](AUTOM%20-%20Dynatrace%20Automation/markdown/-%5BAUTOM%5D-07-cicd-integration.md) — Integrating Dynatrace into CI/CD pipelines
- [AUTOM-08: Migration Automation](AUTOM%20-%20Dynatrace%20Automation/markdown/-%5BAUTOM%5D-08-migration-automation.md) — Automating configuration migrations
- [AUTOM-09: Terraform GitOps Setup Recipe](AUTOM%20-%20Dynatrace%20Automation/markdown/-%5BAUTOM%5D-09-terraform-gitops-setup-recipe.md) — Repo layout, state backends, lifecycle protections, team onboarding
- [AUTOM-99: Best Practice Summary](AUTOM%20-%20Dynatrace%20Automation/markdown/-%5BAUTOM%5D-99-best-practice-summary.md) — Consolidated best practices from the AUTOM series

### [BIZEV - Business Events & Funnel Analysis](BIZEV%20-%20Business%20Events%20%26%20Funnel%20Analysis/README.md)
Turning business transactions into first-class observability data with Dynatrace Business Events.
- [BIZEV-01: Business Events Fundamentals](BIZEV%20-%20Business%20Events%20%26%20Funnel%20Analysis/markdown/-%5BBIZEV%5D-01-business-events-fundamentals.md) — Business events data model and architecture
- [BIZEV-02: Instrumentation](BIZEV%20-%20Business%20Events%20%26%20Funnel%20Analysis/markdown/-%5BBIZEV%5D-02-instrumentation.md) — Capturing business transaction events
- [BIZEV-03: Funnel Analysis](BIZEV%20-%20Business%20Events%20%26%20Funnel%20Analysis/markdown/-%5BBIZEV%5D-03-funnel-analysis.md) — Analyzing conversion and drop-off patterns
- [BIZEV-04: Revenue Impact Analysis](BIZEV%20-%20Business%20Events%20%26%20Funnel%20Analysis/markdown/-%5BBIZEV%5D-04-revenue-impact.md) — Quantifying business impact
- [BIZEV-05: KPIs and Metrics](BIZEV%20-%20Business%20Events%20%26%20Funnel%20Analysis/markdown/-%5BBIZEV%5D-05-kpis-and-metrics.md) — Business metrics and dashboarding
- [BIZEV-06: Executive Reporting](BIZEV%20-%20Business%20Events%20%26%20Funnel%20Analysis/markdown/-%5BBIZEV%5D-06-executive-reporting.md) — Creating business-focused reports
- [BIZEV-99: Best Practice Summary](BIZEV%20-%20Business%20Events%20%26%20Funnel%20Analysis/markdown/-%5BBIZEV%5D-99-best-practice-summary.md) — Consolidated best practices from the BIZEV series

### [CLOUD - Cloud Provider Integrations](CLOUD%20-%20Cloud%20Provider%20Integrations/README.md)
Monitoring cloud infrastructure and services across AWS, Azure, and GCP with Dynatrace.
- [CLOUD-01: Cloud Integration Fundamentals](CLOUD%20-%20Cloud%20Provider%20Integrations/markdown/-%5BCLOUD%5D-01-cloud-integration-fundamentals.md) — Architecture and integration patterns
- [CLOUD-02: AWS Integration](CLOUD%20-%20Cloud%20Provider%20Integrations/markdown/-%5BCLOUD%5D-02-aws-integration.md) — Connecting AWS accounts and services
- [CLOUD-03: AWS EKS Monitoring](CLOUD%20-%20Cloud%20Provider%20Integrations/markdown/-%5BCLOUD%5D-03-aws-eks-monitoring.md) — Monitoring Kubernetes on EKS
- [CLOUD-04: AWS Lambda & Serverless Monitoring](CLOUD%20-%20Cloud%20Provider%20Integrations/markdown/-%5BCLOUD%5D-04-aws-lambda-serverless.md) — Observability for serverless workloads
- [CLOUD-05: Azure Integration](CLOUD%20-%20Cloud%20Provider%20Integrations/markdown/-%5BCLOUD%5D-05-azure-integration.md) — Connecting Azure subscriptions
- [CLOUD-06: GCP Integration](CLOUD%20-%20Cloud%20Provider%20Integrations/markdown/-%5BCLOUD%5D-06-gcp-integration.md) — Connecting Google Cloud projects
- [CLOUD-07: CloudWatch Log Ingestion](CLOUD%20-%20Cloud%20Provider%20Integrations/markdown/-%5BCLOUD%5D-07-cloudwatch-log-ingestion.md) — Ingesting CloudWatch logs via OpenPipeline
- [CLOUD-08: Multi-Cloud Observability Patterns](CLOUD%20-%20Cloud%20Provider%20Integrations/markdown/-%5BCLOUD%5D-08-multi-cloud-patterns.md) — Managing multi-cloud strategies
- [CLOUD-99: Best Practice Summary](CLOUD%20-%20Cloud%20Provider%20Integrations/markdown/-%5BCLOUD%5D-99-best-practice-summary.md) — Consolidated best practices from the CLOUD series

### [DASH - Dashboard Design & Building](DASH%20-%20Dashboard%20Design%20%26%20Building/README.md)
Designing and building effective dashboards for different stakeholder audiences.
- [DASH-01: Dashboard Fundamentals](DASH%20-%20Dashboard%20Design%20%26%20Building/markdown/-%5BDASH%5D-01-dashboard-fundamentals.md) — Dashboards vs notebooks and core concepts
- [DASH-02: Dashboard Hierarchy](DASH%20-%20Dashboard%20Design%20%26%20Building/markdown/-%5BDASH%5D-02-dashboard-hierarchy.md) — Organizing dashboards for discovery and navigation
- [DASH-03: Executive Dashboards](DASH%20-%20Dashboard%20Design%20%26%20Building/markdown/-%5BDASH%5D-03-executive-dashboards.md) — KPI dashboards for leadership
- [DASH-04: Operations Dashboards](DASH%20-%20Dashboard%20Design%20%26%20Building/markdown/-%5BDASH%5D-04-operations-dashboards.md) — Real-time operational view
- [DASH-05: Engineering Dashboards](DASH%20-%20Dashboard%20Design%20%26%20Building/markdown/-%5BDASH%5D-05-engineering-dashboards.md) — Technical deep-dive dashboards
- [DASH-06: Variables and Filters](DASH%20-%20Dashboard%20Design%20%26%20Building/markdown/-%5BDASH%5D-06-variables-and-filters.md) — Dynamic dashboard interactions
- [DASH-07: Sharing and Reporting](DASH%20-%20Dashboard%20Design%20%26%20Building/markdown/-%5BDASH%5D-07-sharing-and-reporting.md) — Distribution and scheduled exports
- [DASH-99: Best Practice Summary](DASH%20-%20Dashboard%20Design%20%26%20Building/markdown/-%5BDASH%5D-99-best-practice-summary.md) — Consolidated best practices from the DASH series

### [DBMON - Database Monitoring](DBMON%20-%20Database%20Monitoring/README.md)
End-to-end monitoring of SQL, NoSQL, cache, and messaging database platforms.
- [DBMON-01: Database Monitoring Fundamentals](DBMON%20-%20Database%20Monitoring/markdown/-%5BDBMON%5D-01-database-monitoring-fundamentals.md) — How Dynatrace monitors databases
- [DBMON-02: SQL Database Monitoring](DBMON%20-%20Database%20Monitoring/markdown/-%5BDBMON%5D-02-sql-databases.md) — SQL Server, PostgreSQL, Oracle, MySQL
- [DBMON-03: NoSQL Database Monitoring](DBMON%20-%20Database%20Monitoring/markdown/-%5BDBMON%5D-03-nosql-databases.md) — MongoDB, DynamoDB, Cassandra, and more
- [DBMON-04: Cache and Messaging Monitoring](DBMON%20-%20Database%20Monitoring/markdown/-%5BDBMON%5D-04-cache-and-messaging.md) — Redis, Memcached, RabbitMQ, Kafka
- [DBMON-05: Query Analysis](DBMON%20-%20Database%20Monitoring/markdown/-%5BDBMON%5D-05-query-analysis.md) — Performance analysis and optimization
- [DBMON-06: Dashboards and Alerting](DBMON%20-%20Database%20Monitoring/markdown/-%5BDBMON%5D-06-dashboards-and-alerting.md) — Database performance dashboards
- [DBMON-99: Best Practice Summary](DBMON%20-%20Database%20Monitoring/markdown/-%5BDBMON%5D-99-best-practice-summary.md) — Consolidated best practices from the DBMON series

### [FAQ - Frequently Asked Questions](FAQ%20-%20Frequently%20Asked%20Questions/README.md)
Frequently asked questions and answers across the Dynatrace Best-Practice Notebooks series.
- [FAQ-01: Why you need a good Host Group naming strategy](FAQ%20-%20Frequently%20Asked%20Questions/markdown/-%5BFAQ%5D-01-host-group-naming-strategy.md) — How host group naming influences access control, alerting, automation, and tenant maintainability
- [FAQ-02: Tagging — Sources, Standards, and Strategy](FAQ%20-%20Frequently%20Asked%20Questions/markdown/-%5BFAQ%5D-02-tagging-sources-standards-strategy.md) — The four tag sources, primary tags vs. ordinary tags, and the standards that turn tag sprawl into a managed asset
- [FAQ-03: OneAgent vs OpenTelemetry — A Decision Framework](FAQ%20-%20Frequently%20Asked%20Questions/markdown/-%5BFAQ%5D-03-oneagent-vs-otel-decision-framework.md) — Cross-runtime decision framework for convert vs. layer vs. leave-alone vs. greenfield, covering Java, .NET, Node.js, Python, Go, PHP, Ruby, with async context propagation edge cases and JVM dynamic-attach guidance
- [FAQ-04: Managing OneAgent Updates on SaaS](FAQ%20-%20Frequently%20Asked%20Questions/markdown/-%5BFAQ%5D-04-managing-oneagent-updates-saas.md) — Update mechanism, the three modes, tenant/host-group/host precedence, update vs. maintenance windows, the DynaKube `autoUpdate` deprecation, validation, rollback, and common pitfalls
- [FAQ-05: Managing ActiveGate Updates on SaaS](FAQ%20-%20Frequently%20Asked%20Questions/markdown/-%5BFAQ%5D-05-managing-activegate-updates-saas.md) — Auto-update vs manual decision, ActiveGates-before-OneAgents sequencing, HA-pair rolling updates, role-specific considerations (routing, synthetic, EF 2.0, cloud), validation, rollback, and common pitfalls
- [FAQ-06: Can We Trust Davis AI? A Risk and Controls Walkthrough](FAQ%20-%20Frequently%20Asked%20Questions/markdown/-%5BFAQ%5D-06-can-we-trust-davis-ai.md) — Four-surface taxonomy (Causal, Predictive, Generative/CoPilot, AI Observability) with data residency, training boundary, hallucination controls, autonomy limits, audit trail, IAM, compliance posture, and a decision framework for when to lean in vs hold back
- [FAQ-07: How Do I Set Up a Launcher Page? (Default + Persona-Based)](FAQ%20-%20Frequently%20Asked%20Questions/markdown/-%5BFAQ%5D-07-launcher-page-setup.md) — Default launcher configuration plus persona-based launcher patterns for tailoring the entry experience by role
- [FAQ-08: How Does OneAgent Decide Which Logs to Collect?](FAQ%20-%20Frequently%20Asked%20Questions/markdown/-%5BFAQ%5D-08-oneagent-log-autodiscovery.md) — Log auto-discovery rules, what gets picked up automatically vs what needs a custom log source, host scanning, include/exclude patterns, and verification checklist
- [FAQ-09: When Should I Query a Metric Instead of Raw Logs?](FAQ%20-%20Frequently%20Asked%20Questions/markdown/-%5BFAQ%5D-09-metrics-instead-of-log-queries.md) — Decision framework for replacing recurring log aggregations with pre-extracted metrics; covers Grail query economics, OOTB metrics, cardinality, and extraction trade-offs
- [FAQ-10: How Do I Size and Scale ActiveGates?](FAQ%20-%20Frequently%20Asked%20Questions/markdown/-%5BFAQ%5D-10-activegate-sizing-and-scaling.md) — Capability-to-sizing-dimension map, host-based and containerized K8s sizing tables, synthetic node sizing, HA/network-zone survivor-capacity math, and saturation detection via `dt.sfm.active_gate.*` Grail metrics
- [FAQ-11: How Do Metrics Work in Dynatrace?](FAQ%20-%20Frequently%20Asked%20Questions/markdown/-%5BFAQ%5D-11-how-metrics-work.md) — End-to-end metric mechanics: data-point model (gauge summaries + count deltas), Classic vs Grail dual-write backends, all ingest paths (OneAgent, API v2, OTLP, OpenPipeline), storage/rollup/retention, cardinality enforcement, timeseries querying, and DPS billing
- [FAQ-12: Coming from Another Tool — How Partial Enablement Handicaps Your Dynatrace Coverage](FAQ%20-%20Frequently%20Asked%20Questions/markdown/-%5BFAQ%5D-12-cost-of-coverage-gaps.md) — Per-capability habit-to-handicap matrix for migration customers; what each gap costs, when reduced coverage is right, and objection handling

### [FINOPS - Cost Management & FinOps](FINOPS%20-%20Cost%20Management%20%26%20FinOps/README.md)
Best practices for understanding, forecasting, and optimizing Dynatrace Platform Subscription (DPS) consumption.
- [FINOPS-01: DPS Capability Units and Querying Consumption with DQL](FINOPS%20-%20Cost%20Management%20%26%20FinOps/markdown/-%5BFINOPS%5D-01-dps-capability-units-querying-consumption.md) — Capability categories, billing event flow, and DQL patterns for consumption queries
- [FINOPS-02: Forecasting and Anomaly Detection on DPS Consumption](FINOPS%20-%20Cost%20Management%20%26%20FinOps/markdown/-%5BFINOPS%5D-02-forecasting-anomaly-detection-consumption.md) — Burn-rate trajectory, three layers of cost visibility, and anomaly detection architecture
- [FINOPS-03: DPS Consumption Optimization — When to Cut, Tune, or Filter](FINOPS%20-%20Cost%20Management%20%26%20FinOps/markdown/-%5BFINOPS%5D-03-optimization-decision-framework.md) — Optimization decision tree and cut/tune/filter trade-offs

### [IAM - IAM Administration](IAM%20-%20IAM%20Administration/README.md)
Enterprise identity and access management administration for Dynatrace.
- [IAM-01: Governance Foundations](IAM%20-%20IAM%20Administration/markdown/-%5BIAM%5D-01-governance-foundations.md) — IAM governance principles and strategy
- [IAM-02: SSO & Authentication](IAM%20-%20IAM%20Administration/markdown/-%5BIAM%5D-02-sso-authentication.md) — Single sign-on and authentication setup
- [IAM-03: Group Architecture](IAM%20-%20IAM%20Administration/markdown/-%5BIAM%5D-03-group-architecture.md) — Designing group structures
- [IAM-04: Policy Authoring](IAM%20-%20IAM%20Administration/markdown/-%5BIAM%5D-04-policy-authoring.md) — Writing and managing policies
- [IAM-05: Boundary Design](IAM%20-%20IAM%20Administration/markdown/-%5BIAM%5D-05-boundary-design.md) — Defining access boundaries
- [IAM-06: User Lifecycle](IAM%20-%20IAM%20Administration/markdown/-%5BIAM%5D-06-user-lifecycle.md) — Managing user provisioning and deprovisioning
- [IAM-07: Audit & Compliance](IAM%20-%20IAM%20Administration/markdown/-%5BIAM%5D-07-audit-compliance.md) — Auditing access and ensuring compliance
- [IAM-08: Multi-Environment](IAM%20-%20IAM%20Administration/markdown/-%5BIAM%5D-08-multi-environment.md) — IAM across multiple environments
- [IAM-09: Troubleshooting](IAM%20-%20IAM%20Administration/markdown/-%5BIAM%5D-09-troubleshooting.md) — Diagnosing and resolving IAM issues
- [IAM-10: Templated Policy Assignments](IAM%20-%20IAM%20Administration/markdown/-%5BIAM%5D-10-templated-policy-assignments.md) — Policy templates and bulk assignments
- [IAM-11: [WORKSHOP] Policy Persona Simulation](IAM%20-%20IAM%20Administration/markdown/-%5BIAM%5D-11-%5BWORKSHOP%5D-policy-persona.md) — Interactive workshop: simulate policy behavior as different personas
- [IAM-12: API Provisioning & Validation](IAM%20-%20IAM%20Administration/markdown/-%5BIAM%5D-12-api-provisioning-validation.md) — Scripts and DQL for provisioning via Account Management API
- [IAM-99: Best Practice Summary](IAM%20-%20IAM%20Administration/markdown/-%5BIAM%5D-99-best-practice-summary.md) — Consolidated best practices from the IAM series

### [K8S - Kubernetes Monitoring](K8S%20-%20Kubernetes%20Monitoring/README.md)
Best practices for monitoring Kubernetes with Dynatrace.
- [K8S-01: Fundamentals](K8S%20-%20Kubernetes%20Monitoring/markdown/-%5BK8S%5D-01-fundamentals.md) — Core Kubernetes monitoring concepts
- [K8S-02: Dynakube Deployment](K8S%20-%20Kubernetes%20Monitoring/markdown/-%5BK8S%5D-02-dynakube-deployment.md) — Deploying Dynatrace Operator and Dynakube
- [K8S-03: GitOps Dynakube](K8S%20-%20Kubernetes%20Monitoring/markdown/-%5BK8S%5D-03-gitops-dynakube.md) — Managing Dynakube with GitOps
- [K8S-04: Cluster Monitoring](K8S%20-%20Kubernetes%20Monitoring/markdown/-%5BK8S%5D-04-cluster-monitoring.md) — Monitoring Kubernetes clusters
- [K8S-05: Workload Monitoring](K8S%20-%20Kubernetes%20Monitoring/markdown/-%5BK8S%5D-05-workload-monitoring.md) — Monitoring workloads and applications
- [K8S-06: Namespace Organization](K8S%20-%20Kubernetes%20Monitoring/markdown/-%5BK8S%5D-06-namespace-organization.md) — Organizing and filtering by namespace
- [K8S-07: Events and Logs](K8S%20-%20Kubernetes%20Monitoring/markdown/-%5BK8S%5D-07-events-and-logs.md) — Kubernetes events and log monitoring
- [K8S-08: DQL for K8s](K8S%20-%20Kubernetes%20Monitoring/markdown/-%5BK8S%5D-08-dql-for-k8s.md) — Querying Kubernetes data with DQL
- [K8S-09: Troubleshooting](K8S%20-%20Kubernetes%20Monitoring/markdown/-%5BK8S%5D-09-troubleshooting.md) — Diagnosing Kubernetes monitoring issues
- [K8S-10: Metadata Telemetry Enrichment](K8S%20-%20Kubernetes%20Monitoring/markdown/-%5BK8S%5D-10-metadata-telemetry-enrichment.md) — Enriching telemetry with Kubernetes metadata
- [K8S-11: Multi-Tool Coexistence](K8S%20-%20Kubernetes%20Monitoring/markdown/-%5BK8S%5D-11-multi-tool-coexistence.md) — Running Dynatrace alongside other monitoring tools
- [K8S-12: Specialized Monitoring](K8S%20-%20Kubernetes%20Monitoring/markdown/-%5BK8S%5D-12-specialized-monitoring.md) — NGINX Ingress, CSI Driver, and resource tuning
- [K8S-13: Kafka Monitoring with Kpow](K8S%20-%20Kubernetes%20Monitoring/markdown/-%5BK8S%5D-13-kafka-monitoring-with-kpow.md) — Kafka observability using Kpow Prometheus metrics
- [K8S-99: Best Practice Summary](K8S%20-%20Kubernetes%20Monitoring/markdown/-%5BK8S%5D-99-best-practice-summary.md) — Consolidated best practices from the K8S series

### [M2S - Managed to SaaS Migration](M2S%20-%20Managed%20to%20SaaS%20Migration/README.md)
Guide for migrating from Dynatrace Managed to Dynatrace SaaS.
- [M2S-01: Step 1 — Discover: Understand SaaS Differences](M2S%20-%20Managed%20to%20SaaS%20Migration/markdown/-%5BM2S%5D-01-step-1-discover.md) — Understanding why you're migrating, the benefits of SaaS, and inventorying your Managed environment
- [M2S-02: Step 2 — Strategize: Define Your Migration Approach](M2S%20-%20Managed%20to%20SaaS%20Migration/markdown/-%5BM2S%5D-02-step-2-strategize.md) — Planning your migration strategy, phasing, and execution approach
- [M2S-03: Step 3 — Design: Create Target Architecture](M2S%20-%20Managed%20to%20SaaS%20Migration/markdown/-%5BM2S%5D-03-step-3-design.md) — Designing your target SaaS architecture and configuration
- [M2S-04: Step 4 — Prepare: Readiness and Pre-Migration](M2S%20-%20Managed%20to%20SaaS%20Migration/markdown/-%5BM2S%5D-04-step-4-prepare.md) — Assessing readiness and preparing for migration
- [M2S-05: Step 5 — Execute: Migrate Configuration and Agents](M2S%20-%20Managed%20to%20SaaS%20Migration/markdown/-%5BM2S%5D-05-step-5-execute.md) — Migrating configurations and redirecting agents to SaaS
- [M2S-06: Step 6 — Integrate: Reconnect Integrations](M2S%20-%20Managed%20to%20SaaS%20Migration/markdown/-%5BM2S%5D-06-step-6-integrate.md) — Reconnecting cloud integrations and third-party tools
- [M2S-07: Step 7 — Expand: Adopt New SaaS Capabilities](M2S%20-%20Managed%20to%20SaaS%20Migration/markdown/-%5BM2S%5D-07-step-7-expand.md) — Discovering and adopting new SaaS-only capabilities
- [M2S-08: Step 8 — Enable: User Enablement and Communication](M2S%20-%20Managed%20to%20SaaS%20Migration/markdown/-%5BM2S%5D-08-step-8-enable.md) — User training, support, and operations handover
- [M2S-09: Step 9 — Optimize: Validate, Optimize, and Decommission](M2S%20-%20Managed%20to%20SaaS%20Migration/markdown/-%5BM2S%5D-09-step-9-optimize.md) — Validating the migration, optimizing the environment, and decommissioning Managed
- [M2S-99: Best Practice Summary](M2S%20-%20Managed%20to%20SaaS%20Migration/markdown/-%5BM2S%5D-99-best-practice-summary.md) — Definitive reference of all best practices from the M2S series

### [MOBL - Mobile Monitoring](MOBL%20-%20Mobile%20Monitoring/README.md)
Dynatrace mobile Real User Monitoring (RUM) for iOS, Android, and cross-platform frameworks.
- [MOBL-01: Mobile Monitoring Fundamentals](MOBL%20-%20Mobile%20Monitoring/markdown/-%5BMOBL%5D-01-fundamentals.md) — Mobile RUM architecture, supported platforms, and beacon data flow
- [MOBL-02: iOS SDK Setup](MOBL%20-%20Mobile%20Monitoring/markdown/-%5BMOBL%5D-02-sdk-setup-ios.md) — Installing and configuring the Dynatrace SDK for Swift and SwiftUI
- [MOBL-03: Android SDK Setup](MOBL%20-%20Mobile%20Monitoring/markdown/-%5BMOBL%5D-03-sdk-setup-android.md) — Setting up Dynatrace RUM for Android with Gradle
- [MOBL-04: Cross-Platform Frameworks](MOBL%20-%20Mobile%20Monitoring/markdown/-%5BMOBL%5D-04-cross-platform-frameworks.md) — Instrumenting Flutter, React Native, Cordova, Xamarin, and .NET MAUI
- [MOBL-05: User Action Tracking](MOBL%20-%20Mobile%20Monitoring/markdown/-%5BMOBL%5D-05-user-action-tracking.md) — Capturing auto-detected and custom user interactions
- [MOBL-06: Crash Reporting & ANR Detection](MOBL%20-%20Mobile%20Monitoring/markdown/-%5BMOBL%5D-06-crash-reporting.md) — Automatic crash capture, symbolication, and grouping
- [MOBL-07: Network Request Monitoring](MOBL%20-%20Mobile%20Monitoring/markdown/-%5BMOBL%5D-07-network-request-monitoring.md) — HTTP(S) request visibility and timing breakdown
- [MOBL-08: Session Replay for Mobile](MOBL%20-%20Mobile%20Monitoring/markdown/-%5BMOBL%5D-08-session-replay.md) — Visual session recording with privacy masking
- [MOBL-09: Session Properties & Data Privacy](MOBL%20-%20Mobile%20Monitoring/markdown/-%5BMOBL%5D-09-session-properties-and-privacy.md) — Custom session enrichment and GDPR/CCPA compliance
- [MOBL-10: DQL for Mobile Analytics](MOBL%20-%20Mobile%20Monitoring/markdown/-%5BMOBL%5D-10-dql-for-mobile.md) — Query reference for mobile entities, crashes, and performance
- [MOBL-11: Dashboards & Alerting](MOBL%20-%20Mobile%20Monitoring/markdown/-%5BMOBL%5D-11-dashboards-and-alerting.md) — KPI dashboards with anomaly detection and metric event alerts
- [MOBL-12: Advanced Instrumentation](MOBL%20-%20Mobile%20Monitoring/markdown/-%5BMOBL%5D-12-advanced-instrumentation.md) — Custom events, nested actions, A/B testing, and multi-app strategies
- [MOBL-99: Best Practice Summary](MOBL%20-%20Mobile%20Monitoring/markdown/-%5BMOBL%5D-99-best-practice-summary.md) — Consolidated best practices from the MOBL series

### [MZ2POL - Management Zone to Policy Migration](MZ2POL%20-%20Management%20Zone%20to%20Policy%20Migration/README.md)
Tools and guidance for migrating from Management Zones to Policy-based access control.
- [MZ2POL-00: SDK MZ Analysis Tool](MZ2POL%20-%20Management%20Zone%20to%20Policy%20Migration/markdown/-%5BMZ2POL%5D-00-sdk-mz-analysis-tool.md) — Analyze existing management zones
- [MZ2POL-01: Introduction: Why Migrate](MZ2POL%20-%20Management%20Zone%20to%20Policy%20Migration/markdown/-%5BMZ2POL%5D-01-introduction-why-migrate.md) — Benefits and overview
- [MZ2POL-02: Access Control Model](MZ2POL%20-%20Management%20Zone%20to%20Policy%20Migration/markdown/-%5BMZ2POL%5D-02-access-control-model.md) — Policies and access control concepts
- [MZ2POL-03: Assessment & Planning](MZ2POL%20-%20Management%20Zone%20to%20Policy%20Migration/markdown/-%5BMZ2POL%5D-03-assessment-planning.md) — Migration assessment and planning
- [MZ2POL-04: Policies and Boundaries](MZ2POL%20-%20Management%20Zone%20to%20Policy%20Migration/markdown/-%5BMZ2POL%5D-04-policies-and-boundaries.md) — Defining policies and boundaries
- [MZ2POL-05: Segments Implementation](MZ2POL%20-%20Management%20Zone%20to%20Policy%20Migration/markdown/-%5BMZ2POL%5D-05-segments-implementation.md) — Implementing segments
- [MZ2POL-06: Migration Execution](MZ2POL%20-%20Management%20Zone%20to%20Policy%20Migration/markdown/-%5BMZ2POL%5D-06-migration-execution.md) — Executing the migration
- [MZ2POL-07: Validation & Troubleshooting](MZ2POL%20-%20Management%20Zone%20to%20Policy%20Migration/markdown/-%5BMZ2POL%5D-07-validation-troubleshooting.md) — Validating and resolving issues
- [MZ2POL-08: Templated Policies Migration](MZ2POL%20-%20Management%20Zone%20to%20Policy%20Migration/markdown/-%5BMZ2POL%5D-08-templated-policies-migration.md) — Policy templates for bulk MZ migration
- [MZ2POL-99: Best Practice Summary](MZ2POL%20-%20Management%20Zone%20to%20Policy%20Migration/markdown/-%5BMZ2POL%5D-99-best-practice-summary.md) — Consolidated best practices from the MZ2POL series

### [NR2DT - New Relic to Dynatrace Migration Steps](NR2DT%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Steps/README.md)
Step-by-step migration path from New Relic to Dynatrace, from discovery through cutover and decommission.
- [NR2DT-00: Step 0 — Tenant Prerequisites](NR2DT%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Steps/markdown/-%5BNR2DT%5D-00-step-0-prerequisites.md) — Readiness gate confirming the target Dynatrace tenant is fit to receive a New Relic migration
- [NR2DT-01: Step 1 — Discover](NR2DT%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Steps/markdown/-%5BNR2DT%5D-01-step-1-discover.md) — Inventorying the New Relic environment and understanding migration scope
- [NR2DT-02: Step 2 — Strategize](NR2DT%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Steps/markdown/-%5BNR2DT%5D-02-step-2-strategize.md) — Planning the migration strategy, phasing, and execution approach
- [NR2DT-03: Step 3 — Design](NR2DT%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Steps/markdown/-%5BNR2DT%5D-03-step-3-design.md) — Designing the target Dynatrace architecture and configuration
- [NR2DT-04: Step 4 — Translate](NR2DT%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Steps/markdown/-%5BNR2DT%5D-04-step-4-translate.md) — Translating NRQL queries, dashboards, and alerts to Dynatrace equivalents
- [NR2DT-05: Step 5 — Migrate Dashboards & Alerts](NR2DT%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Steps/markdown/-%5BNR2DT%5D-05-step-5-migrate-dashboards-alerts.md) — Porting dashboards and alerting rules
- [NR2DT-06: Step 6 — Migrate Synthetics, SLOs & Workloads](NR2DT%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Steps/markdown/-%5BNR2DT%5D-06-step-6-migrate-synthetics-slos-workloads.md) — Moving synthetic monitors, SLOs, and workload definitions
- [NR2DT-07: Step 7 — Migrate Logs, Tags & Drop Rules](NR2DT%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Steps/markdown/-%5BNR2DT%5D-07-step-7-migrate-logs-tags-drops.md) — Migrating log ingestion, tagging schemes, and drop rules
- [NR2DT-08: Step 8 — Validate](NR2DT%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Steps/markdown/-%5BNR2DT%5D-08-step-8-validate.md) — Parallel-run validation and diff checks between source and target
- [NR2DT-09: Step 9 — Cutover, Rollback & Decommission](NR2DT%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Steps/markdown/-%5BNR2DT%5D-09-step-9-cutover-rollback-decommission.md) — Executing cutover, preparing rollback plans, and decommissioning New Relic
- [NR2DT-99: Best Practice Summary](NR2DT%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Steps/markdown/-%5BNR2DT%5D-99-best-practice-summary.md) — Consolidated best practices from the NR2DT series

### [NRLC - New Relic to Dynatrace Migration Deep Dives](NRLC%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Deep%20Dives/README.md)
Technical deep-dive companion to the NR2DT series — focused reference notebooks for query translation, dashboard migration, alerting, synthetics, SLOs, logs, and validation.
- [NRLC-01: Platform Comparison](NRLC%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Deep%20Dives/markdown/-%5BNRLC%5D-01-platform-comparison.md) — Side-by-side comparison of New Relic and Dynatrace concepts
- [NRLC-02: NRQL → DQL Translation](NRLC%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Deep%20Dives/markdown/-%5BNRLC%5D-02-nrql-to-dql-translation.md) — Translating NRQL queries into DQL with worked examples
- [NRLC-03: Dashboard Migration](NRLC%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Deep%20Dives/markdown/-%5BNRLC%5D-03-dashboard-migration.md) — Rebuilding New Relic dashboards in Dynatrace
- [NRLC-04: Alert & Workflow Migration](NRLC%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Deep%20Dives/markdown/-%5BNRLC%5D-04-alert-workflow-migration.md) — Migrating alert conditions and notification workflows
- [NRLC-05: Synthetic Monitor Migration](NRLC%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Deep%20Dives/markdown/-%5BNRLC%5D-05-synthetic-monitor-migration.md) — Moving synthetic monitors and private locations
- [NRLC-06: SLO & Workload Migration](NRLC%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Deep%20Dives/markdown/-%5BNRLC%5D-06-slo-workload-migration.md) — Porting SLOs and workload definitions
- [NRLC-07: Logs, Tags & Drop Rules](NRLC%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Deep%20Dives/markdown/-%5BNRLC%5D-07-logs-tags-drops.md) — Log pipelines, tagging strategies, and drop-rule equivalents
- [NRLC-08: Validation, Diff & Rollback](NRLC%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Deep%20Dives/markdown/-%5BNRLC%5D-08-validation-diff-rollback.md) — Parallel-run validation, diff checks, and rollback strategy
- [NRLC-09: Toolchain Reference & End-to-End Runbook](NRLC%20-%20New%20Relic%20to%20Dynatrace%20Migration%20Deep%20Dives/markdown/-%5BNRLC%5D-09-toolchain-reference.md) — Tooling reference and end-to-end migration runbook

### [ONBRD - Dynatrace Onboarding](ONBRD%20-%20Dynatrace%20Onboarding/README.md)
Step-by-step onboarding series for new Dynatrace users.
- [ONBRD-00: Architect's Sequence & Dependency Runbook](ONBRD%20-%20Dynatrace%20Onboarding/markdown/-%5BONBRD%5D-00-architect-sequence.md) — Sequence, dependencies, and decisions for architects onboarding a new tenant
- [ONBRD-01: First Steps](ONBRD%20-%20Dynatrace%20Onboarding/markdown/-%5BONBRD%5D-01-first-steps.md) — Getting started with Dynatrace
- [ONBRD-02: IAM and Authentication](ONBRD%20-%20Dynatrace%20Onboarding/markdown/-%5BONBRD%5D-02-iam-and-authentication.md) — Identity and access management
- [ONBRD-03: Deploying ActiveGate](ONBRD%20-%20Dynatrace%20Onboarding/markdown/-%5BONBRD%5D-03-deploying-activegate.md) — Installing and configuring ActiveGate
- [ONBRD-04: Cloud/SaaS Integrations](ONBRD%20-%20Dynatrace%20Onboarding/markdown/-%5BONBRD%5D-04-cloud-saas-integrations.md) — Connecting cloud platforms
- [ONBRD-05: Deploying OneAgent](ONBRD%20-%20Dynatrace%20Onboarding/markdown/-%5BONBRD%5D-05-deploying-oneagent.md) — Deployment strategies
- [ONBRD-06: Organizing Your Environment](ONBRD%20-%20Dynatrace%20Onboarding/markdown/-%5BONBRD%5D-06-organizing-your-environment.md) — Structuring environments and entities
- [ONBRD-07: Understanding Your Data](ONBRD%20-%20Dynatrace%20Onboarding/markdown/-%5BONBRD%5D-07-understanding-your-data.md) — Data model and exploration
- [ONBRD-08: Your First Queries](ONBRD%20-%20Dynatrace%20Onboarding/markdown/-%5BONBRD%5D-08-your-first-queries.md) — Introduction to DQL
- [ONBRD-09: Setting Up Alerts](ONBRD%20-%20Dynatrace%20Onboarding/markdown/-%5BONBRD%5D-09-setting-up-alerts.md) — Alerting and notifications
- [ONBRD-10: Building Dashboards](ONBRD%20-%20Dynatrace%20Onboarding/markdown/-%5BONBRD%5D-10-building-dashboards.md) — Creating stakeholder dashboards
- [ONBRD-99: Best Practice Summary](ONBRD%20-%20Dynatrace%20Onboarding/markdown/-%5BONBRD%5D-99-best-practice-summary.md) — Consolidated best practices from the ONBRD series

### [OPIPE - OpenPipeline Beyond Logs](OPIPE%20-%20OpenPipeline%20Beyond%20Logs/README.md)
Advanced OpenPipeline patterns for spans, metrics, business events, and security events — extending observability beyond log processing.
- [OPIPE-01: OpenPipeline as a Multi-Scope Platform](OPIPE%20-%20OpenPipeline%20Beyond%20Logs/markdown/-%5BOPIPE%5D-01-multi-scope-platform.md) — Beyond logs: processing spans, metrics, and events at ingestion
- [OPIPE-02: Span Processing & Enrichment](OPIPE%20-%20OpenPipeline%20Beyond%20Logs/markdown/-%5BOPIPE%5D-02-span-processing-and-enrichment.md) — Filtering, enriching, and routing distributed traces at ingestion
- [OPIPE-03: Sampling-Aware Metrics](OPIPE%20-%20OpenPipeline%20Beyond%20Logs/markdown/-%5BOPIPE%5D-03-sampling-aware-metrics.md) — Extracting accurate metrics from sampled trace data
- [OPIPE-04: Cardinality Management](OPIPE%20-%20OpenPipeline%20Beyond%20Logs/markdown/-%5BOPIPE%5D-04-cardinality-management.md) — Controlling dimension explosion across all scopes
- [OPIPE-05: Business & Security Event Pipelines](OPIPE%20-%20OpenPipeline%20Beyond%20Logs/markdown/-%5BOPIPE%5D-05-business-and-security-event-pipelines.md) — Processing business transactions and security events at ingestion
- [OPIPE-06: Cross-Scope Design Patterns](OPIPE%20-%20OpenPipeline%20Beyond%20Logs/markdown/-%5BOPIPE%5D-06-cross-scope-design-patterns.md) — Correlating logs, spans, metrics, and events across scopes
- [OPIPE-99: Best Practice Summary](OPIPE%20-%20OpenPipeline%20Beyond%20Logs/markdown/-%5BOPIPE%5D-99-best-practice-summary.md) — Consolidated best practices from the OPIPE series

### [OPLOGS - OpenPipeline Logs](OPLOGS%20-%20OpenPipeline%20Logs/README.md)
End-to-end guidance for managing logs with Dynatrace OpenPipeline.
- [OPLOGS-01: OpenPipeline Fundamentals](OPLOGS%20-%20OpenPipeline%20Logs/markdown/-%5BOPLOGS%5D-01-fundamentals.md) — Core OpenPipeline concepts and the unified data ingestion framework
- [OPLOGS-02: Migration to OpenPipeline](OPLOGS%20-%20OpenPipeline%20Logs/markdown/-%5BOPLOGS%5D-02-migration.md) — Moving from classic log processing and other platforms
- [OPLOGS-03: OpenPipeline Processing](OPLOGS%20-%20OpenPipeline%20Logs/markdown/-%5BOPLOGS%5D-03-pipeline-processing.md) — Transformations and processing stages
- [OPLOGS-04: Buckets & Data Governance](OPLOGS%20-%20OpenPipeline%20Logs/markdown/-%5BOPLOGS%5D-04-buckets-governance.md) — Storage management, retention, and governance
- [OPLOGS-05: Querying & Parsing Logs](OPLOGS%20-%20OpenPipeline%20Logs/markdown/-%5BOPLOGS%5D-05-querying-parsing.md) — DQL query techniques and parsing rules
- [OPLOGS-06: Topology & Entity Context](OPLOGS%20-%20OpenPipeline%20Logs/markdown/-%5BOPLOGS%5D-06-topology.md) — Log topology and relationships to monitored entities
- [OPLOGS-07: Analytics & Dashboards](OPLOGS%20-%20OpenPipeline%20Logs/markdown/-%5BOPLOGS%5D-07-analytics.md) — Advanced log analytics and dashboard patterns
- [OPLOGS-08: Security & Data Protection](OPLOGS%20-%20OpenPipeline%20Logs/markdown/-%5BOPLOGS%5D-08-security.md) — Security, masking, and compliance
- [OPLOGS-99: Best Practice Summary](OPLOGS%20-%20OpenPipeline%20Logs/markdown/-%5BOPLOGS%5D-99-best-practice-summary.md) — Consolidated best practices from the OPLOGS series

### [OPMIG - OpenPipeline Migration](OPMIG%20-%20OpenPipeline%20Migration/README.md)
Structured migration path to Dynatrace OpenPipeline.
- [OPMIG-01: Introduction: Why Migrate](OPMIG%20-%20OpenPipeline%20Migration/markdown/-%5BOPMIG%5D-01-introduction-why-migrate.md) — Benefits and goals
- [OPMIG-02: Architecture & Key Concepts](OPMIG%20-%20OpenPipeline%20Migration/markdown/-%5BOPMIG%5D-02-architecture-key-concepts.md) — OpenPipeline architecture overview
- [OPMIG-03: Migration Assessment & Planning](OPMIG%20-%20OpenPipeline%20Migration/markdown/-%5BOPMIG%5D-03-migration-assessment-planning.md) — Migration plan and readiness
- [OPMIG-04: Pipeline Configuration](OPMIG%20-%20OpenPipeline%20Migration/markdown/-%5BOPMIG%5D-04-pipeline-configuration.md) — Configuring OpenPipeline
- [OPMIG-05: Routing & Buckets](OPMIG%20-%20OpenPipeline%20Migration/markdown/-%5BOPMIG%5D-05-routing-buckets.md) — Routing rules and storage strategy
- [OPMIG-06: Processing & Parsing](OPMIG%20-%20OpenPipeline%20Migration/markdown/-%5BOPMIG%5D-06-processing-parsing.md) — Transformation and parsing rules
- [OPMIG-07: Metric & Event Extraction](OPMIG%20-%20OpenPipeline%20Migration/markdown/-%5BOPMIG%5D-07-metric-event-extraction.md) — Deriving metrics/events from logs
- [OPMIG-08: Security & Masking](OPMIG%20-%20OpenPipeline%20Migration/markdown/-%5BOPMIG%5D-08-security-masking.md) — Security controls and data masking
- [OPMIG-09: Troubleshooting & Validation](OPMIG%20-%20OpenPipeline%20Migration/markdown/-%5BOPMIG%5D-09-troubleshooting-validation.md) — Validation and issue resolution
- [OPMIG-99: Best Practice Summary](OPMIG%20-%20OpenPipeline%20Migration/markdown/-%5BOPMIG%5D-99-best-practice-summary.md) — Consolidated best practices from the OPMIG series

### [ORGNZ - Organize Data: Buckets, Segments, Security](ORGNZ%20-%20Organize%20Data%3A%20Buckets%2C%20Segments%2C%20Security/README.md)
Organizing data in Dynatrace Grail using buckets, segments, and security context.
- [ORGNZ-01: Introduction to Organizing Data in Grail](ORGNZ%20-%20Organize%20Data%3A%20Buckets%2C%20Segments%2C%20Security/markdown/-%5BORGNZ%5D-01-introduction-to-organizing-data.md) — Why data organization matters in Grail
- [ORGNZ-02: Understanding Grail Buckets](ORGNZ%20-%20Organize%20Data%3A%20Buckets%2C%20Segments%2C%20Security/markdown/-%5BORGNZ%5D-02-understanding-grail-buckets.md) — Bucket fundamentals and data types
- [ORGNZ-03: Bucket Strategy and Design](ORGNZ%20-%20Organize%20Data%3A%20Buckets%2C%20Segments%2C%20Security/markdown/-%5BORGNZ%5D-03-bucket-strategy-and-design.md) — Naming conventions and retention planning
- [ORGNZ-04: Permissions in Grail Overview](ORGNZ%20-%20Organize%20Data%3A%20Buckets%2C%20Segments%2C%20Security/markdown/-%5BORGNZ%5D-04-permissions-in-grail.md) — Permission levels and the canonical `dt.security_context` boundary pattern
- [ORGNZ-05: Bucket-Level Access Control](ORGNZ%20-%20Organize%20Data%3A%20Buckets%2C%20Segments%2C%20Security/markdown/-%5BORGNZ%5D-05-bucket-level-access-control.md) — IAM policies for bucket access and when bucket-level isolation fits
- [ORGNZ-06: Security Context](ORGNZ%20-%20Organize%20Data%3A%20Buckets%2C%20Segments%2C%20Security/markdown/-%5BORGNZ%5D-06-security-context.md) — Fine-grained record-level access with dt.security_context
- [ORGNZ-07: Advanced Permission Patterns](ORGNZ%20-%20Organize%20Data%3A%20Buckets%2C%20Segments%2C%20Security/markdown/-%5BORGNZ%5D-07-advanced-permission-patterns.md) — Record and field-level permissions
- [ORGNZ-08: Grail Segments](ORGNZ%20-%20Organize%20Data%3A%20Buckets%2C%20Segments%2C%20Security/markdown/-%5BORGNZ%5D-08-grail-segments.md) — Logical data filtering with segments
- [ORGNZ-09: Enterprise Data Organization Patterns](ORGNZ%20-%20Organize%20Data%3A%20Buckets%2C%20Segments%2C%20Security/markdown/-%5BORGNZ%5D-09-enterprise-patterns.md) — Combining buckets, segments, and security context for enterprise governance
- [ORGNZ-10: Advanced Segment Definitions](ORGNZ%20-%20Organize%20Data%3A%20Buckets%2C%20Segments%2C%20Security/markdown/-%5BORGNZ%5D-10-advanced-segment-definitions.md) — Complex segment filters with multi-include patterns
- [ORGNZ-99: Best Practice Summary & DQL Reference](ORGNZ%20-%20Organize%20Data%3A%20Buckets%2C%20Segments%2C%20Security/markdown/-%5BORGNZ%5D-99-best-practice-summary.md) — Consolidated best practices and validation DQL from the ORGNZ series

### [OTEL - OpenTelemetry Integration](OTEL%20-%20OpenTelemetry%20Integration/README.md)
Integrating OpenTelemetry with Dynatrace for observability.
- [OTEL-01: Fundamentals](OTEL%20-%20OpenTelemetry%20Integration/markdown/-%5BOTEL%5D-01-fundamentals.md) — Core OpenTelemetry concepts
- [OTEL-02: Collector Architecture](OTEL%20-%20OpenTelemetry%20Integration/markdown/-%5BOTEL%5D-02-collector-architecture.md) — Understanding the OTel Collector
- [OTEL-03: Collector Deployment](OTEL%20-%20OpenTelemetry%20Integration/markdown/-%5BOTEL%5D-03-collector-deployment.md) — Deploying and configuring collectors
- [OTEL-04: Trace Instrumentation](OTEL%20-%20OpenTelemetry%20Integration/markdown/-%5BOTEL%5D-04-trace-instrumentation.md) — Instrumenting applications for traces
- [OTEL-05: Metrics Instrumentation](OTEL%20-%20OpenTelemetry%20Integration/markdown/-%5BOTEL%5D-05-metrics-instrumentation.md) — Instrumenting applications for metrics
- [OTEL-06: Logs Instrumentation](OTEL%20-%20OpenTelemetry%20Integration/markdown/-%5BOTEL%5D-06-logs-instrumentation.md) — Instrumenting applications for logs
- [OTEL-07: Dynatrace Integration](OTEL%20-%20OpenTelemetry%20Integration/markdown/-%5BOTEL%5D-07-dynatrace-integration.md) — Sending OTel data to Dynatrace
- [OTEL-08: Troubleshooting](OTEL%20-%20OpenTelemetry%20Integration/markdown/-%5BOTEL%5D-08-troubleshooting.md) — Diagnosing OpenTelemetry issues
- [OTEL-99: Best Practice Summary](OTEL%20-%20OpenTelemetry%20Integration/markdown/-%5BOTEL%5D-99-best-practice-summary.md) — Consolidated best practices from the OTEL series

### [S2D - Splunk to Dynatrace Migration](S2D%20-%20Splunk%20to%20Dynatrace%20Migration/README.md)
Guide for migrating monitoring capabilities from Splunk to Dynatrace.
- [S2D-01: Getting Started](S2D%20-%20Splunk%20to%20Dynatrace%20Migration/markdown/-%5BS2D%5D-01-getting-started.md) — Migration overview and roadmap
- [S2D-02: Locating Logs](S2D%20-%20Splunk%20to%20Dynatrace%20Migration/markdown/-%5BS2D%5D-02-locating-logs.md) — Finding and verifying log data in Dynatrace
- [S2D-03: SPL to DQL Translation](S2D%20-%20Splunk%20to%20Dynatrace%20Migration/markdown/-%5BS2D%5D-03-spl-to-dql.md) — Converting Splunk queries to DQL
- [S2D-04: Davis Anomaly Detectors](S2D%20-%20Splunk%20to%20Dynatrace%20Migration/markdown/-%5BS2D%5D-04-anomaly-detectors.md) — Migrating alerts to continuous monitoring
- [S2D-05: Workflow-Based Alerts](S2D%20-%20Splunk%20to%20Dynatrace%20Migration/markdown/-%5BS2D%5D-05-workflow-alerts.md) — Migrating alerts to scheduled workflows
- [S2D-06: ArrayMovingSum](S2D%20-%20Splunk%20to%20Dynatrace%20Migration/markdown/-%5BS2D%5D-06-arraymovingsum.md) — Rolling sums for extended timeframes
- [S2D-07: Metric Creation](S2D%20-%20Splunk%20to%20Dynatrace%20Migration/markdown/-%5BS2D%5D-07-metric-creation.md) — Extracting metrics from logs via OpenPipeline
- [S2D-08: Dashboard Migration](S2D%20-%20Splunk%20to%20Dynatrace%20Migration/markdown/-%5BS2D%5D-08-dashboard-migration.md) — Converting Splunk dashboards to Dynatrace
- [S2D-09: Naming Standards](S2D%20-%20Splunk%20to%20Dynatrace%20Migration/markdown/-%5BS2D%5D-09-naming-standards.md) — Asset naming conventions and organization
- [S2D-99: Best Practice Summary](S2D%20-%20Splunk%20to%20Dynatrace%20Migration/markdown/-%5BS2D%5D-99-best-practice-summary.md) — Consolidated best practices from the S2D series

### [S2S - SaaS to SaaS Migration](S2S%20-%20SaaS%20to%20SaaS%20Migration/README.md)
Comprehensive guide to migrating between Dynatrace SaaS environments.
- [S2S-01: Step 1 — Discover: Migration Scenarios and Inventory](S2S%20-%20SaaS%20to%20SaaS%20Migration/markdown/-%5BS2S%5D-01-step-1-discover.md) — Understanding why you're migrating, inventorying your source environment, and identifying what migrates automatically
- [S2S-02: Step 2 — Strategize: Define Your Migration Approach](S2S%20-%20SaaS%20to%20SaaS%20Migration/markdown/-%5BS2S%5D-02-step-2-strategize.md) — Planning your migration strategy, phasing, and execution approach
- [S2S-03: Step 3 — Design: Target Tenant Architecture](S2S%20-%20SaaS%20to%20SaaS%20Migration/markdown/-%5BS2S%5D-03-step-3-design.md) — Designing your target SaaS architecture and tenant structure
- [S2S-04: Step 4 — Prepare: Export and Pre-Stage](S2S%20-%20SaaS%20to%20SaaS%20Migration/markdown/-%5BS2S%5D-04-step-4-prepare.md) — Exporting configurations and pre-staging for migration
- [S2S-05: Step 5 — Execute: Configuration Import and Agent Cutover](S2S%20-%20SaaS%20to%20SaaS%20Migration/markdown/-%5BS2S%5D-05-step-5-execute.md) — Importing configuration and redirecting agents to the target environment
- [S2S-06: Step 6 — Integrate: Cloud, Dashboards, and Workflows](S2S%20-%20SaaS%20to%20SaaS%20Migration/markdown/-%5BS2S%5D-06-step-6-integrate.md) — Migrating cloud integrations, dashboards, and workflows
- [S2S-07: Step 7 — Expand: OpenPipeline, SLOs, and Alerting](S2S%20-%20SaaS%20to%20SaaS%20Migration/markdown/-%5BS2S%5D-07-step-7-expand.md) — Configuring data pipelines, SLOs, and alerting rules
- [S2S-08: Step 8 — Enable: Parallel Operation and Stakeholder Handover](S2S%20-%20SaaS%20to%20SaaS%20Migration/markdown/-%5BS2S%5D-08-step-8-enable.md) — Running source and target in parallel and handing over to operations
- [S2S-09: Step 9 — Optimize: Cutover Validation and Decommission](S2S%20-%20SaaS%20to%20SaaS%20Migration/markdown/-%5BS2S%5D-09-step-9-optimize.md) — Validating the migration, optimizing the target, and decommissioning source
- [S2S-10: Migration Scripts](S2S%20-%20SaaS%20to%20SaaS%20Migration/markdown/-%5BS2S%5D-10-migration-scripts.md) — Reusable Bash and PowerShell scripts for Monaco export and SaaS Upgrade Assistant packaging
- [S2S-99: Best Practice Summary](S2S%20-%20SaaS%20to%20SaaS%20Migration/markdown/-%5BS2S%5D-99-best-practice-summary.md) — Comprehensive reference of all best practices from the S2S series

### [SL2DT - Sumo Logic to Dynatrace](SL2DT%20-%20Sumo%20Logic%20to%20Dynatrace/README.md)
Step-by-step migration path from Sumo Logic to Dynatrace, from strategy and inventory through cutover and decommission.
- [SL2DT-01: Overview & Migration Strategy](SL2DT%20-%20Sumo%20Logic%20to%20Dynatrace/markdown/-%5BSL2DT%5D-01-overview-and-migration-strategy.md) — Migration strategy, mental model, and the five-wave pattern
- [SL2DT-02: Assessment & Inventory](SL2DT%20-%20Sumo%20Logic%20to%20Dynatrace/markdown/-%5BSL2DT%5D-02-assessment-and-inventory.md) — Inventorying Sumo Logic and producing the cut-scope decision artifact
- [SL2DT-03: Log Ingest Architecture](SL2DT%20-%20Sumo%20Logic%20to%20Dynatrace/markdown/-%5BSL2DT%5D-03-log-ingest-architecture.md) — Designing Grail buckets and deploying OneAgent, OTel, and OpenPipeline
- [SL2DT-04: SumoQL → DQL Translation](SL2DT%20-%20Sumo%20Logic%20to%20Dynatrace/markdown/-%5BSL2DT%5D-04-sumoql-to-dql-translation.md) — Translating SumoQL queries, dashboards, and monitor conditions to DQL
- [SL2DT-05: Monitor & Alert Conversion](SL2DT%20-%20Sumo%20Logic%20to%20Dynatrace/markdown/-%5BSL2DT%5D-05-monitor-and-alert-conversion.md) — Rebuilding Sumo Monitors using Anomaly Detection, Workflows, or Metric Events
- [SL2DT-06: Dashboard Conversion](SL2DT%20-%20Sumo%20Logic%20to%20Dynatrace/markdown/-%5BSL2DT%5D-06-dashboard-conversion.md) — Rebuilding in-scope Sumo dashboards in Dynatrace Notebooks and Dashboards
- [SL2DT-07: User Governance & Access](SL2DT%20-%20Sumo%20Logic%20to%20Dynatrace/markdown/-%5BSL2DT%5D-07-user-governance-and-access.md) — Translating Sumo RBAC to Dynatrace Platform IAM groups and policies
- [SL2DT-08: Automation & GitOps](SL2DT%20-%20Sumo%20Logic%20to%20Dynatrace/markdown/-%5BSL2DT%5D-08-automation-and-gitops.md) — CI/CD promotion paths for all migrated configuration via Monaco
- [SL2DT-09: Cutover, Validation & Decommission](SL2DT%20-%20Sumo%20Logic%20to%20Dynatrace/markdown/-%5BSL2DT%5D-09-cutover-validation-decommission.md) — Parallel validation, declaring cutover, and decommissioning Sumo
- [SL2DT-99: Summary & Runbook Index](SL2DT%20-%20Sumo%20Logic%20to%20Dynatrace/markdown/-%5BSL2DT%5D-99-summary-and-runbook-index.md) — Single-page index and reference card for the entire SL2DT series

### [SLO - Service Level Objectives](SLO%20-%20Service%20Level%20Objectives/README.md)
Defining and operating Service Level Objectives in Dynatrace — SLIs, error budgets, composition, burn-rate alerting, and SLOs as code.
- [SLO-01: SLO and SLI Fundamentals](SLO%20-%20Service%20Level%20Objectives/markdown/-%5BSLO%5D-01-fundamentals.md) — The three building blocks (SLI, SLO, error budget), the Dynatrace SLO model, and choosing your first SLOs
- [SLO-02: Defining SLIs](SLO%20-%20Service%20Level%20Objectives/markdown/-%5BSLO%5D-02-defining-slis.md) — The good ÷ total ratio as DQL: availability, latency, error-rate, and custom business SLIs, validated on a live tenant
- [SLO-03: Composition and Error Budgets](SLO%20-%20Service%20Level%20Objectives/markdown/-%5BSLO%5D-03-composition-and-error-budgets.md) — Error-budget math and burn rate, composite and weighted-global SLOs, and rolling vs calendar windows
- [SLO-04: SLO Alerting](SLO%20-%20Service%20Level%20Objectives/markdown/-%5BSLO%5D-04-alerting.md) — Burn-rate alerting over threshold-on-SLI: fast-burn and slow-burn multiwindow alerts, surfacing, routing, and avoiding fatigue
- [SLO-05: SLOs as Code](SLO%20-%20Service%20Level%20Objectives/markdown/-%5BSLO%5D-05-slos-as-code.md) — Promoting SLOs into version control: the `builtin:slo` schema, the `dynatrace_slo_v2` Terraform resource, Monaco, and the API/CI-CD path

### [SPANS - Distributed Tracing and Spans](SPANS%20-%20Distributed%20Tracing%20and%20Spans/README.md)
Guidance for working with distributed traces and spans in Dynatrace.
- [SPANS-01: Fundamentals](SPANS%20-%20Distributed%20Tracing%20and%20Spans/markdown/-%5BSPANS%5D-01-fundamentals.md) — Core tracing concepts
- [SPANS-02: Querying](SPANS%20-%20Distributed%20Tracing%20and%20Spans/markdown/-%5BSPANS%5D-02-querying.md) — Querying spans and traces with DQL
- [SPANS-03: Troubleshooting](SPANS%20-%20Distributed%20Tracing%20and%20Spans/markdown/-%5BSPANS%5D-03-troubleshooting.md) — Using traces for issue investigation
- [SPANS-04: Topology](SPANS%20-%20Distributed%20Tracing%20and%20Spans/markdown/-%5BSPANS%5D-04-topology.md) — Service topology from traces
- [SPANS-05: Analytics](SPANS%20-%20Distributed%20Tracing%20and%20Spans/markdown/-%5BSPANS%5D-05-analytics.md) — Advanced trace analytics
- [SPANS-06: Security](SPANS%20-%20Distributed%20Tracing%20and%20Spans/markdown/-%5BSPANS%5D-06-security.md) — Span security and data protection
- [SPANS-07: Buckets & Pipeline](SPANS%20-%20Distributed%20Tracing%20and%20Spans/markdown/-%5BSPANS%5D-07-buckets-pipeline.md) — Storage and processing
- [SPANS-08: Cost Optimization](SPANS%20-%20Distributed%20Tracing%20and%20Spans/markdown/-%5BSPANS%5D-08-cost-optimization.md) — Optimizing trace ingestion costs
- [SPANS-99: Best Practice Summary](SPANS%20-%20Distributed%20Tracing%20and%20Spans/markdown/-%5BSPANS%5D-99-best-practice-summary.md) — Consolidated best practices from the SPANS series

### [SYNTH - Synthetic Monitoring](SYNTH%20-%20Synthetic%20Monitoring/README.md)
Series covering Dynatrace Synthetic Monitoring.
- [SYNTH-01: Fundamentals](SYNTH%20-%20Synthetic%20Monitoring/markdown/-%5BSYNTH%5D-01-fundamentals.md) — Core synthetic monitoring concepts
- [SYNTH-02: Browser Monitors](SYNTH%20-%20Synthetic%20Monitoring/markdown/-%5BSYNTH%5D-02-browser-monitors.md) — Creating and managing browser monitors
- [SYNTH-03: HTTP Monitors](SYNTH%20-%20Synthetic%20Monitoring/markdown/-%5BSYNTH%5D-03-http-monitors.md) — HTTP/API monitors
- [SYNTH-04: Private Locations](SYNTH%20-%20Synthetic%20Monitoring/markdown/-%5BSYNTH%5D-04-private-locations.md) — Deploying private synthetic locations
- [SYNTH-05: Network Monitoring](SYNTH%20-%20Synthetic%20Monitoring/markdown/-%5BSYNTH%5D-05-network-monitoring.md) — Network performance monitoring
- [SYNTH-06: Analytics](SYNTH%20-%20Synthetic%20Monitoring/markdown/-%5BSYNTH%5D-06-analytics.md) — Analyzing synthetic monitoring data
- [SYNTH-99: Best Practice Summary](SYNTH%20-%20Synthetic%20Monitoring/markdown/-%5BSYNTH%5D-99-best-practice-summary.md) — Consolidated best practices from the SYNTH series

### [WEBRUM - Web Real User Monitoring](WEBRUM%20-%20Web%20Real%20User%20Monitoring/README.md)
Client-side monitoring and observability for web applications with session replay and performance metrics.
- [WEBRUM-01: Web RUM Fundamentals](WEBRUM%20-%20Web%20Real%20User%20Monitoring/markdown/-%5BWEBRUM%5D-01-rum-fundamentals.md) — RUM architecture and data collection
- [WEBRUM-02: SPA Instrumentation](WEBRUM%20-%20Web%20Real%20User%20Monitoring/markdown/-%5BWEBRUM%5D-02-spa-instrumentation.md) — Monitoring single-page applications
- [WEBRUM-03: Core Web Vitals](WEBRUM%20-%20Web%20Real%20User%20Monitoring/markdown/-%5BWEBRUM%5D-03-core-web-vitals.md) — LCP, FID, CLS metrics and optimization
- [WEBRUM-04: Session Analysis](WEBRUM%20-%20Web%20Real%20User%20Monitoring/markdown/-%5BWEBRUM%5D-04-session-analysis.md) — User session insights and funnels
- [WEBRUM-05: Error Analysis](WEBRUM%20-%20Web%20Real%20User%20Monitoring/markdown/-%5BWEBRUM%5D-05-error-analysis.md) — JavaScript errors and crash detection
- [WEBRUM-06: Performance Analysis](WEBRUM%20-%20Web%20Real%20User%20Monitoring/markdown/-%5BWEBRUM%5D-06-performance-analysis.md) — Page load and runtime performance
- [WEBRUM-07: Session Replay](WEBRUM%20-%20Web%20Real%20User%20Monitoring/markdown/-%5BWEBRUM%5D-07-session-replay.md) — Visual reproduction of user sessions
- [WEBRUM-08: Dashboards and Alerting](WEBRUM%20-%20Web%20Real%20User%20Monitoring/markdown/-%5BWEBRUM%5D-08-dashboards-and-alerting.md) — RUM metrics dashboards and alerts
- [WEBRUM-99: Best Practice Summary](WEBRUM%20-%20Web%20Real%20User%20Monitoring/markdown/-%5BWEBRUM%5D-99-best-practice-summary.md) — Consolidated best practices from the WEBRUM series

### [WFLOW - Workflows and Alert Notifications](WFLOW%20-%20Workflows%20and%20Alert%20Notifications/README.md)
Automating workflows and configuring alert notifications in Dynatrace.
- [WFLOW-01: Fundamentals](WFLOW%20-%20Workflows%20and%20Alert%20Notifications/markdown/-%5BWFLOW%5D-01-fundamentals.md) — Core workflow concepts
- [WFLOW-02: Triggers](WFLOW%20-%20Workflows%20and%20Alert%20Notifications/markdown/-%5BWFLOW%5D-02-triggers.md) — Workflow trigger configuration
- [WFLOW-03: Notification Basics](WFLOW%20-%20Workflows%20and%20Alert%20Notifications/markdown/-%5BWFLOW%5D-03-notification-basics.md) — Setting up basic notifications
- [WFLOW-04: Notification Routing](WFLOW%20-%20Workflows%20and%20Alert%20Notifications/markdown/-%5BWFLOW%5D-04-notification-routing.md) — Advanced notification routing
- [WFLOW-05: Incident Management](WFLOW%20-%20Workflows%20and%20Alert%20Notifications/markdown/-%5BWFLOW%5D-05-incident-management.md) — Integrating with incident management
- [WFLOW-06: Custom Templates](WFLOW%20-%20Workflows%20and%20Alert%20Notifications/markdown/-%5BWFLOW%5D-06-custom-templates.md) — Creating custom notification templates
- [WFLOW-07: Remediation](WFLOW%20-%20Workflows%20and%20Alert%20Notifications/markdown/-%5BWFLOW%5D-07-remediation.md) — Automated remediation workflows
- [WFLOW-08: JavaScript & HTTP](WFLOW%20-%20Workflows%20and%20Alert%20Notifications/markdown/-%5BWFLOW%5D-08-javascript-http.md) — JavaScript actions and HTTP requests
- [WFLOW-09: Governance](WFLOW%20-%20Workflows%20and%20Alert%20Notifications/markdown/-%5BWFLOW%5D-09-governance.md) — Workflow governance and best practices
- [WFLOW-99: Best Practice Summary](WFLOW%20-%20Workflows%20and%20Alert%20Notifications/markdown/-%5BWFLOW%5D-99-best-practice-summary.md) — Consolidated best practices from the WFLOW series

## How to Use

1. Open a topic README for context and prerequisites.
2. Choose your format:
	- Import JSON from notebooks/ into Dynatrace Notebooks for interactive use.
	- Read pdfs/ for printable versions.
	- Use markdown/ for lightweight viewing or diffs.
3. Follow the numbered sequence to progress through each series.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
