# ALERT - Alerting Strategy and Design

End-to-end alerting strategy and design for Dynatrace — detection, routing, reliability targets, and ITSM integration. This series orchestrates the AIOPS, SLO, and WFLOW series into one operational pattern rather than re-documenting each mechanism.

> **Recommended:** Import the JSON files from NOTEBOOKS/ into a Dynatrace tenant for the best experience. These notebooks contain interactive DQL queries that execute against your environment's data.

## Structure
- NOTEBOOKS/ — Dynatrace notebook JSON files
- PDFs/ — Printable versions of each notebook
- markdown/ — Markdown exports of the notebooks

## Notebook Lineup
1. [End-to-End Alerting Architecture](markdown/-[ALERT]-01-end-to-end-architecture.md) — The whole alerting board on one canvas: what to set up, where, and how detection, routing, and reliability connect end to end
2. [Choosing and Building Detection](markdown/-[ALERT]-02-choosing-and-building-detection.md) — Decision framework for the four detection mechanisms — which to use for which signal, and the anti-patterns that cause noise
3. [Routing, Destinations, and Cost](markdown/-[ALERT]-03-routing-destinations-cost.md) — Simple vs multi-step workflow routing, the destination landscape, the legacy alerting-profile path, and cost discipline
4. [ITSM Integration: ServiceNow](markdown/-[ALERT]-04-servicenow-integration.md) — Integration maturity ladder from one-way incident creation to ServiceNow-side event management, with the worked ServiceNow Table API path
99. [Best-Practice Summary and Setup Checklist](markdown/-[ALERT]-99-best-practice-summary.md) — The series on one page: principles, a complete end-to-end setup checklist, and the cross-series build map

## Usage
1. Choose a format: import JSON from NOTEBOOKS/, read PDFs/ for print, or view markdown/ for lightweight browsing.
2. Start with the End-to-End Alerting Architecture for orientation, then follow the numbered sequence or jump to the piece most relevant to your role.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
