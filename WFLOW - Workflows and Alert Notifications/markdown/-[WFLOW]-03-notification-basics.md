# WFLOW-03: Alert Notification Basics

> **Series:** WFLOW — Workflows and Alert Notifications | **Notebook:** 3 of 10 | **Created:** January 2026 | **Last Updated:** 10/06/2026

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
| **Permissions** | `automation:workflows:write`, `automation:workflows:run` (WFLOW-01 §4) |
| **Workflows authorization settings** | `app-settings:objects:read` and `app-settings:objects:write` for connector actions; `email:emails:send` for email. Each connector's setup page lists its full set (Workflows > Settings > Authorization settings) |
| **External requests** | A host pattern for every outbound destination (Settings > General > External requests) |
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
| Slack | `dynatrace.slack:slack-send-message` (Send message); Request approval (copy its ID from the action picker) | Bot OAuth token from a Slack app |
| Microsoft Teams | `dynatrace.msteams:send-message` | Teams Workflows (Power Automate) webhook URL |
| Email | `dynatrace.email:send-email` | Built-in — sends from `no-reply@apps.dynatrace.com`; no SMTP setup |
| PagerDuty | Send event (Events API v2) and REST API actions — copy the exact IDs from the action picker | Events connection (routing key) for Send event; REST API key for the others |
| ServiceNow | `dynatrace.servicenow:snow-create-incident`, plus other `snow-*` actions | OAuth or Basic Auth |
| Jira | `dynatrace.jira:jira-create-issue`, plus other `jira-*` actions | API Token |
| Custom Webhook (HTTP Request) | `dynatrace.automations:http-function` | Credential Vault (Basic or Token) |

The Slack Send message, Teams, Email, ServiceNow, Jira and HTTP Request identifiers were checked against Dynatrace's published workflow samples and connector templates (10/02/2026, re-checked 10/06/2026). The PagerDuty and Slack Request approval identifiers do not appear there, so copy them from the action picker rather than typing them. Once an action has run, the platform records its app and function in `dt.system.events` (`dt.automation_engine.action.app` / `.action.function`), so a tenant can confirm them too. The YAML in this series uses these identifiers and the actions' real input names (`message`, `content`, `payload`, `connectionId`). It is simplified in two places: connection names stand in for connection IDs, and named `conditions:` lists stand in for each task's `conditions` (`states` / `custom`). Export a workflow built in the editor to get the exact document shape.

