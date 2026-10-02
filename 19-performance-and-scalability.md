# ⚡ Performance & Scalability — Production Engineering Guide

> How to make systems fast and keep them fast as load grows: performance requirements and budgets, core theory (percentiles, tail latency, queueing, Little's Law, Amdahl's Law, Universal Scalability Law), the performance engineering lifecycle, load-testing methodology (workload models, open vs closed models, coordinated omission), profiling and bottleneck analysis, backend/database/caching optimisation, concurrency and backpressure, horizontal and data scaling, capacity planning and autoscaling, benchmarks and regression detection in CI, runtime tuning (JVM, Node.js, Go, .NET, Python), network/edge performance, anti-patterns, and playbooks for peak events like festival sales.
>
> Related: [01 Latency numbers & complexity](./01-engineering-foundations.md) · [03 NFRs & quality scenarios](./03-requirements-and-planning.md#9-non-functional-requirements) · [04 Scalability patterns §15](./04-software-architecture.md#15-scalability-patterns) · [08 Performance testing §20.1](./08-testing-and-quality.md#201-performance-testing) · [10 Query optimisation](./10-database-engineering.md#8-query-optimisation) · [12 Core Web Vitals](./12-frontend-engineering.md#13-performance-and-core-web-vitals) · [17 Autoscaling](./17-cloud-and-infrastructure.md#9-kubernetes-scaling-and-reliability) · [18 Observability](./18-observability-and-monitoring.md) · [40 Autocannon load testing](./40-autocannon_production_CLI.md)

---

## 📚 Table of Contents

1. [Principles](#1-principles)
2. [Performance Requirements and Budgets](#2-performance-requirements-and-budgets)
3. [Core Theory](#3-core-theory)
4. [The Performance Engineering Lifecycle](#4-the-performance-engineering-lifecycle)
5. [Load Testing Methodology](#5-load-testing-methodology)
6. [Load Testing Tools and Examples](#6-load-testing-tools-and-examples)
7. [Profiling and Bottleneck Analysis](#7-profiling-and-bottleneck-analysis)
8. [Backend Optimisation](#8-backend-optimisation)
9. [Database Performance](#9-database-performance)
10. [Caching Strategy](#10-caching-strategy)
11. [Concurrency, Backpressure, and Load Shedding](#11-concurrency-backpressure-and-load-shedding)
12. [Scaling Applications](#12-scaling-applications)
13. [Scaling Data](#13-scaling-data)
14. [Capacity Planning](#14-capacity-planning)
15. [Autoscaling](#15-autoscaling)
16. [Benchmarks and Performance Regression Detection](#16-benchmarks-and-performance-regression-detection)
17. [Runtime Tuning](#17-runtime-tuning)
18. [Network and Edge Performance](#18-network-and-edge-performance)
19. [Client Performance (Web, Mobile, Desktop)](#19-client-performance-web-mobile-desktop)
20. [Performance Anti-Patterns](#20-performance-anti-patterns)
21. [Scenario Playbooks](#21-scenario-playbooks)
22. [Checklists](#22-checklists)
23. [References](#23-references)

---

## 1. Principles

| # | Principle |
|---:|---|
| 1 | **Measure first.** No optimisation without a profile, a trace, or a benchmark showing the bottleneck. |
| 2 | **Define "fast enough"** in measurable NFRs (percentiles at a stated load) before optimising. |
| 3 | **Optimise the bottleneck, not the code you happen to be reading.** Everything else is noise (Theory of Constraints). |
| 4 | **Percentiles, not averages.** Users experience the tail. |
| 5 | **Test at realistic scale** — data volume, concurrency, network, and traffic mix. |
| 6 | **Design for horizontal scale** — stateless services, partitionable data, async work. |
| 7 | **Protect the system under overload** — timeouts, backpressure, load shedding beat collapse. |
| 8 | **Performance is a feature with a budget** — tracked in CI and production, not a one-off project. |

> "Premature optimization is the root of all evil" (Knuth) — but the full quote continues that we should not pass up the critical 3%. Find that 3% with data.

---

## 2. Performance Requirements and Budgets

### 2.1 Writing performance NFRs

```text
✅ "GET /v1/products?category={c} returns p95 ≤ 300 ms and p99 ≤ 800 ms at 1 200 RPS sustained for 30 minutes,
    with error rate < 0.1%, on production-sized data (2 M products), with CPU ≤ 70% per instance."
❌ "The API should be fast and handle lots of users."
```

Quality-attribute scenario format: chapter 03 §9.2.

### 2.2 Latency budget decomposition

```text
User-perceived checkout submit budget: 1 000 ms (p95)
├── Client + network (mobile 4G, India):      ~250 ms
├── CDN/edge + gateway:                        ~30 ms
├── Orders service own work:                   ~80 ms
├── Pricing service call (p95):                ~60 ms
├── Inventory reservation (DB):                ~40 ms
├── PSP call (p95, external):                 ~400 ms
└── Headroom:                                 ~140 ms
→ Every hop gets a budget AND a timeout below it; total timeouts must fit the caller's budget.
```

### 2.3 Common budgets (starting points — derive yours from users and business)

| Area | Budget |
|---|---|
| Web Core Web Vitals | LCP ≤ 2.5 s, INP ≤ 200 ms, CLS ≤ 0.1 (p75 field data) — chapter 12 §13 |
| Interactive API reads | p95 ≤ 200–300 ms, p99 ≤ 500–800 ms |
| Interactive API writes | p95 ≤ 300–500 ms |
| Background jobs | Throughput and completion deadline (e.g. reconciliation done by 06:00 IST) |
| Mobile cold start | As fast as possible; many teams target < ~1–2 s on mid-range devices |
| Resource efficiency | Cost per 1 000 requests, CPU/memory per RPS |

---

## 3. Core Theory

### 3.1 Latency vs throughput

| Term | Meaning |
|---|---|
| **Latency** | Time to complete one operation (from the client's perspective, including queueing) |
| **Throughput** | Operations completed per unit time |
| **Concurrency** | Operations in progress at the same time |
| **Utilisation** | Fraction of time a resource is busy |
| **Saturation** | Work waiting (queued) for a resource |

### 3.2 Percentiles and tail latency

```text
p50 = median; p95 = 95% of requests faster; p99 = 1 in 100 slower; p99.9 = 1 in 1 000 slower
Fan-out amplifies tails: if a page calls 20 services in parallel, each with 1% chance of a slow response,
P(at least one slow) = 1 − 0.99^20 ≈ 18% of page loads hit a tail.
```

Tail-latency mitigations ("The Tail at Scale", Dean & Barroso): hedged requests (with care), tied requests, reducing fan-out, timeouts with fallbacks, isolating slow work, prioritisation, and capacity headroom.

### 3.3 Little's Law

```text
L = λ × W
L: average number of requests in the system (concurrency)
λ: arrival/throughput rate (requests per second)
W: average time in the system (latency)

Example: 500 RPS × 0.2 s average latency = 100 concurrent requests in flight
→ size thread pools / connection pools / instances accordingly; if latency doubles, concurrency doubles.
```

### 3.4 Queueing — why utilisation above ~70–80% hurts

For simple queueing models (M/M/1), average time in system grows as `1 / (1 − ρ)` where ρ is utilisation:

| Utilisation ρ | Relative wait (×service time) |
|---:|---:|
| 50% | 2× |
| 70% | ~3.3× |
| 80% | 5× |
| 90% | 10× |
| 95% | 20× |

Latency explodes near saturation — keep **headroom** (often 30–50% at peak) and autoscale before resources saturate.

### 3.5 Amdahl's Law and the Universal Scalability Law

```text
Amdahl: speedup(N) = 1 / ((1 − p) + p/N)   where p = parallelisable fraction
If 95% is parallel, max speedup = 20× no matter how many cores/instances.

Universal Scalability Law (Gunther): adds a coherence/crosstalk term — beyond some point, adding nodes
REDUCES throughput (lock contention, cache coherence, coordination).
→ Remove serial bottlenecks (global locks, single DB writer, shared counters) to scale.
```

---

## 4. The Performance Engineering Lifecycle

```text
1. Requirements   → NFRs with numbers; workload expectations; growth forecast
2. Design         → capacity estimates; choose patterns (cache, async, partitioning); latency budgets
3. Build          → efficient algorithms/data access; instrumentation; micro-benchmarks for hot paths
4. Test           → load/stress/soak/spike tests in production-like environments; profile under load
5. Release        → canary with latency/error analysis; compare against baseline
6. Operate        → SLOs, RUM, continuous profiling, capacity dashboards
7. Improve        → regression detection, periodic tuning, capacity reviews before peaks
```

---

## 5. Load Testing Methodology

### 5.1 Test types (summary — chapter 08 §20.1)

| Type | Question |
|---|---|
| Load | Do we meet NFRs at expected peak? |
| Stress | Where and how do we break? Do we degrade gracefully and recover? |
| Spike | Can we absorb sudden bursts (push notification, sale start)? |
| Soak | Leaks or degradation over hours? |
| Scalability | Does throughput grow linearly when we add capacity? |
| Breakpoint / capacity | Max sustainable RPS within SLOs per instance/cluster |

### 5.2 Workload modelling

```text
1. Derive the traffic mix from production analytics/logs: endpoints, ratios, think times, payload sizes
   e.g. 60% browse/search, 25% product detail, 10% cart, 5% checkout
2. Model data: realistic cardinality and distributions (hot products — Zipf-like popularity; many users; large carts)
3. Model arrival patterns: steady, daily curves, spikes (sale at 00:00 IST), batch jobs overlapping
4. Include background load: cron jobs, consumers, cache warmups, admin reports
5. Define success criteria up front (percentiles, error rate, resource ceilings)
```

### 5.3 Open vs closed workload models

| Model | Behaviour | Use |
|---|---|---|
| **Closed** (fixed virtual users, each waits for a response before sending the next) | Load drops when the system slows — can hide problems | Simulating a fixed pool of clients/workers |
| **Open** (fixed arrival rate regardless of responses) | Matches internet traffic: users keep arriving | Public APIs and websites — **preferred for SLO validation** |

### 5.4 Coordinated omission

When a load generator waits for slow responses before sending new requests, it **fails to send** requests that real users would have sent during the stall — making latency look much better than reality (Gil Tene). Mitigate with **constant-arrival-rate executors** (k6 `constant-arrival-rate`, Gatling open injection, wrk2's `-R` rate) and tools that measure from intended send time.

### 5.5 Environment and execution rules

```text
🔴 Production-like environment (same instance types, versions, configuration, data volume) — or carefully planned production tests
🔴 Load generators with enough capacity, located realistically (another region/network for internet-facing tests); monitor the generators too
🔴 Warm-up period before measuring (JIT, caches, connection pools)
🔴 Observe the system under test with the same dashboards/traces as production
🟠 Run each scenario long enough (≥ 10–30 min for load; hours for soak)
🟠 Repeat runs; compare against a stored baseline
🟠 Coordinate tests touching third parties (use sandboxes/stubs; never load-test someone else's production without permission)
🟠 Isolate test data and clean up; flag synthetic traffic
```

### 5.6 Load test report template

```markdown
# Load Test Report — Checkout v2.5.0 — 2026-10-02
## Goal
Validate checkout NFRs at 3× normal peak (festival sale forecast).
## Setup
Environment: perf (prod-like, 6 × c7g.xlarge, RDS r7g.2xlarge), data: 2 M products, 500 k users
Tool: k6 (constant-arrival-rate), generators: 4 × c7i.2xlarge in a separate region
Workload mix: browse 60% · PDP 25% · cart 10% · checkout 5%
## Results
| Scenario | RPS | p50 | p95 | p99 | Errors | CPU (app) | DB CPU |
|---|---|---|---|---|---|---|---|
| Baseline 1× | 400 | 45 ms | 120 ms | 260 ms | 0.00% | 22% | 18% |
| Peak 3× | 1 200 | 60 ms | 210 ms | 640 ms | 0.02% | 61% | 57% |
| Stress | 2 100 (break) | — | — | 4.1 s | 3.4% | 92% | 96% |
## Bottlenecks found
1. Inventory reservation query missing composite index → p99 spikes (fixed, PR #1932)
2. HTTP client pool to PSP stub capped at 50 → queueing at > 1 500 RPS
## Verdict
✅ Meets NFR at 3× with ~35% headroom. Breaking point at ~5× (DB CPU). Recommendation: add read replica for catalogue before sale.
## Artifacts
Scripts commit, raw results, dashboards snapshots, traces of slowest requests.
```

---

## 6. Load Testing Tools and Examples

| Tool | Language/model | Strengths |
|---|---|---|
| **k6** (Grafana) | JavaScript scripts, Go engine | Developer-friendly, thresholds as pass/fail, open-model executors, CI-friendly |
| **Gatling** | Scala/Java/Kotlin/JS DSL | High performance, open/closed injection profiles, rich reports |
| **Apache JMeter** | GUI/XML | Protocol breadth, mature ecosystem |
| **Locust** | Python | Python-defined user behaviour, distributed |
| **Artillery** | YAML/JS | Node.js ecosystem, serverless load generation |
| **autocannon** | Node.js CLI | Fast HTTP benchmarking of single endpoints — see **40-autocannon_production_CLI.md** |
| **wrk2 / Vegeta / oha / hey** | CLI | Quick constant-rate HTTP benchmarks |
| **ghz** | CLI | gRPC load testing |

### 6.1 k6 open-model load test

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  scenarios: {
    browse: {
      executor: 'ramping-arrival-rate',
      startRate: 100, timeUnit: '1s', preAllocatedVUs: 200, maxVUs: 2000,
      stages: [
        { target: 400, duration: '5m' },    // ramp to normal peak
        { target: 1200, duration: '10m' },  // ramp to 3× peak
        { target: 1200, duration: '30m' },  // hold
        { target: 0, duration: '2m' },
      ],
    },
  },
  thresholds: {
    'http_req_failed': ['rate<0.001'],
    'http_req_duration{endpoint:search}': ['p(95)<300', 'p(99)<800'],
  },
};

const BASE = __ENV.BASE_URL;
export default function () {
  const res = http.get(`${BASE}/v1/products?category=shoes&limit=20`, { tags: { endpoint: 'search' } });
  check(res, { 'status is 200': (r) => r.status === 200 });
}
```

### 6.2 Quick single-endpoint benchmark (autocannon)

```bash
# 100 connections, 60 s, latency percentiles — details and options in 40-autocannon_production_CLI.md
npx autocannon -c 100 -d 60 --renderStatusCodes https://staging.api.shopnow.example/v1/products?category=shoes
```

---

## 7. Profiling and Bottleneck Analysis

### 7.1 Method

```text
1. Start from the symptom: which SLI regressed? Which route/tenant/region/version?
2. Traces: where is time spent across services? (critical path, slowest spans, N+1 patterns)
3. USE method per resource: CPU, memory, disk, network, connection pools, thread pools, locks, queues
   → find the saturated resource
4. Profile the hot service under representative load (CPU, allocation, lock/contention, wall-clock)
5. Form a hypothesis, change ONE thing, measure again
```

### 7.2 Flame graphs

Flame graphs (Brendan Gregg) visualise sampled stack traces: width = proportion of samples (time), so wide plateaus are hot code paths. Use CPU flame graphs for compute, off-CPU/wall-clock for waiting (I/O, locks), and allocation flame graphs for GC pressure.

### 7.3 Profilers by runtime

| Runtime | Tools |
|---|---|
| JVM | **JDK Flight Recorder (JFR)** + JDK Mission Control, **async-profiler**, VisualVM |
| Go | `pprof` (CPU, heap, goroutine, mutex, block), `go tool trace` |
| Node.js | `--cpu-prof`, `--heap-prof`, Chrome DevTools inspector, clinic.js, 0x |
| Python | **py-spy** (sampling, no code changes), Scalene, cProfile, memray |
| .NET | `dotnet-trace`, `dotnet-counters`, `dotnet-gcdump`, PerfView, Visual Studio profiler |
| Rust / C / C++ | `perf`, cargo-flamegraph, Valgrind/heaptrack |
| Linux system | `perf`, eBPF tools (bcc/bpftrace), `pidstat`, `iostat`, `vmstat`, `ss` — see **27-linux-terminal.md** |
| Continuous (production) | Pyroscope, Parca, OTel profiles (alpha), cloud profilers — chapter 18 §9 |

```bash
# Go: 30-second CPU profile from a running service exposing net/http/pprof on an internal port
go tool pprof -http=:0 http://orders.internal:6060/debug/pprof/profile?seconds=30

# Python: profile a live process without restarting it
py-spy record -o profile.svg --pid 12345 --duration 30

# JVM: start a JFR recording on a running JVM
jcmd <pid> JFR.start duration=60s filename=/tmp/orders.jfr settings=profile
```

> Never expose profiling endpoints publicly; bind them to internal interfaces and protect them.

---

## 8. Backend Optimisation

Ordered roughly by typical payoff:

| # | Lever | Examples |
|---:|---|---|
| 1 | **Do less work** | Remove unnecessary calls, fields, joins; avoid repeated computation; paginate |
| 2 | **Fix data access** | Indexes, eliminate N+1, batch reads/writes, keyset pagination (chapter 10 §7–8) |
| 3 | **Cache** | Multi-level caching for read-heavy data (§10) |
| 4 | **Go async** | Move slow non-critical work to queues (emails, PDFs, webhooks) |
| 5 | **Parallelise independent I/O** | Concurrent calls to independent dependencies (bounded) |
| 6 | **Reduce round trips** | Batch APIs, BFF aggregation, gRPC streaming, HTTP/2 multiplexing |
| 7 | **Connection reuse** | Keep-alive, shared HTTP clients, DB pools sized correctly |
| 8 | **Efficient serialisation** | Smaller payloads, field selection, compression (gzip/Brotli/zstd), Protobuf for internal hot paths |
| 9 | **Algorithmic complexity** | Replace O(n²) with O(n log n)/O(n); right data structures (chapter 01 §7) |
| 10 | **Memory/GC pressure** | Fewer allocations in hot paths, object reuse/pooling where justified, streaming instead of buffering |
| 11 | **Runtime tuning** | GC and thread settings (§17) — only after the above |

### Batching example (avoid N+1 calls to a downstream service)

```typescript
// ❌ N calls
const products = await Promise.all(orderItems.map(i => catalog.getProduct(i.productId)));

// ✅ One batched call (or a DataLoader that batches within a tick)
const products = await catalog.getProducts(orderItems.map(i => i.productId));
```

---

## 9. Database Performance

Database-specific techniques live in chapter 10. Performance checklist:

```text
□ Top queries by total time reviewed (pg_stat_statements); slow query log enabled
□ Indexes match query patterns (composite order, covering, partial); unused indexes dropped
□ No N+1 queries; batch lookups; avoid SELECT *
□ Keyset pagination for deep lists
□ Connection pooling sized by measurement; no connection storms on deploy/scale-out (PgBouncer)
□ Statistics current; plans stable after deploys/upgrades
□ Hot rows/locks identified (counters, inventory) → redesign (sharded counters, queues, conditional updates)
□ Read replicas / caches for read-heavy paths; writes kept on primary
□ Partitioning for very large time-series tables
□ Autovacuum healthy (no bloat-induced slowdowns)
```

---

## 10. Caching Strategy

### 10.1 Multi-level caching

```text
Browser cache → CDN/edge → API gateway cache → application in-process cache → distributed cache (Redis/Valkey)
→ database buffer cache → disk
Each level closer to the user is faster and cheaper but harder to invalidate.
```

### 10.2 Cache decision guide

| Data | Cache where | TTL / invalidation |
|---|---|---|
| Static assets (JS/CSS/images) | Browser + CDN | Immutable with content hashes; 1 year |
| Public catalogue/content | CDN + distributed cache | Minutes + purge on change |
| Per-user data (cart, profile) | Distributed cache (per-user keys) or none | Short TTL; invalidate on write |
| Configuration/reference data | In-process | Minutes; refresh in background |
| Expensive aggregates/reports | Materialised views / precomputed tables | Scheduled refresh |
| Auth tokens/sessions | Distributed cache | Session lifetime |

### 10.3 Metrics to watch

Hit ratio, latency per level, eviction rate, memory usage, stampede indicators (concurrent misses on the same key), stale-data incidents. Design rules: chapter 05 §17.

---

## 11. Concurrency, Backpressure, and Load Shedding

### 11.1 Sizing pools with Little's Law

```text
Required concurrency ≈ target RPS × latency of the downstream operation
DB pool: 300 RPS of queries × 0.01 s avg query time ≈ 3 concurrent connections needed on average
         → pool of ~10 with headroom per instance is plenty; 100 would just overload the DB
HTTP client to PSP: 50 RPS × 0.8 s ≈ 40 concurrent → pool/limit ~60 with timeout 2 s
```

### 11.2 Backpressure

```text
🔴 Bounded queues everywhere (in-memory queues, executor queues, message consumers' prefetch)
🔴 When full: reject fast (429/503 with Retry-After) or apply flow control upstream — never grow unbounded until OOM
🟠 Propagate deadlines so downstream services stop work the caller has abandoned
🟠 Adaptive concurrency limits (e.g. Netflix concurrency-limits, Envoy adaptive concurrency, Spring @ConcurrencyLimit)
```

### 11.3 Load shedding and graceful degradation

| Technique | Example |
|---|---|
| Priority-based shedding | Keep checkout and payments; shed recommendations and analytics under overload |
| Feature degradation | Serve cached/stale catalogue; disable personalisation; simplified search |
| Admission control | Virtual waiting room / queue page for flash sales |
| Rate limiting | Per-user/tenant/IP to protect shared capacity (chapter 11 §9) |
| Circuit breakers | Stop calling a failing dependency; fail fast with fallback |

---

## 12. Scaling Applications

### 12.1 Vertical vs horizontal

| | Vertical (scale up) | Horizontal (scale out) |
|---|---|---|
| How | Bigger machine | More instances |
| Pros | Simple; no code changes | Elastic; fault tolerant; near-linear for stateless services |
| Cons | Hardware limits; single point of failure; downtime to resize | Requires statelessness, load balancing, distributed state |
| Good for | Databases (initially), legacy apps | Web/API tiers, workers |

### 12.2 Making services horizontally scalable

```text
🔴 Stateless processes: sessions in Redis/DB or tokens; files in object storage; no local caches required for correctness
🔴 Idempotent handlers and consumers (retries happen during scaling events)
🔴 Externalised configuration; fast startup; graceful shutdown (chapter 13 §16)
🟠 Sticky sessions avoided (or only as an optimisation, not a requirement)
🟠 Singleton tasks coordinated (leader election, scheduler service) — chapter 13 §21
🟠 Partition-aware consumers (Kafka partitions = max parallel consumers per group)
```

### 12.3 Scale cube recap (chapter 04 §15)

```text
X-axis: clone stateless instances behind a load balancer
Y-axis: split by function/service (separate hot capabilities so they scale independently)
Z-axis: split by data (shard by tenant/customer/region)
```

---

## 13. Scaling Data

| Stage | Technique | Notes |
|---|---|---|
| 1 | Query and index optimisation | Biggest wins, lowest cost |
| 2 | Vertical scaling of the DB | Quick relief; buy time |
| 3 | Connection pooling (PgBouncer) | Remove connection overhead limits |
| 4 | Caching | Offload repeated reads |
| 5 | Read replicas | Scale reads; handle replication lag |
| 6 | CQRS / read models / search indexes | Purpose-built read paths |
| 7 | Partitioning | Large tables, retention |
| 8 | Functional split (DB per service/module) | Separate workloads |
| 9 | Sharding (Citus, Vitess, application sharding) or distributed SQL | Write scaling; most complex |

### Hot-spot patterns

| Hot spot | Remedy |
|---|---|
| Global counter row (views, stock) | Sharded counters; batch increments; approximate counts |
| Single partition key overloaded (celebrity product, big tenant) | Key salting, dedicated shard/cell, caching, queues |
| Time-ordered inserts on one index page | Usually fine in PostgreSQL B-trees; for distributed stores, avoid monotonic partition keys |
| Inventory during flash sale | Pre-allocated stock buckets, reservation queue, atomic conditional updates |

---

## 14. Capacity Planning

### 14.1 Process

```text
1. Measure current demand: peak RPS per endpoint, data growth, job volumes (from metrics, not guesses)
2. Measure supply: max sustainable RPS per instance/cluster within SLOs (from load tests)
3. Forecast demand: growth trends + business events (marketing campaigns, festival sales, launches)
4. Compute required capacity with headroom (N+1 or N+2 instances; ≥ 30–50% headroom at forecast peak)
5. Identify limits beyond compute: DB connections, IOPS, cache memory, provider quotas, third-party rate limits
6. Plan: autoscaling config, pre-scaling for known events, reservations/commitments for baseline
7. Review quarterly and before every major event
```

### 14.2 Capacity model example

```text
Forecast festival peak: 4 600 RPS (from §15 of chapter 04 back-of-envelope + marketing forecast)
Load test: 1 instance (2 vCPU) sustains 400 RPS at p99 ≤ 500 ms with 65% CPU
Instances needed: 4 600 / 400 = 11.5 → 12; add N+2 for AZ failure tolerance → 14 (≈ 5 per AZ across 3 AZs)
DB: peak 9 000 queries/s; primary handles 12 000 q/s at 70% CPU → OK; add 1 read replica for catalogue spikes
Third parties: PSP contracted limit 300 TPS; forecast 220 TPS → OK, request temporary increase as buffer
Provider quotas: vCPU quota in ap-south-1 for the node group → request increase 2 weeks ahead
```

### 14.3 Don't forget non-compute limits

```text
Cloud quotas (vCPUs, IPs, load balancers), DB max_connections, disk IOPS/throughput, NAT gateway bandwidth,
cache memory, Kafka partitions, email/SMS provider throughput, CDN origin capacity, third-party API rate limits,
and team capacity for on-call during the event.
```

---

## 15. Autoscaling

| Signal | Use | Caveat |
|---|---|---|
| CPU utilisation | CPU-bound services | Poor for I/O-bound services |
| Requests per second per instance | Request-driven services | Needs custom/external metrics |
| Latency / concurrency (in-flight requests) | Latency-sensitive services | Can oscillate; use stabilisation |
| Queue depth / consumer lag (KEDA) | Workers | Scale-to-zero cold starts |
| Schedule (cron scaling) | Predictable peaks (sale start, business hours) | Combine with reactive scaling |

```text
🔴 Minimum replicas ≥ 2–3 for production (one per AZ minimum)
🔴 Scale-up fast, scale-down slow (stabilisation windows) — chapter 17 §9
🔴 Node autoscaling and quotas ready for pod autoscaling (pods pending = no capacity)
🟠 Pre-scale before known events; autoscaling reacts in minutes, spikes happen in seconds
🟠 Load-test autoscaling behaviour itself (time from spike to new capacity serving traffic)
🟠 Watch cold starts: JVM warm-up, cache priming, connection establishment — use readiness gates
```

---

## 16. Benchmarks and Performance Regression Detection

### 16.1 Micro-benchmarks

| Runtime | Tool |
|---|---|
| JVM | **JMH** |
| .NET | **BenchmarkDotNet** |
| Go | `go test -bench` + `benchstat` |
| Rust | Criterion, `cargo bench` |
| Python | pytest-benchmark, pyperf |
| JS/TS | Vitest `bench`, tinybench, mitata |

```go
func BenchmarkPriceCalculation(b *testing.B) {
    cart := fixtures.LargeCart(200)
    b.ReportAllocs()
    for b.Loop() {                     // Go 1.24+; use `for i := 0; i < b.N; i++` on older versions
        _ = pricing.Calculate(cart)
    }
}
// go test -bench=Price -count=10 ./pricing > new.txt && benchstat old.txt new.txt
```

```text
Rules: benchmark realistic inputs; prevent dead-code elimination; run multiple iterations; compare statistically
(benchstat, JMH error bars); pin CPU frequency/noisy-neighbour effects where possible.
```

### 16.2 Regression detection in CI/CD

| Level | How |
|---|---|
| PR | Micro-benchmarks for hot modules with thresholds; bundle-size checks (frontend); query-count assertions in integration tests |
| Nightly | Load tests against a stable perf environment; compare p95/p99/throughput with baseline; alert on > X% regression |
| Release | Canary analysis comparing latency/error distributions (chapter 16 §8, §20) |
| Production | SLO burn alerts, continuous profiling diffs between versions |

Tools for tracking benchmark history: continuous benchmarking services/actions, Bencher, Grafana dashboards of k6 results, Gatling Enterprise, custom result stores.

---

## 17. Runtime Tuning

Tune **after** fixing algorithms and data access, with measurements before and after.

### 17.1 JVM (Java 21/25)

```text
- Container-aware heap: -XX:MaxRAMPercentage=70–75 (leave room for metaspace, threads, direct buffers)
- GC choice: G1 (default, balanced); Generational ZGC for low-latency/large heaps (ZGC is generational by default in recent JDKs)
- Virtual threads for blocking I/O concurrency (Spring Boot: spring.threads.virtual.enabled=true)
- Startup: Class Data Sharing / AOT cache (newer JDKs), Spring AOT, GraalVM native image or CRaC for fast-start needs
- Observe: JFR continuously in production (low overhead), GC logs (-Xlog:gc*)
```

### 17.2 Node.js

```text
- Never block the event loop; monitor event-loop delay (perf_hooks.monitorEventLoopDelay)
- One process per core via multiple container replicas (preferred) or cluster/PM2
- --max-old-space-size aligned with container memory limit when needed
- Use streaming for large payloads; avoid sync APIs (fs.readFileSync) in request paths
- Keep-alive agents for outbound HTTP (undici/fetch pools)
```

### 17.3 Go

```text
- GOMEMLIMIT set near container memory limit (soft limit) to avoid OOM; tune GOGC for throughput vs memory
- GOMAXPROCS respects container CPU limits in recent Go versions (verify for your version; otherwise use automaxprocs)
- Avoid excessive allocations in hot paths (pprof alloc profiles); sync.Pool only with evidence
- Reuse http.Client/Transport; set MaxIdleConnsPerHost for high-throughput outbound calls
```

### 17.4 .NET

```text
- Server GC for ASP.NET Core (default); consider DATAS / container memory limits settings in recent .NET versions
- Native AOT / ReadyToRun for startup-sensitive services where compatible
- Use pooled/streaming APIs (System.IO.Pipelines, ArrayPool) in hot paths; avoid sync-over-async
- dotnet-counters for GC, thread pool starvation, exception rates
```

### 17.5 Python

```text
- Multiple worker processes (Gunicorn/Uvicorn workers) per container sized to CPU; async frameworks for I/O-heavy APIs
- Offload CPU-heavy work to native libraries (NumPy, Polars), separate services, or task queues
- Free-threaded CPython builds (PEP 703, optional builds in recent Python versions) are emerging — evaluate carefully before production use
- Profile with py-spy/Scalene before rewriting in another language
```

---

## 18. Network and Edge Performance

| Lever | Practice |
|---|---|
| Geography | Serve Indian users from Indian regions; CDN/edge for global users |
| Protocols | HTTP/2 everywhere; HTTP/3 (QUIC) at the edge for lossy mobile networks; TLS 1.3 (fewer round trips), session resumption |
| Connection reuse | Keep-alive; connection pooling; avoid per-request TLS handshakes service-to-service |
| Compression | Brotli for text at the edge; zstd/gzip for APIs; don't compress already-compressed media |
| Payload size | Field selection, pagination, image optimisation (chapter 12 §14) |
| DNS | Fast authoritative DNS; reasonable TTLs; `preconnect`/`dns-prefetch` for critical third-party origins |
| Chattiness | Batch APIs, BFF aggregation, GraphQL with care, avoid cross-region calls in request paths |
| Placement | Co-locate chatty services and their databases in the same region/AZ where possible |

---

## 19. Client Performance (Web, Mobile, Desktop)

| Client | Primary metrics | Chapter |
|---|---|---|
| Web | Core Web Vitals (LCP, INP, CLS), JS bundle size, TTFB | 12 §13 |
| Mobile | Cold start, jank/hitches, ANRs, app size, battery, network usage | 14 §10 |
| Desktop | Startup time, memory footprint, UI thread responsiveness | 15 §14 |

Client-perceived performance often dominates: a 50 ms backend improvement matters less than a 2 s LCP fix on mid-range Android devices over 4G.

---

## 20. Performance Anti-Patterns

| Anti-pattern | Symptom | Fix |
|---|---|---|
| Optimising without profiling | Effort spent, no improvement | Measure first |
| Averages in dashboards | "Latency is fine" while users suffer | Percentiles and histograms |
| N+1 queries / calls | Latency grows with data size | Batch, join, DataLoader |
| Unbounded queries/results | Memory spikes, timeouts | Pagination, limits, streaming |
| Synchronous chains of services | Latency adds up; failures cascade | Async, caching, aggregation, timeouts |
| Retry storms | Outage worsens on recovery | Backoff + jitter, retry budgets, circuit breakers |
| Oversized connection pools | DB overload, lock contention | Size with Little's Law; pooler |
| Cache without TTL/invalidation plan | Stale data incidents, memory leaks | TTLs, keys with versions, invalidation strategy |
| Load testing with closed model only | Coordinated omission hides tail latency | Open-model, constant-arrival-rate |
| Testing on tiny datasets | Production plans differ; surprises at scale | Production-like data volumes |
| Scaling stateful services horizontally without partitioning | Contention, inconsistency | Partition/shard or keep single writer + replicas |
| Ignoring third-party limits | Throttling at peak | Capacity planning includes vendors |

---

## 21. Scenario Playbooks

### 21.1 Preparing for a festival sale (e.g. Diwali/Big Billion-style events)

```text
T−8 weeks: Forecast traffic with business (peak RPS, orders/min, payment mix); identify critical journeys
T−6 weeks: Load test at 3–5× forecast; fix bottlenecks; test autoscaling and cache warm-up
T−4 weeks: Request cloud quota and third-party (PSP, SMS, email) limit increases; CDN pre-warming if offered
T−3 weeks: Game day: dependency failures (PSP down, cache flush, AZ loss); verify degradation modes and runbooks
T−2 weeks: Code freeze for risky changes; feature flags ready for kill switches; virtual waiting room configured
T−1 week:  Pre-scale baseline capacity; verify dashboards, alerts, on-call rota, war-room comms
T−0:       Scale ahead of sale start; watch SLOs and business metrics; shed non-critical features if needed
T+1 week:  Postmortem/retro with data; update capacity model
```

### 21.2 Investigating a latency regression after a deploy

```text
1. Confirm: SLO burn / p99 increase correlated with deploy annotation
2. Scope: which routes, tenants, regions, versions (canary vs stable)?
3. Mitigate: roll back or disable the flag if user impact is significant
4. Diagnose: compare traces before/after (new spans? more DB calls?), profile diff between versions, query plan changes
5. Fix and add a guard: benchmark, query-count test, or load-test threshold to prevent recurrence
```

### 21.3 Scaling from 1 to 10× users (startup)

```text
Single VPS → managed DB + 2+ app instances behind LB → CDN for static/catalogue → Redis/Valkey cache
→ queue for async work → read replica → autoscaling containers → split hot modules into services only when needed
At each step: load test, capacity model, cost per request check.
```

---

## 22. Checklists

### Design / pre-build
- [ ] Performance NFRs with percentiles, load, data volume
- [ ] Latency budget per hop with matching timeouts
- [ ] Capacity estimate (back-of-envelope) and scaling approach documented
- [ ] Caching, async, and data-scaling strategy chosen

### Pre-release
- [ ] Load test (open model) at ≥ expected peak × safety factor on prod-like data
- [ ] Stress and spike tests: graceful degradation and recovery verified
- [ ] Soak test for leaks (for long-running services or after major changes)
- [ ] Bottlenecks profiled and resolved; results compared to baseline
- [ ] Autoscaling tested; quotas and third-party limits confirmed

### Production
- [ ] SLOs and dashboards with percentiles; RUM for clients
- [ ] Continuous profiling or on-demand profiling ready
- [ ] Capacity reviewed quarterly and before known peaks
- [ ] Performance regression checks in CI and canary analysis
- [ ] Load-shedding/degradation modes and kill switches documented in runbooks

---

## 23. References

### Theory and practice
- Google SRE Book — Handling Overload: https://sre.google/sre-book/handling-overload/
- Google SRE Book — Addressing Cascading Failures: https://sre.google/sre-book/addressing-cascading-failures/
- Dean & Barroso — "The Tail at Scale" (Communications of the ACM, 2013): https://research.google/pubs/the-tail-at-scale/
- Brendan Gregg — USE Method & Flame Graphs: https://www.brendangregg.com/
- Gil Tene — "How NOT to Measure Latency" (coordinated omission) — talk/video widely available
- Neil Gunther — Universal Scalability Law: http://www.perfdynamics.com/Manifesto/USLscalability.html
- AWS Builders' Library (timeouts, retries, load shedding): https://aws.amazon.com/builders-library/

### Tools
- k6: https://grafana.com/docs/k6/latest/ · Gatling: https://docs.gatling.io/ · JMeter: https://jmeter.apache.org/ · Locust: https://locust.io/
- autocannon: https://github.com/mcollina/autocannon (and 40-autocannon_production_CLI.md)
- wrk2: https://github.com/giltene/wrk2 · ghz: https://ghz.sh/
- async-profiler: https://github.com/async-profiler/async-profiler · JMH: https://github.com/openjdk/jmh
- Go diagnostics: https://go.dev/doc/diagnostics · benchstat: https://pkg.go.dev/golang.org/x/perf/cmd/benchstat
- py-spy: https://github.com/benfred/py-spy · BenchmarkDotNet: https://benchmarkdotnet.org/
- Pyroscope: https://grafana.com/docs/pyroscope/

### Books
- *Systems Performance* (2nd ed.) — Brendan Gregg
- *Designing Data-Intensive Applications* — Martin Kleppmann
- *The Art of Capacity Planning* — Arun Kejariwal & John Allspaw
- *Release It!* (2nd ed.) — Michael Nygard

---

**Previous:** [18 — Observability & Monitoring](./18-observability-and-monitoring.md) · **Next:** [20 — Reliability & Disaster Recovery](./20-reliability-and-disaster-recovery.md)