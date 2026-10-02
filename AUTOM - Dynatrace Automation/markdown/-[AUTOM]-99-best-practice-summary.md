# AUTOM-99: Best Practice Summary

> **Series:** AUTOM — Dynatrace Automation | **Notebook:** 99 | **Created:** March 2026 | **Last Updated:** 10/02/2026

This notebook consolidates every actionable best practice from the AUTOM series (notebooks 01-09) into a single reference. Each practice is definitive: it tells you exactly what to set, not what to consider.

Use this as a checklist when designing, implementing, or auditing Dynatrace configuration automation.

---

## Table of Contents

1. [Authentication & Token Management](#authentication)
2. [Settings API](#settings-api)
3. [Monaco Configuration-as-Code](#monaco)
4. [Terraform Infrastructure-as-Code](#terraform)
5. [Dynatrace Workflows](#workflows)
6. [SDKs & Programmatic Access](#sdks)
7. [CI/CD & GitOps](#cicd-gitops)
8. [Governance Architecture](#governance)
9. [Migration Automation](#migration)
10. [Tool Selection](#tool-selection)
11. [Community Resources](#community-resources)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **AUTOM Series** | Familiarity with AUTOM-01 through AUTOM-09 |
| **Dynatrace Environment** | SaaS tenant with admin access |
| **Automation Tooling** | Monaco CLI, Terraform CLI, or both installed |

---

<a id="authentication"></a>
## 1. Authentication & Token Management

| Practice | Recommended Setting/Value | Priority |
|----------|----------------|----------|
| Use least-privilege scopes | Grant only `settings.read`, `settings.write`, `ReadConfig`, `WriteConfig` for config tools. Add `ExternalSyntheticIntegration` only when managing synthetics. | Critical |
| Configure a platform credential **and** a classic API token | A Platform Token **or** an OAuth client covers Settings 2.0 and platform resources (Workflows, Documents, Segments); with the Terraform provider set `DYNATRACE_HTTP_OAUTH_PREFERENCE=true` when using a Platform Token for those. A classic API Token (`dt0c01.*`) is still required for Synthetics and `dynatrace_slo_v2`. Account IAM resources need an OAuth client (`client_id`, `client_secret`, `account_id`). In Monaco, reference them as `auth.token` plus `auth.platformToken` or `auth.oAuth` — access and platform tokens are not interchangeable. | Critical |
| Never hardcode tokens | Store tokens in environment variables, HashiCorp Vault, AWS Secrets Manager, or equivalent secret store. | Critical |
| Separate tokens per environment | Create distinct tokens for dev, staging, and production tenants. Never share tokens across environments. | Critical |
| Rotate tokens on a schedule | Set a rotation cadence (e.g., 90 days). For short-lived tokens, retrieve from Vault at pipeline runtime. | Recommended |
| Prefer Platform Tokens over classic Access Tokens | Platform Tokens scope to the user's existing permissions and simplify token management. | Recommended |
| Authenticate pipelines as a service user | Issue a platform token for a service user (AUTOM-96), or create an OAuth client whose subject user is that service user — never a person's own credentials. Account IAM resources need an OAuth client: platform tokens can't be used for IAM (Account Management). | Recommended |

> <sub>**Sources:** [OAuth clients (DT docs)](https://docs.dynatrace.com/docs/manage/identity-access-management/access-tokens-and-oauth-clients/oauth-clients) — *"This needs to be an active user, and either a service user or any user with account-user-management permission."*; [Platform tokens (DT docs)](https://docs.dynatrace.com/docs/manage/identity-access-management/access-tokens-and-oauth-clients/platform-tokens) — *"They can be assigned to the user creating them or to a service user that the creating user has access to."*; [Provider configuration (Dynatrace GitHub)](https://github.com/dynatrace-oss/terraform-provider-dynatrace/blob/main/docs/index.md) — *"Platform tokens can't be used for IAM (Account Management) or classic resources."*</sub>

---

<a id="settings-api"></a>
## 2. Settings API

| Practice | Recommended Setting/Value | Priority |
|----------|----------------|----------|
| Use Settings 2.0 API for all new configuration | Endpoint: `/api/v2/settings/objects`. Do not use Classic Config API for new configs. | Critical |
| Discover schemas before creating objects | Call `GET /api/v2/settings/schemas` to find the correct `schemaId`, then `GET /api/v2/settings/schemas/{schemaId}` for field requirements. | Critical |
| Use bulk POST for multiple objects | Send an array of objects in a single `POST /api/v2/settings/objects` call instead of individual requests. | Recommended |
| Check before create (idempotency) | Query for existing objects by `schemaIds` and `scopes` before creating. Use `PUT` for updates (idempotent), not `POST`. | Recommended |
| Implement exponential backoff on 429 | Retry with increasing delay when rate-limited, and throttle bulk operations client-side. Dynatrace documents no fixed request rate: `429` means the environment's request thread pool and its queue are full, so the limit depends on load. | Recommended |
| Track object IDs for lifecycle management | Store returned `objectId` values for subsequent update and delete operations. | Recommended |
| Review existing objects as examples | Before writing new config, `GET /api/v2/settings/objects?schemaIds=<schema>` to see the structure of existing objects in your tenant. | Optional |

> <sub>**Sources:** [Access limit (DT docs)](https://docs.dynatrace.com/docs/dynatrace-api/basics/access-limit) — *"You reach the limit when both thread pool and its queue are full or the request times out in the queue (timeout is 10 seconds)."*</sub>

---

<a id="monaco"></a>
## 3. Monaco Configuration-as-Code

| Practice | Recommended Setting/Value | Priority |
|----------|----------------|----------|
| Always run `monaco deploy --dry-run` before deploy | Monaco has no `validate` command. `monaco deploy manifest.yaml --dry-run` parses YAML, checks template JSON and resolves references **without contacting the tenant** — it cannot catch payload errors, so a deploy can still fail with HTTP 400 after a clean dry-run. | Critical |
| Use environment variables for auth | Set `DT_TENANT_URL` and `DT_API_TOKEN` as env vars. In `manifest.yaml`, `auth.token.name` holds the **variable name**, never the token. | Critical |
| Use meaningful config IDs | IDs should describe the config purpose (e.g., `production-segment`, `web-app-problem-routing`). These IDs are how Monaco tracks objects. | Critical |
| Use `manifest.yaml` for multi-environment setup | Define all environments in `environmentGroups` with separate URL/token env vars per environment. | Recommended |
| Use environment-specific overrides | Add `environmentOverrides` (or `groupOverrides`) at the config level, sibling of `config:`, to vary values (e.g., `delay_minutes: 1` for prod, `5` for staging). | Recommended |
| Use `skip` with overrides for env-specific configs | `skip` is a boolean on `config:`; set a default and flip it per environment or environment group in `environmentOverrides` / `groupOverrides`. | Recommended |
| Use projects to group related configs | Put related configs in one project and deploy it with `--project`. `--group` selects manifest **environment groups**, not configs; there is no per-config `group:` key. | Recommended |
| Use reference parameters for ordering | Reference other configs with `["<project>", "<configType>", "<configId>", "id"]` (for settings, `configType` is the schema ID) so Monaco deploys the dependency first. | Recommended |
| Modular directory structure | Separate configs by domain: `segments/`, `workflows/`, `slo-v2/` (classic estates: `management-zones/`, `alerting-profiles/` until the upgrade). | Recommended |
| Download before creating from scratch | Run `monaco download` to get the current state, then modify the exported YAML. | Optional |

---

<a id="terraform"></a>
## 4. Terraform Infrastructure-as-Code

| Practice | Recommended Setting/Value | Priority |
|----------|----------------|----------|
| Pin the provider version | `version = "~> 1.105"` in `required_providers` block (AUTOM-04), and commit `.terraform.lock.hcl` so every run uses the same provider build (AUTOM-96). Never use unpinned versions. | Critical |
| Use remote state storage | Configure `backend "s3"` or `terraform.cloud` block. Never use local state for teams. | Critical |
| Enable state locking | Use the S3 backend with `use_lockfile = true` (Terraform 1.11+; DynamoDB-based locking is deprecated), HCP Terraform, or an equivalent locking backend to prevent concurrent applies. | Critical |
| Always run `terraform plan` before `terraform apply` | Review the plan output for every change. Automate plan-as-PR-comment in CI. | Critical |
| Use dual auth for full resource coverage | Configure both `dt_api_token` (for Synthetics and the classic `dynatrace_slo_v2`) and a platform token or OAuth client (for Gen3 resources, including `dynatrace_platform_slo`; OAuth `client_id`/`client_secret`/`account_id` for IAM). The provider routes each resource to the correct auth method. | Critical |
| Use modules for reusable patterns | Create modules for standard patterns (e.g., `modules/slo/`, `modules/problem-routing/` — see AUTOM-09 §5; classic `modules/alerting-profile/`-style modules only while the tenant is unupgraded). | Recommended |
| Use variable validation in modules | Add `validation` blocks to enforce naming, environment values, and delay constraints at module input. | Recommended |
| Use a directory per environment | `envs/dev/`, `envs/staging/`, `envs/prod/`, each with its own backend and credentials (AUTOM-09 §6). CLI workspaces are not appropriate when environments need separate credentials and access controls. | Recommended |
| Import existing resources into state | Write the HCL block first, then `terraform import <resource> "<object-id>"`. Never recreate what already exists. | Recommended |
| Use `terraform state rm` to detach without deleting | When you need to stop managing a resource without destroying it. | Recommended |
| Format code with `terraform fmt` | Run before every commit for consistent HCL formatting. | Recommended |
| Use the export utility for brownfield adoption | Run `DYNATRACE_TARGET_FOLDER=./exported ./terraform-provider-dynatrace -export -ref -id` to generate HCL from existing configs (the output folder comes from `DYNATRACE_TARGET_FOLDER`; there is no `-target-folder` flag). | Optional |

> <sub>**Sources:** [S3 backend (Terraform docs)](https://developer.hashicorp.com/terraform/language/backend/s3) — *"DynamoDB-based locking is deprecated and will be removed in a future minor version."*; [Terraform v1.11.0 release notes (HashiCorp GitHub)](https://github.com/hashicorp/terraform/releases/tag/v1.11.0) — *"S3 native state locking is now generally available."*; [Workspaces (Terraform docs)](https://developer.hashicorp.com/terraform/language/state/workspaces) — *"Workspaces are not appropriate for system decomposition or deployments requiring separate credentials and access controls."*</sub>

---

<a id="workflows"></a>
## 5. Dynatrace Workflows

| Practice | Recommended Setting/Value | Priority |
|----------|----------------|----------|
| One workflow, one purpose | Each workflow handles a single responsibility (e.g., "problem email notification" not "do everything on problem"). | Critical |
| Make tasks idempotent | Tasks must be safe to run multiple times without side effects (e.g., check if ticket exists before creating). | Critical |
| Store secrets in Credential Vault | Use Dynatrace Credential Vault for API keys, webhook URLs, and integration tokens. Never hardcode secrets in workflow YAML or JavaScript. | Critical |
| Scope triggers narrowly | Filter triggers by entity tag (ownership), problem category, severity, and a custom DQL filter — management-zone scoping is classic. Do not trigger on all problems. | Critical |
| Add retries on external calls | Set the task retry `count` (1–99) and `delay` in **seconds** (1–3600) on HTTP tasks and external integrations — e.g., count 3, delay 30. | Recommended |
| Route errors to a handler task | Add a handler task whose condition is the upstream task's state (`states: {api_call: ERROR}` / Terraform `conditions { states = { api_call = "ERROR" } }`). Workflows have no `on_error` construct. | Recommended |
| Validate entity type before remediation | Check `entity.type` in a JavaScript task before executing remediation (e.g., only restart `PROCESS_GROUP_INSTANCE`, not hosts). | Recommended |
| Debounce flapping alerts | Add delays or deduplication checks for alerts that open/close rapidly. | Recommended |
| Use manual triggers for testing | Test workflow logic with manual triggers before enabling detected-problem triggers. | Recommended |
| Log all remediation actions | Ensure every auto-remediation action writes to the Dynatrace audit trail and comments on the originating problem. | Recommended |
| Use sub-workflows for reusable logic | Extract common patterns (e.g., "create ITSM ticket") into sub-workflows callable from multiple parent workflows. | Optional |
| Use approval gates for risky remediations | Enable built-in Approval Requests for actions like scaling, restarts, or config changes. | Optional |

---

<a id="sdks"></a>
## 6. SDKs & Programmatic Access

| Practice | Recommended Setting/Value | Priority |
|----------|----------------|----------|
| Use environment variables for credentials | Set `DT_URL` and `DT_API_TOKEN` as environment variables. Never pass tokens as function arguments or config literals. | Critical |
| Handle paginated responses | Always iterate through all pages. Check for `nextPageKey` in responses and loop until exhausted. | Critical |
| Implement rate limiting | Throttle client-side with a rate-limiting decorator/wrapper (AUTOM-06) and back off exponentially on `429`. Dynatrace documents no fixed request rate — the limit depends on load (§2). | Recommended |
| Use type-safe SDK clients | Use the TypeScript `@dynatrace-sdk/*` clients (e.g., `@dynatrace-sdk/client-query`) in Dynatrace apps and functions. Dynatrace publishes no Python SDK package — Python automation calls the REST APIs directly. | Recommended |
| Wrap API calls in error handlers | Catch errors around every call; log the HTTP status and message (`requests`: `raise_for_status()`). Re-raise what you cannot handle. | Recommended |
| Add structured logging | Log request method, endpoint, status code, and duration for every API call. | Recommended |
| Use MCP Server for AI-assisted access | Connect Dynatrace MCP Server to Claude Code, GitHub Copilot, or Amazon Q for natural-language queries against your tenant. | Optional |

---

<a id="cicd-gitops"></a>
## 7. CI/CD & GitOps

| Practice | Recommended Setting/Value | Priority |
|----------|----------------|----------|
| Store all config in Git | Every Dynatrace configuration (Monaco YAML, Terraform HCL) lives in version control. No exceptions. | Critical |
| Require PR reviews for production changes | Enable branch protection on `main`. Require at least 1 approval. Use CODEOWNERS for Dynatrace config paths. | Critical |
| Mask all tokens in CI/CD | Store tokens as masked secrets in GitHub Actions, GitLab CI, or equivalent. Never echo tokens in logs. | Critical |
| Validate on every PR | Run `monaco deploy --dry-run` or `terraform validate` + `terraform plan` on every pull request. Post plan output as PR comment. | Critical |
| Deploy only from main branch | Gate `terraform apply` and `monaco deploy` to run only on push to `main` (not on PR branches). | Critical |
| Use staged rollout: dev, staging, production | Deploy to dev first, then staging, then production. Require manual approval gate before production. | Critical |
| Schedule drift detection | Run `terraform plan -detailed-exitcode` on a cron schedule (e.g., weekdays 6am UTC). Exit code `2` means drift. Auto-create GitHub Issue on drift. | Recommended |
| Use Vault for runtime credentials | Retrieve tokens at pipeline runtime from HashiCorp Vault using OIDC/JWT auth instead of static CI/CD secrets. | Recommended |
| Use reusable workflows for multi-repo orgs | Define a shared `workflow_call` workflow that all team repos invoke. Standardizes plan/apply across the organization. | Recommended |
| Send deployment events to Dynatrace | `POST /api/v2/events/ingest` with `eventType: CUSTOM_DEPLOYMENT`, commit SHA, and branch name after every deploy. | Recommended |
| Use Kustomize overlays for Operator GitOps | Base DynaKube config + per-cluster patches via `patches:` (`patchesStrategicMerge` is deprecated in Kustomize). Use `apiVersion: dynatrace.com/v1beta6`. | Recommended |
| Encrypt K8s secrets in Git | Use Sealed Secrets, SOPS, or External Secrets Operator. Never commit plaintext Dynatrace tokens to Git. | Critical |
| Use a PR template for config changes | Include sections: Description, Type of Change, Environments Affected, Validation Checklist, Rollback Plan. | Optional |

---

<a id="governance"></a>
## 8. Governance Architecture

| Practice | Recommended Setting/Value | Priority |
|----------|----------------|----------|
| Separate IAM pipeline from config pipeline | Pipeline A (central team) manages `dynatrace_iam_*` resources. Pipeline B (LOB teams) manages configs under grants from Pipeline A. | Critical |
| Single service account writer in production | Only one SA writes to prod. All humans get read-only. Teams develop in dev/test tenant with UI write access. | Critical |
| Never mix Monaco and Terraform for the same config type | Choose one tool per configuration domain. Mixing causes state conflicts and unpredictable drift. | Critical |
| Eliminate manual changes alongside automation | If a config is managed by automation, all changes go through the pipeline. No ClickOps in production. | Critical |
| Always use version control for configs | Every automation artifact (YAML, HCL, JSON) lives in Git with meaningful commit messages. | Critical |
| Use OPA/Conftest for resource type allowlists | Define a Rego policy that restricts which `dynatrace_*` resource types each team repo can create. | Recommended |
| Enforce mandatory team tagging via policy | OPA/Sentinel policy requires every resource to include team ownership metadata (e.g., an `owner` tag, naming prefix; MZ binding on classic tenants). | Recommended |
| Lock down Terraform state file access | Restrict HCP Terraform workspace permissions. State files can contain OAuth credentials and IAM binding details. | Recommended |
| Use brokered self-service for Synthetic monitors | Teams submit declarative requests; a central pipeline owns the environment-wide API token and applies on their behalf. Teams never hold direct API credentials. | Recommended |
| Use IAM policies for schema-level access | Manage `dynatrace_iam_policy` resources with `statement_query` restricting teams to specific `settings:objects:*` schemas. | Recommended |
| Document v1 API limitations for auditors | Frame Synthetic v1 API scoping gap as an accepted platform constraint with compensating controls (brokered access, MZ fencing, Sentinel). | Optional |
| Use module allowlists in Sentinel | Restrict Terraform plans to only approved modules, preventing teams from using unapproved resource patterns. | Optional |

---

<a id="migration"></a>
## 9. Migration Automation

| Practice | Recommended Setting/Value | Priority |
|----------|----------------|----------|
| Export before migrating | Run `monaco download` (with an access token **and** a platform token, or platform types are skipped with only a log warning) or `terraform-provider-dynatrace -export` (naming workflows, segments, documents and platform SLOs explicitly — they are excluded by default) on the source tenant. Keep the full backup. | Critical |
| Re-map hardcoded entity IDs | Entity IDs do not carry over. Use an entity selector only where the target schema has an `entitySelector` field; otherwise map source IDs to target IDs or parameterise them per environment (AUTOM-08 §3). | Critical |
| Validate counts after each migration step | Compare per-type config counts between two `monaco download`s — source and target, same credentials including a platform token (AUTOM-08 §6). Settings API counts cover classic schemas only, and only while both tenants are on the same generation. Note: `fetch dt.settings` is not valid DQL. | Critical |
| Test migration in a non-production tenant first | Deploy exported config to a dev/test tenant before touching production. | Critical |
| Re-enter credentials manually | Credential Vault entries, API tokens, SSL certificates, and cloud integration secrets do not migrate. Re-create them in the target. | Critical |
| Use Monaco for SaaS-to-SaaS migration | Download from source; write a target manifest with `auth.token` plus `auth.platformToken` (or `auth.oAuth`) whose project path is the downloaded `project/` folder; `monaco deploy --dry-run`; deploy (AUTOM-08 §3). | Recommended |
| Use SaaS Upgrade Assistant for Managed-to-SaaS | Install it from Dynatrace Hub in the target SaaS environment, then upload the environment export from the Managed Cluster Management Console (Environments → Export configuration). It tracks progress, flags failed configurations, and supports bulk edit and partial deployment (AUTOM-08 §5). | Recommended |
| Document non-portable configurations | Keep a checklist of configs that need manual re-creation: credentials, historic data, private locations, cloud integrations. | Recommended |
| Use Terraform export for brownfield imports | Export existing configs to HCL, then manage via Terraform going forward. | Optional |

> <sub>**Sources:** [SaaS Upgrade Assistant (DT docs)](https://docs.dynatrace.com/managed/upgrade/saas-upgrade-assistant) — *"Choose which configurations you want to migrate and run partial deployment."*; [Manage resources — auth section (DT docs)](https://docs.dynatrace.com/docs/deliver/configuration-as-code/monaco/configuration/monaco-manage-resources) — *"Access tokens and platform tokens are not interchangeable."*</sub>

---

<a id="tool-selection"></a>
## 10. Tool Selection

| Use Case | Use This Tool | Priority |
|----------|--------------|----------|
| One-off configuration change | Settings API (direct REST) | Critical |
| Repeatable config deployments with GitOps | Monaco | Critical |
| Full IaC with state management and drift detection | Terraform | Critical |
| Event-driven automation and auto-remediation | Dynatrace Workflows | Critical |
| Custom reporting, complex logic, or integrations | TypeScript `@dynatrace-sdk/*` (apps, functions) or the REST APIs (Python `requests`) — there is no official Python SDK | Critical |
| Tenant-to-tenant migration | Monaco (download/deploy pattern) | Critical |
| Managed-to-SaaS migration | SaaS Upgrade Assistant | Recommended |
| AI-assisted querying and triage | Dynatrace MCP Server | Optional |
| Team already uses Terraform for infrastructure | Terraform (extend existing IaC) | Recommended |
| Team needs lowest barrier to entry | Monaco (YAML, simple CLI) | Recommended |

### Anti-Patterns

| Anti-Pattern | Consequence | Correct Approach |
|-------------|-------------|------------------|
| Mixing Monaco and Terraform for the same config type | State conflicts, unpredictable drift | Choose one tool per config domain |
| Manual UI changes alongside automation | Configuration drift, lost changes | All changes go through the pipeline |
| No version control for configs | No rollback, no audit trail | Store everything in Git |
| Hardcoded secrets in config files | Security breach risk | Use env vars, Vault, or secret managers |
| Distributing environment-wide write tokens to teams | No access isolation | Use brokered self-service or scoped OAuth |
| Using `fetch dt.metrics` in DQL | Invalid command | Use `timeseries` for metrics |

---

<a id="community-resources"></a>
## 11. Community Resources

Key GitHub repositories organized by tool, providing starter templates, working examples, and hands-on exercises.

### Monaco Repositories

| Repository | Description |
|------------|-------------|
| [dynatrace-configuration-as-code](https://github.com/Dynatrace/dynatrace-configuration-as-code) | Official Monaco CLI (v2.30.0, released 09/23/2026, at time of writing) |
| [community-examples — configuration-as-code](https://github.com/Dynatrace/community-examples/tree/main/configuration-as-code) | Monaco samples, e.g. `basic-templates-monaco`, pipeline-observability and access-control configs. Replaces the archived `dynatrace-configuration-as-code-samples` repo |
| [easytrade](https://github.com/Dynatrace/easytrade) | Real-world Monaco project structure (manifest.yaml, detection rules, workflows) |
| [Dynatrace-Config-Manager](https://github.com/Dynatrace/Dynatrace-Config-Manager) | **Archived, no longer maintained** — former GUI tool for tenant-to-tenant config migration; historical reference only |
| [monaco-self-paced-exercises](https://github.com/dynatrace-ace/monaco-self-paced-exercises) | 6 structured hands-on exercises |

### Terraform Repositories

| Repository | Description |
|------------|-------------|
| [terraform-provider-dynatrace](https://github.com/dynatrace-oss/terraform-provider-dynatrace) | Official provider (v1.105.0 released 09/23/2026 at time of writing — check the registry for newer) with export capability |
| [community-examples — configuration-as-code](https://github.com/Dynatrace/community-examples/tree/main/configuration-as-code) | Terraform samples, e.g. `basic-templates-terraform`, `terraform_modules`, `terraform_dql_example`, `terraform_team_onboarding`, `iam_tf_sample` |

### CI/CD & Platform Engineering

| Repository | Description |
|------------|-------------|
| [dynatrace-automation-tools](https://github.com/Dynatrace/dynatrace-automation-tools) | **Archived, no longer maintained** — former SRG + Events CLI for CI/CD pipelines; its README points to [dtctl](https://github.com/dynatrace-oss/dtctl) for similar use cases |
| [platform-engineering-demo](https://github.com/dynatrace-perfclinics/platform-engineering-demo) | Full IDP reference: ArgoCD + Backstage + Keptn + Dynatrace |
| [demo-crossplane](https://github.com/Dynatrace/demo-crossplane) | Crossplane + Terraform GitOps pattern |
| [monaco-demo](https://github.com/dt-demos/monaco-demo) | Working GitHub Actions workflow for Monaco deploy |
| [obslab-release-validation](https://github.com/Dynatrace/obslab-release-validation) | Release validation with k6, business events, and SRG |

> **Archive status checked 10/02/2026.** `dynatrace-configuration-as-code-samples`, `Dynatrace-Config-Manager` and `dynatrace-automation-tools` are archived. The [samples repo's README (Dynatrace GitHub)](https://github.com/Dynatrace/dynatrace-configuration-as-code-samples) says *"They now live in Dynatrace Community Examples"*, under its `configuration-as-code/` folder — use that folder instead.

---

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
