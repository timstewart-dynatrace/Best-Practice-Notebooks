# OPMIG-08: Security, Masking & Compliance

> **Series:** OPMIG — OpenPipeline Migration | **Notebook:** 8 of 10 | **Created:** December 2025 | **Last Updated:** 10/02/2026

---

## Table of Contents

1. [Understanding Masking](#understanding-masking)
2. [Built-in Masking Patterns](#built-in-masking-patterns)
3. [Custom Masking with replacePattern](#custom-masking-with-replacepattern)
4. [Compliance Patterns](#compliance-patterns)
5. [Field Removal for Security](#field-removal-for-security)
6. [Validating Masking](#validating-masking)
7. [Testing Masking in DPL Architect](#testing-masking-in-dpl-architect)
8. [Best Practices](#best-practices)
9. [Complete Security Pipeline Example](#complete-security-pipeline-example)

---

## Learning Objectives

By completing this notebook, you will:

1. Configure masking processors for PII protection
2. Use built-in and custom masking patterns
3. ⭐ **NEW:** Implement GDPR compliance checklist
4. ⭐ **NEW:** Implement HIPAA compliance checklist
5. ⭐ **NEW:** Implement PCI-DSS compliance checklist
6. ⭐ **NEW:** Implement SOC 2 compliance checklist
7. Validate masking effectiveness
8. Design complete security pipelines

---

## Prerequisites

| Requirement | Details |
|-------------|---------|
| **Dynatrace Environment** | Dynatrace SaaS with Grail and OpenPipeline access — Managed is not covered by this series |
| **Permissions** | `settings:read` and `settings:write` (OpenPipeline configuration) |
| **API Access** | `logs.read` token scope |
| **DPL Architect** | Access to `https://{env}.apps.dynatrace.com/ui/apps/dynatrace.dpl.architect` |
| **Knowledge** | OPMIG-01 through OPMIG-07; understanding of compliance requirements (PCI, HIPAA, GDPR) |

---

## Security Stage Overview

Place masking processors **first within the Processing stage**, before any processor that copies or parses the sensitive field. Masking is not automatic: a processor listed ahead of the masking processor sees the raw value.

**Routing runs before any pipeline processor**, so routing matchers always see the raw value — keep PII out of routing conditions, or mask at capture (OneAgent-side masking) when the value must never reach Dynatrace.

![Masking Executes First](images/masking-order.png)

<!--MARKDOWN_TABLE_ALTERNATIVE
| Stage | Contents | PII Status |
|-------|----------|------------|
| Incoming | card: 4111-..., email: user@... | ⚠️ Contains PII |
| After Masking | card: [CC_REDACTED], email: [EMAIL_REDACTED] | ✅ PII Protected |
| Remaining Stages | Drop, Process, Extract, Store | ✅ Safe (no PII) |
-->

### Why Masking First?

| Order | Reason |
|-------|--------|
| **Before parsing** | Sensitive data never extracted to fields |
| **Not before routing** | Routing precedes every pipeline processor — keep PII out of routing conditions |
| **Before storage** | Compliance-safe from the start |
| **Before extraction** | Metrics/events don't contain PII |

> <sub>**Sources:** [Data flow (DT docs)](https://docs.dynatrace.com/docs/platform/openpipeline/concepts/data-flow) — *"After data is ingested (and optionally pre-processed), it's routed to pipelines."*; [Processing in OpenPipeline (DT docs)](https://docs.dynatrace.com/docs/platform/openpipeline/concepts/processing) — *"The processor order in the stage; each processor output becomes the input for the next one."*</sub>

---

<a id="understanding-masking"></a>
## Understanding Masking
### What Masking Does

Masking replaces sensitive patterns with redacted values:

**Before:**
```
User email: john.doe@example.com, card: 4111-1111-1111-1111
```

**After:**
```
User email: [EMAIL_REDACTED], card: [CC_REDACTED]
```

### Masking with a DQL processor

Masking is done with a **DQL** processor (`replacePattern`, or `fieldsRemove` for whole fields); there is no dedicated masking processor type. A masking processor is configured as:

| Field | Description |
|-------|-------------|
| **Processor type** | DQL |
| **Name** | Descriptive name for the mask |
| **Matching condition** | When to apply masking (`true` for all records) |
| **Definition** | One `fieldsAdd <field> = replacePattern(<field>, "<DPL pattern>", replacement: "<text>")` per field to mask, joined with `\|` |

> <sub>**Sources:** [Processing in OpenPipeline (DT docs)](https://docs.dynatrace.com/docs/platform/openpipeline/concepts/processing) — *"The following table lists alphabetically all available processors in a pipeline."* **Derived:** none of the processors in that table is a masking type; DQL is the one that runs `replacePattern`.</sub>

### Masking vs. Dropping

| Action | Use When |
|--------|----------|
| **Mask** | Need the record, just hide sensitive parts |
| **Drop** | Entire record is not needed |

---

<a id="built-in-masking-patterns"></a>
## Built-in Masking Patterns
OpenPipeline includes pre-built matchers for common sensitive data.

### Credit Card Numbers

**DPL Pattern:**
```
CREDITCARD
```

**Matches:**
- `4111111111111111`
- `4111-1111-1111-1111`
- `4111 1111 1111 1111`

**Masking Processor (DQL):**
```dql
// Name: Mask Credit Cards · Matching condition: true
fieldsAdd content = replacePattern(content, "CREDITCARD", replacement: "[CC_REDACTED]")
| fieldsAdd message = replacePattern(message, "CREDITCARD", replacement: "[CC_REDACTED]")
```

### Email Addresses

> **There is no built-in `EMAIL` matcher in DPL.** A pattern of `EMAIL` is rejected with
> `Named pattern element 'EMAIL' is not valid`, so a masking processor built on it never
> redacts anything. Use the hand-rolled character-class pattern below.

**DPL Pattern:**
```
[a-zA-Z0-9._%+-]+ '@' [a-zA-Z0-9.-]+
```

**Matches:**
- `user@example.com`
- `first.last@company.org`
- `a-b_c%d+e@sub.domain.co.uk`

**Masking Processor (DQL):**
```dql
// Name: Mask Emails · Matching condition: true
fieldsAdd content = replacePattern(content, "[a-zA-Z0-9._%+-]+ '@' [a-zA-Z0-9.-]+", replacement: "[EMAIL_REDACTED]")
| fieldsAdd message = replacePattern(message, "[a-zA-Z0-9._%+-]+ '@' [a-zA-Z0-9.-]+", replacement: "[EMAIL_REDACTED]")
```

> **Note:** the domain class includes `.`, so a sentence-final period is absorbed into the
> match (`mail x@y.io.` → `mail [EMAIL_REDACTED]`). That over-masks by one character rather
> than under-masking — the safe direction for PII.

### IP Addresses

**DPL Pattern:**
```
IPADDR
IPV4ADDR
IPV6ADDR
```

**Masking Processor (DQL):**
```dql
// Name: Mask IP Addresses · Matching condition: true
fieldsAdd content = replacePattern(content, "IPADDR", replacement: "[IP_REDACTED]")
| fieldsAdd client_ip = replacePattern(client_ip, "IPADDR", replacement: "[IP_REDACTED]")
| fieldsAdd remote_addr = replacePattern(remote_addr, "IPADDR", replacement: "[IP_REDACTED]")
```

### Phone Numbers (Custom Pattern)

**DPL Pattern:**
```
'(' [0-9]{3} ')' SPACE? [0-9]{3} '-' [0-9]{4}
```

> **`INT` does not accept a `{n}` quantifier** — it supports only `*` and `+`, and
> `INT{3}` fails with `INT only supports '*' (nullable) and '+' (not nullable) quantifiers`.
> Use a `[0-9]{n}` character class for fixed-width digit runs.

**Matches:**
- `(555) 123-4567`
- `(555)123-4567`

---

<a id="custom-masking-with-replacepattern"></a>
## Custom Masking with replacePattern
The `replacePattern` function enables custom masking using DPL patterns.

### Syntax

```dql
fieldsAdd content = replacePattern(content, "PATTERN", replacement: "REPLACEMENT")
```

### Example: Mask SSN

**Pattern:** SSN format `123-45-6789`

```dql
fieldsAdd content = replacePattern(content, "[0-9]{3} '-' [0-9]{2} '-' [0-9]{4}", replacement: "[SSN_REDACTED]")
```

### Example: Mask API Keys

**Pattern:** Keys starting with `key_`, masked through to the next whitespace

```dql
fieldsAdd content = replacePattern(content, "'key_' NSPACE", replacement: "[API_KEY_REDACTED]")
```

> **Why not `'key_' WORD`?** `WORD` matches a greedy run of alphanumerics and underscores,
> but stops at `-`. Against `key_XY-99` it yields `[API_KEY_REDACTED]-99`, leaving part of
> the secret in the log. `NSPACE` consumes to the next whitespace and masks the whole token.

### Example: Mask Bearer Tokens

**Pattern:** Bearer tokens in Authorization headers

```dql
fieldsAdd content = replacePattern(content, "'Bearer ' NSPACE", replacement: "Bearer [TOKEN_REDACTED]")
```

### Example: Mask Passwords in URLs

**Pattern:** Password parameter in query strings

```dql
fieldsAdd content = replacePattern(content, "'password=' [^&\\s]+", replacement: "password=[REDACTED]")
```

> **Why not `'password=' LD:pwd ('&'|EOL)`?** It only masks when another parameter follows. On `…?user=bob&password=hunter2`, with the password last, it matched nothing and the password was **stored in clear** (verified 10/02/2026). The DPL character class `[^&\s]+` (written `\\s` inside the DQL string) stops at the next `&` or whitespace and masks both cases.

### Chaining Multiple Masks

```dql
// Apply multiple masks in sequence
fieldsAdd content = replacePattern(content, "CREDITCARD", replacement: "[CC_REDACTED]")
| fieldsAdd content = replacePattern(content, "[a-zA-Z0-9._%+-]+ '@' [a-zA-Z0-9.-]+", replacement: "[EMAIL_REDACTED]")
| fieldsAdd content = replacePattern(content, "IPADDR", replacement: "[IP_REDACTED]")
```

---

<a id="compliance-patterns"></a>
## Compliance Patterns
> **End a key=value mask with `NSPACE`, not `LD`.** A trailing `LD` runs to the end of the line, so `'userId=' LD:uid` turns `userId=u-77 action=view status=OK` into `userId=[USER_REDACTED]` — everything after the key is destroyed, including fields you parse later. `NSPACE` stops at the next space (verified 10/02/2026). If a value can contain spaces, close the pattern with its delimiter instead.

### PCI-DSS Compliance

**Requirements:**
- Mask all Primary Account Numbers (PANs)
- Mask CVV/CVC codes
- Mask cardholder names when combined with PANs

**Pipeline Configuration:**

```dql
// Mask credit card numbers
fieldsAdd content = replacePattern(content, "CREDITCARD", replacement: "[PAN_REDACTED]")
// Mask CVV (3-4 digits after 'cvv=' or 'cvc=')
| fieldsAdd content = replacePattern(content, "('cvv='|'cvc='|'CVV='|'CVC=') [0-9]{3,4}", replacement: "cvv=[REDACTED]")
```

### GDPR Compliance

**Requirements:**
- Mask personal identifiers (emails, phone numbers)
- Mask IP addresses (considered PII in EU)
- Mask names when identifiable

**Pipeline Configuration:**

```dql
// Mask emails
fieldsAdd content = replacePattern(content, "[a-zA-Z0-9._%+-]+ '@' [a-zA-Z0-9.-]+", replacement: "[PII_REDACTED]")
// Mask IP addresses
| fieldsAdd content = replacePattern(content, "IPADDR", replacement: "[IP_REDACTED]")
// Mask user IDs
| fieldsAdd content = replacePattern(content, "'userId=' NSPACE", replacement: "userId=[USER_REDACTED]")
```

### HIPAA Compliance

**Requirements:**
- Mask Protected Health Information (PHI)
- Mask patient identifiers
- Mask medical record numbers

**Pipeline Configuration:**

```dql
// Mask patient IDs
fieldsAdd content = replacePattern(content, "'patientId=' NSPACE", replacement: "patientId=[PHI_REDACTED]")
// Mask MRNs
| fieldsAdd content = replacePattern(content, "'mrn=' NSPACE", replacement: "mrn=[PHI_REDACTED]")
// Mask SSNs
| fieldsAdd content = replacePattern(content, "[0-9]{3} '-' [0-9]{2} '-' [0-9]{4}", replacement: "[SSN_REDACTED]")
```

### SOC 2 Compliance

**Requirements:**
- Mask credentials and secrets
- Mask authentication tokens
- Audit access to sensitive data

**Pipeline Configuration:**

```dql
// Mask passwords
fieldsAdd content = replacePattern(content, "('password='|'pwd='|'passwd=') NSPACE", replacement: "password=[REDACTED]")
// Mask API keys
| fieldsAdd content = replacePattern(content, "('api_key='|'apiKey='|'API_KEY=') NSPACE", replacement: "api_key=[REDACTED]")
// Mask tokens
| fieldsAdd content = replacePattern(content, "'Bearer ' NSPACE", replacement: "Bearer [REDACTED]")
```

---

<a id="field-removal-for-security"></a>
## Field Removal for Security
Sometimes it's better to remove entire fields rather than mask values.

### Using fieldsRemove

```dql
// Remove sensitive fields entirely
fieldsRemove password, secret, api_key, token, authorization
```

### When to Remove vs. Mask

| Scenario | Action |
|----------|--------|
| Field always contains sensitive data | Remove |
| Only some values are sensitive | Mask |
| Need to know field existed | Mask with placeholder |
| Compliance requires no trace | Remove |

### Conditional Field Removal

Apply removal only when field contains sensitive patterns:

```dql
// Remove field only if it contains a pattern
fieldsAdd authorization = if(contains(authorization, "Bearer"), null, else: authorization)
```

---

<a id="validating-masking"></a>
## Validating Masking
After configuring masking, verify it's working correctly.

> ⚠️ **Important:** Test with sample data before deploying to production.

```dql
// Check for any remaining card numbers in stored logs — if masking works, this returns 0 rows.
// replacePattern() finds a CREDITCARD match anywhere; matchesPhrase(content, "4111") would not,
// because an unbroken 16-digit number is a single token that "4111" never equals.
fetch logs, from: now() - 24h
| filter replacePattern(content, "CREDITCARD", replacement: "") != content
| fields timestamp, log.source, dt.openpipeline.pipelines
| limit 10
```

```dql
// Check for any remaining email addresses — if masking works, this returns 0 rows.
// Uses the same DPL pattern as the masking processor, so it finds any address, not a fixed domain list.
fetch logs, from: now() - 24h
| filter replacePattern(content, "[a-zA-Z0-9._%+-]+ '@' [a-zA-Z0-9.-]+", replacement: "") != content
| fields timestamp, log.source, dt.openpipeline.pipelines
| limit 10
```

```dql
// Verify redaction placeholders are present
// This confirms masking is actively working
fetch logs, from: now() - 24h
| filter contains(content, "[REDACTED]")
   OR contains(content, "[CC_REDACTED]")
   OR contains(content, "[EMAIL_REDACTED]")
   OR contains(content, "[PII_REDACTED]")
| summarize {masked_count = count()}, by: {dt.openpipeline.pipelines}
| sort masked_count desc
```

```dql
// Sample masked logs to verify format
fetch logs, from: now() - 24h
| filter contains(content, "REDACTED")
| fields timestamp, content
| limit 20
```

```dql
// Audit: Count masked records by pipeline and type
fetch logs, from: now() - 24h
| fieldsAdd has_cc_mask = contains(content, "[CC_REDACTED]"),
           has_email_mask = contains(content, "[EMAIL_REDACTED]"),
           has_ip_mask = contains(content, "[IP_REDACTED]"),
           has_pii_mask = contains(content, "[PII_REDACTED]")
| summarize {
    total = count(),
    cc_masked = countIf(has_cc_mask),
    email_masked = countIf(has_email_mask),
    ip_masked = countIf(has_ip_mask),
    pii_masked = countIf(has_pii_mask)
  }, by: {dt.openpipeline.pipelines}
```

```dql
// Check for SSN-shaped values (###-##-####) that masking missed — should return 0 rows
fetch logs, from: now() - 24h
| filter replacePattern(content, "[0-9]{3} '-' [0-9]{2} '-' [0-9]{4}", replacement: "") != content
| fields timestamp, log.source, dt.openpipeline.pipelines
| limit 50
```

---

<a id="testing-masking-in-dpl-architect"></a>
## Testing Masking in DPL Architect
Before deploying masking rules, test them:

1. Open **DPL Architect** in Dynatrace
2. Paste sample log with sensitive data
3. Test your DPL pattern matches correctly
4. Verify replacement produces expected output

### Test Cases to Include

| Test Case | Input | Expected Output |
|-----------|-------|------------------|
| Credit card | `4111-1111-1111-1111` | `[CC_REDACTED]` |
| Credit card no dashes | `4111111111111111` | `[CC_REDACTED]` |
| Email | `user@example.com` | `[EMAIL_REDACTED]` |
| IPv4 | `192.168.1.1` | `[IP_REDACTED]` |
| IPv6 | `2001:db8::1` | `[IP_REDACTED]` |
| Mixed content | Multiple patterns | All redacted |

---

<a id="best-practices"></a>
## Best Practices
### Masking Configuration

| Practice | Reason |
|----------|--------|
| Keep PII out of routing conditions | Routing runs before any pipeline processor, so routing matchers see the raw value |
| Use descriptive placeholders | Aids troubleshooting |
| Test patterns thoroughly | Avoid over/under masking |
| Document masking rules | Compliance audits |

### Pattern Design

| Practice | Reason |
|----------|--------|
| Be specific | Avoid false positives |
| Handle variations | Same data, different formats |
| Use built-in matchers | Pre-tested, reliable |
| Chain patterns | Cover all sensitive data |

### Compliance

| Practice | Reason |
|----------|--------|
| Document all masking | Audit trail |
| Regular pattern review | New data sources |
| Test with real samples | Ensure effectiveness |
| Monitor for leaks | Continuous validation |

### Performance

| Practice | Reason |
|----------|--------|
| Apply to specific fields | Faster than all fields |
| Use matching conditions | Skip unneeded records |
| Order patterns by frequency | Most common first |

---

<a id="complete-security-pipeline-example"></a>
## Complete Security Pipeline Example
### Pipeline: `payment-logs-secure`

**Masking Processor 1: Credit Cards** (DQL processor, matching condition `true`)
```dql
fieldsAdd content = replacePattern(content, "CREDITCARD", replacement: "[CC_REDACTED]")
| fieldsAdd card_number = replacePattern(card_number, "CREDITCARD", replacement: "[CC_REDACTED]")
```

**Masking Processor 2: CVV Codes**
```dql
fieldsAdd content = replacePattern(content, "('cvv='|'cvc=') [0-9]{3,4}", replacement: "cvv=[REDACTED]")
```

**Masking Processor 3: Emails** (DQL processor, matching condition `true`)
```dql
fieldsAdd content = replacePattern(content, "[a-zA-Z0-9._%+-]+ '@' [a-zA-Z0-9.-]+", replacement: "[EMAIL_REDACTED]")
| fieldsAdd customer_email = replacePattern(customer_email, "[a-zA-Z0-9._%+-]+ '@' [a-zA-Z0-9.-]+", replacement: "[EMAIL_REDACTED]")
```

**Masking Processor 4: IP Addresses** (DQL processor, matching condition `true`)
```dql
fieldsAdd content = replacePattern(content, "IPADDR", replacement: "[IP_REDACTED]")
| fieldsAdd client_ip = replacePattern(client_ip, "IPADDR", replacement: "[IP_REDACTED]")
| fieldsAdd remote_addr = replacePattern(remote_addr, "IPADDR", replacement: "[IP_REDACTED]")
```

**Masking Processor 5: Auth Tokens**
```dql
fieldsAdd content = replacePattern(content, "'Bearer ' NSPACE", replacement: "Bearer [TOKEN_REDACTED]")
| fieldsRemove authorization, auth_token
```

### Pipeline Verification Query

```dql
// Verify complete masking for payment-logs pipeline
fetch logs, from: now() - 1h
// dt.openpipeline.pipelines is an array of "<scope>:<pipeline id>" strings, so == never matches
| filter matchesValue(dt.openpipeline.pipelines, "*payment-logs-secure*")
| summarize {
    total = count(),
    with_cc = countIf(contains(content, "[CC_REDACTED]")),
    with_email = countIf(contains(content, "[EMAIL_REDACTED]")),
    with_ip = countIf(contains(content, "[IP_REDACTED]")),
    with_token = countIf(contains(content, "[TOKEN_REDACTED]"))
  }
```

---

## Next Steps

Now that security is configured, complete your migration:

| Notebook | Focus Area |
|----------|------------|
| **OPMIG-09** | Troubleshooting & Validation |

---

## References

- [OpenPipeline Masking](https://docs.dynatrace.com/docs/platform/openpipeline/use-cases)
- [DPL replacePattern](https://docs.dynatrace.com/docs/platform/grail/dynatrace-pattern-language)
- [Data Privacy in Dynatrace](https://docs.dynatrace.com/docs/manage/data-privacy-and-security)
- [Compliance Best Practices](https://docs.dynatrace.com/docs/manage/data-privacy-and-security/data-privacy)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
