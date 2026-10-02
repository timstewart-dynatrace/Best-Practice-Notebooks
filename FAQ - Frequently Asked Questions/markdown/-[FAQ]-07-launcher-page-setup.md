# FAQ-07: How Do I Set Up a Launcher Page? (Default + Persona-Based)

> **Series:** FAQ — Frequently Asked Questions | **Reference:** 07 — Launcher Page Setup (Default + Persona-Based) | **Created:** May 2026 | **Last Updated:** 10/02/2026

## Overview

"How do I set up a launcher page?" is one of the most common questions after a tenant is stood up and users start logging in. Under the question sit three different concerns: what users see when they first land in the platform (the *default*), what different teams or roles see (the *persona-based* experience), and how that experience is governed (the *IAM binding*).

The product name for the landing experience is **Launchpads** — built and managed inside the Launcher app. A Launchpad is a customizable home page that combines links to apps, markdown content, and cards (with optional images and destination buttons) into a single first-screen experience. Admins can assign Launchpads as the *Home Launchpad* for "Everyone" (the tenant default) or for specific IAM groups (the persona entries) — all in one rank-ordered list, and individual users can set their own personal Launchpad as a per-user override.

This FAQ unpacks the model: the three layers (tenant default, group entries, user personal — with admin entries resolved by rank), the admin mechanism, persona-content recommendations for seven common roles, and the parts of the model that are first-class features versus parts that are community discipline.

> **Scope:** Dynatrace SaaS. Home launchpads — admin-set start pages for teams and user groups — arrived in SaaS 1.314 (May 2025) and have been broadly available since. The admin mechanism and IAM-group binding are first-class features; persona-specific content recommendations are community guidance.

---

## Table of Contents

