# AUTOM-04: Terraform Provider

> **Series:** AUTOM — Dynatrace Automation | **Notebook:** 4 of 9 | **Created:** January 2026 | **Last Updated:** 05/11/2026

The Dynatrace Terraform provider enables infrastructure-as-code management of Dynatrace configurations. It integrates with Terraform's ecosystem for state management, planning, and CI/CD integration.

---

## Table of Contents

1. [Introduction](#introduction)
2. [Getting Started](#getting-started)
3. [Provider Configuration](#provider-configuration)
4. [Resource Types](#resource-types)
5. [State Management](#state-management)
6. [Advanced Patterns](#advanced-patterns)
7. [Governance Architecture](#governance-architecture)

---

## Prerequisites

Before starting this notebook, ensure you have:

| Requirement | Description |
|-------------|-------------|
| Terraform CLI | Version 1.0+ installed |
| Authentication | One or more of: **API Token** (classic), **Platform Token**, or **OAuth Client** (see [Provider Configuration](#provider-configuration)) |
| Tenant URL | Your Dynatrace SaaS tenant URL |
| HCL Knowledge | Basic familiarity with Terraform syntax |

### Authentication Methods — Three Token Types

The Dynatrace Terraform provider supports three authentication methods. Each covers a different set of resources:

| Token Type | Format | Covers | Cannot Cover |
|------------|--------|--------|-------------|
| **API Token** (classic) | `dt0c01.xxxx` | Settings 2.0, Synthetics, SLOs | Gen3 Platform (workflows, documents, segments) |
| **Platform Token** | `dt0s16.xxxx` | Settings 2.0 + Gen3 Platform | Synthetics, SLOs (removed in v1.88.0) |
| **OAuth Client** | Client ID + Secret | Gen3 Platform, IAM, Settings 2.0 (`HTTP_OAUTH_PREFERENCE`) | Synthetics (v1.88.0+) |

> **Important (v1.88.0):** As of Dynatrace Terraform provider **v1.88.0**, OAuth-based authentication (including Platform Tokens acting as OAuth) **can no longer manage synthetic monitors or SLO definitions**. These resources require a classic **API Token** with the appropriate scopes. When both tokens are configured, the provider automatically uses the correct one for each resource.

> **Recommended setup:** Use **Platform Token + API Token** together for full resource coverage. The Platform Token handles Settings 2.0 and Gen3 resources; the API Token handles Synthetics and SLOs.

---

**Preference order (current Dynatrace guidance):**

1. **Platform Token** — recommended default for most integrations. Pair with `DYNATRACE_HTTP_OAUTH_PREFERENCE=true` to authenticate to Gen3 Platform resources. Any user can create one (no admin required); long-lived; inherits the creating user's privileges.
2. **Classic API Token** — legacy; being phased out. Use only for surfaces Platform Token does not yet cover (synthetic monitors primarily).
3. **OAuth Client** — specialized. Use for external-system integrations and account-level IAM automation (policies, groups, service users). Requires account admin to create.

## Learning Objectives

By the end of this notebook, you will:

- Understand the Dynatrace Terraform provider
- Know how to configure resources in HCL
- Be able to manage state and handle drift
- Implement multi-environment deployments

---

<a id="introduction"></a>
## 1. Introduction
### Why Terraform?

| Benefit | Description |
|---------|-------------|
| **State Management** | Track what exists vs. what's defined |
| **Drift Detection** | Identify manual changes |
| **Planning** | Preview changes before applying |
| **Ecosystem** | Integrate with other Terraform providers |
| **Modules** | Reusable configuration packages |

### Terraform vs Monaco

| Aspect | Terraform | Monaco |
|--------|-----------|--------|
| State file | Required | None |
| Drift detection | Built-in | Manual |
| Learning curve | Higher | Lower |
| Multi-cloud | Yes | Dynatrace only |
| Dependencies | Explicit | Implicit |

---

<a id="getting-started"></a>
## 2. Getting Started
### Installation

**Install Terraform:**

macOS:
```bash
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
```

Linux:
```bash
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform
```

**Verify installation:**
```bash
terraform version
```

---

<a id="provider-configuration"></a>
## 3. Provider Configuration

The Dynatrace Terraform provider supports three authentication methods. Which you need depends on the resources you manage.

### Method 1: API Token (Classic)

Use for **Settings 2.0** resources and resources that require classic API token auth (synthetic monitors, SLOs, legacy config APIs).

```hcl
terraform {
  required_providers {
    dynatrace = {
      source  = "dynatrace-oss/dynatrace"
      version = "~> 1.93"        # Current: v1.93.0 (March 2026)
    }
  }
}

provider "dynatrace" {
  dt_env_url   = var.dynatrace_url
  dt_api_token = var.dynatrace_token  # Classic API token (dt0c01.xxxx)
}
```

> **Important:** Synthetic monitors and SLO definitions require a classic API Token (`dt0c01.*`). OAuth/Platform Token authentication does not support these resource types as of provider v1.88.0.

#### API Token Scopes — Full Access Reference

To manage **all** resources that require API Token authentication, create a token with the following scopes:

| Scope | Purpose |
|-------|---------|
| `settings.read` | Read Settings 2.0 objects |
| `settings.write` | Create/update Settings 2.0 objects |
| `ReadConfig` | Read legacy configuration API |
| `WriteConfig` | Write legacy configuration API |
| `CaptureRequestData` | Request attributes and data privacy |
| `ExternalSyntheticIntegration` | Synthetic monitors (v1 API) |
| `activeGateTokenManagement.create` | Create ActiveGate tokens |
| `activeGateTokenManagement.read` | Read ActiveGate tokens |
| `activeGateTokenManagement.write` | Update/revoke ActiveGate tokens |
| `apiTokens.read` | Read API tokens |
| `apiTokens.write` | Create/update API tokens |
| `attacks.read` | Read Application Security attacks |
| `attacks.write` | Write Application Security attacks |
| `credentialVault.read` | Read credential vault entries |
| `credentialVault.write` | Create/update credential vault entries |
| `entities.read` | Read monitored entities |
| `extensions.write` | Upload Extensions 2.0 |
| `extensionEnvironment.read` | Read extension environment config |
| `extensionEnvironment.write` | Write extension environment config |
| `networkZones.read` | Read network zones |
| `networkZones.write` | Create/update network zones |
| `securityProblems.read` | Read security problems |
| `securityProblems.write` | Update security problems |
| `slo.read` | Read SLO definitions |
| `slo.write` | Create/update SLO definitions |

> **Principle of least privilege:** For production, grant only the scopes your pipeline actually needs. The table above represents the **full-access superset** as documented in the [Terraform Registry](https://registry.terraform.io/providers/dynatrace-oss/dynatrace/latest/docs). A pipeline managing only Settings 2.0 resources needs only `settings.read` + `settings.write`.

### Method 2: Platform Token

**Platform Tokens** (`dt0s16.xxxx`) are a newer token type that bridges Settings 2.0 and Gen3 Platform in a single credential. They work within the assigned user's permissions.

```hcl
provider "dynatrace" {
  # Platform Token handles Settings 2.0 + Gen3 Platform resources
  # Set via DYNATRACE_ENV_URL and DYNATRACE_PLATFORM_TOKEN env vars
}
```

**Key env var:** `DYNATRACE_HTTP_OAUTH_PREFERENCE=true` tells the provider to use the Platform Token for OAuth-based resources (workflows, documents, segments) in addition to Settings 2.0.

> **How `DYNATRACE_HTTP_OAUTH_PREFERENCE` works:** When set to `true` and OAuth/Platform Token credentials are provided, the provider **prefers REST endpoints that support OAuth** over API Token endpoints. When not set (or `false`), the provider defaults to API Token authentication. Not all resources support OAuth — for example, `dynatrace_json_dashboard` can only be configured using API Tokens regardless of this setting.

> **Settings object ownership:** When a settings object is created using Platform Token or OAuth credentials, the owner is set to the credential owner. By default, the object is **private** — only the owner can read/modify it. Use the `dynatrace_settings_permissions` resource to manage access modifiers.

### Method 3: OAuth Client Credentials

**Required** for Automation (Workflows), Document, and Account Management (IAM) resources. The provider exchanges your client ID and secret for short-lived OAuth access tokens automatically.

```hcl
provider "dynatrace" {
  dt_env_url   = var.dynatrace_url
  dt_api_token = var.dynatrace_token  # For settings/classic resources

  # OAuth credentials — required for automation, document, and IAM resources
  client_id     = var.oauth_client_id        # Falls back for both automation + IAM
  client_secret = var.oauth_client_secret
  account_id    = var.account_id             # Required for IAM resources
}
```

> **Tip:** If you only manage automation/document resources, you can omit `dt_api_token` and use OAuth alone.

### Method 4: Combined Auth — Full Coverage (Recommended)

For **full resource coverage**, use **Platform Token + API Token** together. The provider automatically uses the correct token for each resource type:

```hcl
# provider.tf — Combined auth for full coverage
provider "dynatrace" {
  # Auth is read from environment variables automatically.
  # The provider uses the correct token for each resource type.
}
```

```bash
# --- Platform Token (covers Settings 2.0 + Gen3 Platform) ---
export DYNATRACE_ENV_URL="https://abc12345.live.dynatrace.com"
export DYNATRACE_PLATFORM_TOKEN="dt0s16.xxxx.yyyy"
export DYNATRACE_HTTP_OAUTH_PREFERENCE=true

# --- API Token (covers Synthetics — v1.88.0 requirement) ---
export DYNATRACE_API_TOKEN="dt0c01.xxxx.yyyy"
```

| Resource | Token Used |
|----------|------------|
| Settings 2.0 (auto-tags, management zones, alerting) | Platform Token |
| Gen3 Platform (workflows, documents, segments) | Platform Token (via OAuth) |
| SLO definitions (`dynatrace_slo_v2`) | Platform Token (Settings 2.0 — `builtin:monitoring.slo`) |
| Synthetic monitors (`dynatrace_http_monitor`) | **API Token** (v1.88.0) |

### Environment Variables — Complete Reference

| Variable | Purpose |
|----------|---------|
| `DYNATRACE_ENV_URL` / `DT_ENV_URL` / `DT_ENVIRONMENT_URL` | Tenant URL |
| `DYNATRACE_API_TOKEN` / `DT_API_TOKEN` | Classic API token (`dt0c01`) |
| `DYNATRACE_PLATFORM_TOKEN` / `DT_PLATFORM_TOKEN` | Platform token (`dt0s16`) |
| `DYNATRACE_HTTP_OAUTH_PREFERENCE` | Set to `true` to prefer OAuth endpoints for Platform Token / OAuth resources |
| `DT_CLIENT_ID` / `DYNATRACE_CLIENT_ID` | OAuth client ID (for automation/document/IAM resources) |
| `DT_CLIENT_SECRET` / `DYNATRACE_CLIENT_SECRET` | OAuth client secret |
| `DT_ACCOUNT_ID` / `DYNATRACE_ACCOUNT_ID` | Account UUID (required for account-level IAM resources) |

### Which Authentication for Which Resources?

| Resource Category | API Token | Platform Token | OAuth Client |
|-------------------|-----------|----------------|-------------|
| Settings 2.0 (management zones, auto-tags, alerting, SLOs) | Yes | Yes | Yes (`HTTP_OAUTH_PREFERENCE`) |
| Synthetic monitors | **Yes (required)** | No (v1.88.0) | No (v1.88.0) |
| Gen3: Workflows, scheduling | No | Yes (with `HTTP_OAUTH_PREFERENCE`) | **Yes** |
| Gen3: Documents (dashboards, notebooks) | No | Yes (with `HTTP_OAUTH_PREFERENCE`) | **Yes** |
| Gen3: Segments | No | Yes (with `HTTP_OAUTH_PREFERENCE`) | **Yes** |
| Account Management (IAM, policies, groups) | No | No | **Yes** (`DT_ACCOUNT_ID` also needed) |
| Legacy dashboards (`dynatrace_json_dashboard`) | **Yes (only)** | No | No |

### Creating an OAuth Client

1. Go to **Account Management** > **Identity & Access Management** > **OAuth clients**
2. Create a new client with the required scopes (see table below)
3. Copy the generated **client ID** and **client secret** immediately — the secret is only shown once
4. Store credentials securely (vault, CI/CD secrets, etc.)
5. Ensure the **service user's groups** grant the same scopes as the OAuth client for all target environments

#### OAuth Client Scopes — Full Access Reference

To manage **all** resources that require OAuth authentication, create an OAuth client with the following scopes:

| Scope | Purpose |
|-------|---------|
| **Settings** | |
| `settings:objects:read` | Read settings objects |
| `settings:objects:write` | Create/update settings objects |
| `settings:objects:admin` | Admin — manage all settings objects |
| **Automation** | |
| `automation:workflows:read` | Read workflow definitions |
| `automation:workflows:write` | Create/update workflows |
| `automation:workflows:admin` | Admin — manage all workflows |
| `automation:calendars:read` | Read scheduling calendars |
| `automation:calendars:write` | Create/update scheduling calendars |
| `automation:rules:read` | Read scheduling rules |
| `automation:rules:write` | Create/update scheduling rules |
| **Documents** | |
| `document:documents:read` | Read documents (dashboards, notebooks) |
| `document:documents:write` | Create/update documents |
| `document:documents:delete` | Delete documents |
| `document:trash.documents:delete` | Permanently delete trashed documents |
| `document:direct-shares:read` | Read document shares |
| `document:direct-shares:write` | Create/update document shares |
| `document:direct-shares:delete` | Delete document shares |
| **OpenPipeline** | |
| `openpipeline:configurations:read` | Read OpenPipeline configurations |
| `openpipeline:configurations:write` | Create/update OpenPipeline configs |
| **SLO** | |
| `slo:slos:read` | Read SLO definitions |
| `slo:slos:write` | Create/update SLO definitions |
| `slo:objective-templates:read` | Read SLO objective templates |
| **Storage** | |
| `storage:bizevents:read` | Read business events |
| `storage:bucket-definitions:read` | Read Grail bucket definitions |
| `storage:bucket-definitions:write` | Create/update Grail bucket definitions |
| `storage:filter-segments:read` | Read segments |
| `storage:filter-segments:write` | Create/update segments |
| `storage:filter-segments:share` | Share segments |
| `storage:filter-segments:delete` | Delete segments |
| `storage:filter-segments:admin` | Admin — manage all segments |
| **Account Management (IAM)** | |
| `account-idm-read` | Read IAM users, groups, service users |
| `account-idm-write` | Create/update IAM users, groups, service users |
| `iam-policies-management` | Create/update/delete IAM policies |
| `account-env-read` | Read account environment metadata |

> **Principle of least privilege:** The table above is the **full-access superset** from the [Terraform Registry](https://registry.terraform.io/providers/dynatrace-oss/dynatrace/latest/docs). For production, grant only the scopes your pipeline requires. A pipeline managing only workflows needs `automation:workflows:read` + `automation:workflows:write`.

> **Security:** Never commit OAuth client secrets or API tokens to version control. Use environment variables, HashiCorp Vault, AWS Secrets Manager, or your CI/CD platform's secret store.

> **Note on scope formats:** API token scopes use dot notation (`settings.read`, `settings.write`). OAuth scopes and IAM policy actions use colon notation (`settings:objects:read`, `settings:objects:write`). These are different identifiers for the same capability in different auth contexts.

### Initialize and Validate

```bash
# Initialize provider
terraform init

# Validate configuration
terraform validate

# Format code
terraform fmt
```

---

<a id="resource-types"></a>
## 4. Resource Types
### Management Zone

```hcl
resource "dynatrace_management_zone_v2" "production" {
  name = "Production"

  rules {
    type    = "SERVICE"
    enabled = true

    conditions {
      condition {
        key {
          type      = "STATIC"
          attribute = "SERVICE_TAGS"
        }
        tag {
          operator = "EQUALS"
          negate   = false
          value {
            context = "CONTEXTLESS"
            key     = "environment"
            value   = "production"
          }
        }
      }
    }
  }
}
```

### Auto-Tagging Rule

```hcl
resource "dynatrace_autotag_v2" "application" {
  name = "Application"

  rules {
    type         = "SERVICE"
    enabled      = true
    value_format = "{Service:DetectedName}"
    conditions   = []
  }
}
```

---

### Alerting Profile

```hcl
resource "dynatrace_alerting" "production_alerts" {
  name            = "Production Alerting"
  management_zone = dynatrace_management_zone_v2.production.id

  rules {
    delay_in_minutes = 0
    severity_level   = "AVAILABILITY"
    tag_filters      = []
  }

  rules {
    delay_in_minutes = 5
    severity_level   = "ERRORS"
    tag_filters      = []
  }

  rules {
    delay_in_minutes = 10
    severity_level   = "PERFORMANCE"
    tag_filters      = []
  }
}
```

### SLO

```hcl
resource "dynatrace_slo_v2" "availability" {
  name              = "Production Availability"
  enabled           = true
  evaluation_type   = "AGGREGATE"
  evaluation_window = "-1w"
  target_success    = 99.9
  target_warning    = 99.95
  
  metric_expression = "builtin:synthetic.http.availability.location.total:splitBy()"
  
  filter            = "type(SYNTHETIC_TEST)"
}
```

---

### HTTP Monitor (Synthetic)

```hcl
resource "dynatrace_http_monitor" "homepage" {
  name      = "Homepage Check"
  enabled   = true
  frequency = 5

  locations = ["GEOLOCATION-1234567890ABCDEF"]

  anomaly_detection {
    loading_time_thresholds {
      enabled = true
    }
    outage_handling {
      global_outage = true
      local_outage  = false
    }
  }

  script {
    request {
      description = "Homepage"
      method      = "GET"
      url         = "https://example.com"

      validation {
        rule {
          type  = "httpStatusesList"
          value = ">=400"
          pass_if_found = false
        }
      }
    }
  }
}
```

---

### Gen3 Platform Resources

Gen3 resources require **Platform Token** (with `DYNATRACE_HTTP_OAUTH_PREFERENCE=true`) or **OAuth Client Credentials**.

#### Automation Workflow

```hcl
resource "dynatrace_automation_workflow" "problem_email" {
  title       = "Problem Email Notification"
  description = "Sends an email when a Davis problem opens"
  private     = false

  trigger {
    event {
      active = true
      config {
        davis_problem {
          categories {
            error = true
          }
          entity_tags = {
            Environment = "production"
          }
          entity_tags_match = "all"
          on_problem_close  = false
        }
      }
    }
  }

  tasks {
    task {
      name        = "send_email"
      description = "Send email notification"
      action      = "dynatrace.email:email-action"
      active      = true
      input = jsonencode({
        to      = ["team@example.com"]
        subject = "Dynatrace Problem: {{event()['event.name']}}"
        body    = "Problem detected.\nName: {{event()['event.name']}}\nSeverity: {{event()['event.category']}}"
      })
      position {
        x = 0
        y = 1
      }
    }
  }
}
```

#### Grail Dashboard (Document)

```hcl
resource "dynatrace_document" "team_dashboard" {
  type    = "dashboard"
  name    = "Production Overview"
  private = false
  content = jsonencode({
    version = 13
    variables = []
    tiles = {
      "tile-1" = {
        type  = "data"
        title = "Error Rate"
        query = "timeseries avg(dt.service.request.failure_rate), by:{dt.entity.service}"
      }
    }
    layouts = {
      "tile-1" = { x = 0, y = 0, w = 12, h = 6 }
    }
  })
}
```

#### Grail Notebook (Document)

```hcl
resource "dynatrace_document" "runbook" {
  type    = "notebook"
  name    = "Incident Runbook"
  private = false
  content = jsonencode({
    version = "3"
    sections = [
      {
        id    = "section-1"
        type  = "markdown"
        title = "Investigation Steps"
        content = "## Step 1: Check Error Rate\nRun the query below to identify affected services."
      },
      {
        id    = "section-2"
        type  = "dql"
        title = "Error Rate by Service"
        content = "timeseries avg(dt.service.request.failure_rate), by:{dt.entity.service}"
      }
    ]
  })
}
```

#### Segment (Gen3 Management Zone Replacement)

```hcl
resource "dynatrace_segment" "production_services" {
  name            = "Production Services"
  description     = "All services in the production environment"
  is_public       = true
  include_filter  = "fetch dt.entity.service | filter tags = \"Environment:production\""
}
```

#### Maintenance Window

```hcl
resource "dynatrace_maintenance" "weekly_patch" {
  enabled = true
  name    = "Weekly Patch Window"
  type    = "PLANNED"
  suppression = "DONT_DETECT_PROBLEMS"

  schedule {
    type = "WEEKLY"
    weekly_recurrence {
      day_of_week    = "SUNDAY"
      time_window {
        start_time = "02:00"
        end_time   = "04:00"
      }
      recurrence_range {
        start_date = "2026-01-01"
      }
    }
  }
}
```

---

<a id="state-management"></a>
## 5. State Management
### Understanding Terraform State

Terraform tracks resources in a state file (`terraform.tfstate`):

| Concept | Description |
|---------|-------------|
| **State** | JSON file mapping config to real resources |
| **Plan** | Comparison of state vs. desired config |
| **Apply** | Execute changes to reach desired state |
| **Drift** | Difference between state and actual |

### Remote State Storage

For team collaboration, use remote state:

```hcl
terraform {
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "dynatrace/production/terraform.tfstate"
    region = "us-east-1"
  }
}
```

Or using Terraform Cloud:

```hcl
terraform {
  cloud {
    organization = "my-org"
    workspaces {
      name = "dynatrace-production"
    }
  }
}
```

---

### Common State Commands

```bash
# Preview changes
terraform plan

# Apply changes
terraform apply

# Show current state
terraform show

# List resources in state
terraform state list

# Import existing resource
terraform import dynatrace_management_zone_v2.production "<object-id>"

# Remove resource from state (without deleting)
terraform state rm dynatrace_management_zone_v2.production
```

### Importing Existing Resources

To manage existing configurations:

1. Write the resource block in HCL
2. Find the object ID in Dynatrace
3. Import into state

```bash
terraform import dynatrace_management_zone_v2.production "vu9U3hXa3q0AAAABABhidWlsdGluOm1hbmFnZW1lbnQtem9uZXMABnRlbmFudAAGdGVuYW50ABp2dTlVM2hYYTNxMERUVF9fUHJvZHVjdGlvbr7vFJ4"
```

---

<a id="advanced-patterns"></a>
## 6. Advanced Patterns
### Modules for Reusability

Create reusable modules:

**modules/environment/main.tf:**
```hcl
variable "environment_name" {
  type = string
}

variable "environment_tag" {
  type = string
}

resource "dynatrace_management_zone_v2" "zone" {
  name = var.environment_name

  rules {
    type    = "SERVICE"
    enabled = true

    conditions {
      condition {
        key {
          type      = "STATIC"
          attribute = "SERVICE_TAGS"
        }
        tag {
          operator = "EQUALS"
          negate   = false
          value {
            context = "CONTEXTLESS"
            key     = "environment"
            value   = var.environment_tag
          }
        }
      }
    }
  }
}

output "management_zone_id" {
  value = dynatrace_management_zone_v2.zone.id
}
```

**Use the module:**
```hcl
module "production" {
  source           = "./modules/environment"
  environment_name = "Production"
  environment_tag  = "production"
}

module "staging" {
  source           = "./modules/environment"
  environment_name = "Staging"
  environment_tag  = "staging"
}
```

---

### Workspaces for Environments

Use workspaces to manage multiple environments:

```bash
# Create workspaces
terraform workspace new development
terraform workspace new staging
terraform workspace new production

# Switch workspace
terraform workspace select production

# List workspaces
terraform workspace list
```

**Use workspace in config:**
```hcl
locals {
  environment = terraform.workspace
  
  config = {
    development = {
      alert_delay = 30
    }
    staging = {
      alert_delay = 15
    }
    production = {
      alert_delay = 5
    }
  }
}

resource "dynatrace_alerting" "alerts" {
  name = "${local.environment} Alerting"
  
  rules {
    delay_in_minutes = local.config[local.environment].alert_delay
    severity_level   = "AVAILABILITY"
  }
}
```

---

### Modules with Governance Inputs

For multi-team environments, modules can enforce governance through input validation, mandatory tags, and scoping:

```hcl
# modules/alerting-profile/variables.tf

variable "team_name" {
  type        = string
  description = "Name of the team owning this alerting profile"
  validation {
    condition     = can(regex("^[a-z][a-z0-9-]{2,20}$", var.team_name))
    error_message = "Team name must be lowercase alphanumeric with hyphens, 3-21 characters."
  }
}

variable "management_zone_name" {
  type        = string
  description = "Management zone to scope this profile to (required)"
  validation {
    condition     = length(var.management_zone_name) > 0
    error_message = "Management zone name is required."
  }
}

variable "environment" {
  type        = string
  description = "Target environment"
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be one of: dev, staging, prod."
  }
}

variable "error_delay_minutes" {
  type    = number
  default = 0
  validation {
    condition     = contains([0, 5, 10, 15, 30], var.error_delay_minutes)
    error_message = "Error delay must be one of: 0, 5, 10, 15, 30 minutes."
  }
}
```

This pattern ensures teams cannot create unscoped or untagged resources — governance is built into the module itself.

### IAM Policy Management via Terraform

The Dynatrace Terraform provider can manage IAM policies, groups, and bindings. This enables **schema-level access restrictions** — something API tokens cannot do.

> **Important:** Managing IAM resources requires OAuth client credentials with `DT_ACCOUNT_ID`. API tokens cannot manage IAM.

```hcl
# Create a policy that restricts a team to specific settings schemas
resource "dynatrace_iam_policy" "team_settings" {
  name            = "Payments Team - Settings Access"
  environment     = var.dynatrace_environment_id
  statement_query = <<-EOT
    ALLOW settings:objects:read, settings:objects:write
      WHERE settings:schemaId IN (
        "builtin:alerting.profile",
        "builtin:problem.notifications",
        "builtin:maintenance-window"
      );
  EOT
}

# Create an IAM group for the team
resource "dynatrace_iam_group" "payments_team" {
  name        = "Payments Team"
  description = "IAM group for the Payments team"
}

# Bind the policy to the group
resource "dynatrace_iam_policy_bindings_v2" "payments_binding" {
  group = dynatrace_iam_group.payments_team.id

  policy {
    id          = dynatrace_iam_policy.team_settings.id
    environment = var.dynatrace_environment_id
  }
}
```

IAM policies support multiple condition operators:

| Operator | Example | Use Case |
|----------|---------|----------|
| `IN` | `settings:schemaId IN ("builtin:alerting.profile", ...)` | Explicit list |
| `startsWith` | `settings:schemaId startsWith "builtin:alerting"` | Schema family |
| `contains` | `settings:schemaId contains "custom"` | Substring match |
| `=` | `storage:bucket.name = "team_logs"` | Data isolation |

> **Key insight:** The API token scope `settings.write` grants access to ALL schemas. IAM policies with `WHERE settings:schemaId` clauses (using the IAM action `settings:objects:write`) are the only way to restrict schema access at the platform level.

---

<a id="governance-architecture"></a>
## 7. Governance Architecture

### Token Scoping & the Synthetic Access Problem

A common enterprise requirement is **scoped API access** — allowing Team A to manage only their Synthetic monitors via Terraform while preventing them from modifying Team B's monitors. Today, Dynatrace does not provide object-level or team-scoped API access for Synthetic Monitors because they are managed via the **Classic API v1**, not the modern platform APIs.

Here is why each token approach fails for scoped Synthetic access:

| Token Approach | Why It Fails for Synthetic Scoping |
|----------------|-----------------------------------|
| **Platform Token** (cluster page) | Cannot express v1 Synthetic permissions — those APIs are not covered by platform scopes |
| **OAuth Client** | The v1 Synthetic API **does not accept OAuth bearer tokens**. It requires a classic API token (`dt0c01`) with `ExternalSyntheticIntegration` scope. OAuth works for Gen3/platform resources only. |
| **Personal Access Token** | Still a classic API token — scopes are broad and environment-wide |
| **Environment Token** | Cannot be restricted to specific objects or teams. This is how classic tokens work by design. |

> **Key insight:** There is no supported way today to say "Team A can manage only their Synthetic monitors via API." For Gen3/platform resources (workflows, documents, segments, Settings 2.0), Service User + OAuth provides genuinely scoped access through IAM policies. Synthetics remain the gap.

### Dual-Auth: Bridging Gen3 and v1

When a pipeline manages both Gen3 resources (scoped via OAuth) and v1 resources (synthetics, requiring a classic API token), use **dual authentication**:

```hcl
provider "dynatrace" {
  dt_env_url       = var.dynatrace_env_url

  # Service User + OAuth — for Gen3/platform resources (scoped via IAM)
  client_id     = var.dt_client_id           # Canonical attribute (v1.93+)
  client_secret = var.dt_client_secret
  account_id    = var.dt_account_id

  # Classic API Token — still required for v1 resources (synthetics)
  dt_api_token     = var.dynatrace_api_token
}
```

The service user's group memberships and IAM policies provide **real, scoped access** for Gen3 resources. The classic API token covers synthetics with compensating controls (see workaround patterns below).

> **Migration timeline:** Dynatrace has been progressively migrating features off v1 APIs. When Synthetic monitoring moves to platform APIs with IAM policy-aware services, the OAuth client will cover it and the dual-auth requirement disappears. Track [Dynatrace release notes](https://docs.dynatrace.com/docs/whats-new) for updates.

---

### Synthetic Monitor Workaround Patterns

Since OAuth and Sentinel cannot solve scoped Synthetic access via the v1 API, these are the realistic enterprise patterns:

#### Pattern 1: Brokered Self-Service (Recommended)

Teams submit **declarative requests** (YAML, Terraform variables, or JSON) describing their desired Synthetic monitors. A **central pipeline** owns the environment-wide API token, validates team intent, and applies synthetics on their behalf.

![Brokered Self-Service — Synthetic Monitor Workaround Pattern](images/04-brokered-self-service-pattern_930x500.png)

<!-- MARKDOWN_TABLE_ALTERNATIVE
| Team Repo (product team) | Central Terraform Repo (platform team) |
|---|---|
| Declares desired synthetics (YAML / Terraform vars / JSON intent) | Owns the environment-wide API token (long-lived, rotated) |
| No Dynatrace credentials in the repo | Sentinel/OPA enforces team ownership tags, naming conventions, management-zone scoping, allowed locations |
| Repo compromise cannot issue Dynatrace API calls | Applies synthetics on behalf of teams |
| | Result: synthetics in Dynatrace with mandatory tags + management-zone scoping; ownership auditable via tags |
For environments where SVG doesn't render
-->

> **Key principle:** Teams never get direct API access. They get **intent-based self-service**, not credentials.

#### Pattern 2: Management Zone Fencing (Soft Isolation)

Every Synthetic monitor must include a mandatory tag (e.g., `team=payments`) and be bound to a management zone. Terraform modules hard-code the MZ ID and naming prefix. Sentinel or OPA ensures the team repo can only reference its own MZ.

> **Limitation:** This is **policy enforcement**, not permission enforcement. A compromised pipeline token still has full environment-wide access.

#### Pattern 3: Split Environments (True Isolation)

One Dynatrace environment per business unit, platform, or trust boundary. Synthetic API tokens become effectively scoped by environment since each environment has its own token. This is heavy-handed but the **only way to get true security isolation** for Synthetic monitors today. Recommended in regulated or multi-tenant scenarios.

#### Pattern 4: UI Self-Service + API Read-Only

Teams create Synthetic monitors via the Dynatrace UI, where RBAC applies and scoping works. The Terraform pipeline is limited to **reads, drift reporting, and configuration auditing** — not writes. This avoids distributing broad write tokens entirely.

#### What Does Not Work

These approaches **do not** overcome the v1 API limitation:

- Per-team OAuth clients for Synthetics (OAuth works for Gen3, not v1 APIs)
- Sentinel-only enforcement without pipeline architecture (Sentinel constrains Terraform plans, not API permissions)
- Terraform modules alone without CI governance (modules can be bypassed without pipeline guardrails)
- Expecting platform tokens to cover Classic API endpoints

---

### Best Practices

| Practice | Description |
|----------|-------------|
| **Remote state** | Never use local state for teams |
| **State locking** | Enable to prevent concurrent changes |
| **Modular design** | Use modules for reusable patterns |
| **Version pinning** | Pin provider versions |
| **Plan before apply** | Always review plan output |
| **Meaningful names** | Resource names should be descriptive |
| **Synthetic access** | Use brokered self-service for v1 API resources; never distribute environment-wide write tokens to teams |

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| "Resource already exists" | State out of sync | Import the resource |
| "Provider error" | Invalid token | Check credentials |
| "Drift detected" | Manual changes | Re-apply or update state |
| Slow plan/apply | Many resources | Use targets or modules |

---

<a id="next-steps"></a>
## 8. Next Steps

### Terraform Export Utility

The provider binary itself can export existing Dynatrace configuration to HCL `.tf` files:

```bash
# Download the provider binary and use it as an export tool
# See: https://dt-url.net/h203qmc

# Export all configurations
terraform-provider-dynatrace \
  -export \
  -id all \
  -target-folder ./exported-config
```

> **Environment variables required:** Set `DYNATRACE_ENV_URL` and `DYNATRACE_API_TOKEN` (and optionally `DT_CLIENT_ID`, `DT_CLIENT_SECRET`, `DT_ACCOUNT_ID` for Gen3/IAM resources) before running the export.

### Continue the Series

| Next Notebook | Focus |
|---------------|-------|
| **AUTOM-05: Dynatrace Workflows** | Event-driven automation |

### Additional Resources

- [Terraform Provider Documentation](https://registry.terraform.io/providers/dynatrace-oss/dynatrace/latest/docs) (v1.93.0, March 2026)
- [Provider GitHub Repository](https://github.com/dynatrace-oss/terraform-provider-dynatrace)
- [Terraform Style Guide](https://developer.hashicorp.com/terraform/language/style)
- [Dynatrace OAuth Client Guide](https://docs.dynatrace.com/docs/deliver/configuration-as-code/terraform/guides/create-oauth-client)

---

## Summary

In this notebook, you learned:

- How to configure the Dynatrace Terraform provider with three authentication methods
- Creating resources like management zones, auto-tags, SLOs, and Gen3 platform resources
- State management and drift detection
- Advanced patterns with modules, workspaces, and governance inputs
- Why Synthetic monitors require classic API tokens (v1 API limitation) and workaround patterns
- IAM policy management for schema-level access restrictions
- Brokered self-service as the recommended pattern for governed Synthetic access

> **Key Takeaway:** Use Service User + OAuth for Gen3 resources where IAM provides real scoped access. For Synthetic monitors (v1 API), use brokered self-service with compensating controls — never distribute environment-wide write tokens directly to teams.

---

*Continue to **AUTOM-05: Dynatrace Workflows** to learn event-driven automation.*

## Community Resources & Examples

The following GitHub repositories provide starter templates, reusable modules, and real-world examples for the Dynatrace Terraform provider:

### Official Repositories

| Repository | Description |
|------------|-------------|
| [terraform-provider-dynatrace](https://github.com/dynatrace-oss/terraform-provider-dynatrace) | Official provider (v1.93.0, 180 releases) -- supports hundreds of resource types with export capability |
| [dynatrace-configuration-as-code-samples](https://github.com/Dynatrace/dynatrace-configuration-as-code-samples) | Official samples repo with 10 Terraform starter templates in `basic-templates-terraform` |

### Starter Templates & Modules (in `dynatrace-configuration-as-code-samples`)

| Template Directory | What It Demonstrates |
|--------------------|----------------------|
| `basic-templates-terraform` | 10 starter resources: alerting, app detection, synthetic, SLOs, auto-tags, ownership teams |
| `terraform_modules` | Reusable module pattern for synthetic HTTP and NAM monitors |
| `terraform_dql_example` | DQL as a Terraform `data` source -- query entities and create configs dynamically |
| `terraform_team_onboarding` | IAM policies, groups, and Azure Entra ID integration for team onboarding |
| `iam_tf_sample` | IAM policies, Grail buckets, OpenPipelines, segments, and team access |

### Pipeline Observability Samples (Terraform Variants)

| Directory | CI/CD Platform |
|-----------|---------------|
| `github_pipeline_observability_terraform` | GitHub Actions |
| `gitlab_pipeline_observability_terraform` | GitLab CI |
| `argocd_observability_terraform` | ArgoCD |

### Provider Export Feature

The provider binary can export existing Dynatrace configuration to `.tf` files:

```bash
# Export all configs from a tenant as HCL
./terraform-provider-dynatrace -export -env <environment-url> -token <api-token>
```

This is the fastest path from an existing tenant to Terraform-managed configuration.

---

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
