# ORGNZ - Organize Data: Buckets, Segments, Security

Organizing data in Dynatrace Grail using buckets, segments, and security context for access control.

> **Recommended:** For the best experience, import the notebooks into your Dynatrace tenant. Each one is available as a Dynatrace notebook (JSON) file, and its DQL queries run against your own environment's data.

## Structure
- notebooks/ — Dynatrace notebook JSON files
- pdfs/ — Printable versions of each notebook
- markdown/ — Markdown exports of the notebooks

## Notebook Lineup
1. [Introduction to Organizing Data in Grail](markdown/-[ORGNZ]-01-introduction-to-organizing-data.md) — Why data organization matters in Grail
2. [Understanding Grail Buckets](markdown/-[ORGNZ]-02-understanding-grail-buckets.md) — Bucket fundamentals and data types
3. [Bucket Strategy and Design](markdown/-[ORGNZ]-03-bucket-strategy-and-design.md) — Naming conventions and retention planning
4. [Permissions in Grail Overview](markdown/-[ORGNZ]-04-permissions-in-grail.md) — Permission levels and the canonical `dt.security_context` boundary pattern
5. [Bucket-Level Access Control](markdown/-[ORGNZ]-05-bucket-level-access-control.md) — IAM policies for bucket access and when bucket-level isolation fits
6. [Security Context](markdown/-[ORGNZ]-06-security-context.md) — Fine-grained record-level access with dt.security_context
7. [Advanced Permission Patterns](markdown/-[ORGNZ]-07-advanced-permission-patterns.md) — Record and field-level permissions
8. [Grail Segments](markdown/-[ORGNZ]-08-grail-segments.md) — Logical data filtering with segments
9. [Enterprise Data Organization Patterns](markdown/-[ORGNZ]-09-enterprise-patterns.md) — Combining buckets, segments, and security context for enterprise governance
10. [Advanced Segment Definitions](markdown/-[ORGNZ]-10-advanced-segment-definitions.md) — Complex segment filters with multi-include patterns
99. [Best Practice Summary & DQL Reference](markdown/-[ORGNZ]-99-best-practice-summary.md) — Consolidated best practices and validation DQL from the ORGNZ series

## Usage
1. Open a notebook from the lineup below. Each one is also available as a Dynatrace notebook (JSON) to import into your tenant, so its queries run against your own data, and as a PDF for print.
2. Start with Introduction to Organizing Data, then progress through the series.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
