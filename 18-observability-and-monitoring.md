# 🔭 Observability & Monitoring — Production Engineering Guide

> How to understand what production systems are doing and detect problems before users do: monitoring vs observability, telemetry signals (metrics, logs, traces, profiles), **OpenTelemetry** architecture and the Collector, metric design and cardinality, Prometheus (incl. native OTLP ingestion in Prometheus 3), structured logging, distributed tracing and sampling, continuous profiling (OpenTelemetry Profiles alpha), SLIs/SLOs/error budgets with multi-window burn-rate alerts, alert design and routing, dashboards, synthetic and real-user monitoring, Kubernetes/infra/database monitoring, business and AI/LLM observability, stack options, cost control, and data governance.
>
> Related: [01 SLI/SLO basics §15](./01-engineering-foundations.md#15-reliability-fundamentals) · [04 Observability architecture §17](./04-software-architecture.md#17-observability-architecture) · [05 Logging design §19](./05-software-design.md#19-logging-and-telemetry-design) · [13 Backend observability §15](./13-backend-engineering.md#15-observability) · [19 Performance](./19-performance-and-scalability.md) · [20 Reliability](./20-reliability-and-disaster-recovery.md) · [21 Production operations (on-call, incidents)](./21-production-operations.md) · [39 VPS Prometheus/Grafana/Loki setup](./39-vps-enterprise-setup-guide.md)

---

## 📚 Table of Contents

- [🔭 Observability \& Monitoring — Production Engineering Guide](#-observability--monitoring--production-engineering-guide)
  - [📚 Table of Contents](#-table-of-contents)
  - [1. Monitoring vs Observability](#1-monitoring-vs-observability)
  - [2. Telemetry Signals](#2-telemetry-signals)
  - [3. OpenTelemetry Architecture](#3-opentelemetry-architecture)
    - [Minimum resource attributes](#minimum-resource-attributes)
  - [4. The OpenTelemetry Collector](#4-the-opentelemetry-collector)
    - [4.1 Deployment patterns](#41-deployment-patterns)
    - [4.2 Gateway configuration example](#42-gateway-configuration-example)
  - [5. Metrics](#5-metrics)
    - [5.1 Metric types](#51-metric-types)
    - [5.2 Which metrics — standard methods](#52-which-metrics--standard-methods)
    - [5.3 Naming and labels](#53-naming-and-labels)
    - [5.4 Cardinality — the #1 metrics cost and reliability problem](#54-cardinality--the-1-metrics-cost-and-reliability-problem)
  - [6. Prometheus and PromQL](#6-prometheus-and-promql)
    - [6.1 Useful PromQL](#61-useful-promql)
    - [6.2 Recording rules](#62-recording-rules)
  - [7. Logs](#7-logs)
    - [7.1 Structured logging standard](#71-structured-logging-standard)
    - [7.2 Log pipeline](#72-log-pipeline)
    - [7.3 Audit logs vs application logs](#73-audit-logs-vs-application-logs)
  - [8. Distributed Tracing](#8-distributed-tracing)
    - [8.1 Concepts](#81-concepts)
    - [8.2 Instrumentation guidance](#82-instrumentation-guidance)
    - [8.3 Sampling](#83-sampling)
    - [8.4 Backends](#84-backends)
  - [9. Continuous Profiling](#9-continuous-profiling)
  - [10. SLIs, SLOs, and Error Budgets](#10-slis-slos-and-error-budgets)
    - [10.1 Definitions (Google SRE)](#101-definitions-google-sre)
    - [10.2 Choosing SLIs](#102-choosing-slis)
    - [10.3 SLO document template](#103-slo-document-template)
    - [10.4 SLO tooling](#104-slo-tooling)
  - [11. Burn-Rate Alerting](#11-burn-rate-alerting)
  - [12. Alert Design and Routing](#12-alert-design-and-routing)
    - [12.1 Principles](#121-principles)
    - [12.2 Alert quality review (monthly)](#122-alert-quality-review-monthly)
    - [12.3 Alertmanager routing (excerpt)](#123-alertmanager-routing-excerpt)
  - [13. Dashboards](#13-dashboards)
    - [13.1 Dashboard hierarchy](#131-dashboard-hierarchy)
    - [13.2 Design rules](#132-design-rules)
  - [14. Synthetic Monitoring and Real-User Monitoring](#14-synthetic-monitoring-and-real-user-monitoring)
  - [15. Infrastructure and Kubernetes Monitoring](#15-infrastructure-and-kubernetes-monitoring)
    - [Essential infrastructure alerts (ticket or page based on impact)](#essential-infrastructure-alerts-ticket-or-page-based-on-impact)
  - [16. Dependency and Database Monitoring](#16-dependency-and-database-monitoring)
  - [17. Business Observability](#17-business-observability)
  - [18. Observability for AI / LLM Applications](#18-observability-for-ai--llm-applications)
  - [19. Observability Stacks](#19-observability-stacks)
  - [20. Cost Control](#20-cost-control)
  - [21. Data Governance — Retention, PII, Access](#21-data-governance--retention-pii-access)
  - [22. Observability Maturity Model](#22-observability-maturity-model)
  - [23. Checklists](#23-checklists)
    - [New service (observability readiness)](#new-service-observability-readiness)
    - [Platform](#platform)
    - [Monthly](#monthly)
  - [24. References](#24-references)
    - [Standards and specs](#standards-and-specs)
    - [SRE practice](#sre-practice)
    - [Tools](#tools)
    - [Books](#books)

---

## 1. Monitoring vs Observability

| | Monitoring | Observability |
|---|---|---|
| Question | "Is the known thing broken?" | "Why is it behaving this way — even for a failure we never anticipated?" |
| Approach | Predefined checks, dashboards, thresholds | Rich, correlated, high-context telemetry you can slice arbitrarily |
| Good at | Known failure modes | Unknown unknowns in distributed systems |
| Needs | Metrics + alerts | Metrics + logs + traces (+ profiles) with shared context (trace IDs, resource attributes) |

You need both: monitoring tells you **that** something is wrong (ideally from the user's perspective), observability tells you **why**.

```text
Detect (SLO alert) → Triage (dashboard: which service/region/version?) → Investigate (traces/logs/profiles)
→ Mitigate (rollback/flag/scale) → Learn (postmortem → better telemetry and alerts)
```

---

## 2. Telemetry Signals

| Signal | What it is | Strength | Weakness |
|---|---|---|---|
| **Metrics** | Numeric time series aggregated over time | Cheap at scale, great for alerting and trends | Lose individual request detail; cardinality limits |
| **Logs** | Timestamped event records | Rich detail, flexible | Expensive at volume; hard to correlate without IDs |
| **Traces** | Causally linked spans across services for one request | Pinpoint latency and failures across hops | Sampling needed at scale |
| **Profiles** | Sampled stack traces of CPU/memory/allocation use | Find *which code* consumes resources in production | Newer standardisation |
| **Events** (deploys, flag changes, incidents) | Discrete change markers | Explain sudden changes | Must be emitted deliberately |

**Correlation is the multiplier**: the same `trace_id`, `service.name`, `service.version`, `deployment.environment`, and `k8s.pod.name` across all signals lets you jump from an alert → dashboard → trace → logs → profile.

---

## 3. OpenTelemetry Architecture

**OpenTelemetry (OTel)** is the CNCF standard for generating, collecting, and exporting telemetry. It is vendor-neutral: instrument once, send anywhere.

```text
┌────────────────────────── Application ──────────────────────────┐
│ OTel API (instrument code)  ← libraries/frameworks auto-instrumented │
│ OTel SDK (sampling, processors, resource attributes, exporters)      │
└───────────────────────────────┬──────────────────────────────────┘
                                │ OTLP (gRPC 4317 / HTTP 4318)
                ┌───────────────▼────────────────┐
                │ OTel Collector (agent/sidecar  │  receivers → processors → exporters
                │ or DaemonSet on each node)     │
                └───────────────┬────────────────┘
                                │ OTLP
                ┌───────────────▼────────────────┐
                │ OTel Collector (gateway tier)  │  tail sampling, redaction, routing, batching
                └──┬───────────┬───────────┬─────┘
                   ▼           ▼           ▼
               Metrics       Traces       Logs / Profiles backends
```

| Component | Role |
|---|---|
| **API** | Stable interfaces for creating spans, metrics, logs |
| **SDK** | Implementation: sampling, batching, exporting |
| **Instrumentation libraries / auto-instrumentation** | HTTP servers/clients, DB drivers, messaging, frameworks — zero-code agents for Java, .NET, Python, Node.js; eBPF-based auto-instrumentation options for Go and others |
| **OTLP** | Wire protocol for all signals |
| **Semantic conventions** | Standard attribute names (`http.request.method`, `http.response.status_code`, `db.system.name`, `service.name`…) so tools understand your data |
| **Resource** | Attributes describing the source (`service.name`, `service.version`, `deployment.environment.name`, cloud/k8s attributes) |
| **Collector** | Vendor-neutral pipeline (§4) |

### Minimum resource attributes

```bash
OTEL_SERVICE_NAME=orders
OTEL_RESOURCE_ATTRIBUTES=service.version=2.5.0,deployment.environment.name=production,service.namespace=shopnow
OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector.observability:4318
OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
OTEL_TRACES_SAMPLER=parentbased_traceidratio
OTEL_TRACES_SAMPLER_ARG=0.2
```

> Semantic convention names evolve (some attributes were renamed as conventions stabilised). Pin instrumentation versions and check the conventions version they emit.

---

## 4. The OpenTelemetry Collector

### 4.1 Deployment patterns

| Pattern | Use |
|---|---|
| **Agent** (DaemonSet / sidecar / host service) | Receive from local apps, scrape node metrics, collect container logs, add host/k8s metadata |
| **Gateway** (Deployment, horizontally scaled) | Central processing: tail sampling, redaction, routing to multiple backends, auth to vendors |
| Distributions | `otelcol` (core), `otelcol-contrib` (many components), vendor distributions, or build your own with the Collector Builder (`ocb`) to include only needed components |

### 4.2 Gateway configuration example

```yaml
receivers:
  otlp:
    protocols:
      grpc: { endpoint: 0.0.0.0:4317 }
      http: { endpoint: 0.0.0.0:4318 }

processors:
  memory_limiter: { check_interval: 1s, limit_percentage: 80, spike_limit_percentage: 20 }
  k8sattributes: {}                         # enrich with pod/namespace/node metadata (agent tier usually)
  resource:
    attributes:
      - { key: deployment.environment.name, value: production, action: upsert }
  attributes/redact:
    actions:
      - { key: http.request.header.authorization, action: delete }
      - { key: user.email, action: hash }
  tail_sampling:
    decision_wait: 10s
    policies:
      - { name: errors, type: status_code, status_code: { status_codes: [ERROR] } }
      - { name: slow, type: latency, latency: { threshold_ms: 1000 } }
      - { name: baseline, type: probabilistic, probabilistic: { sampling_percentage: 10 } }
  batch: {}

exporters:
  otlphttp/metrics: { endpoint: http://prometheus:9090/api/v1/otlp }   # Prometheus 3 native OTLP receiver (must be enabled)
  otlp/traces: { endpoint: tempo:4317, tls: { insecure: true } }         # internal network example only
  otlphttp/logs: { endpoint: http://loki:3100/otlp }

service:
  pipelines:
    traces:  { receivers: [otlp], processors: [memory_limiter, attributes/redact, tail_sampling, batch], exporters: [otlp/traces] }
    metrics: { receivers: [otlp], processors: [memory_limiter, resource, batch], exporters: [otlphttp/metrics] }
    logs:    { receivers: [otlp], processors: [memory_limiter, attributes/redact, batch], exporters: [otlphttp/logs] }
```

> Endpoint paths and flags depend on backend versions (e.g. Prometheus requires enabling its OTLP receiver; Loki's native OTLP ingestion path). Verify against each backend's docs. Use TLS and authentication between components in production.

```text
🔴 memory_limiter first, batch last in every pipeline
🔴 Collectors monitored themselves (their own metrics: dropped/refused data, queue sizes)
🟠 Tail sampling needs all spans of a trace on the same gateway instance — use a load-balancing exporter tier keyed by trace ID
🟠 Redact PII and secrets centrally as defense in depth (apps should not emit them in the first place)
```

---

## 5. Metrics

### 5.1 Metric types

| Type | Use | Example |
|---|---|---|
| **Counter** (monotonic) | Counts of events | Requests, errors, bytes sent |
| **UpDownCounter / Gauge** | Values that go up and down | Queue depth, active connections, temperature |
| **Histogram** | Distributions (latency, sizes) → percentiles | Request duration |
| **Exponential / native histograms** | High-resolution histograms with automatic buckets | Supported by OTel and Prometheus native histograms — better percentile accuracy with fewer series |

> Never compute percentiles by averaging percentiles across instances. Aggregate histograms, then compute quantiles.

### 5.2 Which metrics — standard methods

| Method | Applies to | Metrics |
|---|---|---|
| **RED** (Tom Wilkie) | Request-driven services | **R**ate, **E**rrors, **D**uration |
| **USE** (Brendan Gregg) | Resources (CPU, memory, disks, pools, queues) | **U**tilisation, **S**aturation, **E**rrors |
| **Four Golden Signals** (Google SRE) | User-facing systems | Latency, traffic, errors, saturation |

### 5.3 Naming and labels

```text
- Follow OTel semantic conventions (e.g. http.server.request.duration in seconds) or Prometheus conventions
  (snake_case, base units, _total suffix for counters, _seconds/_bytes units)
- Units: seconds and bytes (base units), not ms/MB in names
- Labels (attributes): low-cardinality dimensions only — service, route TEMPLATE, method, status class/code, region, version
```

### 5.4 Cardinality — the #1 metrics cost and reliability problem

```text
Series count ≈ product of distinct values of every label.
route (50) × method (4) × status (10) × pod (60) × version (2) = 480 000 series for one histogram (× bucket count!)

🔴 Never use unbounded values as labels: user IDs, emails, order IDs, raw URLs/paths, trace IDs, timestamps, error messages
🔴 Use route templates (/v1/orders/{id}), not raw paths
🟠 Drop or aggregate high-cardinality labels in the Collector/relabelling
🟠 Monitor active series per service; set limits and alerts
🟠 Use exemplars to link a metric data point to a trace instead of adding trace IDs as labels
```

---

## 6. Prometheus and PromQL

**Prometheus** is the de-facto open-source metrics system (CNCF graduated). **Prometheus 3.0** (released late 2024) added native **OTLP ingestion**, UTF-8 metric/label names (aligning with OTel naming), native histograms improvements, and a new UI. Scalable/long-term storage: Thanos, Cortex, **Grafana Mimir**, VictoriaMetrics, or managed Prometheus services.

### 6.1 Useful PromQL

```promql
# Request rate per route (RED: Rate)
sum by (http_route) (rate(http_server_request_duration_seconds_count{service_name="orders"}[5m]))

# Error ratio (RED: Errors) — 5xx share
sum(rate(http_server_request_duration_seconds_count{service_name="orders", http_response_status_code=~"5.."}[5m]))
/
sum(rate(http_server_request_duration_seconds_count{service_name="orders"}[5m]))

# p99 latency from classic histogram buckets (RED: Duration)
histogram_quantile(0.99,
  sum by (le, http_route) (rate(http_server_request_duration_seconds_bucket{service_name="orders"}[5m])))

# CPU saturation: throttling ratio per container
sum by (pod) (rate(container_cpu_cfs_throttled_periods_total[5m]))
/ sum by (pod) (rate(container_cpu_cfs_periods_total[5m]))

# Pods restarting
increase(kube_pod_container_status_restarts_total[1h]) > 3
```

> Exact metric and label names depend on your instrumentation and on how OTLP names are translated (Prometheus 3 offers translation strategies for OTLP metric names). Check names in your metrics explorer before writing alerts.

### 6.2 Recording rules

Precompute expensive or frequently used expressions (SLO ratios, aggregated rates) to speed up dashboards and alerts.

```yaml
groups:
  - name: orders-sli
    interval: 30s
    rules:
      - record: sli:orders_requests:rate5m
        expr: sum(rate(http_server_request_duration_seconds_count{service_name="orders"}[5m]))
      - record: sli:orders_errors:rate5m
        expr: sum(rate(http_server_request_duration_seconds_count{service_name="orders", http_response_status_code=~"5.."}[5m]))
      - record: sli:orders_error_ratio:rate5m
        expr: sli:orders_errors:rate5m / sli:orders_requests:rate5m
```

---

## 7. Logs

### 7.1 Structured logging standard

```json
{
  "timestamp": "2026-10-02T09:30:12.481Z",
  "severity_text": "ERROR",
  "body": "payment charge failed",
  "service.name": "orders",
  "service.version": "2.5.0",
  "deployment.environment.name": "production",
  "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736",
  "span_id": "00f067aa0ba902b7",
  "tenant.id": "t_19",
  "order.id": "ord_7f3kq2m9x1",
  "error.type": "PspTimeout",
  "psp.attempt": 2
}
```

| Rule | Status |
|---|:---:|
| JSON (or OTel log records) with consistent field names | 🔴 |
| `trace_id`/`span_id` injected automatically by logging integrations | 🔴 |
| Levels used consistently (ERROR = needs action; WARN = degraded but handled; INFO = significant business/lifecycle events; DEBUG off in prod by default) | 🔴 |
| No secrets, tokens, passwords, full card numbers, OTPs; minimise PII (chapter 09 §15.4) | 🔴 |
| Log the **why** with context (IDs, counts, durations), not just "error occurred" | 🟠 |
| Avoid logging in hot loops; sample repetitive logs | 🟠 |
| Write to stdout/stderr in containers; let the platform ship logs | 🟠 |

### 7.2 Log pipeline

```text
App → stdout (JSON) → node agent (OTel Collector filelog receiver / Fluent Bit / Vector) → processing (parse, enrich k8s metadata,
redact, drop noise, route) → storage (Loki, Elasticsearch/OpenSearch, ClickHouse-based stores, cloud logging, vendors)
→ query, alerts on log patterns (sparingly), archive to object storage
```

| Backend | Model | Notes |
|---|---|---|
| **Grafana Loki** | Index labels only, store compressed chunks | Cheap at scale; keep labels low-cardinality; query with LogQL |
| **Elasticsearch / OpenSearch** | Full-text indexing | Powerful search; higher cost; index lifecycle management |
| ClickHouse-based (SigNoz, ClickStack, etc.) | Columnar | Fast analytics over logs/traces |
| Cloud logging | Managed | Integrated with cloud services; watch ingestion costs |

### 7.3 Audit logs vs application logs

Audit logs (who did what) have different requirements — immutability, longer retention, restricted access, legal hold — so route them to a separate, protected store (chapter 09 §21).

---

## 8. Distributed Tracing

### 8.1 Concepts

| Term | Meaning |
|---|---|
| **Trace** | The full journey of a request across services |
| **Span** | One operation (HTTP call, DB query, message processing) with start/end, attributes, events, status |
| **Context propagation** | Passing trace context across process boundaries — **W3C Trace Context** (`traceparent`, `tracestate`) over HTTP/gRPC/messaging headers |
| **Baggage** | Key-values propagated with the trace (use sparingly; never sensitive data — it leaks to downstream services) |
| **Span links** | Relate spans across traces (batch processing, fan-in from queues) |

```text
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
             ver  trace-id (16 bytes hex)          parent span id   flags (sampled)
```

### 8.2 Instrumentation guidance

```text
🔴 Auto-instrument inbound/outbound HTTP, gRPC, DB, cache, and messaging
🔴 Propagate context through queues/events (inject into message headers; consumers extract and link)
🟠 Add manual spans for important business steps (e.g. "price.calculate", "payment.charge") with meaningful attributes
🟠 Record errors on spans (status=ERROR, exception events) — but don't duplicate huge payloads
🟠 Name spans by operation template, not by IDs ("GET /v1/orders/{id}", not the raw URL)
```

```python
from opentelemetry import trace
tracer = trace.get_tracer("orders.checkout")

def charge(order):
    with tracer.start_as_current_span("payment.charge") as span:
        span.set_attribute("order.id", order.id)          # high-cardinality is fine on spans, not on metric labels
        span.set_attribute("payment.method", order.method)
        try:
            return psp.charge(order)
        except PspTimeout as e:
            span.record_exception(e)
            span.set_status(trace.Status(trace.StatusCode.ERROR))
            raise
```

### 8.3 Sampling

| Strategy | Where decided | Pros | Cons |
|---|---|---|---|
| **Head sampling** (e.g. 10% by trace ID ratio, parent-based) | At trace start (SDK) | Cheap, simple | May miss rare errors/slow requests |
| **Tail sampling** | After the trace completes (Collector gateway) | Keep all errors and slow traces + a baseline sample | Needs buffering and trace-aware load balancing |
| Rate limiting | SDK/Collector | Caps cost | Biased under spikes |
| Always-on for low-traffic critical paths | — | Full visibility | Cost |

Recommended: parent-based head sampling at a moderate ratio in SDKs + tail sampling at the gateway keeping errors, slow traces, and specific high-value routes.

### 8.4 Backends

Grafana **Tempo**, **Jaeger** (CNCF graduated), Zipkin, Elastic APM, SigNoz, cloud tracing (X-Ray, Cloud Trace, Application Insights), and commercial APM vendors — all accepting OTLP.

---

## 9. Continuous Profiling

Continuous profiling samples stack traces in production at low overhead to show which functions consume CPU, memory, allocations, locks, or wall time.

- The **OpenTelemetry Profiles** signal reached **public alpha in March 2026**, joining traces, metrics, and logs as a core OTel signal. It defines an OTLP profiles format that can round-trip with pprof and links samples to traces via `trace_id`/`span_id`. Being alpha, expect changes before production-grade stability — many teams still use dedicated profilers today.
- eBPF-based profilers can profile without code changes.

| Tool | Notes |
|---|---|
| Grafana **Pyroscope** | Open-source continuous profiling; Grafana integration |
| Parca / Polar Signals | eBPF-based continuous profiling |
| OTel eBPF profiler (donated by Elastic) | Feeds the OTel profiles signal |
| Language profilers | JFR + async-profiler (JVM), pprof (Go), py-spy (Python), `--cpu-prof`/0x (Node.js), dotnet-trace (.NET) |
| Cloud profilers | Cloud Profiler (GCP), CodeGuru Profiler (AWS), Application Insights Profiler (Azure) |

Use cases: "p99 latency regressed after deploy — which function?", "why did memory double?", "where is CPU spent at peak load?" (chapter 19).

---

## 10. SLIs, SLOs, and Error Budgets

### 10.1 Definitions (Google SRE)

| Term | Meaning |
|---|---|
| **SLI** | A carefully defined measurement of service level, usually a ratio of good events to valid events |
| **SLO** | Target for an SLI over a window (e.g. 99.9% over 28 days) |
| **Error budget** | 1 − SLO = allowed bad events; spent by incidents and risky changes |
| **SLA** | External contract with consequences; looser than the internal SLO |

### 10.2 Choosing SLIs

| User journey / service type | SLI (good / valid) |
|---|---|
| Request/response API | **Availability**: non-5xx responses / all valid requests; **Latency**: requests faster than threshold (e.g. 300 ms) / all valid requests |
| Checkout journey | Successful checkouts / attempted checkouts (excluding user errors such as declined cards) |
| Data pipeline | **Freshness**: records processed within X minutes; **Correctness**: valid records / all records |
| Batch job | Jobs completed successfully by deadline / scheduled jobs |
| Streaming | Messages processed within X seconds / all messages |
| Frontend | Page loads with LCP ≤ 2.5 s / page loads (chapter 12 §13) |

```text
Measure SLIs as close to the user as practical: load balancer/gateway or client-side RUM > service > host.
Exclude invalid events explicitly (health checks, synthetic traffic, client errors caused by user input).
```

### 10.3 SLO document template

```markdown
# SLO: Orders API — availability & latency
Owner: orders team · Reviewed: quarterly · Version: 3

## SLIs
- Availability: proportion of valid requests to /v1/orders/* that return non-5xx, measured at the API gateway
- Latency: proportion of valid requests completing in ≤ 400 ms, measured at the API gateway

## Targets (rolling 28 days)
- Availability: 99.9%   → error budget: 0.1% (~40 min of full outage equivalent per 28 days)
- Latency:      99%     (≤ 400 ms)

## Error budget policy
- Budget remaining > 25%: normal feature work
- Budget remaining ≤ 25%: risky releases require extra review; reliability work prioritised
- Budget exhausted: feature freeze for this service except reliability fixes, until budget recovers (exceptions approved by eng director)

## Alerting
Multi-window, multi-burn-rate alerts (fast burn pages, slow burn tickets) — see dashboards/alerts links
```

### 10.4 SLO tooling

OpenSLO (vendor-neutral spec), **Sloth** and **Pyrra** (generate Prometheus recording/alerting rules from SLO definitions), Grafana SLO, Nobl9, Datadog/Dynatrace/New Relic SLO features, cloud-native SLO monitoring.

---

## 11. Burn-Rate Alerting

**Burn rate** = how fast the error budget is consumed relative to the SLO window (burn rate 1 consumes the whole budget exactly at the end of the window).

The Google SRE Workbook recommends **multi-window, multi-burn-rate** alerts — a long window to ensure significance plus a short window to ensure the problem is still happening. Commonly used starting parameters for a 30-day SLO window:

| Severity | Burn rate | Long window | Short window | Budget consumed when it fires |
|---|---:|---|---|---:|
| Page | 14.4 | 1 h | 5 min | 2% |
| Page | 6 | 6 h | 30 min | 5% |
| Ticket | 1 | 3 days | 6 h | 10% |

```yaml
# Prometheus alerting rules for a 99.9% availability SLO (error budget = 0.001)
groups:
  - name: orders-slo-burn
    rules:
      - alert: OrdersErrorBudgetFastBurn
        expr: |
          (
            sum(rate(http_server_request_duration_seconds_count{service_name="orders",http_response_status_code=~"5.."}[1h]))
            / sum(rate(http_server_request_duration_seconds_count{service_name="orders"}[1h]))
          ) > (14.4 * 0.001)
          and
          (
            sum(rate(http_server_request_duration_seconds_count{service_name="orders",http_response_status_code=~"5.."}[5m]))
            / sum(rate(http_server_request_duration_seconds_count{service_name="orders"}[5m]))
          ) > (14.4 * 0.001)
        labels: { severity: page, team: orders }
        annotations:
          summary: "Orders API burning error budget fast (14.4x)"
          runbook_url: "https://runbooks.shopnow.example/orders/high-error-rate"
          dashboard: "https://grafana.shopnow.example/d/orders-slo"
      - alert: OrdersErrorBudgetSlowBurn
        expr: |
          (
            sum(rate(http_server_request_duration_seconds_count{service_name="orders",http_response_status_code=~"5.."}[6h]))
            / sum(rate(http_server_request_duration_seconds_count{service_name="orders"}[6h]))
          ) > (6 * 0.001)
          and
          (
            sum(rate(http_server_request_duration_seconds_count{service_name="orders",http_response_status_code=~"5.."}[30m]))
            / sum(rate(http_server_request_duration_seconds_count{service_name="orders"}[30m]))
          ) > (6 * 0.001)
        labels: { severity: page, team: orders }
        annotations:
          summary: "Orders API burning error budget (6x)"
          runbook_url: "https://runbooks.shopnow.example/orders/high-error-rate"
```

Low-traffic services: burn-rate alerts become noisy — use longer windows, minimum request counts, or synthetic traffic to stabilise the signal.

---

## 12. Alert Design and Routing

### 12.1 Principles

| Principle | Practice |
|---|---|
| **Alert on symptoms, not causes** | Page on user-visible SLO burn; use cause-based signals (CPU, disk) for tickets/dashboards unless they predict imminent failure (disk full in < 4 h) |
| **Every page is actionable** | If no human action is needed, it's not a page |
| **Every alert has a runbook** | Link in annotations: what it means, how to verify, how to mitigate, escalation |
| **Severity is consistent** | Page (now, 24/7), ticket (business hours), info (dashboard only) |
| **Low noise** | Target few pages per on-call shift; review alert quality regularly |
| **Ownership** | Routes to the owning team via labels (`team`, `service`) |

### 12.2 Alert quality review (monthly)

```text
For each alert that fired: Was it actionable? Was it urgent? Did it fire before users noticed? Was the runbook useful?
Delete or demote alerts that were not actionable; tune thresholds; add missing alerts found in incidents.
```

### 12.3 Alertmanager routing (excerpt)

```yaml
route:
  receiver: default-slack
  group_by: [alertname, service]
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  routes:
    - matchers: [severity="page"]
      receiver: oncall-pager
      continue: true
    - matchers: [team="orders"]
      receiver: orders-slack
inhibit_rules:
  - source_matchers: [alertname="ClusterDown"]
    target_matchers: [severity=~"page|ticket"]
    equal: [cluster]
receivers:
  - name: oncall-pager
    pagerduty_configs: [{ routing_key: <secret-from-secret-manager> }]   # or Opsgenie, Grafana OnCall, incident.io, etc.
  - name: orders-slack
    slack_configs: [{ channel: "#orders-alerts", send_resolved: true }]
  - name: default-slack
    slack_configs: [{ channel: "#alerts" }]
```

On-call practices and incident response: chapter **21**.

---

## 13. Dashboards

### 13.1 Dashboard hierarchy

```text
1. Executive / service overview: SLO status and error budget per critical journey
2. Service dashboard (one per service): RED metrics by route, dependencies, saturation, deploy/flag annotations
3. Component dashboards: database, cache, queue, Kubernetes namespace/workload
4. Debug/exploration: ad-hoc queries, traces, logs
```

### 13.2 Design rules

```text
🔴 Top-left: "Is it healthy?" (SLO, error rate, latency p95/p99, traffic)
🔴 Deployment and feature-flag change annotations on time-series panels
🔴 Units on every panel; percentiles not averages for latency
🟠 Consistent layout across services (generated from templates)
🟠 Link panels to traces/logs (exemplars, data links)
🟠 Dashboards as code (Grafana provisioning, Grafonnet/Jsonnet, Terraform provider, Perses — CNCF dashboard spec) reviewed in PRs
🟡 Keep dashboards few and maintained; delete abandoned ones
```

---

## 14. Synthetic Monitoring and Real-User Monitoring

| | Synthetic monitoring | Real-user monitoring (RUM) |
|---|---|---|
| Source | Scripted probes from chosen locations | Actual users' browsers/apps |
| Strength | Detects outages even with no traffic; consistent baseline; tests third-party dependencies | True user experience across devices/networks |
| Use | Uptime checks, multi-step journeys (login, search, checkout), SSL expiry, DNS | Core Web Vitals, JS errors, page/API timings by geography/device |
| Tools | Blackbox exporter, Grafana Synthetic Monitoring, Checkly (Playwright), cloud synthetics, uptime services | `web-vitals` + OTel browser SDK, Faro, vendor RUM, Crashlytics/MetricKit for mobile |

```text
🟠 Run synthetic journeys from multiple Indian cities/regions and key international markets
🟠 Mark synthetic traffic (headers/user agents) and exclude it from business metrics and SLIs where appropriate
🔴 External uptime monitoring from OUTSIDE your infrastructure (so it works when your monitoring stack is down)
```

---

## 15. Infrastructure and Kubernetes Monitoring

| Layer | Telemetry | Sources |
|---|---|---|
| Hosts | CPU, memory, disk space/IO, network, file descriptors, OOM kills | node_exporter, OTel hostmetrics receiver, cloud agents |
| Containers | CPU usage/throttling, memory working set vs limits, restarts | cAdvisor/kubelet metrics |
| Kubernetes objects | Deployments desired vs available, pending pods, PVC usage, HPA state, node conditions | kube-state-metrics, OTel k8s cluster receiver |
| Control plane | API server latency/errors, etcd health (self-managed) | Control plane metrics / provider monitoring |
| Network | LB 5xx/latency, NAT gateway errors, DNS errors, packet drops | Cloud metrics, Cilium/Hubble, eBPF tools |
| Certificates | Days until expiry | cert-manager metrics, blackbox exporter |

Ready-made dashboards/alerts: **kube-prometheus-stack** (Prometheus Operator, Alertmanager, Grafana, kubernetes-mixin rules). Single-server equivalent: **39**.

### Essential infrastructure alerts (ticket or page based on impact)

```text
- Disk will be full within N hours (predict_linear) — page if production DB/log volume
- Node NotReady / pods Pending for > 10 min
- Container OOMKilled / CrashLoopBackOff
- Certificate expires in < 14 days
- Backup job hasn't succeeded within its window (dead man's switch)
- Replication lag above threshold
```

---

## 16. Dependency and Database Monitoring

| Dependency | Key signals |
|---|---|
| Databases | Query latency (pg_stat_statements), connections vs max, pool wait time, locks/deadlocks, replication lag, cache hit, disk, XID age (chapter 10 §16) |
| Caches | Hit ratio, evictions, memory, latency, connected clients |
| Queues/streams | Queue depth, age of oldest message, consumer lag, DLQ size, publish/consume error rates |
| Third-party APIs | Latency/error rate per dependency, circuit breaker state, rate-limit responses (429) |
| DNS / TLS | Resolution failures, handshake errors |

Instrument outbound clients with dependency labels (`peer.service`/`server.address`) so you can answer "is it us or them?" in seconds.

---

## 17. Business Observability

Technical health can be green while the business is broken.

| Business metric | Detects |
|---|---|
| Orders per minute (vs same time last week) | Silent checkout failures, payment provider issues |
| Payment success rate by method (card/UPI/netbanking) | PSP or bank issues |
| Signups/logins per minute; OTP delivery success | Auth/SMS provider issues |
| Search → add-to-cart conversion | Search relevance or UI regressions |
| Refund/chargeback rate | Fraud or billing bugs |

```text
- Emit business events as metrics (low-cardinality labels) and/or analytics events
- Anomaly detection against seasonality (day-of-week, festival periods) rather than static thresholds
- Business SLOs for critical journeys (e.g. 99.5% of checkout attempts succeed, excluding user-caused declines)
```

---

## 18. Observability for AI / LLM Applications

| Signal | What to capture |
|---|---|
| Traces | Spans for prompt assembly, retrieval (vector search), model calls, tool calls, guardrails; OTel **GenAI semantic conventions** (`gen_ai.*` attributes such as model, token usage) |
| Metrics | Latency (time to first token, total), token usage and cost per request/tenant, error and refusal rates, cache hit rate, rate-limit responses from providers |
| Quality | Online evaluation scores, user feedback (thumbs up/down), groundedness checks on samples |
| Logs | Prompts/responses only with redaction, consent, and strict access controls — often sampled |
| Drift | Score trends after model/prompt/retrieval changes (tie to versioned prompts) |

```text
🔴 Never store raw prompts/responses containing personal data without a lawful basis, redaction, and retention limits
🟠 Tag every model call with model name/version and prompt version
🟠 Budget alerts on token spend per tenant/feature
```

Tooling: OTel GenAI instrumentation libraries, OpenLLMetry/OpenInference-style instrumentations, Langfuse, Arize Phoenix, vendor LLM observability features.

---

## 19. Observability Stacks

| Stack | Components | Fit |
|---|---|---|
| **Grafana LGTM** (open source or Grafana Cloud) | **L**oki (logs), **G**rafana (UI), **T**empo (traces), **M**imir/Prometheus (metrics), Pyroscope (profiles), Alloy (collector distribution) | Popular OSS stack; OTLP-native |
| **Elastic / OpenSearch** | Elasticsearch/OpenSearch, Kibana/Dashboards, APM, Beats/OTel | Search-heavy log analytics, SIEM convergence |
| **SigNoz / ClickHouse-based** | Unified OTel-native backend on ClickHouse | Single tool for all signals |
| **Jaeger + Prometheus + Loki** | Composable OSS | Teams wanting minimal stack |
| **Cloud-native** | CloudWatch + X-Ray/Application Signals, Google Cloud Observability, Azure Monitor + Application Insights | Deep cloud integration, managed |
| **Commercial APM** | Datadog, New Relic, Dynatrace, Honeycomb, Splunk Observability, Elastic Cloud, Chronosphere, etc. | Rich features, managed scale; cost management needed |

```text
Selection rule: instrument with OpenTelemetry regardless of backend — it keeps vendor choice open.
Single-server/self-hosted starter: Prometheus + Grafana + Loki (see 39-vps-enterprise-setup-guide.md).
```

---

## 20. Cost Control

| Lever | Action |
|---|---|
| Metrics cardinality | Remove unbounded labels; drop unused metrics; recording rules; limits per tenant |
| Log volume | Drop debug/noisy logs at the source or collector; sample repetitive logs; shorter hot retention + cheap archive |
| Trace volume | Head + tail sampling; drop spans for health checks and noisy internal calls |
| Retention tiers | Hot (days–weeks) → warm → cold/object storage (months) based on use and compliance |
| Ownership | Cost per team/service visible; budgets and alerts on ingestion |
| Collector processing | Filter/aggregate before sending to paid backends |

```text
Typical retention starting points (adjust to compliance):
Metrics: 13–15 months downsampled (year-over-year comparisons)
Traces:  7–30 days
App logs: 7–30 days hot, longer in archive if required (e.g. CERT-In 180-day log retention for in-scope logs)
Audit logs: per legal/regulatory requirements (often years), immutable
```

---

## 21. Data Governance — Retention, PII, Access

```text
🔴 Telemetry is data: classify it; telemetry containing personal data falls under DPDP/GDPR obligations
🔴 Redaction at source (logging libraries, span processors) + Collector-level redaction as backstop
🔴 Role-based access to logs/traces; production telemetry access audited
🔴 Retention policies enforced automatically; deletion supports erasure obligations where telemetry contains personal data
🟠 Data residency: observability backends/regions consistent with data residency requirements
🟠 Separate tenants' telemetry or enforce tenant-based access for multi-tenant platforms
```

---

## 22. Observability Maturity Model

| Level | Characteristics |
|---|---|
| **0 — Reactive** | Users report outages; logs on servers; no dashboards |
| **1 — Basic monitoring** | Uptime checks, host metrics, centralised logs, static threshold alerts |
| **2 — Service monitoring** | RED/USE metrics per service, structured logs, dashboards per service, on-call |
| **3 — Observability** | OpenTelemetry traces correlated with logs and metrics; SLOs with burn-rate alerting; deploy annotations; runbooks |
| **4 — Proactive** | Business SLOs, synthetic + RUM, continuous profiling, tail sampling, cost governance, alert quality reviews |
| **5 — Learning system** | Telemetry drives capacity planning, chaos experiments, error-budget policies, automated rollback, and product decisions |

---

## 23. Checklists

### New service (observability readiness)
- [ ] OTel SDK/auto-instrumentation with `service.name`, `service.version`, environment attributes
- [ ] RED metrics with route templates; runtime and pool saturation metrics
- [ ] Structured JSON logs with trace/span IDs; no secrets/PII
- [ ] Trace context propagated across HTTP, gRPC, and messaging
- [ ] SLOs defined for critical journeys; burn-rate alerts with runbooks
- [ ] Service dashboard from template with deploy annotations
- [ ] Dependency metrics (DB, cache, queue, third parties)
- [ ] Synthetic check for the main user journey
- [ ] Business metrics for the service's outcomes

### Platform
- [ ] Collector agents + gateway with memory limits, batching, redaction, tail sampling
- [ ] Metrics backend with cardinality limits; recording rules for SLIs
- [ ] Log pipeline with parsing, enrichment, retention tiers
- [ ] Alert routing by team/severity; inhibition rules; external uptime monitoring
- [ ] Dashboards and alerts as code
- [ ] Cost and retention policies; access controls; telemetry data classification

### Monthly
- [ ] Alert quality review (actionable? noisy? missing?)
- [ ] SLO review and error-budget status
- [ ] Top cardinality/ingestion contributors reviewed
- [ ] Runbooks updated from recent incidents

---

## 24. References

### Standards and specs
- OpenTelemetry documentation: https://opentelemetry.io/docs/
- OpenTelemetry semantic conventions: https://opentelemetry.io/docs/specs/semconv/
- OpenTelemetry Collector: https://opentelemetry.io/docs/collector/
- OpenTelemetry Profiles public alpha announcement (2026): https://opentelemetry.io/blog/2026/profiles-alpha/
- W3C Trace Context: https://www.w3.org/TR/trace-context/
- OpenSLO: https://openslo.com/

### SRE practice
- Google SRE Book — Monitoring Distributed Systems: https://sre.google/sre-book/monitoring-distributed-systems/
- Google SRE Book — Service Level Objectives: https://sre.google/sre-book/service-level-objectives/
- Google SRE Workbook — Alerting on SLOs: https://sre.google/workbook/alerting-on-slos/
- Brendan Gregg — USE Method: https://www.brendangregg.com/usemethod.html
- Tom Wilkie — RED Method: https://grafana.com/blog/2018/08/02/the-red-method-how-to-instrument-your-services/

### Tools
- Prometheus: https://prometheus.io/docs/ · Prometheus OTLP ingestion guide: https://prometheus.io/docs/guides/opentelemetry/
- Grafana: https://grafana.com/docs/ · Loki: https://grafana.com/docs/loki/ · Tempo: https://grafana.com/docs/tempo/ · Mimir: https://grafana.com/docs/mimir/ · Pyroscope: https://grafana.com/docs/pyroscope/
- Jaeger: https://www.jaegertracing.io/ · kube-prometheus-stack: https://github.com/prometheus-community/helm-charts
- Sloth: https://sloth.dev/ · Pyrra: https://github.com/pyrra-dev/pyrra
- Perses: https://perses.dev/
- SigNoz: https://signoz.io/docs/

### Books
- *Site Reliability Engineering* and *The Site Reliability Workbook* — Google (free online)
- *Observability Engineering* — Majors, Fong-Jones, Miranda
- *Implementing Service Level Objectives* — Alex Hidalgo

---

**Previous:** [17 — Cloud & Infrastructure](./17-cloud-and-infrastructure.md) · **Next:** [19 — Performance & Scalability](./19-performance-and-scalability.md)