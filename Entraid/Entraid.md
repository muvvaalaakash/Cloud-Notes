# Microsoft Entra ID — Comprehensive Deep Dive

## Table of Contents
1. [What is Entra ID?](#1-what-is-entra-id)
2. [Core Concepts & Terminology](#2-core-concepts--terminology)
3. [Architecture](#3-architecture)
4. [Authentication & Protocols](#4-authentication--protocols)
5. [Identity Types](#5-identity-types)
6. [Licensing / Editions](#6-licensing--editions)
7. [Conditional Access](#7-conditional-access)
8. [Multi-Factor Authentication (MFA)](#8-multi-factor-authentication)
9. [Privileged Identity Management (PIM)](#9-privileged-identity-management)
10. [Identity Protection](#10-identity-protection)
11. [Application Integration](#11-application-integration)
12. [Hybrid Identity](#12-hybrid-identity)
13. [B2B & B2C](#13-b2b--b2c)
14. [Governance & Entitlement Management](#14-governance--entitlement-management)
15. [RBAC vs Entra ID Roles](#15-rbac-vs-entra-id-roles)
16. [Security Best Practices](#16-security-best-practices)
17. [Monitoring & Logging](#17-monitoring--logging)
18. [Entra ID vs Traditional AD DS](#18-entra-id-vs-traditional-ad-ds)
19. [Related Entra Products](#19-related-entra-products)
20. [Real-World Architecture Diagram](#20-real-world-architecture-diagram)

---

## 1. What is Entra ID?

**Microsoft Entra ID** (formerly **Azure Active Directory / Azure AD**) is Microsoft's **cloud-based Identity and Access Management (IAM)** service. It is the backbone of identity for:

- Microsoft 365
- Azure Portal & Azure Resource Management
- Thousands of SaaS applications
- Custom / line-of-business applications

> **Key rebranding (July 2023):** Azure AD → Microsoft Entra ID. The technology is the same; only the name changed.

### What problems does it solve?
| Problem | Entra ID Solution |
|---|---|
| Who are you? | **Authentication** (passwords, MFA, passwordless) |
| What can you access? | **Authorization** (roles, groups, Conditional Access) |
| How do we manage identities at scale? | **Lifecycle management** (provisioning, deprovisioning, access reviews) |
| How do we secure access? | **Zero Trust policies** (Conditional Access, Identity Protection, PIM) |
| How do we extend on-prem identity to the cloud? | **Hybrid Identity** (Entra Connect, Cloud Sync) |

---

## 2. Core Concepts & Terminology

### Tenant
```
┌──────────────────────────────────────────┐
│           ENTRA ID TENANT                │
│  (A dedicated instance of Entra ID)      │
│                                          │
│  Tenant ID: a3b2c1d0-xxxx-xxxx-xxxx     │
│  Primary domain: contoso.onmicrosoft.com │
│  Custom domain: contoso.com              │
│                                          │
│  Contains:                               │
│    • Users          • Groups             │
│    • Applications   • Service Principals │
│    • Devices        • Policies           │
│    • Roles          • Licenses           │
└──────────────────────────────────────────┘
```

- Every Azure subscription **trusts** exactly **one** Entra ID tenant.
- One tenant can have **many** subscriptions.
- A tenant is globally unique, identified by a **Tenant ID (GUID)** and a **primary domain**.

### Directory Objects
| Object | Description |
|---|---|
| **User** | Represents a person (member or guest) |
| **Group** | Collection of users/devices/service principals (Security or M365) |
| **Application Registration** | The identity definition of an app (global blueprint) |
| **Service Principal** | The local instance of an app registration in a tenant |
| **Managed Identity** | Auto-managed identity for Azure resources (no credentials to manage) |
| **Device** | Represents a registered/joined device |

---

## 3. Architecture

```
                        ┌─────────────────────┐
                        │   MICROSOFT CLOUD    │
                        │                      │
    ┌───────────────────┼──────────────────────┼───────────────────┐
    │                   │   ENTRA ID TENANT    │                   │
    │                   └──────────┬───────────┘                   │
    │                              │                               │
    │    ┌─────────────┬───────────┼───────────┬──────────────┐   │
    │    │             │           │           │              │   │
    │    ▼             ▼           ▼           ▼              ▼   │
    │ ┌──────┐   ┌──────────┐ ┌───────┐ ┌──────────┐  ┌────────┐│
    │ │Users │   │Enterprise│ │Groups │ │Conditional│  │Identity││
    │ │      │   │   Apps   │ │       │ │  Access   │  │Protect.││
    │ └──┬───┘   └────┬─────┘ └───┬───┘ └─────┬────┘  └───┬────┘│
    │    │            │           │            │           │     │
    │    └────────────┴───────────┴────────────┴───────────┘     │
    │                              │                              │
    │              ┌───────────────┼───────────────┐              │
    │              ▼               ▼               ▼              │
    │        ┌──────────┐   ┌──────────┐   ┌──────────────┐      │
    │        │  Azure   │   │Microsoft │   │  3rd-Party   │      │
    │        │Resources │   │   365    │   │  SaaS Apps   │      │
    │        └──────────┘   └──────────┘   └──────────────┘      │
    └─────────────────────────────────────────────────────────────┘
                              ▲
                              │  Entra Connect / Cloud Sync
                              │
                   ┌──────────┴──────────┐
                   │  ON-PREMISES AD DS  │
                   │  (Domain Controllers)│
                   └─────────────────────┘
```

### Key Architectural Facts
- **Multi-tenant SaaS service** — Microsoft operates the infrastructure globally.
- Built on a **distributed, partitioned directory store** (not LDAP-based like AD DS).
- **Geo-redundant** — data is replicated across multiple datacenters.
- **REST-based APIs** — accessed via **Microsoft Graph API** (`https://graph.microsoft.com`).
- **No domain controllers, no forests, no OUs, no GPOs** (fundamentally different from AD DS).

---

## 4. Authentication & Protocols

### Supported Protocols

| Protocol | Use Case | Notes |
|---|---|---|
| **OAuth 2.0** | Authorization for APIs | Token-based; used for delegated & app permissions |
| **OpenID Connect (OIDC)** | Authentication (Sign-in) | Built on top of OAuth 2.0; provides `id_token` |
| **SAML 2.0** | Enterprise SSO | XML-based; widely used by legacy SaaS apps |
| **WS-Federation** | Legacy federation | Used by older .NET apps, ADFS |
| **FIDO2 / WebAuthn** | Passwordless auth | Hardware security keys |
| **Certificate-based auth** | Smart cards / X.509 certs | CBA now natively supported |

> **Note:** Entra ID does **NOT** support **Kerberos** or **NTLM** natively. For those, you need **Azure AD DS** (Domain Services) or hybrid with on-prem AD.

### Token Types
```
┌──────────────────────────────────────────────────────────────────┐
│                     TOKEN FLOW (OAuth 2.0 / OIDC)               │
│                                                                  │
│  Client App ──► Entra ID (/authorize → /token) ──► Tokens:      │
│                                                                  │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────────┐       │
│  │  id_token   │  │ access_token │  │  refresh_token   │       │
│  │ (who am I)  │  │ (what can I  │  │ (get new tokens  │       │
│  │  JWT/OIDC   │  │  access)     │  │  without re-auth)│       │
│  └─────────────┘  └──────────────┘  └──────────────────┘       │
└──────────────────────────────────────────────────────────────────┘
```

### Authentication Methods
1. **Password** (legacy, least secure)
2. **MFA** — SMS, phone call, Microsoft Authenticator, OATH tokens
3. **Passwordless:**
   - Microsoft Authenticator (phone sign-in)
   - FIDO2 Security Keys (YubiKey, etc.)
   - Windows Hello for Business
   - Certificate-based authentication (CBA)
4. **Temporary Access Pass (TAP)** — time-limited passcode for onboarding

---

## 5. Identity Types

### 5.1 User Identities

```
┌───────────────────────────────────────────────────────┐
│                   USER TYPES                          │
├───────────────────┬───────────────────────────────────┤
│  MEMBER           │  GUEST (B2B)                      │
│  ─────────────    │  ──────────────                   │
│  • Cloud-only     │  • External user invited via      │
│    user created   │    email (B2B collaboration)      │
│    in Entra ID    │  • UserType = "Guest"             │
│  • Synced from    │  • Can use their own IdP          │
│    on-prem AD     │    (Google, SAML, etc.)           │
│  • Full member    │  • Limited directory permissions   │
│    permissions    │  • Governed by External Identity   │
│                   │    policies                        │
└───────────────────┴───────────────────────────────────┘
```

### 5.2 Workload Identities (Non-human)

| Type | Description | Credential Management |
|---|---|---|
| **Service Principal** | Identity for an app/service in your tenant | You manage client secrets/certificates |
| **Managed Identity (System-assigned)** | Auto-created, tied to one Azure resource | Azure manages everything — **no credentials** |
| **Managed Identity (User-assigned)** | You create it; can be shared across resources | Azure manages credentials |

> **Best Practice:** Always prefer **Managed Identities** over Service Principals to eliminate credential management.

### 5.3 Device Identities

| Method | Description |
|---|---|
| **Entra ID Registered** | Personal devices (BYOD); user signs in with personal account |
| **Entra ID Joined** | Corporate devices; no on-prem AD needed; cloud-native |
| **Hybrid Entra ID Joined** | Joined to both on-prem AD and Entra ID |

---

## 6. Licensing / Editions

| Feature | Free | P1 | P2 |
|---|:---:|:---:|:---:|
| User/group management | ✅ | ✅ | ✅ |
| SSO (unlimited apps) | ✅ | ✅ | ✅ |
| B2B collaboration | ✅ | ✅ | ✅ |
| MFA (Security Defaults) | ✅ | ✅ | ✅ |
| **Conditional Access** | ❌ | ✅ | ✅ |
| **MFA (granular CA policies)** | ❌ | ✅ | ✅ |
| **Self-Service Password Reset (full)** | ❌ | ✅ | ✅ |
| **Dynamic Groups** | ❌ | ✅ | ✅ |
| **Application Proxy** | ❌ | ✅ | ✅ |
| **Identity Protection (risk-based CA)** | ❌ | ❌ | ✅ |
| **PIM (Privileged Identity Mgmt)** | ❌ | ❌ | ✅ |
| **Access Reviews** | ❌ | ❌ | ✅ |
| **Entitlement Management** | ❌ | ❌ | ✅ |

> **P1** is included with Microsoft 365 E3; **P2** with Microsoft 365 E5.

---

## 7. Conditional Access

Conditional Access is the **Zero Trust policy engine** — the "brain" of Entra ID security.

```
┌────────────────────────────────────────────────────────────┐
│                   CONDITIONAL ACCESS POLICY                │
│                                                            │
│  IF (Signals/Conditions)          THEN (Controls)          │
│  ──────────────────────          ─────────────────         │
│                                                            │
│  • WHO: Users/Groups/Roles       • GRANT:                  │
│  • WHAT: Cloud apps/actions        - Allow                 │
│  • WHERE: Locations (IP/Country)   - Block                 │
│  • DEVICE: Platform/State          - Require MFA           │
│  • CLIENT: App type                - Require compliant     │
│  • RISK: User/Sign-in risk           device                │
│    (requires P2)                   - Require Hybrid Join   │
│                                    - Require password      │
│                                      change                │
│                                    - Require app           │
│                                      protection policy     │
│                                                            │
│                                  • SESSION:                 │
│                                    - Sign-in frequency     │
│                                    - Persistent browser    │
│                                    - MCAS inline controls  │
│                                    - Token lifetime        │
└────────────────────────────────────────────────────────────┘
```

### How CA Policies Evaluate
```
User signs in
      │
      ▼
┌──────────────┐     ┌──────────┐     ┌──────────┐
│  Policy A    │     │ Policy B │     │ Policy C │
│  Applies?    │     │ Applies? │     │ Applies? │
│  YES         │     │  YES     │     │  NO      │
│  Grant: MFA  │     │Grant:    │     │ (skip)   │
│              │     │Compliant │     │          │
│              │     │Device    │     │          │
└──────┬───────┘     └────┬─────┘     └──────────┘
       │                  │
       ▼                  ▼
  ┌───────────────────────────────┐
  │  ALL applicable grant         │
  │  controls must be satisfied   │
  │  (AND logic by default)       │
  │                               │
  │  Result: MFA + Compliant Dev  │
  └───────────────────────────────┘
```

### Common CA Policy Scenarios
1. **Require MFA for all admins** (always!)
2. **Block legacy authentication** (POP, IMAP, SMTP basic auth)
3. **Require compliant devices for Office 365**
4. **Block access from untrusted countries**
5. **Require MFA for risky sign-ins** (P2)
6. **Restrict access to specific apps from unmanaged devices** (session controls)

---

## 8. Multi-Factor Authentication (MFA)

### Methods (in order of security strength)
```
Most Secure ▲
            │  FIDO2 Security Key / Windows Hello
            │  Certificate-based auth (CBA)
            │  Microsoft Authenticator (passwordless)
            │  Microsoft Authenticator (push notification)
            │  OATH hardware tokens
            │  OATH software tokens (TOTP)
            │  SMS
            │  Voice call
Least Secure▼
```

### MFA Enforcement Approaches
| Approach | Description | Recommendation |
|---|---|---|
| **Security Defaults** | Free; forces MFA for all; blocks legacy auth | Good for small orgs without P1 |
| **Conditional Access** | Granular policies; requires P1 | **Recommended for enterprises** |
| **Per-user MFA** (legacy) | Enabled per-user in old portal | **Avoid** — use CA instead |

### Number Matching (Default since May 2023)
When approving an MFA push notification, the user must type the number displayed on the sign-in screen into the Authenticator app — prevents MFA fatigue attacks.

---

## 9. Privileged Identity Management (PIM)

> **Requires Entra ID P2**

PIM provides **Just-In-Time (JIT)** privileged access to reduce standing admin privileges.

```
┌────────────────────────────────────────────────────────────────┐
│                     PIM WORKFLOW                               │
│                                                                │
│  ┌──────────┐    ┌────────────┐    ┌────────────┐    ┌──────┐ │
│  │ ELIGIBLE │───►│ ACTIVATE   │───►│  ACTIVE    │───►│EXPIRE│ │
│  │ (Assigned│    │ (User      │    │ (Has the   │    │(Auto)│ │
│  │  but not │    │  requests; │    │  role for  │    │      │ │
│  │  active) │    │  may need  │    │  X hours)  │    │      │ │
│  │          │    │  approval) │    │            │    │      │ │
│  └──────────┘    └────────────┘    └────────────┘    └──────┘ │
│                                                                │
│  Features:                                                     │
│  • Time-bound activation (e.g., 4 hours max)                  │
│  • Approval workflow                                           │
│  • MFA required for activation                                 │
│  • Justification required                                      │
│  • Notification to admins                                      │
│  • Access reviews integration                                  │
│  • Audit trail                                                 │
└────────────────────────────────────────────────────────────────┘
```

### PIM Scope
| Scope | What you can manage with PIM |
|---|---|
| **Entra ID Roles** | Global Admin, User Admin, etc. |
| **Azure RBAC Roles** | Owner, Contributor, Reader on subscriptions/RGs/resources |
| **PIM for Groups** | Membership/ownership of privileged groups |

---

## 10. Identity Protection

> **Requires Entra ID P2**

Uses **machine learning** to detect identity-based risks in real time.

### Risk Types

| Risk | Level | Examples |
|---|---|---|
| **Sign-in Risk** | Real-time & Offline | Anonymous IP, atypical travel, malware-linked IP, unfamiliar sign-in properties, token anomaly, password spray |
| **User Risk** | Offline | Leaked credentials (dark web), anomalous user activity, suspicious API traffic |

### Risk Levels
- **Low** / **Medium** / **High**

### Integration with Conditional Access
```
Conditional Access Policy:
  IF sign-in risk = High
    THEN require MFA
  
  IF user risk = High
    THEN require password change + MFA
```

### Risk Remediation
```
┌───────────────────────────────────────────────────────┐
│              RISK REMEDIATION OPTIONS                  │
│                                                       │
│  Self-remediation (via CA):                           │
│    • Require MFA (proves identity)                    │
│    • Require secure password change                   │
│                                                       │
│  Admin remediation:                                   │
│    • Dismiss risk                                     │
│    • Confirm compromise → force password reset        │
│    • Block user                                       │
│                                                       │
│  Automated:                                           │
│    • Risk-based CA policies auto-remediate             │
└───────────────────────────────────────────────────────┘
```

---

## 11. Application Integration

### Application Model — Two Objects

```
┌────────────────────────────────┐
│  APP REGISTRATION              │
│  (Application Object)          │
│  ─────────────────             │
│  • Global definition           │
│  • Lives in HOME tenant        │
│  • Defines:                    │
│    - App ID (client ID)        │
│    - Redirect URIs             │
│    - API permissions           │
│    - Certificates/Secrets      │
│    - Token configuration       │
└──────────┬─────────────────────┘
           │ Creates (1:N relationship)
           ▼
┌────────────────────────────────┐
│  SERVICE PRINCIPAL             │
│  (Enterprise Application)     │
│  ─────────────────             │
│  • Local instance in a tenant  │
│  • One per tenant that uses    │
│    the app                     │
│  • Defines:                    │
│    - Who can access (users/    │
│      groups assignment)        │
│    - Conditional Access applies│
│    - SSO configuration         │
│    - Provisioning              │
└────────────────────────────────┘
```

### Permission Types
| Type | Description | Consent |
|---|---|---|
| **Delegated** | App acts on behalf of signed-in user | User or admin consent |
| **Application** | App acts as itself (daemon/service) | Admin consent only |

### Consent Framework
```
User signs in to app
       │
       ▼
Does app need permissions?
       │
   ┌───┴───┐
   YES     NO → Allow
   │
   ▼
Are permissions pre-consented (admin)?
   │
   ├── YES → Allow
   │
   └── NO → Is user consent allowed?
            │
            ├── YES → Show consent prompt
            │         (only for low-risk permissions)
            │
            └── NO → Admin consent workflow
                     (request sent to admins)
```

### Enterprise App Gallery
- **~6000+ pre-integrated SaaS apps** (Salesforce, ServiceNow, Zoom, etc.)
- Support for **SAML SSO**, **OIDC SSO**, **SCIM provisioning**

---

## 12. Hybrid Identity

### Sync Tools

| Tool | Description | Use Case |
|---|---|---|
| **Entra Connect (v2)** | Server-based sync engine | Full-featured; most common today |
| **Entra Cloud Sync** | Lightweight agent; cloud-managed | Multi-forest, simpler setup, HA |

### Authentication Methods in Hybrid

```
┌──────────────────────────────────────────────────────────────────┐
│                  HYBRID AUTH METHODS                              │
│                                                                  │
│  1. PASSWORD HASH SYNC (PHS) ★ Recommended                      │
│     ┌─────────┐  hash of hash   ┌──────────┐                    │
│     │ On-prem │ ──────────────► │ Entra ID │  Auth happens       │
│     │  AD DS  │                 │          │  in the CLOUD       │
│     └─────────┘                 └──────────┘                     │
│     • Simplest; most resilient                                   │
│     • Enables leaked credential detection                        │
│                                                                  │
│  2. PASS-THROUGH AUTH (PTA)                                      │
│     ┌─────────┐ ◄── validate ── ┌──────────┐                    │
│     │ On-prem │    password     │ Entra ID │  Auth validated     │
│     │  AD DS  │ ──────────────► │          │  ON-PREM            │
│     └─────────┘                 └──────────┘                     │
│     • Password never leaves on-prem                              │
│     • Requires PTA agent(s) on-prem                              │
│                                                                  │
│  3. FEDERATION (ADFS)                                            │
│     ┌─────────┐     ┌──────┐     ┌──────────┐                   │
│     │ On-prem │ ◄── │ ADFS │ ◄── │ Entra ID │                   │
│     │  AD DS  │     │      │     │(redirects│                   │
│     └─────────┘     └──────┘     │to ADFS)  │                   │
│     • Full control over auth flow └──────────┘                   │
│     • Complex; requires ADFS infrastructure                      │
│     • Being deprecated in favor of PHS + CA                      │
└──────────────────────────────────────────────────────────────────┘
```

### What Gets Synced?
- Users, Groups, Contacts
- **Passwords** (as hash of hash — with PHS)
- Device objects (for Hybrid Join)
- Attributes (configurable filtering by OU, domain, or attribute)

### Seamless SSO
- Works with PHS and PTA
- Users on domain-joined machines get SSO to cloud apps without prompts
- Uses Kerberos tickets behind the scenes

---

## 13. B2B & B2C

### Entra ID B2B (Business-to-Business)

```
┌────────────────────────────────────────────────────────────┐
│                    B2B COLLABORATION                       │
│                                                            │
│  Partner user (guest)                                      │
│  partner@fabrikam.com                                      │
│         │                                                  │
│         │ Invited to                                       │
│         ▼                                                  │
│  ┌──────────────────────┐                                  │
│  │ Contoso Tenant       │                                  │
│  │                      │                                  │
│  │ UserType: Guest      │                                  │
│  │ Auth: federated back │                                  │
│  │   to Fabrikam's IdP  │                                  │
│  │ Or: OTP, Google,     │                                  │
│  │   Microsoft account  │                                  │
│  └──────────────────────┘                                  │
│                                                            │
│  Supported Identity Providers for Guests:                  │
│  • Another Entra ID tenant                                 │
│  • Microsoft Account (MSA)                                 │
│  • Google Federation                                       │
│  • Facebook (B2C only)                                     │
│  • SAML/WS-Fed IdP (direct federation)                    │
│  • Email One-Time Passcode (OTP) — fallback               │
└────────────────────────────────────────────────────────────┘
```

### Cross-Tenant Access Settings
- **Inbound** — control how external orgs access YOUR resources
- **Outbound** — control how YOUR users access external orgs
- **Trust settings** — trust MFA/device claims from external tenants

### Entra External ID (B2C)

| Aspect | B2B | B2C |
|---|---|---|
| **Audience** | Partner employees | Consumers / customers |
| **Tenant** | Same tenant as employees | **Separate B2C tenant** |
| **Identity providers** | Entra ID, Google, SAML, OTP | Google, Facebook, Apple, local accounts, any OIDC/SAML |
| **Customization** | Limited | Full UI customization (user flows / custom policies) |
| **Scale** | Thousands | **Millions** of consumer identities |

> **New:** Microsoft is consolidating B2B and B2C into **Microsoft Entra External ID** with a unified platform.

---

## 14. Governance & Entitlement Management

### Identity Governance Features (P2)

```
┌─────────────────────────────────────────────────────────────┐
│                  IDENTITY GOVERNANCE                        │
│                                                             │
│  ┌─────────────────┐  ┌──────────────┐  ┌───────────────┐ │
│  │ ACCESS REVIEWS  │  │ ENTITLEMENT  │  │  LIFECYCLE    │ │
│  │                 │  │ MANAGEMENT   │  │  WORKFLOWS    │ │
│  │ • Periodic      │  │              │  │              │ │
│  │   review of     │  │ • Access     │  │ • Joiner     │ │
│  │   access        │  │   Packages   │  │ • Mover      │ │
│  │ • Auto-remove   │  │ • Catalogs   │  │ • Leaver     │ │
│  │   if not        │  │ • Approval   │  │ • Custom     │ │
│  │   approved      │  │   workflows  │  │   triggers   │ │
│  │ • For groups,   │  │ • Self-      │  │              │ │
│  │   apps, roles   │  │   service    │  │              │ │
│  └─────────────────┘  │   request    │  └───────────────┘ │
│                       └──────────────┘                     │
│                                                             │
│  ┌──────────────────────────────────────────┐              │
│  │ TERMS OF USE                              │              │
│  │ • Present T&C before accessing apps       │              │
│  │ • Track acceptance; require re-acceptance │              │
│  └──────────────────────────────────────────┘              │
└─────────────────────────────────────────────────────────────┘
```

### Entitlement Management — Access Packages
```
Access Package: "Marketing Team Access"
├── Resources:
│   ├── SharePoint Site: Marketing Portal (Member)
│   ├── Group: SG-Marketing-All (Member)
│   ├── App: Salesforce (Marketing Role)
│   └── Teams: Marketing Channel
├── Policies:
│   ├── Who can request: All employees
│   ├── Approval: Manager → Team Lead
│   ├── Duration: 180 days (renewable)
│   └── Access Review: Every 90 days
└── Catalog: Marketing
```

---

## 15. RBAC vs Entra ID Roles

This is a **critical distinction** that often causes confusion.

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  ┌─────────────────────┐      ┌────────────────────────┐    │
│  │  ENTRA ID ROLES     │      │   AZURE RBAC ROLES     │    │
│  │  (Directory Roles)  │      │   (Resource Roles)     │    │
│  │                     │      │                        │    │
│  │  Scope: ENTRA ID    │      │  Scope: AZURE          │    │
│  │  TENANT (directory) │      │  RESOURCES              │    │
│  │                     │      │  (Mgmt Group → Sub →   │    │
│  │  Examples:          │      │   RG → Resource)       │    │
│  │  • Global Admin     │      │                        │    │
│  │  • User Admin       │      │  Examples:             │    │
│  │  • App Admin        │      │  • Owner               │    │
│  │  • Security Admin   │      │  • Contributor         │    │
│  │  • Billing Admin    │      │  • Reader              │    │
│  │  • Exchange Admin   │      │  • Custom roles        │    │
│  │  • ~80+ built-in   │      │  • ~100+ built-in      │    │
│  │                     │      │                        │    │
│  │  Manage: Users,     │      │  Manage: VMs, Storage, │    │
│  │  Groups, Apps,      │      │  Networking, Databases,│    │
│  │  Policies, Licenses │      │  etc.                  │    │
│  └─────────────────────┘      └────────────────────────┘    │
│                                                              │
│  ⚠ Global Admin can "elevate" to get Azure RBAC             │
│    access (User Access Administrator at root scope)          │
└──────────────────────────────────────────────────────────────┘
```

### Custom Roles
Both Entra ID roles and Azure RBAC roles support **custom roles** (Entra ID custom roles require P1).

---

## 16. Security Best Practices

### The Golden Rules (from a Senior Cloud Engineer's perspective)

```
┌──────────────────────────────────────────────────────────────────┐
│              TOP SECURITY RECOMMENDATIONS                        │
│                                                                  │
│  1. 🔒 PROTECT GLOBAL ADMINS                                    │
│     • Maximum 2-4 Global Admins (never just 1)                  │
│     • At least 1 "break glass" account (excluded from CA)       │
│     • All admins use PIM (JIT activation)                       │
│     • Separate admin accounts from daily-use accounts            │
│                                                                  │
│  2. 🛡️  CONDITIONAL ACCESS BASELINE                              │
│     • Require MFA for ALL users                                  │
│     • Block legacy authentication                                │
│     • Require MFA for admins (always, no exceptions)            │
│     • Require compliant/managed devices for sensitive apps       │
│     • Block access from high-risk countries                      │
│                                                                  │
│  3. 🔑 GO PASSWORDLESS                                           │
│     • Deploy Microsoft Authenticator (phone sign-in)             │
│     • FIDO2 keys for admins                                      │
│     • Windows Hello for Business on managed devices              │
│                                                                  │
│  4. 📋 GOVERNANCE                                                │
│     • Regular access reviews (quarterly)                         │
│     • Enable Identity Protection risk policies                   │
│     • Monitor sign-in and audit logs                             │
│     • Use entitlement management for project-based access        │
│                                                                  │
│  5. 🔧 WORKLOAD IDENTITIES                                      │
│     • Use Managed Identities (not service principals w/ secrets)│
│     • Rotate secrets/certificates; set short expiry              │
│     • Use Workload Identity Federation for CI/CD (no secrets)   │
│                                                                  │
│  6. 🚫 LEAST PRIVILEGE                                           │
│     • Never use Global Admin for daily tasks                     │
│     • Use specific roles (User Admin, App Admin, etc.)          │
│     • Scope Azure RBAC to resource groups, not subscriptions     │
│                                                                  │
│  7. 📱 DEVICE TRUST                                              │
│     • Require Entra ID Join or Hybrid Join                       │
│     • Integrate Intune compliance                                │
│     • Use device-based CA policies                               │
└──────────────────────────────────────────────────────────────────┘
```

### Break Glass Accounts
```
Account: breakglass@contoso.onmicrosoft.com
├── Cloud-only (no sync dependency)
├── Global Admin role (permanent, not PIM)
├── EXCLUDED from ALL Conditional Access policies
├── Long, complex password stored in physical safe
├── No MFA (or hardware token in safe)
├── Sign-in monitored via Azure Monitor alerts
└── Tested regularly (quarterly)
```

---

## 17. Monitoring & Logging

### Log Types

| Log | Retention (Free) | Retention (P1/P2) | Description |
|---|---|---|---|
| **Sign-in logs** | 7 days | 30 days | Every authentication event |
| **Audit logs** | 7 days | 30 days | Directory changes (user created, role assigned, etc.) |
| **Provisioning logs** | 7 days | 30 days | SCIM provisioning events |
| **Risky sign-ins** | — | 30 days (P2) | Sign-ins flagged by Identity Protection |
| **Risky users** | — | 30 days (P2) | Users flagged as compromised |

### Long-Term Retention Strategy
```
┌──────────────┐     ┌─────────────────────┐     ┌────────────────┐
│  Entra ID    │────►│ Diagnostic Settings │────►│ Destinations:  │
│  Logs        │     │ (export)            │     │                │
└──────────────┘     └─────────────────────┘     │ • Log Analytics│
                                                  │   Workspace    │
                                                  │ • Storage Acct │
                                                  │ • Event Hub    │
                                                  │ • Sentinel     │
                                                  └────────────────┘
```

### Key Queries (KQL in Log Analytics)
```kusto
// Failed sign-ins in last 24 hours
SigninLogs
| where TimeGenerated > ago(24h)
| where ResultType != "0"
| summarize FailedCount = count() by UserPrincipalName, IPAddress, ResultDescription
| order by FailedCount desc

// Global Admin activations (PIM)
AuditLogs
| where OperationName == "Add member to role completed (PIM activation)"
| where TargetResources[0].displayName == "Global Administrator"
| project TimeGenerated, InitiatedBy.user.userPrincipalName, Result

// Risky sign-ins
AADRiskEvents
| where RiskLevel == "high"
| project TimeGenerated, UserPrincipalName, RiskEventType, RiskLevel, IPAddress
```

---

## 18. Entra ID vs Traditional AD DS

| Feature | Active Directory Domain Services (AD DS) | Microsoft Entra ID |
|---|---|---|
| **Type** | On-premises directory | Cloud-based IDaaS |
| **Protocol** | LDAP, Kerberos, NTLM | REST (Graph API), OAuth, OIDC, SAML |
| **Structure** | Forests → Domains → OUs | Flat (tenants; no OUs) |
| **Group Policy** | GPOs | Conditional Access + Intune |
| **Authentication** | Kerberos / NTLM | OAuth 2.0 / OIDC / SAML |
| **Device Management** | Domain Join + GPO | Entra Join + Intune/MDM |
| **Schema** | Extensible LDAP schema | Extension attributes / Graph |
| **DNS** | AD-integrated DNS | N/A |
| **Replication** | DC replication (intra-site/inter-site) | Microsoft-managed geo-replication |
| **HA** | Multiple DCs | Built-in SLA (99.99%) |
| **Admin tools** | ADUC, GPMC, PowerShell AD module | Azure Portal, Graph API, Az PowerShell |
| **Managed by** | You (infra, patching, backup) | Microsoft |

> **Key insight:** They are **not the same thing** and are not interchangeable. Many organizations use **both** together (hybrid).

---

## 19. Related Entra Products

Microsoft has expanded the **Entra** family beyond just "Entra ID":

```
┌─────────────────────────────────────────────────────────────────┐
│                    MICROSOFT ENTRA FAMILY                       │
│                                                                 │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────┐ │
│  │  Entra ID        │  │  Entra ID        │  │  Entra       │ │
│  │  (Core IAM)      │  │  Governance      │  │  External ID │ │
│  │                  │  │  (Access reviews, │  │  (B2B + B2C) │ │
│  │                  │  │  entitlements,    │  │              │ │
│  │                  │  │  lifecycle)       │  │              │ │
│  └──────────────────┘  └──────────────────┘  └──────────────┘ │
│                                                                 │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────┐ │
│  │  Entra           │  │  Entra           │  │  Entra       │ │
│  │  Permissions     │  │  Verified ID     │  │  Workload    │ │
│  │  Management      │  │  (Decentralized  │  │  Identities  │ │
│  │  (CIEM - multi-  │  │  identity /      │  │  (SPs, MIs,  │ │
│  │  cloud IGA for   │  │  verifiable      │  │  workload ID │ │
│  │  AWS, GCP, Azure)│  │  credentials)    │  │  federation) │ │
│  └──────────────────┘  └──────────────────┘  └──────────────┘ │
│                                                                 │
│  ┌──────────────────┐  ┌──────────────────┐                   │
│  │  Entra Internet  │  │  Entra Private   │   (Part of       │
│  │  Access          │  │  Access          │    Global Secure  │
│  │  (Secure Web     │  │  (Zero Trust     │    Access - SASE) │
│  │  Gateway / SWG)  │  │  Network Access  │                   │
│  │                  │  │  / ZTNA/VPN      │                   │
│  │                  │  │  replacement)    │                   │
│  └──────────────────┘  └──────────────────┘                   │
│                                                                 │
│  ┌──────────────────┐                                          │
│  │  Entra Domain    │  (Formerly Azure AD Domain Services)     │
│  │  Services        │  Managed AD DS in Azure (Kerberos,      │
│  │  (Azure AD DS)   │  NTLM, LDAP, Group Policy)              │
│  └──────────────────┘                                          │
└─────────────────────────────────────────────────────────────────┘
```

---

## 20. Real-World Architecture Diagram

```
                                    INTERNET
                                       │
                           ┌───────────┴───────────┐
                           │    Entra ID Tenant     │
                           │    contoso.com         │
                           │                        │
                           │  ┌──────────────────┐  │
                           │  │ Conditional      │  │
                           │  │ Access Policies  │  │
                           │  └────────┬─────────┘  │
                           │           │             │
          ┌────────────────┤           │             ├────────────────┐
          │                │           │             │                │
    ┌─────▼─────┐    ┌────▼──────┐ ┌──▼──────┐ ┌───▼────────┐ ┌───▼──────┐
    │Microsoft  │    │Azure     │ │SaaS    │ │Custom App │ │B2B       │
    │365        │    │Resources │ │Apps    │ │(App Svc)  │ │Partners  │
    │           │    │          │ │        │ │           │ │(Guests)  │
    │• Exchange │    │• VMs     │ │• SF    │ │• OIDC     │ │          │
    │• Teams    │    │• AKS     │ │• Snow  │ │• Managed  │ │• External│
    │• SharePt  │    │• SQL     │ │• Zoom  │ │  Identity │ │  IdPs    │
    └───────────┘    │• Storage │ │• Slack │ │• RBAC     │ └──────────┘
                     └──────────┘ └────────┘ └───────────┘
                           │
                           │ Entra Connect Sync / Cloud Sync
                           │
                    ┌──────▼──────────────────────────────┐
                    │         ON-PREMISES                  │
                    │                                      │
                    │  ┌──────────┐    ┌─────────────────┐│
                    │  │ AD DS    │    │ Line-of-Business ││
                    │  │ Domain   │    │ Apps (Kerberos)  ││
                    │  │Controller│    │ → App Proxy      ││
                    │  └──────────┘    └─────────────────┘│
                    │                                      │
                    │  ┌──────────────────────────────┐   │
                    │  │ Devices:                      │   │
                    │  │ • Hybrid Entra ID Joined      │   │
                    │  │ • Managed by Intune + GPO     │   │
                    │  └──────────────────────────────┘   │
                    └──────────────────────────────────────┘
```

---

## Quick Reference: PowerShell / CLI Commands

### Microsoft Graph PowerShell (Modern)
```powershell
# Install module
Install-Module Microsoft.Graph -Scope CurrentUser

# Connect
Connect-MgGraph -Scopes "User.Read.All", "Directory.Read.All"

# Get all users
Get-MgUser -All | Select DisplayName, UserPrincipalName, UserType

# Get a specific user
Get-MgUser -UserId "user@contoso.com"

# Get all groups
Get-MgGroup -All

# Get Conditional Access policies
Get-MgIdentityConditionalAccessPolicy

# Get directory roles and members
Get-MgDirectoryRole | ForEach-Object {
    $role = $_
    $members = Get-MgDirectoryRoleMember -DirectoryRoleId $role.Id
    [PSCustomObject]@{
        Role    = $role.DisplayName
        Members = ($members.AdditionalProperties.userPrincipalName -join ", ")
    }
}
```

### Azure CLI
```bash
# Sign in
az login

# List users
az ad user list --output table

# Create a user
az ad user create \
  --display-name "John Doe" \
  --user-principal-name john@contoso.com \
  --password "P@ssw0rd123" \
  --force-change-password-next-sign-in true

# List service principals
az ad sp list --all --output table

# List app registrations
az ad app list --all --output table
```

---

## Summary Mind Map

```
                         MICROSOFT ENTRA ID
                               │
        ┌──────────┬───────────┼───────────┬──────────────┐
        │          │           │           │              │
   IDENTITIES  AUTHENTICATION  ACCESS    GOVERNANCE   HYBRID
        │          │          CONTROL      │              │
   ┌────┤     ┌────┤           │      ┌────┤         ┌────┤
   │    │     │    │           │      │    │         │    │
 Users Groups OAuth Passwordless  Cond. PIM Access  Entra  Cloud
 SPs   Devices OIDC  MFA        Access    Reviews Connect Sync
 MIs         SAML  FIDO2      Policies  Entitle-       PHS
                   CBA         Risk-     ment          PTA
                               based    Lifecycle     ADFS
                               (P2)     Workflows    Seamless
                                                      SSO
```

---

## Key Takeaways for a Senior Cloud Engineer

1. **Entra ID is NOT "Active Directory in the cloud"** — it's a fundamentally different, cloud-native identity platform.
2. **Conditional Access is your most powerful security tool** — invest heavily in getting your policies right.
3. **PIM + Identity Protection (P2) are non-negotiable** for enterprise security.
4. **Managed Identities > Service Principals** — eliminate credential management wherever possible.
5. **Hybrid identity (PHS + Seamless SSO)** is the recommended path for most organizations migrating from on-prem.
6. **Monitor everything** — ship logs to Log Analytics / Sentinel and set up alerts for risky sign-ins and role activations.
7. **Think Zero Trust** — never trust, always verify. Every access decision should go through Conditional Access.
8. **Plan your tenant topology early** — single tenant for most orgs; multi-tenant only for specific regulatory/isolation needs.

---
