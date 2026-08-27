# NRLC-05: Synthetic Monitor Migration

> **Series:** NRLC — New Relic to Dynatrace Migration Deep Dives | **Notebook:** 5 of 9 | **Created:** April 2026 | **Last Updated:** 08/27/2026

## Overview

New Relic offers four synthetic monitor types plus two specialized types: Ping, Simple Browser, API (multi-step), Step (Scripted Browser), Certificate Check, and Broken Links. Dynatrace offers HTTP, Browser, and Multi-step HTTP monitors. This deep dive maps the NR types to their DT equivalents, covers the location mapping (NR public/private → DT public/private), and flags the scripted-browser caveat that drives most of the manual migration effort.

**Phase 24 specialized synthetics (post-2026-04-15):** `synthetic_specialized_transformer` now auto-converts **Certificate Check** monitors → DT HTTP Monitor with `certificateExpiryDate` validation rule, and **Broken Links** monitors → DT Multi-step HTTP Monitor (capped at 50 links). Ping + Simple Browser + API continue to auto-convert via `synthetic_transformer` (Phase 11). Scripted Browser remains manual (R-01 in ENGINE-ENHANCEMENTS).

---

## Table of Contents

1. [Synthetic Type Mapping](#type-map)
2. [Ping → HTTP Monitor](#ping)
3. [Simple Browser → Browser Monitor](#browser)
4. [API (Multi-step) → Multi-step HTTP Monitor](#api)
5. [Step (Scripted Browser) → Browser Monitor](#step)
6. [Location Mapping](#locations)
7. [Authentication & Secrets](#auth)
8. [Validation Pattern](#validation)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Audience** | Engineers migrating synthetic monitors |
| **Standalone** | This notebook is self-contained for synthetic migration. No required prerequisite reading. |
| **Optional depth** | NRLC-08 (validation), NRLC-09 (toolchain) |
| **NR access** | Read scope on Synthetics |
| **DT access** | Synthetic create permission; private location IDs if used |

<a id="embedded-context"></a>
## Embedded Context — Location & Credential Resolution

Synthetic monitors don't contain NRQL queries, so there's no translation context. The standalone considerations are **location mapping** and **credential re-entry**.

### Public location lookup (run at migration time)

```bash
# Cache DT public location IDs into the transformer's lookup table.
# Match the Authorization scheme to your token prefix:
#   dt0s16.* / dt0s01.* (Platform Token) → Bearer
#   dt0c01.*             (Classic token) → Api-Token
curl -H "Authorization: Bearer $DT_TOKEN" \
  https://<tenant>.live.dynatrace.com/api/v2/synthetic/locations
```

Public location IDs are tenant-specific; the transformer queries on first run and caches. The Dynatrace-NewRelic `migrate.py` tool auto-selects the correct scheme based on the token prefix — the scheme only matters when you run raw `curl` as above.

### Credential migration checklist

Every synthetic monitor that uses authentication needs its secrets re-entered in DT — not migrated:

| Auth Type | NR Storage | DT Storage |
|-----------|-----------|------------|
| Bearer token | Inline in script | DT credentials vault |
| Basic auth | Inline | DT credentials vault |
| OAuth client secret | Inline | DT credentials vault |
| API key in header | Inline | DT credentials vault |

The transformer captures the *shape* of the auth (which header, which step) but emits placeholder values.

<a id="type-map"></a>
## 1. Synthetic Type Mapping

| NR Type | DT Type | Transformer | Manual Effort |
|---------|---------|-------------|---------------|
| Ping | HTTP Monitor | `synthetic_transformer` (Phase 11) | None |
| Simple Browser | Browser Monitor | `synthetic_transformer` (Phase 11) | None |
| API (Multi-step) | Multi-step HTTP Monitor | `synthetic_transformer` (Phase 11) | Low — verify chained variable extraction |
| **Certificate Check** | HTTP Monitor w/ `certificateExpiryDate` rule | `synthetic_specialized_transformer` (Phase 24) | None |
| **Broken Links** | Multi-step HTTP Monitor (50-link cap) | `synthetic_specialized_transformer` (Phase 24) | Low — verify link list discovery |
| Step (Scripted Browser) | Browser Monitor (clickpath) | Stub + manual | **High — 30–60 min per script** (R-01 backlog) |

The first five auto-convert. For Scripted Browser monitors, the engine emits a stub with a TODO and the original NR script attached as a tag for reference.

<a id="ping"></a>
## 2. Ping → HTTP Monitor

| NR Field | DT Field |
|----------|----------|
| `name` | `name` |
| `uri` | `requests[0].url` |
| `period` (1, 5, 10, 15, 30, 60 min) | `frequencyMin` |
| `locations[]` | `locations[]` (mapped per §6) |
| `verifyName`, `verifySSL` | `requests[0].validation.rules` |
| `bypassHEADRequest` | `requests[0].requestBody`/method config |

**Output:** a single-request HTTP monitor that runs from the mapped DT locations on the mapped frequency.

<a id="browser"></a>
## 3. Simple Browser → Browser Monitor

Same structure as Ping but `type: BROWSER`. The browser engine differences:

| Aspect | NR | DT |
|--------|----|----|
| Browser engine | Chromium-based | Chromium (private locations) / multi-engine (public) |
| Page load metrics | first-paint, DCL, LCP, etc. | LCP, FCP, TBT, CLS, page load (DT W3C-aligned) |
| RUM injection | optional | optional (separate from synthetic; DT RUM is its own module) |

Migration is structural; the metrics are roughly equivalent but not bit-identical (different timing methodologies). Document this for any dashboard that compared NR/DT browser timings during dual-run.

<a id="api"></a>
## 4. API (Multi-step) → Multi-step HTTP Monitor

NR API monitors are JS scripts with multiple `$http.get/post/...` calls and chained variable extraction. DT Multi-step HTTP monitors use a declarative request chain.

Conversion approach:

1. Parse the NR JS script statement-by-statement
2. Extract each `$http.*` call into a DT request step
3. Map response body extraction (`$http.response.body.json.field`) to DT's body extractor with JSONPath
4. Convert chained variables (e.g., `var token = response.body.token; ...next request uses token`) to DT request variables
5. Flag any non-trivial JS logic (loops, conditionals, custom helpers) for manual review

**Confidence:**

- HIGH for linear request chains with simple variable extraction
- MEDIUM for chains with conditional logic
- LOW for scripts with loops or external helper modules

<a id="step"></a>
## 5. Step (Scripted Browser) → Browser Monitor

**This is the migration's biggest manual cost.**

NR Scripted Browsers use a Selenium-style JS DSL with arbitrary code:

```javascript
$browser.get('https://example.com/login')
  .then(() => $browser.findElement($driver.By.name('username')).sendKeys('user'))
  .then(() => $browser.findElement($driver.By.name('password')).sendKeys('pass'))
  .then(() => $browser.findElement($driver.By.name('submit')).click())
  .then(() => $browser.waitForAndFindElement($driver.By.id('dashboard'), 5000));
```

DT Browser Monitors use a **clickpath** model: a sequence of typed actions (navigate, click, type, validate). Custom JS hooks exist (`pre/postRequestExecutionScript`) but the model is fundamentally different.

**Migration approach:**

1. Auto-emit a stub DT Browser Monitor with the target URL
2. Attach the original NR script as a tag/note
3. Engineer rebuilds the user journey using DT's clickpath recorder or DSL
4. Validation: confirm the new clickpath catches the same regressions during a 1-week dual-run

**Effort estimate:** ~30–60 min per scripted browser monitor, depending on complexity.

<a id="locations"></a>
## 6. Location Mapping

The transformer maintains a lookup table from NR location codes to DT location IDs:

| NR Public Location | DT Public Location | Notes |
|-------------------|-------------------|-------|
| `AWS_US_EAST_1` | `GEOLOCATION-9999000000000001` (lookup) | DT IDs are tenant-specific |
| `AWS_EU_WEST_1` | `GEOLOCATION-...` | |
| `AWS_AP_SOUTHEAST_2` | `GEOLOCATION-...` | |
| ... | ... | |

Public location IDs are queried via `GET /api/v2/synthetic/locations` at migration time — they're not portable across tenants. The transformer queries the DT tenant on first run and caches the mapping.

**Private locations:**

| NR Private Location | DT Private Location |
|--------------------|--------------------|
| Minion installation in your VPC | ActiveGate with synthetic capability |

Private locations require DT ActiveGate setup (see Dynatrace docs). The transformer maps NR private location names to DT private location names by string match; mismatches require manual lookup.

<a id="auth"></a>
## 7. Authentication & Secrets

**Credentials never auto-migrate.** This includes:

- Bearer tokens / API keys in headers
- Basic auth username + password
- Cookie values
- OAuth client secrets

The transformer captures the **shape** of the auth (which header, which step) but emits placeholders. Engineers re-enter secrets in the DT credentials vault and reference them from the migrated monitor.

Document every secret in the migration's secret-rotation runbook so nothing gets missed during cutover.

<a id="validation"></a>
## 8. Validation Pattern

After migration, validate synthetics with this pattern:

1. **Run-level:** confirm the new DT monitor executes successfully on its first scheduled run
2. **Frequency match:** verify DT runs at the same cadence as NR (no gaps, no double-runs)
3. **Location coverage:** confirm all mapped locations are reporting
4. **Failure detection:** intentionally break the target (in a non-prod env) and confirm DT detects the failure within the same window NR would have
5. **SLA continuity:** during dual-run, confirm DT's availability % matches NR's within ±0.5%

## Summary

Synthetic migration is mostly mechanical for Ping, Simple Browser, and API monitors. Scripted Browser monitors are the long pole — budget 30–60 min per script. Plan for credential re-entry as a coordinated event, not as part of the auto-import.

Continue to **NRLC-06 SLO & Workload Migration** for service-level migration.

<a id="tooling-synthetics"></a>
## Tooling for Synthetic-Only Migration

```bash
# 1. Inventory NR synthetic monitors (Ping, Browser, API, Step)
python3 migrate.py migrate --export-only --components synthetics --output ./synthetics-export

# 2. Transform (auto-converts Ping/Browser/API; stubs Step monitors)
python3 migrate.py migrate --transform-only --components synthetics --report

# 3. Diff against existing DT synthetics
python3 migrate.py migrate --diff --components synthetics

# 4. Import
python3 migrate.py migrate --import-only --components synthetics

# 5. Re-enter credentials in the DT credentials vault
# 6. Validate via DT UI: confirm first scheduled run executes successfully
```

**Scripted browser monitors** require manual rebuild as DT clickpaths — budget 30–60 min per script.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources, including the open-source [NewRelic-to-Dynatrace-Migration-Utilities](https://github.com/timstewart-dynatrace/NewRelic-to-Dynatrace-Migration-Utilities), [nrql-engine](https://github.com/timstewart-dynatrace/nrql-engine) (planned future home: the [`dynatrace-dma`](https://github.com/dynatrace-dma) Dynatrace Migration Assistant organization), and [nrql-translator](https://github.com/timstewart-dynatrace/nrql-translator) projects. This notebook series is not officially supported by Dynatrace or New Relic. Always verify information against the official [Dynatrace documentation](https://docs.dynatrace.com/docs) and [New Relic documentation](https://docs.newrelic.com).*</sub>
