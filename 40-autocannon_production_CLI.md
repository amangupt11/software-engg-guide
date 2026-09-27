# Autocannon — Production-Grade CLI Load Testing Course

Version note: This guide is written for CLI-only usage. Verify the installed version with `npx autocannon --version` and consult the official README for version-specific changes.

Official project:
https://github.com/mcollina/autocannon

## 1. Installation

Recommended for a project:

    npm i autocannon

Verify:

    npx autocannon --version
    npx autocannon --help

Global installation is also possible:

    npm i -g autocannon

For reproducible projects, prefer the project-local dependency and `npx autocannon`.

## 2. Core mental model

Autocannon is an HTTP benchmarking/load-generation tool.

    Load generator
          |
          | HTTP/HTTPS
          v
    Load balancer / API
          |
          v
    Node.js / Express
          |
          v
    Database / external services

Autocannon measures the HTTP side of the workload. For production-grade analysis, also monitor application, infrastructure, database, and dependency metrics.

## 3. First benchmark

    npx autocannon http://localhost:3000

Typical baseline controls:

    -c 10       concurrent connections
    -d 10       duration in seconds
    -p 1        pipelining
    -m GET      HTTP method

Example:

    npx autocannon -c 100 -d 60 http://localhost:3000/api/customers

## 4. Important CLI options

| Option | Purpose |
|---|---|
| `-c` | Concurrent connections |
| `-p` | Pipelining |
| `-d` | Duration |
| `-a` | Fixed request amount |
| `-m` | HTTP method |
| `-H` | HTTP header |
| `-b` | Request body |
| `-i` | Body/input file |
| `-F` | Multipart form |
| `-t` | Timeout |
| `-w` | Worker threads |
| `-W` | Warmup |
| `-B` | Bailout |
| `-M` | Maximum requests per connection |
| `-O` | Maximum overall requests |
| `-r` | Connection/request rate control |
| `-D` | Reconnect rate |
| `-j` | JSON/NDJSON output |
| `-l` | Latency data |
| `-T` | Test title |
| `--renderStatusCodes` | Render HTTP status code statistics |
| `--debug` | Debug connection errors |
| `--har` | Use a HAR file |
| `-f` | Forever mode |

Run the authoritative option list any time:

    npx autocannon --help

## 5. Connections

    npx autocannon -c 10 http://localhost:3000
    npx autocannon -c 50 http://localhost:3000
    npx autocannon -c 100 http://localhost:3000
    npx autocannon -c 500 http://localhost:3000

A connection is not the same thing as a real user. Model workload intentionally.

## 6. Duration

    npx autocannon -d 10 URL
    npx autocannon -d 60 URL
    npx autocannon -d 300 URL

Use fixed duration for repeatable benchmark comparisons.

## 7. Fixed request count

    npx autocannon -a 10000 URL

Use this when you want a fixed workload instead of a fixed duration.

## 8. HTTP methods

    npx autocannon -m GET URL
    npx autocannon -m POST URL
    npx autocannon -m PUT URL
    npx autocannon -m PATCH URL
    npx autocannon -m DELETE URL

Never run destructive methods against real production data without an explicitly approved test plan.

## 9. Headers

    npx autocannon \
      -c 100 \
      -d 60 \
      -H "content-type: application/json" \
      URL

Authentication:

    npx autocannon \
      -H "Authorization: Bearer $LOAD_TEST_TOKEN" \
      URL

Use dedicated test credentials. Avoid placing real secrets directly into shell history or committed scripts.

## 10. JSON POST

Example:

    npx autocannon \
      -c 100 \
      -d 60 \
      -m POST \
      -H "content-type: application/json" \
      -b '{"name":"Load Test Customer","mobile":"9999999999"}' \
      http://localhost:3000/api/customers

## 11. Body from file

Create `customer.json`:

    {
      "name": "Load Test Customer",
      "mobile": "9999999999",
      "category": "individual"
    }

Run:

    npx autocannon \
      -c 100 \
      -d 60 \
      -m POST \
      -H "content-type: application/json" \
      -i customer.json \
      URL

## 12. Authentication benchmark

Set a test token:

Linux/macOS:

    export LOAD_TEST_TOKEN="TEST_TOKEN"

PowerShell:

    $env:LOAD_TEST_TOKEN="TEST_TOKEN"

Run:

    npx autocannon \
      -T "customers-authenticated-100c" \
      -c 100 \
      -p 1 \
      -d 60 \
      -t 10 \
      -H "Authorization: Bearer $LOAD_TEST_TOKEN" \
      --renderStatusCodes \
      https://staging.example.com/api/customers