1. [Short Answer](#short-answer)
2. [What a Launchpad Actually Is](#what-launchpad-is)
3. [The Three-Layer Model — Default, Group, Personal](#three-layer-model)
4. [Setting the Home Launchpad — Admin Mechanism](#admin-mechanism)
5. [Persona Worked Examples](#persona-examples)
6. [The IAM Binding and Document Service Backing](#iam-binding)
7. [Recommended Approach](#recommended-approach)
8. [What's Still Evolving](#evolving)
9. [Common Objections and Responses](#objections)

---

## Prerequisites

| Requirement | Details |
|-------------|---------|
| **Audience** | Tenant admins, platform leads, IAM owners; secondary: persona/role leads (SRE, Security, AppDev, etc.) |
| **Format** | Decision-support document — describes the model, admin path, and persona patterns; no hands-on lab |
| **Deployment** | Dynatrace SaaS. Home launchpads arrived in SaaS 1.314 (May 2025); broadly available since. |
| **Related topic series** | IAM (group management and policy DSL), DASH (dashboard design that feeds Launchpad link cards), ORGNZ (persona/security-context naming that often parallels IAM group naming), ONBRD (where Launchpad setup fits in the rollout sequence) |
| **Related FAQ** | FAQ-01 (host-group naming), FAQ-02 (tagging strategy) — both touch on persona axes that often align with Launchpad personas |

<a id="short-answer"></a>
## 1. Short Answer

"Setting up a launcher page" in Dynatrace means setting up a **Launchpad** — a customizable home page in the Launcher app. The four questions customers ask most, in compressed form:

| Question | Short answer | Where to look |
|----------|-------------|---------------|
| What is a "launcher page"? | A **Launchpad** — a customizable home page made of links, markdown, and cards, built in the Launcher app. | §2 |
| How do I set what *everyone* sees? | Assign a Home Launchpad to "Everyone" via **Settings → General → Launcher → Home launchpad**. | §3, §4 |
| How do I set different home pages per team? | Assign Home Launchpads to specific IAM user groups via the same Settings path. Group binding is a first-class feature. | §4, §6 |
| How do users get their own? | A user's personal Home Launchpad overrides the admin's; the admin one drops to "Suggested." | §3 |

**Headline:** the mechanism is first-class — admins bind Launchpads to IAM groups (or "Everyone") and users can override personally. The harder problem is **content discipline** — deciding what to put on each persona's Launchpad. That part is community practice, not a feature.

> <sub>**Sources:** [Launchpads (DT docs)](https://docs.dynatrace.com/docs/discover-dynatrace/get-started/dynatrace-ui/launchpads), [Dynatrace launchpads — customizable home pages (Dynatrace News)](https://www.dynatrace.com/news/blog/dynatrace-launchpads-focus-on-what-matters-with-customizable-home-pages/).</sub>

<a id="what-launchpad-is"></a>
## 2. What a Launchpad Actually Is

A Launchpad is a **customizable start page** built inside the Launcher app. It is a navigation surface, not a visualization surface — it is where users land to *get somewhere*, not where users go to *see a chart*.

### Building blocks

A Launchpad is composed of three content types:

| Building block | What it is | Typical use |
|----------------|------------|-------------|
| **Links** | A pointer to a Dynatrace app, a document (dashboard, notebook), an internal URL, or an external URL | "Open the Problems app", "Jump to the team SLO dashboard", "Go to the on-call runbook in Confluence" |
| **Markdown** | Formatted text with inline links and images | "This week's priorities", "Where to file tickets", "Contact list for the platform team" |
| **Cards** | A brief explanation + optional image + destination button | High-value visual entry points — "Start a CoPilot conversation", "Open the executive KPI dashboard" |

### What a Launchpad is *not*

In community practice, this is a common source of confusion in the first few days of adoption — the three building blocks above are the whole content model, so everything else a Launchpad is sometimes mistaken for falls outside it:

- **Not a dashboard.** A Launchpad has no chart-rendering capability. Dashboards live in the Dashboards app; a Launchpad references a dashboard by **deep-linking** to it.
- **Not a notebook.** Same pattern — a notebook is a Documents-app object; a Launchpad deep-links to it.
- **Not the Apps-menu favorites.** The left-nav favorites are a per-user navigation preference. A Launchpad is the *page* a user lands on. The two coexist — a user might favorite the same apps they have linked from their Launchpad.
- **Not a portal in the heavyweight sense.** No custom JavaScript widgets, no embedded charts, no React components — the building blocks are the three above.

### Persistence and ownership

Launchpads persist via the Dynatrace **Document Service** with `type='launchpad'` — the same storage layer that holds dashboards and notebooks. This is load-bearing for two downstream concerns: (1) sharing and access scoping work the same way as for dashboards and notebooks, and (2) a launchpad moves between environments as JSON — **Download** and **Upload** in Launcher — and is reachable programmatically through the Document Service API.

> <sub>**Sources:** [Launchpads (DT docs)](https://docs.dynatrace.com/docs/discover-dynatrace/get-started/dynatrace-ui/launchpads) — defines the three building blocks (Links, Markdown, Cards) and documents persistence via the Document Service API (`type='launchpad'`); *"Download saves the current launchpad as a local JSON file."* [Dynatrace launchpads — customizable home pages (Dynatrace News)](https://www.dynatrace.com/news/blog/dynatrace-launchpads-focus-on-what-matters-with-customizable-home-pages/) — verbatim on capabilities: *"consolidating relevant resources to a single page"*, *"providing shortcuts to Dynatrace® Apps—including deep links to dashboards, notebooks"*.</sub>

<a id="three-layer-model"></a>
## 3. The Three-Layer Model — Default, Group, Personal

![Launchpad Precedence Model — Three Layers](images/07-launcher-three-layer-model_930x500.png)

<!-- MARKDOWN_TABLE_ALTERNATIVE
| Layer | Scope | Set by | Precedence |
|-------|-------|--------|------------|
| User personal | One user | The user themselves | Highest — overrides everything below |
| Group Home Launchpad | All users in an IAM group | Admin via Settings | Admin entries (groups and Everyone) resolve by rank — rank group entries above Everyone |
| Everyone Home Launchpad | All users in the tenant | Admin via Settings | The floor only when ranked below the group entries |
For environments where SVG doesn't render
-->

### Layer 1 — "Everyone" Home Launchpad (tenant default)

This is meant to be the floor — every user who is not in a group with its own Home Launchpad and who has not set a personal Home Launchpad lands here. It acts as the floor only when it is ranked **below** the group entries (Layer 2). There is one "Everyone" Home Launchpad per tenant — it is the broadest landing experience.

Practical implication: ship this *first*. A solid "Everyone" Launchpad sets a baseline experience for new users, contractors, occasional users, and anyone outside the persona groups you've built specific Launchpads for. Don't skip it; don't ship persona Launchpads without it.

### Layer 2 — Group Home Launchpad (persona override)

Admins can assign a Home Launchpad to a specific IAM user group. Users in that group land on the group's Launchpad instead of the "Everyone" one **provided the group entry is ranked above the Everyone entry**. Everyone is set in the same list as the group entries, and the documented priority is by rank with nothing that exempts Everyone — so if Everyone sits above a group entry, Everyone wins for that group's members. Confirm the order in a non-production tenant. Multiple groups can each have their own Home Launchpad — this is the mechanism for "different teams see different home pages."

The assignment binds a Launchpad to a group, and the admin-set Home launchpads are **ranked**. When a user belongs to several groups that each have a Home Launchpad, the one with the **highest rank** wins; the others are next in line. So the order of the entries in *Settings → General → Launcher → Home launchpad* is a governance decision — put the launchpad that should win for overlapping members (for example, Security over SRE) above the other, and keep the Everyone entry last.

### Layer 3 — Personal Home Launchpad (user override)

Any user can set their own personal Home Launchpad. When they do, the admin's group-bound Launchpad (or "Everyone" Launchpad) drops to the "Suggested" list rather than being displayed as the home page. This is by design — users with strong preferences customize, but the admin defaults remain discoverable.

### Resolution order

The documented Home launchpad priority is:

1. The user's **personal** Home launchpad.
2. The admin-set Home launchpad with the **highest rank**.
3. Other admin-set Home launchpads with lower ranks, and those of other groups the user is a member of.
4. **Getting started with Dynatrace**, the default Home launchpad.

**A launchpad that fails to load is skipped silently** — *"If a home launchpad fails to load (for example, due to failing permissions), the next in line opens."* A group-bound launchpad that the group's members cannot read therefore never errors: they simply land on the next launchpad in line, which is why § 4 makes sharing the launchpad with the group a step of its own. The admin-set layers fall to the "Suggested" list when a user sets a personal Home launchpad.

> <sub>**Sources:** [Launchpads (DT docs)](https://docs.dynatrace.com/docs/discover-dynatrace/get-started/dynatrace-ui/launchpads) — verbatim on per-user override: users can set personal home launchpads that *"override any home launchpad set by your admin"* and admin-set launchpads then appear under *"Suggested"*. Group binding documented as *"select a user group, or select 'Everyone' to change the start page for all users."* Priority documented under *Home launchpad priority*: *"Home launchpad set by your admin with highest rank"*, then *"Other home launchpads set by your admin with lower ranks and other groups you're member of"*, then the default Getting started launchpad; *"If a home launchpad fails to load (for example, due to failing permissions), the next in line opens."* **Derived:** the three-layer framing, and the rule to rank Everyone last, are this entry's reading of that priority list — the page does not say explicitly how Everyone ranks against group entries.</sub>

<a id="admin-mechanism"></a>
## 4. Setting the Home Launchpad — Admin Mechanism

The admin path is short and the same for tenant-default and group-bound assignments.

### Path

**Settings → General → Launcher → Home launchpad → Add home launchpad**

### Steps

1. **Build the Launchpad first** in the Launcher app. The assignment step references an existing Launchpad — you cannot create a Launchpad as a side effect of assignment.
2. **Share the Launchpad with the target group** (or with everyone, for the tenant default). Launchpads are documents with document sharing; a Home launchpad the group's members cannot open fails to load, and they silently land on the next launchpad in line (§ 3).
3. Navigate to **Settings → General → Launcher → Home launchpad**.
4. Select **Add home launchpad**.
5. Pick a Launchpad from the list of existing Launchpads.
6. Select either **Everyone** (tenant default) or a specific IAM **user group**.
7. Save, and check the entry's **rank** against the other group entries (§ 3).

### For per-group bindings

Repeat steps 2 and 4–7 for each group that should have its own Home Launchpad. There is no "clone this Launchpad to a new group" shortcut — each binding is an explicit admin action.

### Sequencing implication

In community practice, this is the trap most new tenants hit: the admin tries to bind a Launchpad before there is content to bind. **Build the content scaffold first** — at minimum the "Everyone" Launchpad, ideally a thin draft for each high-value persona — then do the bindings as a batch.

### What changes when you save

The binding takes effect immediately for new sessions. Users with an active session may need to refresh or re-navigate to the home page to see the change. Users who already have a personal Home Launchpad set will continue to see their personal one — the admin's binding lands in their "Suggested" list.

> <sub>**Sources:** [Launchpads (DT docs)](https://docs.dynatrace.com/docs/discover-dynatrace/get-started/dynatrace-ui/launchpads) — documents the admin path *"Settings → General → Launcher → Home launchpad"*, the **Add home launchpad** action, and the *Everyone / user group* selection. **Derived:** the "share with the group" step follows from launchpads being shared documents (*"For details on sharing Dynatrace documents (including launchpads), see Share documents"*) and from the fail-to-load rule quoted in § 3.</sub>

<a id="persona-examples"></a>
## 5. Persona Worked Examples

The mechanism (Layer 2 — group-bound Home Launchpads) is the easy part. The harder part is **content discipline**: what should actually appear on each persona's Launchpad? This section gives recommended content for seven common roles.

Treat this as a starting point, not a final spec. The persona content discipline is community practice — every tenant's actual personas, naming, and content priorities will differ. The shape below is a reasonable opening draft.

### Persona content matrix

| Persona | Apps to link | Documents to deep-link | Markdown sections | First-screen signal (card) |
|---------|--------------|------------------------|---------------------|----------------------------|
| **SRE / Site Reliability** | Problems, Distributed Traces, Logs | "Open P1s" notebook, on-call runbook | "This week's incidents", incident-channel link | Open Davis problems count |
| **Platform / Infra** | Hosts, Kubernetes, ActiveGates | Cluster health dashboard, capacity-planning notebook | Maintenance windows schedule | Infrastructure problem rate |
| **AppDev / Service Owner** | Services, Logs, SLOs | Service SLO dashboard, deploy-status notebook | Service onboarding checklist, team docs | Latest deploy status / SLO compliance |
| **Security / IAM** | Vulnerabilities, Audit Log, IAM | Security findings dashboard, audit-trail notebook | Compliance contact list, escalation flow | Critical vulnerabilities count |
| **Executive / Business** | Dashboards | Executive KPI dashboard, business funnel notebook | Strategic priorities, monthly business focus | Top-3 business KPIs (card-deep-link to dashboard) |
| **Migration / Onboarding** | Documents, Launcher itself | Migration tracker dashboard, ONBRD runbook | "Where to find help", DT docs deep-links | Migration progress percentage |
| **AIOps / Davis-CoPilot heavy** | Davis CoPilot, Problems | AI Observability dashboards, model-monitoring notebook | Davis CoPilot prompt tips, "ask Davis first" guide | Davis CoPilot launch card |

### Why each shape works

- **SRE** — open Davis problems is the active-incident workflow; everything else cascades from "is there a fire right now?". The link to the incident channel and the on-call runbook keeps the operational context one click away.
- **Platform/Infra** — cluster health and capacity dominate the day-to-day. Maintenance windows on the markdown panel prevent the most common "why is X broken?" question.
- **AppDev** — "did my last deploy break something?" is the canonical first-screen question. SLO compliance and the most recent deploy status answer it directly.
- **Security** — vulnerability count and audit log surface the two highest-priority signals. The escalation flow on markdown reduces the "who do I tell?" friction during a security event.
- **Executive / Business** — strategic dashboards as cards (a brief explanation, an optional static image, and a button that deep-links to the dashboard) give the executive a single glance. Avoid raw Problems lists — executives don't act on individual problems.
- **Migration / Onboarding** — for users in their first 30 days, surface the runbook and the migration tracker. Don't assume they know where anything else is. This persona's Launchpad will be retired or auto-rotated as they graduate.
- **AIOps / Davis-CoPilot heavy** — for teams using Davis CoPilot as a primary workflow, surface it first. The Davis CoPilot launch card is a single click into the conversation interface.

### Anti-patterns

- **One Launchpad to rule them all.** Building a single 40-link Launchpad and binding it to "Everyone" is worse than no customization — users get a wall of links with no signal. If you can't pick the top 8–12 items, the persona is not differentiated enough yet.
- **Persona Launchpads with no content owner.** A Launchpad with no team owner goes stale within a quarter. Assign each persona's Launchpad to a named maintainer.
- **Confusing the Launchpad with a dashboard.** Don't try to show data on the Launchpad. Cards can *deep-link* to a dashboard, and that's the right pattern — but the Launchpad itself doesn't render charts.

> <sub>**Sources:** [Launchpads (DT docs)](https://docs.dynatrace.com/docs/discover-dynatrace/get-started/dynatrace-ui/launchpads), [Dynatrace launchpads — customizable home pages (Dynatrace News)](https://www.dynatrace.com/news/blog/dynatrace-launchpads-focus-on-what-matters-with-customizable-home-pages/) — confirms team-level customization scenarios (onboarding, developer-team daily routine).</sub>

<a id="iam-binding"></a>
## 6. The IAM Binding and Document Service Backing

### How the group binding works

The Home Launchpad assignment in **Settings → General → Launcher → Home launchpad** is a direct binding between a Launchpad object and either:

- The literal value **"Everyone"** (every user in the tenant), or
- A specific **IAM user group** (every user in that group).

The IAM user group is the same primitive used elsewhere in the platform for permission management. If you have a `team-sre` group that already governs the team's IAM policies, you bind the SRE Launchpad to that same group — there is no separate "Launchpad audience" abstraction.

### Persistence — Document Service

Launchpads are stored in the Dynatrace **Document Service** with `type='launchpad'`. The Document Service is the same storage layer that holds dashboards (`type='dashboard'`) and notebooks (`type='notebook'`). Three downstream implications:

| Implication | Detail |
|-------------|--------|
| **Sharing model** | Launchpad sharing inherits from the Document Service sharing model. Owners can share with users directly or with groups — the same primitives as dashboard/notebook sharing. |
| **Download / Upload** | In Launcher, **Download** saves a launchpad as a JSON file and **Upload** in the destination environment imports it — the documented way to move a launchpad between environments. Version-control automation goes through the Document Service API. |
| **API surface** | The Document Service API supports CRUD on Launchpads at `https://<environment>/platform/document/v1/documents?filter=type='launchpad'`. |

### IAM policy scope — what is and isn't documented

What is documented:

- The Home Launchpad assignment mechanism in Settings (group / Everyone binding).
- Per-user override behavior (personal Home Launchpad overrides admin's).
- Document Service persistence with `type='launchpad'`.

What is **not** explicitly documented (IAM policy reference re-read 10/02/2026):

- A Launchpad-specific IAM service / verb catalog. There is no published `launchpad:*` policy statement family analogous to `davis-copilot:*` or `document:documents:*`.
- The exact IAM scopes required to (a) create a Launchpad, (b) share a Launchpad, (c) set a Home Launchpad in Settings.

Practical guidance, from community practice rather than documentation: treat Launchpad access scoping as a **Document Service concern** — the same `document:documents:*` policy statements that govern dashboard and notebook access are the most likely controls in effect. For tightly-governed tenants, test the actual behavior in a non-production tenant before relying on a specific scoping model.

### Why this matters

For most customers, the implicit Document Service IAM model is sufficient — Launchpads don't carry sensitive data themselves, only links to other places that have their own access controls. For customers with strict IAM governance, the lack of a Launchpad-specific policy DSL is a real gap and should be flagged in the platform's IAM model documentation.

*The absence of a published `launchpad:*` policy-statement family is a current-state observation rather than a documented guarantee — the IAM catalog evolves, and a future SaaS release may add explicit Launchpad scopes.*

> <sub>**Sources:** [Launchpads (DT docs)](https://docs.dynatrace.com/docs/discover-dynatrace/get-started/dynatrace-ui/launchpads) — documents Document Service persistence with `type='launchpad'`, the API surface, and moving launchpads between environments: *"Select > Download to download the launchpad. Go to the destination environment, select Launchpads > Upload"*. [IAM policy reference (DT docs)](https://docs.dynatrace.com/docs/manage/identity-access-management/permission-management/manage-user-permissions-policies/advanced/iam-policystatements) — does not enumerate a Launchpad-specific service (re-read 10/02/2026); verify when authoring policies.</sub>

<a id="recommended-approach"></a>
## 7. Recommended Approach

An adoption order that produces good results in community practice:

1. **Stand up the tenant default first ("Everyone" Home Launchpad).** Build one Launchpad that works for the broadest audience — a platform-overview shape with general app links, doc directory entries, and "where to file tickets" markdown. This is the floor every user falls back to.
2. **Pick 2–3 high-value personas before going wider.** Don't try to ship 7 persona Launchpads on day one. SRE + AppDev is a typical starting pair; add Security or Executive if either has a clear sponsor and stated need.
3. **Build the Launchpad content scaffold before creating the binding.** The Settings binding step references an existing Launchpad — you need the content first. Build all the persona Launchpads, get them reviewed by the persona's team lead, *then* batch the bindings.
4. **Bind to existing IAM groups; don't create new groups just for Launchpads.** The IAM groups you already have for permission management are usually the right axes for Launchpad binding too. Creating Launchpad-specific groups (e.g., `launchpad-sre-audience`) duplicates the persona-naming problem and confuses the IAM model.
5. **Iterate quarterly with the persona's team.** Launchpad content goes stale — links to deprecated dashboards, markdown referencing org structures that have changed, runbooks that have moved. Build a quarterly review into the persona team's cadence with a named owner for each Launchpad.
6. **Measure adoption — prune what isn't earning its space.** A Launchpad with no clicks isn't doing its job. If a persona Launchpad isn't being used, find out why (wrong content? wrong audience? users overriding personally?) and either fix it or retire it.
7. **Treat per-user override as a feature, not a problem.** Power users *should* customize. The admin defaults exist to make the experience good for everyone else. Don't try to prevent personal overrides.

### Anti-patterns to avoid

- **Building Launchpads before IAM groups exist.** Without groups, you can only target "Everyone." That is fine for a starting tenant but doesn't unlock the persona-based value.
- **Treating the Launchpad as a dashboard substitute.** Cards can deep-link to dashboards; cards cannot render data themselves. If you want a chart on the home screen, the right answer is a dashboard, not a Launchpad.
- **Skipping the "Everyone" Launchpad.** Building only persona Launchpads means new users, contractors, and anyone outside your persona groups land on the platform default — *Getting started with Dynatrace* — with no team context.
- **Owner-less Launchpads.** Every persona Launchpad needs a named maintainer. Otherwise it goes stale and you end up with the worst of both worlds: the binding exists, but the content is wrong.

> <sub>**Sources:** [Launchpads (DT docs)](https://docs.dynatrace.com/docs/discover-dynatrace/get-started/dynatrace-ui/launchpads), [Dynatrace launchpads — customizable home pages (Dynatrace News)](https://www.dynatrace.com/news/blog/dynatrace-launchpads-focus-on-what-matters-with-customizable-home-pages/).</sub>

<a id="evolving"></a>
## 8. What's Still Evolving

Naming the moving parts explicitly is more credible than pretending everything is settled. The Launchpad **mechanism** is solid; the **content and governance discipline** are where the work happens, and several pieces are still maturing:

- **Launchpad IAM scope catalog is implicit.** Launchpads inherit Document Service scoping (`document:documents:*` is the likely control), but there is no published Launchpad-specific policy DSL (e.g., `launchpad:*`). For most customers this is fine; for tightly-governed tenants, it means the access-control story is implicit rather than explicit.
- **Persona content discipline is community practice.** Dynatrace ships the mechanism; what goes on the SRE Launchpad versus the Exec Launchpad versus the Migration Launchpad is engagement / community framing, not a built-in feature. Treat the persona matrix in §5 as a starting point, not a spec.
- **Launchpad templates / starter kits.** There is no persona-specific template. The starting points that do exist are the **Ready-made** launchpads, **Duplicate** (copy a launchpad and edit the copy), and download/upload of launchpad JSON — persona content itself is still yours to write.
- **Content type is link-first.** Dashboards and notebooks appear on Launchpads as *links*, not as embedded widgets. If you want a "page with a chart on it," that is a dashboard. Many new customers conflate the two and are briefly surprised; the link-first model is by design.
- **Versioning and source control.** Launchpads can be downloaded and uploaded as JSON in Launcher, and read or written through the Document Service API — those are the version-control primitives. Native Git-backed versioning is not first-class today; the community pattern is to export Launchpad JSON periodically and check it into source control.
- **Cross-tenant content sharing.** Launchpads are tenant-scoped. There is no built-in "share this Launchpad with my partner's tenant" feature; the workaround is **Download** from one tenant and **Upload** in the other.

These are *moving parts*, not red flags. The Home launchpad mechanism has been stable since SaaS 1.314 (May 2025) and is broadly available. The right posture is to ship the parts that are solid (mechanism, group binding, per-user override) and treat the parts that are evolving (IAM scope catalog, content templates) as items to verify at the time you depend on them.

> <sub>**Sources:** Community-derived observations across customer engagements; no single Dynatrace doc enumerates these gaps. [Launchpads (DT docs)](https://docs.dynatrace.com/docs/discover-dynatrace/get-started/dynatrace-ui/launchpads) for the mechanism baseline — *"Duplicate makes a copy of the current launchpad so you can build a new launchpad based on the current launchpad."* **Derived + Softened** throughout — these are current-state observations as of May 2026; expect some items to be addressed in future SaaS releases.</sub>

<a id="objections"></a>
## 9. Common Objections and Responses

Short objections, short responses. Where the response depends on customer-specific verification, that is flagged.

**"Why not just use a dashboard as the home page?"**

You can — a Launchpad card can deep-link to a dashboard, and many "executive" Launchpads are essentially a markdown intro plus a dashboard deep-link card. The trade-off: dashboards are powerful but rigid (specific chart types, time-range controls); Launchpads are flexible (links + markdown + cards) but don't visualize data. Use Launchpads as the *navigation hub* and dashboards as the *visualization surface*. The two complement each other.

**"Users will customize their own anyway, so why bother with admin defaults?"**

Two reasons. First, most users don't customize — the default is what they see for years. Second, a good admin default sets a frame; users who customize are often refining that frame, not rejecting it. The admin default is the floor; user customization is the ceiling.

**"We don't have IAM groups by persona yet — can we still do this?"**

Yes. Start with the "Everyone" Home Launchpad and ship one solid generic Launchpad first. The group-binding step waits until you have IAM groups (and building IAM groups is its own decision — see the IAM topic series). Doing the tenant-default tier first is valuable on its own.

**"How is a Launchpad different from the Apps-menu favorites?"**

The Apps-menu favorites are *per-user* preferences for the left-nav. A Launchpad is the *page* the user lands on. Favorites are about navigation shortcuts; Launchpads are about a first-screen experience. They coexist — a user might favorite the same apps they have linked from their Launchpad. The two are not in tension.

**"What if Dynatrace changes Launchpads in the future?"**

Home launchpads have been broadly available since SaaS 1.314 (May 2025) and the mechanism is stable. Major changes typically arrive via release notes; minor UI evolution is normal. The bigger risk is *content drift in the Launchpads themselves* — links to dashboards that no longer exist, markdown that references deprecated docs — and that is operational practice, not platform risk.

**"Can I version-control Launchpads?"**

Launchpads persist in the Document Service; Launcher's **Download** / **Upload** moves them as JSON, and the Document Service API reads and writes them. The community pattern is to save Launchpad JSON to source control periodically. Native Git-backed versioning is not first-class today, but the download/upload primitive gives you a workable approximation.

**"Can I prevent users from customizing their personal Home Launchpad?"**

No — and you probably shouldn't want to. The per-user override is a deliberate design choice and broadly results in better user experience. Power users should customize; the admin defaults exist to make the experience good for everyone else.

**"How do dashboards and notebooks show up on a Launchpad?"**

As links — specifically, deep-links into the Dashboards app or the Documents app pointing at a specific dashboard or notebook. The Launchpad doesn't embed the content; clicking the link opens the dashboard or notebook in its own app. Use this for high-value documents that the persona returns to repeatedly.

> <sub>**Sources:** All claims map back to the Sources blocks in §§2–8 above. **Softened** throughout — these are summary responses; the load-bearing verifications happen in the cited sections.</sub>

## Summary

Setting up "launcher pages" in Dynatrace means setting up **Launchpads** — customizable home pages composed of links, markdown, and cards. The admin mechanism (`Settings → General → Launcher → Home launchpad`) is first-class and supports both tenant default ("Everyone") and IAM-group binding ("persona-based"). The three-layer model is **tenant default, group entries, user personal**: the personal launchpad wins when set, and admin entries — Everyone included — resolve by rank, so keep the group entries above Everyone. Persona-specific content discipline (what links and cards belong on each persona's Launchpad) is community practice — Dynatrace ships the mechanism, the customer team supplies the content.

**Recommended approach:** stand up the "Everyone" Launchpad first, identify 2–3 high-value personas (typically SRE + AppDev to start), build the Launchpad content scaffold *before* creating the bindings, bind to existing IAM groups, iterate quarterly with the persona's team, and measure adoption to prune what isn't earning its space.

## Next Steps

- Read the **IAM** topic series for IAM group management and the Document Service policy DSL that implicitly governs Launchpad access.
- Read the **DASH** topic series for the dashboard design patterns that feed Launchpad link cards (dashboards are the visualization surface; Launchpads are the navigation hub).
- Read the **ORGNZ** topic series for persona / security-context naming — often the same axes that work for Launchpad bindings.
- Read the **ONBRD** topic series for where Launchpad setup fits in the rollout sequence (typically once tenants are stood up and IAM groups exist).
- Share each group-bound Launchpad with its group, and set the rank order of Home launchpad entries deliberately — overlapping group members get the highest-ranked one, and Everyone belongs last.
- Build the tenant "Everyone" Home Launchpad before going wider — resist the temptation to ship seven persona Launchpads on day one.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official [Dynatrace documentation](https://docs.dynatrace.com/docs).*</sub>
