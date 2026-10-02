# 🛠️ Production Operations — Production Engineering Guide

> How to run production services day to day: operating models (you build it, you run it; SRE; platform teams), ITIL 4 practices adapted for modern teams, on-call design and health, incident management (severity levels, roles, lifecycle, communication templates), blameless postmortems, problem management, runbooks and runbook automation, support tiers and escalation, access and service requests, maintenance and patching, status pages and customer communication, operational metrics, toil reduction and ChatOps, operational reviews, tooling, and AI-assisted operations.
>
> Related: [02 SDLC — operate & maintain phases](./02-software-development-lifecycle.md) · [09 Security incident response §22](./09-security-engineering.md#22-incident-response) · [16 Change management §21](./16-devops-and-ci-cd.md#21-change-management) · [18 Alerting & SLOs](./18-observability-and-monitoring.md) · [20 Reliability, DR & production readiness](./20-reliability-and-disaster-recovery.md) · [22 Documentation (runbooks)](./22-documentation-and-knowledge-management.md) · [39 VPS operations](./39-vps-enterprise-setup-guide.md)

---

## 📚 Table of Contents

- [🛠️ Production Operations — Production Engineering Guide](#️-production-operations--production-engineering-guide)
  - [📚 Table of Contents](#-table-of-contents)
  - [1. Operating Models](#1-operating-models)
  - [2. ITIL 4 Practices — Adapted](#2-itil-4-practices--adapted)
  - [3. On-Call Design](#3-on-call-design)
    - [3.1 Rotation patterns](#31-rotation-patterns)
    - [3.2 Paging policy](#32-paging-policy)
    - [3.3 Escalation policy example](#33-escalation-policy-example)
    - [3.4 On-call readiness (before someone joins the rotation)](#34-on-call-readiness-before-someone-joins-the-rotation)
    - [3.5 Handoffs](#35-handoffs)
  - [4. On-Call Health and Sustainability](#4-on-call-health-and-sustainability)
  - [5. Incident Management — Overview](#5-incident-management--overview)
  - [6. Severity Levels](#6-severity-levels)
  - [7. Incident Roles](#7-incident-roles)
  - [8. Incident Lifecycle](#8-incident-lifecycle)
    - [Triage questions (first 5–10 minutes)](#triage-questions-first-510-minutes)
    - [Incident channel conventions](#incident-channel-conventions)
  - [9. Incident Communication](#9-incident-communication)
    - [9.1 Internal update template (every 15–30 min for SEV1, 30–60 min for SEV2)](#91-internal-update-template-every-1530-min-for-sev1-3060-min-for-sev2)
    - [9.2 Status page templates](#92-status-page-templates)
    - [9.3 Stakeholder matrix](#93-stakeholder-matrix)
  - [10. Blameless Postmortems](#10-blameless-postmortems)
    - [10.1 When required](#101-when-required)
    - [10.2 Blameless principles](#102-blameless-principles)
    - [10.3 Postmortem template](#103-postmortem-template)
    - [10.4 Action items](#104-action-items)
  - [11. Problem Management](#11-problem-management)
    - [Known-error record](#known-error-record)
  - [12. Runbooks and Runbook Automation](#12-runbooks-and-runbook-automation)
    - [12.1 Runbook template](#121-runbook-template)
    - [12.2 Runbook quality rules](#122-runbook-quality-rules)
    - [12.3 Automation ladder](#123-automation-ladder)
  - [13. Support Tiers and Escalation](#13-support-tiers-and-escalation)
    - [Escalation ticket template (support → engineering)](#escalation-ticket-template-support--engineering)
    - [Ticket SLAs (example)](#ticket-slas-example)
  - [14. Service Requests and Access Management](#14-service-requests-and-access-management)
  - [15. Maintenance, Patching, and Lifecycle Operations](#15-maintenance-patching-and-lifecycle-operations)
    - [Maintenance window policy](#maintenance-window-policy)
  - [16. Service Catalogue and Configuration Data](#16-service-catalogue-and-configuration-data)
  - [17. Status Pages and Customer Communication](#17-status-pages-and-customer-communication)
  - [18. Operational Metrics](#18-operational-metrics)
  - [19. Toil Reduction and ChatOps](#19-toil-reduction-and-chatops)
    - [19.1 Identifying toil](#191-identifying-toil)
    - [19.2 ChatOps](#192-chatops)
  - [20. Operational Reviews and Cadence](#20-operational-reviews-and-cadence)
  - [21. Tooling](#21-tooling)
  - [22. AI-Assisted Operations](#22-ai-assisted-operations)
  - [23. Scenario Playbooks](#23-scenario-playbooks)
    - [23.1 Small team (3–6 engineers), business-hours focus](#231-small-team-36-engineers-business-hours-focus)
    - [23.2 Growing SaaS (multiple teams, India + global customers)](#232-growing-saas-multiple-teams-india--global-customers)
    - [23.3 Regulated enterprise](#233-regulated-enterprise)
    - [23.4 Festival sale war room](#234-festival-sale-war-room)
  - [24. Checklists](#24-checklists)
    - [Service operational readiness](#service-operational-readiness)
    - [During an incident](#during-an-incident)
    - [After an incident](#after-an-incident)
    - [Monthly](#monthly)
  - [25. References](#25-references)
    - [SRE and incident management](#sre-and-incident-management)
    - [Service management](#service-management)
    - [Regulatory (India)](#regulatory-india)
    - [Books](#books)

---

## 1. Operating Models

| Model | Description | Pros | Cons |
|---|---|---|---|
| **You build it, you run it** | Product teams own their services in production, including on-call | Strong ownership, fast feedback, quality incentives | Needs good platforms and on-call support; load on small teams |
| **SRE team (embedded or central)** | Reliability specialists partner with product teams; may share on-call | Expertise, consistent practices | Risk of "throwing over the wall" if ownership is unclear |
| **Platform team + product teams** | Platform provides paved roads (infra, CI/CD, observability); product teams run their services | Scales well; self-service | Platform must be treated as a product |
| **Traditional ops / NOC** | Separate operations team runs software built by others | Familiar in enterprises; 24×7 eyes | Slow feedback, blame dynamics, knowledge gaps |
| **Managed service provider** | Outsourced operations | Coverage without hiring | Context and accountability gaps; contract-driven |

```text
Recommended default: product teams own services end to end (incl. on-call) on top of a platform team's
paved roads, with SRE expertise for critical systems and incident leadership. Ownership is recorded in the
service catalogue (§16).
```

---

## 2. ITIL 4 Practices — Adapted

**ITIL 4** (PeopleCert/AXELOS) describes 34 management practices. The ones most relevant to engineering teams, adapted for DevOps:

| ITIL 4 practice | Purpose | Modern implementation |
|---|---|---|
| **Incident management** | Restore normal service as quickly as possible | Paging, incident roles, chat-based war rooms (§5–9) |
| **Problem management** | Reduce likelihood and impact of incidents by finding causes and known errors | Postmortems, recurring-incident analysis, known-error records (§10–11) |
| **Change enablement** | Maximise successful changes by assessing risk and authorising | Pipelines as change control; standard/normal/emergency changes (chapter 16 §21) |
| **Monitoring and event management** | Observe services, record and report state changes | Observability stack, SLO alerts (chapter 18) |
| **Service level management** | Set clear, business-based targets | SLOs/SLAs, error budgets (chapter 18, 20) |
| **Service request management** | Handle pre-defined user-initiated requests | Self-service portals, automated workflows (§14) |
| **Information security management** | Protect information | Chapter 09 |
| **Service continuity management** | Keep services available during disasters | Chapter 20 |
| **Knowledge management** | Maintain and use information effectively | Runbooks, docs (chapter 22) |
| **Release / deployment management** | Make new/changed services available | CI/CD (chapter 16) |
| **Capacity and performance management** | Ensure services meet demand cost-effectively | Chapter 19 |
| **Service configuration management** | Accurate info on configuration and relationships | Service catalogue, IaC as source of truth (§16) |

> ISO/IEC 20000-1 is the certifiable standard for service management systems; ITIL is the most common practice framework used to meet it.

---

## 3. On-Call Design

### 3.1 Rotation patterns

| Pattern | Use |
|---|---|
| Weekly primary + secondary per team | Most product teams |
| **Follow-the-sun** (handoff across time zones, e.g. India ↔ Europe ↔ US) | Global teams; avoids night pages |
| Business-hours on-call + after-hours escalation to a smaller pool | Lower-criticality services |
| Dedicated incident commander rotation (separate from responders) | Larger organisations, major incidents |

```text
Team size: a sustainable rotation usually needs ≥ 6–8 engineers per 24×7 rotation (one week on-call every 6–8 weeks);
smaller teams should pool rotations across services or reduce after-hours scope.
```

### 3.2 Paging policy

| Rule | Status |
|---|:---:|
| Page only for urgent, user-impacting, actionable issues (SLO burn, data loss risk, security emergencies) | 🔴 |
| Everything else creates tickets for business hours | 🔴 |
| Every page links to a runbook and a dashboard | 🔴 |
| Acknowledge-time target (e.g. 5 min for SEV1/2) and auto-escalation to secondary, then manager | 🔴 |
| Alert deduplication/grouping so one problem pages once | 🟠 |

### 3.3 Escalation policy example

```text
0 min   Page primary on-call (push + phone)
5 min   Not acknowledged → page secondary
10 min  Not acknowledged → page engineering manager on-call
SEV1    Immediately also page incident commander rotation; notify leadership channel
```

### 3.4 On-call readiness (before someone joins the rotation)

```markdown
- [ ] Access verified: production read access, dashboards, logs, traces, cloud console (break-glass procedure known), VPN/ZTNA, paging app
- [ ] Shadowed at least one full rotation; reverse-shadowed one (led with a backup)
- [ ] Walked through top runbooks and the service architecture
- [ ] Knows how to declare an incident, page others, and update the status page
- [ ] Laptop, charger, and phone ready; working connectivity plan (mobile hotspot) during on-call week
```

### 3.5 Handoffs

```markdown
## On-call handoff — orders team — week 40
- Open incidents / follow-ups: INC-2316 (PSP latency) monitoring continues; vendor ticket #8821
- Noisy alerts this week: OrdersQueueDepthHigh fired 6× (non-actionable) → ticket OPS-412 to tune
- Risky changes upcoming: DB migration Thursday 14:00 IST (expand step), owner @dev2
- Things to watch: festival sale traffic ramp from Friday
```

---

## 4. On-Call Health and Sustainability

```text
🔴 Track pages per shift and after-hours pages; target a low, sustainable number (many teams aim for ≤ ~2 incidents per 12-hour shift as Google SRE suggests as an upper bound)
🔴 Compensation or time off in lieu for after-hours on-call per company/local labour policy
🔴 Post-incident recovery time after night-time incidents
🟠 Monthly on-call review: noise, toil, runbook gaps, burnout signals
🟠 Never let one person be the only one who can fix something (bus factor)
🟠 Psychological safety: escalating or asking for help is encouraged, never penalised
```

---

## 5. Incident Management — Overview

An **incident** is an unplanned interruption or degradation of a service (or a security event) that requires a response. Goals in order:

```text
1. Restore service (mitigate) — as quickly and safely as possible
2. Communicate — keep stakeholders and users informed
3. Preserve evidence — logs, timelines, especially for security incidents
4. Learn — postmortem and follow-up actions
```

> **Mitigate first, root-cause later.** Roll back, fail over, disable a flag, scale up, or shed load before investigating deeply.

Frameworks: Google SRE incident management, PagerDuty Incident Response docs, Incident Command System (ICS) concepts adapted from emergency services.

---

## 6. Severity Levels

| Severity | Definition (examples) | Response | Comms |
|---|---|---|---|
| **SEV1 — Critical** | Full outage of a critical service; data loss/corruption; security breach with customer data; payments failing for most users | Immediate page, incident commander, all-hands as needed, 24×7 until mitigated | Status page within 15–30 min; leadership; regulators/customers per obligations |
| **SEV2 — Major** | Significant degradation or outage for a subset of users/regions; critical feature impaired with workaround | Immediate page; IC assigned | Status page if user-visible; stakeholders |
| **SEV3 — Minor** | Limited impact, non-critical feature degraded, internal tool down | Business hours (or page if trending worse) | Team channel; ticket |
| **SEV4 — Low** | Cosmetic issue, potential risk, near miss | Ticket | None external |

```text
Rules:
- When in doubt, declare higher — downgrading is cheap, slow escalation is expensive
- Severity is about impact, not cause; it may change during the incident
- Security incidents follow chapter 09 §22 in parallel (legal/regulatory clocks, evidence handling)
```

---

## 7. Incident Roles

| Role | Responsibilities | Notes |
|---|---|---|
| **Incident Commander (IC)** | Leads the response, sets priorities, delegates, decides (e.g. rollback, failover), manages the clock | Does NOT debug; coordinates. Can hand over explicitly |
| **Operations / Tech Lead** | Directs technical investigation and mitigation | Owns changes to production during the incident |
| **Communications Lead** | Internal updates, status page, customer support briefings, exec updates | Uses templates and a fixed cadence |
| **Scribe** | Maintains the timeline: actions, decisions, findings with timestamps | Often aided by incident tooling/bots |
| **Subject-matter experts** | Investigate specific systems (DB, network, payments) | Join on request |
| **Customer liaison / Support lead** | Gathers customer reports, shares workarounds | For customer-facing incidents |
| **Executive sponsor** (SEV1) | Business decisions, external escalation, resources | Stays out of technical channel unless needed |

For small teams, one person may hold several roles — but **name** who is IC.

---

## 8. Incident Lifecycle

```text
DETECT → TRIAGE → DECLARE → MOBILISE → MITIGATE → RESOLVE → FOLLOW-UP
```

| Phase | Actions |
|---|---|
| **Detect** | SLO alert, synthetic check, customer report, support ticket spike, security alert |
| **Triage** | Acknowledge; assess scope and impact quickly (dashboards); is it real? which severity? |
| **Declare** | Open incident (tool/slash command): channel `#inc-2026-10-02-checkout-errors`, severity, IC, initial summary |
| **Mobilise** | Page needed roles/SMEs; start bridge if needed; set update cadence |
| **Mitigate** | Check recent changes first (deploys, flags, config, migrations, vendor status); roll back/disable/fail over/scale/shed |
| **Resolve** | Service restored and stable for an agreed observation period; close incident; keep monitoring |
| **Follow-up** | Postmortem scheduled (SEV1/2 mandatory), customer/regulatory follow-ups, action items tracked |

### Triage questions (first 5–10 minutes)

```text
- What is broken from the user's perspective? Since when? How many users/regions/tenants?
- What changed recently? (deploy markers, flag changes, config, infra changes, vendor status pages)
- Is it getting worse, stable, or recovering?
- What is the fastest safe mitigation? Who needs to approve it?
- Is data at risk? Is this possibly a security incident?
```

### Incident channel conventions

```text
- Pin the incident summary (impact, severity, IC, current status, next update time)
- Prefix messages: "ACTION:", "DECISION:", "FINDING:", "QUESTION:" — helps the scribe
- Side discussions in threads; keep the main channel readable
- Record commands run against production
```

---

## 9. Incident Communication

### 9.1 Internal update template (every 15–30 min for SEV1, 30–60 min for SEV2)

```markdown
**[SEV1] Checkout errors — Update #3 — 10:45 IST**
Status: Mitigating
Impact: ~35% of checkout attempts failing (UPI and cards) since 10:12 IST; browse/search unaffected
Current action: Rolled back orders service to v2.4.9 at 10:41; error rate dropping (35% → 8%)
Next steps: Confirm full recovery; investigate PSP connection pool exhaustion
Next update: 11:00 IST
IC: @asha · Tech lead: @rohan · Comms: @meera
```

### 9.2 Status page templates

```text
INVESTIGATING
"We are investigating reports of failed payments during checkout. Browsing and order history are not affected.
 Next update in 30 minutes."

IDENTIFIED
"We have identified the cause of failed payments and are applying a fix. Some customers may still see errors.
 Next update in 30 minutes."

MONITORING
"A fix has been applied and payments are succeeding normally. We are monitoring closely.
 If a payment failed, no money was deducted; if you see a deduction, it will be refunded automatically within X days."

RESOLVED
"Between 10:12 and 10:58 IST, some customers were unable to complete payments. The issue is resolved.
 We apologise for the inconvenience and will publish a summary of what happened."
```

```text
Rules: plain language; impact first; no blame on vendors in public; don't speculate on causes;
give times in the users' time zone (IST for Indian customers) and/or UTC; commit to next-update times and keep them.
```

### 9.3 Stakeholder matrix

| Audience | SEV1 | SEV2 | Channel |
|---|---|---|---|
| Engineering on-call & IC | Immediately | Immediately | Paging + incident channel |
| Customer support | Within 15 min | Within 30 min | Support channel + macro/workaround |
| Leadership | Within 30 min | If prolonged | Exec channel/email |
| Customers | Status page within 30 min | If user-visible | Status page, in-app banner, email for enterprise |
| Regulators / partners | Per legal obligations (e.g. CERT-In 6 h for reportable cyber incidents) | Per obligations | Formal channels via legal/compliance |

---

## 10. Blameless Postmortems

### 10.1 When required

```text
Mandatory: all SEV1 and SEV2 incidents; any data loss; any security incident; any missed SLO with significant budget burn;
           near misses that could have been SEV1
Optional:  SEV3 with interesting learning; recurring minor incidents (as a problem review)
Timing:    draft within 3–5 business days; review meeting within 1–2 weeks
```

### 10.2 Blameless principles

```text
- People make reasonable decisions with the information and tools they had — ask "how did the system make this possible?"
- Focus on contributing factors (plural) rather than a single "root cause"; complex systems fail for multiple reasons
- No names in a blaming context; roles instead ("the on-call engineer")
- Celebrate what went well (fast detection, good rollback) as much as gaps
- Hindsight bias warning: the timeline should show what was known at each moment
```

### 10.3 Postmortem template

```markdown
# Postmortem: Checkout payment failures — 2026-10-02
Status: Draft | In review | Final · Severity: SEV1 · Authors: · Reviewers: · Incident: INC-2316

## Summary
2–3 sentences: what happened, impact, duration, how resolved.

## Impact
- Users: ~35% of checkout attempts failed for 46 min (≈ 4 200 failed attempts); no duplicate charges
- Revenue impact (estimate): ₹X lakh delayed/lost orders
- SLO: Checkout availability error budget for the month: 62% consumed by this incident
- Data: none lost · Security: not a security incident

## Timeline (IST)
| Time | Event |
|---|---|
| 10:05 | Deploy of orders v2.5.0 completes (canary skipped due to config error) |
| 10:12 | Error rate rises; fast-burn SLO alert fires 10:15 |
| 10:17 | On-call acknowledges; incident declared SEV2 → upgraded SEV1 at 10:24 |
| 10:41 | Rollback to v2.4.9 |
| 10:58 | Recovery confirmed; monitoring |

## Contributing factors
1. New HTTP client defaulted to a pool size of 10 to the PSP (previously 100)
2. Canary stage was skipped because the pipeline config for the new service template omitted analysis
3. Load test did not include the payment path due to PSP sandbox rate limits

## What went well
- Fast detection via burn-rate alert (3 min)
- Rollback was a single command and took 2 min

## What went poorly / where we got lucky
- 7 min lost determining whether the PSP itself was degraded (no dependency dashboard)

## Action items
| # | Action | Type | Owner | Due | Ticket |
|---|---|---|---|---|---|
| 1 | Enforce canary analysis in the service template (pipeline fails if missing) | Prevent | @platform | 2026-10-16 | PLAT-901 |
| 2 | Pool-size config validated at startup with explicit values per dependency | Prevent | @orders | 2026-10-09 | ORD-412 |
| 3 | Dependency dashboard for PSP latency/errors/pool usage | Detect | @orders | 2026-10-09 | ORD-413 |
| 4 | Add payment path to load tests using PSP stub with realistic latency | Prevent | @qa | 2026-10-30 | QA-77 |

## Lessons learned
```

### 10.4 Action items

```text
🔴 Every action has an owner, due date, and ticket; tracked to completion in ops reviews
🟠 Prefer actions that prevent classes of problems (guardrails, automation) over "be more careful"
🟠 Classify: Prevent / Detect / Mitigate / Process
🟠 Share postmortems widely (engineering-wide reading, monthly summaries); keep a searchable archive (chapter 22)
```

---

## 11. Problem Management

| Concept | Meaning |
|---|---|
| **Problem** | The underlying cause (or potential cause) of one or more incidents |
| **Known error** | A problem with a documented cause and/or workaround |
| **Workaround** | Reduces/eliminates impact until a permanent fix |

```text
Problem management activities:
- Trend analysis: incidents grouped by service, cause category, alert; top recurring issues each month
- Reactive problems: from postmortems — systemic fixes that span teams
- Proactive problems: from FMEA, chaos experiments, capacity forecasts, vendor advisories
- Known-error records linked from runbooks and support macros so L1/L2 can act quickly
```

### Known-error record

```markdown
KE-031: Intermittent 502s from CDN during origin deploys
Symptoms: spikes of 502 at edge lasting ~30 s during rolling deploys
Cause: origin pods terminated before LB deregistration completes
Workaround: deploy during low traffic; increase preStop sleep to 10 s
Permanent fix: PLAT-877 (graceful shutdown standard) — status: in progress
```

---

## 12. Runbooks and Runbook Automation

### 12.1 Runbook template

```markdown
# Runbook: OrdersHighErrorRate
Service: orders · Owner: orders team · Last reviewed: 2026-09-15 · Last used: 2026-10-02

## What this alert means
5xx ratio for orders API is burning the 99.9% SLO error budget at > 14.4x (fast burn).

## Impact
Customers may fail to place orders.

## Dashboards & tools
- Service dashboard: <link> · Traces filtered by status=error: <link> · Logs query: <link>
- Recent deploys and flag changes: <link>

## Triage steps
1. Check deploy annotations: did a deploy/flag change happen in the last 30 min? → If yes, roll back (step A)
2. Check dependency panel: PSP, DB, cache — any red? → go to dependency section
3. Check error breakdown by route and error.type in traces

## Mitigation
A. Roll back: `./ops/rollback.sh orders production` (or revert GitOps commit <link to procedure>)
B. Disable feature: flag `checkout-payment-retry` → off in flag console
C. DB saturation: see runbook db-saturation.md
D. PSP degraded: enable `psp-secondary-routing` flag; post status page update

## Escalation
- Payments SME: @payments-oncall · DBA: @dba-oncall · Vendor: PSP support +91-…, account ID …

## Verification
Error ratio < 0.1% for 15 min; synthetic checkout passing.
```

### 12.2 Runbook quality rules

```text
🔴 Linked from every alert; reviewed at least twice a year and after every use
🔴 Commands are copy-pasteable and safe (dry-run options, confirmations for destructive steps)
🟠 Convert frequently executed runbooks into automation (scripts, ChatOps commands, workflow tools)
🟠 Test runbooks in game days (chapter 20 §18)
```

### 12.3 Automation ladder

```text
Level 0: tribal knowledge → Level 1: written runbook → Level 2: scripted steps run by humans
→ Level 3: one-click/ChatOps automation with audit → Level 4: auto-remediation with guardrails and alerts to humans
```

Tools: Rundeck/PagerDuty Process Automation, StackStorm, AWS Systems Manager Automation, Azure Automation, Ansible AWX/Automation Controller, Temporal workflows, custom ChatOps bots.

---

## 13. Support Tiers and Escalation

| Tier | Who | Handles |
|---|---|---|
| **L0 — Self-service** | Users | Help centre, status page, chatbots, account self-service |
| **L1 — Frontline support** | Customer support | Triage, known issues, workarounds (known-error records), data collection |
| **L2 — Technical support / ops** | Support engineers, operations | Deeper diagnosis, configuration issues, log analysis, runbook execution |
| **L3 — Engineering** | Owning product team | Bugs, code fixes, complex incidents |
| **L4 — Vendors** | Cloud, PSP, SaaS providers | Issues in third-party systems |

### Escalation ticket template (support → engineering)

```markdown
Title: [Orders] Customers charged but order shows "payment pending" (5 reports today)
Severity: SEV3 (consider SEV2 if > 20 reports/hour)
Customer impact: money deducted, no confirmation email
Examples: order IDs ord_…, ord_…; timestamps (IST); payment method UPI
Steps already taken: checked status page, PSP dashboard shows success
Attachments: screenshots, HAR files (PII redacted)
```

### Ticket SLAs (example)

| Priority | First response | Update cadence | Target resolution |
|---|---|---|---|
| P1 (SEV1 linked) | 15 min | 30 min | Incident-driven |
| P2 | 2 h | Daily | 3 business days |
| P3 | 1 business day | Weekly | Next release |
| P4 | 3 business days | — | Backlog |

---

## 14. Service Requests and Access Management

| Request type | Practice |
|---|---|
| Production access (read) | Role-based, granted via groups in IdP; time-bound for sensitive data |
| Production write / admin | Just-in-time elevation with approval, MFA, session recording, automatic expiry |
| Database access | Read replicas or masked datasets for analysis; direct prod DB access only via audited bastion/proxy |
| New environments / resources | Self-service via platform templates (chapter 16 §15) |
| Secrets rotation | Automated; manual requests tracked |
| Offboarding | Same-day revocation of all access (SSO deprovisioning, SCIM), key rotation if needed |

```text
🔴 Quarterly access reviews for production systems (who has access, why, still needed?)
🔴 Break-glass accounts: sealed credentials, dual control, alerts on use, reviewed after each use
```

---

## 15. Maintenance, Patching, and Lifecycle Operations

| Activity | Cadence (example) | Practice |
|---|---|---|
| OS/base image patching | Weekly–monthly; critical within SLA | Rebuild images; rolling replacement; unattended security upgrades on VMs (see **39**) |
| Dependency updates | Continuous (bots) | Automated PRs with CI (chapter 06 §25) |
| Runtime/framework upgrades | Per support lifecycle | Planned projects; track end-of-life dates |
| Database minor versions | Quarterly (with releases) | Managed maintenance windows or rolling replicas |
| Kubernetes upgrades | Every ~4–6 months | Chapter 17 §7.1 |
| Certificate renewals | Automated | Expiry monitoring as a backstop |
| Domain renewals | Annually/auto-renew | Registrar lock; multiple contacts |
| Backup restore tests | Weekly–monthly | Chapter 20 §17 |
| Access reviews | Quarterly | §14 |
| Capacity reviews | Quarterly + before peaks | Chapter 19 §14 |

### Maintenance window policy

```text
- Prefer zero-downtime maintenance (rolling, blue-green); windows only when unavoidable
- Schedule at lowest traffic (often early morning IST for India-focused products); avoid peak business periods
- Announce ≥ 3–7 days ahead for customer-visible maintenance (status page, email for enterprise customers)
- Change record with rollback plan; on-call aware; post-maintenance verification
```

---

## 16. Service Catalogue and Configuration Data

Every production service has a catalogue entry (Backstage, Port, Cortex, OpsLevel, or a structured repo file):

```yaml
# catalog-info.yaml (Backstage format)
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: orders
  description: Orders API and order lifecycle
  annotations:
    pagerduty.com/service-id: PXXXXXX
    grafana/dashboard-selector: "service=orders"
    backstage.io/techdocs-ref: dir:.
  links:
    - { url: https://runbooks.shopnow.example/orders, title: Runbooks }
  tags: [java, tier-0, pci-scope]
spec:
  type: service
  lifecycle: production
  owner: team-orders
  system: commerce
  dependsOn: [resource:orders-db, component:payments, component:catalog]
  providesApis: [orders-api]
```

```text
Required fields: owner team, on-call rotation, tier (RPO/RTO), SLOs, runbooks, dashboards, repository, dependencies,
data classification, compliance scope (PCI/DPDP), lifecycle stage.
IaC + GitOps repositories are the source of truth for configuration; the catalogue links to them.
```

---

## 17. Status Pages and Customer Communication

| Practice | Detail |
|---|---|
| Public status page | Components matching user-visible functions (Website, Mobile app, Checkout, Payments, API, Notifications) |
| Hosted independently | On a different provider/domain than production (so it works during outages) |
| Subscriptions | Email/SMS/webhook/RSS for customers |
| Uptime history | Per component; honest reporting builds trust |
| Scheduled maintenance | Posted in advance |
| Incident summaries | Customer-facing postmortem summaries for significant incidents (no blame, clear remediation) |
| Enterprise customers | Named contacts; SLA credit process; direct notification for SEV1 |

Tools: Atlassian Statuspage, Instatus, Better Stack, incident.io status pages, cachet-style self-hosted, cloud status features.

---

## 18. Operational Metrics

| Metric | Definition | Use |
|---|---|---|
| Incident count by severity | Per month/service | Trends; reliability investment |
| **MTTD** | Impact start → detection | Monitoring effectiveness |
| **MTTA** | Alert → acknowledgement | On-call responsiveness |
| **MTTM / MTTR** | Impact start → mitigated / resolved | Response effectiveness (use medians and distributions, not just means) |
| % incidents detected by monitoring vs customers | — | Observability gaps |
| Pages per on-call shift; after-hours pages | — | On-call health |
| Postmortem completion rate and action-item closure rate | — | Learning effectiveness |
| Repeat incidents | Same cause within 90 days | Problem management |
| Change-related incidents | Share caused by changes | Change safety (relates to DORA change fail rate) |
| Toil hours | Manual operational work per week | Automation priorities |
| SLO attainment / error budget consumption | — | Reliability outcome |

> Incident metrics can be gamed (e.g. not declaring incidents). Reward fast declaration and good postmortems, not low incident counts.

---

## 19. Toil Reduction and ChatOps

### 19.1 Identifying toil

Toil (Google SRE) is work that is manual, repetitive, automatable, tactical, without enduring value, and scales linearly with service growth.

```text
Toil log (per on-call week): task, frequency, time spent, automatable? → top items become backlog tickets
Examples: manual cert renewals, restarting stuck consumers, granting standard access, copying data for support,
          resizing disks, rotating logs, re-running flaky jobs
```

### 19.2 ChatOps

```text
/incident declare sev2 "checkout errors"        → creates channel, pages IC, opens status page draft
/deploy rollback orders production               → runs audited rollback with approvals per policy
/flag disable checkout-payment-retry             → flag change with audit trail
/runbook orders-high-error-rate                  → posts runbook link and dashboard
/oncall who orders                               → shows current on-call
```

```text
🔴 ChatOps actions authenticated, authorised (same RBAC as other paths), and audited
🟠 Destructive commands require confirmation and, for production, approval per policy
```

---

## 20. Operational Reviews and Cadence

| Cadence | Meeting | Content |
|---|---|---|
| Daily (during busy periods) | Ops stand-up (15 min) | Overnight pages, open incidents, risky changes today |
| Weekly | Team ops review (30 min) | Incidents, alerts noise, SLO/error budget, toil, on-call handoff |
| Monthly | Engineering operations review | Cross-team incidents, postmortem themes, action-item status, capacity, cost |
| Quarterly | Reliability review per critical service (chapter 20 §22.2) | SLO trends, DR tests, dependency risks, roadmap |
| Before major events | Readiness review | Capacity, freezes, war-room plan, vendor contacts |

---

## 21. Tooling

| Need | Tools (examples) |
|---|---|
| Paging & on-call scheduling | PagerDuty, Opsgenie (check vendor roadmap), Grafana OnCall/IRM, incident.io On-call, Splunk On-Call, Better Stack |
| Incident coordination | incident.io, FireHydrant, Rootly, PagerDuty Incident Response, Jira Service Management, Slack/Teams with bots |
| Status pages | Statuspage, Instatus, Better Stack, incident.io |
| Ticketing / ITSM | Jira Service Management, ServiceNow, Freshservice, Zendesk (customer support) |
| Runbook automation | Rundeck/PagerDuty Process Automation, AWS SSM Automation, Ansible Automation Platform, StackStorm |
| Service catalogue | Backstage, Port, Cortex, OpsLevel |
| Postmortem repository | Docs-as-code repo, Confluence/Notion, incident tools' built-in postmortems |
| Observability | Chapter 18 |

---

## 22. AI-Assisted Operations

| Use | Value | Guardrail |
|---|---|---|
| Incident summarisation and timeline drafting | Saves scribe time; faster stakeholder updates | Human review before external communication |
| Alert correlation / noise reduction | Groups related alerts | Don't suppress SLO pages automatically without review |
| Query assistance (PromQL/LogQL/SQL) | Faster investigation | Validate queries; avoid expensive queries on production |
| Runbook suggestions | Surfaces likely runbooks/known errors | Humans decide actions |
| Postmortem drafting | Structures facts | Keep blameless tone; verify facts |
| Automated remediation | Faster mitigation for well-understood issues | Only pre-approved, reversible actions with audit; never free-form agent actions against production without approval |

```text
🔴 AI tools get read-only access by default; production-changing actions go through the same approvals as humans
🔴 No customer PII or secrets sent to unapproved AI services (chapter 09 §7)
```

---

## 23. Scenario Playbooks

### 23.1 Small team (3–6 engineers), business-hours focus
```text
- Shared rotation across all services; after-hours paging only for SEV1 (checkout/payments down)
- External uptime monitoring + SLO fast-burn alerts → phone
- Simple incident process: declare in a channel, one IC, status page updates, postmortem for SEV1/2
- Monthly ops review; toil log
```

### 23.2 Growing SaaS (multiple teams, India + global customers)
```text
- Team-owned rotations + incident commander rotation
- Follow-the-sun with a partner region if available; otherwise compensate night shifts
- Incident tooling with Slack integration; status page per component; enterprise customer notification
- Service catalogue with owners/tiers; quarterly reliability reviews
```

### 23.3 Regulated enterprise
```text
- ITSM tool (Jira Service Management/ServiceNow) integrated with pipelines for change records and evidence
- Formal major-incident process with regulator notification steps (CERT-In, sector regulators)
- Segregation of duties; JIT access; audit trails retained
- DR drills and incident simulations with documented evidence
```

### 23.4 Festival sale war room
```text
- War room (virtual) with IC, SMEs, support lead, business owner; dashboards for SLOs and business KPIs
- Pre-approved mitigations: scale-up, shed non-critical features, enable waiting room, PSP routing changes
- Vendor contacts on standby; update cadence to leadership every hour
- Post-event review within one week
```

---

## 24. Checklists

### Service operational readiness
- [ ] Owner team and on-call rotation in the service catalogue
- [ ] Alerts mapped to runbooks; dashboards linked
- [ ] Severity definitions and escalation policies configured in paging tool
- [ ] Status page component exists; comms templates ready
- [ ] Access model (read/JIT write/break-glass) implemented and tested
- [ ] Maintenance and patching responsibilities defined

### During an incident
- [ ] Incident declared with severity and IC
- [ ] Recent changes checked; mitigation prioritised
- [ ] Updates at the committed cadence (internal + status page)
- [ ] Timeline captured; production commands recorded
- [ ] Security/regulatory implications assessed

### After an incident
- [ ] Postmortem (SEV1/2) drafted within 5 business days and reviewed
- [ ] Action items with owners, due dates, tickets; tracked in ops reviews
- [ ] Runbooks, alerts, dashboards updated
- [ ] Customer-facing summary published if appropriate
- [ ] Known-error records updated

### Monthly
- [ ] On-call health metrics reviewed (pages/shift, after-hours load)
- [ ] Alert noise review completed
- [ ] Toil top-5 items have automation tickets
- [ ] Access review for production (quarterly)

---

## 25. References

### SRE and incident management
- Google SRE Book — Managing Incidents: https://sre.google/sre-book/managing-incidents/
- Google SRE Book — Postmortem Culture: https://sre.google/sre-book/postmortem-culture/
- Google SRE Book — Being On-Call: https://sre.google/sre-book/being-on-call/
- Google SRE Workbook — Incident Response: https://sre.google/workbook/incident-response/
- PagerDuty Incident Response documentation (open): https://response.pagerduty.com/
- PagerDuty Postmortem guide: https://postmortems.pagerduty.com/
- Atlassian Incident Management Handbook: https://www.atlassian.com/incident-management/handbook
- Learning from Incidents community: https://www.learningfromincidents.io/

### Service management
- ITIL 4 (PeopleCert): https://www.peoplecert.org/
- ISO/IEC 20000-1:2018: https://www.iso.org/standard/70636.html
- Backstage Software Catalog: https://backstage.io/docs/features/software-catalog/

### Regulatory (India)
- CERT-In directions and reporting: https://www.cert-in.org.in/

### Books
- *Site Reliability Engineering* — Google
- *The Site Reliability Workbook* — Google
- *Incident Management for Operations* — Rob Schnepp, Ron Vidal, Chris Hawley
- *The DevOps Handbook* (2nd ed.) — Kim, Humble, Debois, Willis, Forsgren

---

**Previous:** [20 — Reliability & Disaster Recovery](./20-reliability-and-disaster-recovery.md) · **Next:** [22 — Documentation & Knowledge Management](./22-documentation-and-knowledge-management.md)