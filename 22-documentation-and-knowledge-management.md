# 📚 Documentation & Knowledge Management — Production Engineering Guide

> How engineering organisations capture, organise, maintain, and find knowledge: why documentation matters, the **Diátaxis** framework, documentation types and ownership, docs-as-code toolchains and CI, writing style guides, templates (README, design docs, RFCs, ADRs, runbooks, postmortems, onboarding), architecture and API documentation, diagrams as code, knowledge management (information architecture, search, freshness, archival), decision records, documentation in the Definition of Done, metrics, documentation for AI assistants, localisation, security of documentation — and how to maintain **this handbook** itself.
>
> Related: [04 ADRs, C4, arc42](./04-software-architecture.md) · [05 Design docs](./05-software-design.md#3-the-design-process-and-design-docs-rfcs) · [06 Code comments & repo docs](./06-coding-standards.md#6-comments-and-documentation) · [07 README/CONTRIBUTING/SECURITY.md](./07-version-control.md#5-repository-structure-and-mandatory-files) · [11 API docs & DX](./11-api-and-integration.md#22-developer-experience-and-documentation) · [20 DR plans](./20-reliability-and-disaster-recovery.md) · [21 Runbooks & postmortems](./21-production-operations.md) · [23 Team collaboration](./23-team-collaboration.md)

---

## 📚 Table of Contents

- [📚 Documentation \& Knowledge Management — Production Engineering Guide](#-documentation--knowledge-management--production-engineering-guide)
  - [📚 Table of Contents](#-table-of-contents)
  - [1. Why Documentation Matters](#1-why-documentation-matters)
  - [2. Principles](#2-principles)
  - [3. The Diátaxis Framework](#3-the-diátaxis-framework)
  - [4. Documentation Types and Ownership](#4-documentation-types-and-ownership)
  - [5. Docs-as-Code](#5-docs-as-code)
    - [5.1 Benefits](#51-benefits)
    - [5.2 Formats](#52-formats)
    - [5.3 Repository docs layout (service)](#53-repository-docs-layout-service)
  - [6. Documentation Toolchain and CI](#6-documentation-toolchain-and-ci)
    - [6.1 Static site generators](#61-static-site-generators)
    - [6.2 Quality tooling](#62-quality-tooling)
    - [6.3 CI pipeline for docs](#63-ci-pipeline-for-docs)
    - [6.4 MkDocs Material starter](#64-mkdocs-material-starter)
  - [7. Writing Style](#7-writing-style)
    - [7.1 Style guides to adopt (don't write your own from scratch)](#71-style-guides-to-adopt-dont-write-your-own-from-scratch)
    - [7.2 Core rules](#72-core-rules)
    - [7.3 Admonitions — use sparingly](#73-admonitions--use-sparingly)
  - [8. Templates](#8-templates)
    - [8.1 Repository README](#81-repository-readme)
    - [8.2 How-to guide](#82-how-to-guide)
    - [8.3 Other templates in this handbook](#83-other-templates-in-this-handbook)
  - [9. Architecture Documentation](#9-architecture-documentation)
  - [10. API and Developer Documentation](#10-api-and-developer-documentation)
  - [11. Operational Documentation](#11-operational-documentation)
  - [12. Onboarding Documentation](#12-onboarding-documentation)
    - [12.1 Onboarding path (example for this organisation)](#121-onboarding-path-example-for-this-organisation)
    - [12.2 Onboarding checklist template](#122-onboarding-checklist-template)
  - [13. Diagrams as Code](#13-diagrams-as-code)
  - [14. Decisions, RFCs, and Meeting Notes](#14-decisions-rfcs-and-meeting-notes)
    - [14.1 Where decisions live](#141-where-decisions-live)
    - [14.2 RFC process](#142-rfc-process)
    - [14.3 Meeting notes](#143-meeting-notes)
  - [15. Knowledge Management](#15-knowledge-management)
    - [15.1 Information architecture](#151-information-architecture)
    - [15.2 Search and findability](#152-search-and-findability)
    - [15.3 Promoting knowledge from chat](#153-promoting-knowledge-from-chat)
  - [16. Keeping Docs Fresh](#16-keeping-docs-fresh)
  - [17. Documentation in the Definition of Done](#17-documentation-in-the-definition-of-done)
  - [18. Documentation Metrics](#18-documentation-metrics)
  - [19. Documentation for AI Assistants](#19-documentation-for-ai-assistants)
  - [20. Localisation and Accessibility of Docs](#20-localisation-and-accessibility-of-docs)
  - [21. Security and Compliance of Documentation](#21-security-and-compliance-of-documentation)
  - [22. Maintaining This Handbook](#22-maintaining-this-handbook)
    - [22.1 Structure and conventions](#221-structure-and-conventions)
    - [22.2 Repository hygiene](#222-repository-hygiene)
    - [22.3 Review cadence](#223-review-cadence)
    - [22.4 Contribution workflow](#224-contribution-workflow)
  - [23. Checklists](#23-checklists)
    - [Per repository](#per-repository)
    - [Per team](#per-team)
    - [Organisation](#organisation)
  - [24. References](#24-references)
    - [Books](#books)

---

## 1. Why Documentation Matters

| Without documentation | With good documentation |
|---|---|
| Knowledge lives in a few heads (bus factor = 1) | Knowledge survives turnover and leave |
| Onboarding takes months | New engineers contribute in days/weeks |
| Incidents last longer (no runbooks) | Faster diagnosis and mitigation |
| Decisions are re-litigated | Rationale is recorded (ADRs) |
| Audits are painful | Evidence exists as a by-product |
| APIs are hard to adopt | Integrations are fast and self-serve |

DORA research has repeatedly found that **quality documentation** is associated with better implementation of technical practices and better organisational performance.

---

## 2. Principles

| # | Principle |
|---:|---|
| 1 | **Audience first** — every document states who it is for and what they will be able to do after reading. |
| 2 | **Docs live close to what they describe** — code docs in the repo, service docs with the service, org-wide knowledge in a well-structured hub. |
| 3 | **Single source of truth** — link, don't copy. Duplicated docs diverge. |
| 4 | **Docs-as-code** — versioned, reviewed, tested, published by CI. |
| 5 | **Small and current beats complete and stale** — a short, correct README is worth more than a wrong 50-page wiki. |
| 6 | **Every doc has an owner and a review date.** |
| 7 | **Write for scanning** — headings, tables, lists, code blocks, and summaries first. |
| 8 | **Documentation is part of "done"** — not a follow-up task. |

---

## 3. The Diátaxis Framework

**Diátaxis** (Daniele Procida) identifies four distinct kinds of documentation, each serving a different user need. Mixing them in one page is the most common cause of confusing docs.

```text
                         PRACTICAL (doing)                     THEORETICAL (knowing)
                 ┌────────────────────────────────┬────────────────────────────────┐
 LEARNING        │  TUTORIALS                      │  EXPLANATION                    │
 (acquisition)   │  learning-oriented lessons      │  understanding-oriented         │
                 │  "Build your first service"     │  "Why we use event sourcing"    │
                 ├────────────────────────────────┼────────────────────────────────┤
 WORKING         │  HOW-TO GUIDES                  │  REFERENCE                      │
 (application)   │  task-oriented recipes          │  information-oriented facts     │
                 │  "How to rotate DB credentials" │  "Config options", "API spec"   │
                 └────────────────────────────────┴────────────────────────────────┘
```

| Type | Answers | Characteristics | Example in this handbook |
|---|---|---|---|
| **Tutorial** | "Teach me" | Step-by-step, guaranteed success, minimal explanation | 41(a)/41(b) fresh setup guides |
| **How-to guide** | "How do I…?" | Goal-oriented, assumes competence, focused steps | 42 Git setup, 39 VPS setup sections |
| **Reference** | "What exactly is…?" | Accurate, complete, consistent structure, no narrative | 25–28 command references, 37 registries |
| **Explanation** | "Why…?" | Context, background, trade-offs, alternatives | 01 Foundations, 04 Architecture |

https://diataxis.fr/

---

## 4. Documentation Types and Ownership

| Document | Diátaxis type | Location | Owner | Review |
|---|---|---|---|---|
| `README.md` (repo) | How-to + reference | Repo root | Repo owners | Every significant change |
| `CONTRIBUTING.md` | How-to | Repo root | Repo owners | Yearly |
| `SECURITY.md` | Reference | Repo root | Security + owners | Yearly |
| `CHANGELOG.md` / release notes | Reference | Repo / release pages | Release automation + PM | Every release |
| ADRs | Explanation | `docs/adr/` | Tech lead | Immutable (superseded by new ADRs) |
| Design docs / RFCs | Explanation | `docs/design/` or RFC repo | Author | Until implemented, then archived |
| Architecture overview (C4, arc42) | Explanation + reference | `docs/architecture/` | Architects/tech lead | Quarterly |
| API reference (OpenAPI/AsyncAPI/proto) | Reference | `api/` → published portal | API owners | Every API change (CI) |
| Runbooks | How-to | `docs/runbooks/` or ops hub | Service owners/on-call | After every use; twice a year |
| Postmortems | Explanation | Postmortem archive | Incident authors | Final once reviewed |
| DR plans | How-to + reference | DR hub (available offline) | Service owners/SRE | After each DR test |
| Onboarding guides | Tutorial | Engineering handbook | Eng managers | Each new joiner gives feedback |
| Coding standards, policies | Reference + explanation | Engineering handbook (this repo) | Standards owners | Twice a year |
| User/product docs | All four | Help centre / docs site | Tech writers + PMs | Every release |

---

## 5. Docs-as-Code

Treat documentation like code: plain-text formats in version control, reviewed in pull requests, validated in CI, and published automatically.

### 5.1 Benefits

```text
✅ Same workflow as code (branches, PRs, reviews, CODEOWNERS)
✅ Docs change in the same PR as the code they describe
✅ Version history and blame; docs versioned with releases
✅ Automated checks (links, spelling, style, examples that compile/run)
✅ Easy to search with developer tools and AI assistants
```

### 5.2 Formats

| Format | Use |
|---|---|
| **Markdown** (CommonMark / GitHub Flavored Markdown) | Default for most engineering docs |
| MDX | Markdown + components (Docusaurus, Astro/Starlight) |
| AsciiDoc | Large technical books/manuals (Antora) |
| reStructuredText | Python ecosystem (Sphinx) |
| OpenAPI/AsyncAPI/proto | API reference sources |
| Mermaid/PlantUML/D2/Structurizr DSL | Diagrams (§13) |

### 5.3 Repository docs layout (service)

```text
docs/
├── index.md                 # landing page: what, who, links
├── getting-started.md       # tutorial
├── how-to/                  # task guides
├── reference/               # config, CLI, events, error codes
├── architecture/            # C4 diagrams, arc42 sections
├── adr/                     # decisions
├── runbooks/                # operational procedures
└── postmortems/             # (or central archive)
mkdocs.yml / docusaurus.config.ts
```

---

## 6. Documentation Toolchain and CI

### 6.1 Static site generators

| Tool | Ecosystem | Strengths |
|---|---|---|
| **MkDocs + Material for MkDocs** | Python | Fast setup, great search, admonitions, versioning (mike), widely used for engineering docs |
| **Docusaurus** | React | Versioned docs, blog, i18n, MDX |
| **Starlight** (Astro) | Astro | Fast, accessible, i18n, modern defaults |
| **Sphinx** | Python | API docs from docstrings, cross-referencing |
| **Antora** | AsciiDoc | Multi-repo, versioned documentation sites |
| **Hugo** | Go | Very fast builds, flexible |
| **Backstage TechDocs** | MkDocs-based | Docs next to services in the developer portal |
| GitHub/GitLab wikis / rendered Markdown | Built-in | Zero setup; limited structure |

### 6.2 Quality tooling

| Check | Tool |
|---|---|
| Markdown lint | **markdownlint** (markdownlint-cli2) |
| Prose style & terminology | **Vale** with Google/Microsoft style packages + custom vocabulary |
| Spelling | cspell, codespell |
| Broken links | **lychee**, markdown-link-check |
| Formatting | Prettier |
| Code samples | Extract and compile/test examples (doctest, mdx testing, example repos in CI) |
| Diagrams | Render Mermaid/PlantUML in CI to catch syntax errors |

### 6.3 CI pipeline for docs

```yaml
# .github/workflows/docs.yml (illustrative)
name: docs
on:
  pull_request: { paths: ['docs/**', '*.md', 'mkdocs.yml'] }
  push: { branches: [main], paths: ['docs/**', '*.md', 'mkdocs.yml'] }
permissions: { contents: read }
jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@<full-commit-sha>
      - uses: DavidAnson/markdownlint-cli2-action@<full-commit-sha>
        with: { globs: '**/*.md' }
      - uses: errata-ai/vale-action@<full-commit-sha>
      - uses: lycheeverse/lychee-action@<full-commit-sha>
        with: { args: --no-progress --exclude-mail './**/*.md' }
      - run: pip install mkdocs-material && mkdocs build --strict
  publish:
    needs: check
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    permissions: { contents: read, pages: write, id-token: write }
    steps:
      - run: echo "deploy built site to GitHub Pages / internal docs host"
```

### 6.4 MkDocs Material starter

```yaml
# mkdocs.yml
site_name: ShopNow Engineering Handbook
repo_url: https://github.com/shopnow/engineering-handbook
theme:
  name: material
  features: [navigation.sections, navigation.instant, search.highlight, content.code.copy]
markdown_extensions:
  - admonition
  - pymdownx.details
  - pymdownx.superfences:
      custom_fences:
        - { name: mermaid, class: mermaid, format: !!python/name:pymdownx.superfences.fence_code_format }
  - tables
  - toc: { permalink: true }
nav:
  - Home: index.md
  - Foundations: 01-engineering-foundations.md
  - SDLC: 02-software-development-lifecycle.md
```

---

## 7. Writing Style

### 7.1 Style guides to adopt (don't write your own from scratch)

- **Google developer documentation style guide** — https://developers.google.com/style
- **Microsoft Writing Style Guide** — https://learn.microsoft.com/style-guide/
- Plain-language guidance (e.g. plainlanguage.gov principles)

### 7.2 Core rules

| Rule | ❌ | ✅ |
|---|---|---|
| Lead with the outcome | "This document describes various aspects of…" | "Use this guide to rotate database credentials without downtime." |
| Second person, active voice, present tense | "The config file should be edited by the user." | "Edit the config file." |
| One idea per sentence; short paragraphs | Long compound sentences | Short, direct sentences |
| Imperative steps, numbered | Prose paragraphs of instructions | 1. Run… 2. Verify… |
| Specific over vague | "Wait a while" | "Wait until the status shows `available` (about 5 minutes)." |
| Show expected output | Commands only | Command + expected output + what to do if different |
| Define terms on first use; link to glossary | Unexplained acronyms | "Recovery point objective (RPO)" |
| Inclusive language | "whitelist/blacklist", "master/slave" | "allowlist/denylist", "primary/replica" |
| Consistent terminology | customer / client / user interchangeably | One term per concept (chapter 06 §4) |
| Accessible | "Click the green button" | "Select **Save**" (don't rely on colour) |
| Dates and times | "next Friday", "5 pm" | "2026-10-09 17:00 IST" |

### 7.3 Admonitions — use sparingly

```markdown
> **Note:** Additional helpful information.
> **Warning:** Something that can cause data loss or downtime.
```

---

## 8. Templates

### 8.1 Repository README

~~~markdown
# orders-service

Orders API and order lifecycle for ShopNow. **Tier 0** · Owner: `team-orders` · On-call: `#orders-oncall`

## What it does
One paragraph. Link to architecture overview and API docs.

## Quick start (local)
```bash
cp .env.example .env            # no real secrets — see docs/how-to/local-secrets.md
docker compose up -d            # PostgreSQL, Valkey, WireMock
./gradlew bootRun
curl -s localhost:8080/health/ready
```

## Common tasks
- Run tests: `./gradlew check`
- Create a migration: see docs/how-to/migrations.md
- Deploy: merged to main → staging automatically; production via GitOps promotion (docs/how-to/deploy.md)

## Configuration
Link to docs/reference/configuration.md (generated where possible).

## Operations
Dashboards · Runbooks · SLOs · Alerts — links

## Contributing
See CONTRIBUTING.md. Coding standards: engineering handbook ch. 06.
~~~

### 8.2 How-to guide

```markdown
# How to <achieve goal>
**Audience:** <role> · **Time:** ~15 min · **Prerequisites:** <access, tools>

## Steps
1. <Action> — `command`
   Expected: <output>
2. …

## Verify
How to confirm success.

## Troubleshooting
| Symptom | Cause | Fix |

## Related
Links.
```

### 8.3 Other templates in this handbook

| Template | Location |
|---|---|
| PRD, SRS, use case, change request | Chapter 03 §12, §15 |
| ADR (Nygard/MADR) | Chapter 04 §9 |
| Design doc / RFC | Chapter 05 §3.2 |
| Threat model | Chapter 09 §4.3 |
| Test plan, test case, test completion report | Chapter 08 §6–10 |
| Load test report | Chapter 19 §5.6 |
| SLO document | Chapter 18 §10.3 |
| DR plan, DR test report, BIA | Chapter 20 §12, §15, §17 |
| Runbook, postmortem, on-call handoff, known-error record | Chapter 21 |

---

## 9. Architecture Documentation

```text
Minimum per system:
- System context and container diagrams (C4 L1/L2) — chapter 04 §8
- Deployment view (where it runs, regions/AZs)
- Key runtime flows as sequence diagrams
- ADRs for significant decisions
- Quality goals and constraints (arc42 sections 1, 2, 10)
- Risks and technical debt (arc42 section 11)
Keep diagrams as code next to the system; render in the docs site.
```

ISO/IEC/IEEE 42010:2022 concepts (stakeholders, concerns, viewpoints, views) help check that the architecture description answers the right questions for each audience (chapter 04 §2).

---

## 10. API and Developer Documentation

| Element | Practice (details in chapter 11 §22) |
|---|---|
| Reference | Generated from OpenAPI/AsyncAPI/proto — never hand-maintained separately |
| Getting started | First successful call in < 10 minutes, with copy-paste examples |
| Guides | Authentication, pagination, errors, idempotency, webhooks, rate limits, versioning |
| Changelog | Dated entries; breaking changes and migration guides |
| SDK docs | Generated + hand-written usage examples per language |
| Error catalogue | Every error `code` with meaning and resolution |
| Try-it consoles & sandbox | Safe test environment and keys |

---

## 11. Operational Documentation

| Document | Requirements |
|---|---|
| Runbooks | Linked from alerts; copy-paste commands; owner; last-used/reviewed dates (chapter 21 §12) |
| Service overview for on-call | Architecture, dependencies, dashboards, common failures, escalation contacts |
| DR plans | Available offline/outside the primary region (chapter 20 §15) |
| Postmortems | Searchable archive with tags (service, cause category, severity) |
| Known-error records | Linked from support macros and runbooks |
| Access and break-glass procedures | Securely stored; tested |

```text
🔴 Operational docs must be reachable when production is down — host them independently of the systems they describe
   (or keep exported copies), and test access during DR exercises.
```

---

## 12. Onboarding Documentation

### 12.1 Onboarding path (example for this organisation)

```text
Day 1:   Laptop setup — 41(a) Windows 11 or 41(b) macOS; Git & SSH — 42; IDE — 30 + your IDE chapter (31–36)
Week 1:  Engineering foundations — 01; SDLC — 02; coding standards — 06; version control workflow — 07;
         team's service READMEs; first small PR merged
Week 2:  Testing — 08; security basics — 09; your platform chapter (12–15); observability — 18; shadow on-call
Month 1: Architecture of your domain (ADRs, C4); production operations — 21; deliver a feature end-to-end
Month 2–3: Reverse-shadow on-call; contribute an improvement to docs or tooling
```

### 12.2 Onboarding checklist template

```markdown
- [ ] Accounts: SSO, Git host, CI, cloud (read), observability, ticketing, chat, paging (when on-call)
- [ ] Development environment working: build + tests pass locally
- [ ] Read: handbook chapters 01, 02, 06, 07; team README and architecture overview
- [ ] Meet: buddy, manager, tech lead, product manager, on-call lead
- [ ] First PR merged (docs fix or small change) within week 1
- [ ] Feedback on onboarding docs submitted (what was confusing/outdated?)
```

New joiners are the best testers of documentation — make "fix one onboarding doc" part of every onboarding.

---

## 13. Diagrams as Code

| Tool | Strengths | Rendering |
|---|---|---|
| **Mermaid** | Flowcharts, sequence, state, ER, class, Gantt, C4 (experimental), Git graphs | Native on GitHub, GitLab, many docs sites |
| **PlantUML** (+ C4-PlantUML) | Mature UML, C4 support | Server/CLI rendering; IDE plugins (31–33) |
| **Structurizr DSL** | Model-based C4 (one model, many views) | Structurizr tooling |
| **D2** | Modern declarative diagrams, good layouts | CLI |
| **Graphviz (DOT)** | Graphs, dependency trees | CLI |
| Excalidraw / draw.io (diagrams.net) | Hand-drawn style / general diagramming | Store source files (`.excalidraw`, `.drawio`) in Git |

```text
🔴 Store diagram SOURCE in Git next to docs; rendered images are build outputs
🟠 Every diagram: title, legend/key, date or version, and a short text explanation (accessibility + searchability)
🟠 Prefer model-based tools (Structurizr) when many diagrams describe the same system
```

---

## 14. Decisions, RFCs, and Meeting Notes

### 14.1 Where decisions live

| Decision scope | Record |
|---|---|
| Architecture/technology (hard to reverse) | ADR in the repo (chapter 04 §9) |
| Cross-team proposals | RFC process (below) |
| Product decisions | PRD decision log (chapter 03 §12.1) |
| Incident decisions | Incident timeline/postmortem |
| Operational policies | Handbook chapter or policy doc with version history |

### 14.2 RFC process

```text
1. Draft RFC (problem, proposal, alternatives, impact, rollout) in the RFC repo as a PR
2. Announce in engineering channel with a comment deadline (e.g. 1–2 weeks)
3. Discussion in PR comments; synchronous meeting only if needed
4. Decision by designated approvers (recorded in the RFC: accepted/rejected/deferred + rationale)
5. Accepted RFCs → implementation tickets; significant ones → ADRs in affected repos
```

### 14.3 Meeting notes

```markdown
# <Meeting> — 2026-10-02
Attendees: · Facilitator: · Note-taker:
## Decisions
- DECISION: … (owner, date)
## Action items
| Action | Owner | Due |
## Notes (brief)
```

Rule: decisions and action items go at the top; notes are optional. Anything decided must be findable later (link from tickets/ADRs).

---

## 15. Knowledge Management

### 15.1 Information architecture

```text
Engineering handbook (this repo)     → org-wide standards, practices, onboarding (reference + explanation)
Service repos (docs/)                → service-specific docs (all four Diátaxis types)
Developer portal (Backstage)         → catalogue + TechDocs aggregated search across services
Wiki (Confluence/Notion)             → team pages, meeting notes, project spaces (link to repos for technical truth)
Ticketing/ITSM                       → work history, known errors
Chat                                 → ephemeral — promote valuable answers to docs ("doc it, don't chat it")
```

### 15.2 Search and findability

```text
- One entry point (developer portal or docs hub) with search across repos and wikis
- Consistent titles ("How to …", "Runbook: …", "ADR-0011: …")
- Tags/metadata: service, team, doc type, last reviewed
- Redirects when moving pages; no dead links
- "Can't find it?" channel monitored; recurring questions become FAQs/how-tos
```

### 15.3 Promoting knowledge from chat

```text
When the same question is answered twice in chat → write a how-to or FAQ entry and reply with the link.
Tag answers in chat (e.g. a 📝 emoji) for a weekly "docs gardening" pass.
```

---

## 16. Keeping Docs Fresh

| Mechanism | How |
|---|---|
| Ownership metadata | Front matter: `owner`, `last_reviewed`, `review_interval` |
| Review reminders | CI/bot flags docs past their review date; opens tickets to owners |
| Docs in PR templates | "Docs updated?" checkbox; CODEOWNERS for docs |
| Generated reference | Config, CLI, API, metrics, events reference generated from source |
| Tested examples | Code snippets compiled/tested in CI |
| Archival | Mark outdated docs as archived (banner) rather than silently deleting; remove from search |
| Docs gardening | Scheduled time (e.g. quarterly "docs day") to prune and update |

```yaml
---
title: Runbook — Orders high error rate
owner: team-orders
last_reviewed: 2026-09-15
review_interval: 180d
tags: [runbook, orders, tier-0]
---
```

---

## 17. Documentation in the Definition of Done

Add to the team DoD (chapter 02 §12.2):

```markdown
- [ ] README/how-to updated if setup, commands, or behaviour changed
- [ ] API reference regenerated; changelog entry added for API changes
- [ ] Configuration reference updated for new settings
- [ ] Runbooks/alerts/dashboards updated for new failure modes
- [ ] ADR written for significant decisions
- [ ] User-facing docs/release notes updated (with product/tech writing)
```

---

## 18. Documentation Metrics

| Metric | Signal |
|---|---|
| % of services with README, runbooks, architecture overview, owner | Coverage |
| % of docs past review date | Freshness |
| Broken links count | Hygiene |
| Search success (searches with clicks) / zero-result searches | Findability |
| Time to first PR for new joiners | Onboarding effectiveness |
| Repeat questions in support/chat channels | Gaps |
| Support ticket deflection (help-centre views vs tickets) | User docs effectiveness |
| Docs feedback ("Was this helpful?") | Quality |

---

## 19. Documentation for AI Assistants

AI coding assistants and agents read repository content. Good documentation makes them more accurate and keeps them within your standards (chapter 01 §19, chapter 06 §24).

| Practice | Detail |
|---|---|
| Repository context files | Emerging conventions such as `AGENTS.md` or tool-specific instruction files describe build/test commands, conventions, architecture boundaries, and "don'ts" for agents — keep them short, accurate, and linked to the real docs |
| Docs site machine-readability | Clean Markdown sources; some sites publish an `llms.txt` (a community proposal) summarising key docs for LLMs |
| Single source of truth | Agents follow whatever they find; stale docs produce stale code |
| No secrets in docs | Anything in the repo may be sent to AI tools |
| Examples that compile | Agents copy examples — make sure they are correct and current |

```markdown
# AGENTS.md (example)
- Build: `./gradlew build` · Test: `./gradlew check` (uses Testcontainers; Docker required)
- Architecture: hexagonal — domain must not import infrastructure (ArchUnit enforces)
- Standards: engineering handbook chapters 05, 06; Conventional Commits; small PRs
- Never commit secrets; never modify files under db/migrations that are already applied — add new migrations
- API changes: update api/openapi.yaml first; run `./gradlew openApiDiff`
```

---

## 20. Localisation and Accessibility of Docs

```text
- Write in clear, simple English (global and multilingual teams); avoid idioms and slang
- Localise user-facing docs for target markets (e.g. Hindi and regional languages for Indian consumer products) with
  translation management tools and glossaries
- Accessible docs: semantic headings, alt text for images, text descriptions for diagrams, sufficient contrast,
  code blocks with language tags, tables with headers
- Date/time formats explicit (ISO 8601) with time zones (IST/UTC)
```

---

## 21. Security and Compliance of Documentation

```text
🔴 No secrets, credentials, private keys, or tokens in docs (secret scanning covers docs too)
🔴 Classify docs (public / internal / confidential); restrict access for sensitive runbooks and architecture details
🔴 No real customer personal data in examples, screenshots, or postmortems — use synthetic data
🟠 Security-sensitive procedures (break-glass, key ceremonies) in access-controlled locations with audit
🟠 Retain documentation required for compliance (change records, DR test reports, policies) per retention rules
🟠 Public docs reviewed for information disclosure (internal hostnames, IP ranges, vulnerable versions)
```

---

## 22. Maintaining This Handbook

This repository is itself a docs-as-code project. Recommended practices:

### 22.1 Structure and conventions

```text
- One chapter per file, numbered (NN-topic.md); each chapter has: title, summary, TOC, numbered sections,
  checklists, references, previous/next links
- Status markers: 🔴 REQUIRED · 🟠 RECOMMENDED · 🟡 OPTIONAL · ⚪ REFERENCE
- Normative keywords per RFC 2119/8174 where precision matters (MUST/SHOULD/MAY)
- Versions and dates stated as "as of <month year>" with "verify" notes for fast-changing facts
- Cross-link instead of duplicating (e.g. setup lives in 41/42; strategy chapters link to them)
```

### 22.2 Repository hygiene

```text
- Add a README.md index (chapters with one-line descriptions and reading paths from chapter 01 §1)
- Resolve numbering conflicts (e.g. multiple files numbered 37) — rename to 37a/37b/37c or renumber, with redirects in the README
- Merge or clearly differentiate overlapping files (e.g. git-github-setup.md vs 42-Git-GitHub-GitLab-Bitbucket-Setup.md)
- markdownlint + lychee + Vale in CI; publish with MkDocs Material or Docusaurus
- CODEOWNERS per chapter; PR template with "sources checked" checkbox
```

### 22.3 Review cadence

| Content type | Review |
|---|---|
| Version-sensitive facts (Node/Java/PostgreSQL/Kubernetes versions, store requirements, standards editions) | Every 3–6 months |
| Practices and checklists | Yearly |
| After major incidents or tooling changes | As needed |
| Annual "handbook health" pass | Owners verify every chapter's references and examples |

### 22.4 Contribution workflow

```text
Issue/idea → branch `docs/<chapter>-<topic>` → edit with sources (official docs preferred) → CI checks
→ review by chapter owner (+ SME for security/legal topics) → squash merge → site auto-publishes → changelog entry
```

---

## 23. Checklists

### Per repository
- [ ] README with quick start, common tasks, ops links, owner
- [ ] CONTRIBUTING, SECURITY.md, CHANGELOG
- [ ] `docs/` with how-to, reference, architecture, ADRs, runbooks as relevant
- [ ] Docs CI: lint, links, build; examples tested
- [ ] Ownership and review dates in front matter

### Per team
- [ ] Docs part of Definition of Done
- [ ] Runbook for every paging alert
- [ ] Onboarding path documented and tested by the latest joiner
- [ ] Quarterly docs gardening

### Organisation
- [ ] Engineering handbook (this repo) with index, owners, CI, and publishing
- [ ] Developer portal / single search entry point
- [ ] Style guide adopted (Google or Microsoft) + Vale rules
- [ ] Diagrams as code standard
- [ ] Postmortem and ADR archives searchable
- [ ] Documentation metrics reviewed

---

## 24. References

- Diátaxis: https://diataxis.fr/
- Google developer documentation style guide: https://developers.google.com/style
- Microsoft Writing Style Guide: https://learn.microsoft.com/style-guide/welcome/
- Write the Docs (community, docs-as-code guide): https://www.writethedocs.org/guide/docs-as-code/
- DORA — documentation quality capability: https://dora.dev/capabilities/documentation-quality/
- MkDocs: https://www.mkdocs.org/ · Material for MkDocs: https://squidfunk.github.io/mkdocs-material/
- Docusaurus: https://docusaurus.io/ · Starlight: https://starlight.astro.build/ · Sphinx: https://www.sphinx-doc.org/ · Antora: https://antora.org/
- Backstage TechDocs: https://backstage.io/docs/features/techdocs/
- markdownlint: https://github.com/DavidAnson/markdownlint · Vale: https://vale.sh/ · lychee: https://lychee.cli.rs/
- Mermaid: https://mermaid.js.org/ · PlantUML: https://plantuml.com/ · Structurizr: https://structurizr.com/ · D2: https://d2lang.com/
- arc42: https://arc42.org/ · C4 model: https://c4model.com/ · ADRs: https://adr.github.io/
- CommonMark: https://commonmark.org/ · GitHub Flavored Markdown: https://github.github.com/gfm/
- AGENTS.md convention: https://agents.md/
- llms.txt proposal: https://llmstxt.org/

### Books
- *Docs for Developers* — Bhatti, Corleissen, Lambourne, Nunez, Waterhouse
- *Docs Like Code* — Anne Gentle
- *The Product is Docs* — Christopher Gales & the Splunk Documentation Team

---

**Previous:** [21 — Production Operations](./21-production-operations.md) · **Next:** [23 — Team Collaboration](./23-team-collaboration.md)