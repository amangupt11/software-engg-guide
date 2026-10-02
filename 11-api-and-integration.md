# 🔌 API & Integration — Production Engineering Guide

> How to design, document, evolve, secure, and operate APIs and integrations: API-first and contract-first workflows, choosing between REST, gRPC, GraphQL, events, webhooks, and real-time channels, REST design rules (resources, methods, status codes, pagination, errors with RFC 9457, idempotency, caching, rate limits), **OpenAPI 3.2**, versioning and deprecation, gRPC/Protocol Buffers, GraphQL, event-driven integration with AsyncAPI and CloudEvents, webhooks, gateways, third-party integration patterns, batch/file exchange, developer experience, observability, governance, and AI/agent integrations.
>
> Related: [04 Integration architecture §12](./04-software-architecture.md#12-integration-architecture) · [05 Interface & error design](./05-software-design.md#11-interface-and-contract-design) · [08 Contract testing §18](./08-testing-and-quality.md#18-contract-testing) · [09 API security §6](./09-security-engineering.md#6-api-security) · [13 Backend](./13-backend-engineering.md)

---

## 📚 Table of Contents

- [🔌 API \& Integration — Production Engineering Guide](#-api--integration--production-engineering-guide)
  - [📚 Table of Contents](#-table-of-contents)
  - [1. Principles](#1-principles)
  - [2. Choosing an Integration Style](#2-choosing-an-integration-style)
    - [Decision guide](#decision-guide)
  - [3. REST API Design](#3-rest-api-design)
    - [3.1 Resources and URIs](#31-resources-and-uris)
    - [3.2 Resource representation](#32-resource-representation)
    - [3.3 Standard headers](#33-standard-headers)
  - [4. HTTP Methods and Status Codes](#4-http-methods-and-status-codes)
    - [4.1 Methods (RFC 9110)](#41-methods-rfc-9110)
    - [4.2 Status codes — use a small, consistent set](#42-status-codes--use-a-small-consistent-set)
  - [5. Errors — RFC 9457 Problem Details](#5-errors--rfc-9457-problem-details)
  - [6. Pagination, Filtering, Sorting, Field Selection](#6-pagination-filtering-sorting-field-selection)
    - [6.1 Pagination styles](#61-pagination-styles)
    - [6.2 Filtering and sorting](#62-filtering-and-sorting)
    - [6.3 Field selection and expansion](#63-field-selection-and-expansion)
    - [6.4 Bulk operations](#64-bulk-operations)
  - [7. Idempotency, Concurrency, and Long-Running Operations](#7-idempotency-concurrency-and-long-running-operations)
    - [7.1 Idempotency keys for POST](#71-idempotency-keys-for-post)
    - [7.2 Retry guidance for clients](#72-retry-guidance-for-clients)
    - [7.3 Optimistic concurrency with ETags](#73-optimistic-concurrency-with-etags)
    - [7.4 Long-running operations](#74-long-running-operations)
  - [8. Caching and Conditional Requests](#8-caching-and-conditional-requests)
  - [9. Rate Limiting and Quotas](#9-rate-limiting-and-quotas)
    - [9.1 Algorithms](#91-algorithms)
    - [9.2 Policy design](#92-policy-design)
  - [10. OpenAPI 3.2 and Contract-First Workflow](#10-openapi-32-and-contract-first-workflow)
    - [10.1 OpenAPI versions](#101-openapi-versions)
    - [10.2 Contract-first workflow](#102-contract-first-workflow)
    - [10.3 Example OpenAPI fragment](#103-example-openapi-fragment)
    - [10.4 Spectral ruleset (style-guide enforcement)](#104-spectral-ruleset-style-guide-enforcement)
  - [11. Versioning, Evolution, and Deprecation](#11-versioning-evolution-and-deprecation)
    - [11.1 Compatible vs breaking changes](#111-compatible-vs-breaking-changes)
    - [11.2 Versioning strategies](#112-versioning-strategies)
    - [11.3 Deprecation signalling](#113-deprecation-signalling)
  - [12. gRPC and Protocol Buffers](#12-grpc-and-protocol-buffers)
    - [12.1 Protobuf design rules](#121-protobuf-design-rules)
    - [12.2 gRPC operations](#122-grpc-operations)
  - [13. GraphQL](#13-graphql)
    - [13.1 Schema design](#131-schema-design)
    - [13.2 Rules](#132-rules)
  - [14. Event-Driven Integration](#14-event-driven-integration)
    - [14.1 Event types](#141-event-types)
    - [14.2 CloudEvents envelope (CNCF, v1.0)](#142-cloudevents-envelope-cncf-v10)
    - [14.3 AsyncAPI 3.0](#143-asyncapi-30)
    - [14.4 Delivery and processing rules](#144-delivery-and-processing-rules)
    - [14.5 Brokers](#145-brokers)
  - [15. Webhooks](#15-webhooks)
    - [15.1 Sending webhooks (provider side)](#151-sending-webhooks-provider-side)
    - [15.2 Receiving webhooks (consumer side)](#152-receiving-webhooks-consumer-side)
  - [16. Real-Time APIs — WebSocket and SSE](#16-real-time-apis--websocket-and-sse)
  - [17. Authentication and Authorisation for APIs](#17-authentication-and-authorisation-for-apis)
  - [18. API Gateways and Management](#18-api-gateways-and-management)
    - [18.1 Gateway responsibilities](#181-gateway-responsibilities)
    - [18.2 Options](#182-options)
    - [18.3 BFF (Backend for Frontend)](#183-bff-backend-for-frontend)
  - [19. Integrating with Third-Party APIs](#19-integrating-with-third-party-apis)
    - [19.1 Integration design checklist](#191-integration-design-checklist)
    - [19.2 Example: resilient client configuration (Resilience4j, Spring Boot `application.yml`)](#192-example-resilient-client-configuration-resilience4j-spring-boot-applicationyml)
  - [20. Batch and File-Based Integration](#20-batch-and-file-based-integration)
  - [21. Workflow Orchestration](#21-workflow-orchestration)
  - [22. Developer Experience and Documentation](#22-developer-experience-and-documentation)
  - [23. API Observability](#23-api-observability)
  - [24. API Governance](#24-api-governance)
  - [25. AI and Agent Integrations](#25-ai-and-agent-integrations)
  - [26. Checklists](#26-checklists)
    - [New API (design review)](#new-api-design-review)
    - [Event / webhook integration](#event--webhook-integration)
    - [Third-party integration](#third-party-integration)
  - [27. References](#27-references)
    - [HTTP and API standards](#http-and-api-standards)
    - [Specifications](#specifications)
    - [Style guides](#style-guides)
    - [Tools](#tools)
    - [Security](#security)

---

## 1. Principles

| # | Principle |
|---:|---|
| 1 | **APIs are products.** They have users (developers), documentation, versioning, SLAs, and a lifecycle. |
| 2 | **Contract-first.** Design and review the contract (OpenAPI/AsyncAPI/proto) before implementing. |
| 3 | **Consistency beats cleverness.** One style guide across all APIs; linted automatically. |
| 4 | **Never break consumers.** Evolve additively; version and deprecate deliberately. |
| 5 | **Secure by default.** Authenticated, authorised per object, rate-limited, validated (chapter 09). |
| 6 | **Design for failure.** Timeouts, retries, idempotency, and clear errors on both sides. |
| 7 | **Observable.** Every request traceable end to end. |
| 8 | **Tolerant reader, strict writer** (Postel, tempered): ignore unknown fields when reading; send only valid, documented data. |

---

## 2. Choosing an Integration Style

| Style | Strengths | Weaknesses | Choose for |
|---|---|---|---|
| **REST / HTTP+JSON** | Universal, cacheable, tooling, human-readable | Over/under-fetching; chatty for complex UIs | Public/partner APIs, CRUD-like resources, most internal APIs |
| **gRPC (Protobuf)** | Fast, typed, streaming, codegen | Browser support needs gRPC-Web/Connect; less human-readable | Internal service-to-service, low latency, streaming |
| **GraphQL** | Client-shaped queries, one round trip, typed schema | Caching, authorisation per field, query cost control | Aggregating data for diverse frontends (BFF) |
| **Async messaging (queues)** | Decoupled, load-levelling, retries | Eventual consistency, ops overhead | Background work, commands processed later |
| **Event streaming / pub-sub** | Loose coupling, replay, many consumers | Schema governance, ordering, debugging | Domain events, data integration, analytics |
| **Webhooks** | Push notifications to external parties | Delivery reliability, security, consumer availability | Notifying partners/customers of events |
| **WebSocket / SSE** | Real-time push | Connection management, scaling | Live updates, chat, dashboards |
| **Batch / files** | Bulk volume, legacy compatibility | Latency, reconciliation | Banking files, bulk imports/exports |

### Decision guide

```text
External developers / partners?         → REST (OpenAPI) + webhooks
Internal, latency-sensitive, polyglot?  → gRPC (or REST if simplicity matters more)
Many UI shapes over many services?      → GraphQL or BFF over REST/gRPC
"Something happened" others care about? → Events (AsyncAPI + CloudEvents) via a broker
Work that can happen later?             → Queue
Live updates to browsers/apps?          → SSE (server→client) or WebSocket (bidirectional)
```

---

## 3. REST API Design

### 3.1 Resources and URIs

```text
✅ Nouns, plural, kebab-case, hierarchical where ownership is real
GET    /v1/customers/{customerId}
GET    /v1/customers/{customerId}/orders
POST   /v1/orders
GET    /v1/orders/{orderId}
PATCH  /v1/orders/{orderId}
POST   /v1/orders/{orderId}/cancel            ← action as sub-resource when it isn't CRUD
GET    /v1/orders?status=paid&createdAfter=2026-10-01T00:00:00Z

❌ /getOrders   ❌ /v1/order/list   ❌ /v1/Orders   ❌ /v1/customers/{id}/orders/{oid}/items/{iid}/discounts/{did} (too deep)
```

| Rule | Status |
|---|:---:|
| Opaque, stable IDs in paths (`ord_7f3k…` or UUID) — never DB auto-increment for public APIs if enumeration matters | 🔴 |
| Max ~2 levels of nesting; beyond that, top-level resources with filters | 🟠 |
| JSON field naming consistent org-wide (`camelCase` recommended) | 🔴 |
| Timestamps RFC 3339 in UTC (`2026-10-02T09:30:00Z`); durations ISO 8601 | 🔴 |
| Money as `{ "amountMinor": 249900, "currency": "INR" }` or decimal string — never floats | 🔴 |
| Enums as strings (`"status": "paid"`), documented; clients must tolerate unknown values | 🔴 |
| `null` vs absent semantics documented (especially for PATCH) | 🟠 |

### 3.2 Resource representation

```json
{
  "id": "ord_7f3kq2m9x1",
  "status": "payment_failed",
  "customerId": "cus_2b8n4v",
  "total": { "amountMinor": 249900, "currency": "INR" },
  "items": [
    { "productId": "prd_k3m9", "quantity": 1, "unitPrice": { "amountMinor": 249900, "currency": "INR" } }
  ],
  "createdAt": "2026-10-02T09:30:00Z",
  "updatedAt": "2026-10-02T09:31:12Z"
}
```

### 3.3 Standard headers

| Header | Use |
|---|---|
| `Authorization: Bearer <token>` | Access token |
| `Content-Type` / `Accept` | `application/json`; `application/problem+json` for errors |
| `Idempotency-Key` | Safe retries of POST (§7) |
| `If-Match` / `ETag` | Optimistic concurrency (§7.3) |
| `traceparent` / `tracestate` | W3C Trace Context propagation |
| `X-Request-Id` (or use trace ID) | Correlation for support |
| `Retry-After` | With 429/503 |
| `RateLimit-*` / `RateLimit-Policy` | Rate limit information (IETF draft — §9) |
| `Deprecation` / `Sunset` / `Link: rel="deprecation"` | Deprecation signalling (§11) |
| `Location` | URI of a created resource (201) or status monitor (202) |

---

## 4. HTTP Methods and Status Codes

### 4.1 Methods (RFC 9110)

| Method | Semantics | Safe | Idempotent | Body |
|---|---|:---:|:---:|---|
| GET | Retrieve | ✅ | ✅ | No |
| HEAD | Headers only | ✅ | ✅ | No |
| POST | Create / process / command | ❌ | ❌ (make idempotent with keys) | Yes |
| PUT | Replace entire resource (or create at known URI) | ❌ | ✅ | Yes |
| PATCH | Partial update (JSON Merge Patch RFC 7396 or JSON Patch RFC 6902) | ❌ | ❌ (can be designed idempotent) | Yes |
| DELETE | Remove | ❌ | ✅ | Usually no |
| OPTIONS | Capabilities / CORS preflight | ✅ | ✅ | No |
| **QUERY** | Safe, idempotent query with a request body (IETF HTTP WG draft; supported as a first-class method in OpenAPI 3.2) | ✅ | ✅ | Yes |

> Until `QUERY` is widely supported by your clients, proxies, and frameworks, complex searches typically use `POST /v1/orders/search` — document that it is safe and idempotent.

### 4.2 Status codes — use a small, consistent set

| Code | Meaning | When |
|---|---|---|
| **200 OK** | Success with body | GET, PATCH/PUT returning resource |
| **201 Created** | Resource created | POST create; include `Location` |
| **202 Accepted** | Accepted for async processing | Long-running ops; `Location` to status resource |
| **204 No Content** | Success, no body | DELETE, some PUT/PATCH |
| **304 Not Modified** | Conditional GET hit | ETag/If-None-Match |
| **400 Bad Request** | Malformed request / validation failure | Include field errors |
| **401 Unauthorized** | Missing/invalid authentication | Include `WWW-Authenticate` |
| **403 Forbidden** | Authenticated but not allowed | Consider 404 to avoid leaking existence |
| **404 Not Found** | Resource doesn't exist (or not visible to caller) | |
| **405 Method Not Allowed** | Wrong method | Include `Allow` |
| **409 Conflict** | State conflict (duplicate, invalid transition) | e.g. cancel an already shipped order |
| **412 Precondition Failed** | `If-Match` failed | Optimistic concurrency conflict |
| **413 Content Too Large** | Payload too big | |
| **415 Unsupported Media Type** | Wrong `Content-Type` | |
| **422 Unprocessable Content** | Syntactically valid, semantically invalid | Business rule violation (team choice vs 400 — be consistent) |
| **428 Precondition Required** | Require `If-Match` for updates | |
| **429 Too Many Requests** | Rate limited | `Retry-After` |
| **500 Internal Server Error** | Unexpected server error | Never leak internals |
| **502 / 503 / 504** | Upstream failure / unavailable / timeout | `Retry-After` on 503 where possible |

---

## 5. Errors — RFC 9457 Problem Details

**RFC 9457** (*Problem Details for HTTP APIs*, obsoletes RFC 7807) defines a standard JSON error format with media type `application/problem+json`.

| Member | Meaning |
|---|---|
| `type` | URI identifying the problem type (dereferenceable to docs ideally) |
| `title` | Short, human-readable summary of the type |
| `status` | HTTP status code |
| `detail` | Human-readable explanation of this occurrence |
| `instance` | URI identifying this occurrence |
| *extensions* | Additional members (e.g. `code`, `errors`, `traceId`) |

```http
HTTP/1.1 422 Unprocessable Content
Content-Type: application/problem+json

{
  "type": "https://api.shopnow.example/problems/validation-error",
  "title": "Request validation failed",
  "status": 422,
  "detail": "2 fields are invalid.",
  "instance": "/v1/orders",
  "code": "VALIDATION_ERROR",
  "traceId": "4bf92f3577b34da6a3ce929d0e0e4736",
  "errors": [
    { "pointer": "/items/0/quantity", "code": "OUT_OF_RANGE", "detail": "Must be between 1 and 100." },
    { "pointer": "/shippingAddress/postalCode", "code": "INVALID_FORMAT", "detail": "Must be a 6-digit PIN code." }
  ]
}
```

```text
🔴 One error format across all APIs; documented error codes (machine-readable, stable)
🔴 Never include stack traces, SQL, internal hostnames, or secrets
🔴 Include a trace/correlation ID so support can find logs
🟠 Use JSON Pointer (RFC 6901) to identify invalid fields
🟠 Localise `title`/`detail` for end-user-facing errors, or let clients map `code` to messages
```

---

## 6. Pagination, Filtering, Sorting, Field Selection

### 6.1 Pagination styles

| Style | Request | Pros | Cons |
|---|---|---|---|
| **Cursor / keyset** ✅ default | `?limit=20&cursor=eyJjIjoi…` | Stable under inserts, fast at depth | No random page jumps |
| Offset | `?limit=20&offset=40` | Simple, page numbers | Slow at depth; duplicates/skips when data changes |
| Page number | `?page=3&pageSize=20` | UI-friendly | Same issues as offset |

```json
{
  "data": [ { "id": "ord_1" }, { "id": "ord_2" } ],
  "page": {
    "nextCursor": "eyJjcmVhdGVkQXQiOiIyMDI2LTEwLTAyVDA5OjMwOjAwWiIsImlkIjoib3JkXzIifQ",
    "hasMore": true
  }
}
```

```text
🔴 Every list endpoint is paginated; enforce a maximum limit (e.g. 100)
🔴 Cursors are opaque (base64 of a key set, optionally signed) — clients must not construct them
🟠 Deterministic ordering with a unique tiebreaker (created_at, id) — chapter 10 §8.4
🟠 Optional Link headers (RFC 8288) with rel="next"
🟡 Total counts are expensive at scale — make them optional or approximate
```

### 6.2 Filtering and sorting

```text
GET /v1/orders?status=paid,shipped&createdAfter=2026-10-01T00:00:00Z&minTotal=100000&sort=-createdAt,id
```

| Rule | Detail |
|---|---|
| Allow-list filterable and sortable fields | Prevents unindexed queries and data leaks |
| Consistent operator conventions | `createdAfter/createdBefore`, `minX/maxX`, comma-separated multi-values |
| Sort syntax | `sort=-createdAt` (descending prefix) — document it |
| Complex search | `POST /v1/orders/search` (or `QUERY`) with a JSON body |

### 6.3 Field selection and expansion

```text
GET /v1/orders/ord_7f3k?fields=id,status,total          ← sparse fieldsets
GET /v1/orders/ord_7f3k?expand=customer,items.product   ← embed related resources (limit depth)
```

### 6.4 Bulk operations

```http
POST /v1/products:batchUpdate      (or POST /v1/products/batch)
{ "requests": [ { "id": "prd_1", "price": {...} }, { "id": "prd_2", "price": {...} } ] }
→ 200 with per-item results (partial success explicit), or 202 for async processing
```

---

## 7. Idempotency, Concurrency, and Long-Running Operations

### 7.1 Idempotency keys for POST

Clients send a unique key per logical operation; servers store the key with the result and replay the same response on retries. An IETF HTTPAPI working-group draft standardises the **`Idempotency-Key`** header; many payment APIs already use this pattern.

```http
POST /v1/payments
Idempotency-Key: 5c1f0c8e-6a4f-4f8e-9b4a-1f2d3c4b5a6e
Content-Type: application/json

{ "orderId": "ord_7f3k", "method": "upi", "amount": { "amountMinor": 249900, "currency": "INR" } }
```

| Server behaviour | Response |
|---|---|
| First request | Process, store key → response (24 h TTL typical) |
| Retry, same key + same payload, completed | Return stored response |
| Retry while still processing | `409 Conflict` (or wait) |
| Same key, **different** payload | `422`/`409` — reject (fingerprint the request body) |

Implementation: chapter 05 §16.

### 7.2 Retry guidance for clients

```text
Retry only: idempotent methods (GET/PUT/DELETE) or POST with Idempotency-Key
Retry on:   connection errors, timeouts, 429 (honour Retry-After), 502, 503, 504
Never on:   400, 401, 403, 404, 409, 422 (fix the request instead)
How:        exponential backoff + full jitter, max attempts, overall deadline
```

### 7.3 Optimistic concurrency with ETags

```http
GET /v1/orders/ord_7f3k            → 200, ETag: "v7"
PATCH /v1/orders/ord_7f3k
If-Match: "v7"                      → 200, ETag: "v8"     (or 412 if someone else updated first)
```

Require `If-Match` on updates of contended resources (respond `428 Precondition Required` if missing).

### 7.4 Long-running operations

```http
POST /v1/reports/exports            → 202 Accepted
Location: /v1/operations/op_91x

GET /v1/operations/op_91x           → 200 { "status": "running", "progress": 0.4 }
GET /v1/operations/op_91x           → 200 { "status": "succeeded", "result": { "downloadUrl": "https://…", "expiresAt": "…" } }
```

Optionally notify completion via webhook or event. Never hold an HTTP request open for minutes.

---

## 8. Caching and Conditional Requests

| Header | Use |
|---|---|
| `Cache-Control: public, max-age=300` | Shared caches (CDN) for public, non-personalised data |
| `Cache-Control: private, max-age=60` | Browser-only caching for user-specific data |
| `Cache-Control: no-store` | Sensitive data (tokens, PII, payment info) |
| `ETag` + `If-None-Match` | Revalidation → `304 Not Modified` |
| `Last-Modified` + `If-Modified-Since` | Time-based revalidation |
| `Vary: Accept-Language, Authorization` | Cache key dimensions |

```text
🔴 Never let shared caches store authenticated/personalised responses
🟠 Version static and immutable resources (content hashes) and cache them for a long time (immutable)
🟠 Use CDN caching for public catalogue/content APIs with short TTLs + purge on change
```

---

## 9. Rate Limiting and Quotas

### 9.1 Algorithms

| Algorithm | Behaviour |
|---|---|
| **Token bucket** | Allows bursts up to bucket size, steady refill rate — most common |
| Leaky bucket | Smooths output to a constant rate |
| Fixed window | Simple counters per window — boundary bursts |
| Sliding window (log/counter) | Smoother fairness |
| Concurrency limits | Caps in-flight requests (protects slow backends) |

### 9.2 Policy design

```text
Dimensions:   per API key/client, per user, per tenant, per IP (unauthenticated), per endpoint (expensive ones)
Tiers:        free / standard / enterprise quotas
Responses:    429 + Retry-After + problem+json body
Headers:      expose limits so clients can self-throttle (IETF draft "RateLimit header fields for HTTP":
              RateLimit-Policy and RateLimit fields; many APIs still use X-RateLimit-Limit/Remaining/Reset)
Protection:   stricter limits on auth, signup, OTP, password reset, search, export endpoints
```

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 30
Content-Type: application/problem+json

{ "type": "https://api.shopnow.example/problems/rate-limited", "title": "Too many requests",
  "status": 429, "detail": "Limit of 100 requests per minute exceeded.", "code": "RATE_LIMITED" }
```

---

## 10. OpenAPI 3.2 and Contract-First Workflow

### 10.1 OpenAPI versions

| Version | Notes |
|---|---|
| **3.2.0** (Sept 2025) | Latest. Adds hierarchical/multipurpose tags (`parent`, `kind`), the `QUERY` method and `additionalOperations` for other HTTP methods, first-class streaming media types (e.g. Server-Sent Events, JSON Lines), security enhancements (e.g. OAuth2 metadata URL, device authorization flow), and more. Keeps 3.1 descriptions valid. |
| 3.1.x | Full JSON Schema 2020-12 alignment; widely supported |
| 3.0.x | Still common in tooling; plan migration |
| Swagger 2.0 | Legacy |

> Check that your generators, linters, gateways, and doc renderers support 3.2 before adopting it; 3.1 is the safe baseline where tooling lags.

Related OpenAPI Initiative specs: **Arazzo** (describing multi-step API workflows) and **Overlay** (applying repeatable modifications to OpenAPI documents).

### 10.2 Contract-first workflow

```text
1. Draft the OpenAPI description in Git (api/openapi.yaml) alongside a short design doc
2. Lint (Spectral with your style-guide ruleset) in CI
3. Review the PR with consumers (frontend, partners, other teams)
4. Generate mocks (Prism) → consumers start building in parallel
5. Generate server stubs / types / client SDKs (openapi-generator, oapi-codegen, openapi-typescript, Kiota, Orval…)
6. Implement; validate requests/responses against the spec in tests (and optionally at runtime)
7. Breaking-change check against main in CI (oasdiff / openapi-diff)
8. Publish docs (Redocly, Scalar, Swagger UI) and changelog
```

### 10.3 Example OpenAPI fragment

```yaml
openapi: 3.1.1          # use 3.2.0 once your toolchain supports it
info:
  title: ShopNow Orders API
  version: 1.14.0
servers:
  - url: https://api.shopnow.example/v1
paths:
  /orders/{orderId}:
    get:
      operationId: getOrder
      summary: Get an order
      tags: [Orders]
      parameters:
        - name: orderId
          in: path
          required: true
          schema: { type: string, pattern: '^ord_[a-z0-9]{10}$' }
      responses:
        '200':
          description: The order
          headers:
            ETag: { schema: { type: string } }
          content:
            application/json:
              schema: { $ref: '#/components/schemas/Order' }
        '404':
          $ref: '#/components/responses/NotFound'
      security:
        - oauth2: [orders:read]
components:
  schemas:
    Money:
      type: object
      required: [amountMinor, currency]
      properties:
        amountMinor: { type: integer, format: int64, minimum: 0 }
        currency: { type: string, pattern: '^[A-Z]{3}$' }
    Order:
      type: object
      required: [id, status, total, createdAt]
      properties:
        id: { type: string }
        status: { type: string, enum: [created, payment_pending, payment_failed, paid, shipped, cancelled, refunded] }
        total: { $ref: '#/components/schemas/Money' }
        createdAt: { type: string, format: date-time }
  responses:
    NotFound:
      description: Not found
      content:
        application/problem+json:
          schema: { $ref: '#/components/schemas/Problem' }
  securitySchemes:
    oauth2:
      type: oauth2
      flows:
        clientCredentials:
          tokenUrl: https://auth.shopnow.example/oauth2/token
          scopes:
            orders:read: Read orders
```

### 10.4 Spectral ruleset (style-guide enforcement)

```yaml
# .spectral.yaml
extends: ["spectral:oas"]
rules:
  operation-operationId: error
  operation-tags: error
  paths-kebab-case:
    description: Paths must be kebab-case
    severity: error
    given: $.paths[*]~
    then:
      function: pattern
      functionOptions:
        match: "^(/[a-z0-9-{}]+)+$"
  error-responses-problem-json:
    description: 4xx/5xx responses should use application/problem+json
    severity: warn
    given: $.paths.*.*.responses[?(@property >= '400')].content
    then:
      field: application/problem+json
      function: truthy
```

---

## 11. Versioning, Evolution, and Deprecation

### 11.1 Compatible vs breaking changes

| ✅ Non-breaking (additive) | ❌ Breaking |
|---|---|
| Add a new endpoint | Remove/rename an endpoint or field |
| Add an optional request field | Make an optional field required |
| Add a response field | Change a field's type or format |
| Add a new enum value* | Change meaning of an existing field/status code |
| Relax validation | Tighten validation |
| Add optional query parameter | Change default behaviour, pagination style, auth requirements |

`*` Only non-breaking if clients were told to tolerate unknown enum values — say so in your docs from day one.

### 11.2 Versioning strategies

| Strategy | Example | Notes |
|---|---|---|
| **URI major version** ✅ common | `/v1/orders` | Visible, cache-friendly, simple routing |
| Header / media type | `Accept: application/vnd.shopnow.v2+json` | Cleaner URIs; harder to test/debug |
| Date-based versions | `API-Version: 2026-10-01` | Fine-grained evolution (used by some large API providers) |
| Query parameter | `?version=2` | Least preferred |

```text
Rules:
- Version only on breaking changes (MAJOR); evolve additively within a version
- Support at least N-1 major versions for a published deprecation period (e.g. 12 months for partners)
- Internal APIs: prefer evolution + consumer-driven contract tests over frequent major versions
```

### 11.3 Deprecation signalling

| Mechanism | Purpose |
|---|---|
| `Deprecation` response header (RFC 9745) | Signals the resource is (or will be) deprecated |
| `Sunset` response header (RFC 8594) | Date after which the resource is expected to become unresponsive |
| `Link: <…>; rel="deprecation"` | Points to migration docs |
| `deprecated: true` in OpenAPI | Machine-readable in docs/SDKs |
| Changelog + email/portal notices | Human communication |
| Usage analytics per client | Know who still calls it; contact them |

```http
Deprecation: @1767225600
Sunset: Thu, 01 Oct 2027 00:00:00 GMT
Link: <https://developer.shopnow.example/migrations/orders-v2>; rel="deprecation"; type="text/html"
```

---

## 12. gRPC and Protocol Buffers

### 12.1 Protobuf design rules

```protobuf
syntax = "proto3";

package shopnow.orders.v1;

import "google/protobuf/timestamp.proto";

service OrderService {
  rpc GetOrder(GetOrderRequest) returns (GetOrderResponse);
  rpc ListOrders(ListOrdersRequest) returns (ListOrdersResponse);
  rpc WatchOrder(WatchOrderRequest) returns (stream OrderEvent);   // server streaming
}

message Money {
  int64 amount_minor = 1;
  string currency = 2;            // ISO 4217
}

enum OrderStatus {
  ORDER_STATUS_UNSPECIFIED = 0;   // always reserve 0 for UNSPECIFIED
  ORDER_STATUS_CREATED = 1;
  ORDER_STATUS_PAID = 2;
  ORDER_STATUS_SHIPPED = 3;
}

message Order {
  string id = 1;
  OrderStatus status = 2;
  Money total = 3;
  google.protobuf.Timestamp create_time = 4;
  reserved 5;                     // never reuse deleted field numbers
  reserved "legacy_note";
}

message GetOrderRequest { string order_id = 1; }
message GetOrderResponse { Order order = 1; }

message ListOrdersRequest {
  int32 page_size = 1;
  string page_token = 2;
  string filter = 3;
}
message ListOrdersResponse {
  repeated Order orders = 1;
  string next_page_token = 2;
}
```

| Rule | Status |
|---|:---:|
| Package names include a version (`shopnow.orders.v1`) | 🔴 |
| Never change or reuse field numbers; `reserved` removed numbers and names | 🔴 |
| Enum zero value = `*_UNSPECIFIED` | 🔴 |
| Request/response message per RPC (no reuse of domain messages as requests) | 🟠 |
| Follow Google API Improvement Proposals (AIPs) for resource-oriented design | 🟠 |
| Lint and breaking-change detection with **Buf** (`buf lint`, `buf breaking`) in CI | 🔴 |

### 12.2 gRPC operations

```text
🔴 Every call sets a DEADLINE; servers respect cancellation
🔴 Use canonical gRPC status codes (INVALID_ARGUMENT, NOT_FOUND, ALREADY_EXISTS, FAILED_PRECONDITION,
    PERMISSION_DENIED, UNAUTHENTICATED, RESOURCE_EXHAUSTED, UNAVAILABLE, DEADLINE_EXCEEDED, INTERNAL)
🔴 Rich error details via google.rpc.Status / error details (BadRequest, ErrorInfo)
🟠 Retries via service config / mesh only for UNAVAILABLE and idempotent methods
🟠 mTLS between services; auth tokens in metadata
🟠 Load balancing: gRPC uses long-lived HTTP/2 connections — use client-side LB or an L7 proxy/mesh
🟡 Browsers: gRPC-Web or Connect protocol; or expose REST via gRPC-Gateway / transcoding
```

---

## 13. GraphQL

### 13.1 Schema design

```graphql
type Query {
  order(id: ID!): Order
  orders(first: Int = 20, after: String, filter: OrderFilter): OrderConnection!
}

type Mutation {
  retryPayment(input: RetryPaymentInput!): RetryPaymentPayload!
}

type Order {
  id: ID!
  status: OrderStatus!
  total: Money!
  customer: Customer!
  createdAt: DateTime!
}

type OrderConnection {          # Relay-style cursor pagination
  edges: [OrderEdge!]!
  pageInfo: PageInfo!
}
type OrderEdge { cursor: String!  node: Order! }
type PageInfo { hasNextPage: Boolean!  endCursor: String }

input RetryPaymentInput { orderId: ID!  method: PaymentMethod!  idempotencyKey: String! }
type RetryPaymentPayload { order: Order  userErrors: [UserError!]! }
type UserError { field: [String!]  code: String!  message: String! }
```

### 13.2 Rules

```text
🔴 Authorise in resolvers / data layer per object and field — not only at the gateway
🔴 Query cost controls: max depth, complexity/cost analysis, timeouts, pagination limits
🔴 Batch data loading (DataLoader pattern) to prevent N+1
🔴 Mutations accept input objects and return payloads with userErrors (business errors in-band)
🟠 Persisted queries / operation allow-lists for first-party clients (security + caching)
🟠 Restrict introspection in production for private APIs (it's not a security boundary on its own)
🟠 Schema evolution: add fields, deprecate with @deprecated(reason: …), track field usage before removal
🟡 Federation (Apollo Federation, GraphQL Mesh, Hive, WunderGraph Cosmo) for composing subgraphs from many teams
```

---

## 14. Event-Driven Integration

### 14.1 Event types

| Type | Content | Use |
|---|---|---|
| **Event notification** | Minimal: "OrderPlaced, id=…" | Consumers call back for details; loose coupling |
| **Event-carried state transfer** | Full/needed state in the event | Consumers build local read models; no callbacks |
| **Domain event** | Business fact inside a bounded context | Internal reactions, sagas |
| **Integration event** | Published contract for other contexts/teams | Cross-service integration (stable, versioned) |
| **Command message** | Request to do something (to one handler) | Queues, work distribution |

### 14.2 CloudEvents envelope (CNCF, v1.0)

```json
{
  "specversion": "1.0",
  "type": "com.shopnow.orders.order_placed.v1",
  "source": "/services/orders",
  "id": "evt_01J9ZK7M8Q",
  "time": "2026-10-02T09:30:00Z",
  "subject": "ord_7f3kq2m9x1",
  "datacontenttype": "application/json",
  "dataschema": "https://schemas.shopnow.example/orders/order_placed/v1.json",
  "data": {
    "orderId": "ord_7f3kq2m9x1",
    "customerId": "cus_2b8n4v",
    "total": { "amountMinor": 249900, "currency": "INR" }
  }
}
```

### 14.3 AsyncAPI 3.0

Describe channels, operations, and message schemas for Kafka, AMQP, MQTT, WebSockets, etc. — the event equivalent of OpenAPI.

```yaml
asyncapi: 3.0.0
info:
  title: Orders Events
  version: 1.3.0
channels:
  orderPlaced:
    address: orders.order_placed.v1
    messages:
      orderPlaced:
        $ref: '#/components/messages/OrderPlaced'
operations:
  publishOrderPlaced:
    action: send
    channel: { $ref: '#/channels/orderPlaced' }
components:
  messages:
    OrderPlaced:
      contentType: application/json
      payload:
        type: object
        required: [orderId, customerId, total]
        properties:
          orderId: { type: string }
          customerId: { type: string }
          total:
            type: object
            properties:
              amountMinor: { type: integer }
              currency: { type: string }
```

### 14.4 Delivery and processing rules

```text
🔴 Assume AT-LEAST-ONCE delivery → idempotent consumers (dedupe by event id)
🔴 Publish reliably with the transactional outbox (or CDC) — never "write DB then publish" without it
🔴 Schema registry + compatibility rules (BACKWARD by default) for Avro/Protobuf/JSON Schema
🔴 Dead-letter queue (DLQ) with alerting and a replay procedure
🟠 Ordering only where needed: partition by entity key (e.g. orderId) for per-entity order
🟠 Retries with backoff; poison-message detection; max attempts
🟠 Event versioning: new type version (order_placed.v2) for breaking changes; dual publish during migration
🟠 Correlation and causation IDs + trace context in headers
🟡 Keep events meaningful (business facts), not CRUD noise ("row updated")
```

### 14.5 Brokers

| Broker | Model | Strengths |
|---|---|---|
| **Apache Kafka** (and compatible: Redpanda, managed MSK/Confluent/Event Hubs Kafka endpoint) | Partitioned log | High throughput, replay, stream processing |
| **RabbitMQ** | Queues/exchanges (AMQP); streams | Flexible routing, work queues |
| **NATS / JetStream** | Subjects; persistence with JetStream | Lightweight, low latency |
| **Apache Pulsar** | Segmented log + queues | Multi-tenancy, geo-replication |
| **Cloud native**: SQS/SNS/EventBridge, Google Pub/Sub, Azure Service Bus/Event Grid/Event Hubs | Managed | Low ops, cloud integration |
| Redis/Valkey Streams | Lightweight streams | Simple cases |

---

## 15. Webhooks

The **Standard Webhooks** specification (community-driven) defines consistent signing and delivery conventions (`webhook-id`, `webhook-timestamp`, `webhook-signature` headers). https://www.standardwebhooks.com/

### 15.1 Sending webhooks (provider side)

```text
🔴 Sign every payload (HMAC-SHA256 over id.timestamp.body, or asymmetric signatures); include a timestamp
🔴 HTTPS only; validate destination URLs (SSRF protections — block internal ranges)
🔴 Retries with exponential backoff over hours/days; mark endpoint disabled after sustained failure and notify
🔴 Unique event id per delivery for consumer deduplication
🟠 Small payloads (or thin events + fetch API); versioned event types
🟠 Delivery logs and manual replay in a developer dashboard
🟠 Secret rotation with overlap (support two active secrets)
```

### 15.2 Receiving webhooks (consumer side)

```text
🔴 Verify signature and timestamp (reject old timestamps to prevent replay) — chapter 09 §6.2
🔴 Respond 2xx quickly; process asynchronously via a queue
🔴 Idempotent processing (store processed event ids)
🔴 Don't trust payload blindly — for critical actions (e.g. "payment succeeded"), confirm via the provider's API
🟠 Handle out-of-order deliveries (use timestamps/versions; fetch latest state)
```

---

## 16. Real-Time APIs — WebSocket and SSE

| | **Server-Sent Events (SSE)** | **WebSocket** |
|---|---|---|
| Direction | Server → client | Bidirectional |
| Protocol | Plain HTTP (`text/event-stream`) | Upgrade to WS protocol (RFC 6455) |
| Reconnect / resume | Built-in (`Last-Event-ID`) | Implement yourself |
| Proxies/CDNs | Generally friendly | Need explicit support |
| Use | Notifications, live feeds, streaming LLM tokens, progress | Chat, collaboration, games, bidirectional control |

```text
Scaling rules:
- Authenticate on connect (short-lived token), re-validate periodically; authorise every subscription/topic
- Fan-out via a pub/sub backbone (Redis/Valkey pub/sub, NATS, Kafka) so any node can deliver
- Heartbeats/pings; idle timeouts; backpressure (drop or buffer limits for slow clients)
- Connection limits per user/IP; message size limits; rate limits
- Graceful shutdown: drain connections and let clients reconnect elsewhere
```

OpenAPI 3.2 adds first-class support for describing streaming media types such as SSE.

---

## 17. Authentication and Authorisation for APIs

| Client type | Recommended mechanism |
|---|---|
| First-party web app | BFF + secure session cookie; BFF calls APIs with tokens |
| Mobile / SPA (public clients) | OAuth 2.0 Authorization Code + **PKCE** via OIDC |
| Server-to-server (partner) | OAuth 2.0 Client Credentials (private_key_jwt or mTLS client auth preferred over shared secrets) |
| Internal service-to-service | mTLS / workload identity (SPIFFE/SPIRE, mesh) + short-lived tokens with audience |
| Developer/public APIs | API keys **only** for identification/quotas of low-risk APIs; prefer OAuth for anything sensitive |
| Webhooks | Signatures (§15) |

```text
🔴 Scopes/permissions per operation (orders:read, orders:write); least privilege
🔴 Validate token issuer, audience, expiry, signature; reject tokens meant for other APIs (audience!)
🔴 Object-level authorisation in the service (BOLA) — scopes alone are not enough
🟠 Sender-constrained tokens (DPoP / mTLS) for high-value APIs
🟠 API keys: hashed at rest, prefixed for secret scanning (e.g. sk_live_…), rotatable, scoped, expiring
```

Details: chapter 09 §8–10.

---

## 18. API Gateways and Management

### 18.1 Gateway responsibilities

| Responsibility | Notes |
|---|---|
| TLS termination, routing | Path/host-based routing to services |
| Authentication | JWT validation, OAuth introspection, API keys, mTLS |
| Rate limiting / quotas | Per client/tier |
| Request validation | Size limits, schema validation (OpenAPI-driven) |
| Transformation | Minimal — avoid putting business logic in the gateway |
| Observability | Access logs, metrics, trace propagation |
| Developer portal | Keys, docs, usage analytics (API management products) |

### 18.2 Options

| Category | Examples |
|---|---|
| Cloud-managed | AWS API Gateway, Azure API Management, Google Apigee / API Gateway |
| Open source / self-hosted | Kong, Tyk, KrakenD, Apache APISIX, Envoy Gateway, Traefik, NGINX |
| Kubernetes standard | **Gateway API** (successor to Ingress) implementations |
| Service mesh (east-west) | Istio, Linkerd, Cilium |

### 18.3 BFF (Backend for Frontend)

One API layer per client type (web, mobile, partner) that aggregates downstream services, shapes responses, and handles client-specific auth (cookies for web). Keeps general-purpose APIs clean.

---

## 19. Integrating with Third-Party APIs

### 19.1 Integration design checklist

```text
🔴 Wrap the vendor behind your own port/adapter (anti-corruption layer — chapter 04 §7)
🔴 Timeouts (connect + read), retries only when safe, circuit breaker, bulkhead
🔴 Idempotency keys on vendor calls that support them (payments!)
🔴 Credentials in a secret manager; least-privilege vendor API keys; rotation plan
🔴 Validate and sanitise vendor responses (OWASP API10: unsafe consumption)
🟠 Reconciliation jobs against vendor reports (payments, shipments, invoices)
🟠 Vendor status monitoring + synthetic checks; fallback/degradation plan
🟠 Sandbox + contract tests + recorded stubs (chapter 08 §17.2)
🟠 Track vendor API versions and deprecation notices; budget for upgrades
🟠 Data processing agreements and data residency checks for vendors receiving personal data
```

### 19.2 Example: resilient client configuration (Resilience4j, Spring Boot `application.yml`)

```yaml
resilience4j:
  timelimiter:
    instances:
      psp: { timeoutDuration: 5s }
  retry:
    instances:
      psp:
        maxAttempts: 3
        waitDuration: 200ms
        enableExponentialBackoff: true
        exponentialBackoffMultiplier: 2
        enableRandomizedWait: true
        retryExceptions: [java.io.IOException, java.util.concurrent.TimeoutException]
  circuitbreaker:
    instances:
      psp:
        slidingWindowSize: 50
        failureRateThreshold: 50
        waitDurationInOpenState: 30s
        permittedNumberOfCallsInHalfOpenState: 5
```

> Verify property names against your Resilience4j version; the structure above is illustrative.

---

## 20. Batch and File-Based Integration

| Mechanism | Use | Rules |
|---|---|---|
| **Pre-signed object storage URLs** | Large uploads/downloads between clients and services | Short expiry, size/type limits, scan uploads, private buckets |
| **SFTP** | Banks, legacy partners | Key-based auth, IP allow-lists, PGP encryption of files where required, checksums |
| **Bulk APIs** | Bulk import/export | Async job pattern (§7.4), per-record error reports |
| **Data sharing** | Analytics partners | Governed shares / exports with masking |

File exchange rules:

```text
🔴 Define file spec: encoding (UTF-8), delimiter/quoting (RFC 4180 for CSV), header row, date/number formats, schema version
🔴 Integrity: checksums (SHA-256), record counts in trailer/manifest, idempotent re-processing by file id
🔴 Reconciliation report for every batch (accepted, rejected with reasons)
🟠 Atomic delivery: write to temp name, rename when complete
🟠 Retention and secure deletion of transferred files
```

---

## 21. Workflow Orchestration

For multi-step, long-running, cross-service processes (order fulfilment, onboarding, KYC), use explicit orchestration instead of ad-hoc chains of callbacks.

| Option | Notes |
|---|---|
| **Temporal** (and similar durable execution engines) | Workflows as code with durable state, retries, timers |
| AWS Step Functions, Azure Durable Functions / Logic Apps, Google Workflows | Managed orchestration |
| Camunda / Zeebe (BPMN) | Business-process modelling with execution |
| Choreography via events | Simple flows; harder to see the whole process |

```text
Orchestration when: many steps, compensations (sagas), timeouts, human approvals, visibility needed
Choreography when:  few steps, loosely related reactions, teams own their reactions independently
```

---

## 22. Developer Experience and Documentation

| Element | Practice |
|---|---|
| **Reference docs** | Generated from OpenAPI/AsyncAPI/proto (Redocly, Scalar, Swagger UI, Buf Schema Registry) |
| **Guides** | Quick start (< 10 minutes to first successful call), authentication, pagination, errors, webhooks, idempotency |
| **Examples** | Realistic request/response examples for every operation; copy-paste `curl` and SDK snippets |
| **SDKs** | Generated (openapi-generator, Kiota, Speakeasy, Fern, Stainless) + hand-tuned ergonomics; versioned with SemVer |
| **Sandbox** | Test mode with test keys and deterministic test data (e.g. card numbers that always decline) |
| **Changelog** | Every change dated; breaking changes flagged with migration guides |
| **Status page** | Uptime and incident communication |
| **Support** | Clear contact; error codes searchable in docs |
| **Developer portal** | Keys, usage, logs, webhook delivery logs |

```bash
# Every doc page should have a runnable example like this
curl -sS https://api.shopnow.example/v1/orders/ord_7f3kq2m9x1 \
  -H "Authorization: Bearer $SHOPNOW_TOKEN" \
  -H "Accept: application/json"
```

---

## 23. API Observability

| Signal | What to capture |
|---|---|
| **RED metrics** per route | Rate, Errors (by status class/code), Duration (p50/p95/p99) |
| Per-consumer metrics | Requests, errors, latency per client/API key/tenant — find who is affected |
| **Traces** | W3C Trace Context (`traceparent`) propagated through gateways, services, messaging headers |
| Logs | Structured access logs: method, route template (not raw path with IDs), status, latency, client id, trace id |
| SLOs | Per critical endpoint/journey (availability, latency) |
| Business metrics | Orders created, payments succeeded — catch "200 OK but broken" |
| Deprecation telemetry | Calls to deprecated endpoints/versions by client |

Use **route templates** (`/v1/orders/{orderId}`) as metric labels — never raw paths or user IDs (cardinality explosion). Details: chapter **18**.

---

## 24. API Governance

```text
1. API style guide (this chapter + org specifics) published and versioned
2. Automated linting (Spectral for OpenAPI/AsyncAPI, Buf for protobuf) in every API repo
3. Design review for new APIs and breaking changes (lightweight, async-first)
4. API catalogue / registry: every API with owner, lifecycle stage, version, SLO, data classification, docs link
5. Breaking-change detection in CI (oasdiff/openapi-diff, buf breaking, schema-registry compatibility)
6. Lifecycle stages: design → alpha/beta → GA → deprecated → retired; documented support policy
7. Security review gate for public APIs and new data exposure
8. Usage analytics to guide evolution and deprecation
```

Reference style guides worth borrowing from: Google AIPs, Microsoft REST API Guidelines, Zalando RESTful API Guidelines, Adidas API Guidelines.

---

## 25. AI and Agent Integrations

AI agents increasingly consume APIs on behalf of users.

| Topic | Guidance |
|---|---|
| **Machine-readable contracts** | High-quality OpenAPI descriptions (clear summaries, examples, error docs) make APIs usable by agents and tool-calling LLMs |
| **Model Context Protocol (MCP)** | An open protocol for connecting AI assistants to tools and data sources; exposing selected capabilities as an MCP server lets agents use them with explicit tool schemas |
| **Least-privilege access** | Agents act with the end user's delegated permissions (OAuth scopes), not service-wide credentials |
| **Confirmation for side effects** | Destructive or financial operations require explicit user confirmation outside the model's control |
| **Rate limits and quotas** | Agents can generate high request volumes — per-agent/per-user limits |
| **Idempotency** | Agents retry; idempotency keys prevent duplicate side effects |
| **Auditability** | Log which agent/client acted on behalf of which user |
| **Prompt-injection awareness** | API responses consumed by agents may contain attacker-controlled text; downstream agents must treat it as data (chapter 09 §7) |

---

## 26. Checklists

### New API (design review)
- [ ] Integration style justified (§2)
- [ ] Contract in Git (OpenAPI 3.1/3.2, AsyncAPI 3.0, or proto) and linted
- [ ] Resource naming, field naming, date/money formats per style guide
- [ ] Consistent status codes; RFC 9457 errors with stable codes
- [ ] Pagination (cursor) on every list; max limits; allow-listed filters/sorts
- [ ] Idempotency keys for non-idempotent operations clients may retry
- [ ] Optimistic concurrency (ETag/If-Match) for contended updates
- [ ] AuthN mechanism per client type; scopes; object-level authorisation
- [ ] Rate limits and quotas defined; 429 with Retry-After
- [ ] Caching headers correct; `no-store` on sensitive responses
- [ ] Versioning and deprecation policy stated; unknown enum values tolerated by clients
- [ ] Observability: traces, RED metrics by route template, per-consumer metrics, SLOs
- [ ] Docs, examples, sandbox, changelog
- [ ] Contract tests / breaking-change checks in CI

### Event / webhook integration
- [ ] AsyncAPI description; CloudEvents envelope; schema registered with compatibility rule
- [ ] Outbox/CDC for reliable publish
- [ ] Idempotent consumers; DLQ + alerting + replay runbook
- [ ] Ordering requirements met via partition keys
- [ ] Webhooks signed with timestamps; retries with backoff; delivery logs; secret rotation

### Third-party integration
- [ ] Adapter/ACL around vendor; timeouts, retries, circuit breaker
- [ ] Vendor idempotency used; reconciliation job in place
- [ ] Secrets managed; least-privilege keys; DPA/residency checked
- [ ] Sandbox tests + stubs; vendor status monitoring; fallback plan

---

## 27. References

### HTTP and API standards
- RFC 9110 HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
- RFC 9457 Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457
- RFC 8594 Sunset header: https://www.rfc-editor.org/rfc/rfc8594
- RFC 9745 Deprecation header: https://www.rfc-editor.org/rfc/rfc9745
- RFC 7396 JSON Merge Patch / RFC 6902 JSON Patch / RFC 6901 JSON Pointer: https://www.rfc-editor.org/
- RFC 8288 Web Linking: https://www.rfc-editor.org/rfc/rfc8288
- IETF HTTPAPI WG (Idempotency-Key, RateLimit headers drafts): https://datatracker.ietf.org/wg/httpapi/documents/
- IETF HTTP WG (QUERY method draft): https://datatracker.ietf.org/wg/httpbis/documents/
- W3C Trace Context: https://www.w3.org/TR/trace-context/

### Specifications
- OpenAPI Specification 3.2.0 release: https://github.com/OAI/OpenAPI-Specification/releases/tag/3.2.0
- OpenAPI specs (all versions): https://spec.openapis.org/
- Arazzo & Overlay: https://www.openapis.org/
- AsyncAPI 3.0: https://www.asyncapi.com/docs/reference/specification/v3.0.0
- CloudEvents: https://cloudevents.io/
- Standard Webhooks: https://www.standardwebhooks.com/
- gRPC: https://grpc.io/docs/ · Protocol Buffers: https://protobuf.dev/ · Buf: https://buf.build/docs/
- GraphQL specification & best practices: https://graphql.org/learn/best-practices/
- Model Context Protocol: https://modelcontextprotocol.io/

### Style guides
- Google API Improvement Proposals (AIPs): https://google.aip.dev/
- Microsoft REST API Guidelines: https://github.com/microsoft/api-guidelines
- Zalando RESTful API Guidelines: https://opensource.zalando.com/restful-api-guidelines/

### Tools
- Spectral: https://docs.stoplight.io/docs/spectral · Prism: https://docs.stoplight.io/docs/prism
- oasdiff: https://www.oasdiff.com/
- openapi-generator: https://openapi-generator.tech/ · Kiota: https://learn.microsoft.com/openapi/kiota/
- Redocly: https://redocly.com/docs/ · Scalar: https://scalar.com/
- Pact: https://docs.pact.io/
- Temporal: https://docs.temporal.io/
- Kubernetes Gateway API: https://gateway-api.sigs.k8s.io/

### Security
- OWASP API Security Top 10: https://owasp.org/API-Security/
- RFC 9700 OAuth 2.0 Security BCP: https://www.rfc-editor.org/rfc/rfc9700

---

**Previous:** [10 — Database Engineering](./10-database-engineering.md) · **Next:** [12 — Frontend Engineering](./12-frontend-engineering.md)