## 13. Pipelining

    npx autocannon -c 100 -p 1 -d 60 URL

Higher pipelining:

    npx autocannon -c 100 -p 10 -d 60 URL

Do not increase pipelining just to produce a larger number. Use it only when the workload represents the client behavior you want to model.

## 14. Timeout

    npx autocannon -c 100 -d 60 -t 5 URL

A timeout is a useful signal. Do not simply increase the timeout to hide an unhealthy API.

## 15. Status-code reporting

    npx autocannon \
      -c 100 \
      -d 60 \
      --renderStatusCodes \
      URL

Always investigate unexpected 4xx/5xx responses.

A benchmark with very high RPS and a high error rate is not a healthy capacity result.

## 16. Debugging

    npx autocannon \
      --debug \
      -c 100 \
      -d 30 \
      URL

Use this when diagnosing connection errors, resets, timeouts, or TLS/network problems.

## 17. Latency data

    npx autocannon \
      -l \
      -c 100 \
      -d 60 \
      URL

Important latency concepts:

- p50: median
- p90: 90th percentile
- p95: 95th percentile
- p99: 99th percentile

Do not judge an API using average latency alone.

## 18. JSON/NDJSON output

    npx autocannon \
      -j \
      -c 100 \
      -d 60 \
      URL > benchmark.ndjson

This is useful for CLI automation, CI/CD, archiving, parsing, and comparing results.

## 19. Warmup

Warmup lets the application settle before the measured period.

Example form:

    npx autocannon \
      -c 100 \
      -d 60 \
      -W "-c 100 -d 10" \
      URL

Verify the exact syntax for your installed version with:

    npx autocannon --help

## 20. Worker threads

Example:

    npx autocannon \
      -c 1000 \
      -w 4 \
      -d 60 \
      URL

Workers can increase load-generator capacity, but more workers do not automatically make a benchmark more accurate.

## 21. Rate controls

Autocannon provides rate-related options. Use them when you need to model controlled traffic instead of simply maximizing generated traffic.

Always verify current option semantics:

    npx autocannon --help

## 22. Bailout

Use bailout to stop a test when a failure/threshold condition is reached.

Example:

    npx autocannon \
      -c 100 \
      -d 300 \
      -B 100 \
      URL

Use conservative stop conditions in shared environments.

## 23. Max requests

Per-connection limit:

    npx autocannon -c 100 -M 100 URL

Overall request limit:

    npx autocannon -c 100 -O 10000 URL

Check current semantics with `--help`, especially when using worker threads.

## 24. HAR

Autocannon supports HAR-based workloads.

Example:

    npx autocannon --har customer-flow.har BASE_URL

Use HAR when a realistic sequence of captured HTTP requests is more representative than a single endpoint.

## 25. Forever mode

    npx autocannon -f -c 100 URL

Use only in controlled environments. It can generate continuous traffic until stopped.

## 26. HTTPS

    npx autocannon \
      -c 100 \
      -d 60 \
      https://staging.example.com/api/customers

Autocannon supports HTTPS and additional TLS-related options. Check `--help` for your version.

## 27. Benchmark result interpretation

Capture at least:

    Requests/sec
    Latency average
    p50
    p95
    p99
    Max latency
    2xx
    3xx
    4xx
    5xx
    Timeouts
    Connection errors
    Bytes/sec

Also monitor:

    API CPU
    API RAM
    Node.js event-loop behavior
    Network
    MongoDB CPU
    MongoDB memory
    MongoDB connections
    MongoDB query latency
    External dependency latency

## 28. Load-generator bottleneck

Autocannon itself consumes CPU.

Example:

    Load generator CPU = 100%
    API CPU            = 40%

You cannot conclude that the API has reached 40% CPU capacity. The generator may be the limiting factor.

For very high-throughput tests, consider:

    - a stronger load-generator machine
    - multiple load generators
    - worker threads
    - another benchmark tool when appropriate

## 29. Production test architecture

Recommended:

    Load Generator
          |
          v
    Load Balancer
          |
      +---+---+
      |   |   |
     API API API
      |   |   |
      +---+---+
          |
       MongoDB
          |
    External services

Keep the load generator separate from the API when serious capacity measurements are required.

## 30. Development vs staging vs production

Preferred progression:

    localhost
       |
    development
       |
    staging
       |
    production-like staging
       |
    production only under controlled, approved conditions

For POST/PUT/PATCH/DELETE tests, use test data.

Never accidentally point a destructive benchmark at production.

## 31. Baseline methodology

1. Run a baseline.
2. Record the exact command.
3. Record application version.
4. Record infrastructure.
5. Change one thing.
6. Run the same benchmark.
7. Compare results.
8. Keep/revert the change.

