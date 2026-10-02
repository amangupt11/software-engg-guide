# ⚙️ Backend Engineering — Production Engineering Guide

> How to build backend services that are correct, secure, observable, and operable: choosing runtimes and frameworks, service structure (hexagonal), request lifecycle and middleware, configuration and secrets, validation and serialisation, error handling, persistence and transactions, background jobs and messaging, concurrency models, caching, calling other services, authentication/authorisation, multi-tenancy, observability (OpenTelemetry, health checks), graceful startup/shutdown, performance, containerisation, files and notifications, scheduled jobs, serverless, upgrades, and reference service skeletons in Java/Spring Boot, Python/FastAPI, Go, Node.js/TypeScript, and .NET.
>
> Related: [04 Architecture](./04-software-architecture.md) · [05 Design](./05-software-design.md) · [06 Coding standards](./06-coding-standards.md) · [09 Security](./09-security-engineering.md) · [10 Database](./10-database-engineering.md) · [11 API & Integration](./11-api-and-integration.md) · [16 CI/CD](./16-devops-and-ci-cd.md) · [18 Observability](./18-observability-and-monitoring.md) · [39 VPS setup](./39-vps-enterprise-setup-guide.md)

---

## 📚 Table of Contents

- [⚙️ Backend Engineering — Production Engineering Guide](#️-backend-engineering--production-engineering-guide)
  - [📚 Table of Contents](#-table-of-contents)
  - [1. Principles](#1-principles)
  - [2. Choosing a Runtime and Framework](#2-choosing-a-runtime-and-framework)
    - [2.1 Mainstream options (verify current versions/support windows before starting)](#21-mainstream-options-verify-current-versionssupport-windows-before-starting)
    - [2.2 Spring Boot 4 notes (for Java teams)](#22-spring-boot-4-notes-for-java-teams)
    - [2.3 Decision factors](#23-decision-factors)
  - [3. Service Structure](#3-service-structure)
    - [3.1 Hexagonal layout (language-agnostic)](#31-hexagonal-layout-language-agnostic)
    - [3.2 Use-case handler shape](#32-use-case-handler-shape)
  - [4. Request Lifecycle and Middleware](#4-request-lifecycle-and-middleware)
  - [5. Configuration and Secrets](#5-configuration-and-secrets)
  - [6. Validation and Serialisation](#6-validation-and-serialisation)
  - [7. Error Handling](#7-error-handling)
  - [8. Persistence and Transactions](#8-persistence-and-transactions)
    - [Transactional outbox (pattern)](#transactional-outbox-pattern)
  - [9. Background Jobs, Queues, and Messaging](#9-background-jobs-queues-and-messaging)
    - [9.1 When to go async](#91-when-to-go-async)
    - [9.2 Job systems by ecosystem](#92-job-systems-by-ecosystem)
    - [9.3 Job rules](#93-job-rules)
  - [10. Concurrency Models](#10-concurrency-models)
  - [11. Caching](#11-caching)
  - [12. Calling Other Services](#12-calling-other-services)
  - [13. Authentication and Authorisation in Services](#13-authentication-and-authorisation-in-services)
  - [14. Multi-Tenancy](#14-multi-tenancy)
  - [15. Observability](#15-observability)
    - [15.1 OpenTelemetry baseline](#151-opentelemetry-baseline)
    - [15.2 Business observability](#152-business-observability)
  - [16. Health Checks, Startup, and Graceful Shutdown](#16-health-checks-startup-and-graceful-shutdown)
    - [16.1 Probes](#161-probes)
    - [16.2 Graceful shutdown sequence](#162-graceful-shutdown-sequence)
  - [17. Performance and Resource Management](#17-performance-and-resource-management)
  - [18. Containerisation and Runtime Configuration](#18-containerisation-and-runtime-configuration)
    - [18.1 General Dockerfile rules (chapter 06 §20, chapter 09 §17)](#181-general-dockerfile-rules-chapter-06-20-chapter-09-17)
    - [18.2 Language-specific runtime notes](#182-language-specific-runtime-notes)
  - [19. Files, Uploads, and Object Storage](#19-files-uploads-and-object-storage)
  - [20. Notifications — Email, SMS, Push](#20-notifications--email-sms-push)
  - [21. Scheduled Jobs](#21-scheduled-jobs)
  - [22. Serverless Backends](#22-serverless-backends)
  - [23. Security Hardening Checklist](#23-security-hardening-checklist)
  - [24. Testing Backend Services](#24-testing-backend-services)
  - [25. Upgrades and Runtime Lifecycle](#25-upgrades-and-runtime-lifecycle)
  - [26. Reference Skeletons](#26-reference-skeletons)
    - [26.1 Java / Spring Boot 4 (controller → use case)](#261-java--spring-boot-4-controller--use-case)
    - [26.2 Python / FastAPI](#262-python--fastapi)
    - [26.3 Go / net/http](#263-go--nethttp)
    - [26.4 Node.js / TypeScript (Fastify)](#264-nodejs--typescript-fastify)
    - [26.5 .NET / ASP.NET Core Minimal API](#265-net--aspnet-core-minimal-api)
  - [27. Checklists](#27-checklists)
    - [New service (production readiness)](#new-service-production-readiness)
    - [Code review (backend PR)](#code-review-backend-pr)
  - [28. References](#28-references)
    - [Runtimes and frameworks](#runtimes-and-frameworks)
    - [Practices](#practices)

---

## 1. Principles

| # | Principle |
|---:|---|
| 1 | **Stateless processes** — state lives in datastores, caches, and queues; any instance can serve any request (Twelve-Factor). |
| 2 | **Thin edges, rich core** — controllers/handlers translate; domain logic lives in the core (chapter 05). |
| 3 | **Every I/O has a timeout**, every retry is safe, every queue is bounded. |
| 4 | **Validate at the boundary, authorise every operation.** |
| 5 | **Observable by default** — structured logs, metrics, traces, health endpoints from the first commit. |
| 6 | **Fail fast at startup, degrade gracefully at runtime.** |
| 7 | **Idempotent by design** for anything that can be retried (HTTP, messages, jobs). |
| 8 | **Boring, supported runtimes** on LTS versions, upgraded on a schedule. |

---

## 2. Choosing a Runtime and Framework

### 2.1 Mainstream options (verify current versions/support windows before starting)

| Language / runtime | Frameworks | Strengths | Typical fit |
|---|---|---|---|
| **Java** (LTS: 25, 21) | **Spring Boot 4.x** (Spring Framework 7; GA Nov 2025, 4.1 in June 2026), Quarkus, Micronaut, Helidon | Mature ecosystem, virtual threads, observability, enterprise integrations | Enterprise services, fintech, large teams |
| **Kotlin** (JVM) | Spring Boot, Ktor | Concise, null-safe, coroutines | Same as Java; Android-sharing teams |
| **Node.js** (Active LTS, e.g. 24) / TypeScript | NestJS, Fastify, Express 5, Hono, Next.js route handlers | Shared language with frontend, great I/O concurrency | BFFs, APIs, real-time, startups |
| **Python** | **FastAPI**, Django (+ DRF), Flask, Litestar | Productivity, data/ML ecosystem | APIs near data/ML, admin-heavy apps (Django) |
| **Go** | Standard library `net/http` (enhanced routing since 1.22), chi, Gin, Echo, Connect/gRPC | Simple deployment (single binary), performance, concurrency | Infrastructure, high-throughput services, platform tools |
| **C# / .NET** (LTS: .NET 10) | ASP.NET Core (Minimal APIs, controllers), Orleans | Performance, tooling, enterprise | Microsoft-centric enterprises, high-performance APIs |
| **PHP** (8.x) | Laravel, Symfony | Productivity, hosting ubiquity | Web apps, CMS-adjacent products |
| **Rust** | Axum, Actix Web | Performance, memory safety | Latency-critical or resource-constrained services |
| **Ruby** | Rails | Productivity, conventions | Product startups, CRUD-rich apps |
| **Elixir** | Phoenix | Fault tolerance, real-time | Real-time systems, high concurrency |

### 2.2 Spring Boot 4 notes (for Java teams)

Spring Boot 4.0 is built on Spring Framework 7 and keeps a **Java 17 baseline** while giving first-class support to Java 25; it adopts **Jakarta EE 11** (Servlet 6.1), modularises auto-configuration into smaller starters, moves to JSpecify null-safety annotations, adds built-in API versioning and resilience features (retry, `@ConcurrencyLimit`), and upgrades major dependencies (Jackson 3, Hibernate 7, Tomcat 11, Spring Security 7). Undertow is not supported in 4.0. Plan the upgrade from 3.x as a project, not a dependency bump.

### 2.3 Decision factors

```text
1. Team expertise and hiring market (dominant factor)
2. Ecosystem fit (libraries for your domain: payments, PDF, ML, messaging)
3. Operational model (containers, serverless cold starts, memory footprint)
4. Performance needs (most business APIs are I/O-bound — any mainstream runtime is fine)
5. Organisational standardisation (paved roads, shared libraries, observability integrations)
Record in an ADR. Limit the number of backend stacks in an organisation (e.g. ≤ 2–3).
```

---

## 3. Service Structure

### 3.1 Hexagonal layout (language-agnostic)

```text
orders-service/
├── src/
│   ├── api/                 # inbound adapters: HTTP controllers/handlers, gRPC, message consumers
│   │   ├── http/
│   │   └── messaging/
│   ├── application/         # use cases / command & query handlers, transactions, authorisation decisions
│   ├── domain/              # entities, value objects, domain services, domain events, ports (interfaces)
│   ├── infrastructure/      # outbound adapters: repositories, PSP client, Kafka producer, cache
│   └── config/              # composition root, DI wiring, typed config
├── migrations/
├── tests/ (unit, integration, contract)
├── api/ (openapi.yaml, proto/, asyncapi.yaml)
├── Dockerfile
└── README.md
```

Dependency rule: `api → application → domain ← infrastructure` (infrastructure implements domain ports). Details: chapter 04 §5.1, chapter 05 §9.

### 3.2 Use-case handler shape

```text
handle(command, principal):
  1. validate command (structure already parsed at the edge; business validation here)
  2. authorise (principal may perform this action on this resource/tenant)
  3. load aggregate(s) via repository
  4. invoke domain behaviour (enforces invariants, records domain events)
  5. persist + write outbox events in ONE transaction
  6. return result DTO (never the entity)
```

---

## 4. Request Lifecycle and Middleware

```text
Ingress (LB / gateway: TLS, WAF, rate limits)
 → Server
   → Middleware chain (order matters):
      1. Request ID / trace context extraction (W3C traceparent)
      2. Access logging (start)
      3. Panic/exception recovery
      4. Timeouts / deadline propagation
      5. Request size limits
      6. CORS (if browser-facing)
      7. Authentication (token/session validation)
      8. Rate limiting (per principal/tenant)
      9. Tenant resolution (from authenticated identity)
     10. Body parsing + schema validation
   → Handler → use case → domain → adapters
   → Response mapping (DTO), error mapping (RFC 9457)
   → Metrics (RED) + access log (end)
```

| Rule | Status |
|---|:---:|
| Server-side request timeout shorter than the upstream LB timeout | 🔴 |
| Max request body size (e.g. 1 MB JSON default; larger only on upload endpoints) | 🔴 |
| Metrics labelled by **route template** and status class (not raw path) | 🔴 |
| Health endpoints excluded from auth and noisy logs (but not exposed publicly with sensitive detail) | 🟠 |

---

## 5. Configuration and Secrets

| Rule | Status |
|---|:---:|
| Config from environment/config service; one artifact for all environments | 🔴 |
| Typed config validated at startup; fail fast with clear errors (chapter 05 §18) | 🔴 |
| Secrets from a secret manager / workload identity; never in images, repos, or logs (chapter 09 §14) | 🔴 |
| Sensible defaults for non-secret settings; no defaults for secrets | 🔴 |
| Config changes audited; dynamic config (feature flags) via a flag service | 🟠 |
| Document every setting (name, type, default, example) in README or generated docs | 🟠 |

```yaml
# Spring Boot application.yaml (excerpt) — values injected from env/secret manager at runtime
spring:
  datasource:
    url: ${DATABASE_URL}
    username: ${DATABASE_USER}
    password: ${DATABASE_PASSWORD}
    hikari:
      maximum-pool-size: ${DB_POOL_SIZE:10}
      connection-timeout: 3000
  threads:
    virtual:
      enabled: true          # Java 21+: virtual threads for blocking I/O workloads
server:
  shutdown: graceful
management:
  endpoints.web.exposure.include: health,info,prometheus
  endpoint.health.probes.enabled: true
```

---

## 6. Validation and Serialisation

```text
🔴 Parse request bodies into typed DTOs with schema validation (Bean Validation, Pydantic, zod/TypeBox, FluentValidation, go-playground/validator)
🔴 Reject unknown fields on write endpoints (prevents mass assignment)
🔴 Separate request DTOs, domain objects, and response DTOs
🔴 Consistent JSON conventions (camelCase, RFC 3339 UTC timestamps, money as minor units + currency — chapter 11 §3)
🟠 Generate DTOs/types from OpenAPI where practical; validate responses against the spec in tests
🟠 Configure serialisers safely: no polymorphic type handling from untrusted input; fail on unknown enum values only where intended
```

```python
# FastAPI + Pydantic v2
from pydantic import BaseModel, ConfigDict, Field

class Money(BaseModel):
    amount_minor: int = Field(ge=0, alias="amountMinor")
    currency: str = Field(pattern=r"^[A-Z]{3}$")

class CreateOrderItem(BaseModel):
    model_config = ConfigDict(extra="forbid", populate_by_name=True)
    product_id: str = Field(pattern=r"^prd_[a-z0-9]{4,}$", alias="productId")
    quantity: int = Field(ge=1, le=100)

class CreateOrderRequest(BaseModel):
    model_config = ConfigDict(extra="forbid", populate_by_name=True)
    items: list[CreateOrderItem] = Field(min_length=1, max_length=100)
```

---

## 7. Error Handling

| Layer | Responsibility |
|---|---|
| Domain | Throws/returns domain errors (e.g. `InsufficientStock`, `InvalidTransition`) |
| Application | Maps infrastructure failures, enforces authorisation, decides retries |
| API | Maps errors to HTTP status + **RFC 9457** problem details; logs unexpected errors with trace ID |

```java
// Spring Framework: ProblemDetail is built in
@RestControllerAdvice
class ApiExceptionHandler {
    @ExceptionHandler(InsufficientStockException.class)
    ProblemDetail insufficientStock(InsufficientStockException ex) {
        var pd = ProblemDetail.forStatusAndDetail(HttpStatus.CONFLICT, ex.getMessage());
        pd.setType(URI.create("https://api.shopnow.example/problems/insufficient-stock"));
        pd.setTitle("Insufficient stock");
        pd.setProperty("code", "INSUFFICIENT_STOCK");
        return pd;
    }

    @ExceptionHandler(Exception.class)
    ProblemDetail unexpected(Exception ex) {
        log.error("Unhandled error", ex);                     // full detail server-side only
        var pd = ProblemDetail.forStatus(HttpStatus.INTERNAL_SERVER_ERROR);
        pd.setTitle("Internal error");
        pd.setProperty("code", "INTERNAL_ERROR");
        return pd;                                            // no stack trace to client
    }
}
```

Principles: chapter 05 §12; error format: chapter 11 §5.

---

## 8. Persistence and Transactions

```text
🔴 Transaction boundary = one use case (application layer); keep it short; no network calls inside
🔴 Repositories per aggregate; parameterised queries; least-privilege DB role
🔴 Migrations versioned and zero-downtime (chapter 10 §10)
🔴 Reliable event publication via transactional outbox (or CDC)
🟠 Optimistic locking for user-editable aggregates; idempotency keys for retried commands
🟠 Read models / replicas for heavy queries; route explicitly
🟠 Detect N+1 queries in tests (query counting)
```

### Transactional outbox (pattern)

```sql
CREATE TABLE outbox (
  id            uuid PRIMARY KEY,
  aggregate_id  text        NOT NULL,
  event_type    text        NOT NULL,
  payload       jsonb       NOT NULL,
  headers       jsonb       NOT NULL DEFAULT '{}',
  created_at    timestamptz NOT NULL DEFAULT now(),
  published_at  timestamptz
);
CREATE INDEX ix_outbox_unpublished ON outbox (created_at) WHERE published_at IS NULL;
```

```text
Use case transaction:  UPDATE orders …; INSERT INTO outbox (…OrderPlaced…);  COMMIT
Relay (worker or CDC/Debezium): read unpublished rows (FOR UPDATE SKIP LOCKED) → publish to broker → mark published
Consumers: idempotent (dedupe by event id)
```

---

## 9. Background Jobs, Queues, and Messaging

### 9.1 When to go async

```text
Move work out of the request path when it is: slow (> ~200 ms), retryable, not needed for the response,
bursty, or calls flaky third parties (emails, PDFs, webhooks, image processing, exports, reconciliation).
```

### 9.2 Job systems by ecosystem

| Ecosystem | Options |
|---|---|
| JVM | Spring `@Async` + queues, JobRunr, Quartz, Kafka/RabbitMQ consumers, Temporal |
| Node.js | BullMQ (Redis/Valkey), Graphile Worker / pg-boss (PostgreSQL), Temporal |
| Python | Celery, RQ, Dramatiq, Arq, Temporal |
| Go | Asynq, River (PostgreSQL), Machinery, Temporal |
| .NET | Hangfire, Quartz.NET, MassTransit, Wolverine |
| PHP | Laravel Queues/Horizon, Symfony Messenger |
| Ruby | Sidekiq, Solid Queue, GoodJob |
| Cloud | SQS + Lambda/workers, Cloud Tasks, Pub/Sub, Azure Service Bus/Queue Storage |

> A PostgreSQL-backed queue (pg-boss, River, Graphile Worker, Solid Queue, GoodJob) is often enough and keeps jobs transactional with your data. Move to a dedicated broker when throughput, fan-out, or streaming needs demand it.

### 9.3 Job rules

```text
🔴 Jobs are idempotent (safe to run twice) — at-least-once delivery is the norm
🔴 Retries with exponential backoff + jitter; max attempts; dead-letter queue with alerting
🔴 Timeouts per job; jobs carry tenant + correlation/trace context
🔴 Bounded concurrency per job type (protect DB and third parties)
🟠 Small payloads (IDs), not large blobs; load fresh state inside the job
🟠 Visibility: dashboard of queue depth, age of oldest job, failure rate — alert on growth
🟠 Graceful shutdown: finish or release in-flight jobs on SIGTERM
```

Messaging design (events, ordering, schemas): chapter 11 §14.

---

## 10. Concurrency Models

| Runtime | Model | Guidance |
|---|---|---|
| **Java 21+** | Platform threads + **virtual threads** | Use virtual threads for blocking I/O workloads (thread-per-request without large pools); avoid long `synchronized` blocks around blocking I/O where pinning applies on older JDKs; bound concurrency to downstream capacity (semaphores, `@ConcurrencyLimit` in Spring 7) |
| **Kotlin** | Coroutines | Structured concurrency; inject dispatchers; never block on `Dispatchers.Default` |
| **Node.js** | Single-threaded event loop + libuv thread pool | Never block the loop (CPU work → worker threads/separate service); monitor event-loop lag; use `cluster`/multiple processes or container replicas per core |
| **Python** | asyncio (FastAPI) or WSGI threads/processes (Django) | Don't call blocking libraries in `async def` without `run_in_threadpool`; scale with multiple worker processes (Gunicorn/Uvicorn workers) |
| **Go** | Goroutines + channels | Every goroutine has an owner and exit path (`errgroup`, context cancellation); bound fan-out |
| **.NET** | async/await on thread pool | Async all the way; pass `CancellationToken`; avoid sync-over-async |
| **Rust** | async (Tokio) | Don't block the executor (`spawn_blocking`) |

```go
// Go: bounded parallel fan-out with cancellation
g, ctx := errgroup.WithContext(ctx)
g.SetLimit(10)
for _, id := range ids {
    id := id
    g.Go(func() error { return fetchAndStore(ctx, id) })
}
if err := g.Wait(); err != nil { return err }
```

---

## 11. Caching

| Layer | Example | Notes |
|---|---|---|
| HTTP caching | `Cache-Control`, ETags for GETs (chapter 11 §8) | Cheapest cache |
| In-process | Caffeine (JVM), `cachetools`, `lru-cache`, `MemoryCache` | Per instance; small; TTL |
| Distributed | Redis/Valkey | Shared across instances; network hop |
| Database | Materialised views, read replicas | For heavy queries |

Design rules (keys, TTLs, stampede protection, tenant isolation): chapter 05 §17 and chapter 10 §18.

---

## 12. Calling Other Services

```text
🔴 One shared, configured HTTP/gRPC client per dependency (connection pooling, keep-alive) — never per request
🔴 Connect timeout + overall request timeout/deadline (derive from the caller's remaining budget)
🔴 Retries only for idempotent operations / with idempotency keys; exponential backoff + jitter; retry budget
🔴 Circuit breaker and bulkhead per dependency; fallbacks where meaningful
🔴 Propagate trace context and correlation IDs; send the caller's identity/audience-specific token
🟠 Typed clients generated from OpenAPI/proto; contract tests (chapter 08 §18)
🟠 Metrics per dependency: latency, errors, circuit state
```

Resilience libraries: Resilience4j / Spring Framework 7 resilience annotations (JVM), Polly / `Microsoft.Extensions.Http.Resilience` (.NET), `tenacity` + `httpx` timeouts (Python), `cockatiel` / `opossum` (Node.js), `failsafe-go`/`gobreaker` (Go), service-mesh policies. Example config: chapter 11 §19.2.

```ts
// Node.js: fetch with timeout via AbortSignal
const res = await fetch(`${PSP_URL}/charges`, {
  method: 'POST',
  headers: { 'content-type': 'application/json', 'idempotency-key': key, traceparent },
  body: JSON.stringify(payload),
  signal: AbortSignal.timeout(5_000),
});
```

---

## 13. Authentication and Authorisation in Services

| Concern | Practice |
|---|---|
| End-user requests | Validate access tokens (JWT signature, `iss`, `aud`, `exp`) or session via BFF; extract principal + tenant |
| Service-to-service | mTLS / workload identity + short-lived tokens scoped to the target service's audience |
| Authorisation | Central policy module/engine; check per resource and action inside use cases (chapter 09 §10) |
| Background jobs | Carry the initiating principal/tenant; re-check permissions when acting on behalf of users |
| Admin endpoints | Separate network path or strong role checks + MFA-backed identities; audited |
| Audit logging | Security-relevant actions logged with actor, tenant, resource, outcome (chapter 09 §21) |

```kotlin
// Spring Security 7 resource server (illustrative)
@Bean
fun security(http: HttpSecurity): SecurityFilterChain = http
    .authorizeHttpRequests {
        it.requestMatchers("/actuator/health/**").permitAll()
          .requestMatchers(HttpMethod.GET, "/v1/orders/**").hasAuthority("SCOPE_orders:read")
          .anyRequest().authenticated()
    }
    .oauth2ResourceServer { it.jwt { } }   // issuer/audience validation configured via properties
    .build()
// Object-level checks (owner/tenant) happen in the use case — scopes alone are not enough.
```

---

## 14. Multi-Tenancy

```text
🔴 Tenant ID derived from the authenticated principal (token claim / session), never trusted from a header or body alone
🔴 Tenant context propagated through request scope, jobs, events, logs, and traces
🔴 Every repository query scoped by tenant; PostgreSQL RLS as defense in depth (chapter 10 §6.4, chapter 09 §10.4)
🟠 Per-tenant rate limits, quotas, and noisy-neighbour protection
🟠 Tenant-aware metrics (top tenants by load/errors) with cardinality controls
🟠 Tenant data export and deletion procedures (offboarding, privacy rights)
```

---

## 15. Observability

### 15.1 OpenTelemetry baseline

```text
Traces:  auto-instrumentation for HTTP server/client, DB, messaging; manual spans for key business steps
Metrics: RED per route (rate, errors, duration histograms), dependency metrics, runtime metrics
         (GC, heap, event-loop lag, goroutines, thread pools), pool saturation (DB, HTTP clients), queue depth
Logs:    structured JSON, with trace_id/span_id, tenant_id, request_id; no secrets/PII (chapter 05 §19)
Export:  OTLP → OpenTelemetry Collector → backends (Prometheus/Mimir, Tempo/Jaeger, Loki/Elastic, vendors)
```

| Ecosystem | OpenTelemetry integration |
|---|---|
| Java | OpenTelemetry Java agent, or Spring Boot 4's OpenTelemetry starter + Micrometer |
| .NET | `OpenTelemetry.Extensions.Hosting` + ASP.NET Core/HttpClient instrumentation |
| Node.js | `@opentelemetry/sdk-node` + auto-instrumentations (load before app code) |
| Python | `opentelemetry-distro` + `opentelemetry-instrument` |
| Go | `go.opentelemetry.io/otel` + `otelhttp`, `otelgrpc` contrib packages |

### 15.2 Business observability

Emit domain metrics (orders placed, payments failed by reason, refunds) — they catch "everything is 200 OK but revenue dropped" failures. Define SLOs per user journey (chapter 18, 20).

---

## 16. Health Checks, Startup, and Graceful Shutdown

### 16.1 Probes

| Probe | Question | Should check | Should NOT check |
|---|---|---|---|
| **Liveness** | Is the process wedged? Restart it? | Process responsive (event loop/threads alive) | Dependencies (DB outage would restart all pods → worse) |
| **Readiness** | Should it receive traffic now? | Initialised, critical dependencies reachable (with care), not shutting down | Non-critical dependencies |
| **Startup** | Has slow initialisation finished? | Warm-up complete (migrations not run here), caches primed | — |

```yaml
# Kubernetes probes (Spring Boot actuator paths as example)
startupProbe:   { httpGet: { path: /actuator/health/liveness, port: 8080 }, failureThreshold: 30, periodSeconds: 2 }
livenessProbe:  { httpGet: { path: /actuator/health/liveness, port: 8080 }, periodSeconds: 10 }
readinessProbe: { httpGet: { path: /actuator/health/readiness, port: 8080 }, periodSeconds: 5 }
```

### 16.2 Graceful shutdown sequence

```text
SIGTERM received
 1. Mark NOT READY (readiness fails) so the load balancer stops sending new requests
 2. Wait briefly for LB/endpoint propagation (a few seconds; preStop hook sleep in Kubernetes)
 3. Stop accepting new connections; finish in-flight requests (bounded by a shutdown timeout)
 4. Stop consumers/workers: finish or nack in-flight messages/jobs
 5. Flush telemetry, close DB pools and clients
 6. Exit 0 before terminationGracePeriodSeconds expires (else SIGKILL)
```

```go
srv := &http.Server{Addr: ":8080", Handler: mux, ReadHeaderTimeout: 5 * time.Second}
go func() { _ = srv.ListenAndServe() }()

ctx, stop := signal.NotifyContext(context.Background(), syscall.SIGINT, syscall.SIGTERM)
<-ctx.Done()
stop()
ready.Store(false)                      // readiness endpoint starts failing
time.Sleep(5 * time.Second)             // allow LB to observe
shutdownCtx, cancel := context.WithTimeout(context.Background(), 20*time.Second)
defer cancel()
_ = srv.Shutdown(shutdownCtx)           // drains in-flight requests
```

Run database migrations as a **separate step/job** in the deployment pipeline, not during app startup on every replica.

---

## 17. Performance and Resource Management

| Area | Practice |
|---|---|
| Profiling | Continuous profiling (Pyroscope, Parca, cloud profilers); JFR/async-profiler (JVM), `pprof` (Go), `--cpu-prof`/clinic.js (Node), py-spy (Python), dotnet-trace/counters (.NET) |
| Connection pools | Size DB/HTTP pools deliberately; monitor wait time and saturation (chapter 10 §11) |
| Memory | Container-aware runtime settings; watch heap, GC pauses, leaks under soak tests |
| Payloads | Pagination, compression (gzip/Brotli/zstd), field selection; stream large responses |
| Hot paths | Avoid N+1, unnecessary serialisation, logging in tight loops |
| Load testing | Before major releases and capacity changes (chapter 08 §20.1, **40**) |

---

## 18. Containerisation and Runtime Configuration

### 18.1 General Dockerfile rules (chapter 06 §20, chapter 09 §17)

Multi-stage, minimal runtime image, non-root user, pinned base images by digest, no secrets in layers, `HEALTHCHECK` only where the orchestrator doesn't probe, explicit `EXPOSE` and entrypoint, `.dockerignore`.

### 18.2 Language-specific runtime notes

| Runtime | Container notes |
|---|---|
| **JVM** | Modern JDKs are container-aware; set `-XX:MaxRAMPercentage=75` (or similar) instead of fixed `-Xmx`; consider CDS/AOT cache or GraalVM native images for startup-sensitive workloads; Spring Boot layered jars / Buildpacks |
| **Node.js** | Run `node` directly (not `npm start`) so signals reach the process; set `--max-old-space-size` relative to memory limit if needed; `NODE_ENV=production`; one process per container, scale with replicas |
| **Python** | Gunicorn/Uvicorn workers sized to CPU; `PYTHONUNBUFFERED=1`; use `uv` for fast, locked installs; slim base images |
| **Go** | Static binary in distroless/scratch; `GOMAXPROCS` respects container CPU limits in recent Go versions (verify for your version) |
| **.NET** | Official runtime/aspnet images (chiseled/distroless variants); `DOTNET_` env config; consider Native AOT for small, fast-starting services |

```dockerfile
# Java (Spring Boot) — layered, non-root (illustrative)
FROM eclipse-temurin:25-jdk AS build
WORKDIR /src
COPY . .
RUN ./gradlew bootJar --no-daemon && java -Djarmode=tools -jar build/libs/app.jar extract --layers --launcher --destination /extracted

FROM eclipse-temurin:25-jre
RUN useradd --system --uid 10001 app
WORKDIR /app
COPY --from=build /extracted/dependencies/ ./
COPY --from=build /extracted/spring-boot-loader/ ./
COPY --from=build /extracted/snapshot-dependencies/ ./
COPY --from=build /extracted/application/ ./
USER 10001
ENV JAVA_TOOL_OPTIONS="-XX:MaxRAMPercentage=75 -XX:+ExitOnOutOfMemoryError"
EXPOSE 8080
ENTRYPOINT ["java", "org.springframework.boot.loader.launch.JarLauncher"]
```

> Layer extraction commands and launcher class names change between Spring Boot versions — check the reference docs for your version, or use Cloud Native Buildpacks (`./gradlew bootBuildImage`).

---

## 19. Files, Uploads, and Object Storage

```text
🔴 Store files in object storage (S3/GCS/Azure Blob/MinIO), not on local disk of stateless instances
🔴 Direct-to-storage uploads with pre-signed URLs (short expiry, size and content-type conditions)
🔴 Validate type by content (magic bytes), size limits, malware scanning for user uploads where required
🔴 Private buckets; serve via signed URLs or an authorising proxy/CDN; never public-by-default
🟠 Metadata row in DB (owner, tenant, checksum, size, content type, scan status)
🟠 Image processing async (thumbnails) via jobs; strip EXIF/location metadata from user images
🟠 Lifecycle rules for retention and storage tiering
```

---

## 20. Notifications — Email, SMS, Push

| Channel | Practice |
|---|---|
| **Email** | Transactional provider (SES, SendGrid, Postmark, Mailgun, etc.); SPF, DKIM, DMARC configured (see **39**); templates versioned; bounce/complaint handling; unsubscribe for marketing |
| **SMS / OTP** | Provider with Indian DLT registration compliance for sender IDs and templates (TRAI rules) where sending to Indian numbers; rate limits per number/IP; OTP expiry and attempt limits |
| **WhatsApp / chat** | Business API via approved providers; template approval; opt-in consent |
| **Push** | FCM (Android/web), APNs (iOS); token lifecycle management; payload without sensitive data |

```text
🔴 Send asynchronously via jobs; idempotent per (user, template, business event)
🔴 Respect consent and preferences (marketing vs transactional); quiet hours where appropriate
🟠 Notification service abstraction with provider failover for critical messages (OTP)
🟠 Track delivery status via provider webhooks (verify signatures)
```

---

## 21. Scheduled Jobs

```text
🔴 Exactly one active scheduler per job across replicas: use a scheduler service (Kubernetes CronJob, cloud schedulers),
    or a distributed lock/leader election (ShedLock, DB advisory locks, Quartz clustering)
🔴 Jobs idempotent and resumable (checkpointing for long jobs); bounded run time; overlapping runs prevented
🔴 Time zone explicit (store schedules in UTC or with IANA zone, e.g. Asia/Kolkata for business-day jobs)
🟠 Monitor: last success time, duration, failures — alert if a job hasn't succeeded within its window ("dead man's switch")
🟠 Batch large work into chunks; throttle to protect the DB
```

```yaml
# Kubernetes CronJob (excerpt)
spec:
  schedule: "30 1 * * *"          # 01:30 in the configured time zone
  timeZone: "Asia/Kolkata"
  concurrencyPolicy: Forbid
  startingDeadlineSeconds: 600
  jobTemplate:
    spec:
      backoffLimit: 2
      activeDeadlineSeconds: 3600
```

---

## 22. Serverless Backends

| Consideration | Guidance |
|---|---|
| Good fit | Event-driven glue, webhooks, scheduled tasks, bursty low-traffic APIs, file processing |
| Cold starts | Keep functions small; choose fast-starting runtimes (Node, Go, Python; JVM with SnapStart/CRaC/native images; .NET Native AOT); provisioned concurrency for latency-critical paths |
| Connections | Use connection proxies/poolers for relational DBs (RDS Proxy, PgBouncer, Data API/HTTP drivers) |
| Limits | Timeouts, payload sizes, concurrency limits — design around them |
| State | Stateless; durable workflows via Step Functions/Durable Functions/Temporal |
| Observability | Structured logs, traces (OTel layers/extensions), cold-start metrics |
| Cost | Great at low/spiky volume; compare with containers at steady high volume |

---

## 23. Security Hardening Checklist

- [ ] Dependencies scanned (SCA) and updated; base images patched
- [ ] Inputs validated with schemas; unknown fields rejected on writes
- [ ] AuthN on all non-public endpoints; object-level authorisation in use cases
- [ ] Parameterised queries; least-privilege DB credentials
- [ ] Secrets from secret manager / workload identity; none in env files committed or images
- [ ] Outbound calls: timeouts, TLS verification on, allow-listed destinations for user-supplied URLs (SSRF)
- [ ] Rate limits and request size limits
- [ ] Errors don't leak internals; security events logged and alerted
- [ ] Admin/debug endpoints (actuator, pprof, metrics) not publicly exposed
- [ ] Containers non-root, read-only FS where possible, minimal capabilities
- [ ] Webhooks verified; idempotent consumers
- [ ] Deserialisation safe; no dynamic code execution on untrusted data

Full guidance: chapter **09**.

---

## 24. Testing Backend Services

| Level | Focus | Tools |
|---|---|---|
| Unit | Domain logic, use cases with fakes | JUnit/Kotest, pytest, Go testing, xUnit, Vitest/Jest |
| Integration | Repositories, migrations, messaging, caches with real dependencies | **Testcontainers**, Docker Compose |
| API / component | HTTP layer in-process (routing, validation, error mapping, auth) | Spring `MockMvc`/`RestTestClient`/`WebTestClient`, FastAPI `TestClient`/`httpx`, `httptest` (Go), `WebApplicationFactory` (.NET), Supertest |
| Contract | Provider verification of consumer contracts / spec conformance | Pact, Schemathesis, Spring Cloud Contract |
| Performance | Latency/throughput budgets | k6, Gatling, autocannon (**40**) |
| Resilience | Dependency failures, timeouts, retries | WireMock/Toxiproxy fault injection |

Strategy and STLC: chapter **08**.

---

## 25. Upgrades and Runtime Lifecycle

```text
🔴 Runtime on a supported LTS (Java 25/21, Node Active/Maintenance LTS, .NET LTS, supported Python/Go/PHP versions)
🔴 Framework on a supported line (e.g. Spring Boot 4.x OSS support window; check end-of-support dates)
🔴 Security patches applied within vulnerability SLAs (chapter 09 §20)
🟠 Quarterly dependency currency review; scheduled major upgrades with tracking tickets
🟠 Upgrade tooling: OpenRewrite recipes (Spring Boot/Java migrations), `dotnet upgrade-assistant`, `ng update`, codemods
🟠 Run tests on the next runtime version in CI early (canary build matrix)
```

Version lifecycle references: https://endoflife.date/ (community-maintained aggregator — confirm with official vendor pages).

---

## 26. Reference Skeletons

### 26.1 Java / Spring Boot 4 (controller → use case)

```java
@RestController
@RequestMapping("/v1/orders")
class OrderController {
    private final PlaceOrderHandler placeOrder;
    OrderController(PlaceOrderHandler placeOrder) { this.placeOrder = placeOrder; }

    @PostMapping
    ResponseEntity<OrderResponse> create(@Valid @RequestBody CreateOrderRequest req,
                                         @RequestHeader("Idempotency-Key") String idempotencyKey,
                                         @AuthenticationPrincipal Jwt jwt) {
        var principal = Principal.from(jwt);
        var order = placeOrder.handle(req.toCommand(idempotencyKey), principal);
        return ResponseEntity.created(URI.create("/v1/orders/" + order.id())).body(OrderResponse.from(order));
    }
}

@Service
class PlaceOrderHandler {
    private final OrderRepository orders;
    private final Outbox outbox;
    private final IdempotencyStore idempotency;
    // constructor injection omitted

    @Transactional
    public Order handle(PlaceOrderCommand cmd, Principal principal) {
        return idempotency.execute(principal.tenantId(), cmd.idempotencyKey(), cmd.fingerprint(), () -> {
            var order = Order.create(principal.tenantId(), principal.userId(), cmd.items());
            orders.save(order);
            outbox.add(order.pullEvents());
            return order;
        });
    }
}
```

### 26.2 Python / FastAPI

```python
from fastapi import APIRouter, Depends, Header, status
router = APIRouter(prefix="/v1/orders")

@router.post("", status_code=status.HTTP_201_CREATED, response_model=OrderResponse)
async def create_order(
    body: CreateOrderRequest,
    idempotency_key: str = Header(alias="Idempotency-Key"),
    principal: Principal = Depends(current_principal),          # validates token, extracts tenant
    handler: PlaceOrderHandler = Depends(get_place_order_handler),
) -> OrderResponse:
    order = await handler.handle(body.to_command(idempotency_key), principal)
    return OrderResponse.from_domain(order)
```

Run with multiple workers behind a reverse proxy: `uvicorn app.main:app --workers 4 --proxy-headers` (or Gunicorn with Uvicorn workers), sized to CPU and load tests.

### 26.3 Go / net/http

```go
func (h *OrderHandler) Create(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()
    p, ok := auth.PrincipalFrom(ctx)
    if !ok { problem.Write(w, http.StatusUnauthorized, "UNAUTHENTICATED", "authentication required"); return }

    var req CreateOrderRequest
    dec := json.NewDecoder(http.MaxBytesReader(w, r.Body, 1<<20))
    dec.DisallowUnknownFields()
    if err := dec.Decode(&req); err != nil { problem.Write(w, http.StatusBadRequest, "INVALID_JSON", err.Error()); return }
    if err := req.Validate(); err != nil { problem.WriteValidation(w, err); return }

    order, err := h.placeOrder.Handle(ctx, req.ToCommand(r.Header.Get("Idempotency-Key")), p)
    switch {
    case errors.Is(err, domain.ErrInsufficientStock):
        problem.Write(w, http.StatusConflict, "INSUFFICIENT_STOCK", "insufficient stock")
    case err != nil:
        slog.ErrorContext(ctx, "place order failed", "err", err)
        problem.Write(w, http.StatusInternalServerError, "INTERNAL_ERROR", "internal error")
    default:
        w.Header().Set("Location", "/v1/orders/"+order.ID)
        writeJSON(w, http.StatusCreated, ToResponse(order))
    }
}
// mux.HandleFunc("POST /v1/orders", h.Create)   // method-aware routing in the standard library (Go 1.22+)
```

### 26.4 Node.js / TypeScript (Fastify)

```ts
app.post('/v1/orders', {
  schema: { body: CreateOrderSchema, headers: IdempotencyHeaderSchema, response: { 201: OrderResponseSchema } },
  preHandler: [app.authenticate],
}, async (req, reply) => {
  const order = await placeOrder.handle(toCommand(req.body, req.headers['idempotency-key']), req.principal);
  return reply.code(201).header('location', `/v1/orders/${order.id}`).send(toResponse(order));
});
```

### 26.5 .NET / ASP.NET Core Minimal API

```csharp
app.MapPost("/v1/orders", async (CreateOrderRequest req, [FromHeader(Name = "Idempotency-Key")] string key,
                                 ClaimsPrincipal user, PlaceOrderHandler handler, CancellationToken ct) =>
{
    var order = await handler.HandleAsync(req.ToCommand(key), Principal.From(user), ct);
    return Results.Created($"/v1/orders/{order.Id}", OrderResponse.From(order));
})
.RequireAuthorization("orders:write")
.WithName("CreateOrder")
.ProducesProblem(StatusCodes.Status409Conflict);
```

---

## 27. Checklists

### New service (production readiness)
- [ ] Runtime/framework choice recorded (ADR); on supported LTS
- [ ] Hexagonal structure; boundaries enforced
- [ ] Contract (OpenAPI/proto/AsyncAPI) in repo; linted; breaking-change checks
- [ ] Typed config validated at startup; secrets from secret manager
- [ ] Input validation; RFC 9457 errors; no internals leaked
- [ ] AuthN + object-level AuthZ; tenant isolation; audit logs
- [ ] Timeouts, retries (safe), circuit breakers for all dependencies
- [ ] Idempotency keys and idempotent consumers/jobs
- [ ] Outbox/CDC for events; DLQs monitored
- [ ] OpenTelemetry traces, RED metrics, structured logs, business metrics, SLOs
- [ ] Liveness/readiness/startup probes; graceful shutdown tested
- [ ] Container: non-root, minimal, pinned, scanned; resource requests/limits set
- [ ] Migrations run as a separate pipeline step; zero-downtime patterns
- [ ] Load test at expected peak × safety factor
- [ ] Runbook: dashboards, alerts, common failures, rollback

### Code review (backend PR)
- [ ] Transaction boundaries correct; no remote calls inside transactions
- [ ] No N+1 queries; pagination on list queries
- [ ] Errors mapped correctly; logs useful and safe
- [ ] Concurrency safe (locks/constraints/optimistic locking)
- [ ] New config documented; defaults safe
- [ ] Tests at the right levels (unit + Testcontainers integration for persistence)

---

## 28. References

### Runtimes and frameworks
- Spring Boot: https://spring.io/projects/spring-boot · Spring Boot 4 / Framework 7 overview (InfoQ): https://www.infoq.com/news/2025/11/spring-7-spring-boot-4
- Quarkus: https://quarkus.io/ · Micronaut: https://micronaut.io/ · Ktor: https://ktor.io/
- Node.js releases: https://nodejs.org/en/about/previous-releases · Fastify: https://fastify.dev/ · NestJS: https://nestjs.com/
- FastAPI: https://fastapi.tiangolo.com/ · Django: https://www.djangoproject.com/ · Pydantic: https://docs.pydantic.dev/
- Go: https://go.dev/doc/ · ASP.NET Core: https://learn.microsoft.com/aspnet/core/ · .NET support policy: https://dotnet.microsoft.com/platform/support/policy
- Laravel: https://laravel.com/docs · Symfony: https://symfony.com/doc · Axum: https://docs.rs/axum · Rails: https://guides.rubyonrails.org/

### Practices
- The Twelve-Factor App: https://12factor.net/
- OpenTelemetry: https://opentelemetry.io/docs/
- Kubernetes probes: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/
- Kubernetes CronJob: https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/
- Transactional outbox (microservices.io): https://microservices.io/patterns/data/transactional-outbox.html
- Resilience4j: https://resilience4j.readme.io/ · Polly: https://www.pollydocs.org/
- Testcontainers: https://testcontainers.com/
- OpenRewrite: https://docs.openrewrite.org/
- endoflife.date: https://endoflife.date/
- RFC 9457 Problem Details: https://www.rfc-editor.org/rfc/rfc9457

---

**Previous:** [12 — Frontend Engineering](./12-frontend-engineering.md) · **Next:** [14 — Mobile Engineering](./14-mobile-engineering.md)