# Best-Practice-Notebooks

Dynatrace best-practice notebooks with matching PDF and Markdown exports. These assets are not officially supported by Dynatrace.

> **Recommended:** Import the JSON files from NOTEBOOKS/ into a Dynatrace tenant for the best experience. These notebooks contain interactive DQL queries that execute against your environment's data.

Want to contribute or understand how pull requests are reviewed? See [CONTRIBUTING.md](CONTRIBUTING.md).

## Layout

Each topic follows the same structure:
- NOTEBOOKS/ — Dynatrace notebook JSON files (import into Dynatrace)
- PDFs/ — Printable/exported versions of the notebooks
- markdown/ — Markdown exports of the same content
- README.md — Topic overview and usage guide

## Table of Contents

### [ADOPT - Observability adoption & maturity](ADOPT%20-%20Observability%20adoption%20%26%20maturity/README.md)
Framework for assessing and advancing Dynatrace observability adoption across the organization.
- [ADOPT-01: Observability Maturity Model](ADOPT%20-%20Observability%20adoption%20%26%20maturity/markdown/-%5BADOPT%5D-01-maturity-model.md) — Five-level maturity framework
- [ADOPT-02: Platform Health Assessment](ADOPT%20-%20Observability%20adoption%20%26%20maturity/markdown/-%5BADOPT%5D-02-platform-health-assessment.md) — Evaluating current state and readiness
- [ADOPT-03: Success Metrics](ADOPT%20-%20Observability%20adoption%20%26%20maturity/markdown/-%5BADOPT%5D-03-success-metrics.md) — Defining KPIs and measuring adoption
- [ADOPT-04: Team Enablement](ADOPT%20-%20Observability%20adoption%20%26%20maturity/markdown/-%5BADOPT%5D-04-team-enablement.md) — Training and organization strategies
- [ADOPT-05: Optimization Roadmap](ADOPT%20-%20Observability%20adoption%20%26%20maturity/markdown/-%5BADOPT%5D-05-optimization-roadmap.md) — Building your adoption roadmap

### [AUTOM - Dynatrace automation](AUTOM%20-%20Dynatrace%20automation/README.md)
Comprehensive guide to automating Dynatrace configuration and operations.
- [AUTOM-01: Automation Landscape](AUTOM%20-%20Dynatrace%20automation/markdown/-%5BAUTOM%5D-01-automation-landscape.md) — Overview of Dynatrace automation options
- [AUTOM-02: Settings API](AUTOM%20-%20Dynatrace%20automation/markdown/-%5BAUTOM%5D-02-settings-api.md) — Working with the Settings 2.0 API
- [AUTOM-03: Monaco](AUTOM%20-%20Dynatrace%20automation/markdown/-%5BAUTOM%5D-03-monaco.md) — Configuration as code with Monaco
- [AUTOM-04: Terraform](AUTOM%20-%20Dynatrace%20automation/markdown/-%5BAUTOM%5D-04-terraform.md) — Infrastructure as code with Terraform provider
- [AUTOM-05: Workflows](AUTOM%20-%20Dynatrace%20automation/markdown/-%5BAUTOM%5D-05-workflows.md) — Dynatrace Workflows automation
- [AUTOM-06: SDKs](AUTOM%20-%20Dynatrace%20automation/markdown/-%5BAUTOM%5D-06-sdks.md) — Using Dynatrace SDKs for automation
- [AUTOM-07: CI/CD Integration](AUTOM%20-%20Dynatrace%20automation/markdown/-%5BAUTOM%5D-07-cicd-integration.md) — Integrating Dynatrace into CI/CD pipelines
- [AUTOM-08: Migration Automation](AUTOM%20-%20Dynatrace%20automation/markdown/-%5BAUTOM%5D-08-migration-automation.md) — Automating configuration migrations

### [BIZEV - Business events & funnel analysis](BIZEV%20-%20Business%20events%20%26%20funnel%20analysis/README.md)
Turning business transactions into first-class observability data with Dynatrace Business Events.
- [BIZEV-01: Business Events Fundamentals](BIZEV%20-%20Business%20events%20%26%20funnel%20analysis/markdown/-%5BBIZEV%5D-01-business-events-fundamentals.md) — Business events data model and architecture
- [BIZEV-02: Instrumentation](BIZEV%20-%20Business%20events%20%26%20funnel%20analysis/markdown/-%5BBIZEV%5D-02-instrumentation.md) — Capturing business transaction events
- [BIZEV-03: Funnel Analysis](BIZEV%20-%20Business%20events%20%26%20funnel%20analysis/markdown/-%5BBIZEV%5D-03-funnel-analysis.md) — Analyzing conversion and drop-off patterns
- [BIZEV-04: Revenue Impact Analysis](BIZEV%20-%20Business%20events%20%26%20funnel%20analysis/markdown/-%5BBIZEV%5D-04-revenue-impact.md) — Quantifying business impact
- [BIZEV-05: KPIs and Metrics](BIZEV%20-%20Business%20events%20%26%20funnel%20analysis/markdown/-%5BBIZEV%5D-05-kpis-and-metrics.md) — Business metrics and dashboarding
- [BIZEV-06: Executive Reporting](BIZEV%20-%20Business%20events%20%26%20funnel%20analysis/markdown/-%5BBIZEV%5D-06-executive-reporting.md) — Creating business-focused reports

