# AGENTS.md — ORGNZ: Organize Data: Buckets, Segments, Security

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

11 notebooks on organizing data in Dynatrace Grail: bucket fundamentals and
strategy, the Grail permission model (bucket-, table-, record-, and
field-level), `dt.security_context`, segments and advanced segment
definitions, enterprise governance patterns, and a best-practice summary with
validation DQL.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| Why organize data at all: buckets vs tables vs views, the three pillars, which mechanism to use when | `-[ORGNZ]-01-introduction-to-organizing-data.md` |
| Bucket fundamentals: supported tables, bucket limits, default buckets, naming rules, querying bucket info | `-[ORGNZ]-02-understanding-grail-buckets.md` |
| Bucket naming conventions, retention planning, design patterns, cost attribution, routing data to buckets | `-[ORGNZ]-03-bucket-strategy-and-design.md` |
| The Grail permission model overview: permission levels, IAM policy structure, storage-policy conditions and operators | `-[ORGNZ]-04-permissions-in-grail.md` |
| IAM policies granting specific buckets, team-isolation via buckets, when bucket-level isolation fits (compliance, retention, cost) | `-[ORGNZ]-05-bucket-level-access-control.md` |
| Record-level access with `dt.security_context`: setting it, policy `WHERE` clauses, patterns, entity security context | `-[ORGNZ]-06-security-context.md` |
| Record- and field-level permissions, combining mechanisms, policy boundaries, testing permissions | `-[ORGNZ]-07-advanced-permission-patterns.md` |
| Segments vs buckets, segment structure/limits, design patterns, OneAgent enrichments, segments and access control | `-[ORGNZ]-08-grail-segments.md` |
| Enterprise patterns: LOB isolation, environment tiering, multi-cloud, Kubernetes-native organization, decision framework | `-[ORGNZ]-09-enterprise-patterns.md` |
| Segment mechanics deep dive: filter syntax, include rules, variables, host-group segments, Settings API/Terraform, Query API consumption, Davis problem includes | `-[ORGNZ]-10-advanced-segment-definitions.md` |
| Consolidated best practices + validation DQL: bucket design, retention, routing, IAM, security context, segments, governance | `-[ORGNZ]-99-best-practice-summary.md` |

If more than three rows match, start with `-[ORGNZ]-99-best-practice-summary.md`
and follow its pointers.

## Related series

- Policy authoring, boundaries, and group architecture in depth: `../IAM - IAM Administration/`
- Routing logs into buckets with OpenPipeline: `../OPLOGS - OpenPipeline Logs/`
- Retention and bucket cost trade-offs: `../FINOPS - Cost Management & FinOps/`
- Replacing Management Zones with segments + policies: `../MZ2POL - Management Zone to Policy Migration/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "ORGNZ-06") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