> <sub>**Sources:** [threat-detection-notification-sender.yaml (Dynatrace GitHub)](https://raw.githubusercontent.com/Dynatrace/Dynatrace-workflow-samples/main/samples/security/threat%20detection/threat-detection-notification-sender.yaml) — *"action: dynatrace.slack:slack-send-message"*, *"action: dynatrace.msteams:send-message"*; [wftpl_sample_servicenow_incident_man.yaml (Dynatrace GitHub)](https://raw.githubusercontent.com/Dynatrace/Dynatrace-workflow-samples/main/samples/Messaging%20and%20Incident%20Management/wftpl_sample_servicenow_incident_man.yaml) — *"action: dynatrace.servicenow:snow-create-incident"*; [Workflow samples action catalog (Dynatrace GitHub)](https://raw.githubusercontent.com/Dynatrace/Dynatrace-workflow-samples/main/AGENTS.md); [PagerDuty Connector actions (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/workflows/default-workflow-actions/actions/pagerduty/pagerduty-workflows-actions) — *"Trigger, acknowledge, or resolve an alert in PagerDuty using the Events API v2"*; [Email (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/workflows/default-workflow-actions/actions/email).</sub>

<a id="setting-up-connections"></a>
## 2. Setting Up Connections
Connections store credentials separately from workflows for security and reusability.

### Accessing Connections

1. Open **Settings**
2. Select **Connections** → **Connectors**, then the tool (Slack, Microsoft Teams, ServiceNow, …)
3. Select **Connection** to add one

Dynatrace's connector pages word this path slightly differently (*Connections > Slack* on the Slack page, *Connections > Connectors > Microsoft Teams* on the Teams page); both lead to the same place.

### Creating a Connection

1. Open the connector (step 2 above) and select **Connection**
2. Enter the credential the connector asks for (a Slack bot token, a Teams webhook URL, …)
3. Allow the destination host under **Settings > General > External requests**
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
### Setting Up the Slack Connection

The Slack connection takes a bot token from a Slack app you create: *"Your Dynatrace Slack Connector requires an OAuth token to authorize sending messages to Slack."*

**Setup:**
1. **Allow the host.** Settings > General > External requests: add a host pattern for `slack.com`.
2. **Grant Workflows the connector permissions.** Workflows > Settings > Authorization settings: enable `app-settings:objects:read`, `app-settings:objects:write` and the `state:app-states:*` and `state:user-app-states:*` permissions the setup page lists.
3. **Create the Slack app** at https://api.slack.com/apps, from an app manifest. To send messages to channels, the minimal bot scopes are `channels:read`, `groups:read` and `chat:write`. Add `chat:write.public` to post to public channels the bot has not joined. Approvals, reactions and file attachments need more scopes, listed on the setup page.
4. **Install** the app to the workspace and copy the **Bot User OAuth Token**.
5. **Create the connection.** Settings > Connections > Slack > **Connection**: name it and paste the token in **Bot token**.

> **There is no webhook option on the Slack connection.** If you must post through a Slack incoming webhook, that is an HTTP Request task (`dynatrace.automations:http-function`) to the webhook URL, outside the Slack connector, with the webhook host allowed under External requests.

> <sub>**Sources:** [Set up Slack Connector (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/workflows/default-workflow-actions/actions/slack/automation-workflows-slack-setup) — the External requests and Authorization settings steps and the minimal manifest; [Slack Connector actions (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/workflows/default-workflow-actions/actions/slack/automation-workflows-slack-actions) — scopes per action.</sub>

### Basic Slack Message Task

```yaml
name: send_slack_alert
action: dynatrace.slack:slack-send-message
input:
  connection: slack-production      # the Slack connection's ID in an exported workflow
  channel: "#alerts-production"
  messageFormat: slack_format
  message: |
    :rotating_light: *Problem Detected*
    
    *Title:* {{ event()["event.name"] }}
    *Category:* {{ event()["event.category"] }}
    *Started:* {{ event()["event.start"] }}
    
    <{{ problem_link() }}|View in Dynatrace>
```

### Slack Message with Blocks

For richer formatting, put a Block Kit payload in `message`. The Send message action has no separate `blocks` input: with `messageFormat: slack_format`, *"Slack Markdown or Slack Block Kit inputs are processed"* from the message field.

```yaml
input:
  connection: slack-production
  channel: "#alerts"
  messageFormat: slack_format
  message: |
    {
      "blocks": [
        { "type": "header",
          "text": { "type": "plain_text", "text": "{{ ':red_circle:' if (event().get('event.severity') | int(5)) <= 1 else ':large_orange_circle:' }} {{ event()['event.category'] }} Alert" } },
        { "type": "section",
          "fields": [
            { "type": "mrkdwn", "text": "*Problem:*\n{{ event()['event.name'] }}" },
            { "type": "mrkdwn", "text": "*Status:*\n{{ event()['event.status'] }}" } ] },
        { "type": "section",
          "fields": [
            { "type": "mrkdwn", "text": "*Started:*\n{{ event()['event.start'] }}" },
            { "type": "mrkdwn", "text": "*ID:*\n{{ event()['display_id'] }}" } ] },
        { "type": "actions",
          "elements": [
            { "type": "button", "text": { "type": "plain_text", "text": "View Problem" }, "url": "{{ problem_link() }}" } ] }
      ]
    }
```

The expressions resolve before the JSON reaches Slack, so a problem title that contains a double quote breaks the payload. Build the layout in Slack's Block Kit Builder and send one test message before you rely on it.

> <sub>**Sources:** [Slack Connector actions (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/workflows/default-workflow-actions/actions/slack/automation-workflows-slack-actions) — *"If messageFormat is slack_format , Slack Markdown or Slack Block Kit inputs are processed."*; [Workflow samples action catalog (Dynatrace GitHub)](https://raw.githubusercontent.com/Dynatrace/Dynatrace-workflow-samples/main/AGENTS.md) — *"Slack messages can be plain text/Slack Markdown, or a JSON Block Kit payload"*.</sub>

<a id="microsoft-teams-notifications"></a>
## 4. Microsoft Teams Notifications

> **Office 365 connectors no longer work.** Microsoft's final schedule: *"Rollout begins : May 18, 2026 Rollout completes : May 22, 2026 After these dates, Office 365 Connectors will no longer function."* An Office 365 incoming-webhook URL in a Dynatrace connection stops delivering. The Dynatrace Microsoft Teams connector takes a Teams **Workflows** (Power Automate) webhook URL instead.

### Setting Up the Teams Connection

1. **Create the webhook in Teams.** Open the channel's context menu, select **Workflows**, and choose **Send webhook alerts to a channel**. Save, then select **Copy webhook link**.
2. **Allow the host.** Settings > General > External requests: add a host pattern for the webhook's domain. Dynatrace's example is `*.api.powerplatform.com`.
3. **Create the connection.** Settings > Connections > Connectors > Microsoft Teams > **Connection**: name it and paste the URL in **Webhook URL**.

Messages arrive under the name of the user who created the webhook. If an existing connection's URL contains `logic.azure.com`, the Teams connector page explains how to replace it with the flow's updated URL.


### Basic Teams Message Task

```yaml
name: send_teams_alert
action: dynatrace.msteams:send-message
input:
  connectionId: teams-production    # the Teams connection's ID in an exported workflow
  messageFormat: dynatrace_markdown # converts each Markdown element to a card element
  message: |
    **Problem Detected**
    
    **Title:** {{ event()["event.name"] }}
    **Category:** {{ event()["event.category"] }}
    **Started:** {{ event()["event.start"] }}
    
    [View in Dynatrace]({{ problem_link() }})
```

### Teams Adaptive Card

For richer formatting, put Adaptive Card JSON in `message` with the default `messageFormat` (`msteams_format`), or pick a predefined card in `selectTemplate`.

```yaml
input:
  connectionId: teams-production
  message: |
    {
      "type": "AdaptiveCard",
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      "version": "1.4",
      "body": [
        { "type": "TextBlock", "text": "{{ event()['event.category'] }} Alert", "size": "Large", "weight": "Bolder",
          "color": "{{ 'Attention' if (event().get('event.severity') | int(5)) <= 1 else 'Warning' }}" },
        { "type": "FactSet", "facts": [
            { "title": "Problem", "value": "{{ event()['event.name'] }}" },
            { "title": "Status", "value": "{{ event()['event.status'] }}" },
            { "title": "Started", "value": "{{ event()['event.start'] }}" } ] }
      ],
      "actions": [
        { "type": "Action.OpenUrl", "title": "View in Dynatrace", "url": "{{ problem_link() }}" }
      ]
    }
```

Use Dynatrace expressions for the dynamic values, as above. The docs say *"We don't support Adaptive Cards Template Language templating"*.

> <sub>**Sources:** [Microsoft Teams Connector (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/workflows/default-workflow-actions/actions/microsoft-teams) — setup steps, `messageFormat` (*"Must be msteams_format or dynatrace_markdown"*) and `selectTemplate`; [Retirement of Office 365 connectors within Microsoft Teams (Microsoft 365 Developer Blog)](https://devblogs.microsoft.com/microsoft365dev/retirement-of-office-365-connectors-within-microsoft-teams/); [threat-detection-notification-sender.yaml (Dynatrace GitHub)](https://raw.githubusercontent.com/Dynatrace/Dynatrace-workflow-samples/main/samples/security/threat%20detection/threat-detection-notification-sender.yaml) — *"action: dynatrace.msteams:send-message"*; [Workflow samples action catalog (Dynatrace GitHub)](https://raw.githubusercontent.com/Dynatrace/Dynatrace-workflow-samples/main/AGENTS.md).</sub>

<a id="email-notifications"></a>
## 5. Email Notifications
### Email Options

| Option | Setup | Best For |
|--------|-------|----------|
| **Send email action** (`dynatrace.email:send-email`) | No setup — sends from `no-reply@apps.dynatrace.com`; the workflow needs the `email:emails:send` permission | Most notifications |
| **Corporate mail relay** | Not an option of the Send email action — call the relay's HTTP API from an HTTP Request or JavaScript task | A corporate sender address is required |
| **SendGrid/SES** | HTTP Request task to the provider's API | High volume, HTML templates |

The two HTTP routes need the provider's host allowed under **Settings > General > External requests**. A host on a private network, such as a corporate relay, is reached through EdgeConnect (WFLOW-94).

The Send email action has two limits: *"For To , Cc , and Bcc fields the number of email addresses is restricted to 10 per field."* and *"Trial environments are prohibited to send emails with Email ."*

### Basic Email Task

```yaml
name: send_email_alert
action: dynatrace.email:send-email
input:
  to:
    - "oncall@company.com"
    - "platform-team@company.com"
  cc: []
  bcc: []
  subject: "[{{ event()['event.category'] }}] {{ event()['event.name'] }}"
  content: |
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
    {{ (event().get("root_cause.smartscape_entity") or {}).get("name", "not yet determined") }}
    
    View this problem:
    {{ problem_link() }}
```

### Formatted Email

The root cause is read with `.get()` because most problems do not carry one: on a validation tenant, 127 of 540 problems over 24 h had `root_cause.smartscape_entity` (10/06/2026), and a bracket read of a missing field fails the task with *Undefined variables*. The older `root_cause_entity_id` is deprecated and just as often absent. Enabling **Wait for root cause analysis** on the trigger (WFLOW-02 §2) addresses the early-lifecycle gap but does not guarantee a root cause: *"Dynatrace Intelligence does not populate a root cause for every problem, particularly early in the lifecycle or for externally ingested events."*

The `content` field takes markdown: bold, headings, lists, tables and links. It does not take HTML, so an HTML template arrives as literal tags, and there is no content-type input. For an HTML template, send through SendGrid or SES from an HTTP Request task (table above).

```yaml
input:
  to: ["oncall@company.com"]
  cc: []
  bcc: []
  subject: "[{{ event()['event.category'] }}] {{ event()['event.name'] }}"
  content: |
    ## {{ event()['event.category'] }} alert

    | | |
    |---|---|
    | **Problem** | {{ event()['event.name'] }} |
    | **Status** | {{ event()['event.status'] }} |
    | **Started** | {{ event()['event.start'] }} |

    [View in Dynatrace]({{ problem_link() }})
```

> <sub>**Sources:** [Email (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/workflows/default-workflow-actions/actions/email) — *"It doesn't offer support for HTML."*; [threat-detection-notification-sender.yaml (Dynatrace GitHub)](https://raw.githubusercontent.com/Dynatrace/Dynatrace-workflow-samples/main/samples/security/threat%20detection/threat-detection-notification-sender.yaml) — *"action: dynatrace.email:send-email"*.</sub>

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
:large_orange_circle: MAJOR ALERT
{% else %}
:large_yellow_circle: {{ event()["event.category"] }} ALERT
{% endif %}
```

### Severity Emoji Map

| `event.severity` | Slack Emoji | Teams Color |
|----------|-------------|-------------|
| 1 (Critical) | `:red_circle:` | `Attention` |
| 2 (Major) | `:large_orange_circle:` | `Warning` |
| 3 (Minor) | `:large_yellow_circle:` | `Accent` |
| 4 (Warning) | `:large_blue_circle:` | `Good` |

Level names follow the Davis severity scale (see WFLOW-04 § 3); Warning and Informational events never open a problem, so a problem-triggered workflow sees levels 1–3.

### Inline Severity Mapping

```jinja
{{ {1: ":red_circle:", 2: ":large_orange_circle:", 3: ":large_yellow_circle:", 4: ":large_blue_circle:"}.get(event().get("event.severity") | int(5), ":white_circle:") }}
```

<a id="complete-alert-workflow-example"></a>
## 7. Complete Alert Workflow Example
A workflow that sends the same problem to Slack, Teams, and email. Tasks are keyed by name and the trigger block uses the exported shape (WFLOW-02 §2); export a workflow built in the editor for the exact document.

### Workflow Configuration

```yaml
name: production-alert-notifications
description: Send alerts to all channels for production problems

trigger:
  eventTrigger:
    isActive: true
    triggerConfiguration:
      type: davis-problem
      value:
        triggerOn: open
        categories:
          availability: true
          error: true
          slowdown: true
          resource: true
        entityTagsMatch: all
        entityTags:
          env:
            - prod
        analysisReady: true          # Wait for root cause analysis

tasks:
  slack_notification:
    name: slack_notification
    action: dynatrace.slack:slack-send-message
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

  teams_notification:
    name: teams_notification
    action: dynatrace.msteams:send-message
    input:
      connectionId: teams-production
      messageFormat: dynatrace_markdown
      message: |
        **{{ event()["event.category"] }} Problem**
        
        **{{ event()["event.name"] }}**
        
        - **Status:** {{ event()["event.status"] }}
        - **Started:** {{ event()["event.start"] }}
        - **ID:** {{ event()["display_id"] }}
        
        [View Problem]({{ problem_link() }})

  email_notification:
    name: email_notification
    action: dynatrace.email:send-email
    input:
      to: ["platform-oncall@company.com"]
      cc: []
      bcc: []
      subject: "[{{ event()['event.category'] }}] {{ event()['event.name'] }}"
      content: |
        A {{ event()["event.category"] }} problem has been detected.
        
        Problem: {{ event()["event.name"] }}
        Status: {{ event()["event.status"] }}
        Started: {{ event()["event.start"] }}
        
        View: {{ problem_link() }}
```

### Task Execution

All three tasks have no predecessor, so they start in parallel when the trigger fires. A task that does have predecessors waits for them: *"By default, a task will run if its predecessors ended successfully."*

### Cost: this is a standard workflow

Three tasks make this a **standard** workflow, which consumes workflow hours for as long as it exists (WFLOW-01 §3.5). For a single destination, a simple workflow with one task costs no workflow hours. The upgrade guide keeps standard workflows for cases like this one, *"when one filter must fan out to several destinations in a single configuration."*

> <sub>**Sources:** [Monitor workflow executions (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/workflows/running); [Upgrade guide — alerting and notifications (DT docs)](https://docs.dynatrace.com/docs/platform/upgrade/keep-problems-and-alerting-working/upgrade-guide-alert-notification); [Automation Workflow consumption (DT docs)](https://docs.dynatrace.com/docs/license/capabilities/automation/automation).</sub>

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
- **Slack** connections take a bot OAuth token from a Slack app; there is no webhook option
- **Teams** connections take a Teams Workflows (Power Automate) webhook URL; Office 365 connectors stopped working in May 2026
- **Email** sends from `no-reply@apps.dynatrace.com`; for another sender, call a mail provider's API from an HTTP Request task
- Every outbound destination needs an **External requests** host pattern
- **Jinja expressions** enable dynamic message content; read optional fields such as the root cause with `.get()`
- Tasks with no predecessor run in **parallel**; more than one task makes a **standard** workflow, billed in workflow hours

---

## Summary

In this notebook, you learned:

- Notification architecture: triggers → tasks → connections → external services
- How to set up connections for Slack, Teams, and email
- Basic and advanced Slack message formatting
- Microsoft Teams notifications with Adaptive Cards
- Plain-text and markdown email notifications (the Send email action does not render HTML)
- Jinja expressions for dynamic content
- A complete multi-channel alert workflow

---

## References

- [Notification actions umbrella (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/workflows/default-workflow-actions/actions)
- [Slack action (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/workflows/default-workflow-actions/actions/slack)
- [Set up Slack Connector (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/workflows/default-workflow-actions/actions/slack/automation-workflows-slack-setup)
- [Retirement of Office 365 connectors within Microsoft Teams (Microsoft 365 Developer Blog)](https://devblogs.microsoft.com/microsoft365dev/retirement-of-office-365-connectors-within-microsoft-teams/)
- [Microsoft Teams action (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/workflows/default-workflow-actions/actions/microsoft-teams)
- [Email action (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/workflows/default-workflow-actions/actions/email)
- [PagerDuty action (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/workflows/default-workflow-actions/actions/pagerduty)
- [Workflow reference / Jinja expressions (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/workflows/reference)
- [Alerting and notifications umbrella (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/alerting-and-notifications)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
