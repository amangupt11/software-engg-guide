# 🧪 Testing & Quality — Production Engineering Guide (incl. full STLC)

> How to build justified confidence that software works, keeps working, and is fit for purpose: quality vs testing, ISTQB's seven testing principles, the **complete Software Testing Life Cycle (STLC)** with entry/exit criteria and templates, test levels and types, the test pyramid and its alternatives, test design techniques, test automation architecture, contract/integration testing with real dependencies, E2E, performance, security, accessibility, and mobile testing, test data and environments, defect management, quality metrics and gates, flaky tests, shift-right testing in production, and testing AI features.
>
> Related: [02 SDLC — STLC mapping §11](./02-software-development-lifecycle.md#11-stlc-inside-the-sdlc) · [03 Requirements (acceptance criteria, NFRs)](./03-requirements-and-planning.md) · [05 Design for testability](./05-software-design.md#14-designing-for-testability) · [06 Test code standards](./06-coding-standards.md#23-test-code-standards) · [09 Security testing](./09-security-engineering.md) · [19 Performance](./19-performance-and-scalability.md) · [40 Autocannon load testing](./40-autocannon_production_CLI.md)

---

## 📚 Table of Contents

1. [Quality, QA, QC, and Testing](#1-quality-qa-qc-and-testing)
2. [Standards and Bodies of Knowledge](#2-standards-and-bodies-of-knowledge)
3. [The Seven Testing Principles](#3-the-seven-testing-principles)
4. [The Software Testing Life Cycle (STLC)](#4-the-software-testing-life-cycle-stlc)
5. [STLC Phase 1 — Requirement Analysis](#5-stlc-phase-1--requirement-analysis)
6. [STLC Phase 2 — Test Planning](#6-stlc-phase-2--test-planning)
7. [STLC Phase 3 — Test Case Development](#7-stlc-phase-3--test-case-development)
8. [STLC Phase 4 — Test Environment Setup](#8-stlc-phase-4--test-environment-setup)
9. [STLC Phase 5 — Test Execution, Monitoring and Control](#9-stlc-phase-5--test-execution-monitoring-and-control)
10. [STLC Phase 6 — Test Cycle Closure](#10-stlc-phase-6--test-cycle-closure)
11. [STLC in Agile and CI/CD](#11-stlc-in-agile-and-cicd)
12. [Test Levels](#12-test-levels)
13. [Test Types](#13-test-types)
14. [Test Strategy Shapes — Pyramid, Trophy, Honeycomb](#14-test-strategy-shapes--pyramid-trophy-honeycomb)
15. [Test Design Techniques](#15-test-design-techniques)
16. [Unit Testing](#16-unit-testing)
17. [Integration Testing with Real Dependencies](#17-integration-testing-with-real-dependencies)
18. [Contract Testing](#18-contract-testing)
19. [End-to-End and UI Testing](#19-end-to-end-and-ui-testing)
20. [Non-Functional Testing](#20-non-functional-testing)
21. [Static Testing and Reviews](#21-static-testing-and-reviews)
22. [Test Automation Architecture](#22-test-automation-architecture)
23. [Test Data Management](#23-test-data-management)
24. [Defect Management](#24-defect-management)
25. [Flaky Tests](#25-flaky-tests)
26. [Quality Metrics and Gates](#26-quality-metrics-and-gates)
27. [Shift-Right: Testing in Production](#27-shift-right-testing-in-production)
28. [Testing AI / LLM Features](#28-testing-ai--llm-features)
29. [Tooling Matrix](#29-tooling-matrix)
30. [Checklists](#30-checklists)
31. [References](#31-references)

---

## 1. Quality, QA, QC, and Testing

| Term | Meaning | Orientation |
|---|---|---|
| **Quality** | Degree to which a product satisfies stated and implied needs (ISO/IEC 25010:2023 quality model — chapter 01 §5) | Outcome |
| **Quality assurance (QA)** | Process-oriented activities that **prevent** defects (standards, reviews, training, DoD) | Process |
| **Quality control (QC)** | Product-oriented activities that **detect** defects (testing, inspections) | Product |
| **Testing** | Evaluating a component or system to find defects and gain confidence; includes static (no execution) and dynamic (execution) testing | Activity |
| **Verification** | Did we build the product **right**? (conforms to spec) | Spec |
| **Validation** | Did we build the **right** product? (meets user needs) | User |

> Quality is a whole-team responsibility. Testers/SDETs bring specialised skills; developers own testing of their own code; product owns acceptance.

### Error → defect → failure

```text
Human ERROR (mistake) → introduces a DEFECT (bug/fault) in an artifact → may cause a FAILURE when executed
Root cause analysis looks at why the error was made (process, knowledge, tooling)
```

---

## 2. Standards and Bodies of Knowledge

| Source | What it gives you |
|---|---|
| **ISTQB CTFL v4.0** (Certified Tester Foundation Level syllabus) | Vocabulary, test process, levels, types, techniques, test management; widely used certification baseline |
| **ISO/IEC/IEEE 29119** series | International testing standards: Part 1 concepts/definitions, Part 2 test processes, Part 3 test documentation, Part 4 test techniques, Part 5 keyword-driven testing (plus technical reports) |
| **SWEBOK v4.0** — Software Testing KA | Body-of-knowledge view of testing |
| **ISO/IEC 25010:2023** | Quality characteristics to target with test types |
| **OWASP WSTG / ASVS** | Security testing guide and verification requirements (chapter 09) |
| **W3C WCAG 2.2** | Accessibility conformance criteria |

ISTQB: https://istqb.org/ · ISO catalogue: https://www.iso.org

---

## 3. The Seven Testing Principles

The ISTQB Foundation syllabus lists seven principles. In summary:

| # | Principle | Practical implication |
|---:|---|---|
| 1 | **Testing shows the presence, not the absence, of defects** | Passing tests reduce risk; they never prove correctness |
| 2 | **Exhaustive testing is impossible** | Prioritise by risk; use test design techniques |
| 3 | **Early testing saves time and money** | Shift-left: review requirements and designs; TDD |
| 4 | **Defects cluster together** | Focus effort on modules with past defects, high complexity, frequent change |
| 5 | **Tests wear out** (the "pesticide paradox") | Refresh and vary tests; add exploratory testing |
| 6 | **Testing is context dependent** | A payments API and a marketing site need different strategies |
| 7 | **Absence-of-defects fallacy** | Bug-free software that doesn't meet user needs is still a failure — validate |

---

## 4. The Software Testing Life Cycle (STLC)

The STLC is the sequence of testing activities that runs **inside** the SDLC. Chapter 02 §11 shows where each activity sits in the SDLC; this chapter is the full reference.

### 4.1 Two equivalent views

| Classic STLC phase (industry) | ISTQB CTFL v4.0 test activity |
|---|---|
| 1. Requirement analysis | Test analysis ("what to test") |
| 2. Test planning | Test planning |
| 3. Test case development | Test design ("how to test") + test implementation |
| 4. Test environment setup | Test implementation (environment & data readiness) |
| 5. Test execution | Test execution + **test monitoring and control** (continuous) |
| 6. Test cycle closure | Test completion |

ISTQB stresses that these activities often run **iteratively and in parallel**, not as a strict waterfall.

### 4.2 STLC at a glance

```text
┌──────────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│ 1 Requirement│──►│ 2 Test       │──►│ 3 Test case  │──►│ 4 Environment│──►│ 5 Execution  │──►│ 6 Cycle      │
│   analysis   │   │   planning   │   │   development│   │   setup      │   │   + control  │   │   closure    │
└──────────────┘   └──────────────┘   └──────────────┘   └──────────────┘   └──────────────┘   └──────────────┘
        ▲                       monitoring & control spans all phases                                 │
        └──────────────────────────── lessons learned feed the next cycle ◄──────────────────────────┘
```

### 4.3 Entry and exit criteria summary

| Phase | Entry criteria | Activities | Exit criteria | Deliverables |
|---|---|---|---|---|
| **1 Requirement analysis** | Requirements/stories, acceptance criteria drafts, architecture overview available | Testability review, identify test conditions, clarify ambiguities, risk analysis, automation feasibility | Testable requirements; questions resolved or logged; risks identified | Test conditions list, RTM seed, clarification log, product risk register |
| **2 Test planning** | Requirement analysis done; scope known | Strategy, scope, levels/types, approach, estimates, roles, tools, environments, schedule, entry/exit criteria | Plan reviewed and approved | Test plan / strategy, estimates, risk-based priorities |
| **3 Test case development** | Approved plan; stable-enough requirements/designs | Design test cases/charters, automation scripts, test data specs, review | Cases reviewed; traceability complete; automation in repo | Test cases, automated tests, data specs, updated RTM |
| **4 Environment setup** | Environment design; data needs; build available | Provision env (IaC), deploy build, load/mask data, configure tools, smoke test | Environment stable; smoke test passed | Ready environment, smoke test result, env doc |
| **5 Execution** | Cases + data + environment ready; build passes smoke; entry criteria met | Execute, log results, report defects, retest, regression, monitor progress, control (re-prioritise) | Exit criteria met (coverage, pass rate, open defect thresholds) | Execution logs, defect reports, daily/sprint status reports |
| **6 Cycle closure** | Execution complete or time-boxed end reached | Evaluate exit criteria, metrics, lessons learned, archive testware, hand-over | Report approved; testware archived | Test summary/completion report, metrics, retrospective actions |

---

## 5. STLC Phase 1 — Requirement Analysis

**Goal:** understand *what* needs testing and make requirements testable before code exists.

### 5.1 Activities

```text
1. Review requirements, user stories, acceptance criteria, NFRs, designs, API specs
2. Assess testability: each requirement verifiable? measurable? (chapter 03 §4)
3. Identify TEST CONDITIONS (testable aspects: features, rules, quality attributes)
4. Identify product RISKS (likelihood × impact) to drive priority
5. Identify test types needed (functional, performance, security, accessibility, compatibility…)
6. Assess automation feasibility
7. Log questions/ambiguities and get them resolved (Three Amigos, example mapping)
```

### 5.2 Testability review checklist

```markdown
- [ ] Each requirement has acceptance criteria (Given/When/Then or measurable statement)
- [ ] Inputs, outputs, and business rules are specified, including boundaries
- [ ] Error and "unwanted behaviour" cases specified (EARS "If…then")
- [ ] NFRs have numbers (latency, throughput, availability, WCAG level)
- [ ] Test data needs identifiable (and obtainable without real PII)
- [ ] External dependencies identified (can they be stubbed / containerised?)
- [ ] Observability: can we see the outcome (UI, API response, events, logs)?
```

### 5.3 Product risk register (testing view)

| Risk ID | Feature / area | Risk | Likelihood | Impact | Level | Test response |
|---|---|---|:---:|:---:|:---:|---|
| PR-01 | Payment retry | Duplicate charge on retry | M | Critical | **High** | Idempotency tests, concurrency tests, contract tests with PSP sandbox |
| PR-02 | Checkout | Latency regression at peak | M | High | **High** | Load test at 3× peak |
| PR-03 | Order history UI | Accessibility failure | M | Medium | Medium | axe automated + manual screen-reader pass |
| PR-04 | Admin export | Data leak across tenants | L | Critical | **High** | Authorisation tests per role/tenant |

**Deliverables:** test conditions, clarification log, product risk register, RTM seed.

---

## 6. STLC Phase 2 — Test Planning

**Goal:** decide the approach, scope, resources, and criteria.

### 6.1 Test strategy vs test plan

| | Test strategy | Test plan |
|---|---|---|
| Scope | Organisation / product line | Project / release / sprint |
| Changes | Rarely | Per release |
| Content | Principles, levels, types, tools, automation approach, environments, quality gates | Scope, schedule, resources, specific risks, entry/exit criteria |

### 6.2 Test plan template (aligned with the content areas described in ISO/IEC/IEEE 29119-3)

```markdown
# Test Plan — <product/release> v<version>
Owner: · Approvers: · Status: · Last updated:

## 1. Context
Scope of the release, references (PRD, SRS, ADRs, risk register).

## 2. Scope
In scope: features/REQ-IDs, platforms, integrations.
Out of scope: (with reason).

## 3. Product risks and priorities
Link to risk register; test focus per high risk.

## 4. Test approach
| Level | Types | Techniques | Automated? | Owner |
|-------|-------|-----------|------------|-------|
| Unit | Functional | EP/BVA, TDD | 100% | Devs |
| Integration | Functional, contract | Testcontainers, Pact | 100% | Devs |
| System/E2E | Functional, regression | Critical journeys | Mostly | SDET |
| Non-functional | Performance, security, a11y | Load @3× peak; DAST; axe | Yes | SDET/SRE/Sec |
| Acceptance | UAT, exploratory | Charters | Manual | PO + QA |

## 5. Entry and exit criteria (per level)
## 6. Test environments and data (env list, data sources, masking)
## 7. Tools (test frameworks, test management, CI)
## 8. Roles and responsibilities (RACI)
## 9. Schedule and estimates (aligned to sprints/milestones)
## 10. Defect management (severity/priority definitions, triage cadence, SLAs)
## 11. Metrics and reporting (what, how often, to whom)
## 12. Suspension and resumption criteria
## 13. Deliverables
## 14. Approvals
```

### 6.3 Example entry/exit criteria

```text
System test ENTRY:
- Build deployed to QA env; smoke suite 100% pass
- Unit + integration suites green in CI
- No open Critical defects from previous cycle on in-scope features
- Test data loaded; environment health checks green

System test EXIT:
- 100% of high-risk test cases executed and passed
- ≥ 95% of all planned cases executed; ≥ 98% of executed passed
- 0 open Critical/High defects (or accepted with documented risk sign-off)
- Performance within budget; no unaccepted Critical/High security findings
- RTM shows every in-scope requirement covered by ≥ 1 passing test
```

### 6.4 Suspension and resumption

```text
SUSPEND when: smoke test fails; > 30% of cases blocked; environment unavailable > 4 h; critical defect blocks main flows
RESUME when: new build passes smoke; blockers resolved; environment stable
```

### 6.5 Estimation for testing

Techniques: percentage of development effort (historical ratio), work breakdown (cases × average time), three-point estimates, Wideband Delphi/Planning Poker with the team. Include environment setup, data preparation, regression, defect retesting, and reporting.

---

## 7. STLC Phase 3 — Test Case Development

**Goal:** turn test conditions into executable tests and testware.

### 7.1 Test case template

```markdown
| Field | Example |
|---|---|
| ID | TC-PAY-121 |
| Title | Offer alternative methods after insufficient-funds decline |
| Requirement(s) | REQ-PAY-042 |
| Priority / risk | P1 / PR-01 |
| Preconditions | Customer logged in; cart with 1 item (₹2 499); card ending 0002 configured to decline (sandbox) |
| Test data | customer=cust_test_17, card=sandbox_decline_insufficient_funds |
| Steps | 1. Go to checkout 2. Pay with card 0002 3. Observe result |
| Expected result | Message "Your payment was declined"; options UPI + Netbanking shown; cart unchanged; one failed attempt recorded |
| Type / level | Functional / System (E2E) |
| Automated | Yes — tests/e2e/payments/retry.spec.ts |
| Postconditions | Order remains PaymentFailed |
```

### 7.2 Good test case qualities

```text
✅ Traceable (REQ-IDs)      ✅ Independent (no order dependency)
✅ Repeatable (deterministic data)   ✅ Clear expected result (no "should work")
✅ One objective per case   ✅ Reviewed by a peer
```

### 7.3 Exploratory testing charters

Exploratory testing is structured, time-boxed, simultaneous learning, design, and execution.

```markdown
## Charter: Explore payment retry with flaky network
- Mission: Discover how retry behaves when connectivity drops during UPI approval
- Areas: checkout UI (mobile web), payments API, order state
- Time-box: 60 minutes
- Tester: @priya   Date: 2026-10-02
- Notes / observations:
- Bugs found: (links)
- Questions / risks:
- Coverage achieved vs. remaining:
```

Session-based test management (James & Jon Bach) adds debriefs and session metrics.

### 7.4 BDD scenarios as executable specifications

```gherkin
@REQ-PAY-042 @risk-high
Scenario: Insufficient funds offers alternatives
  Given a signed-in customer with a ₹2,499 cart
  When the card payment is declined for "insufficient_funds"
  Then the customer is offered "UPI" and "Netbanking"
  And exactly one failed payment attempt is recorded
```

**Deliverables:** test cases, charters, automated tests in the repo, test data specs, updated RTM.

---

## 8. STLC Phase 4 — Test Environment Setup

**Goal:** a stable, production-like, reproducible environment with the right data.

### 8.1 Environment principles

```text
🔴 Environments defined as code (Terraform/Helm/Compose) — chapter 17
🔴 Same artifact (image digest) as will go to production (build once, promote)
🔴 No real personal data unless masked/anonymised and approved (DPDP/GDPR)
🟠 Ephemeral per-PR environments for fast, isolated testing
🟠 Service virtualisation / sandboxes for third parties (PSP, SMS, email)
🟠 Health checks + smoke suite gate every environment deployment
```

### 8.2 Environment readiness checklist

```markdown
- [ ] Build version/commit deployed and recorded
- [ ] Configuration matches test plan (feature flags, integrations)
- [ ] Test data loaded (seed scripts / factories), masked if derived from prod
- [ ] Third-party sandboxes reachable; credentials from secret manager
- [ ] Monitoring/logging available for testers (dashboards, log access)
- [ ] Test tools configured (test runners, browsers/devices, load generators)
- [ ] Smoke test passed
```

### 8.3 Local integration environment (Docker Compose)

```yaml
services:
  postgres:
    image: postgres:18
    environment:
      POSTGRES_PASSWORD: test
      POSTGRES_DB: shop
    ports: ["5432:5432"]
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      retries: 10
  redis:
    image: valkey/valkey:8
    ports: ["6379:6379"]
  mailpit:
    image: axllent/mailpit
    ports: ["8025:8025", "1025:1025"]   # catch outgoing email in tests
  wiremock:
    image: wiremock/wiremock:3x
    ports: ["8089:8080"]
    volumes: ["./test/stubs:/home/wiremock"]   # stub third-party APIs (PSP, courier)
```

> Pin image tags to the versions you run in production.

---

## 9. STLC Phase 5 — Test Execution, Monitoring and Control

### 9.1 Execution loop

```text
Run tests (by priority) → compare actual vs expected → log result
  → on failure: investigate (test bug? environment? product defect?)
     → report defect (§24) → fix → RETEST (confirmation) → REGRESSION around the change
Monitor progress daily → CONTROL: re-prioritise, add tests, escalate risks, adjust scope
```

| Term | Meaning |
|---|---|
| **Confirmation testing (retest)** | Re-run the failed test after the fix |
| **Regression testing** | Verify the change didn't break other things |
| **Smoke test** | Shallow check that the build is testable |
| **Sanity test** | Narrow check of a specific fix/feature |

### 9.2 Test monitoring metrics

| Metric | Formula / description |
|---|---|
| Execution progress | executed ÷ planned |
| Pass rate | passed ÷ executed |
| Blocked rate | blocked ÷ planned |
| Requirement coverage | requirements with ≥ 1 passing test ÷ in-scope requirements |
| Risk coverage | high-risk items tested ÷ high-risk items |
| Defect find/fix trend | new vs closed per day (convergence) |
| Defect density | defects ÷ size (KLOC, story points, or features) — use cautiously |
| Defect age | time open, by severity |

### 9.3 Daily test status report

```markdown
## Test status — Release 2.5 — 2026-10-02 (Day 4 of 6)
Overall: 🟠 At risk
| Metric | Value | Target |
|---|---|---|
| Executed | 212 / 260 (82%) | 100% by Day 6 |
| Passed | 198 (93% of executed) | ≥ 98% |
| Blocked | 9 (env: PSP sandbox outage) | 0 |
| Open defects | Crit 0 · High 2 · Med 7 · Low 11 | Crit/High = 0 at exit |
Risks/issues: PSP sandbox instability blocking retry tests (PR-01). Mitigation: WireMock stubs for non-PSP checks; escalated to vendor.
Decisions needed: accept 1 High (BUG-913, cosmetic on old Android WebView) with workaround?
```

---

## 10. STLC Phase 6 — Test Cycle Closure

### 10.1 Activities

```text
1. Evaluate exit criteria; document deviations and accepted risks
2. Compile metrics and the test summary/completion report
3. Ensure all defects are closed, deferred with owner, or accepted
4. Archive testware (cases, scripts, data, env config) and link to release tag
5. Hand over: regression suite updates, known issues to support, monitoring to ops
6. Retrospective: what to improve in process, tooling, automation
```

### 10.2 Test completion report template

```markdown
# Test Completion Report — Release 2.5.0
## Summary
Recommendation: ✅ Release / ⚠️ Release with accepted risks / ❌ Do not release
## Scope tested vs planned (and deviations)
## Results
| Level | Planned | Executed | Passed | Failed | Blocked |
## Coverage
Requirements: 100% (48/48) · High-risk items: 100% · Code coverage (changed lines): 86%
## Defects
| Severity | Found | Fixed | Deferred | Accepted |
Top defects and root-cause themes
## Non-functional results
Performance: p95 checkout 610 ms @ 3× peak (budget 800 ms) ✅
Security: DAST 0 High; 2 Medium (tickets) ✅  ·  Accessibility: 0 WCAG 2.2 AA violations on key flows ✅
## Residual risks and known issues (with owners)
## Lessons learned & improvement actions
## Approvals
```

---

## 11. STLC in Agile and CI/CD

| STLC phase | In Scrum | In continuous delivery |
|---|---|---|
| Requirement analysis | Backlog refinement, Three Amigos, example mapping | Same, per story |
| Test planning | Test strategy (once) + sprint-level test approach in planning | Pipeline design: which tests at which stage |
| Test case development | During the sprint, alongside code (TDD/BDD) | Tests in the same PR as code |
| Environment setup | Ephemeral/preview environments | Pipeline provisions per run |
| Execution | Continuous in CI; exploratory each sprint | Automated stages; canary analysis |
| Closure | Sprint review + retro; release report | Automated quality report per deployment |

### Agile Testing Quadrants (Brian Marick; Crispin & Gregory)

```text
                     Business-facing
            Q2                          Q3
   Functional tests, examples,     Exploratory, usability, UAT,
   story tests (automated + manual)  alpha/beta (manual)
Supporting ──────────────────────────────────────────── Critique
the team    Q1                          Q4               the product
   Unit, component tests          Performance, load, security,
   (automated)                     "-ility" testing (tools)
                     Technology-facing
```

### Pipeline test stages (fast → slow)

```text
Pre-commit: format, lint, fast unit tests (seconds)
PR / CI:    unit + component + integration (Testcontainers) + contract + SAST/SCA (minutes)
Post-merge: E2E critical journeys, DAST baseline, API fuzzing (≤ 15–30 min)
Nightly:    full regression, load tests, extended security scans, mutation testing
Pre-release / canary: smoke in prod, automated canary analysis, synthetic monitoring
```

---

## 12. Test Levels

| Level | Object under test | Typical owner | Environment | Basis |
|---|---|---|---|---|
| **Component (unit)** | Function/class/module in isolation | Developers | Local/CI, no network | Detailed design, code |
| **Component integration** | Interactions between components in one service (incl. DB) | Developers | CI with containers | Design, interfaces |
| **Contract** | Agreement between consumer and provider | Developers | CI | API specs, consumer expectations |
| **System** | Whole system behaviour | QA/SDET + devs | QA/staging | Requirements, use cases |
| **System integration** | Interactions with external systems | QA/SDET | Staging with sandboxes | Interface specs |
| **Acceptance** (UAT, OAT, contractual, regulatory, alpha/beta) | Fitness for use | Users, PO, ops | Staging/prod-like | Business requirements, contracts, regulations |

**OAT** (operational acceptance testing) covers backups/restore, failover, monitoring, runbooks, deployability.

---

## 13. Test Types

| Type | Question | ISO 25010 characteristic |
|---|---|---|
| Functional | Does it do the right thing? | Functional suitability |
| Regression | Did changes break anything? | All |
| Performance (load, stress, soak, spike, scalability) | Fast and stable under load? | Performance efficiency |
| Security (SAST, DAST, IAST, pentest, fuzzing) | Resistant to attack? | Security |
| Accessibility | Usable by people with disabilities? (WCAG 2.2) | Interaction capability |
| Usability | Effective, efficient, satisfying? | Interaction capability |
| Compatibility (browser, device, OS, API version) | Works across environments and with other systems? | Compatibility / Flexibility |
| Reliability / resilience (chaos, failover) | Tolerates faults and recovers? | Reliability |
| Recovery / DR | Restores within RPO/RTO? | Reliability |
| Installability / upgrade / migration | Deploys, upgrades, migrates data correctly? | Flexibility |
| Localization / internationalization | Correct languages, formats, scripts, RTL? | Interaction capability |
| Data quality / migration | Data complete, correct, reconciled? | Functional suitability |
| Safety | Avoids harm? | Safety |
| Maintainability (static analysis, complexity) | Easy to change? | Maintainability |

---

## 14. Test Strategy Shapes — Pyramid, Trophy, Honeycomb

```text
   Test Pyramid (Cohn)       Testing Trophy (Dodds)        Honeycomb (Spotify, microservices)
        /\  E2E                    ___ E2E                        ⬡ integrated (few)
       /  \                       |   | Integration (most)       ⬡⬡⬡ integration (most)
      /----\ Integration          |___|                          ⬡ implementation detail (few)
     /      \                      | |  Unit
    /--------\ Unit (most)        _|_|_ Static (types, lint)
```

| Shape | Emphasis | Best for |
|---|---|---|
| **Pyramid** | Many fast unit tests, fewer integration, very few E2E | Logic-heavy code, libraries, monoliths |
| **Trophy** | Static analysis + mostly integration tests | Frontend apps (testing components as users use them) |
| **Honeycomb** | Integration tests of each service against real dependencies | Microservices with thin business logic |

> Shared rule across all shapes: **as few slow, flaky, end-to-end tests as possible, covering only critical user journeys; push everything else to faster levels.** Avoid the "ice-cream cone" (mostly manual/E2E).

### Choosing what to test where

| Concern | Best level |
|---|---|
| Business rules, calculations, validation | Unit |
| SQL queries, ORM mappings, migrations | Integration (real DB via Testcontainers) |
| Serialization, HTTP handlers, auth middleware | Integration (in-process API tests) |
| Inter-service compatibility | Contract |
| Critical user journeys (signup, checkout) | E2E |
| Visual layout | Visual regression |

---

## 15. Test Design Techniques

### 15.1 Black-box (specification-based)

| Technique | Use | Example |
|---|---|---|
| **Equivalence partitioning (EP)** | Divide inputs into classes expected to behave alike; test one per class | Quantity: invalid ≤ 0, valid 1–100, invalid > 100 |
| **Boundary value analysis (BVA)** | Test at and around boundaries | 0, 1, 100, 101 (2-value) or 0,1,2 / 99,100,101 (3-value) |
| **Decision table testing** | Combinations of conditions → actions | COD eligibility table (chapter 03 §10) |
| **State transition testing** | Valid/invalid transitions | Order state machine (chapter 05 §13): cover every transition + invalid ones |
| **Use case / scenario testing** | End-to-end flows and alternatives | UC-07 main + extensions |
| **Pairwise / combinatorial** | Many parameters; cover all pairs | Browser × OS × locale × payment method (tools: PICT, allpairs) |
| **Classification tree** | Structured partitioning of many inputs | Search filters |

### BVA example

```text
Requirement: quantity must be between 1 and 100 inclusive.
Partitions:  [≤0 invalid] [1..100 valid] [≥101 invalid]
BVA tests:   0 ❌, 1 ✅, 100 ✅, 101 ❌   (+ e.g. -1, 2, 99 for 3-value BVA)
```

```python
import pytest

@pytest.mark.parametrize("qty, valid", [(0, False), (1, True), (100, True), (101, False)])
def test_quantity_boundaries(qty, valid):
    assert is_valid_quantity(qty) is valid
```

### 15.2 White-box (structure-based)

| Coverage | Meaning | Notes |
|---|---|---|
| Statement coverage | Each statement executed | Weakest |
| **Branch (decision) coverage** | Each branch outcome taken | ISTQB foundation focus; 100% branch ⇒ 100% statement |
| Condition / MC/DC | Each condition independently affects outcome | Required in safety-critical (e.g. avionics) |
| Path coverage | All paths | Usually infeasible |

### 15.3 Experience-based

Error guessing, exploratory testing, checklist-based testing (e.g. OWASP, accessibility checklists).

### 15.4 Advanced techniques

| Technique | What it does | Tools |
|---|---|---|
| **Property-based testing** | Generates many inputs to check invariants | Hypothesis (Python), fast-check (JS/TS), jqwik (Java), Kotest, FsCheck, proptest (Rust) |
| **Mutation testing** | Injects small code changes; good tests should fail ("kill mutants") | PIT (Java), Stryker (JS/TS, .NET), mutmut (Python), cargo-mutants |
| **Fuzzing** | Random/malformed inputs to find crashes and security bugs | libFuzzer, AFL++, Go native fuzzing, Jazzer, RESTler/Schemathesis for APIs |
| **Snapshot / approval testing** | Compare output against approved snapshot | Jest/Vitest snapshots, ApprovalTests — review diffs carefully |
| **Metamorphic testing** | Check relations between outputs of related inputs | ML, search ranking, scientific code |

```typescript
import fc from 'fast-check';
test('adding money is commutative', () => {
  fc.assert(fc.property(fc.bigInt({ min: 0n, max: 10n ** 12n }), fc.bigInt({ min: 0n, max: 10n ** 12n }), (a, b) =>
    Money.of(a, 'INR').add(Money.of(b, 'INR')).equals(Money.of(b, 'INR').add(Money.of(a, 'INR')))));
});
```

---

## 16. Unit Testing

### 16.1 FIRST properties

**F**ast · **I**ndependent · **R**epeatable · **S**elf-validating · **T**imely (written with the code).

### 16.2 TDD cycle

```text
RED    → write a small failing test for the next behaviour
GREEN  → write the simplest code to pass
REFACTOR → improve design with tests green
Repeat in minutes-long cycles.
```

### 16.3 What to unit test

```text
✅ Domain logic, calculations, validation, state transitions, edge cases, error paths
✅ Pure functions and value objects
❌ Framework glue with no logic, trivial getters/setters, third-party library internals
❌ Private methods directly — test through the public behaviour
```

### 16.4 Frameworks

| Language | Unit test framework | Mocking |
|---|---|---|
| Java | JUnit 5 (Jupiter), AssertJ | Mockito |
| Kotlin | JUnit 5, Kotest | MockK |
| JS/TS | Vitest, Jest, Node test runner | built-in mocks, MSW (network) |
| Python | pytest | `unittest.mock`, pytest-mock |
| C# | xUnit, NUnit, MSTest | NSubstitute, Moq |
| Go | `testing` + testify | interfaces + fakes, gomock |
| Rust | built-in `#[test]` | mockall |
| PHP | PHPUnit, Pest | Mockery, PHPUnit mocks |
| Swift | Swift Testing, XCTest | protocol fakes |
| Dart | `package:test`, `flutter_test` | mocktail |

---

## 17. Integration Testing with Real Dependencies

Mocking a database or message broker tests your mock, not your system. Prefer **real dependencies in containers**.

### 17.1 Testcontainers

Testcontainers starts throwaway Docker containers from test code; available for Java, .NET, Go, Node.js, Python, Rust, and more. https://testcontainers.com/

```java
@Testcontainers
class OrderRepositoryIT {
    @Container
    static PostgreSQLContainer<?> pg = new PostgreSQLContainer<>("postgres:18");

    @DynamicPropertySource
    static void props(DynamicPropertyRegistry r) {
        r.add("spring.datasource.url", pg::getJdbcUrl);
        r.add("spring.datasource.username", pg::getUsername);
        r.add("spring.datasource.password", pg::getPassword);
    }

    @Autowired OrderRepository repo;

    @Test
    void savesAndLoadsOrderWithLines() {
        var order = OrderMother.withLines(2);
        repo.save(order);
        assertThat(repo.findById(order.id())).hasValueSatisfying(o -> assertThat(o.lines()).hasSize(2));
    }
}
```

```python
# pytest + testcontainers-python
from testcontainers.postgres import PostgresContainer

def test_migration_and_query():
    with PostgresContainer("postgres:18") as pg:
        engine = create_engine(pg.get_connection_url())
        run_migrations(engine)
        assert count_orders(engine) == 0
```

### 17.2 Third-party APIs

| Approach | Use |
|---|---|
| **HTTP stubs** (WireMock, MockServer, MSW, Prism from OpenAPI) | Deterministic tests of your client logic, error handling, timeouts |
| **Vendor sandbox** | Periodic integration verification; not in every CI run (flaky, rate-limited) |
| **Contract tests** | Agreement verification (§18) |
| **Record/replay** (VCR-style) | Legacy APIs without specs — re-record regularly |

Always test: timeouts, 429/5xx with retries, malformed responses, slow responses, duplicate webhooks.

---

## 18. Contract Testing

Contract tests verify that a **consumer** and **provider** agree on an interface without running both together in an E2E environment.

| Approach | How | Tools |
|---|---|---|
| **Consumer-driven contracts** | Consumers publish expectations; provider verifies them in its CI | **Pact** (+ Pact Broker / PactFlow) |
| **Provider-driven / spec-based** | OpenAPI/AsyncAPI is the contract; validate both sides against it | Schemathesis, Dredd-style tools, Prism, Spectral, openapi-diff |
| **Schema compatibility for events** | Enforce backward/forward compatibility rules | Confluent/Apicurio Schema Registry, Buf (protobuf breaking-change detection) |

```text
Consumer CI: unit tests generate pact file → publish to broker (with version + branch)
Provider CI: fetch pacts for relevant consumer versions → verify against running provider
Deploy gate: "can-i-deploy" check ensures compatibility with versions in the target environment
```

Breaking-change checks in CI for APIs: chapter 11.

---

## 19. End-to-End and UI Testing

### 19.1 E2E principles

```text
🔴 Cover only critical user journeys (signup/login, search, checkout, payment, key admin flows)
🔴 Independent tests: create own data via API/fixtures; never depend on other tests
🔴 Stable selectors: roles/labels/test IDs (getByRole, data-testid) — not CSS paths
🔴 No fixed sleeps; use auto-waiting assertions
🟠 Run in parallel, sharded; retries only to collect evidence — investigate every retry
🟠 Capture traces, screenshots, videos on failure
```

### 19.2 Playwright example

```typescript
import { test, expect } from '@playwright/test';

test('customer can retry with UPI after card decline @REQ-PAY-042', async ({ page, request }) => {
  const customer = await (await request.post('/test-api/customers', { data: { cartTotal: 249900 } })).json();
  await page.goto(`/login-as/${customer.token}`);   // test-only auth helper, disabled in production
  await page.getByRole('link', { name: 'Checkout' }).click();
  await page.getByLabel('Card number').fill('4000 0000 0000 0002');
  await page.getByRole('button', { name: 'Pay ₹2,499' }).click();

  await expect(page.getByRole('alert')).toHaveText(/payment was declined/i);
  await expect(page.getByRole('button', { name: 'Pay with UPI' })).toBeVisible();
});
```

### 19.3 UI testing tools

| Need | Tools |
|---|---|
| Web E2E | **Playwright**, Cypress, WebdriverIO, Selenium (legacy/grid scale) |
| Component testing | Testing Library (React/Vue/Angular/Svelte), Playwright/Cypress component testing, Storybook interaction tests |
| Visual regression | Playwright screenshots, Chromatic, Percy, Applitools |
| Android | Espresso, Compose UI testing, UI Automator |
| iOS | XCUITest, Swift Testing for logic |
| Cross-platform mobile | Appium, Maestro, Detox (React Native), Flutter integration_test/Patrol |
| Device farms | Firebase Test Lab, AWS Device Farm, BrowserStack, Sauce Labs |

Mobile testing specifics: chapter **14**; IDE setup: **35**, **36**.

---

## 20. Non-Functional Testing

### 20.1 Performance testing

| Type | Purpose | Shape |
|---|---|---|
| **Load** | Behaviour at expected peak | Ramp to target RPS, hold |
| **Stress** | Find the breaking point and failure mode | Increase until failure |
| **Spike** | Sudden bursts (sales, push notifications) | Instant jump |
| **Soak / endurance** | Leaks, degradation over time | Moderate load for hours |
| **Scalability** | Does adding capacity add throughput? | Step load with scaling |
| **Capacity** | Max users/RPS within SLOs | Search for limit |

```text
Rules:
- Test in a production-like environment with production-like data volumes
- Define pass/fail in advance from NFRs (p95/p99 latency, error rate, throughput, resource use)
- Measure from the client and the server (traces/metrics) — find the bottleneck, not just the symptom
- Warm up; avoid coordinated omission (use tools that measure intended vs actual send times where available)
- Keep scripts in the repo; run nightly and before major releases
```

Tools: **k6**, Gatling, JMeter, Locust, Artillery, **autocannon** (HTTP benchmarking — see **40-autocannon_production_CLI.md**), wrk/wrk2, Vegeta.

```javascript
// k6: load test with thresholds as automated pass/fail
import http from 'k6/http';
import { check } from 'k6';
export const options = {
  stages: [{ duration: '2m', target: 300 }, { duration: '10m', target: 300 }, { duration: '1m', target: 0 }],
  thresholds: { http_req_failed: ['rate<0.001'], http_req_duration: ['p(95)<500', 'p(99)<900'] },
};
export default function () {
  const res = http.get(`${__ENV.BASE_URL}/v1/products?category=shoes`);
  check(res, { 'status 200': r => r.status === 200 });
}
```

Deep dive: chapter **19**.

### 20.2 Security testing

| Technique | When | Tools |
|---|---|---|
| SAST | Every PR | CodeQL, Semgrep, SonarQube |
| SCA (dependencies) | Every PR + daily | Dependabot, Renovate + OSV-Scanner, Snyk, Trivy |
| Secret scanning | Pre-commit, PR, push protection | gitleaks, trufflehog, platform scanning |
| IaC / container scanning | Every PR | Trivy, Checkov, Grype |
| DAST | Post-deploy to test/staging | OWASP ZAP, Burp Suite |
| API security / fuzzing | Nightly | Schemathesis, RESTler, ZAP API scan |
| Penetration testing | Risk-based, before major releases, at least annually for critical systems | Internal/external specialists |
| Authorisation tests | Every PR (automated) | Test matrix: role × tenant × resource × action |

Security testing guide: **OWASP WSTG**; requirements: **OWASP ASVS** — chapter **09**.

### 20.3 Accessibility testing

```text
Automated (catches a portion of issues): axe-core (@axe-core/playwright), Lighthouse, Pa11y, Android Accessibility Scanner, Xcode Accessibility Inspector
Manual (required): keyboard-only navigation, screen readers (NVDA, JAWS, VoiceOver, TalkBack), zoom 200–400%, contrast, reduced motion
Target: WCAG 2.2 Level AA
```

```typescript
import AxeBuilder from '@axe-core/playwright';
test('checkout has no detectable WCAG A/AA violations', async ({ page }) => {
  await page.goto('/checkout');
  const results = await new AxeBuilder({ page }).withTags(['wcag2a', 'wcag2aa', 'wcag21aa', 'wcag22aa']).analyze();
  expect(results.violations).toEqual([]);
});
```

### 20.4 Reliability, resilience, and DR testing

| Test | Example |
|---|---|
| Dependency failure injection | Make PSP return 503 / time out → orders queue, user informed |
| Instance/pod kill | Kill a pod during load → no user-visible errors beyond SLO |
| Zone failure simulation | Drain one AZ in staging → failover within RTO |
| Backup restore drill | Restore last night's backup to a clean environment → verify data and time |
| Chaos engineering | Hypothesis-driven experiments with blast-radius limits (LitmusChaos, Chaos Mesh, AWS FIS, Gremlin) |

Details: chapter **20**.

### 20.5 Compatibility testing

Maintain a support matrix (browsers, OS versions, devices, screen sizes, API versions) based on analytics, and test the top combinations plus the minimum supported versions. Use pairwise selection for large matrices.

---

## 21. Static Testing and Reviews

Static testing finds defects **without executing** code — cheaper and earlier.

| Static technique | Finds |
|---|---|
| Requirements reviews | Ambiguity, gaps, conflicts (chapter 03 §14) |
| Design/architecture reviews | Structural risks (chapter 04 §19) |
| Code review | Logic, security, maintainability (chapter 07 §10) |
| Static analysis / linters / type checkers | Bug patterns, vulnerabilities, style (chapter 06) |
| Threat modelling | Security design flaws (chapter 09) |

ISTQB review types (increasing formality): informal review → walkthrough → technical review → inspection.

---

## 22. Test Automation Architecture

### 22.1 Layers

```text
┌───────────────────────────────────────────────┐
│ Test cases / specs (Gherkin or code)          │  what to verify
├───────────────────────────────────────────────┤
│ Domain/business layer (actions, flows)        │  "placeOrder", "retryPayment"
├───────────────────────────────────────────────┤
│ Interaction layer (page objects / API clients │  how to drive the system
│  / screen objects / fixtures)                 │
├───────────────────────────────────────────────┤
│ Core: drivers, config, data builders, waits,  │  infrastructure
│ reporting, logging, retries, environments     │
└───────────────────────────────────────────────┘
```

### 22.2 Patterns

| Pattern | Purpose |
|---|---|
| Page Object / Screen Object | Encapsulate UI structure; tests read like user actions |
| Screenplay | Actors perform tasks with abilities — scales for large suites |
| Test data builders / Object Mother | Readable, minimal test data creation |
| API-first setup | Create state via APIs/DB seeding, not via the UI |
| Fixtures (pytest, Playwright) | Shared setup/teardown with explicit scope |
| Keyword-driven | Business-readable keywords (ISO/IEC/IEEE 29119-5; Robot Framework) |

### 22.3 Automation rules

```text
🔴 Automated tests live in version control with the code they test (or a dedicated repo with versioning)
🔴 Tests run in CI on every change; results visible on the PR
🔴 Test code follows coding standards and is reviewed
🟠 Automate what is repeated, stable, and high-value; keep humans for exploration and judgement
🟠 Measure automation by defects prevented and feedback speed, not by number of scripts
🟠 Parallelise; keep the PR feedback loop under ~10–15 minutes
```

### 22.4 Automation ROI heuristic

```text
Automate when: (manual time × expected runs) > (build time + maintenance time over its lifetime)
High value: regression of critical paths, data-driven checks, cross-browser/device, performance, API contracts
Low value:  rapidly changing UI under active design, one-off checks, subjective UX evaluation
```

---

## 23. Test Data Management

| Strategy | Use | Notes |
|---|---|---|
| **Synthetic data** (factories, Faker) | Default for unit/integration/E2E | No privacy risk; covers edge cases deliberately |
| **Seed datasets** (versioned SQL/JSON) | Reference data, demo environments | Version with migrations |
| **Masked/anonymised production subsets** | Realistic volume/distribution for performance or migration tests | Requires approval; irreversible masking; data minimisation; DPDP/GDPR compliance |
| **Generated at scale** | Load tests | Realistic distributions (Zipf for popularity, etc.) |

```text
🔴 No real personal data in non-production without approved masking/anonymisation
🔴 Each test creates and cleans up its own data (or uses isolated schemas/tenants)
🟠 Use unique IDs per test run to allow parallel execution
🟠 Edge-case data catalogue: Unicode names, long strings, RTL text, emoji, zero/negative amounts,
    leap days, DST transitions, IST/UTC boundaries, max-length fields
```

```python
# Builder with sensible defaults; tests override only what matters
@dataclass
class OrderBuilder:
    customer_id: str = "cust_test"
    currency: str = "INR"
    lines: list = field(default_factory=lambda: [("prd_test", 1, 49900)])
    def with_lines(self, n: int) -> "OrderBuilder":
        self.lines = [(f"prd_{i}", 1, 10000) for i in range(n)]
        return self
    def build(self) -> Order: ...
```

---

## 24. Defect Management

### 24.1 Defect lifecycle

```text
New → Triaged (valid? severity/priority/owner) → In progress → Fixed → Ready for retest
   → Verified → Closed
Side paths: Rejected (not a bug / duplicate / cannot reproduce) · Deferred (accepted for later) · Reopened
```

### 24.2 Severity vs priority

| Severity (impact) | Definition |
|---|---|
| **S1 Critical** | Data loss/corruption, security breach, core function unavailable, no workaround |
| **S2 High** | Major function impaired, workaround difficult |
| **S3 Medium** | Partial impairment with reasonable workaround |
| **S4 Low** | Cosmetic, minor inconvenience |

| Priority (urgency) | Definition |
|---|---|
| **P1** | Fix immediately / blocks release |
| **P2** | Fix in current sprint/release |
| **P3** | Schedule in backlog |
| **P4** | Fix if time permits |

> Severity is set by impact (QA/engineering); priority by business urgency (PO). A typo on the homepage logo can be S4/P1.

### 24.3 Defect report template

```markdown
**Title:** [Payments] Retry creates duplicate charge when UPI callback arrives after timeout
**ID:** BUG-917 · **Severity:** S1 · **Priority:** P1 · **Component:** payments
**Environment:** staging, build 2.5.0-rc.3 (sha a1b2c3d), Chrome 1xx / Android 15
**Requirement / test:** REQ-PAY-042 / TC-PAY-124
**Steps to reproduce:**
1. Pay with UPI; do not approve for 6 minutes (client timeout 5 min)
2. Retry with UPI and approve
3. Approve the first (stale) UPI request in the UPI app
**Expected:** Second approval rejected or refunded automatically; one charge only
**Actual:** Two successful charges for order ord_7f3k
**Frequency:** 3/3
**Evidence:** trace ID 4bf92f…, logs link, screenshots, PSP dashboard IDs
**Impact:** Financial loss / customer harm
**Notes:** Likely missing idempotency check on late callbacks
```

### 24.4 Triage and SLAs (example)

| Severity | Triage | Fix target (production) |
|---|---|---|
| S1 | Immediately (incident process if in prod) | Hotfix within hours–1 day |
| S2 | Same business day | Within current sprint / ≤ 1 week |
| S3 | Within 2 business days | Prioritised in backlog |
| S4 | Weekly triage | Best effort |

### 24.5 Root cause analysis

For escaped defects (found in production) and S1/S2 defects, run a short RCA: **5 Whys** or fishbone (Ishikawa) across people, process, tools, requirements, environment. Output: a regression test + at least one prevention action (lint rule, checklist item, test type, design change). Defect taxonomies (e.g. Orthogonal Defect Classification) help spot trends.

---

## 25. Flaky Tests

A flaky test passes and fails on the same code. Flakiness destroys trust in CI.

| Common cause | Fix |
|---|---|
| Timing / fixed sleeps | Auto-waiting assertions; await events; deterministic clocks |
| Shared state / order dependence | Isolate data; reset state; random test order to detect |
| Async not awaited | Lint for floating promises; proper awaits |
| Real network / third parties | Stubs; containers |
| Time zones / dates | Fixed clock; UTC; test around DST deliberately |
| Resource limits in CI | Right-size runners; limit parallelism |
| Non-deterministic ordering (maps, DB without ORDER BY) | Sort or assert as sets |
| Animations in UI tests | Disable animations in test mode |

### Flaky test policy

```text
1. Detect: CI tracks pass/fail history per test (most CI/test platforms can flag flaky tests)
2. Quarantine: move to a non-blocking quarantine suite with a ticket and owner (max N days)
3. Fix or delete: within the SLA; a test nobody fixes is a test nobody needs
4. Never "just add retries" to hide flakiness in blocking suites
```

---

## 26. Quality Metrics and Gates

### 26.1 Useful metrics

| Metric | Insight | Caution |
|---|---|---|
| **DORA change fail rate & deployment rework rate** | Escaped quality problems | Team-level only |
| Escaped defects (found in prod) by severity | Effectiveness of the whole quality system | Normalise by change volume |
| Defect detection percentage (DDP) | Share of defects found before release | Requires consistent counting |
| Mean time to detect / to fix defects | Feedback speed | — |
| Test suite duration (PR feedback time) | Developer flow | Target ≤ ~10–15 min |
| Flaky test rate | CI trust | Track and burn down |
| Coverage on changed code | Untested changes | Not a quality proof |
| Mutation score (critical modules) | Test effectiveness | Expensive; use selectively |
| Requirement & risk coverage | Completeness | Requires traceability |

> **Goodhart's law:** when a measure becomes a target, it ceases to be a good measure. Never reward raw test counts or coverage percentages.

### 26.2 Coverage guidance

```text
- Gate on "no decrease" and on changed-line coverage (e.g. ≥ 80% of changed lines), not a global absolute
- 100% coverage does not mean 100% tested — assertions matter
- Exclude generated code; include integration tests in coverage where tooling allows
```

### 26.3 Quality gate (PR) example

```text
✅ Build passes  ✅ Lint/format/typecheck clean  ✅ Unit + integration + contract tests pass
✅ Changed-line coverage ≥ 80%  ✅ No new Critical/High SAST findings
✅ No new Critical/High vulnerable dependencies  ✅ No secrets detected
✅ Approvals per CODEOWNERS
```

Release gates: chapter 02 §13.

---

## 27. Shift-Right: Testing in Production

Pre-production testing cannot reproduce real traffic, data, and scale. **Shift-right** practices manage the remaining risk safely.

| Practice | Purpose |
|---|---|
| **Smoke tests after deploy** | Verify the deployment works |
| **Synthetic monitoring** | Scripted user journeys run continuously against production |
| **Canary releases + automated canary analysis** | Compare canary vs baseline metrics before full rollout |
| **Feature flags & dark launches** | Expose to internal users/cohorts first |
| **A/B testing / experimentation** | Validate user impact statistically |
| **Real user monitoring (RUM)** | Actual performance/errors in browsers/apps |
| **Chaos engineering** | Verify resilience hypotheses in production with guardrails |
| **Error tracking** | Crash/error aggregation (Sentry, Crashlytics, etc.) |

Guardrails: limited blast radius, automatic rollback triggers, no destructive actions on real customer data, test accounts flagged and excluded from analytics/billing.

---

## 28. Testing AI / LLM Features

LLM-based features are non-deterministic; test them with **evaluations** rather than exact-match assertions.

| Layer | Approach |
|---|---|
| Deterministic code around the model | Normal unit/integration tests (prompt assembly, parsing, tool routing, guardrails) |
| Structured outputs | Validate against JSON Schema; test parser on malformed outputs |
| Quality | Offline eval datasets with graders (exact match, rubric scoring, LLM-as-judge with human calibration) |
| Retrieval (RAG) | Retrieval precision/recall on labelled queries; groundedness/citation checks |
| Safety & security | Red-team suites: prompt injection, jailbreaks, data exfiltration, harmful content (OWASP Top 10 for LLM Applications) |
| Authorisation | Users can't retrieve documents they lack access to via RAG |
| Regression | Re-run evals on every prompt, model, or retrieval change; block on score drops |
| Performance & cost | Latency (time to first token), tokens and cost per request under load |
| Production | Sampled human review, user feedback, drift monitoring |

```text
Eval set hygiene: version eval datasets; keep a held-out set; include adversarial and edge cases;
record model version, prompt version, parameters, and scores per run.
```

---

## 29. Tooling Matrix

| Category | Tools (examples) |
|---|---|
| Unit | JUnit 5, Kotest, Vitest, Jest, pytest, xUnit, NUnit, Go `testing`, PHPUnit/Pest, Swift Testing/XCTest |
| Integration | Testcontainers, Docker Compose, WireMock, MockServer, MSW, LocalStack (AWS emulation) |
| Contract | Pact, Spring Cloud Contract, Schemathesis, Prism, Buf, schema registries |
| API testing | REST Assured, Supertest, httpx/pytest, Postman/Newman, Bruno, Karate, IDE HTTP clients |
| E2E web | Playwright, Cypress, WebdriverIO, Selenium |
| Mobile | Espresso, XCUITest, Appium, Maestro, Detox, Patrol |
| BDD | Cucumber, Reqnroll (SpecFlow successor), Behave, pytest-bdd, Karate |
| Performance | k6, Gatling, JMeter, Locust, Artillery, autocannon, wrk2 |
| Security | ZAP, Burp Suite, CodeQL, Semgrep, Trivy, Grype, OSV-Scanner, gitleaks |
| Accessibility | axe-core, Lighthouse, Pa11y, screen readers |
| Visual | Playwright snapshots, Chromatic, Percy, Applitools |
| Mutation / property | PIT, Stryker, mutmut, Hypothesis, fast-check, jqwik |
| Chaos | Chaos Mesh, LitmusChaos, AWS FIS, Azure Chaos Studio, Gremlin |
| Test management | TestRail, Xray/Zephyr (Jira), Qase, Allure TestOps, Azure Test Plans |
| Reporting | Allure, JUnit XML in CI, ReportPortal |

---

## 30. Checklists

### Per story / PR
- [ ] Acceptance criteria testable; test conditions identified (STLC 1)
- [ ] Unit tests for logic, including boundaries and error paths
- [ ] Integration tests with real dependencies for persistence/messaging
- [ ] Contract/spec checks updated for API changes
- [ ] E2E updated only if a critical journey changed
- [ ] Accessibility checks for UI changes
- [ ] Tests deterministic, independent, tagged with REQ-IDs
- [ ] CI green; no new flaky tests

### Per release (STLC closure)
- [ ] Test plan executed; deviations documented
- [ ] Exit criteria met or risks formally accepted
- [ ] Performance, security, accessibility results within targets
- [ ] RTM complete; all in-scope requirements covered
- [ ] Defects closed/deferred with owners; known issues communicated to support
- [ ] Test completion report approved; testware archived and linked to release tag
- [ ] Lessons learned captured

### Test strategy (per product)
- [ ] Quality attributes ranked (ISO 25010) and mapped to test types
- [ ] Test levels, ownership, and automation approach defined
- [ ] Pipeline stages and time budgets defined
- [ ] Environments and test data strategy (no real PII) defined
- [ ] Defect severity/priority definitions and SLAs published
- [ ] Metrics chosen (DORA instability metrics, escaped defects, flaky rate, PR feedback time)
- [ ] Shift-right practices: smoke, synthetic monitoring, canaries

---

## 31. References

### Standards and syllabi
- ISTQB CTFL v4.0: https://istqb.org/certifications/certified-tester-foundation-level-ctfl-v4-0
- ISTQB Glossary: https://glossary.istqb.org/
- ISO/IEC/IEEE 29119 series (catalogue): https://www.iso.org
- SWEBOK v4.0 — Software Testing KA: https://www.computer.org/education/bodies-of-knowledge/software-engineering
- ISO/IEC 25010:2023: https://www.iso.org/standard/78176.html
- W3C WCAG 2.2: https://www.w3.org/TR/WCAG22/
- OWASP Web Security Testing Guide: https://owasp.org/www-project-web-security-testing-guide/
- OWASP ASVS: https://owasp.org/www-project-application-security-verification-standard/

### Practice
- Martin Fowler — Practical Test Pyramid: https://martinfowler.com/articles/practical-test-pyramid.html
- Google Testing Blog: https://testing.googleblog.com/
- Agile Testing (Crispin & Gregory): https://agiletester.ca/
- Kent C. Dodds — Testing Trophy: https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications
- Spotify — Testing of Microservices (honeycomb): https://engineering.atspotify.com/2018/01/testing-of-microservices
- DORA — continuous testing capability: https://dora.dev/capabilities/test-automation/

### Tools
- Testcontainers: https://testcontainers.com/
- Pact: https://docs.pact.io/
- Playwright: https://playwright.dev/
- k6: https://grafana.com/docs/k6/latest/
- axe-core: https://github.com/dequelabs/axe-core
- Schemathesis: https://schemathesis.io/
- Stryker: https://stryker-mutator.io/ · PIT: https://pitest.org/
- Hypothesis: https://hypothesis.readthedocs.io/ · fast-check: https://fast-check.dev/
- OWASP ZAP: https://www.zaproxy.org/

---

**Previous:** [07 — Version Control](./07-version-control.md) · **Next:** [09 — Security Engineering](./09-security-engineering.md)