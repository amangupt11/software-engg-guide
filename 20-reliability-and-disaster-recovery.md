# 🛟 Reliability & Disaster Recovery — Production Engineering Guide

> How to keep services available, recover quickly when they fail, and survive disasters: reliability principles from Site Reliability Engineering, failure modes and failure-mode analysis, availability engineering and dependency maths, SLO-driven reliability and error-budget policies, resilience patterns, redundancy and fault isolation (cells, bulkheads, static stability), overload protection, data durability and backups, business continuity (ISO 22301) and business impact analysis, RPO/RTO tiers, DR strategies and runbooks, failover and failback, DR testing, chaos engineering and game days, reliability of dependencies and third parties, ransomware resilience, regulatory expectations, and reliability reviews.
>
> Related: [01 Reliability fundamentals §15](./01-engineering-foundations.md#15-reliability-fundamentals) · [04 Resilience patterns §14](./04-software-architecture.md#14-resilience-patterns) · [10 Backup & recovery §14](./10-database-engineering.md#14-backup-and-recovery) · [17 HA & multi-region §16](./17-cloud-and-infrastructure.md#16-high-availability-and-multi-region) · [18 SLOs & burn-rate alerts](./18-observability-and-monitoring.md#10-slis-slos-and-error-budgets) · [19 Overload & capacity](./19-performance-and-scalability.md) · [21 Incidents & on-call](./21-production-operations.md) · [39 VPS backups](./39-vps-enterprise-setup-guide.md)

---

## 📚 Table of Contents

- [🛟 Reliability \& Disaster Recovery — Production Engineering Guide](#-reliability--disaster-recovery--production-engineering-guide)
  - [📚 Table of Contents](#-table-of-contents)
  - [1. Reliability Principles](#1-reliability-principles)
  - [2. Vocabulary](#2-vocabulary)
  - [3. How Systems Fail](#3-how-systems-fail)
  - [4. Failure Mode and Effects Analysis (FMEA)](#4-failure-mode-and-effects-analysis-fmea)
    - [4.1 Process](#41-process)
    - [4.2 FMEA table template](#42-fmea-table-template)
  - [5. Availability Engineering](#5-availability-engineering)
    - [5.1 Composite availability](#51-composite-availability)
    - [5.2 Availability targets in context](#52-availability-targets-in-context)
  - [6. SLO-Driven Reliability and Error-Budget Policy](#6-slo-driven-reliability-and-error-budget-policy)
  - [7. Redundancy and Fault Isolation](#7-redundancy-and-fault-isolation)
    - [7.1 Failure domains (smallest → largest)](#71-failure-domains-smallest--largest)
    - [7.2 Techniques](#72-techniques)
    - [7.3 Hidden single points of failure (check them)](#73-hidden-single-points-of-failure-check-them)
  - [8. Resilience Patterns in Practice](#8-resilience-patterns-in-practice)
  - [9. Overload and Cascading Failure Protection](#9-overload-and-cascading-failure-protection)
    - [9.1 Cascading failure anatomy](#91-cascading-failure-anatomy)
    - [9.2 Defences](#92-defences)
  - [10. Safe Change — The Largest Source of Outages](#10-safe-change--the-largest-source-of-outages)
  - [11. Data Durability and Backups](#11-data-durability-and-backups)
    - [11.1 Backup principles](#111-backup-principles)
    - [11.2 Backups vs replication](#112-backups-vs-replication)
    - [11.3 Data protection layers](#113-data-protection-layers)
  - [12. Business Continuity and Business Impact Analysis](#12-business-continuity-and-business-impact-analysis)
    - [12.1 ISO 22301](#121-iso-22301)
    - [12.2 Business Impact Analysis (BIA)](#122-business-impact-analysis-bia)
    - [12.3 BIA table template](#123-bia-table-template)
    - [12.4 Beyond IT](#124-beyond-it)
  - [13. RPO/RTO Tiers](#13-rporto-tiers)
  - [14. Disaster Recovery Strategies](#14-disaster-recovery-strategies)
    - [Key design questions](#key-design-questions)
  - [15. DR Plans and Runbooks](#15-dr-plans-and-runbooks)
    - [15.1 DR plan contents](#151-dr-plan-contents)
    - [15.2 Runbook step format](#152-runbook-step-format)
  - [16. Failover and Failback](#16-failover-and-failback)
    - [16.1 Failover decision](#161-failover-decision)
    - [16.2 Traffic switching](#162-traffic-switching)
    - [16.3 Avoiding split-brain](#163-avoiding-split-brain)
    - [16.4 Failback](#164-failback)
  - [17. DR Testing](#17-dr-testing)
    - [DR test report template](#dr-test-report-template)
  - [18. Chaos Engineering and Game Days](#18-chaos-engineering-and-game-days)
    - [18.1 Principles of Chaos Engineering](#181-principles-of-chaos-engineering)
    - [18.2 Experiment catalogue (progress from staging to production carefully)](#182-experiment-catalogue-progress-from-staging-to-production-carefully)
    - [18.3 Tools](#183-tools)
    - [18.4 Game days](#184-game-days)
  - [19. Dependency and Third-Party Reliability](#19-dependency-and-third-party-reliability)
  - [20. Ransomware and Cyber Resilience](#20-ransomware-and-cyber-resilience)
  - [21. Regulatory Expectations](#21-regulatory-expectations)
  - [22. Production Readiness and Reliability Reviews](#22-production-readiness-and-reliability-reviews)
    - [22.1 Production Readiness Review (PRR) — before launch](#221-production-readiness-review-prr--before-launch)
    - [22.2 Periodic reliability review (quarterly per critical service)](#222-periodic-reliability-review-quarterly-per-critical-service)
  - [23. Reliability Playbook by Component](#23-reliability-playbook-by-component)
  - [24. Scenario Playbooks](#24-scenario-playbooks)
    - [24.1 Single-server startup (VPS) — minimum viable resilience](#241-single-server-startup-vps--minimum-viable-resilience)
    - [24.2 SaaS on one cloud region (multi-AZ)](#242-saas-on-one-cloud-region-multi-az)
    - [24.3 Regulated fintech (India)](#243-regulated-fintech-india)
    - [24.4 Responding to an AZ outage (live)](#244-responding-to-an-az-outage-live)
    - [24.5 Accidental data deletion / bad migration](#245-accidental-data-deletion--bad-migration)
  - [25. Checklists](#25-checklists)
    - [Design](#design)
    - [Operate](#operate)
    - [Data \& DR](#data--dr)
    - [Learning](#learning)
  - [26. References](#26-references)
    - [SRE and reliability](#sre-and-reliability)
    - [Standards and frameworks](#standards-and-frameworks)
    - [Chaos engineering](#chaos-engineering)
    - [Books](#books)

---

## 1. Reliability Principles

Site Reliability Engineering (SRE), as defined by Google, treats operations as a software problem. Core tenets adapted for any organisation:

| # | Principle |
|---:|---|
| 1 | **Reliability is a feature** — the most important one; users can't use features of a service that is down. |
| 2 | **100% is the wrong target** — it is impossibly expensive and users can't tell the difference above a certain level; choose an SLO. |
| 3 | **Error budgets balance velocity and reliability** — spend budget on change; when exhausted, prioritise reliability. |
| 4 | **Eliminate toil** — automate repetitive operational work; cap toil (Google targets < 50% of SRE time). |
| 5 | **Monitor symptoms users feel** — SLO-based alerting (chapter 18). |
| 6 | **Expect failure; design for graceful degradation.** |
| 7 | **Change is the main risk** — progressive rollouts, fast rollback (§10). |
| 8 | **Learn without blame** — postmortems focus on systems, not people (chapter 21). |
| 9 | **Recovery capability is proven by testing**, not by documentation. |

---

## 2. Vocabulary

| Term | Definition |
|---|---|
| **Reliability** | Probability the system performs its required function under stated conditions for a period |
| **Availability** | Fraction of time (or requests) the service is usable |
| **Durability** | Probability that stored data is not lost |
| **Resilience** | Ability to absorb disruption, keep operating (possibly degraded), and recover |
| **Fault** | A defect or abnormal condition in a component |
| **Failure** | The system not delivering its expected service |
| **MTBF / MTTF** | Mean time between/to failures |
| **MTTD / MTTA / MTTR** | Mean time to detect / acknowledge / restore (recover) |
| **RPO** | Recovery Point Objective — maximum acceptable data loss measured in time |
| **RTO** | Recovery Time Objective — maximum acceptable time to restore service |
| **MTPD / MTD** | Maximum tolerable period of disruption — beyond it, business viability is threatened (ISO 22301 terminology area) |
| **Business continuity (BC)** | Keeping the business operating during and after disruption |
| **Disaster recovery (DR)** | Restoring IT systems and data after a major disruption |
| **Blast radius** | Scope of impact when something fails |

```text
Availability ≈ MTBF / (MTBF + MTTR)
→ You can improve availability by failing less often (MTBF) OR recovering faster (MTTR).
   Recovery speed is usually the cheaper lever: detection, automation, rollback, runbooks.
```

---

## 3. How Systems Fail

| Category | Examples | Typical defences |
|---|---|---|
| **Changes** (deploys, config, migrations, flags) | Bad release, wrong config value, breaking schema change | Progressive delivery, automated rollback, config validation, expand/contract |
| **Capacity / overload** | Traffic spike, retry storm, thundering herd | Autoscaling, rate limiting, load shedding, backoff + jitter |
| **Dependencies** | Database, cache, DNS, IdP, PSP, cloud API outage | Timeouts, circuit breakers, fallbacks, redundancy, caching |
| **Infrastructure** | Instance/disk/AZ/region failure, network partition | Redundancy, multi-AZ, failover, static stability |
| **Data** | Corruption, accidental deletion, bad migration | Backups, PITR, soft delete, constraints, immutability |
| **Software defects** | Memory leaks, race conditions, time bombs (certificate expiry, leap days, Y2038, integer overflow) | Testing, soak tests, expiry monitoring, defensive coding |
| **Security events** | Ransomware, credential compromise, DDoS | Defense in depth, immutable backups, DDoS protection |
| **Human/operational error** | Wrong command in production, expired domain/certificate | Automation, guardrails, peer review, least privilege |
| **Gray failures** | Partial degradation not caught by health checks (slow disk, packet loss) | Differential observability, client-side SLIs, outlier detection |
| **Metastable failures** | System stays degraded even after the trigger is removed (retry amplification, cache cold start) | Load shedding, retry budgets, controlled warm-up, capacity headroom |

> Industry postmortems repeatedly show that a large share of outages are triggered by **changes**. Invest in change safety first.

---

## 4. Failure Mode and Effects Analysis (FMEA)

Systematically ask, per component: *how can it fail, what happens, how would we know, and what do we do?*

### 4.1 Process

```text
1. List components and dependencies from the architecture diagram (chapter 04 §8)
2. For each: failure modes (down, slow, wrong data, partial, overloaded, misconfigured)
3. Effects on users and data; detection method; existing mitigations
4. Score Severity (S), Occurrence (O), Detectability (D) on 1–10 → Risk Priority Number RPN = S × O × D
5. Act on the highest RPNs; re-score after mitigation
```

### 4.2 FMEA table template

| Component | Failure mode | Effect | S | O | D | RPN | Detection | Mitigation | Owner |
|---|---|---|---:|---:|---:|---:|---|---|---|
| PSP | Timeout > 5 s | Checkout stalls, duplicate charges on retry | 9 | 5 | 3 | 135 | Dependency latency alert | Timeouts, idempotency keys, pending state, circuit breaker | Payments |
| Redis cache | Node failure | Latency spike, DB overload | 6 | 4 | 2 | 48 | Cache hit-ratio alert | Replica + failover, DB headroom, request coalescing | Platform |
| Primary DB | AZ failure | Writes unavailable | 10 | 2 | 2 | 40 | Health checks | Multi-AZ automatic failover | DBA |
| TLS certificate | Expiry | Full outage for clients | 10 | 3 | 4 | 120 | Expiry alerts at 30/14/7 days | Automated renewal (ACME/managed certs) | Platform |
| DNS provider | Outage | Unreachable | 10 | 1 | 5 | 50 | External synthetics | Secondary DNS provider for critical domains | Platform |

---

## 5. Availability Engineering

### 5.1 Composite availability

```text
Serial (all required):      A_total = A1 × A2 × … × An
   3 services at 99.9% → 99.7%;  10 services at 99.9% → ~99.0%
Parallel redundant (any one suffices): A_total = 1 − (1 − A1)(1 − A2)
   two independent 99% replicas → 99.99% (assuming independent failures — often NOT true in practice)
```

Implications:

- Every **hard** synchronous dependency lowers your ceiling — make dependencies soft (cache, fallback, async) where possible.
- Redundancy only helps if failures are **independent** (different AZs, versions deployed progressively, separate failure domains).

### 5.2 Availability targets in context

| Target | Downtime / 30 days | Implies |
|---|---|---|
| 99% | ~7.2 h | Single instance + good backups may suffice |
| 99.9% | ~43 min | Multi-AZ, automated deploys with rollback, on-call |
| 99.95% | ~22 min | + progressive delivery, mature observability, dependency hardening |
| 99.99% | ~4.3 min | + multi-region or cell architecture, automated failover, very fast detection, strict change safety |
| 99.999% | ~26 s | Specialised engineering; very high cost; few business cases justify it |

> Set targets per user journey from business impact and user expectations — not one number for everything. Your availability can't exceed that of your critical dependencies without redundancy across them.

---

## 6. SLO-Driven Reliability and Error-Budget Policy

SLO definitions and burn-rate alerting: chapter 18 §10–11. Reliability management uses them to make decisions:

```text
Error budget remaining      Engineering behaviour
> 50%                        Ship features normally; experiments allowed
25–50%                       Extra scrutiny on risky changes; prioritise top reliability issues
< 25%                        Reliability work prioritised in planning; risky launches need approval
Exhausted                    Feature freeze for the service (except reliability/security fixes) until budget recovers
                             or leadership explicitly accepts the risk; postmortem-driven fixes first
```

| Practice | Detail |
|---|---|
| Error-budget policy agreed in advance | Signed off by product and engineering leadership — avoids arguments mid-incident |
| Budget reviews | Weekly/biweekly in team rituals; monthly at leadership level |
| Toil budget | Track hours of manual ops work; automate the biggest sources |
| Reliability backlog | Postmortem actions, FMEA mitigations, dependency hardening — with owners and due dates |

---

## 7. Redundancy and Fault Isolation

### 7.1 Failure domains (smallest → largest)

```text
process → instance/pod → node/host → rack → availability zone → region → cloud provider → organisation-wide (identity, DNS, CI/CD)
Design so that the failure of one domain doesn't take down the others.
```

### 7.2 Techniques

| Technique | Description |
|---|---|
| **N+1 / N+2 redundancy** | Enough spare capacity to lose 1 (or 2) units at peak |
| **Multi-AZ by default** | Spread instances across ≥ 2–3 AZs with capacity to lose one AZ |
| **Bulkheads** | Separate thread pools, connection pools, queues, or deployments per dependency/customer tier |
| **Cell-based architecture** | Partition customers into independent full-stack cells; a bad deploy or overload affects one cell |
| **Shuffle sharding** | Assign each customer to a random combination of resources, so one noisy/poisoned customer affects few others |
| **Static stability** | System keeps working with existing capacity/config when the control plane (autoscaler, config service, DNS updates) is impaired — pre-provision headroom |
| **Constant work** | Systems that do the same amount of work regardless of load/state changes are less prone to surprises (e.g. push full config periodically rather than deltas) |
| **Graceful degradation** | Non-critical features fail independently (recommendations off, checkout still works) |
| **Independent control and data planes** | Data plane (serving traffic) continues if the control plane (provisioning) fails |

### 7.3 Hidden single points of failure (check them)

```text
DNS provider · TLS certificates · identity provider/SSO · secrets manager · CI/CD (needed to deploy fixes!)
· container registry · a single NAT gateway · a single region for the IdP · shared "common" database
· a single on-call person · a vendor API without fallback · billing/payment provider · a single domain registrar account
```

---

## 8. Resilience Patterns in Practice

Pattern definitions: chapter 04 §14; implementation in services: chapter 13 §12. Operational configuration guidance:

| Pattern | Production configuration guidance |
|---|---|
| **Timeouts** | Every call; based on downstream p99.9 + margin; total within caller's budget; propagate deadlines |
| **Retries** | Max 2–3 attempts; exponential backoff with full jitter; only for transient errors and idempotent operations; retry at ONE layer (not every layer — retries multiply: 3 layers × 3 retries = 27× load) |
| **Retry budgets** | Cap retries to a fraction of normal traffic (e.g. ≤ 10–20%) |
| **Circuit breakers** | Per dependency; trip on error rate/slow-call rate over a window; half-open probes; emit metrics |
| **Fallbacks** | Cached/stale data, default values, queued processing, degraded UI — and test them regularly |
| **Hedged requests** | Only for idempotent reads with tail-latency problems; cap extra load |
| **Health checks** | Liveness checks only the process; readiness checks the ability to serve; avoid cascading restarts when a shared dependency fails (chapter 13 §16) |
| **Idempotency** | Required wherever retries or at-least-once delivery exist |

---

## 9. Overload and Cascading Failure Protection

### 9.1 Cascading failure anatomy

```text
Dependency slows → callers' threads/connections fill up → callers slow → their callers retry
→ load multiplies → healthy components overload → system-wide outage that persists after the trigger (metastable)
```

### 9.2 Defences

```text
🔴 Timeouts + bounded concurrency (pools, semaphores) at every hop
🔴 Retry budgets and jittered backoff; no retries on overload signals (429/503 with Retry-After honoured)
🔴 Load shedding: reject early (cheaply) when saturated, prioritising critical traffic (chapter 19 §11)
🔴 Rate limits per client/tenant at the edge
🟠 Queue limits with age-based dropping (requests waiting longer than the client timeout are useless — drop them)
🟠 Controlled recovery: ramp traffic back gradually after an outage; warm caches; avoid synchronized client reconnects (jitter)
🟠 Capacity headroom and autoscaling ahead of load (chapter 19 §14–15)
```

---

## 10. Safe Change — The Largest Source of Outages

| Practice | Effect |
|---|---|
| Small, frequent changes (trunk-based) | Smaller blast radius, easier diagnosis |
| Automated testing and quality gates | Catch defects before production (chapter 08) |
| **Progressive delivery** (canary, rings, cells) | Limit impact; detect early (chapter 16 §8) |
| **Automated rollback** on SLO/metric regression | Fast recovery (chapter 16 §20) |
| Feature flags + kill switches | Instant disable without deploy |
| Config treated as code | Reviewed, validated (schema), progressively rolled out — config changes cause many outages |
| Expand/contract migrations | Reversible schema evolution (chapter 10 §10) |
| Deploy freezes around peak events (sparingly) | Reduce risk when impact is highest |
| Change correlation | Deploy/flag/config annotations on dashboards; incident tooling shows recent changes |

```text
Golden rule: every change must be able to be (1) deployed to a small fraction first, (2) observed,
and (3) reversed quickly — or it needs a written risk acceptance.
```

---

## 11. Data Durability and Backups

### 11.1 Backup principles

| Principle | Practice |
|---|---|
| **3-2-1-1-0** | 3 copies, 2 media types, 1 off-site (other region/account), 1 offline/immutable, 0 errors in restore verification |
| Coverage | Databases, object storage (versioning), file systems, configuration, secrets (escrow), IaC state, Kubernetes resources, SaaS data (source control, wikis, ticketing, CRM) |
| Frequency | Derived from RPO (continuous WAL/PITR for databases; snapshots for volumes; daily for SaaS exports) |
| Isolation | Backups in a separate account/subscription with separate credentials; delete protection/object lock (WORM) |
| Encryption | At rest and in transit; keys managed separately (and also backed up/escrowed!) |
| Retention | Per policy and regulation; balance with privacy (erasure obligations) |
| **Restore testing** | Automated and scheduled; measure time vs RTO; verify integrity and application consistency |

### 11.2 Backups vs replication

```text
Replication (sync/async replicas, multi-AZ) protects against INFRASTRUCTURE failure.
It faithfully replicates mistakes: accidental DELETE, bad migration, ransomware encryption, corruption.
Backups + PITR protect against DATA failures. You need both.
```

### 11.3 Data protection layers

```text
Constraints and validation (prevent bad data) → soft-delete/undo windows for user actions →
audit logs (who changed what) → PITR (rewind to before the incident) → immutable backups (ransomware) →
off-site/cross-region copies (regional disaster)
```

Database-specific backup/PITR tooling: chapter 10 §14. Single-server scripts (restic/rclone, pg_dump): **39**.

---

## 12. Business Continuity and Business Impact Analysis

### 12.1 ISO 22301

**ISO 22301:2019** (*Security and resilience — Business continuity management systems — Requirements*), amended by **ISO 22301:2019/Amd 1:2024** (climate action changes), specifies requirements for a business continuity management system (BCMS); it is certifiable. **ISO 22313:2020** gives guidance on its use. Related: ISO/IEC 27031 (ICT readiness for business continuity) and NIST SP 800-34 Rev. 1 (contingency planning for federal information systems).

### 12.2 Business Impact Analysis (BIA)

```text
For each business process/service:
1. Describe the process and its owners
2. Impact over time if disrupted (financial, customer, regulatory, reputational, safety) at 1 h, 4 h, 24 h, 72 h, 1 week
3. Maximum tolerable period of disruption (MTPD)
4. RTO (must be < MTPD) and RPO
5. Dependencies: systems, data, people, facilities, suppliers
6. Minimum resources needed to operate in degraded mode (manual workarounds)
```

### 12.3 BIA table template

| Process | Owner | Impact at 1 h / 4 h / 24 h | MTPD | RTO | RPO | Critical systems | Suppliers | Manual workaround |
|---|---|---|---|---|---|---|---|---|
| Customer checkout | Head of Commerce | High / Severe / Critical | 4 h | 30 min | 1 min | Orders, Payments, Catalogue, IdP | PSP, CDN, cloud | None (online only) |
| Order fulfilment | Ops Lead | Low / Medium / High | 24 h | 4 h | 15 min | Orders, Warehouse integration | Courier API | Manual CSV export to courier |
| Internal reporting | Finance | Low / Low / Medium | 72 h | 24 h | 24 h | Data warehouse | — | Delay reports |

### 12.4 Beyond IT

Business continuity also covers people (succession, remote work), facilities, suppliers (alternate vendors, contractual SLAs), communications (crisis comms plans), and regulators. Engineering owns the IT/DR part but participates in the wider BCMS.

---

## 13. RPO/RTO Tiers

Define a small set of tiers and map every service to one.

| Tier | Examples | RTO | RPO | DR strategy (§14) | Test frequency |
|---|---|---|---|---|---|
| **Tier 0 — Mission critical** | Payments, checkout, authentication | ≤ 15–30 min | ≈ 0–1 min | Warm standby / active-active | Quarterly failover |
| **Tier 1 — Business critical** | Orders, customer support tools | ≤ 1–4 h | ≤ 15 min | Pilot light / warm standby | Semi-annual |
| **Tier 2 — Important** | Reporting, internal admin | ≤ 24 h | ≤ 1–4 h | Backup & restore (automated) | Annual |
| **Tier 3 — Non-critical** | Dev tools, experiments | ≤ 72 h+ | ≤ 24 h | Backup & restore / rebuild from code | Annual / ad hoc |

```text
Rules:
- A service's tier can't be better than the tier of its hard dependencies — check the dependency graph
- RTO includes detection + decision + execution + validation time — not just "time to run the script"
- Tiers are agreed with business owners (cost vs risk), documented, and reviewed annually
```

---

## 14. Disaster Recovery Strategies

| Strategy | Description | RTO | RPO | Cost |
|---|---|---|---|---|
| **Backup & restore** | Backups copied to another region/account; rebuild infra from IaC and restore data | Hours–days | Hours (or minutes with continuous PITR shipping) | $ |
| **Pilot light** | Core data replicated continuously to DR region; minimal always-on infra (e.g. DB replica); app tier provisioned on failover | Tens of minutes–hours | Minutes | $$ |
| **Warm standby** | Scaled-down but fully functional copy running in DR region; scale up on failover | Minutes | Seconds–minutes | $$$ |
| **Multi-site active/active** | Full capacity in ≥ 2 regions serving traffic; data replicated with conflict handling | Near-zero | Near-zero (depends on replication mode) | $$$$ |

(These four strategy names follow the AWS disaster recovery whitepaper and are widely used across providers.)

### Key design questions

```text
- Where is the "source of truth" for each dataset during and after failover? How are conflicts handled?
- Does failover depend on anything in the failed region (IaC state, CI/CD, secrets, DNS control plane, IdP)?
- Are quotas/capacity reserved in the DR region (capacity may be scarce during regional events)?
- Is data residency preserved (e.g. both regions in India for regulated data)?
- Can the team execute it at 3 a.m. under stress with the documented runbook?
```

---

## 15. DR Plans and Runbooks

### 15.1 DR plan contents

```markdown
# Disaster Recovery Plan — <Service/Platform>  (version, owner, last tested date)
1. Scope and tier; RTO/RPO targets
2. Architecture: primary/DR regions, data replication, dependencies (diagram)
3. Disaster declaration: criteria, who can declare (and deputies), communication channels
4. Roles: incident commander, DR lead, comms lead, DB lead, app leads, vendor liaison
5. Pre-requisites: access (break-glass), tooling, quotas, documentation location (available offline!)
6. Failover procedure (step-by-step runbook, §15.2)
7. Validation: health checks, synthetic journeys, data consistency checks, business confirmation
8. Communication: internal updates cadence, status page, customer/regulator notifications
9. Operating in DR: limitations, capacity, monitoring
10. Failback procedure (§16)
11. Contacts: vendors (cloud, PSP, CDN, DNS), escalation matrix
12. Test history and improvement actions
```

### 15.2 Runbook step format

```markdown
### Step 4 — Promote DR database replica
- Who: DB lead
- Pre-check: replication lag on DR replica (dashboard link); last WAL received timestamp: ____
- Action:
    aws rds promote-read-replica --db-instance-identifier orders-dr-hyd   # example command; use your platform's equivalent
- Expected result: instance status "available", writable within ~5–10 min
- Verify: `SELECT pg_is_in_recovery();` returns false
- If it fails: escalate to cloud support (case severity: critical); fallback: restore from latest snapshot (RTO +60 min)
- Record: start/end time, observed data loss window (RPO actual)
```

```text
🔴 Runbooks stored where they are reachable during a disaster (not only in the failed region's wiki); printed/offline copy for Tier 0
🔴 Commands copy-pasteable, parameterised, and tested; automation preferred over manual steps
🟠 Each runbook has an owner and a "last exercised" date
```

---

## 16. Failover and Failback

### 16.1 Failover decision

```text
Automatic failover: within a region/AZ (managed DB multi-AZ, load balancer health checks) — fast, low risk
Manual (human-decided) failover: cross-region — risk of split-brain and data loss requires judgement
Decision inputs: impact and expected duration of the outage (provider status), current RPO exposure,
time to fail over vs time to recover in place, data consistency risk
```

### 16.2 Traffic switching

| Method | Speed | Notes |
|---|---|---|
| DNS failover (health-checked records, low TTL) | Minutes (client caching varies) | Simple; some clients ignore TTLs |
| Global load balancer / anycast | Seconds | Best for active-active/warm standby |
| CDN origin failover | Seconds | For web/static and cacheable APIs |
| Client-side endpoint lists (mobile/SDKs) | App-controlled | Plan ahead in clients |

### 16.3 Avoiding split-brain

```text
- Fence the old primary (revoke writes, stop instances, block network) before promoting the new one
- Use a single authority for "who is primary" (consensus-based or human decision recorded)
- Make writers idempotent and include region/epoch in records when doing active-active
```

### 16.4 Failback

Failback is a planned change, often riskier than failover because it is less practised:

```text
1. Restore primary region health; rebuild/replicate data from DR (now the source of truth) back to primary
2. Verify replication caught up; schedule a maintenance window or use controlled traffic shift
3. Fence DR writers; switch traffic; validate; keep DR as standby again
4. Post-event review and runbook updates
```

---

## 17. DR Testing

| Test type | What it proves | Frequency (by tier) |
|---|---|---|
| **Backup restore test** (automated) | Backups are complete and restorable within time | Weekly–monthly |
| **Tabletop exercise** | People know roles, decisions, and communications | Quarterly–semi-annual |
| **Component failover** (DB, cache, AZ) | Automatic failover works and meets RTO | Quarterly |
| **Partial DR test** (single service in DR region) | Runbooks and dependencies for that service | Semi-annual |
| **Full DR exercise / region evacuation** | End-to-end capability for Tier 0–1 | Annual (or as regulators require) |

### DR test report template

```markdown
# DR Test — Orders (Tier 0) — 2026-09-20
Type: Region failover (Mumbai → Hyderabad), production traffic 10% via weighted DNS
Targets: RTO 30 min · RPO 1 min
Results: RTO achieved 24 min (detection 3, decision 4, execution 14, validation 3) · RPO observed 18 s
Issues found:
1. Secrets for PSP not replicated to DR region (manual copy, +6 min) → action: replicate secrets (owner, due date)
2. Runbook step 7 command outdated → fixed in runbook v14
3. DR region vCPU quota insufficient for full peak → quota increase requested
Verdict: ⚠️ Pass with actions
```

```text
🔴 A DR capability that hasn't been tested in the last cycle is reported as "unverified", not "available"
🟠 Rotate who executes DR tests (avoid single-person knowledge)
🟠 Track RTO/RPO achieved over time
```

---

## 18. Chaos Engineering and Game Days

### 18.1 Principles of Chaos Engineering

```text
1. Define steady state as measurable output (SLIs/business metrics)
2. Hypothesise that steady state continues in both control and experimental groups
3. Introduce variables reflecting real-world events (instance death, latency, dependency failure, region loss)
4. Try to disprove the hypothesis by comparing steady state between groups
5. Minimise blast radius; automate experiments to run continuously
```

### 18.2 Experiment catalogue (progress from staging to production carefully)

| Experiment | Hypothesis |
|---|---|
| Kill random pods/instances | No user-visible errors beyond SLO; autoscaling/rescheduling recovers |
| Add 500 ms latency to PSP calls | Checkout p99 rises but stays within SLO; circuit breaker doesn't trip falsely |
| Make cache unavailable | Latency increases; DB stays below 80% CPU; no errors |
| Drop 5% of packets between services | Retries absorb errors without retry storms |
| Fail one AZ (drain/cordon nodes, block subnet) | Traffic shifts; capacity sufficient; no data loss |
| Expire a certificate in staging | Monitoring alerts at thresholds; renewal automation works |
| Revoke a service's secret | Clear errors; alerting; documented recovery |
| DNS resolution failures | Clients use cached results/retries appropriately |

### 18.3 Tools

Chaos Mesh, LitmusChaos (CNCF), AWS Fault Injection Service, Azure Chaos Studio, Gremlin, Steadybit, Toxiproxy (network faults in tests), service mesh fault injection (Istio/Envoy).

### 18.4 Game days

```text
Plan: scenario, hypothesis, blast radius, abort criteria, roles, communication, observers
Run:  inject failure; on-call responds as in a real incident (using runbooks, not tribal knowledge)
Learn: what surprised us? Detection time? Runbook gaps? → actions with owners
```

---

## 19. Dependency and Third-Party Reliability

| Practice | Detail |
|---|---|
| Dependency inventory | Every external dependency (cloud services, PSPs, SMS/email, maps, IdP, CDN, DNS) with criticality, SLA, contacts, status page |
| Soft vs hard dependencies | Convert hard dependencies to soft (cache, queue, fallback) where possible |
| Vendor SLAs vs your SLOs | Your SLO cannot exceed a hard dependency's availability without redundancy |
| Multi-vendor for critical functions | Secondary SMS/OTP provider, secondary DNS, payment gateway routing across PSPs |
| Monitoring | Synthetic checks and client-side metrics per dependency; subscribe to status pages |
| Contracts | Support levels, incident notification commitments, data export, exit clauses |
| Quotas and rate limits | Known, monitored, increased before peaks (chapter 19 §14) |
| Concentration risk | Avoid all critical vendors being in one region/provider without a plan |

---

## 20. Ransomware and Cyber Resilience

```text
🔴 Immutable/offline backups (object lock, separate account, separate credentials, MFA delete)
🔴 Backup infrastructure isolated from production identity (a compromised admin account must not delete backups)
🔴 Tested restore of entire environments from IaC + backups ("rebuild from scratch" capability)
🔴 Credentials and secrets recoverable (escrow/break-glass) even if the primary identity system is compromised
🟠 Clean-room recovery environment and malware scanning of restored data
🟠 Incident response integration: legal, regulatory notification timelines (chapter 09 §22)
🟠 Prioritised recovery order based on BIA tiers (identity → networking → data → Tier 0 services …)
```

---

## 21. Regulatory Expectations

Regulated sectors increasingly mandate operational resilience — confirm current requirements with compliance teams.

| Regime / body | Relevance (summary) |
|---|---|
| **EU Digital Operational Resilience Act (DORA)** — Regulation (EU) 2022/2554, applying from January 2025 | ICT risk management, incident reporting, resilience testing (including threat-led penetration testing for some entities), third-party ICT risk for EU financial entities (not to be confused with Google's DORA research programme) |
| **RBI** (India) | IT governance, risk, controls and assurance practices; business continuity/DR drills; outsourcing and cyber resilience expectations for regulated entities |
| **SEBI** (India) | Cybersecurity and Cyber Resilience Framework (CSCRF) for regulated entities, including recovery objectives and drills |
| **IRDAI** (India) | Information and cybersecurity guidelines for insurers |
| **CERT-In** (India) | Incident reporting within 6 hours; log retention (chapter 09 §15.3) |
| **ISO 22301 / ISO/IEC 27001** | Certifiable continuity and security management systems often required by enterprise customers |
| **SOC 2 (Availability criterion)** | Commitments on availability, backups, DR testing for SaaS providers |

---

## 22. Production Readiness and Reliability Reviews

### 22.1 Production Readiness Review (PRR) — before launch

```markdown
## PRR — <service>
Architecture & dependencies
- [ ] Architecture diagram, dependency list with criticality; no unmitigated single points of failure
- [ ] FMEA completed; top risks mitigated
Reliability
- [ ] SLOs defined; error-budget policy agreed; burn-rate alerts with runbooks
- [ ] Multi-AZ; capacity for AZ loss; autoscaling tested
- [ ] Timeouts, retries (budgeted), circuit breakers, bulkheads; graceful degradation defined
- [ ] Load, stress, and soak tests passed (chapter 19)
Change safety
- [ ] Progressive delivery + automated rollback; feature flags/kill switches
- [ ] Zero-downtime migrations
Data
- [ ] Backups/PITR configured; restore tested; RPO/RTO tier assigned
- [ ] DR strategy for tier implemented and tested
Operations
- [ ] Dashboards, logs, traces; on-call rotation and escalation; runbooks
- [ ] Security review passed (chapter 09); access controls
- [ ] Cost estimate and budget alerts
Sign-off: service owner, SRE/platform, security, product
```

### 22.2 Periodic reliability review (quarterly per critical service)

SLO attainment and budget trends · incidents and postmortem action status · top alerts and noise · dependency changes · capacity forecast · DR test results · toil hours · upcoming risky changes and peak events.

---

## 23. Reliability Playbook by Component

| Component | Common failure | Prevention | Detection | Recovery |
|---|---|---|---|---|
| **Stateless app tier** | Bad deploy, memory leak, crash loops | Canary + automated rollback, soak tests, resource limits | Error-rate/latency SLO alerts, restart counts | Roll back digest; scale out; disable flag |
| **PostgreSQL primary** | Instance/AZ failure, disk full, runaway query, bad migration | Multi-AZ, disk alerts with prediction, statement timeouts, migration linting | Health checks, replication lag, disk forecast, long-query alerts | Automatic failover; kill query; PITR to new instance for data errors |
| **Read replicas** | Lag, replica down | Monitor lag; read routing with fallback to primary | Lag alerts | Route reads to primary/other replica; rebuild replica |
| **Redis/Valkey cache** | Node loss, eviction storm, memory pressure | Replication + failover, maxmemory policy, TTL jitter | Hit ratio, evictions, latency | Fail over; warm gradually; protect DB with request coalescing and rate limits |
| **Message broker / queue** | Broker loss, consumer lag, poison messages | Replication factor ≥ 3 (Kafka), DLQs, consumer autoscaling | Lag/age-of-oldest alerts, DLQ size | Scale consumers; replay from DLQ after fix; partition reassignment |
| **Kubernetes cluster** | Node pool failure, control-plane issues, failed upgrade, IP exhaustion | Multi-AZ node pools, PDBs, surge upgrades in staging first, IP planning | Node readiness, pending pods, API server errors | Cordon/drain, add node pool, roll back node image; redeploy to standby cluster for Tier 0 |
| **Load balancer / gateway** | Misconfiguration, certificate issues | Config as code with validation; staged rollouts of gateway config | Synthetic checks from outside, 5xx at edge | Revert config; fail over to secondary gateway/region |
| **DNS** | Provider outage, accidental record deletion, domain expiry | Secondary DNS provider for critical zones, records as code, registrar lock + auto-renew | External synthetics, expiry monitoring | Restore records from code; switch NS delegation (pre-planned) |
| **TLS certificates** | Expiry, CA issues | Automated renewal (ACME/managed), multiple CA options | Expiry alerts at 30/14/7 days | Manual issuance runbook; switch CA |
| **Identity provider (SSO)** | IdP outage blocks all logins (customers or staff) | Session lifetimes balanced; break-glass accounts; secondary IdP for staff where feasible | Login success rate SLI | Extend sessions; break-glass access; vendor escalation |
| **Secrets manager** | Unavailable at startup → services can't boot | Cache secrets in memory; retry with backoff; multi-region replication | Secret fetch errors | Restart with cached secrets; regional failover |
| **CI/CD platform** | Outage blocks hotfix deploys | Documented manual break-glass deploy path; artifacts in registry independent of CI | CI status monitoring | Break-glass deploy (audited) |
| **Third-party API (PSP, SMS)** | Outage, latency, rate limiting | Timeouts, circuit breakers, multi-provider routing, queues | Dependency SLIs, vendor status | Route to secondary provider; queue and retry later; inform users |

---

## 24. Scenario Playbooks

### 24.1 Single-server startup (VPS) — minimum viable resilience
```text
- Daily encrypted off-site backups (restic/rclone to object storage in another provider/region) + WAL archiving for PITR if feasible
- Weekly automated restore test to a throwaway VPS; measured restore time documented as your real RTO
- Infrastructure reproducible via scripts/Ansible; provider snapshots before risky upgrades
- External uptime monitoring + alerting to phone; status page
- Plan: when revenue depends on uptime, move DB to a managed multi-AZ service first
Reference: 39-vps-enterprise-setup-guide.md
```

### 24.2 SaaS on one cloud region (multi-AZ)
```text
- Multi-AZ for app, DB, cache, broker; capacity to lose one AZ at peak
- Backups + PITR copied to a second region (same country if residency applies) with object lock
- DR strategy: pilot light for Tier 0–1 (IaC ready, DB cross-region replica), backup & restore for Tier 2–3
- Annual region-evacuation exercise for Tier 0; quarterly AZ failure game day
```

### 24.3 Regulated fintech (India)
```text
- Two Indian regions (e.g. Mumbai + Hyderabad) for data localisation; warm standby for payments
- DR drills at the frequency the regulator expects, with evidence (reports, timings, sign-offs)
- Vendor (PSP, bank, card network) dependency mapping with alternate routing
- CERT-In incident reporting and log retention integrated into incident runbooks
```

### 24.4 Responding to an AZ outage (live)
```text
1. Confirm via provider status + your per-AZ metrics; declare incident (chapter 21)
2. Verify automatic failovers completed (DB, cache); check capacity in remaining AZs
3. Cordon/drain affected nodes; scale up in healthy AZs; pause non-critical batch jobs
4. Shed non-critical features if saturated; communicate on status page
5. After recovery: rebalance across AZs gradually; postmortem on gaps (capacity, failover times)
```

### 24.5 Accidental data deletion / bad migration
```text
1. Stop the bleeding: disable the job/feature (flag), put affected area in read-only/maintenance mode
2. Identify exact time window and affected scope (audit logs, migration logs)
3. PITR restore to a NEW instance just before the event; never overwrite production blindly
4. Extract and reconcile affected rows into production (scripts reviewed by two people), or fail over to the restored instance if appropriate
5. Verify with business owners; communicate impact; postmortem → add guardrails (constraints, soft delete, migration review, least privilege)
```

---

## 25. Checklists

### Design
- [ ] Failure modes analysed (FMEA); hard dependencies minimised
- [ ] Redundancy across failure domains; static stability considered
- [ ] Bulkheads/cells for blast-radius control where scale warrants
- [ ] Overload protection: timeouts, bounded concurrency, retry budgets, load shedding

### Operate
- [ ] SLOs + error-budget policy in use for decisions
- [ ] Progressive delivery with automated rollback; config changes treated as code
- [ ] Certificate, domain, and quota expiry monitored
- [ ] Toil tracked and reduced

### Data & DR
- [ ] 3-2-1-1-0 backups; immutable copies; isolated credentials
- [ ] Automated restore tests with measured times
- [ ] BIA done; every service mapped to an RPO/RTO tier
- [ ] DR plan and runbooks per tier, available offline, owned, exercised
- [ ] DR region quotas, secrets, and dependencies ready
- [ ] Failover and failback tested at required frequency

### Learning
- [ ] Chaos experiments/game days scheduled
- [ ] Postmortem actions tracked to completion (chapter 21)
- [ ] Quarterly reliability review per critical service

---

## 26. References

### SRE and reliability
- Google SRE books (SRE Book, Workbook, Building Secure & Reliable Systems): https://sre.google/books/
- SRE Book — Embracing Risk: https://sre.google/sre-book/embracing-risk/
- SRE Book — Eliminating Toil: https://sre.google/sre-book/eliminating-toil/
- SRE Workbook — Error Budget Policy example: https://sre.google/workbook/error-budget-policy/
- AWS Builders' Library (static stability, shuffle sharding, timeouts/retries/jitter): https://aws.amazon.com/builders-library/
- AWS Well-Architected Reliability Pillar: https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/
- AWS — Disaster Recovery of Workloads on AWS (whitepaper): https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/
- Azure reliability documentation: https://learn.microsoft.com/azure/reliability/
- Google Cloud — Disaster recovery planning guide: https://cloud.google.com/architecture/dr-scenarios-planning-guide

### Standards and frameworks
- ISO 22301:2019 (+ Amd 1:2024): https://www.iso.org/standard/75106.html
- ISO 22313:2020 guidance: https://www.iso.org
- NIST SP 800-34 Rev. 1 Contingency Planning Guide: https://csrc.nist.gov/pubs/sp/800/34/r1/upd1/final
- EU DORA Regulation (EU) 2022/2554: https://eur-lex.europa.eu/eli/reg/2022/2554/oj
- RBI notifications & master directions: https://www.rbi.org.in/
- SEBI circulars (CSCRF): https://www.sebi.gov.in/
- CERT-In: https://www.cert-in.org.in/

### Chaos engineering
- Principles of Chaos Engineering: https://principlesofchaos.org/
- Chaos Mesh: https://chaos-mesh.org/ · LitmusChaos: https://litmuschaos.io/
- AWS Fault Injection Service: https://aws.amazon.com/fis/ · Azure Chaos Studio: https://learn.microsoft.com/azure/chaos-studio/

### Books
- *Site Reliability Engineering* & *The Site Reliability Workbook* — Google
- *Release It!* (2nd ed.) — Michael Nygard
- *Chaos Engineering* — Casey Rosenthal & Nora Jones
- *Building Secure and Reliable Systems* — Google

---

**Previous:** [19 — Performance & Scalability](./19-performance-and-scalability.md) · **Next:** [21 — Production Operations](./21-production-operations.md)