### [CLOUD - Cloud provider integrations](CLOUD%20-%20Cloud%20provider%20integrations/README.md)
Monitoring cloud infrastructure and services across AWS, Azure, and GCP with Dynatrace.
- [CLOUD-01: Cloud Integration Fundamentals](CLOUD%20-%20Cloud%20provider%20integrations/markdown/-%5BCLOUD%5D-01-cloud-integration-fundamentals.md) — Architecture and integration patterns
- [CLOUD-02: AWS Integration](CLOUD%20-%20Cloud%20provider%20integrations/markdown/-%5BCLOUD%5D-02-aws-integration.md) — Connecting AWS accounts and services
- [CLOUD-03: AWS EKS Monitoring](CLOUD%20-%20Cloud%20provider%20integrations/markdown/-%5BCLOUD%5D-03-aws-eks-monitoring.md) — Monitoring Kubernetes on EKS
- [CLOUD-04: AWS Lambda & Serverless Monitoring](CLOUD%20-%20Cloud%20provider%20integrations/markdown/-%5BCLOUD%5D-04-aws-lambda-serverless.md) — Observability for serverless workloads
- [CLOUD-05: Azure Integration](CLOUD%20-%20Cloud%20provider%20integrations/markdown/-%5BCLOUD%5D-05-azure-integration.md) — Connecting Azure subscriptions
- [CLOUD-06: GCP Integration](CLOUD%20-%20Cloud%20provider%20integrations/markdown/-%5BCLOUD%5D-06-gcp-integration.md) — Connecting Google Cloud projects
- [CLOUD-07: CloudWatch Log Ingestion](CLOUD%20-%20Cloud%20provider%20integrations/markdown/-%5BCLOUD%5D-07-cloudwatch-log-ingestion.md) — Ingesting CloudWatch logs via OpenPipeline
- [CLOUD-08: Multi-Cloud Observability Patterns](CLOUD%20-%20Cloud%20provider%20integrations/markdown/-%5BCLOUD%5D-08-multi-cloud-patterns.md) — Managing multi-cloud strategies

### [DASH - Dashboard design & building](DASH%20-%20Dashboard%20design%20%26%20building/README.md)
Designing and building effective dashboards for different stakeholder audiences.
- [DASH-01: Dashboard Fundamentals](DASH%20-%20Dashboard%20design%20%26%20building/markdown/-%5BDASH%5D-01-dashboard-fundamentals.md) — Dashboards vs notebooks and core concepts
- [DASH-02: Dashboard Hierarchy](DASH%20-%20Dashboard%20design%20%26%20building/markdown/-%5BDASH%5D-02-dashboard-hierarchy.md) — Organizing dashboards for discovery and navigation
- [DASH-03: Executive Dashboards](DASH%20-%20Dashboard%20design%20%26%20building/markdown/-%5BDASH%5D-03-executive-dashboards.md) — KPI dashboards for leadership
- [DASH-04: Operations Dashboards](DASH%20-%20Dashboard%20design%20%26%20building/markdown/-%5BDASH%5D-04-operations-dashboards.md) — Real-time operational view
- [DASH-05: Engineering Dashboards](DASH%20-%20Dashboard%20design%20%26%20building/markdown/-%5BDASH%5D-05-engineering-dashboards.md) — Technical deep-dive dashboards
- [DASH-06: Variables and Filters](DASH%20-%20Dashboard%20design%20%26%20building/markdown/-%5BDASH%5D-06-variables-and-filters.md) — Dynamic dashboard interactions
- [DASH-07: Sharing and Reporting](DASH%20-%20Dashboard%20design%20%26%20building/markdown/-%5BDASH%5D-07-sharing-and-reporting.md) — Distribution and scheduled exports

### [DBMON - Database monitoring](DBMON%20-%20Database%20monitoring/README.md)
End-to-end monitoring of SQL, NoSQL, cache, and messaging database platforms.
- [DBMON-01: Database Monitoring Fundamentals](DBMON%20-%20Database%20monitoring/markdown/-%5BDBMON%5D-01-database-monitoring-fundamentals.md) — How Dynatrace monitors databases
- [DBMON-02: SQL Database Monitoring](DBMON%20-%20Database%20monitoring/markdown/-%5BDBMON%5D-02-sql-databases.md) — SQL Server, PostgreSQL, Oracle, MySQL
- [DBMON-03: NoSQL Database Monitoring](DBMON%20-%20Database%20monitoring/markdown/-%5BDBMON%5D-03-nosql-databases.md) — MongoDB, DynamoDB, Cassandra, and more
- [DBMON-04: Cache and Messaging Monitoring](DBMON%20-%20Database%20monitoring/markdown/-%5BDBMON%5D-04-cache-and-messaging.md) — Redis, Memcached, RabbitMQ, Kafka
- [DBMON-05: Query Analysis](DBMON%20-%20Database%20monitoring/markdown/-%5BDBMON%5D-05-query-analysis.md) — Performance analysis and optimization
- [DBMON-06: Dashboards and Alerting](DBMON%20-%20Database%20monitoring/markdown/-%5BDBMON%5D-06-dashboards-and-alerting.md) — Database performance dashboards

