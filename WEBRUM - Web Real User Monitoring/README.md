# WEBRUM - Web Real User Monitoring

Client-side monitoring and observability for web applications with session replay and performance metrics.

> **Recommended:** Import the JSON files from notebooks/ into a Dynatrace tenant for the best experience. These notebooks contain interactive DQL queries that execute against your environment's data.

## Structure
- notebooks/ — Dynatrace notebook JSON files
- pdfs/ — Printable versions of each notebook
- markdown/ — Markdown exports of the notebooks

## Notebook Lineup
1. [Web RUM Fundamentals](markdown/-[WEBRUM]-01-rum-fundamentals.md) — RUM architecture, JavaScript agent, and data collection model
2. [SPA Instrumentation](markdown/-[WEBRUM]-02-spa-instrumentation.md) — Monitoring single-page applications and JavaScript frameworks
3. [Core Web Vitals](markdown/-[WEBRUM]-03-core-web-vitals.md) — LCP, FID, CLS metrics, thresholds, and optimization
4. [Session Analysis](markdown/-[WEBRUM]-04-session-analysis.md) — User session insights, funnels, and user journey analysis
5. [Error Analysis](markdown/-[WEBRUM]-05-error-analysis.md) — JavaScript errors, crash detection, and error categorization
6. [Performance Analysis](markdown/-[WEBRUM]-06-performance-analysis.md) — Page load analysis, runtime performance, and bottleneck identification
7. [Session Replay](markdown/-[WEBRUM]-07-session-replay.md) — Visual session recording, privacy masking, and replay playback
8. [Dashboards and Alerting](markdown/-[WEBRUM]-08-dashboards-and-alerting.md) — RUM metrics dashboards and performance-based alerting
9. [Migrating USQL to DQL](markdown/-[WEBRUM]-09-usql-to-dql-migration.md) — USQL → DQL grammar and field mapping, with the classic-RUM vs New RUM decision that determines which field names apply
99. [Best Practice Summary](markdown/-[WEBRUM]-99-best-practice-summary.md) — Consolidated best practices from the WEBRUM series

## Usage
1. Choose a format: import JSON from notebooks/, read pdfs/ for print, or view markdown/ for lightweight browsing.
2. Start with Web RUM Fundamentals to understand how client-side monitoring works.
3. Instrument your web applications and explore the data model with DQL queries.
4. Create dashboards for end-user experience monitoring and set up alerts for performance degradation.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
