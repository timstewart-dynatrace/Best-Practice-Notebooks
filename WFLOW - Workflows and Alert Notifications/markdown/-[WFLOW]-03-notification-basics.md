# WFLOW-03: Alert Notification Basics

> **Series:** WFLOW — Workflows and Alert Notifications | **Notebook:** 3 of 10 | **Created:** January 2026 | **Last Updated:** 09/24/2026

## Sending Notifications with Workflows
The most common workflow use case is sending alert notifications to Slack, Microsoft Teams, and email. This notebook covers setting up connections, configuring notification tasks, and best practices for effective alerting.

---

## Table of Contents

1. [Notification Architecture](#notification-architecture)
2. [Setting Up Connections](#setting-up-connections)
3. [Slack Notifications](#slack-notifications)
4. [Microsoft Teams Notifications](#microsoft-teams-notifications)
5. [Email Notifications](#email-notifications)
6. [Message Formatting](#message-formatting)
7. [Complete Alert Workflow Example](#complete-alert-workflow-example)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Dynatrace Environment** | SaaS with Platform subscription |
| **Permissions** | `automation:workflows:write`, `automation:connections:write` |
| **Prior Knowledge** | **WFLOW-01** and **WFLOW-02** |
| **External Access** | Slack workspace admin or Teams channel access |

<a id="notification-architecture"></a>
## 1. Notification Architecture
### How Workflow Notifications Work

```
Trigger (Detected Problem, Schedule, etc.)
            ↓
    Workflow Executes
            ↓
    Notification Task
            ↓
    Connection (credentials)
            ↓
    External Service (Slack, Teams, Email)
```

### Components

| Component | Purpose |
|-----------|----------|
| **Connection** | Securely stores credentials (tokens, webhooks) |
| **Notification Task** | Sends message using connection |
| **Message Template** | Dynamic content using Jinja expressions |

### Supported Notification Channels

| Channel | Task Type | Authentication |
|---------|-----------|----------------|
| Slack | `dynatrace.slack:message` (illustrative) | OAuth App or Webhook |
| Microsoft Teams | `dynatrace.msteams:message` (illustrative) | Power Automate or Webhook (deprecated) |
| Email | `dynatrace.email:send-email` | SMTP or built-in |
| PagerDuty | `dynatrace.pagerduty:*` | Integration Key |
| ServiceNow | `dynatrace.servicenow:*` | OAuth or Basic Auth |
| Jira | `dynatrace.jira:*` | API Token |
| Custom Webhook (HTTP Request) | `dynatrace.automations:http-function` | Credential Vault (Basic or Token) |

The email and HTTP identifiers are the ones the platform records in `dt.system.events` (`dt.automation_engine.action.app` / `.action.function`), and the HTTP one appears in the Jinja reference's task sample. The other action IDs in this series' YAML are illustrative: export a workflow built in the editor to get the exact identifiers.

<a id="setting-up-connections"></a>
## 2. Setting Up Connections
Connections store credentials separately from workflows for security and reusability.

### Accessing Connections

1. Open **Settings** (gear icon)
2. Navigate to **Integration** → **Connections**
3. Or use direct URL: `https://<env>/ui/apps/dynatrace.hub/connections`

### Creating a Connection

1. Click **+ Connection**
2. Select connection type (Slack, Teams, etc.)
3. Enter credentials
4. Name the connection (e.g., `slack-production-alerts`)
5. Save

### Connection Best Practices

| Practice | Description |
|----------|-------------|
| **Descriptive names** | `slack-prod-oncall` not `slack1` |
| **Environment separation** | Different connections for prod/staging |
| **Least privilege** | Minimum required scopes |
| **Regular rotation** | Update tokens periodically |

<a id="slack-notifications"></a>
## 3. Slack Notifications
### Option 1: Slack App (Recommended)

Provides richer features: channels, DMs, reactions, threads.

**Setup:**
1. Create Slack App at https://api.slack.com/apps
2. Add OAuth scopes: `chat:write`, `chat:write.public`
3. Install to workspace
4. Copy Bot User OAuth Token
5. Create connection in Dynatrace with token

### Option 2: Incoming Webhook (Simpler)

Single channel, no advanced features.

**Setup:**
1. Go to Slack channel settings → Integrations
2. Add **Incoming Webhook**
3. Copy webhook URL
4. Create connection with webhook URL

### Basic Slack Message Task

```yaml
name: send_slack_alert
type: dynatrace.slack:message
input:
  connection: slack-production
  channel: "#alerts-production"
  message: |
    :rotating_light: *Problem Detected*
    
    *Title:* {{ event()["event.name"] }}
    *Category:* {{ event()["event.category"] }}
    *Started:* {{ event()["event.start"] }}
    
    <{{ problem_link() }}|View in Dynatrace>
```

### Slack Message with Blocks

For richer formatting:

```yaml
input:
  connection: slack-production
  channel: "#alerts"
  blocks:
    - type: header
      text:
        type: plain_text
        text: "{{ ':red_circle:' if (event().get('event.severity') | int(5)) <= 1 else ':large_orange_circle:' }} {{ event()['event.category'] }} Alert"
    - type: section
      fields:
        - type: mrkdwn
          text: "*Problem:*\n{{ event()['event.name'] }}"
        - type: mrkdwn
          text: "*Status:*\n{{ event()['event.status'] }}"
    - type: section
      fields:
        - type: mrkdwn
          text: "*Started:*\n{{ event()['event.start'] }}"
        - type: mrkdwn
          text: "*ID:*\n{{ event()['display_id'] }}"
    - type: actions
      elements:
        - type: button
          text:
            type: plain_text
            text: "View Problem"
          url: "{{ problem_link() }}"
```

<a id="microsoft-teams-notifications"></a>
## 4. Microsoft Teams Notifications

> **Warning — O365 Connectors Deprecated:** Microsoft retired Office 365 Connectors (Incoming Webhooks) in late 2024. Existing webhooks may continue to work temporarily, but **new O365 Connector creation is disabled**. Use one of these alternatives instead:
>
> | Method | Status | Notes |
> |--------|--------|-------|
> | **Power Automate Workflow** | Recommended | Create a "When a Teams webhook request is received" flow in Power Automate; use the resulting URL in an HTTP Request task (`dynatrace.automations:http-function`) |
> | **Teams Workflows Connector** | Recommended | Available in new Teams client under channel **...** → **Workflows** |
> | **O365 Incoming Webhook** | Deprecated | Legacy method shown below for reference only |

### Setting Up Teams Webhook (Legacy)

1. Open target Teams channel
2. Click **...** → **Connectors** (or **Workflows** in new Teams)
3. Add **Incoming Webhook**
4. Name it (e.g., "Dynatrace Alerts")
5. Copy the webhook URL
6. Create connection in Dynatrace


### Basic Teams Message Task

```yaml
name: send_teams_alert
type: dynatrace.msteams:message
input:
  connection: teams-production
  message: |
    **Problem Detected**
    
    **Title:** {{ event()["event.name"] }}
    **Category:** {{ event()["event.category"] }}
    **Started:** {{ event()["event.start"] }}
    
    [View in Dynatrace]({{ problem_link() }})
```

### Teams Adaptive Card

For richer formatting:

```yaml
input:
  connection: teams-production
  card:
    type: AdaptiveCard
    $schema: "https://adaptivecards.microsoft.com/schemas/adaptive-card.json"
    version: "1.4"
    body:
      - type: TextBlock
        text: "{{ event()['event.category'] }} Alert"
        size: Large
        weight: Bolder
        color: "{{ 'Attention' if (event().get('event.severity') | int(5)) <= 1 else 'Warning' }}"
      - type: FactSet
        facts:
          - title: "Problem"
            value: "{{ event()['event.name'] }}"
          - title: "Status"
            value: "{{ event()['event.status'] }}"
          - title: "Started"
            value: "{{ event()['event.start'] }}"
    actions:
      - type: Action.OpenUrl
        title: "View in Dynatrace"
        url: "{{ problem_link() }}"
```

<a id="email-notifications"></a>
## 5. Email Notifications
### Email Options

| Option | Setup | Best For |
|--------|-------|----------|
| **Built-in** | No setup required | Simple notifications |
| **Custom SMTP** | Configure SMTP server | Corporate email servers |
| **SendGrid/SES** | API integration | High volume, templates |

### Basic Email Task

```yaml
name: send_email_alert
type: dynatrace.email:send-email
input:
  to:
    - "oncall@company.com"
    - "platform-team@company.com"
  subject: "[{{ event()['event.category'] }}] {{ event()['event.name'] }}"
  body: |
    A problem has been detected in your Dynatrace environment.
    
    Problem Details:
    ----------------
    Title: {{ event()["event.name"] }}
    Category: {{ event()["event.category"] }}
    Status: {{ event()["event.status"] }}
    Start Time: {{ event()["event.start"] }}
    Problem ID: {{ event()["display_id"] }}
    
    Affected Entities:
    {{ event()["affected_entity_ids"] | join(", ") }}
    
    Root Cause:
    {{ event()["root_cause_entity_id"] }}
    
    View this problem:
    {{ problem_link() }}
```

### HTML Email

```yaml
input:
  to: ["oncall@company.com"]
  subject: "[{{ event()['event.category'] }}] {{ event()['event.name'] }}"
  contentType: "text/html"
  body: |
    <html>
    <body style="font-family: Arial, sans-serif;">
      <h2 style="color: {{ '#dc3545' if (event().get('event.severity') | int(5)) <= 1 else '#ffc107' }};">
        {{ event()['event.category'] }} Alert
      </h2>
      <table style="border-collapse: collapse;">
        <tr><td><b>Problem:</b></td><td>{{ event()['event.name'] }}</td></tr>
        <tr><td><b>Status:</b></td><td>{{ event()['event.status'] }}</td></tr>
        <tr><td><b>Started:</b></td><td>{{ event()['event.start'] }}</td></tr>
      </table>
      <p><a href="{{ problem_link() }}">View in Dynatrace</a></p>
    </body>
    </html>
```

<a id="message-formatting"></a>
## 6. Message Formatting
### Jinja Expression Reference

| Expression | Result | Use |
|------------|--------|------|
| `{{ event()["event.name"] }}` | Problem title | Main content |
| `{{ event()["event.category"] }}` | AVAILABILITY/ERROR/SLOWDOWN/… | What kind of problem |
| `{{ event().get("event.severity") \| int(5) }}` | 1 (most severe) … 5 | Color coding |
| `{{ event()["affected_entity_ids"] \| join(", ") }}` | Comma-separated list | Show all affected |
| `{{ event()["event.start"] }}` | ISO timestamp | When started |
| `{{ problem_link() }}` | Full URL | Link to problem (Problem trigger only) |

`event.severity` is `experimental` in the semantic dictionary and is a 1–5 scale; the problem record carries no `CRITICAL`/`HIGH` strings. `int(5)` treats a missing severity as least severe — see WFLOW-04 §3 for why that default matters from SaaS 1.348.

### Conditional Formatting

```jinja
{% set sev = event().get("event.severity") | int(5) %}
{% if sev <= 1 %}
:red_circle: CRITICAL ALERT
{% elif sev == 2 %}
:large_orange_circle: HIGH ALERT
{% else %}
:large_yellow_circle: {{ event()["event.category"] }} ALERT
{% endif %}
```

### Severity Emoji Map

| `event.severity` | Slack Emoji | Teams Color |
|----------|-------------|-------------|
| 1 (Critical) | `:red_circle:` | `Attention` |
| 2 (High) | `:large_orange_circle:` | `Warning` |
| 3 (Medium) | `:large_yellow_circle:` | `Accent` |
| 4 (Low) | `:large_blue_circle:` | `Good` |

### Inline Severity Mapping

```jinja
{{ {1: ":red_circle:", 2: ":large_orange_circle:", 3: ":large_yellow_circle:", 4: ":large_blue_circle:"}.get(event().get("event.severity") | int(5), ":white_circle:") }}
```

<a id="complete-alert-workflow-example"></a>
## 7. Complete Alert Workflow Example
A production-ready workflow that sends alerts to Slack, Teams, and email.

### Workflow Configuration

```yaml
name: production-alert-notifications
description: Send alerts to all channels for production problems

trigger:
  type: davis-problem
  config:
    entityTagsMatch: all
    entityTags:
      - key: env
        value: prod

tasks:
  - name: slack_notification
    type: dynatrace.slack:message
    input:
      connection: slack-production
      channel: "#alerts-production"
      message: |
        {{ {1: ":red_circle:", 2: ":large_orange_circle:", 3: ":large_yellow_circle:"}.get(event().get("event.severity") | int(5), ":white_circle:") }} *{{ event()["event.category"] }} Problem*
        
        *{{ event()["event.name"] }}*
        
        • *Status:* {{ event()["event.status"] }}
        • *Started:* {{ event()["event.start"] }}
        • *ID:* {{ event()["display_id"] }}
        
        <{{ problem_link() }}|:mag: View Problem>

  - name: teams_notification
    type: dynatrace.msteams:message
    input:
      connection: teams-production
      message: |
        **{{ event()["event.category"] }} Problem**
        
        **{{ event()["event.name"] }}**
        
        - **Status:** {{ event()["event.status"] }}
        - **Started:** {{ event()["event.start"] }}
        - **ID:** {{ event()["display_id"] }}
        
        [View Problem]({{ problem_link() }})

  - name: email_notification
    type: dynatrace.email:send-email
    input:
      to: ["platform-oncall@company.com"]
      subject: "[{{ event()['event.category'] }}] {{ event()['event.name'] }}"
      body: |
        A {{ event()["event.category"] }} problem has been detected.
        
        Problem: {{ event()["event.name"] }}
        Status: {{ event()["event.status"] }}
        Started: {{ event()["event.start"] }}
        
        View: {{ problem_link() }}
```

### Task Execution

By default, tasks execute **in parallel**. All three notifications send simultaneously.

### Monitor Your Notification Workflows

```dql
// Notification workflow executions
// Data object corrected 08/12/2026. Workflow executions are NOT in `events`, and there is no
// `automation.task.execution` / `automation.workflow.execution` event type in any spelling — those
// filters matched nothing, silently, in 25 cells. AutomationEngine writes to `dt.system.events` with
// `event.kind == "WORKFLOW_EVENT"` (5.7M records / 30d here), split by `event.type`:
//   WORKFLOW_EXECUTION · TASK_EXECUTION · ACTION_EXECUTION · WORKFLOW_CREATED/UPDATED/DELETED
// Fields are the `dt.automation_engine.*` family:
//   .workflow.title / .workflow.id      (was workflow.name)
//   .task.name                          (was task.name)
//   .action.function / .action.app      (was task.type — e.g. run-javascript, send-email,
//                                        snow-search-incidents)
//   .state                              (was task.status / execution.status)
//                                        values RUNNING · SUCCESS · ERROR · DISCARDED · SKIPPED
//                                        — note "FAILED" is NOT a value; it is ERROR
//   .state.is_final                     true only on terminal records — filter on it, otherwise a
//                                        single execution is counted once as RUNNING and again as
//                                        SUCCESS/ERROR
//   .state_info                         (was task.error)  ·  duration (was task.duration)
// Enumerate with:
//   fetch dt.system.events, from:-24h | filter event.kind == "WORKFLOW_EVENT" | limit 1
fetch dt.system.events, from:-24h
| filter event.kind == "WORKFLOW_EVENT"
| filter event.type == "ACTION_EXECUTION"
| filter dt.automation_engine.state.is_final == true
| filter in(dt.automation_engine.action.app, {"dynatrace.email", "dynatrace.slack", "dynatrace.servicenow"})
| summarize executions = count(), by:{dt.automation_engine.action.app, dt.automation_engine.state}
| sort executions desc
```

```dql
// Notification task success rates
// Data object corrected 08/12/2026. Workflow executions are NOT in `events`, and there is no
// `automation.task.execution` / `automation.workflow.execution` event type in any spelling — those
// filters matched nothing, silently, in 25 cells. AutomationEngine writes to `dt.system.events` with
// `event.kind == "WORKFLOW_EVENT"` (5.7M records / 30d here), split by `event.type`:
//   WORKFLOW_EXECUTION · TASK_EXECUTION · ACTION_EXECUTION · WORKFLOW_CREATED/UPDATED/DELETED
// Fields are the `dt.automation_engine.*` family:
//   .workflow.title / .workflow.id      (was workflow.name)
//   .task.name                          (was task.name)
//   .action.function / .action.app      (was task.type — e.g. run-javascript, send-email,
//                                        snow-search-incidents)
//   .state                              (was task.status / execution.status)
//                                        values RUNNING · SUCCESS · ERROR · DISCARDED · SKIPPED
//                                        — note "FAILED" is NOT a value; it is ERROR
//   .state.is_final                     true only on terminal records — filter on it, otherwise a
//                                        single execution is counted once as RUNNING and again as
//                                        SUCCESS/ERROR
//   .state_info                         (was task.error)  ·  duration (was task.duration)
// Enumerate with:
//   fetch dt.system.events, from:-24h | filter event.kind == "WORKFLOW_EVENT" | limit 1
fetch dt.system.events, from:-7d
| filter event.kind == "WORKFLOW_EVENT"
| filter event.type == "ACTION_EXECUTION"
| filter dt.automation_engine.state.is_final == true
| summarize {total = count(), succeeded = countIf(dt.automation_engine.state == "SUCCESS")}, by:{dt.automation_engine.action.function}
| fieldsAdd success_pct = round(succeeded * 100.0 / total, decimals: 1)
| sort total desc
| limit 20
```

```dql
// Failed notification tasks
// Data object corrected 08/12/2026. Workflow executions are NOT in `events`, and there is no
// `automation.task.execution` / `automation.workflow.execution` event type in any spelling — those
// filters matched nothing, silently, in 25 cells. AutomationEngine writes to `dt.system.events` with
// `event.kind == "WORKFLOW_EVENT"` (5.7M records / 30d here), split by `event.type`:
//   WORKFLOW_EXECUTION · TASK_EXECUTION · ACTION_EXECUTION · WORKFLOW_CREATED/UPDATED/DELETED
// Fields are the `dt.automation_engine.*` family:
//   .workflow.title / .workflow.id      (was workflow.name)
//   .task.name                          (was task.name)
//   .action.function / .action.app      (was task.type — e.g. run-javascript, send-email,
//                                        snow-search-incidents)
//   .state                              (was task.status / execution.status)
//                                        values RUNNING · SUCCESS · ERROR · DISCARDED · SKIPPED
//                                        — note "FAILED" is NOT a value; it is ERROR
//   .state.is_final                     true only on terminal records — filter on it, otherwise a
//                                        single execution is counted once as RUNNING and again as
//                                        SUCCESS/ERROR
//   .state_info                         (was task.error)  ·  duration (was task.duration)
// Enumerate with:
//   fetch dt.system.events, from:-24h | filter event.kind == "WORKFLOW_EVENT" | limit 1
fetch dt.system.events, from:-24h
| filter event.kind == "WORKFLOW_EVENT"
| filter event.type == "ACTION_EXECUTION"
| filter dt.automation_engine.state == "ERROR"
| fields timestamp, dt.automation_engine.action.function, dt.automation_engine.task.name, dt.automation_engine.state_info
| sort timestamp desc
| limit 25
```

## Next Steps

Now that you can send basic notifications, learn advanced patterns:

### Recommended Path

1. **WFLOW-04: Advanced Notification Routing** - Conditional routing by severity, team, time
2. **WFLOW-05: PagerDuty & ServiceNow** - Incident management integration
3. **WFLOW-06: Custom Notification Templates** - Rich formatting and dynamic content

### Key Takeaways

- **Connections** store credentials securely and are reusable
- **Slack** supports both OAuth apps and webhooks
- **Teams** requires Power Automate or Teams Workflows connector (O365 Incoming Webhooks are deprecated)
- **Email** can use built-in SMTP or custom servers
- **Jinja expressions** enable dynamic message content
- Tasks execute in **parallel** by default

---

## Summary

In this notebook, you learned:

- Notification architecture: triggers → tasks → connections → external services
- How to set up connections for Slack, Teams, and email
- Basic and advanced Slack message formatting
- Microsoft Teams notifications with Adaptive Cards
- Plain text and HTML email notifications
- Jinja expressions for dynamic content
- A complete multi-channel alert workflow

---

## References

- [Notification actions umbrella (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/workflows/default-workflow-actions/actions)
- [Slack action (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/workflows/default-workflow-actions/actions/slack)
- [Microsoft Teams action (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/workflows/default-workflow-actions/actions/microsoft-teams)
- [Email action (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/workflows/default-workflow-actions/actions/email)
- [PagerDuty action (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/workflows/default-workflow-actions/actions/pagerduty)
- [Workflow reference / Jinja expressions (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/workflows/reference)
- [Alerting and notifications umbrella (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/alerting-and-notifications)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