### [IAM - IAM administration](IAM%20-%20IAM%20administration/README.md)
Enterprise identity and access management administration for Dynatrace.
- [IAM-01: Governance Foundations](IAM%20-%20IAM%20administration/markdown/-%5BIAM%5D-01-governance-foundations.md) — IAM governance principles and strategy
- [IAM-02: SSO & Authentication](IAM%20-%20IAM%20administration/markdown/-%5BIAM%5D-02-sso-authentication.md) — Single sign-on and authentication setup
- [IAM-03: Group Architecture](IAM%20-%20IAM%20administration/markdown/-%5BIAM%5D-03-group-architecture.md) — Designing group structures
- [IAM-04: Policy Authoring](IAM%20-%20IAM%20administration/markdown/-%5BIAM%5D-04-policy-authoring.md) — Writing and managing policies
- [IAM-05: Boundary Design](IAM%20-%20IAM%20administration/markdown/-%5BIAM%5D-05-boundary-design.md) — Defining access boundaries
- [IAM-06: User Lifecycle](IAM%20-%20IAM%20administration/markdown/-%5BIAM%5D-06-user-lifecycle.md) — Managing user provisioning and deprovisioning
- [IAM-07: Audit & Compliance](IAM%20-%20IAM%20administration/markdown/-%5BIAM%5D-07-audit-compliance.md) — Auditing access and ensuring compliance
- [IAM-08: Multi-Environment](IAM%20-%20IAM%20administration/markdown/-%5BIAM%5D-08-multi-environment.md) — IAM across multiple environments
- [IAM-09: Troubleshooting](IAM%20-%20IAM%20administration/markdown/-%5BIAM%5D-09-troubleshooting.md) — Diagnosing and resolving IAM issues
- [IAM-10: Templated Policy Assignments](IAM%20-%20IAM%20administration/markdown/-%5BIAM%5D-10-templated-policy-assignments.md) — Policy templates and bulk assignments
- [IAM-11: [WORKSHOP] Policy Persona Simulation](IAM%20-%20IAM%20administration/markdown/-%5BIAM%5D-11-%5BWORKSHOP%5D-policy-persona.md) — Interactive workshop: simulate policy behavior as different personas
- [IAM-12: API Provisioning & Validation](IAM%20-%20IAM%20administration/markdown/-%5BIAM%5D-12-api-provisioning-validation.md) — Scripts and DQL for provisioning via Account Management API

### [K8S - Kubernetes monitoring](K8S%20-%20Kubernetes%20monitoring/README.md)
Best practices for monitoring Kubernetes with Dynatrace.
- [K8S-01: Fundamentals](K8S%20-%20Kubernetes%20monitoring/markdown/-%5BK8S%5D-01-fundamentals.md) — Core Kubernetes monitoring concepts
- [K8S-02: Dynakube Deployment](K8S%20-%20Kubernetes%20monitoring/markdown/-%5BK8S%5D-02-dynakube-deployment.md) — Deploying Dynatrace Operator and Dynakube
- [K8S-03: GitOps Dynakube](K8S%20-%20Kubernetes%20monitoring/markdown/-%5BK8S%5D-03-gitops-dynakube.md) — Managing Dynakube with GitOps
- [K8S-04: Cluster Monitoring](K8S%20-%20Kubernetes%20monitoring/markdown/-%5BK8S%5D-04-cluster-monitoring.md) — Monitoring Kubernetes clusters
- [K8S-05: Workload Monitoring](K8S%20-%20Kubernetes%20monitoring/markdown/-%5BK8S%5D-05-workload-monitoring.md) — Monitoring workloads and applications
- [K8S-06: Namespace Organization](K8S%20-%20Kubernetes%20monitoring/markdown/-%5BK8S%5D-06-namespace-organization.md) — Organizing and filtering by namespace
- [K8S-07: Events and Logs](K8S%20-%20Kubernetes%20monitoring/markdown/-%5BK8S%5D-07-events-and-logs.md) — Kubernetes events and log monitoring
- [K8S-08: DQL for K8s](K8S%20-%20Kubernetes%20monitoring/markdown/-%5BK8S%5D-08-dql-for-k8s.md) — Querying Kubernetes data with DQL
- [K8S-09: Troubleshooting](K8S%20-%20Kubernetes%20monitoring/markdown/-%5BK8S%5D-09-troubleshooting.md) — Diagnosing Kubernetes monitoring issues
- [K8S-10: Metadata Telemetry Enrichment](K8S%20-%20Kubernetes%20monitoring/markdown/-%5BK8S%5D-10-metadata-telemetry-enrichment.md) — Enriching telemetry with Kubernetes metadata
- [K8S-11: Multi-Tool Coexistence](K8S%20-%20Kubernetes%20monitoring/markdown/-%5BK8S%5D-11-multi-tool-coexistence.md) — Running Dynatrace alongside other monitoring tools
- [K8S-12: Specialized Monitoring](K8S%20-%20Kubernetes%20monitoring/markdown/-%5BK8S%5D-12-specialized-monitoring.md) — NGINX Ingress, CSI Driver, and resource tuning
- [K8S-13: Kafka Monitoring with Kpow](K8S%20-%20Kubernetes%20monitoring/markdown/-%5BK8S%5D-13-kafka-monitoring-with-kpow.md) — Kafka observability using Kpow Prometheus metrics

