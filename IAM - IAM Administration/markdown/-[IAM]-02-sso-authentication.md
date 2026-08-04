# IAM-02: SSO and Authentication Configuration

> **Series:** IAM — IAM Administration | **Notebook:** 2 of 12 | **Created:** January 2026 | **Last Updated:** 08/03/2026

## Setting Up Enterprise Authentication
Single Sign-On (SSO) is the foundation of enterprise IAM. Configure SSO before setting up groups, policies, or user provisioning. This notebook covers SAML configuration, IdP integration, and authentication best practices.

---

## Table of Contents

1. [Why SSO First?](#why-sso-first)
2. [SAML 2.0 Fundamentals](#saml-20-fundamentals)
3. [IdP Configuration](#idp-configuration)
4. [Dynatrace SAML Setup](#dynatrace-saml-setup)
5. [Attribute Mapping](#attribute-mapping)
6. [Testing SSO](#testing-sso)
7. [Multi-Factor Authentication](#multi-factor-authentication)
8. [Troubleshooting SSO](#troubleshooting-sso)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Dynatrace Environment** | SaaS with Gen3 IAM enabled |
| **Permissions** | Account administrator |
| **IdP Access** | Admin access to enterprise IdP (Okta, Azure AD, etc.) |

<a id="why-sso-first"></a>
## 1. Why SSO First?
SSO must be configured before other IAM components because:

| Dependency | Reason |
|------------|--------|
| **Groups** | Groups are synced from IdP via SAML assertions |
| **Users** | Users are provisioned through SSO (SCIM or JIT) |
| **Policies** | Policies are assigned to groups that come from IdP |
| **Audit** | Authentication events require SSO integration |

### Authentication Flow

```
User → IdP Login → SAML Assertion → Dynatrace
                        ↓
              Contains: Email, Groups, Attributes
                        ↓
              Dynatrace maps to local groups
                        ↓
              Policies and boundaries applied
```

### Order of Operations

1. **Configure SSO** (this notebook)
2. Design group architecture (IAM-03)
3. Create policies (IAM-04)
4. Design boundaries (IAM-05)
5. Enable SCIM/JIT provisioning (IAM-06)

<a id="saml-20-fundamentals"></a>
## 2. SAML 2.0 Fundamentals
Dynatrace uses SAML 2.0 for SSO authentication.

### Key Concepts

| Term | Definition |
|------|------------|
| **IdP** | Identity Provider - authenticates users (Okta, Azure AD, etc.) |
| **SP** | Service Provider - Dynatrace, the application being accessed |
| **Assertion** | XML document containing user identity and attributes |
| **ACS URL** | Assertion Consumer Service - where Dynatrace receives assertions |
| **Entity ID** | Unique identifier for each party |

### SAML Flow

```
1. User accesses Dynatrace
2. Dynatrace redirects to IdP
3. User authenticates with IdP
4. IdP generates SAML assertion
5. IdP redirects back to Dynatrace with assertion
6. Dynatrace validates assertion
7. User is logged in with appropriate permissions
```

### Required SAML Attributes

| Attribute | Purpose | Example |
|-----------|---------|----------|
| **NameID** | User identifier (email) | `user@company.com` |
| **Groups** | Group memberships | `IT-DT-Admins, All-Employees` |
| **FirstName** | Display name | `John` |
| **LastName** | Display name | `Smith` |

<a id="idp-configuration"></a>
## 3. IdP Configuration
Configure your Identity Provider to work with Dynatrace.

### Supported Identity Providers

| IdP | SSO Support | SCIM Support | Notes |
|-----|-------------|--------------|-------|
| **Okta** | Full | Full | Native integration |
| **Azure AD (Entra ID)** | Full | Full | Enterprise app |
| **OneLogin** | Full | Full | App catalog |
| **PingFederate** | Full | Full | Federation server |
| **Google Workspace** | Full | Partial | Custom SAML app |
| **ADFS** | Full | Manual | On-premises |

### IdP Setup Steps (Generic)

1. **Create SAML Application**
   - Log into IdP admin console
   - Create new SAML 2.0 application
   - Name it "Dynatrace" or "Dynatrace [Environment]"

2. **Configure Service Provider Settings**
   - ACS URL: `https://sso.dynatrace.com/sso/saml2/<account-uuid>`
   - Entity ID: `https://sso.dynatrace.com/sso/saml2/<account-uuid>`
   - Name ID Format: Email address

3. **Configure Attributes**
   - Map user email to NameID
   - Add groups attribute
   - Add first/last name attributes

4. **Assign Users/Groups**
   - Assign IdP groups that should access Dynatrace
   - Or assign all users and control access via Dynatrace groups

### Okta Configuration Example

1. **Create Application**
   - Applications → Create App Integration
   - Select SAML 2.0
   - Name: "Dynatrace"

2. **SAML Settings**
   ```
   Single sign-on URL: https://sso.dynatrace.com/sso/saml2/<account-uuid>
   Audience URI (SP Entity ID): https://sso.dynatrace.com/sso/saml2/<account-uuid>
   Name ID format: EmailAddress
   Application username: Email
   ```

3. **Attribute Statements**
   | Name | Value |
   |------|-------|
   | `email` | `user.email` |
   | `firstName` | `user.firstName` |
   | `lastName` | `user.lastName` |

4. **Group Attribute Statements**
   | Name | Filter |
   |------|--------|
   | `groups` | Matches regex `.*` (or filter to DT groups) |

### Azure AD (Entra ID) Configuration Example

1. **Enterprise Application**
   - Enterprise Applications → New Application
   - Create your own application
   - Name: "Dynatrace"

2. **Single Sign-On → SAML**
   ```
   Identifier (Entity ID): https://sso.dynatrace.com/sso/saml2/<account-uuid>
   Reply URL (ACS URL): https://sso.dynatrace.com/sso/saml2/<account-uuid>
   ```

3. **Attributes & Claims**
   | Claim | Source attribute |
   |-------|------------------|
   | `emailaddress` | `user.mail` |
   | `givenname` | `user.givenname` |
   | `surname` | `user.surname` |
   | `groups` | Group IDs or names |

<a id="dynatrace-saml-setup"></a>
## 4. Dynatrace SAML Setup
Configure Dynatrace to accept SAML assertions from your IdP.

### Steps in Dynatrace Account Management

1. **Navigate to Identity Providers**
   - Account Management → Identity & access management → Identity providers

2. **Add SAML Configuration**
   - Click "Add identity provider"
   - Select "SAML 2.0"

3. **Enter IdP Information**
   - **Name**: Descriptive name (e.g., "Corporate Okta")
   - **IdP Entity ID**: From IdP metadata
   - **IdP SSO URL**: From IdP metadata
   - **IdP Certificate**: Upload X.509 certificate from IdP

4. **Get Dynatrace SP Information**
   - Copy ACS URL and Entity ID to configure in IdP
   - Download Dynatrace SP metadata if IdP supports it

### Configuration Options

| Option | Recommendation | Notes |
|--------|----------------|-------|
| **Sign assertions** | Required | Validate IdP signatures |
| **Encrypt assertions** | Optional | Additional security |
| **Sign requests** | Optional | SP signs auth requests |
| **Allow IdP-initiated** | Optional | Direct IdP login |

<a id="attribute-mapping"></a>
## 5. Attribute Mapping
Map SAML attributes to Dynatrace user properties.

### User Attribute Mapping

| SAML Attribute | Dynatrace Field | Required |
|----------------|-----------------|----------|
| NameID | User email/login | Yes |
| firstName | First name | Recommended |
| lastName | Last name | Recommended |
| email | Email (if not NameID) | If NameID is different |

### Group Attribute Mapping

Groups from SAML assertions map to Dynatrace groups:

```
IdP Group (in assertion)  →  Dynatrace Group
IT-DT-Platform-Admins     →  dt-platform-admins
IT-DT-Checkout-Team       →  dt-checkout-editors
IT-DT-Security-Audit      →  dt-security-viewers
```

### Configuring Group Mapping

1. **In Dynatrace Account Management**
   - Identity providers → Select your IdP
   - Group mapping section

2. **Add Mappings**
   - IdP group name (exact match)
   - Dynatrace group to assign

3. **Options**
   | Option | Description |
   |--------|-------------|
   | Auto-create groups | Create DT group if not exists |
   | Default group | Assign all users to a base group |
   | Strict mapping | Only mapped groups, ignore others |

### Group Naming Strategy

| Approach | IdP Group | DT Group | Pros/Cons |
|----------|-----------|----------|------------|
| **Exact match** | `dt-admins` | `dt-admins` | Simple, but IdP naming restricted |
| **Prefixed** | `IT-DT-Admins` | `dt-admins` | Explicit mapping needed |
| **Descriptive** | `Dynatrace Platform Admins` | `dt-platform-admins` | Clear in IdP, mapping required |

<a id="testing-sso"></a>
## 6. Testing SSO
Verify SSO works correctly before rolling out.

### Test Checklist

```
□ SP-initiated login works (start from Dynatrace)
□ IdP-initiated login works (start from IdP portal)
□ User attributes map correctly (name, email)
□ Groups sync correctly
□ Logout works (single logout)
□ Session timeout behaves correctly
```

### Testing Steps

1. **Test with Admin Account First**
   - Use an IdP admin account
   - Verify login works
   - Check group assignments

2. **Test SP-Initiated Login**
   - Go to Dynatrace login page
   - Click SSO login
   - Should redirect to IdP
   - Authenticate and return to Dynatrace

3. **Test IdP-Initiated Login**
   - Go to IdP portal
   - Click Dynatrace app tile
   - Should land in Dynatrace authenticated

4. **Verify User Profile**
   - Check user's name is correct
   - Check user's email is correct
   - Check group memberships

### Common Test Failures

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| Redirect loop | Misconfigured ACS URL | Verify URL matches exactly |
| Invalid signature | Certificate mismatch | Re-upload IdP certificate |
| User not found | NameID format wrong | Use email format |
| No groups | Groups attribute missing | Add groups to assertion |

<a id="multi-factor-authentication"></a>
## 7. Multi-Factor Authentication
MFA should be enforced at the IdP level for Dynatrace access.

### MFA Strategy

| Approach | Description | Recommendation |
|----------|-------------|----------------|
| **IdP MFA** | MFA at IdP login | Recommended |
| **Dynatrace MFA** | Additional MFA in Dynatrace | For sensitive actions |
| **Conditional** | MFA based on risk | For adaptive security |

### IdP MFA Configuration

**Okta:**
- Security → Authenticators
- Add factors (TOTP, Push, WebAuthn)
- Create sign-on policy requiring MFA for Dynatrace app

**Azure AD:**
- Azure AD → Security → MFA
- Conditional Access policies
- Require MFA for Dynatrace enterprise app

### MFA Best Practices

| Practice | Rationale |
|----------|----------|
| Require MFA for all users | Baseline security |
| Use phishing-resistant factors | WebAuthn > TOTP > SMS |
| Allow backup factors | Prevent lockout |
| Enforce for privileged users | Critical for admins |

<a id="troubleshooting-sso"></a>
## 8. Troubleshooting SSO
Common SSO issues and their resolutions.

### SAML Debugging Tools

| Tool | Purpose |
|------|----------|
| Browser dev tools | View SAML response in network tab |
| SAML tracer extension | Decode SAML in browser |
| Base64 decoder | Decode assertion manually |
| IdP logs | View authentication events |

### Decoding SAML Response

1. Open browser dev tools → Network
2. Perform SSO login
3. Find POST to ACS URL
4. Copy SAMLResponse value
5. Base64 decode to see XML

### Common Issues

| Issue | Symptom | Resolution |
|-------|---------|------------|
| **Clock skew** | "Assertion expired" | Sync IdP/Dynatrace time |
| **Certificate expired** | Signature invalid | Renew and update cert |
| **Wrong ACS URL** | 404 or loop | Correct URL in IdP |
| **Missing groups** | No permissions | Add groups to assertion |
| **Wrong NameID** | User mismatch | Use email format |
| **Audience mismatch** | Audience error | Match Entity ID exactly |

```dql
// Monitor SSO authentication events
fetch logs, from: now() - 24h
| filter matchesPhrase(log.source, "audit")
| filter matchesPhrase(content, "saml") or matchesPhrase(content, "sso") or matchesPhrase(content, "authentication")
| fields timestamp, content
| sort timestamp desc
| limit 50
```

```dql
// Find failed authentication attempts
fetch logs, from: now() - 7d
| filter matchesPhrase(log.source, "audit")
| filter matchesPhrase(content, "failed") and (matchesPhrase(content, "login") or matchesPhrase(content, "authentication"))
| fields timestamp, content
| sort timestamp desc
| limit 50
```

## Next Steps

With SSO configured, proceed to group architecture:

### Recommended Path

1. **IAM-03: Group Architecture and Design** - Design groups that map from IdP
2. **IAM-04: Policy Authoring** - Create policies for groups
3. **IAM-06: User Lifecycle** - Enable SCIM/JIT provisioning

### SSO Checklist

Before moving on, ensure you have:

- [ ] IdP SAML application configured
- [ ] Dynatrace identity provider added
- [ ] Attribute mapping configured
- [ ] Group mapping planned
- [ ] SSO login tested successfully
- [ ] MFA enabled at IdP

---

## Summary

In this notebook, you learned:

- Why SSO must be configured first
- SAML 2.0 fundamentals
- IdP configuration for Okta and Azure AD
- Dynatrace SAML setup process
- Attribute and group mapping
- SSO testing methodology
- MFA best practices
- Troubleshooting common SSO issues

---

## References

- [Single Sign-On](https://docs.dynatrace.com/docs/manage/identity-access-management/user-and-group-management/access-saml)
- [SAML 2.0 Configuration](https://docs.dynatrace.com/docs/manage/identity-access-management/user-and-group-management/access-saml/saml-configurations)
- [Okta Integration](https://docs.dynatrace.com/docs/manage/identity-access-management/user-and-group-management/access-saml/idp-specific/saml-okta)
- [Azure AD Integration](https://docs.dynatrace.com/docs/manage/identity-access-management/user-and-group-management/access-saml/idp-specific/saml-azure)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
