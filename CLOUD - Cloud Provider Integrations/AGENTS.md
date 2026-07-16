# AGENTS.md — CLOUD: Cloud Provider Integrations

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

9 notebooks on integrating Dynatrace with AWS, Azure, and GCP: the Clouds app
vs classic ActiveGate polling, per-provider connection and authentication
setup, EKS and serverless deep dives, CloudWatch log ingestion, and
multi-cloud observability patterns.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| Clouds app vs classic ActiveGate polling, cloud entity model, cloud metrics vs host metrics, multi-cloud strategy planning | `-[CLOUD]-01-cloud-integration-fundamentals.md` |
| Connecting AWS: IAM role via CloudFormation, key-based auth removal (1.267), enabling services, CloudWatch metric delays, Metric Streams via Firehose, API cost control | `-[CLOUD]-02-aws-integration.md` |
| EKS clusters: DynaKube on EKS, node groups vs Fargate, IRSA, Container Insights comparison, namespace cost allocation | `-[CLOUD]-03-aws-eks-monitoring.md` |
| Lambda and serverless: cold starts, throttles, `dt.faas.*` metrics, Service Detection v2, API Gateway, Step Functions, DynamoDB, end-to-end serverless tracing | `-[CLOUD]-04-aws-lambda-serverless.md` |
| Connecting Azure: Azure Native Dynatrace Service, Entra ID app registration, supported services, AKS, resource group mapping | `-[CLOUD]-05-azure-integration.md` |
| Connecting GCP: service account auth, push-based Helm/Pub/Sub integration on GKE, supported services, GKE and Cloud Run monitoring | `-[CLOUD]-06-gcp-integration.md` |
| Forwarding cloud logs: Amazon Data Firehose setup, deprecated Lambda forwarder migration, S3 ingestion, source-vs-Dynatrace filtering, OpenPipeline processing, log cost | `-[CLOUD]-07-cloudwatch-log-ingestion.md` |
| Running on more than one cloud: cross-cloud comparison queries, naming conventions, unified dashboards and alerting, cost comparison, governance | `-[CLOUD]-08-multi-cloud-patterns.md` |
| Consolidated checklist: connection/auth choices, service tiering, log ingestion defaults, alert windows for delayed cloud metrics | `-[CLOUD]-99-best-practice-summary.md` |

If more than three rows match, start with `-[CLOUD]-99-best-practice-summary.md`
and follow its pointers.

## Related series

- Operator/DynaKube depth behind the EKS/AKS/GKE notebooks: `../K8S - Kubernetes Monitoring/`
- Parsing and routing forwarded cloud logs after ingestion: `../OPLOGS - OpenPipeline Logs/`
- Collector-based ingest as an alternative cloud data path: `../OTEL - OpenTelemetry Integration/`
- Controlling cloud-integration ingest cost: `../FINOPS - Cost Management & FinOps/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "CLOUD-02") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
