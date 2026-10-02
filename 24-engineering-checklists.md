# ✅ Engineering Checklists — Production Engineering Guide

> A consolidated, copy-paste-ready set of checklists covering the whole software life cycle — from project kickoff to decommissioning. Each checklist is short enough to use in practice and links to the chapter with the full rationale. Use them in PR templates, issue templates, release tickets, readiness reviews, and audits.
>
> **How to use:** copy the relevant checklist into the PR/ticket/review doc, tick items, and record "n/a" with a reason instead of deleting items. Customise per team — but change the source checklist here via PR so everyone benefits.

**Status markers:** 🔴 required baseline · 🟠 recommended · 🟡 optional / context-dependent

---

## 📚 Table of Contents

**Plan & design**
1. [Project Kickoff](#1-project-kickoff)
2. [Requirements Review](#2-requirements-review)
3. [Architecture / Design Review](#3-architecture--design-review)
4. [Threat Model Review](#4-threat-model-review)
5. [API Design Review](#5-api-design-review)
6. [Data Model & Migration Review](#6-data-model--migration-review)

**Build & verify**
7. [New Repository Setup](#7-new-repository-setup)
8. [Pull Request — Author](#8-pull-request--author)
9. [Pull Request — Reviewer](#9-pull-request--reviewer)
10. [Definition of Done](#10-definition-of-done)
11. [Test Strategy & Release Testing](#11-test-strategy--release-testing)
12. [Security Pre-Release](#12-security-pre-release)
13. [Dependency Addition & Upgrade](#13-dependency-addition--upgrade)
14. [Performance Pre-Release](#14-performance-pre-release)

**Platform-specific**
15. [Frontend Release](#15-frontend-release)
16. [Backend Service Release](#16-backend-service-release)
17. [Mobile App Release](#17-mobile-app-release)
18. [Desktop App Release](#18-desktop-app-release)
19. [AI / LLM Feature Launch](#19-ai--llm-feature-launch)

**Deliver & run**
20. [CI/CD Pipeline](#20-cicd-pipeline)
21. [Infrastructure Change (IaC)](#21-infrastructure-change-iac)
22. [Kubernetes Workload](#22-kubernetes-workload)
23. [Production Readiness Review](#23-production-readiness-review)
24. [Deployment / Release Day](#24-deployment--release-day)
25. [Observability Readiness](#25-observability-readiness)
26. [Backup & DR Readiness](#26-backup--dr-readiness)
27. [On-Call Readiness](#27-on-call-readiness)
28. [Incident Response](#28-incident-response)
29. [Postmortem](#29-postmortem)
30. [Peak Event Readiness](#30-peak-event-readiness)

**People & governance**
31. [New Engineer Onboarding](#31-new-engineer-onboarding)
32. [Engineer Offboarding](#32-engineer-offboarding)
33. [Third-Party Vendor / SaaS Onboarding](#33-third-party-vendor--saas-onboarding)
34. [Privacy & Compliance (New Personal Data)](#34-privacy--compliance-new-personal-data)
35. [Service Decommissioning](#35-service-decommissioning)

**Cadence**
36. [Weekly](#36-weekly)
37. [Monthly](#37-monthly)
38. [Quarterly](#38-quarterly)
39. [Annually](#39-annually)

40. [Using These Checklists in Tooling](#40-using-these-checklists-in-tooling)
41. [References](#41-references)

---

## 1. Project Kickoff

*Details: [02 SDLC](./02-software-development-lifecycle.md), [03 Requirements](./03-requirements-and-planning.md)*

- [ ] 🔴 Problem statement with evidence; success metrics with baseline and target
- [ ] 🔴 Stakeholders mapped (incl. support, ops, security, legal, finance)
- [ ] 🔴 Data classification and regulatory scope identified (DPDP, GDPR, PCI DSS, sector rules)
- [ ] 🔴 Life-cycle model chosen; tailoring recorded
- [ ] 🔴 Owner team, tech lead, product owner named; RACI agreed
- [ ] 🔴 Definition of Ready / Definition of Done agreed
- [ ] 🟠 Non-goals listed
- [ ] 🟠 Initial risk register created
- [ ] 🟠 Rough estimate as a range with confidence; key unknowns and spikes planned
- [ ] 🟠 Architecture approach sketched; ADRs planned for one-way-door decisions
- [ ] 🟡 Cost estimate (infra + licences) and budget owner

---

## 2. Requirements Review

*Details: [03 Requirements §4, §14](./03-requirements-and-planning.md)*

- [ ] 🔴 Every requirement has ID, owner, priority, rationale, verification method
- [ ] 🔴 Necessary, singular, unambiguous, verifiable (ISO/IEC/IEEE 29148 characteristics)
- [ ] 🔴 No vague words without measures ("fast", "secure", "user-friendly")
- [ ] 🔴 Acceptance criteria (Given/When/Then) for every story
- [ ] 🔴 Error and unwanted-behaviour cases specified ("If… then…")
- [ ] 🔴 NFRs for relevant ISO/IEC 25010:2023 characteristics, each measurable
- [ ] 🔴 Security, privacy, accessibility (WCAG 2.2 AA), and compliance requirements explicit
- [ ] 🟠 Interfaces reference concrete specs/versions
- [ ] 🟠 Assumptions, constraints, dependencies documented with "needed-by" dates
- [ ] 🟠 QA reviewed testability; stakeholders/PO signed off; baseline versioned

---

## 3. Architecture / Design Review

*Details: [04 Architecture](./04-software-architecture.md), [05 Design](./05-software-design.md)*

- [ ] 🔴 Context, drivers, constraints, top quality-attribute scenarios with measures
- [ ] 🔴 C4 context + container diagrams (as code); deployment view
- [ ] 🔴 ADRs for significant decisions with alternatives considered
- [ ] 🔴 Data ownership, consistency model, residency defined
- [ ] 🔴 Failure modes of each dependency handled (timeouts, retries, circuit breakers, fallbacks)
- [ ] 🔴 Security: trust boundaries and threat model (see §4)
- [ ] 🔴 Observability: logs, metrics, traces, SLOs designed
- [ ] 🟠 Capacity estimate and load-test plan; cost estimate at expected and peak load
- [ ] 🟠 Rollout and migration plan (flags, expand/contract, rollback)
- [ ] 🟠 Fitness functions/architecture tests to enforce key decisions
- [ ] 🟠 Interfaces: contracts, errors, idempotency, versioning
- [ ] 🟡 Reversibility and exit strategy for vendor dependencies

---

## 4. Threat Model Review

*Details: [09 Security §4](./09-security-engineering.md#4-threat-modelling)*

- [ ] 🔴 Data-flow diagram with trust boundaries, assets, actors
- [ ] 🔴 STRIDE applied to each element/flow crossing a boundary (+ LINDDUN for personal data)
- [ ] 🔴 Threats rated; high risks mitigated or formally accepted with owner and expiry
- [ ] 🔴 Authentication and object-level authorisation designed for every entry point
- [ ] 🔴 Secrets handling, encryption in transit and at rest specified
- [ ] 🟠 Abuse cases (rate limiting, fraud, enumeration, automation) covered
- [ ] 🟠 Mitigations have tests (TC-SEC-…)
- [ ] 🟠 LLM features: prompt injection, data leakage, excessive agency reviewed (OWASP LLM Top 10)

---

## 5. API Design Review

*Details: [11 API & Integration](./11-api-and-integration.md)*

- [ ] 🔴 Integration style justified; contract (OpenAPI 3.1/3.2, AsyncAPI 3.0, proto) in repo and linted
- [ ] 🔴 Naming, casing, timestamps (RFC 3339 UTC), money format per style guide
- [ ] 🔴 Consistent status codes; RFC 9457 problem details with stable error codes
- [ ] 🔴 Cursor pagination on every list; max limits; allow-listed filters/sorts
- [ ] 🔴 AuthN mechanism per client type; scopes; object- and property-level authorisation
- [ ] 🔴 Idempotency keys for retried non-idempotent operations
- [ ] 🔴 Rate limits/quotas; 429 + Retry-After
- [ ] 🔴 Backward-compatibility rules followed; breaking-change check in CI
- [ ] 🟠 ETag/If-Match for contended updates; caching headers correct (`no-store` for sensitive)
- [ ] 🟠 Deprecation policy; clients tolerate unknown enum values/fields
- [ ] 🟠 Docs with examples, sandbox, changelog
- [ ] 🟠 Events: CloudEvents envelope, schema registry compatibility, outbox, idempotent consumers, DLQ

---

## 6. Data Model & Migration Review

*Details: [10 Database](./10-database-engineering.md)*

- [ ] 🔴 Constraints express invariants (NOT NULL, CHECK, FK, UNIQUE)
- [ ] 🔴 Correct types (timestamptz, bigint/numeric money, text, jsonb, uuid)
- [ ] 🔴 Migration backward compatible with running app (expand/contract)
- [ ] 🔴 `lock_timeout` set; no table-rewriting operations on large tables
- [ ] 🔴 Indexes created `CONCURRENTLY`; FKs/CHECKs added `NOT VALID` then validated
- [ ] 🔴 FK columns indexed; queries have supporting indexes (EXPLAIN reviewed)
- [ ] 🔴 Tenant scoping (tenant_id, RLS) for tenant data
- [ ] 🟠 Tested on production-sized masked data; duration measured
- [ ] 🟠 Backfill plan in batches with monitoring and throttling
- [ ] 🟠 New personal data: classification, retention, deletion method documented (§34)
- [ ] 🟠 Backup/PITR verified before risky migrations; rollback (roll-forward) plan

---

## 7. New Repository Setup

*Details: [06 Coding standards](./06-coding-standards.md), [07 Version control](./07-version-control.md), [42 Git setup](./42-Git-GitHub-GitLab-Bitbucket-Setup.md)*

- [ ] 🔴 Default branch `main`, protected: PR-only, required checks, approvals, code-owner review, no force push
- [ ] 🔴 README, CODEOWNERS, `.gitignore`, `.gitattributes`, `.editorconfig`
- [ ] 🔴 Formatter, linter, strict type checking from shared org config
- [ ] 🔴 CI: build, tests, lint, SAST, SCA, secret scanning; branch protection requires them
- [ ] 🔴 Secret scanning push protection enabled
- [ ] 🔴 Dependency update bot (Dependabot/Renovate)
- [ ] 🟠 CONTRIBUTING, SECURITY.md, CHANGELOG, PR template, issue templates
- [ ] 🟠 Pre-commit hooks (format, lint, secrets)
- [ ] 🟠 Merge strategy (squash) + linear history; auto-delete branches; merge queue if busy
- [ ] 🟠 Release automation (release-please/semantic-release/changesets); tag protection
- [ ] 🟠 `docs/` structure and docs CI (lint, links)
- [ ] 🟡 Catalogue entry (owner, tier, links) and `AGENTS.md` context file

---

## 8. Pull Request — Author

*Details: [07 §9](./07-version-control.md#9-pull-requests--merge-requests)*

- [ ] 🔴 Self-reviewed the diff; PR focused on one change; < ~400 lines (or justified)
- [ ] 🔴 Title follows Conventional Commits; description: what, why, how, testing, risk, rollout, rollback
- [ ] 🔴 Linked ticket/requirement IDs
- [ ] 🔴 Tests added/updated at the right level; CI green
- [ ] 🔴 No secrets, no sensitive data in logs/errors; inputs validated; authorisation checked
- [ ] 🔴 Errors handled; resources closed; timeouts on new outbound calls
- [ ] 🟠 Docs/runbooks/ADRs/changelog updated
- [ ] 🟠 Observability added for new paths (logs/metrics/traces)
- [ ] 🟠 Feature flag for risky behaviour; migrations backward compatible
- [ ] 🟠 AI-generated portions reviewed, understood, and owned

---

## 9. Pull Request — Reviewer

*Details: [07 §10](./07-version-control.md#10-code-review)*

- [ ] 🔴 Design fits architecture and module boundaries
- [ ] 🔴 Correctness: edge cases, error paths, concurrency, idempotency
- [ ] 🔴 Security: authZ per object, injection, secrets, data exposure
- [ ] 🔴 Tests meaningful and deterministic; would fail if the code broke
- [ ] 🟠 Operability: logging, metrics, timeouts, migration safety, rollback
- [ ] 🟠 Readability: names, complexity, comments explain why
- [ ] 🟠 Performance: N+1, unbounded queries, hot-path allocations
- [ ] 🟠 Comments labelled (blocking / suggestion / question / nit / praise)
- [ ] 🟠 Responded within the review SLA

---

## 10. Definition of Done

*Details: [02 §12](./02-software-development-lifecycle.md#12-definition-of-ready--definition-of-done)*

- [ ] 🔴 Code meets standards; reviewed and approved
- [ ] 🔴 Unit + integration/contract tests pass; acceptance criteria verified
- [ ] 🔴 No new critical/high static-analysis or dependency vulnerabilities (or accepted with reason)
- [ ] 🔴 Accessibility checks for UI changes
- [ ] 🔴 Observability added; alerts/dashboards updated where relevant
- [ ] 🔴 Backward-compatible migrations; feature flag for risky changes; rollback path known
- [ ] 🔴 Docs updated (README, API reference, runbooks, ADRs, release notes)
- [ ] 🔴 Merged to main; deployed to staging; smoke tests green; PO accepted

---

## 11. Test Strategy & Release Testing

*Details: [08 Testing & Quality](./08-testing-and-quality.md)*

**Strategy (per product)**
- [ ] 🔴 Quality attributes ranked and mapped to test types
- [ ] 🔴 Test levels, owners, automation approach, pipeline stages, and time budgets defined
- [ ] 🔴 Test environments and synthetic/masked test data strategy (no real PII)
- [ ] 🔴 Defect severity/priority definitions and SLAs
- [ ] 🟠 Flaky-test policy; metrics (escaped defects, flaky rate, PR feedback time)

**Release (STLC closure)**
- [ ] 🔴 Exit criteria met (coverage of high-risk items, pass rate, no open Critical/High) or risks accepted in writing
- [ ] 🔴 Requirements traceability complete for in-scope items
- [ ] 🔴 Regression suite green; E2E critical journeys green
- [ ] 🟠 Exploratory testing sessions done on changed areas
- [ ] 🟠 Test completion report approved; known issues shared with support

---

## 12. Security Pre-Release

*Details: [09 Security](./09-security-engineering.md)*

- [ ] 🔴 Threat model current; high risks mitigated
- [ ] 🔴 OWASP Top 10:2025 categories considered; ASVS target level verified for key controls
- [ ] 🔴 SAST, SCA, secret scanning, IaC/container scans clean or exceptions approved
- [ ] 🔴 DAST/API security scan in staging; authorisation test matrix passing
- [ ] 🔴 Security headers and CSP; CORS allow-list; cookies hardened
- [ ] 🔴 Secrets in secret manager; no long-lived cloud keys; least-privilege roles
- [ ] 🔴 Security event logging with alerting
- [ ] 🟠 SBOM generated; artifacts signed; provenance recorded
- [ ] 🟠 Penetration test for high-risk releases (findings triaged)
- [ ] 🟠 Rate limiting on sensitive endpoints (auth, OTP, signup, export)

---

## 13. Dependency Addition & Upgrade

*Details: [06 §25](./06-coding-standards.md#25-dependencies), [09 §16](./09-security-engineering.md#16-software-supply-chain-security), [37](./37-package-and-dependency-management.md)*

**Adding**
- [ ] 🔴 Genuine need; standard library or existing dependency insufficient
- [ ] 🔴 Package verified (correct name — no typosquat/hallucinated package), maintained, acceptable licence
- [ ] 🔴 No known critical vulnerabilities; reasonable transitive footprint
- [ ] 🟠 OpenSSF Scorecard / project health reviewed; ownership/bus factor considered

**Upgrading (major)**
- [ ] 🔴 Release notes and breaking changes read; codemods/migration guides applied
- [ ] 🔴 Full test suite + integration tests pass; staging soak
- [ ] 🟠 Rollback plan; deployed progressively
- [ ] 🟠 Deprecation warnings resolved

---

## 14. Performance Pre-Release

*Details: [19 Performance](./19-performance-and-scalability.md)*

- [ ] 🔴 NFRs stated as percentiles at a defined load and data volume
- [ ] 🔴 Load test with open-model arrival rate at ≥ expected peak × safety factor
- [ ] 🔴 Results compared to baseline; no unexplained regressions
- [ ] 🟠 Stress/spike tests: graceful degradation and recovery verified
- [ ] 🟠 Soak test for long-running services or memory-sensitive changes
- [ ] 🟠 Bottlenecks profiled; capacity model updated; quotas and third-party limits confirmed
- [ ] 🟠 Autoscaling behaviour verified

---

## 15. Frontend Release

*Details: [12 Frontend](./12-frontend-engineering.md)*

- [ ] 🔴 Core Web Vitals within thresholds on key pages (lab + field trend)
- [ ] 🔴 Bundle size within budget; no unexpected new heavy dependencies
- [ ] 🔴 No new accessibility violations (axe) + manual keyboard/screen-reader pass on changed flows
- [ ] 🔴 CSP and security headers intact; no secrets in client bundles
- [ ] 🔴 Error tracking with release tag and source maps uploaded privately
- [ ] 🟠 Cross-browser/device checks per support matrix
- [ ] 🟠 i18n strings externalised; layouts tested with longer translations
- [ ] 🟠 Cache headers correct (immutable hashed assets, short HTML cache); version-skew handling
- [ ] 🟠 Framework security advisories reviewed (RSC/server actions patched)

---

## 16. Backend Service Release

*Details: [13 Backend](./13-backend-engineering.md)*

- [ ] 🔴 Contract changes backward compatible; consumers' contract tests pass
- [ ] 🔴 Migrations as separate step; expand phase deployed first
- [ ] 🔴 Timeouts/retries/circuit breakers for new dependencies
- [ ] 🔴 Health probes correct; graceful shutdown verified
- [ ] 🔴 Config validated; secrets present in target environment
- [ ] 🟠 Idempotency for new mutations/consumers; DLQ monitoring
- [ ] 🟠 Dashboards/alerts updated; runbook updated for new failure modes
- [ ] 🟠 Container image scanned, signed, deployed by digest

---

## 17. Mobile App Release

*Details: [14 Mobile](./14-mobile-engineering.md)*

- [ ] 🔴 Store requirements met: Android target API level (API 36 for new apps/updates from 31 Aug 2026), iOS built with required Xcode/SDK (Xcode 26+ since 28 Apr 2026) — re-verify each year
- [ ] 🔴 Privacy disclosures accurate (Data safety, App Privacy, privacy manifests incl. SDKs)
- [ ] 🔴 Crash-free and ANR metrics on beta/internal track within targets
- [ ] 🔴 Mapping files/dSYMs uploaded
- [ ] 🔴 Backend compatible with all supported app versions
- [ ] 🔴 Kill switches/remote config defaults verified; forced-update path works
- [ ] 🟠 Staged/phased rollout plan with halt criteria
- [ ] 🟠 Accessibility (TalkBack/VoiceOver, large fonts), localisation checks
- [ ] 🟠 Release notes, screenshots, metadata updated; reviewer test account provided
- [ ] 🟠 Account deletion and permission flows verified

---

## 18. Desktop App Release

*Details: [15 Desktop](./15-desktop-engineering.md)*

- [ ] 🔴 Windows binaries/installers Authenticode-signed (HSM/cloud-backed keys) and timestamped
- [ ] 🔴 macOS: Developer ID signed, hardened runtime, notarised, stapled
- [ ] 🔴 Linux packages/repos signed; checksums published
- [ ] 🔴 Auto-update artifacts signed and verified; staged rollout configured
- [ ] 🔴 Electron/Tauri security settings intact (isolation, sandbox, IPC validation, CSP, capabilities)
- [ ] 🟠 Install/upgrade (N-1, N-2)/uninstall tested on clean machines per OS/arch
- [ ] 🟠 Crash reporting symbols uploaded
- [ ] 🟠 Code-signing certificate expiry tracked (≤ 460-day validity)
- [ ] 🟡 Enterprise: MSI/PKG, silent install switches, managed config docs updated

---

## 19. AI / LLM Feature Launch

*Details: [03 §21](./03-requirements-and-planning.md#21-requirements-for-ai--llm-features), [09 §7](./09-security-engineering.md#7-llm-and-ai-application-security), [18 §18](./18-observability-and-monitoring.md#18-observability-for-ai--llm-applications)*

- [ ] 🔴 Task success metrics and versioned eval dataset; evals pass thresholds
- [ ] 🔴 Red-team suite (prompt injection, jailbreaks, data exfiltration) run; findings addressed
- [ ] 🔴 Tools run with end-user permissions; side effects require confirmation/authorisation
- [ ] 🔴 Permission-aware retrieval (RAG) — no cross-tenant/unauthorised documents
- [ ] 🔴 No personal data to unapproved providers; DPA and residency checked; redaction in place
- [ ] 🔴 Model output validated/encoded before use (HTML, SQL, shell, tools)
- [ ] 🔴 Token/cost budgets, rate limits, timeouts; fallback when the provider is unavailable
- [ ] 🟠 Users informed when content is AI-generated; feedback mechanism
- [ ] 🟠 Model/prompt versions logged; evals re-run on any model or prompt change
- [ ] 🟠 Quality and cost dashboards; drift monitoring

---

## 20. CI/CD Pipeline

*Details: [16 DevOps & CI/CD](./16-devops-and-ci-cd.md)*

- [ ] 🔴 Pipeline as code; runs on every PR/push; PR feedback ≤ ~15 min
- [ ] 🔴 Build once; artifacts immutable, deployed by digest; SBOM + signature + provenance
- [ ] 🔴 OIDC to cloud; least-privilege job permissions; actions/components pinned by SHA
- [ ] 🔴 Untrusted input never interpolated into scripts; fork PRs without secrets
- [ ] 🔴 Protected environments with approvals for production
- [ ] 🔴 Progressive delivery with automated verification and rollback
- [ ] 🟠 Caching, concurrency cancellation, affected-only builds (monorepo)
- [ ] 🟠 Deployment events recorded for DORA metrics and dashboards
- [ ] 🟠 Ephemeral runners for sensitive jobs; pipeline files under CODEOWNERS

---

## 21. Infrastructure Change (IaC)

*Details: [17 Cloud & Infrastructure](./17-cloud-and-infrastructure.md)*

- [ ] 🔴 Change in code via PR; plan output reviewed (destroy/replace actions explicitly checked)
- [ ] 🔴 Policy-as-code checks pass (encryption, no public buckets, approved regions, tags)
- [ ] 🔴 Remote state with locking/encryption; correct workspace/environment
- [ ] 🔴 Deletion protection on stateful resources
- [ ] 🟠 Cost estimate (Infracost) reviewed
- [ ] 🟠 Applied by pipeline with approval for production; drift check after apply
- [ ] 🟠 Provider/module versions pinned; upgrades tested in staging

---

## 22. Kubernetes Workload

*Details: [17 §7–9](./17-cloud-and-infrastructure.md#7-kubernetes-in-production), [09 §17](./09-security-engineering.md#17-container-and-kubernetes-security)*

- [ ] 🔴 Image by digest; signed; scanned; non-root; read-only root FS; capabilities dropped
- [ ] 🔴 Requests set (and memory limits); HPA/KEDA configured; ≥ 2–3 replicas across zones
- [ ] 🔴 Startup/liveness/readiness probes correct; graceful shutdown (preStop, terminationGracePeriod)
- [ ] 🔴 PodDisruptionBudget; topology spread constraints
- [ ] 🔴 NetworkPolicies; ServiceAccount least privilege; no token automount unless needed
- [ ] 🔴 Secrets via external secret store; ConfigMaps for non-secret config
- [ ] 🔴 Traffic via Gateway API or a maintained controller (**not** retired Ingress NGINX)
- [ ] 🟠 Deployed via GitOps; Rollout/canary analysis for critical services
- [ ] 🟠 Labels for ownership/cost allocation

---

## 23. Production Readiness Review

*Details: [20 §22](./20-reliability-and-disaster-recovery.md#22-production-readiness-and-reliability-reviews)*

- [ ] 🔴 Owner team, on-call rotation, catalogue entry, tier (RPO/RTO)
- [ ] 🔴 SLOs + error-budget policy; burn-rate alerts with runbooks
- [ ] 🔴 Dashboards (RED/USE, dependencies, business metrics); logs and traces correlated
- [ ] 🔴 Multi-AZ; capacity for AZ loss; load tested
- [ ] 🔴 Resilience patterns for every dependency; graceful degradation defined
- [ ] 🔴 Progressive delivery + automated rollback; feature flags/kill switches
- [ ] 🔴 Backups/PITR; restore tested; DR strategy for tier implemented
- [ ] 🔴 Security review passed; access model (read / JIT write / break-glass)
- [ ] 🟠 FMEA top risks mitigated; chaos/game day scheduled
- [ ] 🟠 Cost estimate and budget alerts
- [ ] 🟠 Status page component; support briefed (known issues, escalation path)

---

## 24. Deployment / Release Day

*Details: [02 §15](./02-software-development-lifecycle.md#15-release-management-and-versioning), [16 §20](./16-devops-and-ci-cd.md#20-deployment-verification-and-automated-rollback)*

**Before**
- [ ] 🔴 Release artifact is the tested one (same digest); version tagged; changelog/release notes ready
- [ ] 🔴 Rollback procedure verified; on-call aware; not in a freeze window
- [ ] 🔴 Migrations (expand) applied/ready; flags configured per environment
- [ ] 🟠 Dashboards and alert links in the release ticket; stakeholders informed

**During**
- [ ] 🔴 Canary/progressive rollout; watch error rate, latency, saturation, business KPIs
- [ ] 🔴 Halt and roll back on regression — don't debug in production under load

**After**
- [ ] 🔴 Smoke tests and post-deploy verification complete
- [ ] 🟠 Deployment recorded (DORA); release notes published; support informed
- [ ] 🟠 Contract migrations and flag clean-up scheduled

---

## 25. Observability Readiness

*Details: [18 Observability](./18-observability-and-monitoring.md)*

- [ ] 🔴 OTel instrumentation with `service.name`, `service.version`, environment
- [ ] 🔴 RED metrics by route template; runtime and pool saturation metrics; no unbounded labels
- [ ] 🔴 Structured JSON logs with trace/span IDs; no secrets/PII
- [ ] 🔴 Trace context propagated over HTTP, gRPC, messaging
- [ ] 🔴 SLO burn-rate alerts (fast and slow) routed to the owning team with runbooks
- [ ] 🟠 Dependency dashboards; synthetic check for the main journey; business metrics
- [ ] 🟠 Deploy/flag annotations on dashboards
- [ ] 🟠 Retention and access controls appropriate for data classification

---

## 26. Backup & DR Readiness

*Details: [20 Reliability & DR](./20-reliability-and-disaster-recovery.md), [10 §14](./10-database-engineering.md#14-backup-and-recovery)*

- [ ] 🔴 3-2-1-1-0: copies off-site and immutable; backup credentials isolated
- [ ] 🔴 PITR configured for databases; object storage versioning/lock where needed
- [ ] 🔴 Automated restore test with measured time ≤ RTO in the last cycle
- [ ] 🔴 Service mapped to an RPO/RTO tier agreed with the business (BIA)
- [ ] 🔴 DR plan and runbooks: owned, current, available offline/outside primary region
- [ ] 🟠 DR region quotas, secrets, IaC, and dependencies ready
- [ ] 🟠 Failover and failback exercised at the tier's required frequency
- [ ] 🟠 Ransomware scenario (rebuild from scratch) considered for Tier 0–1

---

## 27. On-Call Readiness

*Details: [21 §3](./21-production-operations.md#3-on-call-design)*

- [ ] 🔴 Paging app installed; escalation policy configured; contact details current
- [ ] 🔴 Access verified (dashboards, logs, traces, consoles, break-glass procedure, VPN/ZTNA)
- [ ] 🔴 Knows how to declare an incident, page others, update the status page
- [ ] 🔴 Shadowed and reverse-shadowed a rotation
- [ ] 🟠 Read top runbooks and service architecture; handoff notes reviewed
- [ ] 🟠 Laptop/charger/phone/connectivity plan for the on-call week

---

## 28. Incident Response

*Details: [21 §5–9](./21-production-operations.md#5-incident-management--overview), [09 §22](./09-security-engineering.md#22-incident-response)*

- [ ] 🔴 Incident declared with severity; IC named; channel opened
- [ ] 🔴 Recent changes checked first (deploys, flags, config, migrations, vendor status)
- [ ] 🔴 Mitigation prioritised over root cause (rollback, flag off, failover, scale, shed load)
- [ ] 🔴 Updates at committed cadence (internal + status page)
- [ ] 🔴 Timeline recorded; production commands logged
- [ ] 🔴 Security/privacy implications assessed; legal/regulatory clocks (e.g. CERT-In 6 h) considered
- [ ] 🟠 Customer support briefed with workaround/macros
- [ ] 🟠 Resolution confirmed after observation period; postmortem scheduled

---

## 29. Postmortem

*Details: [21 §10](./21-production-operations.md#10-blameless-postmortems)*

- [ ] 🔴 Required for SEV1/SEV2, data loss, security incidents, significant SLO burn
- [ ] 🔴 Blameless; timeline shows what was known when
- [ ] 🔴 Impact quantified (users, duration, revenue, SLO budget, data)
- [ ] 🔴 Contributing factors (not a single "root cause"); what went well included
- [ ] 🔴 Actions: owner, due date, ticket, type (prevent/detect/mitigate/process)
- [ ] 🟠 Reviewed in a meeting; published to the archive; shared org-wide
- [ ] 🟠 Runbooks, alerts, dashboards, tests updated
- [ ] 🟠 Customer-facing summary when appropriate

---

## 30. Peak Event Readiness

*Details: [19 §21.1](./19-performance-and-scalability.md#21-scenario-playbooks), [21 §23.4](./21-production-operations.md#23-scenario-playbooks)*

- [ ] 🔴 Traffic forecast agreed with business; critical journeys identified
- [ ] 🔴 Load tests at 3–5× forecast passed; bottlenecks fixed
- [ ] 🔴 Cloud quotas and third-party limits (PSP, SMS, email, CDN) increased/confirmed
- [ ] 🔴 Pre-scaling plan; autoscaling verified; caches warm-up plan
- [ ] 🔴 Kill switches and degradation modes tested; waiting room ready if needed
- [ ] 🔴 Change freeze window announced; on-call and war room staffed
- [ ] 🟠 Game day for dependency failures done
- [ ] 🟠 Vendor contacts on standby; leadership update cadence agreed
- [ ] 🟠 Post-event review scheduled

---

## 31. New Engineer Onboarding

*Details: [22 §12](./22-documentation-and-knowledge-management.md#12-onboarding-documentation), [41](./41-(a)Windows-11-Fresh-Setup.md), [42](./42-Git-GitHub-GitLab-Bitbucket-Setup.md)*

- [ ] 🔴 Accounts: SSO + MFA, Git host, CI, ticketing, chat, observability (read), docs
- [ ] 🔴 Laptop set up per 41(a)/41(b); Git, SSH, signing per 42; IDE per 30–36
- [ ] 🔴 Builds and tests run locally for the team's main repos
- [ ] 🔴 Buddy assigned; meetings with manager, TL, PM, on-call lead
- [ ] 🟠 Read handbook chapters 01, 02, 06, 07 + team charter and service READMEs
- [ ] 🟠 First PR merged in week 1; onboarding docs feedback submitted
- [ ] 🟠 30/60/90-day goals agreed

---

## 32. Engineer Offboarding

*Details: [21 §14](./21-production-operations.md#14-service-requests-and-access-management)*

- [ ] 🔴 SSO account disabled same day; SCIM deprovisioning verified across tools
- [ ] 🔴 Personal access tokens, SSH keys, deploy keys, API keys revoked; shared secrets they knew rotated
- [ ] 🔴 Cloud, database, VPN/ZTNA, paging, and vendor console access removed
- [ ] 🔴 Device returned/wiped per policy
- [ ] 🟠 Ownership transferred: repos (CODEOWNERS), services, on-call slots, docs, tickets
- [ ] 🟠 Knowledge transfer sessions recorded/documented

---

## 33. Third-Party Vendor / SaaS Onboarding

*Details: [09](./09-security-engineering.md), [11 §19](./11-api-and-integration.md#19-integrating-with-third-party-apis), [20 §19](./20-reliability-and-disaster-recovery.md#19-dependency-and-third-party-reliability)*

- [ ] 🔴 Security assessment (certifications such as ISO 27001/SOC 2, pentest summaries, vulnerability disclosure)
- [ ] 🔴 Data processing agreement; data residency and cross-border transfer compliance (DPDP/GDPR)
- [ ] 🔴 SSO/SCIM integration where available; least-privilege API keys stored in secret manager
- [ ] 🔴 SLA, support levels, incident notification commitments; status page subscribed
- [ ] 🟠 Exit plan: data export format, contract termination, alternatives
- [ ] 🟠 Integration wrapped in adapter; timeouts/retries/circuit breaker; reconciliation for financial data
- [ ] 🟠 Cost monitored; renewal dates tracked

---

## 34. Privacy & Compliance (New Personal Data)

*Details: [09 §15](./09-security-engineering.md#15-data-protection-and-privacy), [03 §20](./03-requirements-and-planning.md#20-compliance-and-regulatory-requirements)*

- [ ] 🔴 Purpose defined; data minimised; lawful basis/consent captured and recorded
- [ ] 🔴 Classification assigned; encryption, access controls, and logging appropriate
- [ ] 🔴 Retention period and deletion method (incl. backups, caches, search, analytics) defined
- [ ] 🔴 Data-principal/subject rights supported (access, correction, erasure, grievance)
- [ ] 🔴 Privacy notice and store disclosures updated (web, mobile)
- [ ] 🟠 DPIA for high-risk processing; children's data rules checked
- [ ] 🟠 Third parties receiving the data covered by agreements; transfers assessed
- [ ] 🟠 Breach notification runbook covers this data

---

## 35. Service Decommissioning

*Details: [02 §17](./02-software-development-lifecycle.md#17-retirement-and-decommissioning)*

- [ ] 🔴 ADR documenting decision, replacement, and timeline; stakeholders notified
- [ ] 🔴 Deprecation communicated (Sunset/Deprecation headers, docs, emails); usage monitored to zero
- [ ] 🔴 Data migrated/archived/deleted per retention and legal obligations
- [ ] 🔴 Credentials, keys, service accounts, DNS records, certificates revoked/removed
- [ ] 🔴 Infrastructure destroyed via IaC; billing confirmed stopped
- [ ] 🟠 Repository archived read-only; catalogue updated; dashboards/alerts removed
- [ ] 🟠 Runbooks and docs marked archived

---

## 36. Weekly

- [ ] Review SLO/error-budget status and top alerts (noise vs actionable)
- [ ] On-call handoff with notes; toil log updated
- [ ] Flaky tests triaged; CI duration and failure rate checked
- [ ] Dependency update PRs merged or triaged
- [ ] Security findings (new critical/high) triaged against SLAs
- [ ] Team update written (shipped, next, risks, asks)

## 37. Monthly

- [ ] Ops review: incidents, postmortem action status, recurring problems
- [ ] Alert quality review; runbooks updated
- [ ] Cost review per team/service; anomalies investigated
- [ ] OS/base-image patching status; certificate/domain expiry check
- [ ] Observability cardinality/ingestion top contributors reviewed
- [ ] On-call health metrics (pages per shift, after-hours load)

## 38. Quarterly

- [ ] DORA metrics trend and developer experience survey
- [ ] Access reviews (production, cloud, Git hosting, databases, CI)
- [ ] Tech-debt register and maintenance backlog reviewed
- [ ] Capacity review and forecast; quota checks
- [ ] Restore tests and DR exercises per tier schedule
- [ ] Runtime/framework/Kubernetes version currency vs end-of-life dates
- [ ] Team health check, psychological safety pulse, working agreements revisited
- [ ] Docs gardening: stale docs past review date updated or archived
- [ ] Threat models reviewed for changed systems

## 39. Annually

- [ ] Architecture review of critical systems; tech radar update
- [ ] Full DR exercise for Tier 0–1; BIA refresh with business owners
- [ ] Penetration tests for critical applications
- [ ] Policy and standards review (this handbook's chapters)
- [ ] Licence and vendor contract review; exit plans checked
- [ ] Supported OS/browser/device matrix review (web, mobile, desktop)
- [ ] Retention schedules and privacy notices reviewed against current laws
- [ ] Code-signing certificates, store accounts, and developer programme memberships renewed

---

## 40. Using These Checklists in Tooling

### 40.1 PR template (GitHub/GitLab/Bitbucket)

```markdown
<!-- .github/PULL_REQUEST_TEMPLATE.md -->
## What / Why
## How tested
## Checklist (handbook ch. 24 §8)
- [ ] Self-reviewed; focused; < 400 lines (or justified)
- [ ] Tests at the right level; CI green
- [ ] No secrets/PII; inputs validated; authorisation checked
- [ ] Observability added; docs updated
- [ ] Risk/rollout/rollback described; flags/migrations safe
```

### 40.2 Issue templates

Create templates for: production readiness review (§23), release (§24), postmortem (§29), vendor onboarding (§33), decommissioning (§35), each pre-filled with the checklist.

```yaml
# .github/ISSUE_TEMPLATE/production-readiness.yml (excerpt)
name: Production Readiness Review
description: Checklist before first production launch (handbook ch. 24 §23)
title: "[PRR] <service>"
labels: [prr]
body:
  - type: checkboxes
    id: reliability
    attributes:
      label: Reliability
      options:
        - label: SLOs + error-budget policy; burn-rate alerts with runbooks
        - label: Multi-AZ; capacity for AZ loss; load tested
        - label: Backups/PITR; restore tested; DR strategy for tier
```

### 40.3 Automate what you can

```text
Many checklist items can become automated gates instead of manual ticks:
- Lint/format/type/test/coverage, SAST/SCA/secrets → CI required checks
- Breaking API changes → oasdiff/buf breaking in CI
- Migration safety → squawk/Atlas lint
- Kubernetes manifest rules → Kyverno/OPA admission policies
- IaC rules → Checkov/Conftest in PR
- Docs freshness → bot flags docs past review date
- Certificate/domain expiry → monitoring alerts
Every automated item frees human review for judgement calls.
```

---

## 41. References

This chapter consolidates checklists from the rest of the handbook. Source chapters:

| Area | Chapters |
|---|---|
| Foundations & process | [01](./01-engineering-foundations.md), [02](./02-software-development-lifecycle.md), [03](./03-requirements-and-planning.md) |
| Design | [04](./04-software-architecture.md), [05](./05-software-design.md), [06](./06-coding-standards.md) |
| Build & verify | [07](./07-version-control.md), [08](./08-testing-and-quality.md), [09](./09-security-engineering.md), [10](./10-database-engineering.md), [11](./11-api-and-integration.md) |
| Platforms | [12](./12-frontend-engineering.md), [13](./13-backend-engineering.md), [14](./14-mobile-engineering.md), [15](./15-desktop-engineering.md) |
| Deliver & run | [16](./16-devops-and-ci-cd.md), [17](./17-cloud-and-infrastructure.md), [18](./18-observability-and-monitoring.md), [19](./19-performance-and-scalability.md), [20](./20-reliability-and-disaster-recovery.md), [21](./21-production-operations.md) |
| People | [22](./22-documentation-and-knowledge-management.md), [23](./23-team-collaboration.md) |
| Tooling | [25–29](./29-terminal-and-cli-engineering.md), [30–36](./30-ide-and-developer-tooling.md), [37](./37-package-managers.md), [38](./38-developer-desktop-software-tool-stack.md), [39](./39-vps-enterprise-setup-guide.md), [40](./40-autocannon_production_CLI.md), [41](./41-(a)Windows-11-Fresh-Setup.md), [42](./42-Git-GitHub-GitLab-Bitbucket-Setup.md) |

External: *The Checklist Manifesto* — Atul Gawande (why short, well-designed checklists improve outcomes in complex work).

---

**Previous:** [23 — Team Collaboration](./23-team-collaboration.md) · **Next:** [25 — Windows PowerShell](./25-windows-powershell.md)