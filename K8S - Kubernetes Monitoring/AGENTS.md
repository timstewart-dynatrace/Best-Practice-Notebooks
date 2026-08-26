# AGENTS.md — K8S: Kubernetes Monitoring

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

15 notebooks on monitoring Kubernetes with Dynatrace: deploying and configuring
the Operator/DynaKube, GitOps, cluster and workload monitoring, namespace
organization, event/log ingestion, metadata enrichment, multi-tool coexistence,
specialized scenarios, troubleshooting, and a DQL cookbook.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| Architecture, entity model, deployment modes, how data flows | `-[K8S]-01-fundamentals.md` |
| Installing the Operator, tokens/secrets, DynaKube CR configuration; Helm mechanics — the documented OCI chart reference vs the still-served classic repo, what the install creates, pre-upgrade checks, the upgrade process | `-[K8S]-02-dynakube-deployment.md` |
| Managing DynaKube via ArgoCD/Flux, app-of-apps, repo layout | `-[K8S]-03-gitops-dynakube.md` |
| Node/cluster health, capacity planning, cluster alerting, and querying Kubernetes events — they land in `events`, not `logs`, with discriminator `event.provider == "KUBERNETES_EVENT"` and reason field `dt.kubernetes.event.reason`; plus which event reasons to watch | `-[K8S]-04-cluster-monitoring.md` |
| Deployment/rollout health, pod & container metrics | `-[K8S]-05-workload-monitoring.md` |
| Namespace strategy, selectors, per-namespace service detection | `-[K8S]-06-namespace-organization.md` |
| Ingesting K8s events and logs, ActiveGate config, filtering (the ingest side — for *querying* events see `-[K8S]-04-cluster-monitoring.md`) | `-[K8S]-07-events-and-logs.md` |
| DQL query cookbook: entities, metrics, logs, traces, alerting | `-[K8S]-08-dql-for-k8s.md` |
| Broken monitoring: injection skips, CSI crash-loops, operator errors, support archive, symptom→fix index | `-[K8S]-09-troubleshooting.md` |
| Enriching telemetry with K8s metadata, oneagentctl, Grail fields, and the Operator 1.10.2 tagging-rule precedence change (first matching rule now wins, previously last) with its knock-on effect on `dt.security_context`, cost attribution, and primary tags | `-[K8S]-10-metadata-telemetry-enrichment.md` |
| Coexisting with Datadog/Prometheus/etc., infra-only, opt-in mode | `-[K8S]-11-multi-tool-coexistence.md` |
| NGINX Ingress, CSI driver internals, resource tuning, StatsD; choosing how code modules reach pods — CSI driver vs ephemeral volumes, a deliberate decision only from Operator 1.10.0+ | `-[K8S]-12-specialized-monitoring.md` |
| Kafka observability via Kpow Prometheus metrics | `-[K8S]-13-kafka-monitoring-with-kpow.md` |
| Hands-on end-to-end deployment walkthrough (lab format) | `-[K8S]-14-[LAB]-deployment-guide.md` |
| Consolidated checklist: mode selection, token scopes, minimal prod DynaKube, ActiveGate sizing | `-[K8S]-99-best-practice-summary.md` |

If more than three rows match, start with `-[K8S]-99-best-practice-summary.md`
and follow its pointers.

## Related series

- Cloud-provider-managed clusters (EKS/AKS/GKE integrations): `../CLOUD - Cloud Provider Integrations/`
- OpenTelemetry alongside OneAgent in K8s: `../OTEL - OpenTelemetry Integration/`
- Log parsing/routing after ingestion: `../OPLOGS - OpenPipeline Logs/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "K8S-02") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
