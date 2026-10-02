# 🔄 Software Development Lifecycle (SDLC) — Production Engineering Guide

> How production software moves from an idea to retirement: the international process standard (ISO/IEC/IEEE 12207:2026), practical phases with entry/exit gates, life-cycle models (Waterfall → Agile → DevOps), Scrum and Kanban, the Secure SDLC (NIST SSDF), the **Software Testing Life Cycle (STLC)** woven into every phase, quality gates, environments, release management, maintenance, and scenario playbooks.
>
> Related: [01 Foundations](./01-engineering-foundations.md) · [03 Requirements & Planning](./03-requirements-and-planning.md) · [08 Testing & Quality — full STLC](./08-testing-and-quality.md) · [16 DevOps & CI/CD](./16-devops-and-ci-cd.md)

---

## 📚 Table of Contents

1. [What the SDLC Is](#1-what-the-sdlc-is)
2. [ISO/IEC/IEEE 12207:2026 — The Process Standard](#2-isoiecieee-122072026--the-process-standard)
3. [Life-Cycle Stages vs Processes](#3-life-cycle-stages-vs-processes)
4. [The Practical SDLC — 9 Phases with Gates](#4-the-practical-sdlc--9-phases-with-gates)
5. [Life-Cycle Models](#5-life-cycle-models)
6. [Choosing a Model](#6-choosing-a-model)
7. [Scrum (2020 Scrum Guide)](#7-scrum-2020-scrum-guide)
8. [Kanban](#8-kanban)
9. [DevOps and Continuous Delivery](#9-devops-and-continuous-delivery)
10. [Secure SDLC — NIST SSDF](#10-secure-sdlc--nist-ssdf)
11. [STLC Inside the SDLC](#11-stlc-inside-the-sdlc)
12. [Definition of Ready / Definition of Done](#12-definition-of-ready--definition-of-done)
13. [Quality Gates](#13-quality-gates)
14. [Environments and Promotion](#14-environments-and-promotion)
15. [Release Management and Versioning](#15-release-management-and-versioning)
16. [Maintenance (ISO/IEC/IEEE 14764)](#16-maintenance-isoiecieee-14764)
17. [Retirement and Decommissioning](#17-retirement-and-decommissioning)
18. [Roles and RACI](#18-roles-and-raci)
19. [Artifacts by Phase](#19-artifacts-by-phase)
20. [Scenario Playbooks](#20-scenario-playbooks)
21. [AI in the SDLC](#21-ai-in-the-sdlc)
22. [Anti-Patterns](#22-anti-patterns)
23. [SDLC Checklists](#23-sdlc-checklists)
24. [References](#24-references)

---

## 1. What the SDLC Is

The **Software Development Life Cycle** is the structured set of processes an organisation uses to **conceive, specify, design, build, verify, release, operate, maintain, and retire** software.

An SDLC answers five questions for every change:

```text
WHY    → What problem, for whom, measured how?          (Discovery, Requirements)
WHAT   → What exactly will we build?                     (Requirements, Design)
HOW    → How will we build and verify it?                (Design, Build, Test)
WHEN   → How does it reach users safely?                 (Release, Deploy)
THEN   → How do we run, support, evolve, and retire it?  (Operate, Maintain, Retire)
```

### SDLC vs STLC vs ALM vs DevOps

| Term | Scope |
|---|---|
| **SDLC** | Whole life of the software — concept to retirement |
| **STLC** | The testing life cycle that runs *inside* the SDLC (§11, chapter 08) |
| **Secure SDLC** | SDLC with security practices embedded in every phase (§10, chapter 09) |
| **ALM** (Application Lifecycle Management) | The tooling and governance that manages the SDLC (issue trackers, repos, pipelines) |
| **DevOps** | Culture + practices that unify development and operations to shorten the cycle safely (§9, chapter 16) |

---

## 2. ISO/IEC/IEEE 12207:2026 — The Process Standard

**ISO/IEC/IEEE 12207** is the international standard for software life cycle processes. The current edition, **ISO/IEC/IEEE 12207:2026**, was published in **April 2026**. It technically revises and replaces the 2017 edition. Among the changes, Annex D on model-based systems and software engineering (MBSSE) was revised. The companion standard for systems is **ISO/IEC/IEEE 15288**; which one you apply depends on whether the system-of-interest is primarily software or a wider system.

Key ideas:

- The standard defines **processes** (purpose, outcomes, activities, tasks). It does **not** prescribe a sequence or a life-cycle model; you tailor and order the processes to fit Waterfall, Agile, DevOps, or hybrid.
- Processes are organised into **four process groups**:

```text
┌──────────────────────────────────────────────────────────────┐
│ 1. AGREEMENT PROCESSES                                       │
│    Acquisition · Supply                                      │
├──────────────────────────────────────────────────────────────┤
│ 2. ORGANIZATIONAL PROJECT-ENABLING PROCESSES                 │
│    Life cycle model management · Infrastructure management · │
│    Portfolio management · Human resource management ·       │
│    Quality management · Knowledge management                 │
├──────────────────────────────────────────────────────────────┤
│ 3. TECHNICAL MANAGEMENT PROCESSES                            │
│    Project planning · Project assessment and control ·       │
│    Decision management · Risk management ·                   │
│    Configuration management · Information management ·      │
│    Measurement · Quality assurance                           │
├──────────────────────────────────────────────────────────────┤
│ 4. TECHNICAL PROCESSES                                       │
│    Business or mission analysis                              │
│    Stakeholder needs and requirements definition             │
│    System/software requirements definition                   │
│    Architecture definition                                   │
│    Design definition                                         │
│    System analysis                                           │
│    Implementation                                            │
│    Integration                                               │
│    Verification                                              │
│    Transition                                                │
│    Validation                                                │
│    Operation                                                 │
│    Maintenance                                               │
│    Disposal                                                  │
└──────────────────────────────────────────────────────────────┘
```

> The process names above follow the structure established in the 2017 edition and retained in the four-group structure of 2026. Before a formal compliance claim, verify exact process names and counts against your purchased copy of the 2026 standard.

### Mapping 12207 technical processes to this handbook

| 12207 technical process | Handbook chapter |
|---|---|
| Business or mission analysis | 03 §2–3 |
| Stakeholder needs & requirements definition | 03 |
| System/software requirements definition | 03 |
| Architecture definition | 04 |
| Design definition | 05 |
| System analysis | 04, 19 |
| Implementation | 06, 12–15 |
| Integration | 11, 16 |
| Verification | 08 |
| Transition | 16, 21 |
| Validation | 08 (acceptance), 03 §14 |
| Operation | 18, 21 |
| Maintenance | §16 here, 21 |
| Disposal | §17 here |

### Tailoring

12207 explicitly allows **tailoring** — removing, adding, or adapting processes for the project's context. Document tailoring decisions:

```markdown
## SDLC Tailoring Record — Project Atlas
| Process | Tailoring | Rationale | Approved by |
|---|---|---|---|
| Acquisition | Not applicable | Built in-house | Eng Director |
| Architecture definition | Lightweight (ADRs + C4 L1/L2 only) | Single team, < 6 months | Tech Lead |
| Verification | Extended (independent pentest) | Handles payment data | CISO |
```

---

## 3. Life-Cycle Stages vs Processes

**Stages** describe *where the product is in its life*; **processes** describe *what work is done*. ISO/IEC/IEEE 24748-1 uses these generic stages:

| Stage | Purpose | Main processes active |
|---|---|---|
| **Concept** | Explore needs, feasibility, options | Business analysis, stakeholder needs, risk |
| **Development** | Specify, design, build, verify | Requirements → design → implementation → integration → verification |
| **Production** | (for software) build and package releases | Configuration management, release |
| **Utilization** | Users use it in operation | Operation, validation, measurement |
| **Support** | Keep it working and evolving | Maintenance, operation |
| **Retirement** | Withdraw and dispose | Disposal |

In Agile/DevOps, a product is in **development, utilization, and support simultaneously** — each increment runs a mini life cycle.

---

## 4. The Practical SDLC — 9 Phases with Gates

This is the handbook's reference SDLC. It works for sequential and iterative delivery. In Agile, phases 1–6 compress into each sprint or each pull request; the **gates still apply**, just at smaller scale.

```text
 0 Discovery → 1 Requirements → 2 Architecture & Design → 3 Planning
   → 4 Build → 5 Verify (STLC) → 6 Release → 7 Operate & Monitor
   → 8 Maintain & Evolve → (9 Retire)
          ▲                                            │
          └──────────────── feedback ──────────────────┘
```

### Phase 0 — Discovery / Inception

| Item | Detail |
|---|---|
| **Goal** | Confirm the problem is worth solving |
| **Inputs** | Business idea, customer feedback, incident trends, strategy, regulations |
| **Activities** | Problem statement, user research, market/competitor scan, feasibility (technical, legal, financial), rough sizing, success metrics |
| **Outputs** | Problem statement, opportunity brief / business case, success metrics (OKRs), initial risk list |
| **Exit gate** | Sponsor approves funding for requirements & design |
| **Owner** | Product manager + tech lead |
| **STLC** | Testability concerns raised; quality goals identified |
| **Security** | Data classification; regulatory scope (GDPR, DPDP, PCI DSS, HIPAA) |

### Phase 1 — Requirements

| Item | Detail |
|---|---|
| **Goal** | Agree what the system must do and how well |
| **Activities** | Elicitation, analysis, specification (user stories / SRS), NFRs mapped to ISO/IEC 25010, prioritisation, validation |
| **Outputs** | Product backlog or SRS, acceptance criteria, NFR list with measurable targets, traceability matrix seed |
| **Exit gate** | Requirements reviewed and baselined; top-priority items meet Definition of Ready |
| **STLC** | **Requirement analysis for testing** — testability review, identify test conditions, acceptance tests drafted (ATDD/BDD) |
| **Security** | Security & privacy requirements, abuse cases |
| **Details** | Chapter 03 |

### Phase 2 — Architecture & Design

| Item | Detail |
|---|---|
| **Goal** | Decide structure and key mechanisms that satisfy requirements and quality attributes |
| **Activities** | Context & container diagrams (C4), ADRs, data model, API contracts, technology selection, threat modelling, capacity estimates |
| **Outputs** | Architecture description, ADRs, API specs (OpenAPI/AsyncAPI/proto), data model, threat model |
| **Exit gate** | Architecture/design review passed; open risks have owners |
| **STLC** | **Test planning** — test strategy, levels, environments, data, tools; contract tests defined from API specs |
| **Security** | Threat model (STRIDE), security architecture review |
| **Details** | Chapters 04, 05, 09, 10, 11 |

### Phase 3 — Planning

| Item | Detail |
|---|---|
| **Goal** | Plan delivery: scope, sequence, capacity, risks |
| **Activities** | Release/iteration planning, estimation, dependency mapping, risk register, staffing |
| **Outputs** | Roadmap, release plan, sprint backlog, risk register, communication plan |
| **Exit gate** | Team commits to an iteration or milestone |
| **STLC** | Test estimation; test environment and data provisioning scheduled |
| **Details** | Chapter 03 §15–19 |

### Phase 4 — Build (Implementation)

| Item | Detail |
|---|---|
| **Goal** | Produce working, tested, reviewed increments |
| **Activities** | Coding to standards, unit tests, code review, static analysis, dependency management, CI on every commit, feature flags |
| **Outputs** | Merged code, passing pipeline, build artifacts (images, packages), SBOM |
| **Exit gate** | Definition of Done met per item |
| **STLC** | **Test analysis, design, implementation** — unit/integration tests written; automation suites updated |
| **Security** | SAST, secret scanning, SCA (dependency vulnerabilities), secure code review |
| **Details** | Chapters 06, 07, 12–15, 37, 42 |

### Phase 5 — Verify (Testing)

| Item | Detail |
|---|---|
| **Goal** | Gain evidence the increment meets requirements and quality targets |
| **Activities** | Integration, system, end-to-end, performance, security, accessibility, exploratory, UAT |
| **Outputs** | Test reports, defect reports, coverage, performance results, sign-off |
| **Exit gate** | Exit criteria met (see §13) |
| **STLC** | **Test execution, monitoring & control** |
| **Security** | DAST, penetration test (risk-based), IaC/container scanning |
| **Details** | Chapters 08, 09, 19, 40 |

### Phase 6 — Release & Deploy

| Item | Detail |
|---|---|
| **Goal** | Deliver to users safely and reversibly |
| **Activities** | Change approval (risk-based), progressive delivery (canary/blue-green), DB migrations (expand/contract), release notes, comms |
| **Outputs** | Production deployment, release notes, change record, tagged version |
| **Exit gate** | Post-deploy verification green (smoke tests, SLOs, error rates) |
| **STLC** | Smoke/sanity tests in production; **test completion** report |
| **Details** | Chapters 16, 17 |

### Phase 7 — Operate & Monitor

| Item | Detail |
|---|---|
| **Goal** | Keep the service healthy and learn from real use |
| **Activities** | Monitoring, alerting, on-call, incident response, capacity management, cost management, product analytics |
| **Outputs** | SLO reports, incident postmortems, usage analytics |
| **STLC** | Shift-right testing: synthetic monitoring, canary analysis, chaos experiments, A/B tests |
| **Details** | Chapters 18, 20, 21 |

### Phase 8 — Maintain & Evolve

| Item | Detail |
|---|---|
| **Goal** | Fix, adapt, improve, and prevent |
| **Activities** | Bug fixes, dependency/runtime upgrades, OS patching, refactoring, performance work, feature iterations |
| **Outputs** | Patch/minor releases, updated docs, reduced tech debt |
| **STLC** | **Regression and maintenance testing**, impact analysis |
| **Details** | §16 here, chapter 21 |

### Phase 9 — Retire

| Item | Detail |
|---|---|
| **Goal** | Withdraw the system without harming users, data, or compliance |
| **Details** | §17 here |

---

## 5. Life-Cycle Models

### 5.1 Waterfall (sequential)

```text
Requirements → Design → Implementation → Verification → Deployment → Maintenance
```

| Strengths | Weaknesses |
|---|---|
| Clear milestones and documents | Late feedback; integration risk at the end |
| Fits fixed-scope contracts and heavy regulation | Expensive to change requirements |
| Easy to audit | Users see value late |

**Use when:** requirements are stable and well understood, change is expensive (hardware, safety-critical), or contracts demand phase sign-offs.

### 5.2 V-Model

Each development phase has a matching verification phase — the backbone of classic STLC thinking.

```text
Requirements ────────────────────────────── Acceptance testing
   System design ──────────────────────── System testing
      Architecture design ─────────── Integration testing
         Module design ────────── Unit testing
                      Implementation
```

**Use when:** safety-critical or regulated domains (medical, automotive, aerospace) where verification evidence per level is mandatory.

### 5.3 Iterative and Incremental

- **Incremental:** deliver the system in slices of functionality.
- **Iterative:** repeatedly refine the same functionality.
- Most modern methods are both.

### 5.4 Spiral (Boehm)

Risk-driven cycles: **determine objectives → identify & resolve risks (prototype) → develop & test → plan next iteration.** Good for large, high-risk, novel systems.

### 5.5 Agile

The **Manifesto for Agile Software Development** (2001) values individuals and interactions, working software, customer collaboration, and responding to change — over processes and tools, comprehensive documentation, contract negotiation, and following a plan. The items on the right still have value; the items on the left are valued more. It is backed by **12 principles** (https://agilemanifesto.org/principles.html).

Common Agile methods:

| Method | Core mechanics | Best for |
|---|---|---|
| **Scrum** | Fixed-length Sprints, three accountabilities, five events, three artifacts | Product teams with evolving requirements |
| **Kanban** | Visualise flow, limit WIP, manage flow | Ops, support, platform teams, continuous flow |
| **Extreme Programming (XP)** | TDD, pair programming, continuous integration, small releases, refactoring | Engineering-excellence practices inside any method |
| **Lean software development** | Eliminate waste, amplify learning, deliver fast | Process improvement |
| **Scaled frameworks** (SAFe, LeSS, Nexus, Scrum@Scale) | Coordinating many teams | Large programmes — adopt carefully |

### 5.6 DevOps / Continuous Delivery

Extends Agile through to production: every change flows through an automated pipeline to a releasable (or released) state, with fast feedback from production. See §9.

### 5.7 Hybrid ("Water-Scrum-Fall")

Upfront phase-gated discovery and funding, Agile delivery, phase-gated release. Very common in enterprises. Acceptable when gates are **risk-based and lightweight**; harmful when the release gate batches months of work.

---

## 6. Choosing a Model

### 6.1 Selection matrix

| Factor | Waterfall / V | Spiral | Scrum | Kanban | Continuous delivery |
|---|:---:|:---:|:---:|:---:|:---:|
| Requirements stable | ✅ | ➖ | ➖ | ➖ | ➖ |
| Requirements evolving | ❌ | ✅ | ✅ | ✅ | ✅ |
| High novelty / technical risk | ❌ | ✅ | ✅ | ➖ | ✅ |
| Strict regulatory evidence | ✅ | ✅ | ✅* | ✅* | ✅* |
| Unplanned work dominates (ops/support) | ❌ | ❌ | ➖ | ✅ | ✅ |
| Fixed-price, fixed-scope contract | ✅ | ➖ | ➖ | ❌ | ➖ |
| Web/SaaS with frequent releases | ❌ | ❌ | ✅ | ✅ | ✅ |
| Embedded / hardware-coupled | ✅ | ✅ | ✅* | ➖ | ➖ |

`*` with documented traceability and evidence generated by the pipeline.

### 6.2 Decision guide

```text
Is most work unplanned (tickets, incidents, ops)?        → Kanban
Is the product evolving and user feedback essential?     → Scrum (+ XP practices) + CD
Is it safety-critical with mandatory per-level evidence? → V-Model (often with Agile inside)
Is it large, novel, and high-risk?                       → Spiral / risk-driven iterations
Is scope fixed by contract and truly stable?             → Waterfall with incremental delivery inside
In all cases                                             → CI, automated tests, and DORA metrics
```

---

## 7. Scrum (2020 Scrum Guide)

The authoritative definition of Scrum is the **Scrum Guide** by Ken Schwaber and Jeff Sutherland. The **November 2020** revision remains the current official version (as of this writing there is no newer official revision; independently published "expansion packs" exist but are not official revisions). Scrum is free and intentionally incomplete — it is a framework, not a full process.

Official: https://scrumguides.org/

### 7.1 Theory, pillars, values

- **Empiricism** and **lean thinking**.
- Three pillars: **Transparency, Inspection, Adaptation**.
- Five values: **Commitment, Focus, Openness, Respect, Courage**.

### 7.2 Scrum Team — three accountabilities

| Accountability | Owns |
|---|---|
| **Product Owner** | Maximising value; the Product Backlog and Product Goal |
| **Scrum Master** | Scrum effectiveness; coaching team and organisation; removing impediments |
| **Developers** | Creating a usable Increment every Sprint; the Sprint Backlog; adhering to the Definition of Done |

The 2020 guide uses **accountabilities** rather than "roles", describes the team as **self-managing**, and emphasises one Scrum Team (no separate "Development Team"). Teams are typically **10 or fewer** people.

### 7.3 Five events

| Event | Timebox (for a one-month Sprint; shorter Sprints → usually shorter) | Purpose |
|---|---|---|
| **The Sprint** | ≤ 1 month | Container for all other events |
| **Sprint Planning** | ≤ 8 hours | Why is this Sprint valuable? What can be Done? How will the work get done? |
| **Daily Scrum** | 15 minutes | Inspect progress toward the Sprint Goal; adapt the plan |
| **Sprint Review** | ≤ 4 hours | Inspect the outcome with stakeholders; adapt the Product Backlog |
| **Sprint Retrospective** | ≤ 3 hours | Improve quality and effectiveness |

### 7.4 Three artifacts and their commitments

| Artifact | Commitment |
|---|---|
| **Product Backlog** | **Product Goal** |
| **Sprint Backlog** | **Sprint Goal** |
| **Increment** | **Definition of Done** |

### 7.5 Engineering add-ons Scrum does not define (but production teams need)

- CI/CD, trunk-based development, automated testing (XP practices)
- Backlog refinement with Definition of Ready (team convention, not part of Scrum)
- Architecture runway and ADRs
- On-call and operational work (often a separate Kanban lane)

---

## 8. Kanban

Kanban manages **flow**. Core practices (Kanban Guide by Daniel Vacanti & Prateek Singh; Kanban Method by David J. Anderson):

1. **Visualise** the workflow (board with explicit columns and policies).
2. **Limit work in progress (WIP)**.
3. **Actively manage** items in progress.
4. **Improve** the workflow based on flow metrics.

### Flow metrics

| Metric | Meaning |
|---|---|
| **WIP** | Items started but not finished |
| **Throughput** | Items finished per unit time |
| **Cycle time** | Time from start to finish of an item |
| **Work item age** | How long an in-progress item has been in progress |

**Little's Law** (stable system): `Average cycle time = Average WIP ÷ Average throughput` → lowering WIP shortens cycle time.

### Example board with policies

```text
| Backlog | Ready (DoR met) | In Dev (WIP 4) | Review (WIP 3) | Test (WIP 3) | Deploy | Done |
Policies:
- Ready: acceptance criteria + estimate + no blocking dependency
- Review: ≥1 approval, CI green
- Test: automated suite green + exploratory notes attached
- Expedite lane: Sev1/Sev2 incidents only, max 1 item
```

---

## 9. DevOps and Continuous Delivery

### 9.1 Core practices

| Practice | Description | Chapter |
|---|---|---|
| Version control for everything | Code, config, IaC, pipelines, docs | 07, 42 |
| Trunk-based development | Short-lived branches, merge at least daily | 07 |
| Continuous integration | Build + test on every commit | 16 |
| Continuous delivery | Every passing build is releasable | 16 |
| Continuous deployment | Every passing build is released automatically | 16 |
| Infrastructure as Code | Reproducible environments | 17 |
| Progressive delivery | Feature flags, canary, blue-green | 16 |
| Observability | Logs, metrics, traces, SLOs | 18 |
| Blameless postmortems | Learn from incidents | 20, 21 |
| Loosely coupled architecture | Teams deploy independently | 04 |

### 9.2 Delivery pipeline (reference)

```text
commit → build → unit tests → SAST/secrets/SCA → package (image + SBOM + signature)
  → deploy to test → integration/contract tests → deploy to staging
  → e2e + performance + DAST → approval (risk-based) → canary in prod
  → automated analysis (errors, latency, SLO burn) → full rollout → post-deploy checks
```

### 9.3 Measure with DORA's five metrics

| Throughput | Instability |
|---|---|
| Change lead time | Change fail rate |
| Deployment frequency | Deployment rework rate |
| Failed deployment recovery time | |

Definitions and rules: chapter 01 §17. Official guide: https://dora.dev/guides/dora-metrics/

---

## 10. Secure SDLC — NIST SSDF

NIST notes that few SDLC models address security in detail, so the **Secure Software Development Framework (SSDF, SP 800-218)** provides high-level practices to **integrate into whatever SDLC you use**. Status: v1.1 final (Feb 2022); **v1.2 (SP 800-218 Rev. 1)** published as an initial public draft on 17 Dec 2025 (comments closed 30 Jan 2026) — check https://csrc.nist.gov/projects/ssdf for the final version.

### 10.1 SSDF practice groups mapped to SDLC phases

| SSDF group | Example practices | SDLC phases |
|---|---|---|
| **PO** Prepare the Organization | Define security requirements, roles & training, secure toolchains, criteria for security checks | 0, 1, 3 + ongoing |
| **PS** Protect the Software | Protect code from unauthorised access/tampering, provide integrity verification for releases, archive releases | 4, 6 |
| **PW** Produce Well-Secured Software | Secure design & threat modelling, reuse vetted components, secure coding, code review/analysis, testing, secure default configuration | 2, 4, 5 |
| **RV** Respond to Vulnerabilities | Identify and confirm vulnerabilities, assess/prioritise/remediate, analyse root causes | 7, 8 |

### 10.2 Security activities per phase

| Phase | Mandatory security activity |
|---|---|
| 0 Discovery | Data classification; regulatory scoping |
| 1 Requirements | Security & privacy requirements (e.g. OWASP ASVS level), abuse cases |
| 2 Design | Threat model (STRIDE), security design review |
| 4 Build | SAST, secrets scanning, dependency (SCA) scanning, secure code review |
| 5 Verify | DAST, IaC/container scans, risk-based penetration test |
| 6 Release | Signed artifacts, SBOM, provenance (SLSA), change approval |
| 7 Operate | Runtime protection, logging/alerting, vulnerability intake (security.txt, disclosure policy) |
| 8 Maintain | Patch SLAs, dependency upgrades, root-cause analysis |

Full detail: chapter **09**.

---

## 11. STLC Inside the SDLC

The **Software Testing Life Cycle** is the sequence of testing activities that runs alongside development. This section shows **where** each STLC activity sits in the SDLC; chapter **08** contains the full STLC with templates, techniques, and tooling.

### 11.1 ISTQB test process (CTFL v4.0)

The ISTQB Foundation Level syllabus (v4.0) describes the test process as a group of activities which may look sequential but are often performed **iteratively or in parallel**:

```text
1. Test planning
2. Test monitoring and control
3. Test analysis          → "what to test" (test conditions)
4. Test design            → "how to test" (test cases, data, coverage)
5. Test implementation    → testware ready (scripts, data, environments, suites)
6. Test execution         → run, compare, log, report defects
7. Test completion        → summary report, lessons learned, archive testware
```

### 11.2 Classic STLC phases (industry vocabulary)

Many organisations describe the STLC with six phases. They map onto the ISTQB activities:

| Classic STLC phase | ISTQB activity | Entry criteria | Exit criteria / deliverables |
|---|---|---|---|
| **1. Requirement analysis** | Test analysis | Requirements / user stories available | Testable requirements list, RTM seed, clarification log |
| **2. Test planning** | Test planning | Requirements analysed | Test plan / strategy, estimates, tool & environment decisions |
| **3. Test case development** | Test design + implementation | Approved test plan | Test cases, automation scripts, test data |
| **4. Test environment setup** | Test implementation | Environment design, data needs | Ready environment, smoke test passed |
| **5. Test execution** | Test execution + monitoring & control | Test cases + environment ready | Execution results, defect reports, updated RTM |
| **6. Test cycle closure** | Test completion | Execution complete or exit criteria met | Test summary report, metrics, lessons learned |

### 11.3 SDLC ↔ STLC mapping

| SDLC phase | STLC activity | Key output |
|---|---|---|
| 0 Discovery | Quality goals, risk identification | Quality attribute priorities |
| 1 Requirements | Requirement analysis (testability review), acceptance criteria / BDD scenarios | Testable acceptance criteria, RTM |
| 2 Architecture & Design | Test strategy & planning, contract tests from API specs | Test strategy, test environment design |
| 3 Planning | Test estimation, environment/data scheduling | Test plan |
| 4 Build | Unit + component tests (TDD), automation implementation | Automated suites in CI |
| 5 Verify | Integration, system, E2E, NFR testing, UAT; monitoring & control | Test reports, defect metrics |
| 6 Release | Smoke tests, release sign-off, test completion | Test summary report |
| 7 Operate | Shift-right: synthetic monitoring, canary analysis, chaos | Production quality signals |
| 8 Maintain | Regression & maintenance testing, impact analysis | Updated regression suite |

### 11.4 Shift-left and shift-right

```text
SHIFT-LEFT (prevent)                         SHIFT-RIGHT (detect & learn)
Requirement reviews                          Canary analysis
BDD / ATDD                                   Feature-flag experiments
TDD, static analysis                         Synthetic monitoring
Contract tests                               Chaos engineering
Threat modelling                             Real-user monitoring
```

### 11.5 Test levels by model

| Model | Where test levels sit |
|---|---|
| Waterfall / V | Each level has its own phase and sign-off |
| Scrum | All levels inside each Sprint; DoD requires them |
| Continuous delivery | Levels are pipeline stages; fast tests first |

---

## 12. Definition of Ready / Definition of Done

### 12.1 Definition of Ready (team convention)

```markdown
A backlog item is READY when:
- [ ] Clear user value and "so that" outcome
- [ ] Acceptance criteria written (Given/When/Then where useful)
- [ ] NFRs identified (performance, security, accessibility) or marked n/a
- [ ] Dependencies identified and unblocked
- [ ] UX designs attached if UI is involved
- [ ] Sized by the team; small enough to finish within one Sprint (ideally a few days)
- [ ] Test approach agreed (what to automate, what data is needed)
```

> Do not let DoR become a waterfall gate. It exists to reduce mid-sprint surprises, not to block conversation.

### 12.2 Definition of Done (enterprise baseline)

```markdown
An increment is DONE when:
Code & review
- [ ] Code follows coding standards (chapter 06); linters pass
- [ ] Peer reviewed and approved (≥1 reviewer; ≥2 for high-risk areas)
- [ ] No new critical/high static-analysis findings
Testing
- [ ] Unit tests written and passing; coverage not reduced on changed code
- [ ] Integration/contract tests updated and passing
- [ ] Acceptance criteria verified (automated where practical)
- [ ] Accessibility checks for UI changes
Security
- [ ] Secrets scan clean; dependency scan has no unaccepted critical/high vulns
- [ ] Threat model updated if trust boundaries changed
Operability
- [ ] Logs, metrics, traces added for new paths; alerts/dashboards updated
- [ ] Feature flag in place for risky changes; rollback path known
- [ ] Database migrations backward compatible (expand/contract)
Documentation
- [ ] API docs / README / runbooks / ADRs updated
- [ ] Release notes entry written
Delivery
- [ ] Merged to main; deployed to staging; smoke tests green
- [ ] Product Owner accepted
```

---

## 13. Quality Gates

Gates are **automated where possible**, risk-based where human approval is needed.

| Gate | When | Automated checks | Human checks |
|---|---|---|---|
| **G0 Idea → Discovery** | Funding | — | Sponsor approval |
| **G1 Requirements baseline** | Before design | Lint/consistency of specs | Requirements review (PO, tech lead, QA, security) |
| **G2 Design approval** | Before major build | API spec lint, schema checks | Architecture & threat model review |
| **G3 Merge** | Every PR | Build, unit tests, lint, SAST, secrets, SCA, coverage delta | Code review |
| **G4 Promote to staging** | After merge | Integration + contract tests, container/IaC scan | — |
| **G5 Release** | Before prod | E2E, performance budget, DAST, migration dry run | Risk-based change approval |
| **G6 Post-deploy** | During rollout | Canary metrics, SLO burn, error rates, smoke tests | On-call watches |
| **G7 Release closure** | After rollout | DORA metrics captured | Test completion report, retro |

### Example exit criteria for G5 (release)

```text
- 100% of P1 test cases executed; ≥ 95% of all planned cases executed
- 0 open Critical / High defects (or accepted with written risk sign-off)
- p95 latency within budget under expected peak load
- No unaccepted Critical/High vulnerabilities
- Rollback tested in staging
- Release notes and runbook updated
```

---

## 14. Environments and Promotion

| Environment | Purpose | Data | Deploy trigger | Who has access |
|---|---|---|---|---|
| **Local** | Developer inner loop | Synthetic / seeded | Manual | Developer |
| **CI (ephemeral)** | Automated tests per commit | Synthetic | Every push | Pipeline |
| **Preview / PR env** | Review a branch end-to-end | Synthetic | PR opened | Team, reviewers |
| **Dev / Integration** | Integrate services | Synthetic | Merge to main | Engineering |
| **QA / Test** | Structured testing | Synthetic / masked | Promotion | Engineering, QA |
| **Staging / Pre-prod** | Production-like rehearsal | Masked / synthetic, prod-like volume | Release candidate | Restricted |
| **Production** | Real users | Real | Approved release | Highly restricted, audited |
| **DR** | Disaster recovery | Replicated | Failover | Ops |

Rules:

```text
🔴 Build once, promote the same artifact (same image digest) through all environments.
🔴 Configuration via environment variables / config service, never rebuilt per env.
🔴 Real personal data MUST NOT be copied to non-production without masking/anonymisation and approval.
🟠 Staging mirrors production topology (versions, networking, TLS, flags).
🟠 Environments are created via IaC and can be rebuilt from scratch.
```

Setup references: **39-vps-enterprise-setup-guide.md** (single-server), chapter **17** (cloud).

---

## 15. Release Management and Versioning

### 15.1 Semantic Versioning 2.0.0

`MAJOR.MINOR.PATCH` (https://semver.org/)

| Increment | When |
|---|---|
| **MAJOR** | Incompatible API changes |
| **MINOR** | Backward-compatible functionality |
| **PATCH** | Backward-compatible bug fixes |
| Pre-release | `1.4.0-rc.1`, `2.0.0-beta.3` |
| Build metadata | `1.4.0+20261002.sha.a1b2c3` |

Alternatives: **CalVer** (`2026.10.1`) for apps released on a schedule; internal services often use **commit SHA + build number** and reserve SemVer for public APIs and libraries.

### 15.2 Release strategies

| Strategy | How | Risk | Rollback |
|---|---|---|---|
| Big bang | Everyone at once | High | Redeploy previous |
| Rolling | Replace instances gradually | Medium | Roll back remaining |
| Blue-green | Switch traffic between two full stacks | Low | Switch back |
| Canary | Small % of traffic first, then expand | Low | Shift traffic back |
| Feature flags / dark launch | Deploy code off, enable per cohort | Lowest | Toggle off |
| Ring deployment | Internal → beta → region → global | Low | Halt rings |

### 15.3 Release checklist

```markdown
- [ ] Version tagged (`git tag -s v1.8.0`) and changelog updated (Keep a Changelog format)
- [ ] Artifacts built once, signed, SBOM attached
- [ ] DB migrations: expand step deployed first; contract step scheduled after
- [ ] Feature flags configured per environment
- [ ] Release notes for users and internal support
- [ ] Monitoring dashboard and alert links in the release ticket
- [ ] On-call informed; deployment window respects freeze calendar
- [ ] Rollback procedure verified
```

Git tagging and signing: **42-Git-GitHub-GitLab-Bitbucket-Setup.md**.

---

## 16. Maintenance (ISO/IEC/IEEE 14764)

ISO/IEC/IEEE 14764 (software maintenance) classifies modifications into four types:

| Type | Trigger | Example | Planned? |
|---|---|---|---|
| **Corrective** | Fix discovered faults | Null pointer in checkout | Reactive |
| **Adaptive** | Keep usable in a changed environment | Upgrade to new OS / runtime / API version | Proactive |
| **Perfective** | Improve performance, maintainability, or add enhancements | Refactor, speed up search | Proactive |
| **Preventive** | Fix latent faults before they become failures | Add input validation, remove deprecated library | Proactive |

### Maintenance rhythm (example)

| Cadence | Activity |
|---|---|
| Continuous | Dependency update bots (Dependabot/Renovate), security patches per SLA |
| Weekly | Triage bugs, review flaky tests, review error dashboards |
| Monthly | OS/base-image updates, minor framework upgrades |
| Quarterly | Runtime LTS review, tech-debt review, DR test, access review |
| Yearly | Architecture review, cost review, licence review, retirement candidates |

### Vulnerability remediation SLA (example — adjust to policy)

| Severity | Internet-facing | Internal |
|---|---|---|
| Critical (actively exploited) | 24–72 h | 7 days |
| Critical | 7 days | 14 days |
| High | 30 days | 30–60 days |
| Medium | 90 days | 90 days |
| Low | Next planned release | Best effort |

---

## 17. Retirement and Decommissioning

```markdown
## Decommission Plan — <system>
1. Decision: ADR documenting why, replacement, and date
2. Stakeholders: users, support, legal, finance, dependent teams notified
3. Deprecation: announce with timeline; add deprecation headers/warnings (e.g. Sunset header, RFC 8594)
4. Migration: data and users moved to replacement; dual-run period if needed
5. Traffic: monitor until usage reaches zero (or the agreed threshold)
6. Data: retain per retention policy; export/archive; delete per law (GDPR/DPDP erasure duties)
7. Access: revoke credentials, API keys, service accounts, DNS records, certificates
8. Infrastructure: destroy via IaC; verify billing stops
9. Code: archive repository read-only; update service catalogue
10. Docs: mark runbooks/docs as archived; note in knowledge base
```

---

## 18. Roles and RACI

**R** Responsible · **A** Accountable · **C** Consulted · **I** Informed

| Activity | Product Mgr / PO | Tech Lead / Architect | Developers | QA / SDET | Security | SRE / Ops | Eng Manager |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| Problem & business case | **A/R** | C | I | I | C | I | C |
| Requirements & backlog | **A/R** | C | C | C | C | C | I |
| NFRs & quality targets | A | **R** | C | C | C | C | I |
| Architecture & ADRs | C | **A/R** | C | C | C | C | I |
| Threat model | I | R | C | C | **A** | C | I |
| Implementation | I | A | **R** | C | C | I | I |
| Test strategy & plan | C | C | C | **A/R** | C | C | I |
| Code review | — | A | **R** | C | C | — | — |
| Release approval | **A** | R | C | C | C | C | I |
| Deployment | I | A | R | I | I | **R** | I |
| Incident response | I | C | R | I | C | **A/R** | I |
| Postmortem | I | R | R | C | C | **A** | C |
| Process & DORA metrics | C | C | C | C | I | C | **A/R** |

---

## 19. Artifacts by Phase

| Phase | Artifact | Format / tool | Lives in |
|---|---|---|---|
| 0 | Opportunity brief, OKRs | Doc | Wiki / docs repo |
| 1 | Backlog, user stories, SRS, NFRs, RTM | Issue tracker, Markdown | Tracker + `docs/requirements/` |
| 2 | C4 diagrams, ADRs, API specs, data model, threat model | Markdown, Mermaid/PlantUML, OpenAPI, AsyncAPI | `docs/architecture/`, `api/` |
| 3 | Roadmap, release plan, risk register | Tracker, Markdown | Tracker / wiki |
| 4 | Source, tests, pipeline config, IaC, SBOM | Git | Repo |
| 5 | Test plan, cases, reports, defects | Test management tool / repo | Tracker / CI artifacts |
| 6 | Release notes, changelog, change record, signed artifacts | Markdown, registry | Repo, registry |
| 7 | Dashboards, alerts, SLO reports, postmortems | Observability stack, Markdown | Observability tool, `docs/postmortems/` |
| 8 | Patch notes, upgrade records | Markdown | Repo |
| 9 | Decommission ADR, archive record | Markdown | Wiki |

Suggested repository docs layout:

```text
docs/
├── adr/                 0001-record-architecture-decisions.md
├── architecture/        c4-context.md, c4-container.md
├── requirements/        prd.md, nfr.md, rtm.csv
├── runbooks/            service-x.md
├── postmortems/         2026-09-14-checkout-outage.md
└── testing/             test-strategy.md
```

---

## 20. Scenario Playbooks

### 20.1 Startup MVP (2–6 engineers, weeks to launch)

| Choice | Recommendation |
|---|---|
| Model | Kanban or 1-week Scrum + continuous deployment |
| Requirements | Lean PRD (1–2 pages) + user stories |
| Architecture | Modular monolith; ADRs for key choices |
| Testing | Unit + a few critical E2E paths; manual exploratory before launch |
| Security | Auth via proven provider, secrets manager, dependency scanning from day 1 |
| Ops | Managed services; error tracking + uptime monitoring + basic SLO |
| Watch out | Skipping backups and access control "for now" |

### 20.2 Enterprise / regulated (fintech, health, government)

| Choice | Recommendation |
|---|---|
| Model | Hybrid: phase-gated funding, Scrum delivery, risk-based release gate |
| Requirements | SRS aligned with ISO/IEC/IEEE 29148, full traceability |
| Architecture | Formal architecture review board, ISO/IEC/IEEE 42010-style descriptions |
| Testing | V-model evidence generated by CI; independent verification; UAT sign-off |
| Security | SSDF practices, threat models, annual pentest, SBOM, signed artifacts |
| Ops | Change management (ITIL 4), audit logs, DR drills |
| Watch out | Gates that batch months of work; evidence produced manually after the fact |

### 20.3 Mobile app

| Choice | Recommendation |
|---|---|
| Model | Scrum with release trains (e.g. every 1–2 weeks) |
| Release | Phased rollout in App Store / Play Console; staged percentages |
| Testing | Device matrix, UI tests, beta channels (TestFlight / Play internal testing) |
| Flags | Remote config for kill switches — store review delays hotfixes |
| Versioning | SemVer marketing version + monotonically increasing build number |
| Details | Chapter 14; IDE setup 35, 36 |

### 20.4 Legacy modernisation / migration

| Step | Practice |
|---|---|
| 1 | Characterisation tests around existing behaviour |
| 2 | Strangler-fig pattern — route slices to new system |
| 3 | Dual-run / shadow traffic comparison |
| 4 | Data migration rehearsals with reconciliation reports |
| 5 | Decommission old slices (§17) |

### 20.5 Adding an AI / LLM feature

| Phase | Extra activity |
|---|---|
| Requirements | Define task success metrics, evaluation dataset, safety/abuse cases, cost budget |
| Design | Model/provider selection ADR, prompt and context design, guardrails, fallback behaviour |
| Build | Prompt/version control, structured outputs, rate limiting |
| Verify | Offline evals (accuracy, groundedness), red-teaming (prompt injection), latency/cost tests |
| Release | Feature flag + small cohort; human feedback loop |
| Operate | Monitor quality drift, cost per request, refusal/error rates; re-run evals on model upgrades |

### 20.6 Internal platform / ops team

Kanban with explicit classes of service (expedite, fixed-date, standard, intangible), SLOs for platform APIs, and a published roadmap for consumers.

---

## 21. AI in the SDLC

| Phase | Useful AI assistance | Guardrail |
|---|---|---|
| Discovery | Summarise research, cluster feedback | Validate with real users |
| Requirements | Draft stories, find ambiguities, suggest edge cases | Humans approve; check against stakeholders |
| Design | Explore options, draft ADRs, review diagrams | Architect owns the decision |
| Build | Code generation, refactoring, test generation | Same review, tests, and security scans as human code |
| Verify | Generate test data and cases, triage failures | Verify generated tests actually assert behaviour |
| Release | Draft release notes | Human review |
| Operate | Summarise incidents, suggest runbook steps | No autonomous production changes without approval and audit |

The 2025 DORA research found that AI **amplifies** existing strengths and weaknesses and is linked to higher throughput but **lower stability** — strong automated testing, small batches, and solid platforms are prerequisites for AI-assisted delivery (chapter 01 §17, §19).

---

## 22. Anti-Patterns

| Anti-pattern | Symptom | Fix |
|---|---|---|
| **Cargo-cult Agile** | Ceremonies happen, nothing ships | Measure outcomes and DORA metrics; shorten feedback loops |
| **Mini-waterfall sprints** | Dev weeks 1–2, test week 3, "hardening sprint" | Test within each item; DoD includes tests |
| **Testing as a phase at the end** | Late defects, crunch | Shift-left, TDD/BDD, CI |
| **Big-batch releases** | Quarterly releases, risky nights | Continuous delivery, feature flags |
| **Gate theatre** | Approvals by people who can't assess risk | Risk-based, automated gates with evidence |
| **Environment drift** | "Works in staging" | IaC; build once, promote same artifact |
| **No ownership after launch** | Nobody on call, rotting dependencies | "You build it, you run it"; maintenance budget |
| **Documentation debt** | Tribal knowledge | Docs-as-code; DoD includes docs |
| **Security bolted on** | Pentest finds design flaws pre-launch | SSDF in every phase |
| **Velocity as a target** | Inflated estimates | Use velocity only for forecasting within a team |

---

## 23. SDLC Checklists

### New project kickoff
- [ ] Life-cycle model chosen and tailoring recorded (§2, §6)
- [ ] Problem statement, success metrics, and stakeholders documented
- [ ] Data classification and regulatory scope determined
- [ ] Definition of Ready and Definition of Done agreed
- [ ] Repository, branch protection, CI pipeline, and environments created
- [ ] Test strategy drafted (chapter 08)
- [ ] Threat model scheduled
- [ ] Observability baseline (logs, metrics, traces, error tracking)
- [ ] DORA metrics collection configured
- [ ] RACI agreed and published

### Per iteration / sprint
- [ ] Sprint Goal set; items meet DoR
- [ ] Daily inspection of progress toward goal
- [ ] Every item meets DoD before "done"
- [ ] Review with stakeholders; backlog updated
- [ ] Retrospective with at least one concrete improvement action

### Per release
- [ ] Release exit criteria met (§13)
- [ ] Artifacts signed, SBOM attached, version tagged
- [ ] Rollback verified; on-call aware
- [ ] Post-deploy verification complete
- [ ] Test completion report filed

### Per quarter
- [ ] DORA metrics trend reviewed
- [ ] Tech-debt and maintenance backlog reviewed
- [ ] Dependency/runtime currency reviewed
- [ ] DR test executed
- [ ] Process retrospective (is the SDLC still fit for purpose?)

---

## 24. References

### Standards and frameworks
- ISO/IEC/IEEE 12207:2026 — overview from ANSI: https://blog.ansi.org/ansi/iso-iec-ieee-12207-2026-software-life-cycle/
- ISO catalogue (12207, 15288, 24748-1, 14764, 15289): https://www.iso.org
- SWEBOK v4.0: https://www.computer.org/education/bodies-of-knowledge/software-engineering
- NIST SSDF project: https://csrc.nist.gov/projects/ssdf
- NIST SP 800-218 Rev. 1 (SSDF 1.2) draft: https://csrc.nist.gov/pubs/sp/800/218/r1/ipd
- ISTQB CTFL v4.0: https://istqb.org/certifications/certified-tester-foundation-level-ctfl-v4-0

### Methods
- Manifesto for Agile Software Development: https://agilemanifesto.org/
- Agile principles: https://agilemanifesto.org/principles.html
- The Scrum Guide (2020): https://scrumguides.org/
- The Kanban Guide: https://kanbanguides.org/
- Semantic Versioning 2.0.0: https://semver.org/
- Keep a Changelog: https://keepachangelog.com/
- RFC 8594 Sunset HTTP header: https://www.rfc-editor.org/rfc/rfc8594

### Delivery performance
- DORA metrics: https://dora.dev/guides/dora-metrics/
- DORA research and reports: https://dora.dev/research/
- 2025 DORA report announcement: https://cloud.google.com/blog/products/ai-machine-learning/announcing-the-2025-dora-report

---

**Previous:** [01 — Engineering Foundations](./01-engineering-foundations.md) · **Next:** [03 — Requirements & Planning](./03-requirements-and-planning.md)