### [M2S - Managed to SaaS migration](M2S%20-%20Managed%20to%20SaaS%20migration/README.md)
Guide for migrating from Dynatrace Managed to Dynatrace SaaS.
- [M2S-01: What's Different](M2S%20-%20Managed%20to%20SaaS%20migration/markdown/-%5BM2S%5D-01-whats-different.md) — Key differences between Managed and SaaS
- [M2S-02: Migration Framework](M2S%20-%20Managed%20to%20SaaS%20migration/markdown/-%5BM2S%5D-02-migration-framework.md) — Overall migration approach and framework
- [M2S-03: Planning & Assessment](M2S%20-%20Managed%20to%20SaaS%20migration/markdown/-%5BM2S%5D-03-planning-assessment.md) — Assessing readiness and planning migration
- [M2S-04: Architecture Design](M2S%20-%20Managed%20to%20SaaS%20migration/markdown/-%5BM2S%5D-04-architecture-design.md) — Designing your SaaS architecture
- [M2S-05: Configuration Migration](M2S%20-%20Managed%20to%20SaaS%20migration/markdown/-%5BM2S%5D-05-configuration-migration.md) — Migrating configurations and settings
- [M2S-06: OneAgent & ActiveGate](M2S%20-%20Managed%20to%20SaaS%20migration/markdown/-%5BM2S%5D-06-oneagent-activegate.md) — Migrating agents and gateways
- [M2S-07: Security & Privacy](M2S%20-%20Managed%20to%20SaaS%20migration/markdown/-%5BM2S%5D-07-security-privacy.md) — Security considerations for migration
- [M2S-08: Validation & Optimization](M2S%20-%20Managed%20to%20SaaS%20migration/markdown/-%5BM2S%5D-08-validation-optimization.md) — Validating and optimizing post-migration

### [MOBL - Mobile monitoring](MOBL%20-%20Mobile%20monitoring/README.md)
Dynatrace mobile Real User Monitoring (RUM) for iOS, Android, and cross-platform frameworks.
- [MOBL-01: Mobile Monitoring Fundamentals](MOBL%20-%20Mobile%20monitoring/markdown/-%5BMOBL%5D-01-fundamentals.md) — Mobile RUM architecture, supported platforms, and beacon data flow
- [MOBL-02: iOS SDK Setup](MOBL%20-%20Mobile%20monitoring/markdown/-%5BMOBL%5D-02-sdk-setup-ios.md) — Installing and configuring the Dynatrace SDK for Swift and SwiftUI
- [MOBL-03: Android SDK Setup](MOBL%20-%20Mobile%20monitoring/markdown/-%5BMOBL%5D-03-sdk-setup-android.md) — Setting up Dynatrace RUM for Android with Gradle
- [MOBL-04: Cross-Platform Frameworks](MOBL%20-%20Mobile%20monitoring/markdown/-%5BMOBL%5D-04-cross-platform-frameworks.md) — Instrumenting Flutter, React Native, Cordova, Xamarin, and .NET MAUI
- [MOBL-05: User Action Tracking](MOBL%20-%20Mobile%20monitoring/markdown/-%5BMOBL%5D-05-user-action-tracking.md) — Capturing auto-detected and custom user interactions
- [MOBL-06: Crash Reporting & ANR Detection](MOBL%20-%20Mobile%20monitoring/markdown/-%5BMOBL%5D-06-crash-reporting.md) — Automatic crash capture, symbolication, and grouping
- [MOBL-07: Network Request Monitoring](MOBL%20-%20Mobile%20monitoring/markdown/-%5BMOBL%5D-07-network-request-monitoring.md) — HTTP(S) request visibility and timing breakdown
- [MOBL-08: Session Replay for Mobile](MOBL%20-%20Mobile%20monitoring/markdown/-%5BMOBL%5D-08-session-replay.md) — Visual session recording with privacy masking
- [MOBL-09: Session Properties & Data Privacy](MOBL%20-%20Mobile%20monitoring/markdown/-%5BMOBL%5D-09-session-properties-and-privacy.md) — Custom session enrichment and GDPR/CCPA compliance
- [MOBL-10: DQL for Mobile Analytics](MOBL%20-%20Mobile%20monitoring/markdown/-%5BMOBL%5D-10-dql-for-mobile.md) — Query reference for mobile entities, crashes, and performance
- [MOBL-11: Dashboards & Alerting](MOBL%20-%20Mobile%20monitoring/markdown/-%5BMOBL%5D-11-dashboards-and-alerting.md) — KPI dashboards with anomaly detection and metric event alerts
- [MOBL-12: Advanced Instrumentation](MOBL%20-%20Mobile%20monitoring/markdown/-%5BMOBL%5D-12-advanced-instrumentation.md) — Custom events, nested actions, A/B testing, and multi-app strategies

### [MZ2POL - Management Zone to Policy migration](MZ2POL%20-%20Management%20Zone%20to%20Policy%20migration/README.md)
Tools and guidance for migrating from Management Zones to Policy-based access control.
- [MZ2POL-00: SDK MZ Analysis Tool](MZ2POL%20-%20Management%20Zone%20to%20Policy%20migration/markdown/-%5BMZ2POL%5D-00-sdk-mz-analysis-tool.md) — Analyze existing management zones
- [MZ2POL-01: Introduction: Why Migrate](MZ2POL%20-%20Management%20Zone%20to%20Policy%20migration/markdown/-%5BMZ2POL%5D-01-introduction-why-migrate.md) — Benefits and overview
- [MZ2POL-02: Access Control Model](MZ2POL%20-%20Management%20Zone%20to%20Policy%20migration/markdown/-%5BMZ2POL%5D-02-access-control-model.md) — Policies and access control concepts
- [MZ2POL-03: Assessment & Planning](MZ2POL%20-%20Management%20Zone%20to%20Policy%20migration/markdown/-%5BMZ2POL%5D-03-assessment-planning.md) — Migration assessment and planning
- [MZ2POL-04: Policies and Boundaries](MZ2POL%20-%20Management%20Zone%20to%20Policy%20migration/markdown/-%5BMZ2POL%5D-04-policies-and-boundaries.md) — Defining policies and boundaries
- [MZ2POL-05: Segments Implementation](MZ2POL%20-%20Management%20Zone%20to%20Policy%20migration/markdown/-%5BMZ2POL%5D-05-segments-implementation.md) — Implementing segments
- [MZ2POL-06: Migration Execution](MZ2POL%20-%20Management%20Zone%20to%20Policy%20migration/markdown/-%5BMZ2POL%5D-06-migration-execution.md) — Executing the migration
- [MZ2POL-07: Validation & Troubleshooting](MZ2POL%20-%20Management%20Zone%20to%20Policy%20migration/markdown/-%5BMZ2POL%5D-07-validation-troubleshooting.md) — Validating and resolving issues
- [MZ2POL-08: Templated Policies Migration](MZ2POL%20-%20Management%20Zone%20to%20Policy%20migration/markdown/-%5BMZ2POL%5D-08-templated-policies-migration.md) — Policy templates for bulk MZ migration

### [ONBRD - Dynatrace onboarding](ONBRD%20-%20Dynatrace%20onboarding/README.md)
Step-by-step onboarding series for new Dynatrace users.
- [ONBRD-01: First Steps](ONBRD%20-%20Dynatrace%20onboarding/markdown/-%5BONBRD%5D-01-first-steps.md) — Getting started with Dynatrace
- [ONBRD-02: IAM and Authentication](ONBRD%20-%20Dynatrace%20onboarding/markdown/-%5BONBRD%5D-02-iam-and-authentication.md) — Identity and access management
- [ONBRD-03: Deploying ActiveGate](ONBRD%20-%20Dynatrace%20onboarding/markdown/-%5BONBRD%5D-03-deploying-activegate.md) — Installing and configuring ActiveGate
- [ONBRD-04: Cloud/SaaS Integrations](ONBRD%20-%20Dynatrace%20onboarding/markdown/-%5BONBRD%5D-04-cloud-saas-integrations.md) — Connecting cloud platforms
- [ONBRD-05: Deploying OneAgent](ONBRD%20-%20Dynatrace%20onboarding/markdown/-%5BONBRD%5D-05-deploying-oneagent.md) — Deployment strategies
- [ONBRD-06: Organizing Your Environment](ONBRD%20-%20Dynatrace%20onboarding/markdown/-%5BONBRD%5D-06-organizing-your-environment.md) — Structuring environments and entities
- [ONBRD-07: Understanding Your Data](ONBRD%20-%20Dynatrace%20onboarding/markdown/-%5BONBRD%5D-07-understanding-your-data.md) — Data model and exploration
- [ONBRD-08: Your First Queries](ONBRD%20-%20Dynatrace%20onboarding/markdown/-%5BONBRD%5D-08-your-first-queries.md) — Introduction to DQL
- [ONBRD-09: Setting Up Alerts](ONBRD%20-%20Dynatrace%20onboarding/markdown/-%5BONBRD%5D-09-setting-up-alerts.md) — Alerting and notifications
- [ONBRD-10: Building Dashboards](ONBRD%20-%20Dynatrace%20onboarding/markdown/-%5BONBRD%5D-10-building-dashboards.md) — Creating stakeholder dashboards

### [OPLOGS - OpenPipeline logs](OPLOGS%20-%20OpenPipeline%20logs/README.md)
End-to-end guidance for managing logs with Dynatrace OpenPipeline.
- [OPLOGS-01: Fundamentals](OPLOGS%20-%20OpenPipeline%20logs/markdown/-%5BOPLOGS%5D-01-fundamentals.md) — Core OpenPipeline concepts
- [OPLOGS-02: Migration](OPLOGS%20-%20OpenPipeline%20logs/markdown/-%5BOPLOGS%5D-02-migration.md) — Moving from other logging platforms
- [OPLOGS-03: Pipeline Processing](OPLOGS%20-%20OpenPipeline%20logs/markdown/-%5BOPLOGS%5D-03-pipeline-processing.md) — Transformations and processing stages
- [OPLOGS-04: Buckets & Governance](OPLOGS%20-%20OpenPipeline%20logs/markdown/-%5BOPLOGS%5D-04-buckets-governance.md) — Storage management and governance
- [OPLOGS-05: Querying & Parsing](OPLOGS%20-%20OpenPipeline%20logs/markdown/-%5BOPLOGS%5D-05-querying-parsing.md) — Query techniques and parsing rules
- [OPLOGS-06: Topology](OPLOGS%20-%20OpenPipeline%20logs/markdown/-%5BOPLOGS%5D-06-topology.md) — Log topology and relationships
- [OPLOGS-07: Analytics](OPLOGS%20-%20OpenPipeline%20logs/markdown/-%5BOPLOGS%5D-07-analytics.md) — Advanced log analytics
- [OPLOGS-08: Security](OPLOGS%20-%20OpenPipeline%20logs/markdown/-%5BOPLOGS%5D-08-security.md) — Security, masking, and compliance

### [OPMIG - OpenPipeline migration](OPMIG%20-%20OpenPipeline%20migration/README.md)
Structured migration path to Dynatrace OpenPipeline.
- [OPMIG-01: Introduction: Why Migrate](OPMIG%20-%20OpenPipeline%20migration/markdown/-%5BOPMIG%5D-01-introduction-why-migrate.md) — Benefits and goals
- [OPMIG-02: Architecture & Key Concepts](OPMIG%20-%20OpenPipeline%20migration/markdown/-%5BOPMIG%5D-02-architecture-key-concepts.md) — OpenPipeline architecture overview
- [OPMIG-03: Migration Assessment & Planning](OPMIG%20-%20OpenPipeline%20migration/markdown/-%5BOPMIG%5D-03-migration-assessment-planning.md) — Migration plan and readiness
- [OPMIG-04: Pipeline Configuration](OPMIG%20-%20OpenPipeline%20migration/markdown/-%5BOPMIG%5D-04-pipeline-configuration.md) — Configuring OpenPipeline
- [OPMIG-05: Routing & Buckets](OPMIG%20-%20OpenPipeline%20migration/markdown/-%5BOPMIG%5D-05-routing-buckets.md) — Routing rules and storage strategy
- [OPMIG-06: Processing & Parsing](OPMIG%20-%20OpenPipeline%20migration/markdown/-%5BOPMIG%5D-06-processing-parsing.md) — Transformation and parsing rules
- [OPMIG-07: Metric & Event Extraction](OPMIG%20-%20OpenPipeline%20migration/markdown/-%5BOPMIG%5D-07-metric-event-extraction.md) — Deriving metrics/events from logs
- [OPMIG-08: Security & Masking](OPMIG%20-%20OpenPipeline%20migration/markdown/-%5BOPMIG%5D-08-security-masking.md) — Security controls and data masking
- [OPMIG-09: Troubleshooting & Validation](OPMIG%20-%20OpenPipeline%20migration/markdown/-%5BOPMIG%5D-09-troubleshooting-validation.md) — Validation and issue resolution

### [ORGNZ - Organize data: buckets, segments, security](ORGNZ%20-%20Organize%20data%3A%20buckets%2C%20segments%2C%20security/README.md)
Organizing data in Dynatrace Grail using buckets, segments, and security context.
- [ORGNZ-01: Introduction to Organizing Data](ORGNZ%20-%20Organize%20data%3A%20buckets%2C%20segments%2C%20security/markdown/-%5BORGNZ%5D-01-introduction-to-organizing-data.md) — Why data organization matters in Grail
  - [ORGNZ-01: [LAB] Introduction to Organizing Data](ORGNZ%20-%20Organize%20data%3A%20buckets%2C%20segments%2C%20security/markdown/-%5BORGNZ%5D-01-%5BLAB%5D-introduction-to-organizing-data.md) — Hands-on exercises
- [ORGNZ-02: Understanding Grail Buckets](ORGNZ%20-%20Organize%20data%3A%20buckets%2C%20segments%2C%20security/markdown/-%5BORGNZ%5D-02-understanding-grail-buckets.md) — Bucket fundamentals and data types
  - [ORGNZ-02: [LAB] Understanding Grail Buckets](ORGNZ%20-%20Organize%20data%3A%20buckets%2C%20segments%2C%20security/markdown/-%5BORGNZ%5D-02-%5BLAB%5D-understanding-grail-buckets.md) — Hands-on exercises
- [ORGNZ-03: Bucket Strategy and Design](ORGNZ%20-%20Organize%20data%3A%20buckets%2C%20segments%2C%20security/markdown/-%5BORGNZ%5D-03-bucket-strategy-and-design.md) — Naming conventions and retention planning
  - [ORGNZ-03: [LAB] Bucket Strategy and Design](ORGNZ%20-%20Organize%20data%3A%20buckets%2C%20segments%2C%20security/markdown/-%5BORGNZ%5D-03-%5BLAB%5D-bucket-strategy-and-design.md) — Hands-on exercises
- [ORGNZ-04: Permissions in Grail](ORGNZ%20-%20Organize%20data%3A%20buckets%2C%20segments%2C%20security/markdown/-%5BORGNZ%5D-04-permissions-in-grail.md) — Overview of Grail permission levels
- [ORGNZ-05: Bucket-Level Access Control](ORGNZ%20-%20Organize%20data%3A%20buckets%2C%20segments%2C%20security/markdown/-%5BORGNZ%5D-05-bucket-level-access-control.md) — IAM policies for bucket access
  - [ORGNZ-05: [LAB] Bucket-Level Access Control](ORGNZ%20-%20Organize%20data%3A%20buckets%2C%20segments%2C%20security/markdown/-%5BORGNZ%5D-05-%5BLAB%5D-bucket-level-access-control.md) — Hands-on exercises
- [ORGNZ-06: Security Context](ORGNZ%20-%20Organize%20data%3A%20buckets%2C%20segments%2C%20security/markdown/-%5BORGNZ%5D-06-security-context.md) — Fine-grained access with dt.security_context
  - [ORGNZ-06: [LAB] Security Context](ORGNZ%20-%20Organize%20data%3A%20buckets%2C%20segments%2C%20security/markdown/-%5BORGNZ%5D-06-%5BLAB%5D-security-context.md) — Hands-on exercises
- [ORGNZ-07: Advanced Permission Patterns](ORGNZ%20-%20Organize%20data%3A%20buckets%2C%20segments%2C%20security/markdown/-%5BORGNZ%5D-07-advanced-permission-patterns.md) — Record and field-level permissions
  - [ORGNZ-07: [LAB] Advanced Permission Patterns](ORGNZ%20-%20Organize%20data%3A%20buckets%2C%20segments%2C%20security/markdown/-%5BORGNZ%5D-07-%5BLAB%5D-advanced-permission-patterns.md) — Hands-on exercises
- [ORGNZ-08: Grail Segments](ORGNZ%20-%20Organize%20data%3A%20buckets%2C%20segments%2C%20security/markdown/-%5BORGNZ%5D-08-grail-segments.md) — Logical data filtering with segments
  - [ORGNZ-08: [LAB] Grail Segments](ORGNZ%20-%20Organize%20data%3A%20buckets%2C%20segments%2C%20security/markdown/-%5BORGNZ%5D-08-%5BLAB%5D-grail-segments.md) — Hands-on exercises
- [ORGNZ-09: Enterprise Patterns](ORGNZ%20-%20Organize%20data%3A%20buckets%2C%20segments%2C%20security/markdown/-%5BORGNZ%5D-09-enterprise-patterns.md) — Combining mechanisms for enterprise governance
  - [ORGNZ-09: [LAB] Enterprise Patterns](ORGNZ%20-%20Organize%20data%3A%20buckets%2C%20segments%2C%20security/markdown/-%5BORGNZ%5D-09-%5BLAB%5D-enterprise-patterns.md) — Hands-on exercises
- [ORGNZ-10: Advanced Segment Definitions](ORGNZ%20-%20Organize%20data%3A%20buckets%2C%20segments%2C%20security/markdown/-%5BORGNZ%5D-10-advanced-segment-definitions.md) — Complex segment filters with multi-include patterns
  - [ORGNZ-10: [LAB] Advanced Segment Definitions](ORGNZ%20-%20Organize%20data%3A%20buckets%2C%20segments%2C%20security/markdown/-%5BORGNZ%5D-10-%5BLAB%5D-advanced-segment-definitions.md) — Hands-on exercises

### [OTEL - OpenTelemetry integration](OTEL%20-%20OpenTelemetry%20integration/README.md)
Integrating OpenTelemetry with Dynatrace for observability.
- [OTEL-01: Fundamentals](OTEL%20-%20OpenTelemetry%20integration/markdown/-%5BOTEL%5D-01-fundamentals.md) — Core OpenTelemetry concepts
- [OTEL-02: Collector Architecture](OTEL%20-%20OpenTelemetry%20integration/markdown/-%5BOTEL%5D-02-collector-architecture.md) — Understanding the OTel Collector
- [OTEL-03: Collector Deployment](OTEL%20-%20OpenTelemetry%20integration/markdown/-%5BOTEL%5D-03-collector-deployment.md) — Deploying and configuring collectors
- [OTEL-04: Trace Instrumentation](OTEL%20-%20OpenTelemetry%20integration/markdown/-%5BOTEL%5D-04-trace-instrumentation.md) — Instrumenting applications for traces
- [OTEL-05: Metrics Instrumentation](OTEL%20-%20OpenTelemetry%20integration/markdown/-%5BOTEL%5D-05-metrics-instrumentation.md) — Instrumenting applications for metrics
- [OTEL-06: Logs Instrumentation](OTEL%20-%20OpenTelemetry%20integration/markdown/-%5BOTEL%5D-06-logs-instrumentation.md) — Instrumenting applications for logs
- [OTEL-07: Dynatrace Integration](OTEL%20-%20OpenTelemetry%20integration/markdown/-%5BOTEL%5D-07-dynatrace-integration.md) — Sending OTel data to Dynatrace
- [OTEL-08: Troubleshooting](OTEL%20-%20OpenTelemetry%20integration/markdown/-%5BOTEL%5D-08-troubleshooting.md) — Diagnosing OpenTelemetry issues

### [SPANS - Distributed tracing and spans](SPANS%20-%20Distributed%20tracing%20and%20spans/README.md)
Guidance for working with distributed traces and spans in Dynatrace.
- [SPANS-01: Fundamentals](SPANS%20-%20Distributed%20tracing%20and%20spans/markdown/-%5BSPANS%5D-01-fundamentals.md) — Core tracing concepts
- [SPANS-02: Querying](SPANS%20-%20Distributed%20tracing%20and%20spans/markdown/-%5BSPANS%5D-02-querying.md) — Querying spans and traces with DQL
- [SPANS-03: Troubleshooting](SPANS%20-%20Distributed%20tracing%20and%20spans/markdown/-%5BSPANS%5D-03-troubleshooting.md) — Using traces for issue investigation
- [SPANS-04: Topology](SPANS%20-%20Distributed%20tracing%20and%20spans/markdown/-%5BSPANS%5D-04-topology.md) — Service topology from traces
- [SPANS-05: Analytics](SPANS%20-%20Distributed%20tracing%20and%20spans/markdown/-%5BSPANS%5D-05-analytics.md) — Advanced trace analytics
- [SPANS-06: Security](SPANS%20-%20Distributed%20tracing%20and%20spans/markdown/-%5BSPANS%5D-06-security.md) — Span security and data protection
- [SPANS-07: Buckets & Pipeline](SPANS%20-%20Distributed%20tracing%20and%20spans/markdown/-%5BSPANS%5D-07-buckets-pipeline.md) — Storage and processing
- [SPANS-08: Cost Optimization](SPANS%20-%20Distributed%20tracing%20and%20spans/markdown/-%5BSPANS%5D-08-cost-optimization.md) — Optimizing trace ingestion costs

### [S2D - Splunk to Dynatrace migration](S2D%20-%20Splunk%20to%20Dynatrace%20migration/README.md)
Guide for migrating monitoring capabilities from Splunk to Dynatrace.
- [S2D-01: Getting Started](S2D%20-%20Splunk%20to%20Dynatrace%20migration/markdown/-%5BS2D%5D-01-getting-started.md) — Migration overview and roadmap
- [S2D-02: Locating Logs](S2D%20-%20Splunk%20to%20Dynatrace%20migration/markdown/-%5BS2D%5D-02-locating-logs.md) — Finding and verifying log data in Dynatrace
- [S2D-03: SPL to DQL Translation](S2D%20-%20Splunk%20to%20Dynatrace%20migration/markdown/-%5BS2D%5D-03-spl-to-dql.md) — Converting Splunk queries to DQL
- [S2D-04: Davis Anomaly Detectors](S2D%20-%20Splunk%20to%20Dynatrace%20migration/markdown/-%5BS2D%5D-04-davis-anomaly-detectors.md) — Migrating alerts to continuous monitoring
- [S2D-05: Workflow-Based Alerts](S2D%20-%20Splunk%20to%20Dynatrace%20migration/markdown/-%5BS2D%5D-05-workflow-alerts.md) — Migrating alerts to scheduled workflows
- [S2D-06: ArrayMovingSum](S2D%20-%20Splunk%20to%20Dynatrace%20migration/markdown/-%5BS2D%5D-06-arraymovingsum.md) — Rolling sums for extended timeframes
- [S2D-07: Metric Creation](S2D%20-%20Splunk%20to%20Dynatrace%20migration/markdown/-%5BS2D%5D-07-metric-creation.md) — Extracting metrics from logs via OpenPipeline
- [S2D-08: Dashboard Migration](S2D%20-%20Splunk%20to%20Dynatrace%20migration/markdown/-%5BS2D%5D-08-dashboard-migration.md) — Converting Splunk dashboards to Dynatrace
- [S2D-09: Naming Standards](S2D%20-%20Splunk%20to%20Dynatrace%20migration/markdown/-%5BS2D%5D-09-naming-standards.md) — Asset naming conventions and organization

### [SYNTH - Synthetic monitoring](SYNTH%20-%20Synthetic%20monitoring/README.md)
Series covering Dynatrace Synthetic Monitoring.
- [SYNTH-01: Fundamentals](SYNTH%20-%20Synthetic%20monitoring/markdown/-%5BSYNTH%5D-01-fundamentals.md) — Core synthetic monitoring concepts
- [SYNTH-02: Browser Monitors](SYNTH%20-%20Synthetic%20monitoring/markdown/-%5BSYNTH%5D-02-browser-monitors.md) — Creating and managing browser monitors
- [SYNTH-03: HTTP Monitors](SYNTH%20-%20Synthetic%20monitoring/markdown/-%5BSYNTH%5D-03-http-monitors.md) — HTTP/API monitors
- [SYNTH-04: Private Locations](SYNTH%20-%20Synthetic%20monitoring/markdown/-%5BSYNTH%5D-04-private-locations.md) — Deploying private synthetic locations
- [SYNTH-05: Network Monitoring](SYNTH%20-%20Synthetic%20monitoring/markdown/-%5BSYNTH%5D-05-network-monitoring.md) — Network performance monitoring
- [SYNTH-06: Analytics](SYNTH%20-%20Synthetic%20monitoring/markdown/-%5BSYNTH%5D-06-analytics.md) — Analyzing synthetic monitoring data

### [WFLOW - Workflows and alert notifications](WFLOW%20-%20Workflows%20and%20alert%20notifications/README.md)
Automating workflows and configuring alert notifications in Dynatrace.
- [WFLOW-01: Fundamentals](WFLOW%20-%20Workflows%20and%20alert%20notifications/markdown/-%5BWFLOW%5D-01-fundamentals.md) — Core workflow concepts
- [WFLOW-02: Triggers](WFLOW%20-%20Workflows%20and%20alert%20notifications/markdown/-%5BWFLOW%5D-02-triggers.md) — Workflow trigger configuration
- [WFLOW-03: Notification Basics](WFLOW%20-%20Workflows%20and%20alert%20notifications/markdown/-%5BWFLOW%5D-03-notification-basics.md) — Setting up basic notifications
- [WFLOW-04: Notification Routing](WFLOW%20-%20Workflows%20and%20alert%20notifications/markdown/-%5BWFLOW%5D-04-notification-routing.md) — Advanced notification routing
- [WFLOW-05: Incident Management](WFLOW%20-%20Workflows%20and%20alert%20notifications/markdown/-%5BWFLOW%5D-05-incident-management.md) — Integrating with incident management
- [WFLOW-06: Custom Templates](WFLOW%20-%20Workflows%20and%20alert%20notifications/markdown/-%5BWFLOW%5D-06-custom-templates.md) — Creating custom notification templates
- [WFLOW-07: Remediation](WFLOW%20-%20Workflows%20and%20alert%20notifications/markdown/-%5BWFLOW%5D-07-remediation.md) — Automated remediation workflows
- [WFLOW-08: JavaScript & HTTP](WFLOW%20-%20Workflows%20and%20alert%20notifications/markdown/-%5BWFLOW%5D-08-javascript-http.md) — JavaScript actions and HTTP requests
- [WFLOW-09: Governance](WFLOW%20-%20Workflows%20and%20alert%20notifications/markdown/-%5BWFLOW%5D-09-governance.md) — Workflow governance and best practices

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

## How to Use

1. Open a topic README for context and prerequisites.
2. Choose your format:
	- Import JSON from NOTEBOOKS/ into Dynatrace Notebooks for interactive use.
	- Read PDFs/ for printable versions.
	- Use markdown/ for lightweight viewing or diffs.
3. Follow the numbered sequence to progress through each series.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
