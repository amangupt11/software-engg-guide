# 🛡️ Security Engineering — Production Engineering Guide

> How to build and operate software that resists attack and protects data: security frameworks (NIST CSF 2.0, SSDF, ISO/IEC 27001, OWASP SAMM/ASVS), the Secure SDLC, threat modelling, the **OWASP Top 10:2025**, API and LLM security risks, identity and authentication (NIST SP 800-63-4, passkeys, OAuth/OIDC), authorisation, input/output handling, security headers, cryptography (including post-quantum), secrets, privacy and data protection (India DPDP Act, GDPR), software supply chain (SBOM, SLSA, Sigstore), container/cloud/CI-CD security, vulnerability management (CVSS v4, EPSS, KEV), security logging, incident response, disclosure, and security culture.
>
> Related: [01 Foundations §16](./01-engineering-foundations.md#16-security-fundamentals) · [02 Secure SDLC §10](./02-software-development-lifecycle.md#10-secure-sdlc--nist-ssdf) · [04 Security architecture §16](./04-software-architecture.md#16-security-architecture) · [06 Secure coding §21](./06-coding-standards.md#21-secure-coding-standards) · [08 Security testing §20.2](./08-testing-and-quality.md#202-security-testing) · [39 VPS hardening](./39-vps-enterprise-setup-guide.md)

---

## 📚 Table of Contents

1. [Principles](#1-principles)
2. [Frameworks and Standards](#2-frameworks-and-standards)
3. [Secure SDLC in Practice](#3-secure-sdlc-in-practice)
4. [Threat Modelling](#4-threat-modelling)
5. [OWASP Top 10:2025](#5-owasp-top-102025)
6. [API Security](#6-api-security)
7. [LLM and AI Application Security](#7-llm-and-ai-application-security)
8. [Identity and Authentication](#8-identity-and-authentication)
9. [Sessions and Tokens](#9-sessions-and-tokens)
10. [Authorisation](#10-authorisation)
11. [Input Handling, Injection, XSS, CSRF, SSRF](#11-input-handling-injection-xss-csrf-ssrf)
12. [Security Headers and Browser Security](#12-security-headers-and-browser-security)
13. [Cryptography](#13-cryptography)
14. [Secrets Management](#14-secrets-management)
15. [Data Protection and Privacy](#15-data-protection-and-privacy)
16. [Software Supply Chain Security](#16-software-supply-chain-security)
17. [Container and Kubernetes Security](#17-container-and-kubernetes-security)
18. [Cloud Security](#18-cloud-security)
19. [CI/CD Pipeline Security](#19-cicd-pipeline-security)
20. [Vulnerability Management](#20-vulnerability-management)
21. [Security Logging, Monitoring, and Alerting](#21-security-logging-monitoring-and-alerting)
22. [Incident Response](#22-incident-response)
23. [Vulnerability Disclosure and Bug Bounties](#23-vulnerability-disclosure-and-bug-bounties)
24. [Security Culture and Programme](#24-security-culture-and-programme)
25. [Checklists](#25-checklists)
26. [References](#26-references)

---

## 1. Principles

| Principle | Meaning in practice |
|---|---|
| **Secure by design and by default** | Safe configuration out of the box; insecure options require explicit opt-in |
| **Least privilege** | Users, services, CI jobs, and humans get only the access they need, for as long as they need it |
| **Defense in depth** | Independent layers: edge → network → identity → app → data → monitoring |
| **Zero trust** (NIST SP 800-207) | No implicit trust from network location; authenticate and authorise every request |
| **Minimise attack surface** | Remove unused features, endpoints, ports, packages, permissions |
| **Fail securely** | Errors deny access; never "fail open" (now an explicit OWASP 2025 category) |
| **Assume breach** | Segmentation, detection, and tested recovery |
| **Don't trust the client** | All validation and authorisation enforced server-side |
| **Keep security simple** | Complex controls are misconfigured; prefer platform defaults and paved roads |
| **Security is everyone's job** | Developers own the security of their code; specialists enable and verify |

---

## 2. Frameworks and Standards

| Framework | Purpose | Status / version (verify before audits) |
|---|---|---|
| **NIST Cybersecurity Framework (CSF)** | Organisation-wide cyber risk programme; six functions: **Govern, Identify, Protect, Detect, Respond, Recover** | 2.0 (2024) |
| **NIST SP 800-218 SSDF** | Secure software development practices (PO, PS, PW, RV) | v1.1 final; v1.2 (Rev. 1) draft Dec 2025 |
| **NIST SP 800-218A** | SSDF profile for generative AI and dual-use foundation models | Final (2024) |
| **NIST SP 800-63-4** | Digital identity: proofing, authentication, federation | **Final, July 2025** (supersedes 800-63-3) |
| **NIST SP 800-207** | Zero Trust Architecture | Final (2020) |
| **NIST SP 800-61** | Incident response | Rev. 3 (aligned with CSF 2.0) |
| **ISO/IEC 27001 / 27002** | Information security management system and controls | 2022 editions |
| **OWASP SAMM** | Software assurance maturity model (measure/improve programme) | v2 |
| **OWASP ASVS** | Application security verification requirements (Levels 1–3) | 5.0 |
| **OWASP Top 10** | Awareness of the most critical web app risks | **2025** |
| **OWASP API Security Top 10** | API-specific risks | 2023 |
| **OWASP Top 10 for LLM Applications** | GenAI app risks | 2025 |
| **CIS Controls** | Prioritised safeguards | v8.1 |
| **CIS Benchmarks** | Hardening baselines for OS, cloud, Kubernetes, DBs | Per product |
| **SLSA** | Supply-chain integrity levels for builds | v1.x |
| **PCI DSS** | Cardholder data security | v4.0.1 |
| **SOC 2** | Trust services criteria attestation | AICPA |
| **EU Cyber Resilience Act** | Security obligations for products with digital elements in the EU | Obligations phasing in |

---

## 3. Secure SDLC in Practice

Mapped to the SDLC phases in chapter 02 and SSDF groups.

| Phase | Activities | Owner | Gate |
|---|---|---|---|
| 0 Discovery | Data classification, regulatory scoping, initial risk | PM + Security | Classification recorded |
| 1 Requirements | Security & privacy requirements (ASVS level), abuse cases | PM + Security champion | Requirements reviewed |
| 2 Design | Threat model, security architecture review, privacy review (DPIA when required) | Tech lead + Security | Threat model approved; high risks have mitigations |
| 4 Build | Secure coding, SAST, SCA, secret scanning, secure code review | Developers | PR gates (chapter 06/07) |
| 5 Verify | DAST, API fuzzing, authorisation tests, IaC/container scans, pentest (risk-based) | QA/SDET + Security | No unaccepted Critical/High |
| 6 Release | Signed artifacts, SBOM, provenance, change approval | Platform | Release gate |
| 7 Operate | WAF/RASP as needed, monitoring/alerting, vulnerability intake | SRE + Security | Alerts routed; on-call |
| 8 Maintain | Patch SLAs, dependency upgrades, RCA for vulnerabilities | Developers | SLA tracking |

### ASVS levels — choosing a target

| Level | For | Notes |
|---|---|---|
| **L1** | All applications (baseline) | Largely verifiable via testing |
| **L2** | Applications handling sensitive data (most business apps) | Recommended default |
| **L3** | Critical applications (high-value transactions, health, critical infrastructure) | Requires deeper verification |

---

## 4. Threat Modelling

> "What are we working on? What can go wrong? What are we going to do about it? Did we do a good job?" — the Threat Modeling Manifesto's four key questions.

### 4.1 Process

```text
1. Model the system: data-flow diagram (DFD) with processes, data stores, external entities, data flows, TRUST BOUNDARIES
2. Identify threats: STRIDE per element/flow crossing a boundary (+ LINDDUN for privacy)
3. Rate: likelihood × impact (or DREAD-style / CVSS-like scoring)
4. Mitigate: control per threat (or accept/transfer with sign-off)
5. Validate: tests for mitigations; revisit when architecture changes
```

### 4.2 STRIDE quick reference

| Threat | Property | Typical mitigations |
|---|---|---|
| **S**poofing | Authentication | MFA/passkeys, mTLS, signed tokens |
| **T**ampering | Integrity | TLS, signatures/HMAC, DB constraints, immutable logs |
| **R**epudiation | Non-repudiation | Audit logs with user/time/action, tamper-evident storage |
| **I**nformation disclosure | Confidentiality | Encryption, authorisation, minimisation, redaction |
| **D**enial of service | Availability | Rate limits, quotas, autoscaling, timeouts, CDN/WAF |
| **E**levation of privilege | Authorisation | Least privilege, server-side checks, sandboxing |

### 4.3 Threat model template

```markdown
# Threat Model — Payment Retry (v1, 2026-10-02)
Scope: checkout UI → Orders API → Payments module → PSP; webhook callbacks from PSP
Assets: payment tokens, order data, customer PII, PSP API keys
Trust boundaries: Internet ↔ edge; edge ↔ API; API ↔ PSP; PSP webhooks ↔ API

| ID | Element/flow | STRIDE | Threat | Likelihood | Impact | Mitigation | Status | Test |
|---|---|---|---|---|---|---|---|---|
| T1 | PSP webhook | S | Forged callback marks order paid | M | Critical | Verify HMAC signature + timestamp; allow-list PSP IPs where offered; idempotent processing | Done | TC-SEC-11 |
| T2 | Retry endpoint | E/I | IDOR: retry another user's order | M | High | Scope by authenticated customer + tenant | Done | TC-SEC-12 |
| T3 | Retry endpoint | D | Card-testing abuse via retries | H | High | Rate limit per user/IP/card BIN; CAPTCHA after N failures; max 3 attempts | Done | TC-SEC-13 |
| T4 | Logs | I | PAN/UPI IDs logged | M | High | Redaction middleware; log allow-list | Done | TC-SEC-14 |
| T5 | Late callback | T | Duplicate charge after timeout | M | Critical | Idempotency key per attempt; auto-refund late approvals | In progress | TC-PAY-124 |
Residual risks & sign-off:
```

Tools: OWASP Threat Dragon, Microsoft Threat Modeling Tool, threagile (threat model as code), IriusRisk; or diagrams-as-code + Markdown tables.

### 4.4 Privacy threat modelling (LINDDUN)

**L**inkability · **I**dentifiability · **N**on-repudiation (as a privacy harm) · **D**etectability · **D**isclosure of information · **U**nawareness · **N**on-compliance — https://linddun.org/

---

## 5. OWASP Top 10:2025

The 2025 edition (released late 2025, the eighth edition) introduced two new categories — **Software Supply Chain Failures** and **Mishandling of Exceptional Conditions** — folded SSRF into Broken Access Control, moved Security Misconfiguration up to #2, and renamed the logging category to emphasise **alerting**.

| # | Category | What goes wrong | Key mitigations |
|---|---|---|---|
| **A01** | **Broken Access Control** (now includes **SSRF**) | IDOR/BOLA, missing function-level checks, privilege escalation, CORS misconfig, forced browsing, SSRF to internal services | Deny by default; server-side checks per resource and action; tenant scoping; centralised policy; outbound allow-lists for server-side fetches; automated authorisation tests |
| **A02** | **Security Misconfiguration** | Default credentials, verbose errors, open storage buckets, unnecessary features, missing hardening/headers | Hardened, automated, identical environments (IaC); CIS benchmarks; config scanning; minimal images; security headers |
| **A03** | **Software Supply Chain Failures** (expands "Vulnerable and Outdated Components") | Vulnerable/malicious dependencies, compromised build systems or distribution, typosquatting | SCA, lockfiles, pinned versions/digests, SBOM, signed artifacts, provenance (SLSA), hardened CI, vetted registries (§16, §19) |
| **A04** | **Cryptographic Failures** | Weak/missing encryption, poor key management, plaintext sensitive data | TLS 1.2+ (prefer 1.3), vetted libraries, KMS-managed keys, strong password hashing (§13) |
| **A05** | **Injection** (incl. XSS) | SQL/NoSQL/OS/LDAP injection, XSS, template injection | Parameterised queries, safe APIs, context-aware output encoding, CSP, input validation (§11) |
| **A06** | **Insecure Design** | Missing controls by design (no rate limits, no abuse-case thinking) | Threat modelling, secure design patterns, abuse cases, reference architectures |
| **A07** | **Authentication Failures** | Credential stuffing, weak passwords, broken session management, missing MFA | Phishing-resistant MFA/passkeys, breached-password checks, rate limiting, secure sessions (§8–9) |
| **A08** | **Software or Data Integrity Failures** | Unsigned updates, insecure deserialisation, untrusted CI/CD inputs | Signatures, integrity checks, safe deserialisation, protected pipelines |
| **A09** | **Security Logging & Alerting Failures** | Attacks not logged or not alerted on | Log security events, centralise, **alert** on them, test detection (§21) |
| **A10** | **Mishandling of Exceptional Conditions** *(new)* | Improper error handling, logic errors, failing open, leaking details in errors | Fail closed; consistent error handling; resource cleanup; fuzzing; no sensitive detail in errors (chapter 05 §12) |

Official: https://owasp.org/Top10/

> The Top 10 is an **awareness** document, not a complete standard. Use **ASVS** as the verification standard and the **Cheat Sheet Series** for implementation guidance.

---

## 6. API Security

### 6.1 OWASP API Security Top 10 (2023)

| # | Risk | Mitigation |
|---|---|---|
| API1 | Broken Object Level Authorization (BOLA) | Check ownership/tenant for every object ID on every request |
| API2 | Broken Authentication | Standard protocols (OIDC/OAuth), rate limits on auth endpoints, token validation |
| API3 | Broken Object Property Level Authorization | Explicit response DTOs (no mass exposure); allow-list writable fields (no mass assignment) |
| API4 | Unrestricted Resource Consumption | Rate limits, quotas, pagination caps, payload size limits, timeouts |
| API5 | Broken Function Level Authorization | Role/permission checks per operation, especially admin endpoints |
| API6 | Unrestricted Access to Sensitive Business Flows | Anti-automation (rate limits, device fingerprinting, CAPTCHA) for flows like signup, checkout, ticket buying |
| API7 | Server Side Request Forgery | Allow-list destinations; block internal ranges and cloud metadata endpoints |
| API8 | Security Misconfiguration | Hardened defaults, CORS allow-lists, no verbose errors |
| API9 | Improper Inventory Management | API catalogue, versioning, retire old/shadow APIs |
| API10 | Unsafe Consumption of APIs | Validate and sanitise third-party API responses; TLS; timeouts |

https://owasp.org/API-Security/

### 6.2 API security baseline

```text
🔴 TLS everywhere; HSTS on public endpoints
🔴 Authenticate every endpoint unless explicitly public (documented)
🔴 Authorise every object and function (BOLA/BFLA tests in CI)
🔴 Validate request schemas (OpenAPI-driven validation); reject unknown fields on writes
🔴 Rate limiting + quotas per client/tenant; 429 with Retry-After
🔴 Explicit response models — never serialise entities directly
🟠 API gateway: auth, rate limits, request size limits, logging
🟠 Webhooks: HMAC signatures with timestamp (replay protection), idempotent handlers
🟠 Inventory: every API in the catalogue with owner, version, and data classification
```

Webhook signature verification (Node.js):

```typescript
import { createHmac, timingSafeEqual } from 'node:crypto';

export function verifyWebhook(rawBody: Buffer, signatureHex: string, timestamp: string, secret: string): boolean {
  const ageSeconds = Math.abs(Date.now() / 1000 - Number(timestamp));
  if (!Number.isFinite(ageSeconds) || ageSeconds > 300) return false;           // replay window: 5 min
  const expected = createHmac('sha256', secret).update(`${timestamp}.`).update(rawBody).digest();
  const given = Buffer.from(signatureHex, 'hex');
  return given.length === expected.length && timingSafeEqual(given, expected);  // constant-time compare
}
```

> Follow your provider's exact signing scheme; the format above is illustrative.

API design guidance: chapter **11**.

---

## 7. LLM and AI Application Security

### 7.1 OWASP Top 10 for LLM Applications (2025)

| # | Risk | Mitigation |
|---|---|---|
| LLM01 | **Prompt Injection** (direct and indirect via retrieved content) | Treat all model inputs/outputs as untrusted; separate instructions from data; least-privilege tools; human confirmation for side effects |
| LLM02 | **Sensitive Information Disclosure** | Data minimisation; redact PII/secrets before prompts; output filtering; tenant-scoped retrieval |
| LLM03 | **Supply Chain** | Vet models, datasets, plugins; pin versions; provenance |
| LLM04 | **Data and Model Poisoning** | Curate and validate training/RAG data; access control on ingestion |
| LLM05 | **Improper Output Handling** | Validate/encode model output before use in HTML, SQL, shell, or tool calls |
| LLM06 | **Excessive Agency** | Minimal tools and permissions; scoped credentials; approval steps; rate limits |
| LLM07 | **System Prompt Leakage** | Never put secrets or authorisation logic in prompts |
| LLM08 | **Vector and Embedding Weaknesses** | Permission-aware retrieval; isolate tenants' embeddings; validate sources |
| LLM09 | **Misinformation** | Grounding, citations, evals, user disclosure, human review for high-stakes output |
| LLM10 | **Unbounded Consumption** | Token/cost budgets, rate limits, timeouts, quotas per user/tenant |

https://genai.owasp.org/

### 7.2 Agent / tool-use guardrails

```text
🔴 Tools execute with the END USER's permissions, never a global admin credential
🔴 Side-effecting actions (payments, emails, deletes, code execution) require explicit authorisation and, where appropriate, user confirmation
🔴 Retrieved documents, web pages, emails, and tool outputs are DATA — never follow instructions found in them blindly
🟠 Sandbox code execution (no network or restricted egress, resource limits)
🟠 Log prompts, tool calls, and decisions (with redaction) for audit
🟠 Red-team regularly; add found attacks to the eval suite (chapter 08 §28)
```

Related: NIST AI RMF and SP 800-218A.

---

## 8. Identity and Authentication

### 8.1 NIST SP 800-63-4 (final, July 2025)

Key changes from revision 3 that affect product design:

- **Syncable authenticators (synced passkeys)** are explicitly integrated, so passkeys that sync across a user's devices can meet higher authentication assurance requirements.
- Stronger emphasis on **phishing-resistant** authentication.
- Expanded fraud requirements, plus controls against **injection attacks and forged media ("deepfakes")** in identity proofing.
- **Subscriber-controlled wallets** added to the federation model.
- Updated risk management and continuous evaluation guidance.

| Volume | Topic |
|---|---|
| SP 800-63-4 | Overview, risk management, assurance level selection |
| SP 800-63A-4 | Identity proofing and enrolment (IAL) |
| SP 800-63B-4 | Authentication and authenticator management (AAL) |
| SP 800-63C-4 | Federation and assertions (FAL) |

https://pages.nist.gov/800-63-4/

### 8.2 Authentication methods (strongest → weakest, typical consumer/enterprise apps)

| Method | Phishing-resistant | Notes |
|---|:---:|---|
| **Passkeys / WebAuthn (FIDO2)** — device-bound or synced | ✅ | Recommended primary method |
| Hardware security keys | ✅ | Admins, high-risk users |
| Platform SSO (enterprise IdP with phishing-resistant MFA) | ✅ (if configured) | Workforce apps |
| Authenticator app TOTP / push with number matching | ❌ | Acceptable second factor; push fatigue risk without number matching |
| SMS / voice OTP | ❌ | Weakest; SIM-swap risk — fallback only |
| Password only | ❌ | Not acceptable for sensitive applications |

### 8.3 Password policy (aligned with NIST 800-63B guidance)

```text
🔴 Minimum length 8 (15 when password is the only factor in many enterprise contexts); allow long passphrases (≥ 64 chars)
🔴 Check new passwords against breached/common password lists
🔴 No composition rules (forced symbols/uppercase) and no periodic forced rotation — rotate on evidence of compromise
🔴 Allow paste and password managers; allow all printable characters incl. spaces and Unicode
🔴 Rate-limit and monitor failed attempts (throttling, not permanent lockout that enables DoS)
🔴 Hash with a memory-hard function (§13.4); never store reversibly
🟠 Offer passkeys and MFA; nudge users to enrol
```

### 8.4 Federation and SSO

| Protocol | Use |
|---|---|
| **OpenID Connect (OIDC)** | User authentication for web/mobile apps (on top of OAuth 2.0) |
| **OAuth 2.0** (+ security BCP **RFC 9700**, Jan 2025) | Delegated authorisation for APIs |
| **SAML 2.0** | Enterprise SSO (legacy but ubiquitous) |
| **SCIM 2.0** | User provisioning/deprovisioning from enterprise IdPs |

OAuth/OIDC rules (from RFC 9700 and OAuth 2.1 drafts):

```text
🔴 Authorization Code flow with PKCE for all clients (public and confidential)
🔴 No implicit flow; no resource owner password credentials flow
🔴 Exact redirect URI matching
🔴 Validate tokens fully: signature, issuer, audience, expiry, nonce (OIDC)
🟠 Sender-constrained tokens (mTLS or DPoP, RFC 9449) for high-value APIs
🟠 Short-lived access tokens; refresh token rotation with reuse detection
🟠 Use a proven IdP/library — do not implement OAuth servers yourself
```

---

## 9. Sessions and Tokens

### 9.1 Cookie-based sessions (web)

```http
Set-Cookie: __Host-session=<opaque-random-256-bit>; Path=/; Secure; HttpOnly; SameSite=Lax; Max-Age=28800
```

| Rule | Status |
|---|:---:|
| Opaque, random session IDs (≥ 128 bits entropy) generated by the framework | 🔴 |
| `Secure`, `HttpOnly`, `SameSite=Lax` (or `Strict`); `__Host-` prefix where possible | 🔴 |
| Regenerate session ID on login and privilege change | 🔴 |
| Idle timeout and absolute timeout (shorter for admin) | 🔴 |
| Server-side invalidation on logout and password change | 🔴 |
| Show active sessions/devices; allow revocation | 🟠 |

### 9.2 JWT pitfalls (see RFC 8725 — JWT Best Current Practices)

```text
🔴 Pin accepted algorithms; never accept "alg": "none"; don't let the token choose the key type
🔴 Validate iss, aud, exp, nbf; keep clock skew small
🔴 Don't put sensitive data in JWT payloads (they're only base64url-encoded unless encrypted)
🔴 Short expiry; plan revocation (short TTL + refresh, or introspection/deny-lists for high-risk)
🟠 Prefer opaque session cookies for first-party browser apps; JWTs for service-to-service/API access tokens
🟠 Rotate signing keys; publish via JWKS; support key IDs (kid)
```

### 9.3 Browser token storage

Avoid storing long-lived tokens in `localStorage` (readable by any XSS). For SPAs, prefer the **Backend-for-Frontend (BFF)** pattern: the BFF holds tokens server-side and the browser gets an `HttpOnly` session cookie.

---

## 10. Authorisation

### 10.1 Models

| Model | Decision based on | Example | Tools |
|---|---|---|---|
| **RBAC** | User's roles | Admin, Editor, Viewer | Framework roles, IdP groups |
| **ABAC** | Attributes of user, resource, action, environment | "Managers can approve refunds < ₹10 000 in their region during business hours" | OPA/Rego, AWS Cedar, XACML-style engines |
| **ReBAC** | Relationships between subjects and objects (Google Zanzibar model) | "User can edit doc if editor of the folder that contains it" | OpenFGA, SpiceDB, Permify |
| **ACLs** | Per-object permission lists | File sharing | DB tables |

> Most products start with RBAC + ownership/tenant checks and add ABAC/ReBAC as sharing rules grow.

### 10.2 Implementation rules

```text
🔴 Deny by default; explicit allow
🔴 Enforce on the server for EVERY request — UI hiding is not authorisation
🔴 Scope every data access by tenant AND owner/permission (prevents IDOR/BOLA)
🔴 Centralise policy logic (middleware, policy engine, or a single authz module) — no scattered ad-hoc checks
🔴 Re-check authorisation for background jobs acting on behalf of users
🟠 Use row-level security (e.g. PostgreSQL RLS) as defense-in-depth for multi-tenancy
🟠 Log authorisation denials (signals attacks and bugs)
🟠 Automated authz test matrix: role × tenant × resource × action
```

### 10.3 Policy-as-code example (OPA / Rego)

```rego
package orders.authz

import rego.v1

default allow := false

allow if {
    input.action == "read"
    input.resource.tenant_id == input.subject.tenant_id
    input.resource.customer_id == input.subject.user_id
}

allow if {
    input.action in {"read", "refund"}
    input.resource.tenant_id == input.subject.tenant_id
    "support_agent" in input.subject.roles
    input.resource.amount_minor <= 1000000
}
```

### 10.4 PostgreSQL row-level security (defense in depth)

```sql
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON orders
  USING (tenant_id = current_setting('app.tenant_id')::uuid);
-- App sets per transaction: SET LOCAL app.tenant_id = '<tenant-uuid>';
-- Ensure the app role is not the table owner / does not have BYPASSRLS.
```

---

## 11. Input Handling, Injection, XSS, CSRF, SSRF

### 11.1 Input validation

```text
- Validate at every trust boundary: type, length, range, format, allow-listed values
- Parse into typed objects (chapter 05 §11.3); reject, don't "sanitise and continue", for structural errors
- Canonicalise before validating (Unicode normalisation, path canonicalisation)
- File uploads: allow-list types (check content, not just extension), size limits, store outside web root
  or in object storage, generate new names, malware scan where required, serve with Content-Disposition
```

### 11.2 Injection prevention

| Sink | Safe approach |
|---|---|
| SQL | Parameterised queries / ORM bindings; never concatenate; least-privilege DB user |
| NoSQL | Typed query builders; reject operator injection (`$where`, `$ne` from user input) |
| OS commands | Avoid shells; use APIs; if unavoidable, pass argument arrays with allow-listed values |
| LDAP / XPath | Library escaping functions |
| Templates (SSTI) | Never render user-controlled templates |
| Logs | Encode/strip CR/LF to prevent log forging |
| Headers | Reject CR/LF in header values |

### 11.3 XSS prevention

```text
🔴 Use frameworks with auto-escaping (React, Angular, Vue, Razor, Thymeleaf, Jinja2 autoescape)
🔴 Never insert untrusted data via innerHTML / dangerouslySetInnerHTML / v-html / bypassSecurityTrust*
🔴 If rich HTML is required, sanitise with a vetted library (e.g. DOMPurify) on output
🔴 Content-Security-Policy with nonces/hashes; no 'unsafe-inline' scripts (§12)
🟠 Trusted Types in supporting browsers for DOM XSS hardening
```

### 11.4 CSRF

```text
- SameSite=Lax/Strict cookies (baseline)
- Anti-CSRF tokens (synchronizer or double-submit) for cookie-authenticated state-changing requests
- Verify Origin/Referer (or Fetch Metadata headers: Sec-Fetch-Site) for state-changing requests
- Never change state on GET
```

### 11.5 SSRF (now within A01 Broken Access Control)

```text
🔴 Allow-list destination hosts/schemes/ports for any server-side fetch of user-supplied URLs
🔴 Resolve DNS and block private, loopback, link-local ranges (incl. 169.254.169.254 cloud metadata) — re-check after redirects
🔴 Enforce IMDSv2 / metadata protections on cloud instances
🟠 Route outbound fetches through an egress proxy with policy
```

---

## 12. Security Headers and Browser Security

| Header | Recommended value (adapt to app) | Purpose |
|---|---|---|
| `Strict-Transport-Security` | `max-age=63072000; includeSubDomains; preload` | Force HTTPS (only after confirming all subdomains support HTTPS) |
| `Content-Security-Policy` | `default-src 'self'; script-src 'self' 'nonce-{random}' 'strict-dynamic'; object-src 'none'; base-uri 'none'; frame-ancestors 'none'` | Mitigate XSS, clickjacking |
| `X-Content-Type-Options` | `nosniff` | Disable MIME sniffing |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | Limit referrer leakage |
| `Permissions-Policy` | `camera=(), microphone=(), geolocation=()` | Disable unused powerful features |
| `Cross-Origin-Opener-Policy` | `same-origin` | Isolate browsing context |
| `Cross-Origin-Resource-Policy` | `same-origin` (or `same-site`) | Prevent cross-origin reads of resources |
| `Cache-Control` | `no-store` on sensitive responses | Prevent caching of private data |

`X-Frame-Options: DENY` remains a fallback for old browsers; CSP `frame-ancestors` supersedes it. `X-XSS-Protection` is obsolete — don't rely on it.

### Nginx snippet

```nginx
add_header Strict-Transport-Security "max-age=63072000; includeSubDomains" always;
add_header X-Content-Type-Options "nosniff" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Permissions-Policy "camera=(), microphone=(), geolocation=()" always;
add_header Cross-Origin-Opener-Policy "same-origin" always;
# CSP is best set by the application (to include per-request nonces)
```

### CORS rules

```text
🔴 Never reflect arbitrary Origin with Access-Control-Allow-Credentials: true
🔴 Explicit allow-list of origins; no wildcard with credentials
🟠 Limit allowed methods/headers; short preflight max-age during rollout
```

Validate with: Mozilla HTTP Observatory, securityheaders-style scanners, ZAP baseline. Server-level config: **39-vps-enterprise-setup-guide.md**.

---

## 13. Cryptography

### 13.1 Rules

```text
🔴 Never invent algorithms or protocols; use high-level, vetted libraries (libsodium, Tink, platform crypto APIs, JCA with care)
🔴 Use authenticated encryption (AES-256-GCM or ChaCha20-Poly1305); never ECB; never unauthenticated CBC
🔴 Unique nonces/IVs per encryption with GCM (nonce reuse is catastrophic)
🔴 Keys in a KMS/HSM or secret manager — never in code, config files, or images
🔴 Cryptographically secure randomness only (crypto.randomBytes / SecureRandom / secrets module)
🟠 Envelope encryption: data keys encrypted by KMS master keys; rotate
🟠 Crypto agility: abstract algorithms so they can be replaced (post-quantum migration)
```

### 13.2 Algorithm choices (current guidance)

| Purpose | Use | Avoid |
|---|---|---|
| Symmetric encryption | AES-256-GCM, ChaCha20-Poly1305 | DES/3DES, RC4, AES-ECB |
| Hashing (integrity) | SHA-256/384/512, SHA-3, BLAKE2/BLAKE3 | MD5, SHA-1 for security purposes |
| MAC | HMAC-SHA-256 | Plain hash of secret+message |
| Signatures | Ed25519, ECDSA P-256, RSA-PSS ≥ 3072 | RSA < 2048, DSA |
| Key exchange | X25519, ECDHE P-256 (+ hybrid PQC where supported) | Static RSA key exchange |
| Password hashing | **Argon2id**, scrypt, bcrypt (legacy), PBKDF2 (FIPS contexts) | Fast hashes (SHA-*), unsalted hashes |
| TLS | TLS 1.3 (preferred), TLS 1.2 with modern AEAD cipher suites | TLS 1.0/1.1, SSL |

### 13.3 TLS configuration

Use the **Mozilla SSL Configuration Generator** (intermediate or modern profile) for servers and load balancers, automate certificates (ACME/Let's Encrypt or managed certificates), and monitor expiry. https://ssl-config.mozilla.org/

### 13.4 Password hashing parameters

OWASP's Password Storage Cheat Sheet recommends **Argon2id** with a minimum configuration such as **19 MiB memory, 2 iterations, 1 degree of parallelism** (or equivalent alternatives listed there), bcrypt with a work factor of at least 10 for legacy systems, and PBKDF2-HMAC-SHA256 with a high iteration count when FIPS compliance is required. Re-check the cheat sheet for current figures and tune to your hardware.

```python
from argon2 import PasswordHasher
ph = PasswordHasher(time_cost=2, memory_cost=19 * 1024, parallelism=1)   # memory_cost in KiB
hash_ = ph.hash(password)
ph.verify(hash_, candidate)              # raises on mismatch
if ph.check_needs_rehash(hash_): ...     # upgrade parameters transparently at login
```

### 13.5 Post-quantum cryptography (PQC)

NIST published its first PQC standards in **August 2024**: **FIPS 203 (ML-KEM)** for key encapsulation, **FIPS 204 (ML-DSA)** and **FIPS 205 (SLH-DSA)** for signatures. "Harvest now, decrypt later" makes long-lived confidential data a priority.

```text
1. Inventory cryptography (where, which algorithms, key sizes, data lifetimes) — a "crypto bill of materials"
2. Prefer platforms/libraries that support hybrid key exchange (e.g. X25519 + ML-KEM in TLS) as they become available
3. Build crypto agility into designs
4. Track your cloud provider's, OS's, and browser's PQC roadmap; follow national guidance for migration timelines
```

---

## 14. Secrets Management

| Rule | Status |
|---|:---:|
| Secrets live in a secret manager (HashiCorp Vault/OpenBao, AWS Secrets Manager, Azure Key Vault, GCP Secret Manager, Doppler, 1Password Secrets Automation) | 🔴 |
| Applications fetch at runtime or via platform injection (CSI driver, env from secret store) — not baked into images | 🔴 |
| Prefer **workload identity / OIDC federation** over static credentials (cloud IAM roles for pods/VMs, CI OIDC to cloud) | 🔴 |
| Rotate regularly and on any suspicion; automate rotation where supported | 🔴 |
| Unique secrets per environment and per service | 🔴 |
| Secret scanning in pre-commit, CI, and push protection (chapter 07 §16) | 🔴 |
| Never log secrets; mask in CI logs | 🔴 |
| Developer machines: use secret manager CLIs or `.env` files that are git-ignored, never production secrets locally | 🟠 |
| Break-glass credentials stored offline with dual control and audit | 🟠 |

Kubernetes `Secret` objects are only base64-encoded by default — enable encryption at rest (KMS provider) and restrict RBAC, or use External Secrets Operator / Secrets Store CSI Driver with a real secret manager.

---

## 15. Data Protection and Privacy

### 15.1 Data classification

| Class | Examples | Controls |
|---|---|---|
| **Public** | Marketing pages, public docs | Integrity |
| **Internal** | Internal wikis, non-sensitive metrics | Authenticated access |
| **Confidential** | Customer PII, contracts, source code | Encryption, RBAC, audit, DLP |
| **Restricted** | Payment card data, health data, credentials, Aadhaar/national IDs, children's data | Strongest controls, minimised scope, tokenisation, strict access with approval and monitoring |

### 15.2 Privacy by design

```text
- Data minimisation: collect only what is needed for a stated purpose
- Purpose limitation and consent tracking (record purpose + consent version + timestamp)
- Retention schedules with automated deletion
- Data-subject / data-principal rights flows: access, correction, erasure, grievance, portability (as applicable)
- Pseudonymise/tokenise in analytics; aggregate where possible
- Privacy impact assessment (DPIA) for high-risk processing
- Data residency/localisation requirements in architecture (chapter 04)
- Breach notification runbooks with legal timelines
```

### 15.3 Regulatory quick map (confirm current rules with legal)

| Regime | Highlights relevant to engineers |
|---|---|
| **India Digital Personal Data Protection (DPDP) Act, 2023** and Rules | Notice and consent (via consent managers where applicable), purpose limitation, reasonable security safeguards, breach notification to the Data Protection Board and affected data principals, data principal rights, verifiable parental consent for children's data, extra duties for Significant Data Fiduciaries |
| **CERT-In Directions (India, 2022)** | Report specified cyber incidents to CERT-In within **6 hours**; retain ICT system logs for 180 days (within Indian jurisdiction); synchronise clocks to NTP sources specified in the directions — confirm current requirements |
| **EU GDPR** | Lawful basis, DPIAs, DSARs, records of processing, breach notification to authority within 72 hours, international transfer safeguards |
| **PCI DSS v4.0.1** | Minimise cardholder data scope (tokenisation via PSP), strong cryptography, MFA, logging, vulnerability management, script integrity on payment pages |
| **HIPAA (US)** | Safeguards for PHI, audit controls, BAAs |
| **Sector regulators (RBI, SEBI, IRDAI, etc.)** | Data localisation (e.g. payment system data in India), outsourcing and cyber resilience frameworks |

### 15.4 Logging and personal data

```text
🔴 Never log: passwords, tokens, full card numbers (PAN), CVV, OTPs, private keys, full government ID numbers
🟠 Mask/tokenise: emails, phone numbers, addresses, UPI IDs where not needed
🟠 Retention for logs defined and enforced; access to logs is itself audited
```

---

## 16. Software Supply Chain Security

Now **#3 in the OWASP Top 10:2025** (Software Supply Chain Failures).

### 16.1 Threats

```text
Source:      compromised developer accounts, malicious commits, unreviewed changes
Dependencies: vulnerable packages, typosquatting, dependency confusion, malicious maintainers, protestware
Build:       compromised CI runners, poisoned caches, unpinned actions/images, secret exfiltration
Distribution: tampered artifacts, compromised registries, unsigned releases
```

### 16.2 Controls

| Control | Tools / standards |
|---|---|
| **Dependency hygiene** | Lockfiles, pinned versions, automated updates (Dependabot/Renovate), minimal dependencies |
| **SCA** | OSV-Scanner, Dependabot alerts, Snyk, Trivy, Grype, OWASP Dependency-Check, `npm audit`, `pip-audit`, `govulncheck`, `cargo audit` |
| **Private registry / proxy** | Artifactory, Nexus, GitHub Packages, cloud artifact registries — scoped namespaces to prevent dependency confusion |
| **SBOM** | **SPDX** or **CycloneDX** generated per build (Syft, cdxgen, build-tool plugins) and stored with the release |
| **VEX** | Vulnerability Exploitability eXchange statements to say whether a known CVE actually affects your product |
| **Signing** | **Sigstore** (cosign, keyless signing with OIDC identities), signed commits/tags |
| **Provenance** | **SLSA** build provenance (in-toto attestations) — GitHub artifact attestations, SLSA generators |
| **Project health** | **OpenSSF Scorecard** for dependencies and your own repos |
| **Pinned CI actions/images** | Pin by full commit SHA / image digest |
| **Verification at deploy** | Admission policies (Kyverno, Sigstore policy-controller) that require signatures/attestations |

### 16.3 SLSA build levels (v1.x, summary)

| Level | Requirement (summary) |
|---|---|
| Build L1 | Provenance exists showing how the package was built |
| Build L2 | Signed provenance generated by a hosted build platform |
| Build L3 | Hardened build platform: builds isolated from each other; signing material inaccessible to build steps |

https://slsa.dev/

### 16.4 Example: sign and attest in CI

```bash
# Generate SBOM, sign image, attach SBOM attestation (cosign keyless via CI OIDC identity)
syft ghcr.io/shop/orders@sha256:<digest> -o cyclonedx-json > sbom.cdx.json
cosign sign --yes ghcr.io/shop/orders@sha256:<digest>
cosign attest --yes --type cyclonedx --predicate sbom.cdx.json ghcr.io/shop/orders@sha256:<digest>
# Verify (e.g. in deployment pipeline)
cosign verify ghcr.io/shop/orders@sha256:<digest> \
  --certificate-identity-regexp 'https://github.com/shop/orders/.github/workflows/release.yml@.*' \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com
```

Package manager specifics: **37-package-managers.md**, **37-package-and-dependency-management.md**.

---

## 17. Container and Kubernetes Security

### 17.1 Images

```text
🔴 Minimal base images (distroless, slim, Alpine/Wolfi/Chainguard-style minimal images) pinned by digest
🔴 Run as non-root user; no SUID binaries needed
🔴 No secrets in layers (use build secrets / runtime injection)
🔴 Scan images in CI and continuously in registries (Trivy, Grype, registry scanners)
🟠 Multi-stage builds; no compilers/package managers in runtime images
🟠 Rebuild regularly to pick up base-image patches
```

### 17.2 Kubernetes workload hardening

```yaml
securityContext:              # pod/container level
  runAsNonRoot: true
  runAsUser: 10001
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities:
    drop: ["ALL"]
  seccompProfile:
    type: RuntimeDefault
resources:
  requests: { cpu: "250m", memory: "256Mi" }
  limits:   { memory: "512Mi" }
automountServiceAccountToken: false   # unless the pod needs the API
```

| Control | Implementation |
|---|---|
| Pod Security Standards | Enforce **restricted** profile via Pod Security Admission labels on namespaces |
| Network policies | Default-deny ingress/egress per namespace; allow explicit flows |
| RBAC | Least privilege; no cluster-admin for apps/CI; audit RoleBindings |
| Admission control | Kyverno / OPA Gatekeeper / ValidatingAdmissionPolicy for policies (signed images, no `latest`, required labels) |
| Secrets | KMS encryption at rest; external secret managers |
| Runtime detection | Falco, Tetragon, cloud runtime security |
| Benchmarks | CIS Kubernetes Benchmark (kube-bench) |
| Upgrades | Stay within supported Kubernetes versions |

---

## 18. Cloud Security

| Area | Baseline |
|---|---|
| **Shared responsibility** | Know what the provider secures vs what you secure for each service type (IaaS/PaaS/SaaS) |
| **Identity (IAM)** | SSO for humans, no long-lived access keys, MFA (phishing-resistant for admins), least-privilege roles, permission boundaries, regular access reviews |
| **Organisation structure** | Separate accounts/projects/subscriptions per environment and workload; guardrail policies (SCPs, Azure Policy, GCP Org Policies) |
| **Network** | Private subnets by default; no public DBs; security groups least-open; private endpoints for managed services; WAF/DDoS protection at edge |
| **Data** | Encryption at rest with KMS (customer-managed keys where required); block public access on storage buckets by default |
| **Logging** | Org-wide audit trails (CloudTrail / Azure Activity Log / Cloud Audit Logs) to a protected log account |
| **Posture management** | CSPM / CNAPP findings triaged (provider security hubs, Prowler, ScoutSuite, commercial CNAPP) |
| **IaC scanning** | Checkov, Trivy config, tfsec-style rules in PR pipelines |
| **Metadata protection** | Enforce IMDSv2 (AWS) or equivalents |
| **Cost/abuse alerts** | Budget alerts detect cryptomining and abuse early |

Details: chapter **17**. Single-server hardening (SSH, UFW, Fail2Ban/CrowdSec): **39**.

---

## 19. CI/CD Pipeline Security

CI/CD systems hold the keys to production — treat them as production.

```text
🔴 Federate CI to cloud with OIDC (short-lived credentials) instead of storing long-lived cloud keys
🔴 Least-privilege tokens: set explicit `permissions:` (GitHub Actions) / scoped job tokens; read-only by default
🔴 Pin third-party actions/plugins/images by full commit SHA or digest
🔴 Untrusted PRs (forks) never get secrets; beware `pull_request_target` and similar privileged triggers
🔴 Treat PR titles, branch names, issue bodies as untrusted input — never interpolate directly into shell scripts
🔴 Protected environments with required reviewers for production deployments
🟠 Ephemeral, isolated runners for sensitive builds; no shared self-hosted runners for public repos
🟠 Audit pipeline configuration changes (CODEOWNERS on /.github/workflows, .gitlab-ci.yml)
🟠 Lint workflows (actionlint, zizmor) and scan with OpenSSF Scorecard
```

```yaml
# GitHub Actions — least privilege + OIDC to AWS (illustrative)
permissions:
  contents: read
  id-token: write        # required for OIDC federation
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production          # protected environment with reviewers
    steps:
      - uses: actions/checkout@<full-commit-sha>
      - uses: aws-actions/configure-aws-credentials@<full-commit-sha>
        with:
          role-to-assume: arn:aws:iam::123456789012:role/deploy-orders
          aws-region: ap-south-1
```

Pipeline design: chapter **16**.

---

## 20. Vulnerability Management

### 20.1 Lifecycle

```text
Discover (SCA, scanners, pentests, bug bounty, advisories) → Triage (applicable? reachable? exploited?)
→ Prioritise (severity × exploitability × exposure × asset criticality) → Remediate / mitigate
→ Verify fix → Close / document exception (VEX, risk acceptance with expiry) → Learn (RCA)
```

### 20.2 Prioritisation signals

| Signal | Source | Use |
|---|---|---|
| **CVSS v4.0** | FIRST (released Nov 2023) | Technical severity (Base + Threat + Environmental + Supplemental metrics) |
| **EPSS** | FIRST — Exploit Prediction Scoring System | Probability of exploitation in the near term |
| **CISA KEV** | Known Exploited Vulnerabilities catalogue | Actively exploited → fix first |
| **Reachability** | SCA tools with call-graph analysis | Is the vulnerable function actually used? |
| **Exposure** | Internet-facing vs internal | Raises/lowers urgency |
| **Asset criticality** | Data classification, business impact | Raises/lowers urgency |

### 20.3 Remediation SLAs (example — align with your policy and regulators)

| Priority | Criteria | Internet-facing | Internal |
|---|---|---|---|
| **P0** | In CISA KEV / known exploited, or Critical + exposed | 24–72 h | 7 days |
| **P1** | Critical | 7 days | 14 days |
| **P2** | High | 30 days | 30–60 days |
| **P3** | Medium | 90 days | 90 days |
| **P4** | Low | Next planned release | Best effort |

Exceptions require an owner, compensating controls, and an expiry date.

---

## 21. Security Logging, Monitoring, and Alerting

The 2025 OWASP category name emphasises **alerting** — logs that nobody acts on don't stop attacks.

### 21.1 Events to log

| Category | Events |
|---|---|
| Authentication | Login success/failure, MFA enrol/challenge/failure, password reset, passkey registration, lockouts |
| Authorisation | Access denied, privilege changes, role assignments |
| Sessions | Creation, invalidation, anomalies (impossible travel, new device) |
| Data | Exports, bulk reads, access to restricted data, deletions |
| Admin | Configuration changes, user management, feature flag changes |
| Security controls | Rate-limit triggers, WAF blocks, input validation failures (aggregated), webhook signature failures |
| Platform | Deployments, IAM changes, secret access, CI/CD changes |

Log fields: timestamp (UTC), event type, actor (user/service ID), tenant, source IP/device, target resource, outcome, correlation/trace ID. No secrets or unnecessary PII (§15.4).

### 21.2 Detection and response

```text
- Centralise logs into a SIEM / log platform with retention per policy and regulation (e.g. CERT-In log retention)
- Protect logs: append-only/immutable storage, separate account, restricted access
- Alert rules for high-signal events: admin login from new location, mass data export, privilege escalation,
  spikes in auth failures, disabled security controls, new access keys, CI/CD config changes
- Route alerts to on-call with runbooks; test detections (purple teaming, attack simulation)
- Map detections to MITRE ATT&CK techniques to find coverage gaps
```

OWASP Logging Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html

---

## 22. Incident Response

NIST **SP 800-61 Rev. 3** reorganised incident response guidance around the **CSF 2.0** functions (Govern, Identify, Protect, Detect, Respond, Recover) rather than a standalone life cycle. Practically, teams still run these steps:

```text
1. Prepare      → roles, contacts, runbooks, tooling, legal/regulatory obligations, tabletop exercises
2. Detect       → alerts, user reports, third-party notifications
3. Analyse      → scope, severity, affected data/systems, timeline (preserve evidence)
4. Contain      → isolate systems, revoke credentials/tokens, block IOCs, feature-flag off
5. Eradicate    → remove persistence, patch, rotate secrets
6. Recover      → restore from clean backups, monitor closely
7. Notify       → regulators (e.g. CERT-In within 6 h, DPDP Board, GDPR authority within 72 h), customers, partners — per legal advice
8. Learn        → blameless postmortem, actions with owners, detection improvements
```

### Security incident severity (example)

| Sev | Example | Response |
|---|---|---|
| SEV1 | Confirmed breach of restricted data; active attacker in production | Immediate war room; exec + legal; regulatory clock starts |
| SEV2 | Exploitable critical vuln exposed; credential leak with possible use | Same-day containment |
| SEV3 | Contained malware on a laptop; suspicious activity without impact | Business-hours response |
| SEV4 | Policy violation, low-risk finding | Ticketed |

General incident management, postmortems: chapter **21**.

---

## 23. Vulnerability Disclosure and Bug Bounties

```text
🔴 Publish a vulnerability disclosure policy (VDP): scope, safe harbour, how to report, response timelines
🔴 Provide security.txt (RFC 9116) at /.well-known/security.txt
🟠 SECURITY.md in every public repo (chapter 07 §5)
🟠 Acknowledge reports quickly (e.g. within 2–3 business days); keep reporters updated; credit with permission
🟡 Bug bounty programme (self-hosted or via platforms) once VDP and triage capacity are mature
```

```text
# /.well-known/security.txt
Contact: mailto:security@shopnow.example
Expires: 2027-10-01T00:00:00.000Z
Encryption: https://shopnow.example/pgp-key.txt
Policy: https://shopnow.example/security/disclosure-policy
Preferred-Languages: en, hi
Canonical: https://shopnow.example/.well-known/security.txt
```

For shipped software, publish **security advisories** (GitHub Security Advisories, CVE assignment via a CNA) and fix in supported versions.

---

## 24. Security Culture and Programme

| Element | Practice |
|---|---|
| **Security champions** | One trained engineer per team; monthly sync with the security team |
| **Training** | Secure coding (OWASP Top 10, language-specific), threat modelling workshops, phishing awareness; refreshed yearly |
| **Paved roads** | Secure templates and libraries (auth, logging with redaction, HTTP clients with timeouts, crypto wrappers) so the secure way is the easy way |
| **Maturity measurement** | OWASP SAMM assessments; track improvements quarterly |
| **Metrics** | Mean time to remediate by severity, % of services with threat models, % repos with secret scanning/SCA, SLA adherence, security findings escaping to production |
| **Blameless culture** | Reward reporting of mistakes and near misses |
| **Exercises** | Tabletops, red/purple team, game days |

---

## 25. Checklists

### New service (before first production release)
- [ ] Data classified; regulatory scope recorded
- [ ] Threat model completed; high risks mitigated or accepted
- [ ] ASVS target level chosen and key requirements verified
- [ ] AuthN via standard IdP (OIDC); MFA/passkeys available; sessions hardened
- [ ] AuthZ enforced server-side per resource; tenant isolation tested
- [ ] Input validation at boundaries; parameterised queries; output encoding; CSP
- [ ] Security headers configured; CORS allow-listed
- [ ] TLS 1.2+ (prefer 1.3); secrets in a secret manager; no static cloud keys
- [ ] Logging of security events with alerting; no secrets/PII in logs
- [ ] SAST, SCA, secret scanning, IaC/container scanning in CI; DAST in staging
- [ ] SBOM generated; artifacts signed; provenance recorded
- [ ] Rate limiting, quotas, timeouts
- [ ] Backups encrypted and restore-tested
- [ ] Incident runbook and contacts; security.txt / VDP in place

### Every PR touching security-sensitive code
- [ ] Authorisation checks for new endpoints/queries
- [ ] No new secrets; no sensitive data in logs or errors
- [ ] Error paths fail closed
- [ ] Dependency additions reviewed (need, maintenance, licence, vulnerabilities)
- [ ] Threat model updated if trust boundaries changed

### Quarterly
- [ ] Access reviews (cloud, Git hosting, CI, databases, admin panels)
- [ ] Secret rotation status
- [ ] Vulnerability SLA review and exceptions expiry
- [ ] Dependency and base-image currency
- [ ] Detection tests / tabletop exercise
- [ ] SAMM or programme maturity check-in

---

## 26. References

### Frameworks and standards
- NIST CSF 2.0: https://www.nist.gov/cyberframework
- NIST SSDF (SP 800-218): https://csrc.nist.gov/projects/ssdf
- NIST SP 800-63-4 Digital Identity Guidelines: https://pages.nist.gov/800-63-4/ · https://csrc.nist.gov/pubs/sp/800/63/4/final
- NIST SP 800-207 Zero Trust: https://csrc.nist.gov/pubs/sp/800/207/final
- NIST SP 800-61 Incident Response: https://csrc.nist.gov/pubs/sp/800/61/r3/final
- NIST PQC standards (FIPS 203/204/205): https://csrc.nist.gov/projects/post-quantum-cryptography
- ISO/IEC 27001: https://www.iso.org/standard/27001
- CIS Controls & Benchmarks: https://www.cisecurity.org/

### OWASP
- OWASP Top 10:2025: https://owasp.org/Top10/
- OWASP API Security Top 10: https://owasp.org/API-Security/
- OWASP Gen AI Security Project (LLM Top 10): https://genai.owasp.org/
- OWASP ASVS: https://owasp.org/www-project-application-security-verification-standard/
- OWASP SAMM: https://owaspsamm.org/
- OWASP Cheat Sheet Series: https://cheatsheetseries.owasp.org/
- OWASP Threat Dragon: https://owasp.org/www-project-threat-dragon/

### Protocols and specifications
- RFC 9700 OAuth 2.0 Security Best Current Practice: https://www.rfc-editor.org/rfc/rfc9700
- RFC 8725 JWT Best Current Practices: https://www.rfc-editor.org/rfc/rfc8725
- RFC 9449 DPoP: https://www.rfc-editor.org/rfc/rfc9449
- RFC 9116 security.txt: https://www.rfc-editor.org/rfc/rfc9116
- WebAuthn / passkeys: https://www.w3.org/TR/webauthn-3/ · https://fidoalliance.org/passkeys/
- Mozilla SSL Configuration Generator: https://ssl-config.mozilla.org/

### Supply chain and vulnerabilities
- SLSA: https://slsa.dev/ · Sigstore: https://www.sigstore.dev/ · OpenSSF Scorecard: https://scorecard.dev/
- SPDX: https://spdx.dev/ · CycloneDX: https://cyclonedx.org/
- CVSS v4.0: https://www.first.org/cvss/v4-0/ · EPSS: https://www.first.org/epss/
- CISA KEV catalogue: https://www.cisa.gov/known-exploited-vulnerabilities-catalog
- OSV: https://osv.dev/
- MITRE CWE Top 25: https://cwe.mitre.org/top25/ · MITRE ATT&CK: https://attack.mitre.org/

### Privacy and regulation (India and global)
- MeitY (DPDP Act & Rules): https://www.meity.gov.in/
- CERT-In: https://www.cert-in.org.in/
- EU GDPR: https://eur-lex.europa.eu/eli/reg/2016/679/oj
- PCI Security Standards Council: https://www.pcisecuritystandards.org/
- LINDDUN privacy threat modelling: https://linddun.org/
- Threat Modeling Manifesto: https://www.threatmodelingmanifesto.org/

---

**Previous:** [08 — Testing & Quality](./08-testing-and-quality.md) · **Next:** [10 — Database Engineering](./10-database-engineering.md)