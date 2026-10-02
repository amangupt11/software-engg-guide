# 🧱 Engineering Foundations — Production Engineering Guide

> The baseline knowledge every engineer on an enterprise / production team is expected to share: what software engineering is, the standards it rests on, computer-science and systems fundamentals, design principles, reliability and security fundamentals, how engineering performance is measured, and professional ethics.
>
> This file is the **entry point** of the handbook. Every later chapter assumes the vocabulary defined here.

---

## 📚 Table of Contents

1. [How to Use This Handbook](#1-how-to-use-this-handbook)
2. [What Software Engineering Is](#2-what-software-engineering-is)
3. [SWEBOK v4 — The Body of Knowledge](#3-swebok-v4--the-body-of-knowledge)
4. [The Standards Landscape](#4-the-standards-landscape)
5. [Software Quality — ISO/IEC 25010:2023](#5-software-quality--isoiec-250102023)
6. [The Engineering Mindset](#6-the-engineering-mindset)
7. [Computer Science Fundamentals](#7-computer-science-fundamentals)
8. [Computer Systems Fundamentals](#8-computer-systems-fundamentals)
9. [Concurrency Fundamentals](#9-concurrency-fundamentals)
10. [Networking Fundamentals](#10-networking-fundamentals)
11. [Data Representation Fundamentals](#11-data-representation-fundamentals)
12. [Design Principles](#12-design-principles)
13. [Programming Paradigms](#13-programming-paradigms)
14. [Distributed Systems Fundamentals](#14-distributed-systems-fundamentals)
15. [Reliability Fundamentals](#15-reliability-fundamentals)
16. [Security Fundamentals](#16-security-fundamentals)
17. [Measuring Engineering — DORA](#17-measuring-engineering--dora)
18. [Technical Debt](#18-technical-debt)
19. [AI-Assisted Engineering Fundamentals](#19-ai-assisted-engineering-fundamentals)
20. [Professional Ethics](#20-professional-ethics)
21. [Engineer Competency Matrix](#21-engineer-competency-matrix)
22. [Glossary](#22-glossary)
23. [Foundations Checklist](#23-foundations-checklist)
24. [References](#24-references)

---

## 1. How to Use This Handbook

The handbook is organised as a path from **knowledge → process → build → run → collaborate → tools**.

```text
FOUNDATIONS        01 Engineering Foundations
                   02 SDLC (+ STLC)
                   03 Requirements & Planning
DESIGN             04 Architecture
                   05 Software Design
                   06 Coding Standards
BUILD & VERIFY     07 Version Control
                   08 Testing & Quality (+ STLC)
                   09 Security Engineering
                   10 Database Engineering
                   11 API & Integration
PLATFORMS          12 Frontend   13 Backend   14 Mobile   15 Desktop
DELIVER & RUN      16 DevOps & CI/CD
                   17 Cloud & Infrastructure
                   18 Observability & Monitoring
                   19 Performance & Scalability
                   20 Reliability & Disaster Recovery
                   21 Production Operations
PEOPLE             22 Documentation & Knowledge Management
                   23 Team Collaboration
                   24 Engineering Checklists
TOOLING            25–29 Terminals, Shell, CLI
                   30–36 IDEs and Extensions
                   37 Package & Dependency Management
                   38 Workstation Tool Stack
                   39 VPS Enterprise Setup
                   40 Load Testing (Autocannon)
                   41 Windows 11 / macOS Fresh Setup
                   42 Git, GitHub, GitLab, Bitbucket
```

### Reading paths by role

| Role | Read first | Then |
|---|---|---|
| New engineer | 01 → 02 → 06 → 07 → 08 | 41 (setup), 42 (Git), your platform chapter (12–15) |
| Backend engineer | 01, 04, 05, 10, 11, 13 | 16–21 |
| Frontend engineer | 01, 05, 06, 12 | 08, 19 |
| Mobile engineer | 01, 05, 14 | 35 (Android Studio), 36 (Xcode) |
| DevOps / SRE | 01, 16–21 | 27, 28, 39 |
| Tech lead / architect | 01–05, 09, 20 | 22–24 |
| QA / SDET | 01, 02, 08 | 03 (requirements), 40 (load testing) |
| Engineering manager | 01, 02, 03, 17 (DORA in §17 here), 23 | 22, 24 |

### Conventions used in every chapter

| Marker | Meaning |
|:---:|---|
| 🔴 **REQUIRED** | Enterprise baseline — do it unless you have a written exception |
| 🟠 **RECOMMENDED** | Strongly advised for most production systems |
| 🟡 **OPTIONAL** | Adopt when the context calls for it |
| ⚪ **REFERENCE** | Background knowledge |

Normative words follow the IETF convention ([RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) / [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174)): **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, **MAY**.

---

## 2. What Software Engineering Is

Software engineering is the **systematic, disciplined, and quantifiable** application of engineering to the development, operation, and maintenance of software (the classic IEEE definition, carried into ISO/IEC/IEEE 24765, the systems and software engineering vocabulary).

The practical difference between *programming* and *engineering*:

| Programming | Software Engineering |
|---|---|
| Make it work | Make it work, keep it working, let others change it safely |
| One author | Many authors over many years |
| Code | Code + requirements + design + tests + operations + documentation |
| "Done" when it runs | "Done" when it is deployed, observable, supported, and secure |
| Personal judgement | Shared standards, reviews, measurements |
| Optimises for now | Optimises for total cost of ownership over the system lifetime |

> **Software engineering is programming integrated over time.** — popularised by Google's *Software Engineering at Google* (Winters, Manshreck, Wright). Time, scale, and trade-offs are the three forces that turn code into engineering.

### The three forces

```text
TIME     → Will this still be correct, secure, and changeable in 5 years?
SCALE    → What happens with 10× users, 10× data, 10× engineers?
TRADE-OFFS → What are we giving up, and is that decision written down?
```

---

## 3. SWEBOK v4 — The Body of Knowledge

The **Guide to the Software Engineering Body of Knowledge (SWEBOK Guide)** is published by the IEEE Computer Society. **Version 4.0** was released in October 2024 (editor: Hironori Washizaki). It contains **18 knowledge areas (KAs)** and added three new ones compared with v3 (2014): **Software Architecture, Software Security, and Software Engineering Operations**. Existing KAs were updated with Agile and DevOps practice, and AI/ML and IoT considerations appear across KAs. The SWEBOK Guide is also recognised as ISO/IEC TR 19759.

Official: https://www.computer.org/education/bodies-of-knowledge/software-engineering

### SWEBOK v4 knowledge areas → this handbook

| # | SWEBOK v4 Knowledge Area | Covered in |
|---:|---|---|
| 1 | Software Requirements | 03 |
| 2 | Software Architecture *(new in v4)* | 04 |
| 3 | Software Design | 05 |
| 4 | Software Construction | 06, 12–15 |
| 5 | Software Testing | 08 |
| 6 | Software Engineering Operations *(new in v4)* | 16–21 |
| 7 | Software Maintenance | 02 §15, 21 |
| 8 | Software Configuration Management | 07, 37, 42 |
| 9 | Software Engineering Management | 03, 23 |
| 10 | Software Engineering Process | 02 |
| 11 | Software Engineering Models and Methods | 02, 05 |
| 12 | Software Quality | 01 §5, 08 |
| 13 | Software Security *(new in v4)* | 09 |
| 14 | Software Engineering Professional Practice | 01 §20, 23 |
| 15 | Software Engineering Economics | 01 §18, 03 |
| 16 | Computing Foundations | 01 §7–11 |
| 17 | Mathematical Foundations | 01 §7 |
| 18 | Engineering Foundations | 01 §6 |

---

## 4. The Standards Landscape

You do not need to buy every standard. You **do** need to know which standard owns which question, so that internal processes use shared vocabulary and can be audited.

### 4.1 Core standards map

| Question | Standard / Framework | Current edition (verify before audit) | Owner |
|---|---|---|---|
| What processes make up a software life cycle? | ISO/IEC/IEEE **12207** | **2026** (replaced 2017 edition, published April 2026) | ISO/IEC JTC 1/SC 7 + IEEE |
| What processes make up a *system* life cycle? | ISO/IEC/IEEE **15288** | 2023 | ISO/IEC JTC 1/SC 7 + IEEE |
| How do we engineer requirements? | ISO/IEC/IEEE **29148** | 2018 | ISO/IEC JTC 1/SC 7 + IEEE |
| How do we describe architecture? | ISO/IEC/IEEE **42010** | 2022 | ISO/IEC JTC 1/SC 7 + IEEE |
| What does "quality" mean for a product? | ISO/IEC **25010** (SQuaRE) | **2023** | ISO/IEC JTC 1/SC 7 |
| How do we test? | ISO/IEC/IEEE **29119** series | Parts 1–5 (multiple editions) | ISO/IEC JTC 1/SC 7 + IEEE |
| How do we maintain software? | ISO/IEC/IEEE **14764** | 2022 | ISO/IEC JTC 1/SC 7 + IEEE |
| What are the life-cycle documents? | ISO/IEC/IEEE **15289** | 2019 | ISO/IEC JTC 1/SC 7 + IEEE |
| Vocabulary | ISO/IEC/IEEE **24765** | 2017 (+ online SEVOCAB) | ISO/IEC JTC 1/SC 7 + IEEE |
| Information security management | ISO/IEC **27001** | 2022 | ISO/IEC JTC 1/SC 27 |
| Secure development practices | NIST **SP 800-218 (SSDF)** | v1.1 final (2022); **v1.2 (Rev. 1)** draft released Dec 2025 — check CSRC for final status | NIST |
| Cybersecurity program | NIST **CSF** | 2.0 (2024) | NIST |
| Web app security risks | **OWASP Top 10** | Check owasp.org for current edition | OWASP |
| App security verification | **OWASP ASVS** | 5.0 | OWASP |
| Supply-chain integrity | **SLSA** | v1.x | OpenSSF |
| IT service management | **ITIL 4** / ISO/IEC **20000-1** | ITIL 4; 20000-1:2018 | PeopleCert / ISO |
| Testing knowledge / certification | **ISTQB CTFL** | v4.0 | ISTQB |
| Body of knowledge | **SWEBOK** | v4.0 (2024) | IEEE Computer Society |

> ⚠️ ISO/IEC/IEEE standards are paid documents. This handbook summarises their structure and intent; it does not reproduce their text. For audit or contractual compliance, buy and read the actual standard.

### 4.2 How the standards fit together

```text
                ISO/IEC/IEEE 24765  (vocabulary)
                          │
        ┌─────────────────┼─────────────────┐
   15288 (system)   12207 (software)   14764 (maintenance)
        │                 │
   29148 (requirements)  42010 (architecture)
        │                 │
   25010 (quality model) ─┴── 29119 (testing)
        │
   27001 / SSDF / OWASP  (security overlays on every process)
        │
   ITIL 4 / 20000-1      (operating the service)
```

### 4.3 When standards matter most

| Context | Standards you will be asked about |
|---|---|
| Government / defence contracts | 12207, 15288, SSDF, NIST CSF, SBOM requirements |
| Banking / fintech | ISO 27001, PCI DSS, SOC 2, regional regulators |
| Healthcare | HIPAA (US), IEC 62304 (medical device software), ISO 27001 |
| Automotive | ISO 26262 (functional safety), ISO/SAE 21434 (cybersecurity), Automotive SPICE |
| SaaS B2B | SOC 2, ISO 27001, GDPR, India DPDP Act 2023 |
| Selling software into the EU | EU **Cyber Resilience Act** obligations phasing in |

---

## 5. Software Quality — ISO/IEC 25010:2023

ISO/IEC 25010:2023 revised the **product quality model** of the 2011 edition. It defines **nine** product quality characteristics. Compared with 2011: **Usability** was replaced by **Interaction capability**, **Portability** by **Flexibility**, and **Safety** was added as a new characteristic.

| # | Characteristic | Engineering question | Typical evidence |
|---:|---|---|---|
| 1 | **Functional suitability** | Does it do the right things, correctly and completely? | Acceptance tests, requirement coverage |
| 2 | **Performance efficiency** | Is it fast enough with acceptable resource use? | Load tests (see 40), p95/p99 latency, cost per request |
| 3 | **Compatibility** | Does it co-exist and interoperate with other systems? | Contract tests, API compatibility checks |
| 4 | **Interaction capability** *(was Usability)* | Can the intended users achieve goals effectively, including accessibility? | Usability tests, WCAG audit |
| 5 | **Reliability** | Does it keep working, recover, and tolerate faults? | SLO attainment, chaos tests, DR drills |
| 6 | **Security** | Does it protect data and resist attack? | Threat model, SAST/DAST, pentest, ASVS level |
| 7 | **Maintainability** | Can engineers change it safely and cheaply? | Modularity, test coverage, change lead time |
| 8 | **Flexibility** *(was Portability)* | Can it adapt to new environments, scale, or install targets? | Container images, IaC, multi-region tests |
| 9 | **Safety** *(new)* | Does it avoid harm to people, property, or the environment? | Hazard analysis, fail-safe design |

### Using the model in practice

1. During **requirements** (chapter 03), write at least one measurable non-functional requirement per relevant characteristic.
2. During **architecture** (chapter 04), rank the characteristics — you cannot maximise all nine; trade-offs must be explicit.
3. During **testing** (chapter 08), map test types to characteristics so nothing is untested by accident.

```text
Quality attribute priority (example: payments API)
1. Security        — must
2. Reliability     — must (99.95% monthly availability SLO)
3. Functional      — must
4. Performance     — should (p99 < 300 ms)
5. Maintainability — should
6. Compatibility   — should (versioned API, no breaking changes without deprecation)
7. Flexibility     — could
8. Interaction     — could (developer-facing only)
9. Safety          — n/a (document why)
```

---

## 6. The Engineering Mindset

### 6.1 Principles that separate senior from junior work

| Principle | In practice |
|---|---|
| **Own the outcome, not the task** | Ship → verify in production → watch metrics → close the loop |
| **Make it work, make it right, make it fast** (Kent Beck) | Correctness first; optimise with measurements |
| **Measure, don't guess** | Profilers, traces, load tests before "optimising" |
| **Write it down** | Decisions → ADRs; incidents → postmortems; knowledge → docs (chapter 22) |
| **Reversible vs. irreversible decisions** | Move fast on two-way doors; slow down and review one-way doors (schema, public APIs, data deletion) |
| **Small batches** | Small PRs, small deploys, small experiments → lower risk, faster feedback |
| **Automate the second time** | Manual once is learning; manual three times is a bug |
| **Blameless by default** | Systems fail; ask "how did the system allow this?" not "who?" |
| **Security and accessibility are features** | Not phases at the end |
| **Delete code** | The cheapest code to maintain is code that does not exist |

### 6.2 Problem-solving loop

```text
1. Understand   → Restate the problem; identify users, constraints, success metric
2. Explore      → At least two options; spike or prototype if uncertain
3. Decide       → Choose; record trade-offs (ADR)
4. Build        → Small increments, tests alongside code
5. Verify       → Tests, review, staging, production metrics
6. Learn        → Retrospective; update docs, checklists, automation
```

### 6.3 Estimating under uncertainty

- Give **ranges**, not points: "3–5 days, 80% confidence".
- Name the **biggest unknown** and how you will retire it.
- Re-estimate when you learn something; estimates are forecasts, not promises.
- See chapter 03 §15 for estimation techniques.

---

## 7. Computer Science Fundamentals

### 7.1 Asymptotic complexity

| Notation | Name | Example | 1 000 items | 1 000 000 items |
|---|---|---|---|---|
| O(1) | Constant | Hash map lookup (average) | 1 | 1 |
| O(log n) | Logarithmic | Binary search, B-tree lookup | ~10 | ~20 |
| O(n) | Linear | Scan a list | 1 000 | 1 000 000 |
| O(n log n) | Linearithmic | Efficient comparison sort | ~10 000 | ~20 000 000 |
| O(n²) | Quadratic | Nested loops over the same data | 1 000 000 | 10¹² ❌ |
| O(2ⁿ) | Exponential | Brute-force subsets | ❌ | ❌ |

> Rule of thumb: anything O(n²) over user-controlled input is a latent outage. Look for nested loops over collections, N+1 queries, and repeated linear searches.

### 7.2 Data structure selection matrix

| Need | Use | Access | Insert | Notes |
|---|---|---|---|---|
| Index by position | Array / dynamic array | O(1) | O(1) amortised at end | Cache-friendly |
| Lookup by key | Hash map | O(1) avg | O(1) avg | Unordered; worst case degrades |
| Ordered keys, range queries | Balanced tree / B-tree | O(log n) | O(log n) | Databases use B+ trees |
| FIFO processing | Queue / deque | O(1) | O(1) | Work queues, BFS |
| LIFO / undo | Stack | O(1) | O(1) | Parsers, DFS |
| Always get min/max | Heap / priority queue | O(1) peek | O(log n) | Schedulers, top-k |
| Membership test, big sets | Hash set / Bloom filter | O(1) | O(1) | Bloom: false positives possible, no false negatives |
| Prefix search | Trie | O(k) | O(k) | Autocomplete |
| Relationships | Graph (adjacency list) | — | O(1) | Dependencies, social, routing |
| Frequent append, rare middle insert | Linked list | O(n) | O(1) at node | Rarely the right choice in practice |

### 7.3 Algorithms every engineer should recognise

| Family | Algorithms | Production use |
|---|---|---|
| Searching | Binary search | Sorted data, `git bisect` |
| Sorting | Merge sort, quicksort, Timsort | Stable vs unstable sort matters for UI ordering |
| Graph | BFS, DFS, Dijkstra, topological sort | Build systems, dependency resolution, routing |
| Hashing | Consistent hashing | Cache/shard distribution |
| Caching | LRU, LFU, TTL | CDN, in-process caches |
| Rate limiting | Token bucket, leaky bucket, sliding window | API gateways |
| Streaming | Reservoir sampling, HyperLogLog, Count-Min sketch | Analytics at scale |
| Consensus | Raft, Paxos | etcd, Consul, distributed databases |
| Diffing | Myers diff | Git, code review |
| Compression | DEFLATE/gzip, Brotli, Zstandard | HTTP, storage |

### 7.4 Mathematical foundations that show up at work

| Topic | Where you meet it |
|---|---|
| Logic & sets | Query filters, permissions, feature-flag rules |
| Probability | A/B tests, SLO error budgets, Bloom filters, retries |
| Statistics | Percentiles (p50/p95/p99), confidence intervals, load-test analysis |
| Discrete math | Graphs, state machines, combinatorics of test cases |
| Linear algebra | ML, embeddings, graphics |
| Number representation | Floating point, overflow, money handling (§11) |

> **Averages lie.** Always report latency as percentiles. A p50 of 40 ms with a p99 of 4 s is a broken experience for 1 in 100 requests.

---

## 8. Computer Systems Fundamentals

### 8.1 Approximate latency orders of magnitude

Numbers vary by hardware generation; the **ratios** are what matter. (Popularised by Jeff Dean / Peter Norvig, values approximate.)

| Operation | Approx. time | Relative |
|---|---|---|
| L1 cache reference | ~1 ns | 1× |
| Main memory reference | ~100 ns | 100× |
| Read 1 MB sequentially from memory | ~tens of µs | |
| NVMe SSD random read | ~tens–hundreds of µs | |
| Round trip within a datacenter | ~0.5 ms | 500 000× |
| Read 1 MB sequentially from SSD | ~hundreds of µs – 1 ms | |
| HDD seek | ~10 ms | |
| Round trip across a continent | ~tens of ms | |
| Round trip intercontinental (e.g. India ↔ US) | ~150–250 ms | |

**Consequences**

- One network round trip costs as much as millions of CPU operations → batch, cache, and avoid chatty APIs.
- Users far from your region feel geography → CDN, edge, regional deployments (chapter 17).
- Memory is fast, disk is slower, network is slowest → design data access accordingly (chapter 10).

### 8.2 Memory hierarchy

```text
Registers → L1 → L2 → L3 → RAM → SSD/NVMe → Network storage → Object storage
  faster, smaller, costlier  ───────────────►  slower, bigger, cheaper
```

### 8.3 Operating system concepts

| Concept | Why it matters in production |
|---|---|
| Process vs thread | Isolation vs shared memory; crash blast radius |
| Virtual memory, paging | OOM kills, swap thrashing (see 39 for swap tuning) |
| File descriptors | "Too many open files" under load (see 39 `ulimit`) |
| Signals (SIGTERM, SIGKILL) | Graceful shutdown in containers |
| Scheduling, CPU limits | Container throttling, noisy neighbours |
| Permissions, users, groups | Least privilege, non-root containers |
| System calls | Cost of I/O; why async I/O exists |

Hands-on references in this repo: **27-linux-terminal.md**, **28-shell-scripting.md**, **25-windows-powershell.md**.

### 8.4 Containers and virtualisation (concepts)

| Model | Isolation | Startup | Typical use |
|---|---|---|---|
| Bare metal | Physical | Minutes | Databases, HPC |
| Virtual machine | Hypervisor | Tens of seconds | Multi-tenant cloud, legacy |
| Container | Kernel namespaces + cgroups | Sub-second–seconds | Microservices, CI |
| MicroVM (e.g. Firecracker) | Lightweight hypervisor | ~sub-second | Serverless |
| WebAssembly runtime | Sandbox | Milliseconds | Edge functions, plugins |

---

## 9. Concurrency Fundamentals

### 9.1 Concurrency vs parallelism

- **Concurrency** — structuring a program to deal with many things at once (interleaving).
- **Parallelism** — doing many things at the same instant (multiple cores).

### 9.2 Models

| Model | Languages / runtimes | Strength | Watch out for |
|---|---|---|---|
| Threads + locks | Java, C#, C++ | Familiar, powerful | Deadlocks, races |
| Event loop (async I/O) | Node.js, Python asyncio | High I/O concurrency | Blocking the loop with CPU work |
| async/await | JS/TS, C#, Python, Rust, Kotlin, Swift | Readable async code | Forgotten awaits, unbounded fan-out |
| Green threads / virtual threads | Go goroutines, Java virtual threads | Cheap concurrency | Shared-state races still possible |
| Actors / message passing | Erlang/Elixir, Akka | Fault isolation | Mailbox overflow |
| Structured concurrency | Kotlin coroutines, Swift, Java (preview APIs) | Cancellation and lifetimes are explicit | Learning curve |

### 9.3 Classic hazards

| Hazard | Symptom | Prevention |
|---|---|---|
| Race condition | Intermittent wrong results | Immutability, atomic ops, locks, DB constraints |
| Deadlock | Everything hangs | Lock ordering, timeouts |
| Livelock / starvation | Busy but no progress | Fair scheduling, backoff |
| Lost update | Last write wins silently | Optimistic locking (version column), compare-and-swap |
| Thundering herd | Spike after cache expiry or restart | Jittered TTLs, request coalescing |
| Unbounded queue | Memory grows until OOM | Bounded queues, backpressure, load shedding |

---

## 10. Networking Fundamentals

### 10.1 Layers (practical TCP/IP view)

| Layer | Protocols | You debug it with |
|---|---|---|
| Application | HTTP, gRPC, WebSocket, DNS, SMTP | `curl`, browser devtools, `dig` |
| Security | TLS 1.3 (RFC 8446) | `openssl s_client` |
| Transport | TCP, UDP, QUIC | `ss`, `netstat`, `tcpdump` |
| Network | IPv4, IPv6, ICMP | `ping`, `traceroute`, `ip route` |
| Link | Ethernet, Wi-Fi | NIC stats |

### 10.2 HTTP essentials

| Item | Reference |
|---|---|
| HTTP semantics (methods, status codes, headers) | [RFC 9110](https://www.rfc-editor.org/rfc/rfc9110) |
| HTTP/1.1 | [RFC 9112](https://www.rfc-editor.org/rfc/rfc9112) |
| HTTP/2 | [RFC 9113](https://www.rfc-editor.org/rfc/rfc9113) |
| HTTP/3 over QUIC | [RFC 9114](https://www.rfc-editor.org/rfc/rfc9114), QUIC [RFC 9000](https://www.rfc-editor.org/rfc/rfc9000) |
| Problem details for HTTP APIs | [RFC 9457](https://www.rfc-editor.org/rfc/rfc9457) |

Method safety and idempotency (RFC 9110):

| Method | Safe | Idempotent | Typical use |
|---|:---:|:---:|---|
| GET | ✅ | ✅ | Read |
| HEAD | ✅ | ✅ | Metadata |
| OPTIONS | ✅ | ✅ | CORS preflight, capabilities |
| PUT | ❌ | ✅ | Replace resource |
| DELETE | ❌ | ✅ | Remove resource |
| POST | ❌ | ❌ | Create / command (make idempotent with an `Idempotency-Key`) |
| PATCH | ❌ | ❌ | Partial update |

Status code classes: **1xx** informational, **2xx** success, **3xx** redirection, **4xx** client error, **5xx** server error. Retry only on transient failures (e.g. 429 with `Retry-After`, 502, 503, 504, connection resets) and only for idempotent operations.

### 10.3 DNS, TLS, and ports

```bash
dig example.com A +short
dig example.com AAAA +short
dig example.com MX +short
curl -sv https://example.com -o /dev/null
openssl s_client -connect example.com:443 -servername example.com </dev/null
ss -tulpn
```

| Port | Service |
|---:|---|
| 22 | SSH |
| 53 | DNS |
| 80 / 443 | HTTP / HTTPS |
| 5432 | PostgreSQL |
| 3306 | MySQL / MariaDB |
| 6379 | Redis / Valkey |
| 27017 | MongoDB |
| 9090 | Prometheus |
| 3000 | Grafana / many dev servers |

Server hardening and firewall setup: **39-vps-enterprise-setup-guide.md**.

---

## 11. Data Representation Fundamentals

### 11.1 Text

- Use **UTF-8 everywhere** — source files, databases, APIs, logs.
- Normalise before comparing user-entered text (Unicode NFC).
- Length ≠ bytes ≠ user-perceived characters (grapheme clusters, emoji).
- Database collation decides sorting and case-insensitive comparison; choose deliberately.

### 11.2 Time

| Rule | Why |
|---|---|
| Store timestamps in **UTC** | No DST ambiguity |
| Serialize as **RFC 3339 / ISO 8601** (`2026-10-02T09:30:00Z`) | Interoperable, sortable |
| Convert to local time only at the edge (UI) | Users see their zone |
| Store the **IANA time zone** (`Asia/Kolkata`) for future local events | Offsets change; zones carry rules |
| Never use local server time for business logic | Servers move regions |
| Use monotonic clocks for durations | Wall clocks can jump |

### 11.3 Numbers and money

```text
❌ 0.1 + 0.2 = 0.30000000000000004   (binary floating point)
✅ Store money as integer minor units (paise/cents) or DECIMAL/NUMERIC
✅ Store the ISO 4217 currency code with every amount (INR, USD)
✅ Define rounding mode explicitly (e.g. banker's rounding vs half-up) per regulation
```

Watch for integer overflow (32-bit IDs run out at ~2.1 billion), and JavaScript's safe integer limit (2⁵³ − 1) when sending 64-bit IDs to browsers — send them as strings.

### 11.4 Identifiers

| ID type | Pros | Cons | Use when |
|---|---|---|---|
| Auto-increment integer | Small, ordered | Leaks volume, hard to merge across shards | Internal tables |
| UUIDv4 (random) | Globally unique, no coordination | Poor index locality | Public IDs where order doesn't matter |
| **UUIDv7** (time-ordered, [RFC 9562](https://www.rfc-editor.org/rfc/rfc9562)) | Unique + roughly sortable, good index locality | Embeds creation time | 🟠 Recommended default for new distributed systems |
| ULID / Snowflake-style | Sortable | Non-standard / needs coordination | Existing ecosystems |

### 11.5 Serialization formats

| Format | Human-readable | Schema | Typical use |
|---|:---:|:---:|---|
| JSON | ✅ | Optional (JSON Schema) | REST APIs, config |
| YAML | ✅ | Optional | Config, Kubernetes, CI |
| TOML | ✅ | Optional | App/tool config |
| Protocol Buffers | ❌ | ✅ | gRPC, internal services |
| Avro | ❌ | ✅ | Kafka, data pipelines |
| Parquet | ❌ | ✅ | Analytics, columnar storage |
| CSV | ✅ | ❌ | Data exchange (define escaping/encoding!) |

---

## 12. Design Principles

These principles apply at every scale — function, class, module, service, organisation. Chapter 05 applies them in depth.

### 12.1 SOLID

| Principle | Meaning | Smell when violated |
|---|---|---|
| **S**ingle Responsibility | A module has one reason to change | "Manager"/"Util" god classes |
| **O**pen/Closed | Extend behaviour without modifying stable code | `switch` on type everywhere |
| **L**iskov Substitution | Subtypes honour the base contract | Overrides that throw "not supported" |
| **I**nterface Segregation | Small, focused interfaces | Clients implementing unused methods |
| **D**ependency Inversion | Depend on abstractions; inject dependencies | `new` of infrastructure inside domain logic |

### 12.2 Other core principles

| Principle | One-line rule |
|---|---|
| **KISS** | Prefer the simplest design that meets the requirements |
| **YAGNI** | Do not build for speculative futures |
| **DRY** | One authoritative representation of each piece of *knowledge* (not every similar line) |
| **Separation of concerns** | UI, domain, and infrastructure change for different reasons; keep them apart |
| **High cohesion, low coupling** | Things that change together live together |
| **Composition over inheritance** | Assemble behaviour; avoid deep hierarchies |
| **Law of Demeter** | Talk to direct collaborators, not their internals |
| **Principle of least astonishment** | Behaviour matches what users and readers expect |
| **Fail fast** | Validate early; crash loudly on programmer errors |
| **Explicit over implicit** | Configuration, dependencies, and side effects should be visible |
| **Idempotency** | Repeating an operation has the same effect as doing it once |
| **Immutability by default** | Fewer races, easier reasoning |
| **Secure by default** | Safe settings out of the box; opt-in to risk |
| **Design for failure** | Every network call can fail, hang, or return garbage |

### 12.3 Cohesion and coupling at service level

```text
Good:  OrderService owns orders table, publishes OrderPlaced event
Bad:   OrderService, BillingService, ShippingService all write the same orders table
```

---

## 13. Programming Paradigms

| Paradigm | Core idea | Strength | Languages |
|---|---|---|---|
| Imperative / procedural | Step-by-step state changes | Direct, efficient | C, Go, scripts |
| Object-oriented | Objects encapsulate state + behaviour | Modelling domains, polymorphism | Java, C#, Kotlin, Swift, PHP |
| Functional | Pure functions, immutability, composition | Testability, concurrency safety | Haskell, F#, Elixir; FP features in most modern languages |
| Declarative | Describe *what*, not *how* | Concise, optimisable | SQL, HTML, Terraform, Kubernetes YAML |
| Reactive / event-driven | Streams of events and reactions | UI, real-time, async systems | RxJS, Kotlin Flow, Combine |
| Logic | Facts + rules + queries | Rules engines, policy | Prolog, Datalog, OPA Rego |

> Production advice: mix paradigms deliberately — **functional core, imperative shell** (pure domain logic, side effects at the edges) is a robust default.

---

## 14. Distributed Systems Fundamentals

### 14.1 The fallacies of distributed computing (Deutsch et al.)

1. The network is reliable.
2. Latency is zero.
3. Bandwidth is infinite.
4. The network is secure.
5. Topology doesn't change.
6. There is one administrator.
7. Transport cost is zero.
8. The network is homogeneous.

Every one of these assumptions eventually causes an incident.

### 14.2 CAP and PACELC

- **CAP**: during a network **P**artition, a system must choose between **C**onsistency and **A**vailability.
- **PACELC**: if **P**artitioned choose **A** or **C**; **E**lse (normal operation) choose **L**atency or **C**onsistency.

| System style | Partition behaviour | Normal behaviour | Example use |
|---|---|---|---|
| PC/EC | Consistent | Consistent | Ledgers, inventory reservation |
| PA/EL | Available | Low latency | Social feeds, carts, caches |

### 14.3 Consistency vocabulary

| Term | Meaning |
|---|---|
| Strong / linearizable | Reads see the latest committed write |
| Read-your-writes | A client always sees its own writes |
| Monotonic reads | Never see older data after newer data |
| Eventual consistency | Replicas converge if writes stop |
| ACID | Atomic, Consistent, Isolated, Durable transactions |
| BASE | Basically Available, Soft state, Eventually consistent |

### 14.4 Delivery semantics

| Guarantee | Reality |
|---|---|
| At-most-once | May lose messages |
| At-least-once | May duplicate → consumers MUST be idempotent |
| Exactly-once | Achieved end-to-end only via idempotency + deduplication or transactional systems |

Patterns: **transactional outbox**, **idempotency keys**, **saga** (chapter 05 / 11).

---

## 15. Reliability Fundamentals

### 15.1 Timeouts, retries, backoff, jitter

```text
🔴 Every outbound call MUST have a timeout.
🔴 Retries MUST use exponential backoff with jitter and a max attempt count.
🔴 Only retry idempotent operations (or make them idempotent).
🟠 Use a retry budget / circuit breaker to avoid retry storms.
🟠 Propagate deadlines across service hops.
```

Example: full-jitter backoff

```python
import random, time

def call_with_retry(fn, attempts=5, base=0.1, cap=5.0):
    for attempt in range(attempts):
        try:
            return fn()
        except TransientError:
            if attempt == attempts - 1:
                raise
            sleep = random.uniform(0, min(cap, base * 2 ** attempt))
            time.sleep(sleep)
```

### 15.2 Availability arithmetic

| Availability | Downtime / 30-day month | Downtime / year |
|---|---|---|
| 99% | ~7.2 h | ~3.65 days |
| 99.9% | ~43.2 min | ~8.76 h |
| 99.95% | ~21.6 min | ~4.38 h |
| 99.99% | ~4.3 min | ~52.6 min |
| 99.999% | ~26 s | ~5.3 min |

Serial dependencies **multiply**: three services at 99.9% each in series ≈ 99.7% overall. Redundancy and graceful degradation are how you climb back up.

### 15.3 SLI / SLO / SLA / error budget

| Term | Definition | Owner |
|---|---|---|
| **SLI** | A measured indicator (e.g. % of requests < 300 ms and non-5xx) | Engineering |
| **SLO** | Target for an SLI over a window (e.g. 99.9% over 28 days) | Engineering + product |
| **SLA** | Contractual promise with penalties | Business / legal |
| **Error budget** | 100% − SLO; the allowed unreliability to spend on change | Shared |

Deep dive: chapters **18** (observability) and **20** (reliability). Source: Google SRE books — https://sre.google/books/

---

## 16. Security Fundamentals

### 16.1 Core properties

| Property | Meaning |
|---|---|
| **Confidentiality** | Only authorised parties can read |
| **Integrity** | Data is not altered without detection |
| **Availability** | Authorised users can use the system when needed |
| Authenticity | Identity of parties and origin of data are verified |
| Non-repudiation | Actions can be proven to have happened |
| Privacy | Personal data is processed lawfully and minimally |

### 16.2 Principles

| Principle | Application |
|---|---|
| **Least privilege** | Minimum permissions for users, services, CI tokens |
| **Defense in depth** | Multiple independent controls (WAF + validation + parameterised queries + DB permissions) |
| **Zero trust** ([NIST SP 800-207](https://csrc.nist.gov/pubs/sp/800/207/final)) | No implicit trust from network location; verify every request |
| **Secure defaults** | Deny by default, HTTPS only, MFA on |
| **Minimise attack surface** | Remove unused endpoints, ports, packages |
| **Don't roll your own crypto** | Use vetted libraries and current algorithms |
| **Secrets never in code** | Vault / cloud secret manager; scanning in CI |
| **Validate input, encode output** | Prevents injection and XSS classes |
| **Audit everything sensitive** | Who did what, when, from where |
| **Assume breach** | Segmentation, detection, and recovery plans |

### 16.3 Secure development as a framework

NIST **SSDF (SP 800-218)** groups secure development practices into four groups:

| Group | Focus |
|---|---|
| **PO** — Prepare the Organization | Roles, training, policies, toolchains |
| **PS** — Protect the Software | Protect code, builds, and releases from tampering |
| **PW** — Produce Well-Secured Software | Secure design, review, testing, secure defaults |
| **RV** — Respond to Vulnerabilities | Identify, triage, fix, and learn from vulnerabilities |

SSDF v1.1 (2022) is the final version; **v1.2 (SP 800-218 Rev. 1)** was released as an initial public draft on 17 Dec 2025 with comments closing 30 Jan 2026. A community profile for generative AI, **SP 800-218A**, was finalised in July 2024. Full treatment: chapter **09**.

---

## 17. Measuring Engineering — DORA

DORA (DevOps Research and Assessment, now part of Google Cloud) runs the longest-running research programme on software delivery performance. DORA now defines **five** software delivery performance metrics, grouped into **throughput** and **instability**.

| Factor | Metric | Definition | How to measure |
|---|---|---|---|
| Throughput | **Change lead time** | Time from commit to running in production | VCS + deploy events |
| Throughput | **Deployment frequency** | How often changes are deployed to production | Deploy events |
| Throughput | **Failed deployment recovery time** | Time to recover from a deployment that fails and needs immediate intervention | Failed deploy → restoring deploy |
| Instability | **Change fail rate** | Share of deployments needing immediate intervention (rollback, hotfix) | Deploy outcomes |
| Instability | **Deployment rework rate** | Share of deployments that are unplanned and happen because of a production incident | Deploys tagged as incident response |

History worth knowing:

- "Mean time to recover" was redefined as **failed deployment recovery time** so that external outages (e.g. a datacenter failure) are not counted as delivery failures.
- **Deployment rework rate** was added as the fifth metric, and recovery time moved into the throughput group.
- In 2025 the annual report was renamed **"State of AI-assisted Software Development"** and replaced the old four performance tiers with **seven team archetypes**.

### Key 2025 DORA findings (summarised)

- About **90%** of respondents use AI at work, up 14 points from 2024; the median is roughly two hours a day.
- AI acts as an **amplifier**: it strengthens what a team already does well and exposes its existing weaknesses.
- AI adoption is now linked to **higher throughput**, but it still has a **negative relationship with delivery stability**.
- Trust is mixed: only about a quarter of respondents trust AI output "a lot" or "a great deal".
- The report introduced the **DORA AI Capabilities Model**, which lists seven capabilities that determine whether AI helps or hurts a team.

### Measurement rules

```text
🔴 Measure teams and systems, never individuals.
🔴 Look at all five metrics together — speed without stability is just shipping incidents faster.
🟠 Break lead time into stages (coding → review → CI → approval → deploy) to find the bottleneck.
🟠 Pair metrics with developer-experience surveys (friction, burnout).
🟡 Compare a team with its own past, not with other teams.
```

Official: https://dora.dev/guides/dora-metrics/ · Report: https://dora.dev/research/

---

## 18. Technical Debt

Technical debt is the future cost of choosing an easier solution now. Some debt is a rational loan; unmanaged debt is a slow outage.

### 18.1 Technical debt quadrant (Martin Fowler)

| | Reckless | Prudent |
|---|---|---|
| **Deliberate** | "We don't have time for design." | "Ship now, refactor next sprint — ticket filed." |
| **Inadvertent** | "What's layering?" | "Now we know how we should have built it." |

### 18.2 Managing debt

```text
1. Make it visible     → label tickets `tech-debt`, link to the code location
2. Quantify interest   → incidents, slowed lead time, onboarding pain, cost
3. Budget capacity     → e.g. a fixed share of each sprint for debt + maintenance
4. Pay at the boundary → boy-scout rule when you touch code
5. Prevent new debt    → Definition of Done, linters, review checklists
6. Re-assess quarterly → retire, schedule, or accept with a written rationale
```

### 18.3 Debt register template

```markdown
| ID | Area | Description | Interest (impact) | Principal (effort) | Risk if ignored | Owner | Decision | Review date |
|----|------|-------------|-------------------|--------------------|-----------------|-------|----------|-------------|
| TD-014 | billing | Hand-rolled retry logic without jitter | 2 incidents/quarter | 3 days | Retry storms | @team-billing | Fix Q4 | 2026-12-15 |
```

---

## 19. AI-Assisted Engineering Fundamentals

AI coding assistants and agents are now part of the default toolchain. Treat them like a fast, tireless, sometimes wrong colleague.

| Rule | Status |
|---|:---:|
| Every AI-generated change goes through the same review, tests, and CI gates as human code | 🔴 |
| The author who submits AI-generated code owns it fully (correctness, security, licence) | 🔴 |
| Never paste secrets, customer data, or restricted code into tools not approved by your organisation | 🔴 |
| Keep changes small — large AI-generated diffs overwhelm review (code review becomes the bottleneck) | 🔴 |
| Verify dependencies suggested by AI exist and are legitimate (typosquatting / hallucinated packages) | 🔴 |
| Strengthen automated tests, since AI raises throughput but can lower stability | 🟠 |
| Write project context files (architecture notes, conventions) so tools follow your standards | 🟠 |
| Track deployment rework rate and change fail rate as AI adoption grows | 🟠 |
| Record which tools and models are approved, and for what data classification | 🟠 |

For AI features *inside* your product (LLM calls, RAG, agents), see chapter 03 §20 for requirements and chapter 09 for security (prompt injection, data leakage, output handling).

---

## 20. Professional Ethics

The **ACM/IEEE-CS Software Engineering Code of Ethics and Professional Practice** sets out eight principles. In summary, engineers should:

| # | Principle | In practice |
|---:|---|---|
| 1 | **Public** | Act consistently with the public interest — safety, privacy, accessibility |
| 2 | **Client and employer** | Serve them well, consistent with the public interest |
| 3 | **Product** | Meet the highest professional standards possible |
| 4 | **Judgment** | Maintain integrity and independence of professional judgement |
| 5 | **Management** | Leaders promote an ethical approach to development and maintenance |
| 6 | **Profession** | Advance the integrity and reputation of the profession |
| 7 | **Colleagues** | Be fair to and supportive of colleagues |
| 8 | **Self** | Keep learning and promote an ethical approach |

Also see the **ACM Code of Ethics and Professional Conduct (2018)**: https://www.acm.org/code-of-ethics

### Everyday ethical checkpoints

- Would I be comfortable if this data use were explained to the affected users?
- Does this feature exclude users with disabilities?
- Am I shipping something I know is unsafe because of a deadline? Escalate in writing.
- Am I respecting licences of open-source and AI-generated code?
- Are dark patterns being introduced into the UX?

---

## 21. Engineer Competency Matrix

A generic matrix — adapt titles to your organisation.

| Dimension | Junior (L1–L2) | Mid (L3) | Senior (L4) | Staff+ (L5+) |
|---|---|---|---|---|
| **Scope** | Tasks | Features | Systems / projects | Multiple teams / org |
| **Technical** | Writes working code with guidance | Independently delivers features with tests | Designs systems, sets standards for a team | Shapes architecture and technical strategy |
| **Quality** | Follows checklists | Writes good tests, catches issues in review | Defines quality gates, prevents classes of bugs | Builds platforms that make quality the default |
| **Operations** | Learns on-call | Handles incidents in own area | Leads incidents, writes postmortems | Designs reliability programmes |
| **Communication** | Asks good questions | Clear PRs, design notes | Writes RFCs/ADRs, aligns stakeholders | Influences across org without authority |
| **Mentoring** | — | Helps juniors | Mentors several engineers | Grows senior engineers and leaders |
| **Ambiguity** | Needs defined tasks | Handles defined problems | Turns ambiguous problems into plans | Finds the right problems to solve |

---

## 22. Glossary

| Term | Definition |
|---|---|
| **ADR** | Architecture Decision Record — short document capturing one decision, its context, and consequences |
| **Artifact** | Any output of the life cycle: build, document, test report, image |
| **Backpressure** | Mechanism for a consumer to slow a producer |
| **Baseline** | Approved, version-controlled snapshot of requirements, design, or code |
| **Blast radius** | The scope of impact when something fails |
| **Canary** | Release to a small subset of traffic before full rollout |
| **Change lead time** | Commit → production (DORA) |
| **Circuit breaker** | Stops calling a failing dependency for a period |
| **Definition of Done (DoD)** | Shared quality checklist an increment must meet |
| **Error budget** | Allowed unreliability = 100% − SLO |
| **Feature flag** | Runtime switch to enable/disable behaviour without deploy |
| **Idempotent** | Repeating has the same effect as once |
| **IaC** | Infrastructure as Code |
| **NFR** | Non-functional requirement (quality attribute) |
| **Observability** | Ability to understand internal state from outputs (logs, metrics, traces) |
| **p95 / p99** | 95th / 99th percentile |
| **RPO / RTO** | Recovery Point / Time Objective — max data loss / max downtime |
| **SBOM** | Software Bill of Materials |
| **SLI / SLO / SLA** | Indicator / Objective / Agreement |
| **Toil** | Manual, repetitive, automatable operational work |
| **Traceability** | Ability to link requirement ↔ design ↔ code ↔ test ↔ release |

---

## 23. Foundations Checklist

### Every engineer
- [ ] Can explain the SDLC used by the team and where their work fits (chapter 02)
- [ ] Knows the team's Definition of Done
- [ ] Uses UTF-8, UTC, RFC 3339, and correct money handling
- [ ] Sets timeouts on every network call; retries with backoff + jitter only when safe
- [ ] Reads latency as percentiles, not averages
- [ ] Understands least privilege and never commits secrets
- [ ] Writes small PRs with tests
- [ ] Reviews AI-generated code with the same rigour as human code
- [ ] Knows how to find logs, metrics, and traces for their service

### Every team
- [ ] Ranked quality attributes (ISO/IEC 25010) for each major system
- [ ] Five DORA metrics collected automatically
- [ ] SLOs defined for user-facing services
- [ ] Tech-debt register reviewed quarterly
- [ ] Approved-tools list for AI assistants and data classes
- [ ] Onboarding path through this handbook documented

---

## 24. References

### Bodies of knowledge and standards
- SWEBOK Guide v4.0 — IEEE Computer Society: https://www.computer.org/education/bodies-of-knowledge/software-engineering
- ISO/IEC/IEEE 12207:2026 — Software life cycle processes: https://www.iso.org (search the catalogue for "12207")
- ISO/IEC/IEEE 15288 — System life cycle processes: https://www.iso.org
- ISO/IEC 25010:2023 — Product quality model: https://www.iso.org/standard/78176.html
- ISO/IEC/IEEE 24765 vocabulary (SEVOCAB): https://pascal.computer.org/
- NIST SP 800-218 SSDF: https://csrc.nist.gov/projects/ssdf
- NIST SP 800-218 Rev. 1 (SSDF 1.2) draft: https://csrc.nist.gov/pubs/sp/800/218/r1/ipd
- NIST SP 800-207 Zero Trust Architecture: https://csrc.nist.gov/pubs/sp/800/207/final
- NIST Cybersecurity Framework 2.0: https://www.nist.gov/cyberframework

### Metrics and research
- DORA metrics guide: https://dora.dev/guides/dora-metrics/
- DORA metrics history: https://dora.dev/guides/dora-metrics/history/
- 2025 DORA report announcement: https://cloud.google.com/blog/products/ai-machine-learning/announcing-the-2025-dora-report
- Google SRE books: https://sre.google/books/

### Protocols and data
- RFC 2119 / 8174 (requirement keywords): https://www.rfc-editor.org/rfc/rfc2119
- RFC 9110 HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9114 HTTP/3: https://www.rfc-editor.org/rfc/rfc9114
- RFC 8446 TLS 1.3: https://www.rfc-editor.org/rfc/rfc8446
- RFC 9457 Problem Details: https://www.rfc-editor.org/rfc/rfc9457
- RFC 3339 Date and Time on the Internet: https://www.rfc-editor.org/rfc/rfc3339
- RFC 9562 UUIDs (incl. v7): https://www.rfc-editor.org/rfc/rfc9562

### Ethics and practice
- ACM Code of Ethics: https://www.acm.org/code-of-ethics
- Software Engineering Code of Ethics (ACM/IEEE-CS): https://ethics.acm.org/code-of-ethics/software-engineering-code/
- *Software Engineering at Google* (free online): https://abseil.io/resources/swe-book
- Martin Fowler — Technical Debt Quadrant: https://martinfowler.com/bliki/TechnicalDebtQuadrant.html

---

**Next:** [02 — Software Development Lifecycle](./02-software-development-lifecycle.md)