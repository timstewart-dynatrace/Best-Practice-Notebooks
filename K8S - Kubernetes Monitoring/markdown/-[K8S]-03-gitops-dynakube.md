# K8S-03: GitOps for DynaKube

> **Series:** K8S — Kubernetes Monitoring | **Notebook:** 3 of 13 | **Created:** January 2026 | **Last Updated:** 08/11/2026

## Managing DynaKube with ArgoCD and Flux
GitOps enables declarative, version-controlled management of your Dynatrace monitoring configuration. This notebook covers integrating DynaKube with popular GitOps tools: ArgoCD and Flux.

---

## Table of Contents

1. [GitOps Principles for Observability](#gitops-principles-for-observability)
2. [ArgoCD Integration](#argocd-integration)
3. [Flux Integration](#flux-integration)
4. [Secret Management](#secret-management)
5. [Multi-Cluster Patterns](#multi-cluster-patterns)
6. [Progressive Rollouts](#progressive-rollouts)
7. [Troubleshooting GitOps Deployments](#troubleshooting-gitops-deployments)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **GitOps Tool** | ArgoCD v2.x or Flux v2.x |
| **Git Repository** | Write access for configuration |
| **Kubernetes** | Dynatrace Operator installed |
| **Knowledge** | Familiarity with K8S-02: DynaKube Deployment |

<a id="gitops-principles-for-observability"></a>
## 1. GitOps Principles for Observability
### Why GitOps for Monitoring?

| Benefit | Description |
|---------|-------------|
| **Version Control** | Track all config changes in Git history |
| **Audit Trail** | Who changed what, when, and why |
| **Consistency** | Same config across dev/staging/prod |
| **Self-Healing** | Automatic drift correction |
| **Rollback** | Revert by reverting Git commits |

### GitOps Workflow for DynaKube

![GitOps Workflow](images/03-gitops-workflow.png)

<!-- MARKDOWN_TABLE_ALTERNATIVE
| Step | Action | Component |
|------|--------|-----------|
| 1 | Developer commits YAML | Git Repository |
| 2 | Pull Request reviewed | Git Repository |
| 3 | Merge to main | Git Repository |
| 4 | GitOps controller detects | ArgoCD / Flux |
| 5 | Apply to cluster | Kubernetes API |
| 6 | Operator reconciles | Dynatrace Operator |
| 7 | Monitoring updated | OneAgent / ActiveGate |
For environments where SVG doesn't render
-->

### Repository Structure

```
monitoring-config/
├── base/
│   ├── namespace.yaml
│   ├── operator/
│   │   └── kustomization.yaml
│   └── dynakube/
│       ├── dynakube.yaml
│       └── kustomization.yaml
├── overlays/
│   ├── dev/
│   │   ├── kustomization.yaml
│   │   └── dynakube-patch.yaml
│   ├── staging/
│   │   └── ...
│   └── prod/
│       └── ...
└── clusters/
    ├── dev-cluster/
    ├── staging-cluster/
    └── prod-cluster/
```

<a id="argocd-integration"></a>
## 2. ArgoCD Integration
### ArgoCD Application for Operator

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: dynatrace-operator
  namespace: argocd
spec:
  project: monitoring
  source:
    repoURL: https://raw.githubusercontent.com/Dynatrace/dynatrace-operator/main/config/helm/repos/stable
    chart: dynatrace-operator
    targetRevision: 1.10.2  # Pin to a version you have validated — check https://github.com/Dynatrace/dynatrace-operator/releases for latest
    helm:
      releaseName: dynatrace-operator
      values: |
        platform: kubernetes
        csidriver:
          enabled: true
  destination:
    server: https://kubernetes.default.svc
    namespace: dynatrace
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

> **Choosing the pinned version.** A GitOps pin is copy-pasted far more often than it is revisited, so pick it deliberately:
>
> | Constraint | Why |
> |------------|-----|
> | **Do not pin below 1.4.1** | Operator 1.3.0 through 1.4.0 are inside the CSI driver liveness-probe crash-loop window; 1.4.1 adjusted the probe parameters. See K8S-09 §2. |
> | **Skip 1.10.0** | Its own release notes advise skipping it (auto-update defect), and it is now flagged `prerelease: true` on the GitHub releases page — a signal you can verify yourself before pinning. Fixed in 1.10.1. |
> | **1.10.2 is the recommended pin** | Released July 30, 2026 with a published changelog. It fixes a workload/namespace **tagging-precedence regression** (subsequent matching rules used to overwrite earlier ones; now only the *first* matching rule per key applies — a behaviour change, see K8S-10), a `dynatrace-webhook` `CrashLoopBackOff` under the gVisor runtime class, injected pods hanging on the OneAgent-binary download (timeout raised to 15 minutes), and it now logs metadata-enrichment rules it cannot apply instead of dropping them silently. |
> | **1.10.1 remains a working pin** | Estates move GitOps pins on their own cadence. Until you upgrade, 1.10.1 is still supported and still the fix for the 1.10.0 defects — subject to the OpenShift-manifest caveat in K8S-09 §2. Read the K8S-10 tagging note before bumping a cluster whose enrichment relies on last-rule-wins ordering. |
> | **Anything below 1.7.x is likely past EOL** | Standard 9-month / Enterprise 12-month support window. |
>
> Pin an exact version rather than a range for the operator itself — an operator upgrade can change chart *defaults* (see K8S-02 §8), and a range lets that happen without a Git commit to review.

### ArgoCD Application for DynaKube

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: dynakube
  namespace: argocd
spec:
  project: monitoring
  source:
    repoURL: https://github.com/your-org/monitoring-config
    targetRevision: main
    path: overlays/prod
  destination:
    server: https://kubernetes.default.svc
    namespace: dynatrace
  syncPolicy:
    automated:
      prune: false  # Don't auto-delete DynaKube
      selfHeal: true
```

### ArgoCD App of Apps Pattern

Manage multiple clusters from a single Application:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: monitoring-apps
  namespace: argocd
spec:
  project: monitoring
  source:
    repoURL: https://github.com/your-org/monitoring-config
    targetRevision: main
    path: clusters
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

<a id="flux-integration"></a>
## 3. Flux Integration
### Flux HelmRelease for Operator

```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: HelmRepository
metadata:
  name: dynatrace
  namespace: flux-system
spec:
  interval: 1h
  url: https://raw.githubusercontent.com/Dynatrace/dynatrace-operator/main/config/helm/repos/stable
---
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: dynatrace-operator
  namespace: dynatrace
spec:
  interval: 5m
  chart:
    spec:
      chart: dynatrace-operator
      version: "1.10.2"  # Pin exactly — see the version-choice table in §2 (do not pin below 1.4.1; skip 1.10.0)
      sourceRef:
        kind: HelmRepository
        name: dynatrace
        namespace: flux-system
  values:
    platform: kubernetes
    csidriver:
      enabled: true
```

### Flux Kustomization for DynaKube

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: dynakube
  namespace: flux-system
spec:
  interval: 10m
  path: ./overlays/prod
  prune: true
  sourceRef:
    kind: GitRepository
    name: monitoring-config
  healthChecks:
    - apiVersion: dynatrace.com/v1beta5
      kind: DynaKube
      name: dynakube
      namespace: dynatrace
  dependsOn:
    - name: dynatrace-operator
```

### Flux GitRepository Source

```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: monitoring-config
  namespace: flux-system
spec:
  interval: 1m
  url: https://github.com/your-org/monitoring-config
  ref:
    branch: main
  secretRef:
    name: git-credentials
```

<a id="secret-management"></a>
## 4. Secret Management
API tokens should never be stored in Git. Use these approaches:

### Option 1: External Secrets Operator

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: dynakube
  namespace: dynatrace
spec:
  refreshInterval: 1h
  secretStoreRef:
    kind: ClusterSecretStore
    name: vault  # or aws-secrets-manager, gcp-secret-manager
  target:
    name: dynakube
    creationPolicy: Owner
  data:
    - secretKey: apiToken
      remoteRef:
        key: dynatrace/operator-token
    - secretKey: dataIngestToken
      remoteRef:
        key: dynatrace/data-ingest-token
```

### Option 2: Sealed Secrets

```bash
# Create sealed secret from raw secret
kubectl create secret generic dynakube \
  --namespace dynatrace \
  --from-literal=apiToken=<your-api-token> \
  --from-literal=dataIngestToken=<your-data-ingest-token> \
  --dry-run=client -o yaml | \
  kubeseal --format yaml > dynakube-sealed.yaml
```

```yaml
# Sealed secret (safe to commit)
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  name: dynakube
  namespace: dynatrace
spec:
  encryptedData:
    apiToken: AgBy8hM...
    dataIngestToken: AgCtr9...
```

### Option 3: SOPS with Age/GPG

```yaml
# .sops.yaml in repo root
creation_rules:
  - path_regex: .*.secret.yaml$
    age: age1ql3z7hjy54pw...
```

```bash
# Encrypt secrets
sops --encrypt secrets.yaml > secrets.secret.yaml

# Flux decrypts automatically with kustomize-controller
```

<a id="multi-cluster-patterns"></a>
## 5. Multi-Cluster Patterns
### Kustomize Overlays by Environment

**Base DynaKube (`base/dynakube/dynakube.yaml`):**

```yaml
apiVersion: dynatrace.com/v1beta5
kind: DynaKube
metadata:
  name: dynakube
spec:
  apiUrl: PLACEHOLDER  # Patched per environment
  oneAgent:
    cloudNativeFullStack: {}
  activeGate:
    capabilities:
      - kubernetes-monitoring
      - routing
```

**Production Overlay (`overlays/prod/kustomization.yaml`):**

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: dynatrace
resources:
  - ../../base/dynakube
patches:
  - patch: |-
      - op: replace
        path: /spec/apiUrl
        value: https://your-tenant.live.dynatrace.com/api
    target:
      kind: DynaKube
      name: dynakube
```

### Environment-Specific Configuration

| Environment | Config Difference |
|-------------|-------------------|
| **Dev** | Lower resources, fewer replicas |
| **Staging** | Match prod config, different tenant |
| **Prod** | High availability, more resources |

**Dev Patch (`overlays/dev/dynakube-patch.yaml`):**

```yaml
apiVersion: dynatrace.com/v1beta5
kind: DynaKube
metadata:
  name: dynakube
spec:
  activeGate:
    replicas: 1
    resources:
      requests:
        cpu: 100m
        memory: 256Mi
```

<a id="progressive-rollouts"></a>
## 6. Progressive Rollouts
### ArgoCD Sync Waves

Control deployment order:

```yaml
# Wave 0: CRDs and Namespace
apiVersion: v1
kind: Namespace
metadata:
  name: dynatrace
  annotations:
    argocd.argoproj.io/sync-wave: "0"
---
# Wave 1: Operator
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: dynatrace-operator
  annotations:
    argocd.argoproj.io/sync-wave: "1"
---
# Wave 2: DynaKube
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: dynakube
  annotations:
    argocd.argoproj.io/sync-wave: "2"
```

### Flux Dependency Ordering

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: dynakube
spec:
  dependsOn:
    - name: dynatrace-operator
    - name: external-secrets  # If using ESO for tokens
```

### Canary Deployments

Roll out monitoring changes gradually:

1. Deploy to dev cluster → validate
2. Deploy to staging → run tests
3. Deploy to prod-canary (subset) → monitor
4. Deploy to prod (all clusters)

<a id="troubleshooting-gitops-deployments"></a>
## 7. Troubleshooting GitOps Deployments
### ArgoCD Troubleshooting

```bash
# Check application status
argocd app get dynakube

# View sync diff
argocd app diff dynakube

# Force sync
argocd app sync dynakube --force

# View logs
kubectl -n argocd logs -l app.kubernetes.io/name=argocd-application-controller
```

### Flux Troubleshooting

```bash
# Check kustomization status
flux get kustomizations

# Check helm releases
flux get helmreleases -n dynatrace

# Force reconciliation
flux reconcile kustomization dynakube --with-source

# View events
kubectl -n flux-system get events --sort-by='.lastTimestamp'
```

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| **Secret not found** | External secret sync delay | Check ESO status, wait for sync |
| **CRD not found** | Operator not deployed | Check operator app status |
| **Sync failed** | YAML syntax error | Validate with `kubectl --dry-run` |
| **Drift detected** | Manual changes | Revert or accept drift in Git |

```dql
// Monitor GitOps-related events in the cluster
fetch logs, from:-1h
| filter matchesPhrase(content, "argocd") or matchesPhrase(content, "flux")
| fields timestamp, content
| sort timestamp desc
| limit 30
```

```dql
// Track DynaKube configuration changes
fetch logs, from:-1h
| filter matchesPhrase(content, "dynakube") and (matchesPhrase(content, "updated") or matchesPhrase(content, "created") or matchesPhrase(content, "deleted"))
| fields timestamp, content
| sort timestamp desc
| limit 20
```

## Next Steps

With GitOps configured, proceed to:

| Next Notebook | Topic |
|---------------|-------|
| **K8S-04: Cluster Health Monitoring** | Deep-dive into cluster metrics |
| **K8S-05: Workload Monitoring** | Application-level observability |
| **K8S-06: Namespace Organization** | Boundaries and access control |

---

## Summary

In this notebook, you learned:

- GitOps principles and benefits for monitoring
- ArgoCD Application definitions for operator and DynaKube
- Flux HelmRelease and Kustomization for DynaKube
- Secret management with External Secrets, Sealed Secrets, and SOPS
- Multi-cluster patterns with Kustomize overlays
- Progressive rollout strategies
- Troubleshooting GitOps deployments

---

## References

- [Set up Dynatrace on Kubernetes (DT docs)](https://docs.dynatrace.com/docs/ingest-from/setup-on-k8s)
- [DynaKube parameters (DT docs)](https://docs.dynatrace.com/docs/ingest-from/setup-on-k8s/reference/dynakube-parameters)
- [Dynatrace Operator releases (Dynatrace GitHub)](https://github.com/Dynatrace/dynatrace-operator/releases)
- [Dynatrace Operator Helm chart (Dynatrace GitHub)](https://github.com/Dynatrace/dynatrace-operator/blob/main/config/helm/chart/default/values.yaml)
- [ArgoCD documentation (argoproj.io)](https://argo-cd.readthedocs.io/en/stable/)
- [Flux documentation (fluxcd.io)](https://fluxcd.io/flux/)
- [External Secrets Operator (external-secrets.io)](https://external-secrets.io/)
- [Kustomize (kustomize.io)](https://kustomize.io/)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
