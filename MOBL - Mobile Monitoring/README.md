# MOBL - Mobile Monitoring

Dynatrace mobile Real User Monitoring (RUM) — SDK setup, user action tracking, crash reporting, session replay, and DQL analytics for iOS, Android, and cross-platform frameworks.

> **Recommended:** Import the JSON files from notebooks/ into a Dynatrace tenant for the best experience. These notebooks contain interactive DQL queries that execute against your environment's data.

## Structure
- notebooks/ — Dynatrace notebook JSON files
- pdfs/ — Printable versions of each notebook
- markdown/ — Markdown exports of the notebooks

## Notebook Lineup
1. [Mobile Monitoring Fundamentals](markdown/-[MOBL]-01-fundamentals.md) — Mobile RUM architecture, supported platforms, and beacon data flow
2. [iOS SDK Setup](markdown/-[MOBL]-02-sdk-setup-ios.md) — Installing and configuring the Dynatrace SDK for Swift and SwiftUI
3. [Android SDK Setup](markdown/-[MOBL]-03-sdk-setup-android.md) — Setting up Dynatrace RUM for Android with Gradle
4. [Cross-Platform Frameworks](markdown/-[MOBL]-04-cross-platform-frameworks.md) — Instrumenting Flutter, React Native, Cordova, and .NET MAUI (Xamarin end of support)
5. [User Action Tracking](markdown/-[MOBL]-05-user-action-tracking.md) — Capturing auto-detected and custom user interactions
6. [Crash Reporting & ANR Detection](markdown/-[MOBL]-06-crash-reporting.md) — Automatic crash capture, symbolication, and grouping
7. [Network Request Monitoring](markdown/-[MOBL]-07-network-request-monitoring.md) — HTTP(S) request visibility and timing breakdown
8. [Session Replay for Mobile](markdown/-[MOBL]-08-session-replay.md) — Visual session recording with privacy masking
9. [Session Properties & Data Privacy](markdown/-[MOBL]-09-session-properties-and-privacy.md) — Custom session enrichment and GDPR/CCPA compliance
10. [DQL for Mobile Analytics](markdown/-[MOBL]-10-dql-for-mobile.md) — Query reference for mobile RUM in user.events and user.sessions
11. [Dashboards & Alerting](markdown/-[MOBL]-11-dashboards-and-alerting.md) — KPI dashboards, crash-rate detectors, and problem workflows
12. [Advanced Instrumentation](markdown/-[MOBL]-12-advanced-instrumentation.md) — Custom events, nested actions, A/B testing, and multi-app strategies
99. [Best Practice Summary](markdown/-[MOBL]-99-best-practice-summary.md) — Consolidated best practices from the MOBL series

## Usage
1. Open a notebook from the lineup below. Each one is also available as a Dynatrace notebook (JSON) to import into your tenant, so its queries run against your own data, and as a PDF for print.
2. Start with Mobile Monitoring Fundamentals, then progress through the series.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