Example baseline:

    npx autocannon \
      -T "baseline-customers-100c" \
      -c 100 \
      -p 1 \
      -d 60 \
      --renderStatusCodes \
      https://staging.example.com/api/customers

## 32. Ramp-up test

Use controlled levels:

    10 connections
    25 connections
    50 connections
    100 connections
    250 connections
    500 connections
    750 connections
    1000 connections

At each level record:

    RPS
    p95
    p99
    errors
    timeouts
    CPU
    RAM
    database load

Stop if the environment or agreed thresholds become unsafe.

## 33. Capacity test

The goal is to determine the maximum sustainable workload under defined objectives.

Do not define success as maximum RPS alone.

A useful capacity result looks like:

    Workload:
    250 concurrent connections

    Throughput:
    X req/s

    p95:
    X ms

    p99:
    X ms

    5xx:
    X%

    Timeout:
    X

    API CPU:
    X%

    MongoDB CPU:
    X%

Then state whether the workload satisfies your pre-defined SLO/performance requirements.

## 34. Stress test

Stress testing pushes beyond normal expected load.

Example progression:

    100
    250
    500
    750
    1000
    1500

Use a staging or dedicated environment.

The objective is to understand degradation and failure behavior, not simply obtain a large RPS number.

## 35. Spike test

Controlled sequence:

    100 connections for 30s
          |
          v
    1000 connections for 30s
          |
          v
    100 connections for 30s

Example CLI runs:

    npx autocannon -c 100 -d 30 URL

    npx autocannon -c 1000 -d 30 URL

    npx autocannon -c 100 -d 30 URL

Observe both degradation and recovery.

## 36. Soak test

Example:

    npx autocannon \
      -T "customers-soak" \
      -c 100 \
      -p 1 \
      -d 1800 \
      URL

30 minutes.

Look for:

    memory leaks
    connection leaks
    latency drift
    resource exhaustion
    database connection problems

## 37. Node.js + Express + MongoDB workflow

For a database-backed endpoint:

    Autocannon
        |
        v
    Express
        |
        v
    Route/controller
        |
        v
    MongoDB driver
        |
        v
    MongoDB

If latency increases, inspect every layer.

Possible bottlenecks:

    Node.js CPU
    event loop
    synchronous code
    database query
    missing index
    connection pool
    serialization
    network
    external API
    load balancer

## 38. MongoDB index validation

Example workload:

    GET /api/customers/:mobile

If the database searches by `mobile`, benchmark before and after an appropriate index.

Do not assume an index is useful without measuring the query and workload.

The correct workflow:

    Baseline
       ↓
    Inspect query
       ↓
    Add/adjust index
       ↓
    Repeat exact benchmark
       ↓
    Compare

## 39. Authentication safety

Use:

    dedicated test account
    dedicated test token
    staging environment
    test database

Avoid:

    production credentials
    personal tokens
    real customer data

Use environment variables for secrets and ensure CI logs do not expose them.

## 40. External API safety

If your API calls:

    payment provider
    SMS provider
    WhatsApp provider
    email provider
    maps provider
    other external APIs

decide whether the test should use:

    provider sandbox
    mock service
    controlled integration environment

Otherwise your load test may unintentionally load or rate-limit a third-party service.

## 41. Rate-limit testing

If your application has:

    100 requests/minute/IP

and you run a large benchmark, you may primarily test your rate limiter.

That is valid if intentional.

Separate:

    rate-limit test
    application capacity test
    database capacity test

## 42. CI/CD

A CLI-only CI workflow can look like:

    Build
      ↓
    Deploy staging
      ↓
    Run Autocannon
      ↓
    Capture NDJSON
      ↓
    Evaluate thresholds
      ↓
    Pass / fail

Example:

    npx autocannon \
      -j \
      -c 100 \
      -d 60 \
      https://staging.example.com/api/customers \
      > benchmark.ndjson

## 43. Benchmark naming

Use consistent names:

    customers-get-10c-60s
    customers-get-100c-60s
    customers-get-500c-60s
    customers-post-100c-60s
    drivers-get-100c-60s
    vendors-get-100c-60s
    auth-login-50c-60s

## 44. CLI-only production template

GET:

    npx autocannon \
      -T "customers-get-100c" \
      -c 100 \
      -p 1 \
      -d 60 \
      -t 10 \
      --renderStatusCodes \
      https://staging.example.com/api/customers

