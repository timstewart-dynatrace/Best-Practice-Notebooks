# AGENTS.md — WEBRUM: Web Real User Monitoring

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

9 notebooks on Dynatrace web RUM: the JavaScript agent and injection methods,
SPA instrumentation, Core Web Vitals, session/error/performance analysis,
session replay, and RUM dashboards and alerting — all queried via
`user.sessions` and `user.events` in Grail.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| How web RUM works: JS agent injection, RUM data model, `user.sessions`/`user.events`, `frontend.name`, bot filtering (`user.type == "REAL_USER"`), first queries | `-[WEBRUM]-01-rum-fundamentals.md` |
| Angular/React/Vue apps: injection methods, route change detection, XHR/fetch monitoring, custom action naming, validating SPA instrumentation, hybrid WebView apps | `-[WEBRUM]-02-spa-instrumentation.md` |
| Core Web Vitals: LCP, INP, CLS measurement, good/poor thresholds, CWV by page group, trends and scoring | `-[WEBRUM]-03-core-web-vitals.md` |
| User sessions: segmentation, journey mapping, conversion funnels, bounce rate, geographic and device/browser breakdowns | `-[WEBRUM]-04-session-analysis.md` |
| Browser-side errors: JavaScript exceptions, XHR/fetch errors (HTTP 4xx/5xx), `dtrum.reportError()`, error grouping, impact analysis, rage clicks | `-[WEBRUM]-05-error-analysis.md` |
| Slow pages: page load waterfall, TTFB, DOM interactive/load event timing, performance by geography/network/device, bottleneck identification | `-[WEBRUM]-06-performance-analysis.md` |
| Session replay for web: DOM mutation recording (not video), privacy masking, finding replay-eligible sessions, correlating replay with performance, third-party replay tools | `-[WEBRUM]-07-session-replay.md` |
| Dashboards and alerts: executive vs operational views, Apdex calculation in DQL, error-rate and performance-degradation alerts, combined RUM + synthetic view | `-[WEBRUM]-08-dashboards-and-alerting.md` |
| Consolidated checklist: injection method, bot filtering, CWV thresholds, replay/privacy settings, alerting values, DQL patterns | `-[WEBRUM]-99-best-practice-summary.md` |

If more than three rows match, start with `-[WEBRUM]-99-best-practice-summary.md`
and follow its pointers.

## Related series

- Mobile RUM counterpart — native/cross-platform SDKs, crashes, mobile replay: `../MOBL - Mobile Monitoring/`
- Proactive availability testing that complements RUM: `../SYNTH - Synthetic Monitoring/`
- Conversion funnels and business events beyond session analysis: `../BIZEV - Business Events & Funnel Analysis/`
- Dashboard design for the audiences these views serve: `../DASH - Dashboard Design & Building/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "WEBRUM-03") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
