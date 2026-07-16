# AGENTS.md — ADOPT: Observability Adoption & Maturity

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

7 notebooks on advancing Dynatrace adoption across an organization: a
five-level maturity model, platform health assessment, success metrics
(MTTD/MTTR), team enablement, an optimization roadmap, a coverage-audit and
staged-enablement program, and a consolidated best-practice checklist.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| The five maturity levels (Reactive → Autonomous), assessing your current level, mapping features to levels | `-[ADOPT]-01-maturity-model.md` |
| OneAgent deployment coverage, ingestion rates, ActiveGate health, license consumption, building a health scorecard | `-[ADOPT]-02-platform-health-assessment.md` |
| MTTD/MTTR baselines, problem-count trends, alert noise ratio, change failure rate, proving value to leadership | `-[ADOPT]-03-success-metrics.md` |
| Role-based learning paths, the "hero problem", skill assessment, internal champions, training roadmaps | `-[ADOPT]-04-team-enablement.md` |
| Quick wins vs strategic investments, monthly milestones, cost-optimization candidates, version support policy / EOL tiers | `-[ADOPT]-05-optimization-roadmap.md` |
| Coverage-audit DQL (`monitoringMode`), estate segmentation, enablement Waves 1–4, value gates before each wave | `-[ADOPT]-06-maximizing-value-staged-enablement.md` |
| Consolidated checklist of every ADOPT best practice with priorities and source references | `-[ADOPT]-99-best-practice-summary.md` |

If more than three rows match, start with `-[ADOPT]-99-best-practice-summary.md`
and follow its pointers.

## Related series

- What each coverage gap costs (the matrix ADOPT-06 operationalizes): `../FAQ - Frequently Asked Questions/`
- DPS consumption tracking and cost optimization levers: `../FINOPS - Cost Management & FinOps/`
- Initial tenant setup before the maturity program: `../ONBRD - Dynatrace Onboarding/`
- Stakeholder dashboards for reporting adoption metrics: `../DASH - Dashboard Design & Building/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "ADOPT-06") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