Authenticated GET:

    npx autocannon \
      -T "customers-authenticated-100c" \
      -c 100 \
      -p 1 \
      -d 60 \
      -t 10 \
      -H "Authorization: Bearer $LOAD_TEST_TOKEN" \
      --renderStatusCodes \
      https://staging.example.com/api/customers

POST:

    npx autocannon \
      -T "customer-create-100c" \
      -c 100 \
      -p 1 \
      -d 60 \
      -t 10 \
      -m POST \
      -H "content-type: application/json" \
      -b '{"name":"Load Test Customer","mobile":"9999999999"}' \
      --renderStatusCodes \
      https://staging.example.com/api/customers

Machine-readable:

    npx autocannon \
      -j \
      -c 100 \
      -d 60 \
      https://staging.example.com/api/customers \
      > benchmark.ndjson

## 45. Production safety checklist

Before a serious test:

[ ] Correct target URL
[ ] Staging/dedicated environment
[ ] Test database
[ ] Test credentials
[ ] Test tokens
[ ] External services identified
[ ] Monitoring active
[ ] Database monitoring active
[ ] Load generator capacity checked
[ ] Team notified
[ ] Stop conditions defined
[ ] Destructive operations reviewed
[ ] Results storage defined

## 46. What a good benchmark report contains

    Endpoint:
    HTTP method:
    Environment:
    Application version:
    Database:
    Node.js version:
    Autocannon version:
    Connections:
    Pipelining:
    Duration:
    Warmup:
    Authentication:
    Payload:

    Throughput:
    p50:
    p95:
    p99:
    Max latency:

    2xx:
    3xx:
    4xx:
    5xx:
    Timeouts:
    Connection errors:

    API CPU:
    API RAM:
    MongoDB CPU:
    MongoDB connections:

    Conclusion:
    Bottleneck:
    Follow-up action:

## 47. Rules for reproducible benchmarks

Keep constant:

    endpoint
    HTTP method
    payload
    authentication state
    connection count
    pipelining
    duration
    timeout
    infrastructure
    database dataset
    application version

Change one variable at a time.

## 48. Common mistakes

1. Testing production accidentally.
2. Looking only at average latency.
3. Looking only at RPS.
4. Ignoring 5xx.
5. Ignoring timeouts.
6. Running the generator and API on the same small machine.
7. Changing multiple variables between benchmarks.
8. Testing with unrealistic payloads.
9. Ignoring database metrics.
10. Ignoring external services.
11. Using production credentials.
12. Increasing timeout to hide slow responses.
13. Increasing pipelining without a realistic reason.
14. Treating connections as equivalent to users.
15. Assuming the load generator can always produce unlimited traffic.

## 49. Minimal CLI cheat sheet

    npx autocannon --help
    npx autocannon --version

    npx autocannon URL

    npx autocannon -c 100 URL
    npx autocannon -d 60 URL
    npx autocannon -c 100 -d 60 URL

    npx autocannon -a 10000 URL

    npx autocannon -m POST URL

    npx autocannon \
      -m POST \
      -H "content-type: application/json" \
      -b '{"name":"Aman"}' \
      URL

    npx autocannon \
      -H "Authorization: Bearer $LOAD_TEST_TOKEN" \
      URL

    npx autocannon \
      --renderStatusCodes \
      --debug \
      URL

    npx autocannon \
      -j \
      URL > benchmark.ndjson

## 50. Recommended learning path

Level 1:
    installation
    --help
    GET
    -c
    -d

Level 2:
    POST
    PUT
    PATCH
    DELETE
    -H
    -b
    -i

Level 3:
    RPS
    latency
    p50
    p95
    p99
    errors
    status codes

Level 4:
    -p
    -t
    -W
    -w
    -B
    -M
    -O
    rate controls

Level 5:
    -j
    CI/CD
    machine-readable results
    benchmark comparison

Level 6:
    load
    stress
    spike
    soak
    capacity testing

Level 7:
    Node.js bottlenecks
    MongoDB bottlenecks
    connection pools
    event loop
    infrastructure
    load-generator limits

Level 8:
    production-grade performance methodology
    SLO-based thresholds
    reproducibility
    safe test environments
    automated regression testing

## 51. Official references

Autocannon GitHub:
https://github.com/mcollina/autocannon

Autocannon package:
https://www.npmjs.com/package/autocannon

Always check the installed version:

    npx autocannon --version

Then inspect its exact CLI:

    npx autocannon --help

## 52. Final production principle

Do not optimize for:

    maximum RPS

Optimize and test for:

    required throughput
    acceptable p95/p99 latency
    acceptable error rate
    acceptable resource utilization
    predictable degradation
    recovery behavior

The correct production benchmark is the workload that represents your real application and has explicit success/failure criteria.
