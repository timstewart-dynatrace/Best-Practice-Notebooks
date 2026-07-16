# AGENTS.md — MOBL: Mobile Monitoring

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

13 notebooks on Dynatrace mobile Real User Monitoring: SDK setup for iOS,
Android, and cross-platform frameworks, user action tracking, crash reporting,
network request monitoring, session replay, privacy compliance, a mobile DQL
cookbook, and dashboards/alerting.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| Mobile RUM architecture, supported platforms, mobile entity types, beacon-to-Grail data flow, mobile vs web RUM | `-[MOBL]-01-fundamentals.md` |
| iOS setup: SPM/CocoaPods install, Info.plist config (`DTXApplicationID`, beacon URL), UIKit auto-instrumentation, SwiftUI `.dtAction()`, verifying data | `-[MOBL]-02-sdk-setup-ios.md` |
| Android setup: Gradle plugin, build.gradle config, Activities/Fragments auto-instrumentation, Jetpack Compose, ProGuard/R8 | `-[MOBL]-03-sdk-setup-android.md` |
| Flutter, React Native, Cordova, Xamarin, or .NET MAUI apps: plugin setup, configuration bridging, React Native symbolication | `-[MOBL]-04-cross-platform-frameworks.md` |
| Taps, swipes, app starts not showing or misnamed: auto-detected vs custom actions, action lifecycle, rage tap detection, action naming | `-[MOBL]-05-user-action-tracking.md` |
| Crashes and ANRs: dSYM upload/symbolication, ProGuard mapping, crash grouping, crash-rate DQL, manual error reporting | `-[MOBL]-06-crash-reporting.md` |
| HTTP(S) requests from the app: automatic capture, connection type (WiFi/5G/LTE), timing breakdown, frontend-to-backend trace correlation, slow/failed requests | `-[MOBL]-07-network-request-monitoring.md` |
| Mobile session replay: enabling it, privacy masking levels, sampling/cost control, crash replay, per-platform config | `-[MOBL]-08-session-replay.md` |
| Session enrichment and privacy: custom session properties, user tagging, data collection levels, opt-in mode, GDPR/CCPA, retention | `-[MOBL]-09-session-properties-and-privacy.md` |
| Mobile DQL cookbook: entities, user actions, crashes, performance metrics, device/OS segmentation, geolocation, reusable templates | `-[MOBL]-10-dql-for-mobile.md` |
| KPI dashboards and alerts: crash-free rate tiles, session volume trends, metric events, anomaly detection, problem correlation | `-[MOBL]-11-dashboards-and-alerting.md` |
| Advanced SDK use: `sendBizEvent()`, nested/manual actions, web request tagging, A/B testing and feature flags, SDK tuning, multi-app strategies | `-[MOBL]-12-advanced-instrumentation.md` |
| Consolidated checklist: SDK config values, symbolication, replay/privacy settings, alerting thresholds, DQL patterns | `-[MOBL]-99-best-practice-summary.md` |

If more than three rows match, start with `-[MOBL]-99-best-practice-summary.md`
and follow its pointers.

## Related series

- Web (browser) RUM counterpart — JS agent, Core Web Vitals, web session replay: `../WEBRUM - Web Real User Monitoring/`
- Business events sent from mobile apps, funnels: `../BIZEV - Business Events & Funnel Analysis/`
- Backend traces that mobile network requests correlate into: `../SPANS - Distributed Tracing and Spans/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "MOBL-06") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
