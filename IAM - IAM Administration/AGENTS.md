# AGENTS.md — IAM: IAM Administration

Per-series routing for AI agents. Repo-wide rules: [../AGENTS.md](../AGENTS.md).
Humans: see [README.md](README.md).

15 notebooks on enterprise identity and access management in Dynatrace Gen3
IAM: governance, SSO, group architecture, policy authoring, boundaries, user
lifecycle, audit, multi-environment patterns, troubleshooting, templated
policies, a persona-design workshop, and three provisioning flavors
(curl/shell, Terraform, Python) plus a best-practice summary.

## Routing table

Read only the file(s) matching the question. All paths are under `markdown/`.

| When the question is about… | Read |
|---|---|
| The ABAC model, governance roles, account vs environment permissions, centralized vs federated ownership | `-[IAM]-01-governance-foundations.md` |
| SAML 2.0 configuration, IdP integration (Okta/Azure AD), attribute mapping, MFA, SSO troubleshooting | `-[IAM]-02-sso-authentication.md` |
| Group design and hierarchy, naming conventions, SAML group mapping, separation of duties | `-[IAM]-03-group-architecture.md` |
| Policy statement syntax (`ALLOW service:resource:action WHERE …`), condition expressions, `environment:roles:*`, managed vs custom policies, GitOps for policies | `-[IAM]-04-policy-authoring.md` |
| Boundaries vs policies, the three-domain model, `dt.security_context` partitioning, multi-tenant isolation | `-[IAM]-05-boundary-design.md` |
| SCIM provisioning, JIT access, onboarding/offboarding, service accounts and OAuth clients, token management, external users | `-[IAM]-06-user-lifecycle.md` |
| Audit log queries, authentication/authorization events, SOC2/SOX/HIPAA compliance reports, access reviews | `-[IAM]-07-audit-compliance.md` |
| IAM across dev/staging/prod, account vs environment level, break-glass procedures, syncing IAM config | `-[IAM]-08-multi-environment.md` |
| Permission-denied debugging, policy conflict resolution, boundary issues, token 401s, diagnostic queries, escalation | `-[IAM]-09-troubleshooting.md` |
| Parameterized policies with `${bindParam:…}`, binding via IAM API, Monaco policy-as-code, template patterns | `-[IAM]-10-templated-policy-assignments.md` |
| Hands-on workshop: designing a persona-based permissions model (personas, AD alignment, schema/app audits, data boundaries) | `-[IAM]-11-[WORKSHOP]-policy-persona.md` |
| curl/jq provisioning scripts against the Account Management API, OAuth client setup, policy/binding reports, cleanup, validation DQL | `-[IAM]-12-api-provisioning-validation.md` |
| Hands-on lab: provisioning groups, policies, boundaries, and bindings as code with Terraform modules | `-[IAM]-95-[LAB]-terraform-iam-provisioning.md` |
| Hands-on lab: one-command team onboarding (group + shared policy + boundary + binding) with a Python tool | `-[IAM]-96-[LAB]-python-iam-provisioning.md` |
| Consolidated checklist: SSO, groups, policies, boundaries, lifecycle, tokens, audit, governance | `-[IAM]-99-best-practice-summary.md` |

If more than three rows match, start with `-[IAM]-99-best-practice-summary.md`
and follow its pointers.

## Related series

- Migrating existing Management Zones to policies/boundaries: `../MZ2POL - Management Zone to Policy Migration/`
- Data-side access control (buckets, segments, security context): `../ORGNZ - Organize Data: Buckets, Segments, Security/`
- Terraform/Monaco automation beyond IAM objects: `../AUTOM - Dynatrace Automation/`
- SSO and token basics during initial tenant setup: `../ONBRD - Dynatrace Onboarding/`

## Rules

- Read-only; markdown only (see repo-root AGENTS.md for the full format table).
- Filenames contain literal brackets and a leading dash — quote paths in shell.
- Prefer `smartscapeNodes` query forms when quoting; `fetch dt.entity.*`
  variants shown in these notebooks are deprecated alternatives.
- Cite by notebook ID (e.g. "IAM-04") and mention that the matching JSON in
  `notebooks/` can be imported into a Dynatrace tenant for interactive use.
