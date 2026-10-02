# 🤝 Team Collaboration — Production Engineering Guide

> How engineering teams organise, communicate, decide, and grow together: team structures (Team Topologies, Conway's Law, cognitive load), roles and responsibilities, team charters and working agreements, synchronous vs asynchronous communication and writing culture, meetings, distributed and hybrid teams across time zones, pair and mob programming, code review culture, cross-team collaboration and dependencies, the product–engineering partnership, decision-making frameworks, conflict resolution, psychological safety, feedback and 1:1s, mentoring and career growth, hiring and interviewing, inclusion, developer experience (SPACE), team health checks, retrospectives, and norms for working with AI assistants.
>
> Related: [01 Competency matrix & ethics](./01-engineering-foundations.md) · [02 Scrum/Kanban & RACI](./02-software-development-lifecycle.md) · [03 Planning & stakeholders](./03-requirements-and-planning.md) · [07 Code review](./07-version-control.md#10-code-review) · [16 Platform engineering](./16-devops-and-ci-cd.md#15-platform-engineering-and-golden-paths) · [21 On-call & incidents](./21-production-operations.md) · [22 Documentation & onboarding](./22-documentation-and-knowledge-management.md)

---

## 📚 Table of Contents

- [🤝 Team Collaboration — Production Engineering Guide](#-team-collaboration--production-engineering-guide)
  - [📚 Table of Contents](#-table-of-contents)
  - [1. Principles](#1-principles)
  - [2. Team Structures — Team Topologies](#2-team-structures--team-topologies)
    - [2.1 Four team types](#21-four-team-types)
    - [2.2 Three interaction modes](#22-three-interaction-modes)
    - [2.3 Team size](#23-team-size)
  - [3. Conway's Law and Cognitive Load](#3-conways-law-and-cognitive-load)
    - [3.1 Conway's Law](#31-conways-law)
    - [3.2 Team cognitive load](#32-team-cognitive-load)
  - [4. Roles and Responsibilities](#4-roles-and-responsibilities)
  - [5. Team Charter and Working Agreements](#5-team-charter-and-working-agreements)
    - [5.1 Team charter template](#51-team-charter-template)
    - [5.2 Working agreements (examples)](#52-working-agreements-examples)
  - [6. Communication — Sync vs Async](#6-communication--sync-vs-async)
    - [6.1 Channel guidelines](#61-channel-guidelines)
  - [7. Writing Culture](#7-writing-culture)
    - [Weekly team update template](#weekly-team-update-template)
  - [8. Meetings](#8-meetings)
    - [8.1 Meeting rules](#81-meeting-rules)
    - [8.2 Core team rituals (Scrum example — chapter 02 §7)](#82-core-team-rituals-scrum-example--chapter-02-7)
  - [9. Distributed and Hybrid Teams](#9-distributed-and-hybrid-teams)
    - [9.1 Time-zone strategy](#91-time-zone-strategy)
    - [9.2 Hybrid fairness](#92-hybrid-fairness)
    - [9.3 Working with vendors and outsourced teams](#93-working-with-vendors-and-outsourced-teams)
  - [10. Pair Programming and Ensemble (Mob) Programming](#10-pair-programming-and-ensemble-mob-programming)
  - [11. Code Review Culture](#11-code-review-culture)
  - [12. Cross-Team Collaboration and Dependencies](#12-cross-team-collaboration-and-dependencies)
    - [12.1 Reducing dependencies](#121-reducing-dependencies)
    - [12.2 Working across teams](#122-working-across-teams)
    - [12.3 Requesting work from another team (template)](#123-requesting-work-from-another-team-template)
  - [13. Product–Engineering–Design Partnership](#13-productengineeringdesign-partnership)
  - [14. Decision-Making](#14-decision-making)
    - [14.1 Choose a decision method explicitly](#141-choose-a-decision-method-explicitly)
    - [14.2 DACI / RAPID](#142-daci--rapid)
    - [14.3 Reversible vs irreversible decisions](#143-reversible-vs-irreversible-decisions)
    - [14.4 Disagree and commit](#144-disagree-and-commit)
  - [15. Conflict Resolution](#15-conflict-resolution)
    - [15.1 Technical disagreements](#151-technical-disagreements)
    - [15.2 Interpersonal conflict](#152-interpersonal-conflict)
  - [16. Psychological Safety](#16-psychological-safety)
    - [Quick team pulse questions (anonymous, quarterly)](#quick-team-pulse-questions-anonymous-quarterly)
  - [17. Feedback and 1:1s](#17-feedback-and-11s)
    - [17.1 Feedback principles](#171-feedback-principles)
    - [17.2 SBI model (Center for Creative Leadership)](#172-sbi-model-center-for-creative-leadership)
    - [17.3 1:1 meetings](#173-11-meetings)
  - [18. Mentoring, Knowledge Sharing, and Career Growth](#18-mentoring-knowledge-sharing-and-career-growth)
    - [18.1 Knowledge sharing mechanisms](#181-knowledge-sharing-mechanisms)
    - [18.2 Mentoring](#182-mentoring)
    - [18.3 Career frameworks](#183-career-frameworks)
  - [19. Hiring and Interviewing](#19-hiring-and-interviewing)
    - [19.1 Structured interviewing](#191-structured-interviewing)
    - [19.2 Typical loop (adapt to level)](#192-typical-loop-adapt-to-level)
  - [20. Inclusion and Wellbeing](#20-inclusion-and-wellbeing)
  - [21. Developer Experience and Productivity](#21-developer-experience-and-productivity)
    - [21.1 SPACE framework](#211-space-framework)
    - [21.2 Common friction to remove](#212-common-friction-to-remove)
  - [22. Team Health Checks and Retrospectives](#22-team-health-checks-and-retrospectives)
    - [22.1 Team health check (adapted from Spotify's Squad Health Check model)](#221-team-health-check-adapted-from-spotifys-squad-health-check-model)
    - [22.2 Retrospective formats](#222-retrospective-formats)
    - [22.3 Retrospective rules](#223-retrospective-rules)
  - [23. Collaborating with AI Assistants](#23-collaborating-with-ai-assistants)
  - [24. Collaboration Tooling](#24-collaboration-tooling)
  - [25. Checklists](#25-checklists)
    - [New team](#new-team)
    - [Quarterly](#quarterly)
    - [Leaders](#leaders)
  - [26. References](#26-references)
    - [Team design and organisation](#team-design-and-organisation)
    - [Collaboration practices](#collaboration-practices)
    - [Productivity and developer experience](#productivity-and-developer-experience)
    - [Books](#books)

---

## 1. Principles

| # | Principle |
|---:|---|
| 1 | **Teams, not individuals, own outcomes** — services, roadmaps, and on-call belong to teams. |
| 2 | **Small, stable, long-lived teams** — trust and shared context take time to build. |
| 3 | **Clear ownership, clear interfaces** — between people as between services. |
| 4 | **Default to writing** — decisions and context in durable, searchable form. |
| 5 | **Psychological safety enables performance** — people speak up about risks, mistakes, and ideas. |
| 6 | **Disagree openly, commit fully** — debate before deciding; align after. |
| 7 | **Optimise for flow** — minimise handoffs, waiting, and context switching. |
| 8 | **Respect time and time zones** — async first, meetings with purpose. |

---

## 2. Team Structures — Team Topologies

*Team Topologies* (Matthew Skelton & Manuel Pais) defines four fundamental team types and three interaction modes.

### 2.1 Four team types

| Team type | Purpose | Example |
|---|---|---|
| **Stream-aligned** | Aligned to a flow of work (product, user journey, customer segment); delivers end to end | Checkout team, Mobile onboarding team |
| **Enabling** | Helps stream-aligned teams adopt new skills/practices; temporary engagements | Test automation coaching, observability enablement, security champions programme |
| **Complicated-subsystem** | Owns a part requiring deep specialist knowledge | Pricing/recommendation engine, video encoding, ML ranking |
| **Platform** | Provides internal services as a product to reduce stream-aligned teams' cognitive load | Developer platform (CI/CD, Kubernetes, observability), data platform |

### 2.2 Three interaction modes

| Mode | Description | When |
|---|---|---|
| **Collaboration** | Two teams work closely together for a period | Discovery, new technology, unclear boundaries |
| **X-as-a-Service** | One team consumes another's service with minimal collaboration | Mature platform capabilities with clear APIs/docs |
| **Facilitating** | One team helps another learn/unblock | Enabling team engagements |

```text
Most teams should be stream-aligned. Platform and enabling teams exist to make stream-aligned teams faster.
Make interaction modes explicit and time-boxed ("collaborate for 6 weeks, then move to X-as-a-Service").
```

### 2.3 Team size

```text
- Typical effective team: ~5–9 people (Scrum Guide: 10 or fewer; Amazon's "two-pizza team" idea)
- Above that: communication paths grow quadratically (n(n−1)/2) → split by stream/domain
- Keep teams stable; move work to teams rather than reshuffling people for every project
```

---

## 3. Conway's Law and Cognitive Load

### 3.1 Conway's Law

> Organisations design systems that mirror their own communication structure (Melvin Conway, 1967).

**Inverse Conway manoeuvre:** shape team boundaries deliberately to get the architecture you want — e.g. align teams to bounded contexts (chapter 04 §7) so service boundaries and team boundaries match.

```text
Smell: two teams must coordinate on every release of one service → boundary is wrong (organisational or architectural)
Fix:   reassign ownership, split/merge the service, or create a platform capability
```

### 3.2 Team cognitive load

| Load type | Description | Reduce by |
|---|---|---|
| Intrinsic | Inherent difficulty of the domain/skills | Training, pairing, good design |
| Extraneous | Friction unrelated to the task (environment setup, slow CI, unclear processes) | Platforms, automation, golden paths |
| Germane | Learning that builds useful expertise | Protect it — this is where value grows |

```text
Signals of overload: too many services per team, constant context switching, long on-call queues, nobody can explain
the whole of what the team owns. Response: reduce scope, invest in platform, split the team, retire services.
```

---

## 4. Roles and Responsibilities

| Role | Core responsibilities |
|---|---|
| **Engineering Manager (EM)** | People (hiring, growth, performance, wellbeing), team health, delivery process, staffing, cross-team alignment |
| **Tech Lead (TL)** / Staff Engineer | Technical direction, architecture, quality standards, mentoring, unblocking; usually still codes |
| **Product Manager (PM) / Product Owner** | Problem and outcome ownership, prioritisation, roadmap, stakeholder alignment (chapter 03) |
| **Designer (UX/UI)** | User research, interaction and visual design, accessibility in design |
| **Software Engineers** | Design, build, test, deploy, operate; code review; documentation |
| **QA/SDET** | Test strategy, automation frameworks, exploratory testing, quality coaching (chapter 08) |
| **SRE / Platform Engineers** | Reliability, platform capabilities, observability, incident leadership (chapters 16–21) |
| **Security Champion** | Team's security point of contact; threat modelling; liaison with security team (chapter 09) |
| **Engineering Director / VP** | Org design, strategy, budgets, cross-org priorities |

Use a RACI per recurring activity (chapter 02 §18) and keep it published. Unclear ownership is the root of many collaboration problems.

---

## 5. Team Charter and Working Agreements

### 5.1 Team charter template

```markdown
# Team Charter — Checkout Team
## Mission
Make buying on ShopNow fast, reliable, and trustworthy for every customer.
## Scope / ownership
Services: checkout-web, orders-service, payments-adapter · Journeys: cart → payment → confirmation
Not in scope: catalogue, search (Discovery team), refunds back-office (Finance Tools)
## Customers & stakeholders
End customers, Customer Support, Finance, Payments partners
## Success measures
Checkout success rate ≥ 96%; checkout p95 ≤ 1 s; DORA change lead time < 1 day; on-call pages ≤ 2/week
## Team
EM, TL, PM, Designer, 5 engineers, 1 SDET · On-call: weekly rotation
## Interfaces
Platform team (X-as-a-Service: CI/CD, Kubernetes) · Payments partner (vendor) · Security (facilitating)
## Rituals
Planning (Mon), stand-up (async daily), demo (alt Fri), retro (alt Fri), ops review (Thu)
```

### 5.2 Working agreements (examples)

```markdown
- Core collaboration hours: 11:00–16:00 IST (overlap with Europe partners 13:30–16:00 IST)
- Async by default: decisions in writing (RFC/ADR/issue); meetings only with agenda
- PRs: < 400 lines; first review within 1 business day; reviewers rotate
- Stand-up: async thread by 10:30 IST (yesterday/today/blockers); sync only on Mondays
- "Ask early" rule: stuck > 1 hour → ask in team channel
- Camera optional; meeting notes always
- Focus blocks: no meetings Wed afternoons
- On-call: handoff notes every Monday; post-incident rest after night pages
- We disagree in the PR/RFC, commit after the decision, and revisit only with new information
```

Review working agreements in retrospectives every quarter.

---

## 6. Communication — Sync vs Async

| Use **async** (written) for | Use **sync** (call/in person) for |
|---|---|
| Status updates, FYIs | Complex, emotionally charged, or ambiguous topics |
| Proposals, designs, RFCs | Brainstorming and early exploration |
| Code review | Conflict resolution, sensitive feedback |
| Decisions that need input from many people | Incidents (plus written timeline) |
| Questions that can wait a few hours | Relationship building, onboarding |
| Cross-time-zone collaboration | When async has stalled (> 2 rounds without convergence) |

### 6.1 Channel guidelines

| Channel | Purpose | Response expectation |
|---|---|---|
| Issue tracker / PRs / RFCs | Work, decisions, reviews | Per review SLAs |
| Team chat channel | Coordination, quick questions | Hours (not minutes) |
| Direct messages | Personal/sensitive topics | — (avoid for work decisions) |
| Incident channels / paging | Urgent production issues | Minutes |
| Email | External, formal, cross-org announcements | 1 business day |
| Wiki / docs | Durable knowledge | — |

```text
Chat etiquette: ask the full question in one message (no "hi" then wait); use threads; summarise long threads
and link the outcome to a ticket/doc; prefer public channels over DMs so knowledge is shared.
```

---

## 7. Writing Culture

```text
- Decisions are made in documents (design docs, RFCs, ADRs — chapters 04, 05, 22), discussed in comments, finalised in writing
- Documents start with context, problem, and proposal; readers can comment asynchronously across time zones
- "Silent reading" meetings (first 10 minutes reading the doc, then discussion) improve quality of debate
- Status updates written weekly (what shipped, what's next, risks) instead of status meetings
- Brag documents / work logs help people track impact for reviews and promotions
```

### Weekly team update template

```markdown
## Checkout — Week 40
**Shipped:** payment retry (5% rollout), UPI intent fix · **Metrics:** checkout success 95.1% (+0.6)
**Next:** retry to 50%, saved cards design review · **Risks:** PSP sandbox instability (mitigation: stubs)
**Asks:** Platform — Gateway API migration slot before 20 Oct
```

---

## 8. Meetings

### 8.1 Meeting rules

| Rule | Status |
|---|:---:|
| Every meeting has a purpose, agenda, owner, and desired outcome | 🔴 |
| Only necessary attendees; others get notes | 🔴 |
| Decisions and action items recorded and shared (chapter 22 §14.3) | 🔴 |
| Start and end on time; default to 25/50-minute slots | 🟠 |
| Cancel recurring meetings that no longer deliver value; review calendar quarterly | 🟠 |
| Protect maker time (focus blocks, no-meeting days/afternoons) | 🟠 |
| Record or summarise for absent/other-time-zone colleagues | 🟠 |

### 8.2 Core team rituals (Scrum example — chapter 02 §7)

| Ritual | Cadence | Time-box (2-week sprint) |
|---|---|---|
| Sprint planning | Each sprint | ≤ 4 h (often 1–2 h) |
| Daily stand-up (sync or async) | Daily | 15 min |
| Backlog refinement | Weekly | 1 h |
| Sprint review / demo | Each sprint | ≤ 2 h |
| Retrospective | Each sprint | ≤ 1.5 h |
| Ops review | Weekly | 30 min (chapter 21 §20) |

---

## 9. Distributed and Hybrid Teams

### 9.1 Time-zone strategy

```text
- Publish each person's working hours and time zone in profiles
- Define overlap hours (e.g. 2–4 hours) for sync collaboration; everything else async
- Rotate inconvenient meeting times fairly across regions
- Write dates/times with time zones (e.g. "17:00 IST / 11:30 UTC / 12:30 CET")
- Follow-the-sun handoffs with written summaries (chapter 21 §3)
- Indian public holidays and regional holidays in shared team calendars; plan capacity accordingly (chapter 03 §18.1)
```

### 9.2 Hybrid fairness

```text
- "Remote-first" meeting setup: everyone joins from their own device when anyone is remote
- Decisions never made in hallway conversations without writing them down
- Same access to leaders, information, and visibility regardless of location
- Periodic in-person gatherings for relationship building (planning, offsites, hackathons)
```

### 9.3 Working with vendors and outsourced teams

```text
- Same engineering standards, CI gates, and code review for vendor contributions
- Clear contracts on ownership, IP, security, on-call, and knowledge transfer
- Shared channels and documentation; avoid email-only collaboration
- Least-privilege access with expiry; offboarding on contract end
```

---

## 10. Pair Programming and Ensemble (Mob) Programming

| Practice | How | Benefits | Use when |
|---|---|---|---|
| **Pair programming** (driver/navigator) | Two people, one task; swap roles every ~15–30 min | Fewer defects, knowledge sharing, faster onboarding | Complex/risky code, onboarding, debugging, cross-skill tasks |
| Strong-style pairing | "For an idea to go from your head into the computer, it must go through someone else's hands" | Transfers knowledge efficiently | Mentoring |
| **Ensemble/mob programming** | Whole team on one task, rotating driver | Shared understanding, no handoffs | Critical designs, tricky incidents, new codebase areas |
| Remote pairing | Shared editors (VS Code Live Share, JetBrains Code With Me), screen sharing, tuple-style tools | Same benefits across locations | Distributed teams |

```text
Tips: time-box sessions (with breaks); agree on the goal first; rotate pairs; pairing counts as review for low-risk changes if your policy allows.
```

---

## 11. Code Review Culture

Mechanics and standards: chapter 07 §9–10. Cultural norms:

```text
- Review the code, not the person; assume good intent
- Explain the "why" behind requests; link to standards rather than personal preference
- Label comment intent (blocking / suggestion / question / nit / praise — Conventional Comments)
- Prefer questions over commands when unsure ("What happens if X is null?")
- Authors: keep PRs small, describe context, respond to every comment, don't take feedback personally
- Reviewing is first-class work — measured and recognised, with response SLAs
- Senior engineers' code gets reviewed too; juniors are encouraged to review
- Automate style debates away (formatters/linters)
```

---

## 12. Cross-Team Collaboration and Dependencies

### 12.1 Reducing dependencies

```text
1. Architectural: loosely coupled services, stable APIs, contract tests (chapters 04, 11)
2. Organisational: stream-aligned teams owning end-to-end journeys; platform self-service
3. Process: plan dependencies early with "needed-by" dates (chapter 03 §18.2)
```

### 12.2 Working across teams

| Practice | Detail |
|---|---|
| Inner source | Contribute to other teams' repos via PRs following their CONTRIBUTING guidelines; owners review |
| API/contract-first agreements | Agree interfaces before building; mocks unblock parallel work |
| Dependency board / program sync | Visible cross-team dependencies with owners and dates |
| Guilds / chapters / communities of practice | Cross-team groups for frontend, security, data, testing, SRE |
| Architecture forums / RFC reviews | Shared technical decisions (chapter 04 §21) |
| Shared on-call for platform incidents | Clear escalation paths |

### 12.3 Requesting work from another team (template)

```markdown
## Request: Gateway API route for /v1/payments/retry (from Checkout → Platform)
Context & goal: …   Needed by: 2026-10-20   Priority/impact: blocks 5% rollout of payment retry
Proposed interface/solution: HTTPRoute with 10 s timeout   Alternatives we can do ourselves: …
Contact: @checkout-tl   Tracking issue: PLAT-934
```

---

## 13. Product–Engineering–Design Partnership

```text
"Product trio": PM + Designer + Tech Lead collaborate continuously from discovery to delivery (Teresa Torres)
- Engineers join discovery (user interviews, feasibility spikes) — not just implementation
- PM owns WHY/WHAT (outcomes, priorities); engineering owns HOW and technical health; design owns experience
- Shared metrics: outcomes (OKRs) + technical health (SLOs, DORA, tech debt) both on the roadmap
- Capacity allocation agreed explicitly, e.g. 70% product · 20% tech debt/reliability · 10% learning/experiments
- Engineering says "no" with options: "We can ship A by date X, or A+B by date Y"
```

---

## 14. Decision-Making

### 14.1 Choose a decision method explicitly

| Method | How | Use for |
|---|---|---|
| **Single decider (informed by input)** | One accountable person decides after consultation | Most team decisions — fast and clear |
| **Consensus** | Everyone agrees | Rarely — high-stakes team norms |
| **Consent** (sociocracy) | Proceed unless someone has a reasoned objection ("safe enough to try?") | Working agreements, experiments |
| **Majority vote** | Count votes | Low-stakes choices (naming, tooling preferences) |
| **Delegation** | Leader delegates the decision with constraints | Growth opportunities, local decisions |

### 14.2 DACI / RAPID

| DACI role | Meaning |
|---|---|
| **D**river | Runs the process, gathers input, keeps it moving |
| **A**pprover | Makes the decision (one person) |
| **C**ontributors | Provide expertise and input |
| **I**nformed | Told about the outcome |

### 14.3 Reversible vs irreversible decisions

```text
Two-way doors (reversible): decide fast, at the lowest reasonable level, with less consultation
One-way doors (hard to reverse: data models, public APIs, vendor lock-in, hiring): slow down, write it up, review widely
```

### 14.4 Disagree and commit

```text
1. Everyone voices concerns during the decision process (in writing where possible)
2. The decider decides and documents rationale and dissent
3. Everyone commits to making the decision succeed
4. Revisit only when new information appears — with a defined review date for contentious decisions
```

---

## 15. Conflict Resolution

### 15.1 Technical disagreements

```text
1. Clarify the goal and constraints both sides agree on (requirements, NFRs, deadlines)
2. Make options explicit; list trade-offs in a table (performance, cost, risk, maintainability, time)
3. Seek data: spikes, prototypes, benchmarks, prior incidents
4. Time-box the debate; escalate to the designated decider (TL/architect) if still stuck
5. Record the decision (ADR) including dissenting views
```

### 15.2 Interpersonal conflict

```text
- Address early, privately, and directly; focus on behaviour and impact, not character (use SBI — §17.2)
- Listen to understand; restate the other person's view before responding
- Look for shared interests behind positions
- Involve the manager (or HR) when patterns persist, when there is harassment, or when power imbalances exist
- Never tolerate harassment or discrimination — follow company policy and legal obligations (e.g. POSH Act in India for workplace sexual harassment)
```

---

## 16. Psychological Safety

**Psychological safety** (Amy Edmondson) is a shared belief that the team is safe for interpersonal risk-taking — asking questions, admitting mistakes, raising concerns, proposing ideas. Google's Project Aristotle found it to be the most important factor among the team dynamics it studied.

| Behaviour | Leaders and team members… |
|---|---|
| Frame work as learning | "This is new; we will make mistakes — let's learn quickly" |
| Model fallibility | Leaders admit their own mistakes and uncertainty |
| Respond productively | Thank people for raising problems; no blame in postmortems (chapter 21 §10) |
| Invite participation | Ask quieter members directly; use written input before discussion |
| Separate idea from person | Critique ideas, protect people |
| Close the loop | Show what changed because someone spoke up |

### Quick team pulse questions (anonymous, quarterly)

```text
1. If you make a mistake on this team, it is not held against you.
2. Members of this team are able to bring up problems and tough issues.
3. It is safe to take a risk on this team.
4. No one on this team would deliberately act to undermine my efforts.
(Agree/disagree scale — adapted from Edmondson's team psychological safety survey items)
```

---

## 17. Feedback and 1:1s

### 17.1 Feedback principles

```text
- Timely (close to the event), specific, actionable, and balanced
- Positive feedback publicly (when the person is comfortable); corrective feedback privately
- Peer feedback is normal, not only manager feedback
- Ask for feedback regularly — especially leaders
```

### 17.2 SBI model (Center for Creative Leadership)

```text
Situation:  "In yesterday's incident call…"
Behaviour:  "…you summarised the status every 15 minutes and assigned clear owners…"
Impact:     "…which kept everyone aligned and cut our recovery time."
(+ for corrective feedback: "What could we try next time?")
```

### 17.3 1:1 meetings

| Practice | Detail |
|---|---|
| Cadence | Weekly or biweekly, 30 min; rarely cancelled |
| Ownership | The report owns the agenda; shared running doc |
| Topics | Wellbeing, blockers, feedback both ways, career growth, context on org changes — not status updates |
| Follow-through | Action items tracked in the shared doc |

```markdown
## 1:1 — @dev2 / @em — 2026-10-02
- How are you doing? (energy, workload, on-call load)
- Wins since last time:
- Blockers / frustrations:
- Feedback (both directions):
- Growth: progress on "lead the saved-cards design" goal
- Actions:
```

---

## 18. Mentoring, Knowledge Sharing, and Career Growth

### 18.1 Knowledge sharing mechanisms

| Mechanism | Format |
|---|---|
| Tech talks / brown bags | 30–45 min internal talks, recorded |
| Guilds / communities of practice | Cross-team interest groups with regular sessions and shared standards |
| Reading groups | Books/papers (e.g. DDIA, SRE books) |
| Demos | Sprint reviews open to other teams |
| Postmortem reviews | Org-wide learning from incidents |
| Internal docs and handbook | This repository (chapter 22) |
| Hackathons / innovation days | Explore ideas, fix papercuts, cross-team mixing |

### 18.2 Mentoring

```text
- Every new joiner gets a buddy (onboarding) and access to a mentor (growth)
- Mentoring ≠ managing: mentors advise; managers own performance and career processes
- Sponsorship: senior people actively create opportunities (visible projects, speaking, promotion advocacy) — especially for under-represented engineers
- Reverse mentoring: juniors teach seniors new tools/practices (e.g. AI tooling)
```

### 18.3 Career frameworks

Use a published competency matrix (chapter 01 §21) with dual tracks — **individual contributor** (Engineer → Senior → Staff → Principal) and **management** (EM → Senior EM → Director) — at equal levels of seniority and recognition. Growth plans: one or two concrete goals per quarter tied to the matrix, with evidence collected (brag document).

---

## 19. Hiring and Interviewing

### 19.1 Structured interviewing

```text
1. Define the role scorecard: must-have competencies, level expectations (from the competency matrix)
2. Map each interview stage to specific competencies (no duplicated, no unassessed areas)
3. Use consistent questions/exercises and rubrics; interviewers trained (incl. bias awareness)
4. Independent written feedback BEFORE the debrief (avoid groupthink)
5. Decision based on evidence against the rubric; hiring manager accountable
```

### 19.2 Typical loop (adapt to level)

| Stage | Assesses | Notes |
|---|---|---|
| Recruiter screen | Motivation, logistics | Share the process and preparation guidance |
| Technical screen | Fundamentals, problem-solving | Realistic problems over puzzles |
| Practical exercise (pairing or short take-home with time cap) | Coding, testing, readability | Respect candidates' time; pay for long take-homes where appropriate |
| System design (mid/senior+) | Architecture, trade-offs, NFRs | Use realistic scenarios (chapter 04) |
| Collaboration/behavioural | Communication, ownership, conflict, learning | Past-behaviour questions (STAR) |
| Hiring manager | Team fit with values, growth, expectations | "Culture add", not "culture fit" |

```text
🔴 Same rubric for every candidate for the role; documented decisions
🔴 Accessibility accommodations offered; no questions about protected characteristics
🟠 Policy on AI assistant use in interviews stated clearly in advance (allowed/not allowed per stage)
🟠 Candidate experience: timely updates, feedback where possible
```

---

## 20. Inclusion and Wellbeing

| Area | Practices |
|---|---|
| Inclusive meetings | Rotate facilitation and note-taking; written input before discussion; no interruptions; captions on calls |
| Inclusive language | In code, docs, and conversation (chapter 22 §7.2) |
| Fair opportunities | Rotate high-visibility projects; transparent promotion criteria; calibrated reviews |
| Accessibility | Tools and documents accessible; accommodations for disabilities |
| Language diversity | Clear, simple English in written communication; patience with non-native speakers; avoid idioms |
| Wellbeing | Sustainable pace; no hero culture; on-call recovery time; respect leave and holidays (including regional festivals); right to disconnect outside working hours |
| Burnout signals | Persistent overtime, cynicism, withdrawal, rising errors — managers act early (scope reduction, support, time off) |

---

## 21. Developer Experience and Productivity

### 21.1 SPACE framework

The **SPACE** framework (Forsgren, Storey, et al.) measures developer productivity across five dimensions — never with a single metric:

| Dimension | Example measures |
|---|---|
| **S**atisfaction & well-being | Survey: satisfaction with tools, work, team; burnout indicators |
| **P**erformance | Outcomes: quality, reliability, customer impact (not lines of code) |
| **A**ctivity | Counts of PRs, reviews, deploys — context only, never targets |
| **C**ommunication & collaboration | Review turnaround, knowledge sharing, onboarding time |
| **E**fficiency & flow | Lead time, interruptions, time in focus, handoffs, waiting time |

Combine with DORA's delivery metrics (chapter 01 §17) and developer experience (DevEx) surveys on feedback loops, cognitive load, and flow state.

### 21.2 Common friction to remove

```text
Slow CI (> 15 min) · flaky tests · slow local builds · environment setup taking days · unclear ownership ·
waiting for reviews/approvals · excessive meetings · access requests taking days · unclear priorities ·
noisy on-call · outdated docs
→ Prioritise by survey + measurement; assign owners (often the platform team); re-measure quarterly
```

```text
🔴 Never measure or rank individuals by activity metrics (commits, PRs, story points, lines of code)
```

---

## 22. Team Health Checks and Retrospectives

### 22.1 Team health check (adapted from Spotify's Squad Health Check model)

Quarterly, each team rates (🟢 good / 🟡 some problems / 🔴 bad) and notes trend (↑ → ↓):

| Area | Prompt |
|---|---|
| Delivering value | We deliver stuff we're proud of and stakeholders are happy |
| Easy to release | Releasing is simple, safe, and mostly automated |
| Fun / energy | We love going to work and have great fun working together |
| Health of codebase | Our code is clean, easy to read, and has good test coverage |
| Learning | We're learning lots of interesting stuff all the time |
| Mission | We know exactly why we are here and are excited about it |
| Pawns or players | We are in control of our destiny; we decide what to build and how |
| Speed | We get stuff done quickly; no waiting, no delays |
| Suitable process | Our way of working fits us perfectly |
| Support | We always get great support and help when we ask |
| Teamwork | We are a tight-knit team that works together really well |
| On-call health *(addition)* | On-call is sustainable and rarely disrupts our lives |

Use results for conversation and improvement actions — not for comparing teams.

### 22.2 Retrospective formats

| Format | Prompts | Good for |
|---|---|---|
| **Start / Stop / Continue** | What to start, stop, continue | Quick, general |
| **Mad / Sad / Glad** | Emotional reflection | After stressful periods |
| **4Ls** | Liked, Learned, Lacked, Longed for | Project end |
| **Sailboat** | Wind (helps), anchors (slows), rocks (risks), island (goal) | Visual, goal-oriented |
| **Timeline** | Plot events and energy over time | After incidents or long projects |
| **Five Whys / fishbone** | Root-cause a specific problem | Recurring issues |

### 22.3 Retrospective rules

```text
- Prime directive (Norm Kerth): "Regardless of what we discover, we understand and truly believe that everyone did the best job
  they could, given what they knew at the time, their skills and abilities, the resources available, and the situation at hand."
- Rotate facilitators; vary formats
- Leave with 1–3 concrete actions with owners; review last retro's actions first
- Psychological safety: what's said in retro is used for improvement, not evaluation
```

---

## 23. Collaborating with AI Assistants

| Norm | Detail |
|---|---|
| Transparency | Follow organisational policy on disclosing substantial AI-generated content in PRs/docs |
| Ownership | The person who submits AI-assisted work owns it fully (chapter 06 §24) |
| Shared context | Maintain repository context files (`AGENTS.md`, chapter 22 §19) so assistants follow team conventions |
| Review load | AI increases code volume — keep PRs small; protect reviewer capacity; consider review SLAs carefully |
| Learning | Juniors should still understand fundamentals; use AI for explanation, not just generation; pair on AI-assisted work |
| Data handling | Only approved tools; no secrets/customer data in prompts (chapter 09) |
| Team agreements | Agree where AI helps (tests, boilerplate, docs drafts, refactors) and where humans must lead (architecture decisions, security-critical code, incident command) |

DORA's 2025 research found AI amplifies existing team strengths and weaknesses — healthy collaboration practices become more important, not less.

---

## 24. Collaboration Tooling

| Need | Tools (examples) |
|---|---|
| Chat | Slack, Microsoft Teams, Google Chat, Mattermost |
| Video & async video | Zoom, Google Meet, Teams; Loom-style async recordings |
| Docs & knowledge | Docs-as-code sites, Confluence, Notion, Google Docs, SharePoint (chapter 22) |
| Whiteboarding | Miro, FigJam, MURAL, Excalidraw |
| Planning & tracking | Jira, Linear, GitHub Projects, GitLab Issues, Azure Boards |
| Design | Figma |
| Code collaboration | GitHub, GitLab, Bitbucket (chapter 42); Live Share / Code With Me for pairing |
| Surveys & health checks | Internal survey tools, DevEx platforms |
| Scheduling across time zones | Calendar time-zone overlays, world-clock tools |

```text
Tool hygiene: one tool per purpose; documented conventions (channel naming, labels, workflows);
integrations (PR → chat notifications, incidents → channels) configured centrally.
```

---

## 25. Checklists

### New team
- [ ] Team type and mission defined (Team Topologies)
- [ ] Charter with scope, ownership, success measures, interfaces
- [ ] Working agreements (hours, communication, reviews, on-call, focus time)
- [ ] Rituals scheduled; decision method clear (who decides what)
- [ ] RACI for recurring activities
- [ ] Onboarding plan and buddies for new members

### Quarterly
- [ ] Team health check + psychological safety pulse
- [ ] Working agreements revisited
- [ ] Cognitive load review (services owned, on-call load)
- [ ] DevEx survey actions reviewed
- [ ] Calendar/meeting audit
- [ ] Growth goals updated in 1:1s

### Leaders
- [ ] Regular 1:1s held; feedback flowing both directions
- [ ] Decisions and rationale written down and shared
- [ ] High-visibility work rotated fairly
- [ ] Burnout signals monitored; sustainable on-call
- [ ] Hiring uses structured interviews and rubrics

---

## 26. References

### Team design and organisation
- Team Topologies (Skelton & Pais): https://teamtopologies.com/
- Conway's Law (original paper, 1968): https://www.melconway.com/Home/Committees_Paper.html
- Scrum Guide (team size, accountabilities): https://scrumguides.org/
- Martin Fowler — Conway's Law: https://martinfowler.com/bliki/ConwaysLaw.html

### Collaboration practices
- Google re:Work — Project Aristotle (team effectiveness): https://rework.withgoogle.com/
- Amy Edmondson — *The Fearless Organization*
- Center for Creative Leadership — SBI feedback model: https://www.ccl.org/
- Spotify Squad Health Check model: https://engineering.atspotify.com/2014/09/squad-health-check-model/
- Retrospective formats (Retromat): https://retromat.org/
- Conventional Comments: https://conventionalcomments.org/
- Google Engineering Practices — code review: https://google.github.io/eng-practices/

### Productivity and developer experience
- SPACE framework (ACM Queue, 2021): https://queue.acm.org/detail.cfm?id=3454124
- DevEx framework (ACM Queue, 2023): https://queue.acm.org/detail.cfm?id=3595878
- DORA research: https://dora.dev/

### Books
- *Team Topologies* — Matthew Skelton & Manuel Pais
- *The Manager's Path* — Camille Fournier
- *An Elegant Puzzle* — Will Larson · *Staff Engineer* — Will Larson
- *Accelerate* — Forsgren, Humble, Kim
- *Radical Candor* — Kim Scott
- *Continuous Discovery Habits* — Teresa Torres

---

**Previous:** [22 — Documentation & Knowledge Management](./22-documentation-and-knowledge-management.md) · **Next:** [24 — Engineering Checklists](./24-engineering-checklists.md)