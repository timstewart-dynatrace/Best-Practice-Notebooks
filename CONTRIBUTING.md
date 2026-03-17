# Contributing to Best-Practice-Notebooks

Thank you for your interest in contributing! This guide explains how to submit changes, what maintainers look for during review, and how pull requests may be accepted, revised, or rejected.

> **Note:** These assets are not officially supported by Dynatrace. Maintainers are volunteers and review time may vary.

---

## Table of Contents

- [How to Contribute](#how-to-contribute)
- [Pull Request Review Process](#pull-request-review-process)
- [Acceptance Criteria](#acceptance-criteria)
- [Reasons a Pull Request May Be Rejected or Closed](#reasons-a-pull-request-may-be-rejected-or-closed)
- [How to Respond to Feedback](#how-to-respond-to-feedback)
- [Appealing a Decision](#appealing-a-decision)

---

## How to Contribute

1. **Fork** the repository and create a branch from `main`.
2. Follow the existing folder structure:
   ```
   <TOPIC-CODE> - <Topic name>/
   ├── NOTEBOOKS/   # Dynatrace notebook JSON files
   ├── PDFs/        # Exported PDF versions
   ├── markdown/    # Markdown exports
   └── README.md    # Topic overview
   ```
3. Open a **Pull Request** against `main` and fill in the PR template.
4. Respond promptly to review comments to keep the PR active.

---

## Pull Request Review Process

When a pull request is opened, a maintainer will:

1. **Check for completeness** — all three formats (JSON, PDF, markdown) should be present where applicable, and a README entry should be added or updated.
2. **Review content quality** — accuracy against current Dynatrace documentation, DQL correctness, and clarity of explanations.
3. **Verify naming conventions** — files and folders must follow the `[TOPIC-CODE]-NN-slug` pattern used throughout the repository.
4. **Leave a review decision** — one of:
   - ✅ **Approve** — the PR is merged as-is.
   - 💬 **Request Changes** — the PR needs revisions before it can be merged; the maintainer will explain what is needed.
   - ❌ **Close / Reject** — the PR will not be merged; see below for common reasons.

---

## Acceptance Criteria

A pull request is more likely to be merged when it:

- Adds or improves content within the existing topic scope.
- Includes accurate, tested DQL queries.
- Follows the file and folder naming conventions.
- Provides all three deliverable formats (notebook JSON, PDF, markdown) where possible.
- Keeps changes focused — one topic or fix per PR.
- Has a clear description of what changed and why.

---

## Reasons a Pull Request May Be Rejected or Closed

Maintainers may **close a pull request without merging** for any of the following reasons:

| Reason | Description |
|--------|-------------|
| **Out of scope** | The content does not fit the Dynatrace best-practice notebook format or covers a topic outside the repository's current roadmap. |
| **Duplicate** | The same content already exists or is covered by another open PR. |
| **Inaccurate content** | The notebook contains incorrect DQL, outdated API references, or advice that contradicts official Dynatrace documentation. |
| **Missing deliverables** | The PR is missing the notebook JSON, PDF export, or markdown export without a clear reason. |
| **Naming convention violations** | Files or folders do not follow the established `[TOPIC-CODE]-NN-slug` pattern and the contributor has not addressed feedback. |
| **No response** | The contributor has not responded to review feedback within **30 days** and the branch has gone stale. |
| **Breaking changes** | The PR restructures existing content in a way that would break links or navigation without a compelling reason. |
| **Quality below bar** | Despite revision requests, the content does not reach the quality level maintained across the repository. |

When a PR is closed, the maintainer will leave a comment explaining the specific reason.

---

## How to Respond to Feedback

- **Request Changes:** Push new commits to your branch addressing each point raised. Re-request review once you have made all changes.
- **Closed / Rejected:** Read the closing comment carefully. If you believe the feedback can be addressed, you are welcome to open a **new** pull request with the issues resolved.

---

## Appealing a Decision

If you believe a PR was closed in error:

1. Leave a comment on the **closed pull request** explaining your reasoning.
2. Tag a maintainer (`@timstewart-dynatrace`) for visibility.
3. Maintainers will re-evaluate and reopen the PR if appropriate.

Please keep all communication respectful. Decisions made in good faith by maintainers are final unless new information is provided.

---

## Questions?

Open a [GitHub Discussion](../../discussions) or file an issue if you have questions before submitting a contribution.
