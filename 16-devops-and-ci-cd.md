# 🚀 DevOps & CI/CD — Production Engineering Guide

> How to deliver changes to production quickly, safely, and repeatably: DevOps culture and DORA capabilities, continuous integration, pipeline design and time budgets, reproducible builds and artifact management, continuous delivery vs deployment, deployment strategies and progressive delivery, GitOps, environment promotion, database migrations in pipelines, feature flags, pipeline and supply-chain security, secrets, platform engineering and golden paths, CI/CD platform comparison, reference pipelines (GitHub Actions, GitLab CI, Bitbucket Pipelines), monorepo CI, deployment verification and automated rollback, change management, metrics, runners, and cost.
>
> Related: [02 SDLC (gates, environments, releases)](./02-software-development-lifecycle.md) · [07 Version control](./07-version-control.md) · [08 Testing (pipeline stages)](./08-testing-and-quality.md#11-stlc-in-agile-and-cicd) · [09 CI/CD security §19](./09-security-engineering.md#19-cicd-pipeline-security) · [17 Cloud & IaC](./17-cloud-and-infrastructure.md) · [18 Observability](./18-observability-and-monitoring.md) · [21 Production operations](./21-production-operations.md) · [39 VPS deploys](./39-vps-enterprise-setup-guide.md) · [42 Git hosting](./42-Git-GitHub-GitLab-Bitbucket-Setup.md)

---

## 📚 Table of Contents

- [🚀 DevOps \& CI/CD — Production Engineering Guide](#-devops--cicd--production-engineering-guide)
  - [📚 Table of Contents](#-table-of-contents)
  - [1. What DevOps Is](#1-what-devops-is)
    - [CALMS model](#calms-model)
    - [The Three Ways (from *The DevOps Handbook*)](#the-three-ways-from-the-devops-handbook)
  - [2. DORA Capabilities](#2-dora-capabilities)
  - [3. Continuous Integration](#3-continuous-integration)
    - [3.1 Definition](#31-definition)
    - [3.2 CI rules](#32-ci-rules)
    - [3.3 Local parity](#33-local-parity)
  - [4. Pipeline Design](#4-pipeline-design)
    - [4.1 Reference stages](#41-reference-stages)
    - [4.2 Time budgets](#42-time-budgets)
    - [4.3 Design principles](#43-design-principles)
  - [5. Builds — Reproducible, Cached, Versioned](#5-builds--reproducible-cached-versioned)
    - [5.1 Reproducibility](#51-reproducibility)
    - [5.2 Caching](#52-caching)
    - [5.3 Versioning artifacts](#53-versioning-artifacts)
  - [6. Artifact Management](#6-artifact-management)
  - [7. Continuous Delivery vs Continuous Deployment](#7-continuous-delivery-vs-continuous-deployment)
  - [8. Deployment Strategies and Progressive Delivery](#8-deployment-strategies-and-progressive-delivery)
    - [8.1 Strategies](#81-strategies)
    - [8.2 Progressive delivery tools](#82-progressive-delivery-tools)
    - [8.3 Canary with Argo Rollouts (excerpt)](#83-canary-with-argo-rollouts-excerpt)
  - [9. GitOps](#9-gitops)
    - [9.1 OpenGitOps principles](#91-opengitops-principles)
    - [9.2 Tools](#92-tools)
    - [9.3 Repository layout (config repo)](#93-repository-layout-config-repo)
    - [9.4 GitOps flow](#94-gitops-flow)
  - [10. Environments and Promotion](#10-environments-and-promotion)
    - [Protected environments (GitHub)](#protected-environments-github)
  - [11. Database Migrations in Pipelines](#11-database-migrations-in-pipelines)
  - [12. Feature Flags in Delivery](#12-feature-flags-in-delivery)
  - [13. Pipeline Security and Supply Chain](#13-pipeline-security-and-supply-chain)
  - [14. Secrets in CI/CD](#14-secrets-in-cicd)
  - [15. Platform Engineering and Golden Paths](#15-platform-engineering-and-golden-paths)
  - [16. CI/CD Platforms Compared](#16-cicd-platforms-compared)
  - [17. Reference Pipeline — GitHub Actions](#17-reference-pipeline--github-actions)
  - [18. Reference Pipelines — GitLab CI and Bitbucket Pipelines](#18-reference-pipelines--gitlab-ci-and-bitbucket-pipelines)
    - [18.1 GitLab CI](#181-gitlab-ci)
    - [18.2 Bitbucket Pipelines](#182-bitbucket-pipelines)
  - [19. Monorepo CI](#19-monorepo-ci)
  - [20. Deployment Verification and Automated Rollback](#20-deployment-verification-and-automated-rollback)
    - [20.1 Post-deploy verification](#201-post-deploy-verification)
    - [20.2 Automated rollback triggers](#202-automated-rollback-triggers)
    - [20.3 Deployment records](#203-deployment-records)
  - [21. Change Management](#21-change-management)
    - [Change freeze policy (example)](#change-freeze-policy-example)
  - [22. Measuring Delivery](#22-measuring-delivery)
  - [23. Runners and Build Infrastructure](#23-runners-and-build-infrastructure)
  - [24. CI/CD Cost Optimisation](#24-cicd-cost-optimisation)
  - [25. Scenario Playbooks](#25-scenario-playbooks)
    - [25.1 Single VPS (startup) — GitHub Actions + SSH deploy](#251-single-vps-startup--github-actions--ssh-deploy)
    - [25.2 Kubernetes with GitOps](#252-kubernetes-with-gitops)
    - [25.3 Serverless (AWS Lambda / Cloud Run / Azure Functions)](#253-serverless-aws-lambda--cloud-run--azure-functions)
    - [25.4 Mobile apps](#254-mobile-apps)
    - [25.5 Regulated enterprise](#255-regulated-enterprise)
  - [26. Checklists](#26-checklists)
    - [CI](#ci)
    - [Build \& artifacts](#build--artifacts)
    - [CD](#cd)
    - [Security \& governance](#security--governance)
  - [27. References](#27-references)
    - [Research and principles](#research-and-principles)
    - [Platforms and tools](#platforms-and-tools)
    - [Change management](#change-management)

---

## 1. What DevOps Is

DevOps is a set of **cultural values, practices, and tools** that unify software development and operations so that teams can deliver changes quickly **and** reliably. It is not a job title or a tool.

### CALMS model

| Pillar | Meaning | Practices |
|---|---|---|
| **C**ulture | Shared ownership, blameless learning, collaboration | "You build it, you run it", blameless postmortems |
| **A**utomation | Automate build, test, deploy, infrastructure | CI/CD, IaC, policy as code |
| **L**ean | Small batches, limit WIP, remove waste | Trunk-based development, small PRs |
| **M**easurement | Data-driven improvement | DORA metrics, SLOs, value-stream metrics |
| **S**haring | Knowledge and tooling shared across teams | Docs, golden paths, internal platforms |

### The Three Ways (from *The DevOps Handbook*)

```text
1. Flow:      fast left-to-right flow from dev to ops to customer (small batches, automation, WIP limits)
2. Feedback:  fast right-to-left feedback (monitoring, testing, telemetry, peer review)
3. Learning:  continual experimentation and learning (blameless culture, improvement time, chaos/game days)
```

---

## 2. DORA Capabilities

DORA research identifies capabilities that predict better software delivery and organisational performance. Selected technical capabilities relevant to this chapter:

| Capability | What it looks like |
|---|---|
| **Version control** for everything | Code, config, IaC, pipelines (chapter 07) |
| **Trunk-based development** | Short-lived branches, frequent merges |
| **Continuous integration** | Every commit built and tested automatically |
| **Test automation** | Reliable, fast automated suites (chapter 08) |
| **Deployment automation** | Push-button / fully automated deploys |
| **Continuous delivery** | Always releasable main branch |
| **Loosely coupled architecture** | Teams deploy independently (chapter 04) |
| **Shift-left on security** | Security in design and pipelines (chapter 09) |
| **Monitoring and observability** | Fast detection and diagnosis (chapter 18) |
| **Database change management** | Versioned, automated migrations (chapter 10) |
| **Platform engineering / internal developer platforms** | Self-service golden paths |
| **AI capabilities** (2025 DORA AI Capabilities Model) | Clear AI policy, strong version control, small batches, quality internal platforms — AI amplifies existing strengths and weaknesses |

Five delivery metrics (throughput and instability): chapter 01 §17. Capability catalogue: https://dora.dev/capabilities/

---

## 3. Continuous Integration

### 3.1 Definition

Continuous integration means every developer integrates work into the shared mainline **at least daily**, and every integration is verified by an automated build and tests. A long-lived branch with a CI server is not CI.

### 3.2 CI rules

| Rule | Status |
|---|:---:|
| Every push/PR triggers build + tests automatically | 🔴 |
| Main branch is always green; a broken main is fixed (or reverted) immediately — top priority | 🔴 |
| Fast feedback: PR pipeline ≤ ~10–15 minutes | 🔴 |
| Tests are deterministic; flaky tests quarantined and fixed (chapter 08 §25) | 🔴 |
| Same pipeline definition for everyone; pipeline as code in the repo | 🔴 |
| Build once; later stages reuse the same artifact | 🔴 |
| Developers can run the same checks locally (scripts/Make/Task/just targets) | 🟠 |
| Merge queue for busy repos (chapter 07 §11.2) | 🟠 |

### 3.3 Local parity

```makefile
# Makefile — same commands locally and in CI
.PHONY: lint test build
lint:   ; npm run format:check && npm run lint && npm run typecheck
test:   ; npm test -- --coverage
build:  ; docker build -t orders:$(shell git rev-parse --short=12 HEAD) .
ci: lint test build
```

---

## 4. Pipeline Design

### 4.1 Reference stages

```text
┌──────────────────────────── CI (per PR / push) ────────────────────────────┐
│ 1 Checkout → 2 Setup (cached deps) → 3 Static checks (format, lint, types)  │
│ → 4 Unit tests → 5 Build artifact → 6 Integration/contract tests            │
│ → 7 Security: SAST, SCA, secrets, IaC/container scan → 8 Package + SBOM     │
│ → 9 Publish artifact (immutable, signed) [main only]                        │
└─────────────────────────────────────────────────────────────────────────────┘
┌──────────────────────────── CD (per artifact) ─────────────────────────────┐
│ 10 Deploy to staging (migrations expand step) → 11 E2E smoke + DAST baseline│
│ → 12 Performance check (budgets) → 13 Approval (risk-based)                 │
│ → 14 Production progressive rollout (canary) → 15 Automated analysis        │
│ → 16 Full rollout / automatic rollback → 17 Post-deploy verification        │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 4.2 Time budgets

| Stage group | Target |
|---|---|
| Pre-commit hooks | < 10 s |
| PR pipeline (to mergeable signal) | ≤ 10–15 min |
| Main → staging deployed | ≤ 20–30 min |
| Staging → production (continuous delivery) | Minutes to hours depending on gates |
| Rollback | Minutes |

### 4.3 Design principles

```text
- Fail fast: cheapest, fastest checks first; parallelise independent jobs
- Hermetic jobs: no hidden dependencies on runner state; pin tool versions
- Idempotent deploy steps: re-running a deploy is safe
- Visibility: every run links commit → artifact → environment → deployment record
- Pipeline code is reviewed, linted (actionlint, gitlab-ci lint), and owned (CODEOWNERS)
- Reuse via shared templates/reusable workflows/components — with version pinning
```

---

## 5. Builds — Reproducible, Cached, Versioned

### 5.1 Reproducibility

```text
🔴 Lockfiles and pinned toolchains (.nvmrc/.node-version, .tool-versions/mise, Gradle wrapper, go.mod toolchain, global.json)
🔴 Pinned base images by digest; pinned CI actions/components by SHA/version
🔴 Build from a clean checkout; no artifacts from developer machines
🟠 Deterministic builds where tooling supports (SOURCE_DATE_EPOCH, reproducible-build flags)
```

### 5.2 Caching

| Cache | Example |
|---|---|
| Dependency caches | npm/pnpm store, Maven/Gradle caches, pip/uv cache, Go module cache, NuGet |
| Build caches | Gradle build cache, Turborepo/Nx remote cache, Bazel remote cache, ccache |
| Docker layer cache | BuildKit cache (`--cache-from/--cache-to`, registry or GitHub Actions cache backends) |
| Test result caches | Affected-only testing in monorepos |

Cache keys include lockfile hashes and tool versions; caches are a performance optimisation, never a source of truth.

### 5.3 Versioning artifacts

```text
Container image tags:  <service>:<semver>  AND  <service>:<git-sha>   — deploy by DIGEST (sha256:…)
Never deploy mutable tags like :latest
Library packages:      SemVer via release automation (chapter 07 §13)
Build metadata:        commit SHA, build number, build time, pipeline URL embedded as labels/annotations (OCI annotations)
```

```dockerfile
LABEL org.opencontainers.image.source="https://github.com/shopnow/orders" \
      org.opencontainers.image.revision="${GIT_SHA}" \
      org.opencontainers.image.version="${VERSION}"
```

---

## 6. Artifact Management

| Artifact | Registry options |
|---|---|
| Container images | GHCR, GitLab Container Registry, Amazon ECR, Google Artifact Registry, Azure Container Registry, Harbor, Docker Hub |
| Language packages | npm, Maven, PyPI, NuGet, Go proxy (private: Artifactory, Nexus, GitHub Packages, GitLab Package Registry, CodeArtifact, Artifact Registry) |
| Helm charts / OCI artifacts | OCI registries |
| Binaries / installers | Release assets, object storage with signed URLs |

```text
🔴 Artifacts immutable (tag immutability enabled where supported)
🔴 Signed (cosign) with SBOM and provenance attestations attached (chapter 09 §16)
🔴 Promotion by reference (same digest across environments), not by rebuilding
🟠 Retention policies: keep release artifacts long-term; prune PR/snapshot artifacts
🟠 Vulnerability scanning in the registry (continuous, catches new CVEs in old images)
🟠 Pull-through caches/mirrors for public registries (rate limits, availability, supply-chain control)
```

---

## 7. Continuous Delivery vs Continuous Deployment

| | Continuous delivery | Continuous deployment |
|---|---|---|
| Every change on main is… | **Releasable** (deployed to staging, production deploy is a business decision/button) | **Released** automatically if all checks pass |
| Human approval to production | Yes (ideally risk-based, quick) | No (automated gates) |
| Prerequisites | Strong automation and tests | All of CD + excellent tests, observability, progressive delivery, automatic rollback, feature flags |
| Common in | Regulated industries, mobile/shipped software | SaaS/web services with mature practices |

> Decouple **deploy** (code to production) from **release** (feature visible to users) using feature flags — this makes continuous deployment safe.

---

## 8. Deployment Strategies and Progressive Delivery

### 8.1 Strategies

| Strategy | Mechanism | Downtime | Rollback speed | Infra cost | Notes |
|---|---|---|---|---|---|
| Recreate | Stop old, start new | Yes | Slow | Low | Only for non-critical/dev |
| **Rolling** | Replace instances gradually | No | Medium | Low | Kubernetes Deployment default |
| **Blue-green** | Two environments; switch traffic | No | Instant | 2× during switch | Great for stateful cutovers with care |
| **Canary** | Small % of traffic → analyse → increase | No | Fast | Low | Needs good metrics |
| Shadow / dark traffic | Mirror traffic to new version, discard responses | No | n/a | Extra | Validate performance/behaviour safely (no side effects!) |
| **Feature-flag release** | Code deployed off; enable per cohort | No | Instant (toggle) | Low | Best for user-facing features |
| Ring deployment | Internal → beta → regions → all | No | Fast | Low | Large-scale/global services |

### 8.2 Progressive delivery tools

| Tool | Platform |
|---|---|
| **Argo Rollouts** | Kubernetes canary/blue-green with analysis (Prometheus, Datadog, etc.) |
| **Flagger** | Kubernetes progressive delivery with service mesh/ingress/Gateway API integrations |
| AWS CodeDeploy / ECS deployment controllers | Canary/linear for ECS/Lambda |
| Google Cloud Deploy | Canary across GKE/Cloud Run |
| Azure Container Apps revisions / App Service slots | Traffic splitting / swaps |
| Spinnaker | Multi-cloud deployment pipelines with canary analysis (Kayenta) |

### 8.3 Canary with Argo Rollouts (excerpt)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata: { name: orders }
spec:
  replicas: 10
  strategy:
    canary:
      steps:
        - setWeight: 5
        - pause: { duration: 5m }
        - analysis:
            templates: [{ templateName: success-rate-and-latency }]
        - setWeight: 25
        - pause: { duration: 10m }
        - setWeight: 50
        - pause: { duration: 10m }
  selector: { matchLabels: { app: orders } }
  template:
    metadata: { labels: { app: orders } }
    spec:
      containers:
        - name: orders
          image: ghcr.io/shopnow/orders@sha256:<digest>
```

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata: { name: success-rate-and-latency }
spec:
  metrics:
    - name: error-rate
      interval: 1m
      failureLimit: 1
      successCondition: result[0] < 0.01
      provider:
        prometheus:
          address: http://prometheus.monitoring:9090
          query: |
            sum(rate(http_server_request_duration_seconds_count{service="orders",http_response_status_code=~"5..",version="canary"}[2m]))
            /
            sum(rate(http_server_request_duration_seconds_count{service="orders",version="canary"}[2m]))
```

> Metric names depend on your instrumentation (OpenTelemetry semantic conventions vs framework defaults) — adapt queries.

---

## 9. GitOps

### 9.1 OpenGitOps principles

| # | Principle |
|---:|---|
| 1 | **Declarative** — desired system state expressed declaratively |
| 2 | **Versioned and immutable** — desired state stored in a way that enforces immutability and versioning with complete history (Git) |
| 3 | **Pulled automatically** — agents pull desired state from the source |
| 4 | **Continuously reconciled** — agents observe actual state and reconcile to desired state |

### 9.2 Tools

| Tool | Notes |
|---|---|
| **Argo CD** (CNCF graduated) | UI, app-of-apps/ApplicationSets, multi-cluster, RBAC, sync waves/hooks |
| **Flux** (CNCF graduated) | Toolkit of controllers, Helm/Kustomize, image automation, OCI sources |

### 9.3 Repository layout (config repo)

```text
platform-config/
├── apps/
│   └── orders/
│       ├── base/                    # Kustomize base (Deployment/Rollout, Service, HPA, PDB)
│       └── overlays/
│           ├── staging/             # image digest, replicas, config for staging
│           └── production/
├── infrastructure/                  # cluster add-ons: cert-manager, gateway controller, observability
└── clusters/
    ├── staging/                     # Argo CD Applications / Flux Kustomizations per cluster
    └── production/
```

### 9.4 GitOps flow

```text
App repo CI: build → test → push image (digest) → open PR to config repo updating staging overlay digest
Config repo: PR review/auto-merge → Argo CD/Flux syncs staging → verification
Promotion:   PR updates production overlay with the SAME digest → approval → sync → progressive rollout
Rollback:    git revert the promotion commit → controller reconciles to previous digest
Drift:       manual cluster changes are reverted/flagged by reconciliation
```

Rules: no `kubectl apply` to production by humans (break-glass only, audited); secrets via External Secrets/Sealed Secrets/SOPS, never plaintext in Git.

---

## 10. Environments and Promotion

Environment definitions: chapter 02 §14. Delivery-specific rules:

```text
🔴 Build once, promote the same digest: dev → staging → production
🔴 Environment config separated from artifacts (Kustomize overlays, Helm values, env vars, config service)
🔴 Production deploys only via pipeline/GitOps with protected environments and audit trail
🟠 Ephemeral preview environments per PR (namespaces, Vercel/Netlify previews, ephemeral stacks) with automatic teardown
🟠 Staging parity: same versions, topology, and deployment mechanism as production
🟠 Promotion gates codified (tests passed, scans clean, SLO healthy, approvals) — not tribal knowledge
```

### Protected environments (GitHub)

```text
Settings → Environments → production:
  - Required reviewers (e.g. on-call lead) for risky services; or wait timer
  - Deployment branches/tags: only main or release tags
  - Environment secrets / OIDC role scoped to production
```

---

## 11. Database Migrations in Pipelines

```text
Order of operations for a release that changes schema (expand/contract — chapter 10 §10.3):
  1. Pipeline runs migration job (EXPAND) against the target environment — separate step, with lock_timeout
  2. Deploy application version compatible with old AND new schema
  3. Background backfill job (if needed), monitored
  4. Later release: deploy app that no longer uses old schema
  5. CONTRACT migration in a subsequent release
```

| Rule | Status |
|---|:---:|
| Migrations run once per deploy as a dedicated job/step (Kubernetes Job, pipeline step, Argo CD PreSync hook), not on every app replica start | 🔴 |
| Migration linting in CI (squawk, Atlas lint) | 🟠 |
| Migration dry-run/test against production-like data in staging | 🟠 |
| Backups/PITR verified before risky migrations | 🔴 |
| Rollback plan documented (usually roll-forward fix) | 🔴 |

---

## 12. Feature Flags in Delivery

| Use | Delivery benefit |
|---|---|
| Release toggles | Merge to main early (trunk-based), deploy dark, release when ready |
| Canary by cohort | Expose to staff → beta users → % of users |
| Kill switches | Turn off a feature without redeploying |
| Ops toggles | Degrade gracefully under load |

Rules (owners, expiry, safe defaults, cleanup): chapter 05 §18.2. Tools: LaunchDarkly, Unleash, Flagsmith, ConfigCat, GrowthBook, Statsig, cloud services; vendor-neutral API via **OpenFeature**.

---

## 13. Pipeline Security and Supply Chain

Full checklist: chapter 09 §16 and §19. Pipeline-specific essentials:

```text
🔴 OIDC federation from CI to cloud (no long-lived cloud keys in CI secrets)
🔴 Least-privilege job permissions (GitHub `permissions:`, GitLab job token scopes, Bitbucket deployment permissions)
🔴 Third-party actions/orbs/components pinned by full SHA or immutable version; reviewed before adoption
🔴 Untrusted fork PRs run without secrets; privileged triggers (pull_request_target, workflow_run) used with extreme care
🔴 Never interpolate untrusted input (PR titles, branch names, issue text) directly into shell — pass via env variables
🔴 Protected branches/environments; CODEOWNERS on pipeline files
🔴 Artifacts signed; SBOM + provenance (SLSA) generated; deploy-time verification (admission policies)
🟠 Hardened, ephemeral runners; egress restrictions on runners for sensitive builds (e.g. Harden-Runner style egress policies)
🟠 Scan pipeline definitions (actionlint, zizmor, OpenSSF Scorecard)
🟠 Audit logs of pipeline runs and deployments retained
```

```yaml
# Safe handling of untrusted input in GitHub Actions
- name: Check PR title
  env:
    TITLE: ${{ github.event.pull_request.title }}   # passed as data, not interpolated into the script body
  run: |
    if ! printf '%s' "$TITLE" | grep -Eq '^(feat|fix|docs|chore|refactor|perf|test|build|ci)(\(.+\))?!?: .+'; then
      echo "PR title must follow Conventional Commits"; exit 1
    fi
```

---

## 14. Secrets in CI/CD

| Rule | Implementation |
|---|---|
| Prefer **no secrets**: workload identity / OIDC | Cloud roles assumed via CI OIDC tokens (AWS, Azure, GCP, Vault JWT auth) |
| Scope remaining secrets narrowly | Environment-level secrets (production only in production environment) |
| Mask in logs | Platform masking + never `echo` secrets; disable shell tracing (`set +x`) around secret use |
| Rotate | Automated rotation; rotate immediately when a contributor with access leaves |
| Fetch at runtime | Pull from Vault/secret manager during the job rather than storing copies in CI |
| Separate build and deploy credentials | Build jobs can't deploy; deploy jobs can't publish packages |

```yaml
# GitHub Actions → AWS via OIDC (no stored keys)
permissions:
  id-token: write
  contents: read
steps:
  - uses: aws-actions/configure-aws-credentials@<full-commit-sha>
    with:
      role-to-assume: arn:aws:iam::123456789012:role/gha-orders-deploy-prod
      aws-region: ap-south-1
```

The cloud role's trust policy must restrict the token's `sub` claim (repository, branch/environment) — otherwise any repo could assume it.

---

## 15. Platform Engineering and Golden Paths

**Platform engineering** builds an **internal developer platform (IDP)** that offers self-service, paved roads ("golden paths") for common needs — so product teams get CI/CD, infrastructure, observability, and security by default.

| Platform capability | Examples |
|---|---|
| Service templates / scaffolding | Backstage Software Templates, cookiecutter, Copier, `create-*` CLIs |
| Service catalogue | **Backstage** (CNCF), Port, Cortex, OpsLevel — owners, docs, APIs, SLOs, dependencies |
| CI/CD templates | Reusable workflows (GitHub), CI/CD components/includes (GitLab), pipes (Bitbucket) |
| Infrastructure self-service | Terraform/OpenTofu modules, Crossplane compositions, cloud landing zones |
| Runtime platform | Kubernetes with standard add-ons, or managed containers/serverless |
| Observability by default | OTel auto-instrumentation, standard dashboards/alerts per template |
| Security by default | Pre-configured scanning, signing, policies |
| Documentation | TechDocs (docs-as-code in Backstage) |

```text
Treat the platform as a PRODUCT: internal customers, roadmap, user research, adoption metrics,
and paved roads that are easier than going off-road (not mandated by decree).
```

Reusable workflow (GitHub) example:

```yaml
# .github/workflows/service-ci.yml in org/platform-workflows
on:
  workflow_call:
    inputs:
      service: { type: string, required: true }
      node-version: { type: string, default: '24' }
jobs:
  ci:
    runs-on: ubuntu-latest
    permissions: { contents: read }
    steps:
      - uses: actions/checkout@<full-commit-sha>
      - uses: actions/setup-node@<full-commit-sha>
        with: { node-version: '${{ inputs.node-version }}', cache: npm }
      - run: npm ci && npm run lint && npm test
```

```yaml
# Consumer repo
jobs:
  ci:
    uses: org/platform-workflows/.github/workflows/service-ci.yml@<full-commit-sha-or-tag>
    with: { service: orders }
```

---

## 16. CI/CD Platforms Compared

| Platform | Model | Strengths | Consider |
|---|---|---|---|
| **GitHub Actions** | YAML workflows, marketplace actions, hosted/self-hosted runners | Tight GitHub integration, OIDC, environments, merge queue, artifact attestations | Third-party action supply-chain risk; pin actions |
| **GitLab CI/CD** | `.gitlab-ci.yml`, components/catalog, runners | Single DevSecOps platform, merge trains, environments, built-in security scanners (tier-dependent) | Self-managed ops if not SaaS |
| **Bitbucket Pipelines** | `bitbucket-pipelines.yml`, pipes, deployments | Jira integration, simple setup | Fewer advanced features; runner/minute limits per plan |
| **Azure Pipelines** (Azure DevOps) | YAML pipelines, environments, approvals | Enterprise Microsoft integration | — |
| **Jenkins** | Self-hosted, Groovy pipelines, plugins | Ultimate flexibility, on-prem | Plugin sprawl, security maintenance burden |
| **CircleCI / Buildkite / Harness / TeamCity** | Hosted/hybrid | Performance, scale, enterprise features | Additional vendor |
| **Tekton / Argo Workflows** | Kubernetes-native pipelines | Cloud-native, composable | Build your own UX |
| **Argo CD / Flux** | GitOps CD (not CI) | Declarative continuous deployment | Pair with any CI |

Platform setup and account configuration: **42-Git-GitHub-GitLab-Bitbucket-Setup.md**.

---

## 17. Reference Pipeline — GitHub Actions

```yaml
# .github/workflows/orders.yml
name: orders
on:
  pull_request:
  push:
    branches: [main]
  merge_group:

permissions:
  contents: read

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ github.event_name == 'pull_request' }}

env:
  IMAGE: ghcr.io/shopnow/orders

jobs:
  quality:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - uses: actions/checkout@<full-commit-sha>
      - uses: actions/setup-java@<full-commit-sha>
        with: { distribution: temurin, java-version: '25', cache: gradle }
      - run: ./gradlew spotlessCheck check --no-daemon        # format, lint, unit + Testcontainers integration tests
      - uses: actions/upload-artifact@<full-commit-sha>
        if: always()
        with: { name: test-reports, path: build/reports/ }

  security:
    runs-on: ubuntu-latest
    permissions: { contents: read, security-events: write }
    steps:
      - uses: actions/checkout@<full-commit-sha>
      - uses: github/codeql-action/init@<full-commit-sha>
        with: { languages: java-kotlin }
      - uses: github/codeql-action/analyze@<full-commit-sha>
      - name: Dependency & secret scan
        run: |
          # e.g. osv-scanner / trivy fs / gitleaks — pin tool versions in a setup step
          echo "run scanners here"

  build-and-publish:
    needs: [quality, security]
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    permissions: { contents: read, packages: write, id-token: write, attestations: write }
    outputs:
      digest: ${{ steps.push.outputs.digest }}
    steps:
      - uses: actions/checkout@<full-commit-sha>
      - uses: docker/setup-buildx-action@<full-commit-sha>
      - uses: docker/login-action@<full-commit-sha>
        with: { registry: ghcr.io, username: ${{ github.actor }}, password: ${{ secrets.GITHUB_TOKEN }} }
      - id: push
        uses: docker/build-push-action@<full-commit-sha>
        with:
          push: true
          tags: ${{ env.IMAGE }}:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          sbom: true
          provenance: mode=max
      - uses: actions/attest-build-provenance@<full-commit-sha>
        with: { subject-name: '${{ env.IMAGE }}', subject-digest: '${{ steps.push.outputs.digest }}', push-to-registry: true }

  deploy-staging:
    needs: build-and-publish
    runs-on: ubuntu-latest
    environment: staging
    permissions: { contents: read, id-token: write }
    steps:
      - uses: actions/checkout@<full-commit-sha>
      - name: Run DB migrations (expand)
        run: ./scripts/migrate.sh staging
      - name: Deploy digest to staging
        run: ./scripts/deploy.sh staging "${{ env.IMAGE }}@${{ needs.build-and-publish.outputs.digest }}"
      - name: Smoke tests
        run: ./scripts/smoke.sh https://staging.api.shopnow.example

  deploy-production:
    needs: [build-and-publish, deploy-staging]
    runs-on: ubuntu-latest
    environment: production            # required reviewers / wait timer configured in repo settings
    permissions: { contents: read, id-token: write }
    steps:
      - uses: actions/checkout@<full-commit-sha>
      - run: ./scripts/migrate.sh production
      - run: ./scripts/deploy.sh production "${{ env.IMAGE }}@${{ needs.build-and-publish.outputs.digest }}"   # canary via Argo Rollouts or GitOps PR
      - run: ./scripts/verify.sh production                                                                     # SLO/error-rate checks; fail → rollback
```

> Replace `<full-commit-sha>` with pinned SHAs (Dependabot/Renovate can keep them updated). `deploy.sh` might open a GitOps PR, call Argo CD, update an ECS service, or SSH to a VPS — see §25.

---

## 18. Reference Pipelines — GitLab CI and Bitbucket Pipelines

### 18.1 GitLab CI

```yaml
# .gitlab-ci.yml
stages: [verify, build, deploy]

default:
  image: eclipse-temurin:25-jdk
  interruptible: true

variables:
  IMAGE: $CI_REGISTRY_IMAGE/orders

include:
  - template: Jobs/SAST.gitlab-ci.yml                 # availability of scanners depends on tier
  - template: Jobs/Secret-Detection.gitlab-ci.yml

test:
  stage: verify
  services: [docker:dind]                             # if Testcontainers needs Docker; or use a runner with Docker socket policy
  script: ./gradlew spotlessCheck check --no-daemon
  artifacts:
    when: always
    reports: { junit: build/test-results/test/*.xml }

build-image:
  stage: build
  image: docker:27
  services: [docker:27-dind]
  rules: [{ if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH' }]
  script:
    - docker login -u "$CI_REGISTRY_USER" -p "$CI_REGISTRY_PASSWORD" "$CI_REGISTRY"
    - docker build -t "$IMAGE:$CI_COMMIT_SHA" .
    - docker push "$IMAGE:$CI_COMMIT_SHA"

deploy-staging:
  stage: deploy
  environment: { name: staging, url: https://staging.api.shopnow.example }
  rules: [{ if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH' }]
  script: ./scripts/deploy.sh staging "$IMAGE:$CI_COMMIT_SHA"

deploy-production:
  stage: deploy
  environment: { name: production, url: https://api.shopnow.example }
  rules: [{ if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH', when: manual }]   # protected environment approvals
  script: ./scripts/deploy.sh production "$IMAGE:$CI_COMMIT_SHA"
```

GitLab features worth using: **merge trains**, protected environments with approvals, CI/CD components catalog, OIDC ID tokens (`id_tokens:`) for cloud auth, environments/deployments tracking.

### 18.2 Bitbucket Pipelines

```yaml
# bitbucket-pipelines.yml
image: eclipse-temurin:25-jdk

definitions:
  caches:
    gradlewrapper: ~/.gradle/wrapper
  steps:
    - step: &test
        name: Test
        caches: [gradle, gradlewrapper]
        services: [docker]
        script:
          - ./gradlew spotlessCheck check --no-daemon

pipelines:
  pull-requests:
    '**':
      - step: *test
  branches:
    main:
      - step: *test
      - step:
          name: Build & push image
          services: [docker]
          script:
            - docker build -t $REGISTRY/orders:$BITBUCKET_COMMIT .
            - echo $REGISTRY_PASSWORD | docker login $REGISTRY -u $REGISTRY_USER --password-stdin
            - docker push $REGISTRY/orders:$BITBUCKET_COMMIT
      - step:
          name: Deploy to staging
          deployment: staging
          script: [./scripts/deploy.sh staging $REGISTRY/orders:$BITBUCKET_COMMIT]
      - step:
          name: Deploy to production
          deployment: production
          trigger: manual
          script: [./scripts/deploy.sh production $REGISTRY/orders:$BITBUCKET_COMMIT]
```

Bitbucket Pipelines supports OIDC for cloud deployments and deployment permissions per environment — use them instead of stored cloud keys where possible.

---

## 19. Monorepo CI

```text
🔴 Affected-only builds/tests: Nx affected, Turborepo filters, Bazel/Pants target selection, Gradle configuration
    cache + build cache, or path filters (dorny/paths-filter, GitLab rules:changes)
🔴 Remote build cache shared across CI and developers
🟠 Per-project pipelines triggered by paths; required checks computed dynamically (aggregate status job)
🟠 Separate deploy pipelines per deployable unit; independent release cadence
🟠 Ownership of pipeline config per path (CODEOWNERS)
```

```yaml
# Aggregate "required" job so branch protection doesn't depend on dynamic job names
ci-ok:
  if: always()
  needs: [orders, payments, web]
  runs-on: ubuntu-latest
  steps:
    - run: |
        [[ "${{ contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled') }}" == "false" ]]
```

---

## 20. Deployment Verification and Automated Rollback

### 20.1 Post-deploy verification

| Check | Example |
|---|---|
| Smoke tests | Health endpoints, a synthetic login + key read, version endpoint returns new SHA |
| Golden signals | Error rate, latency p95/p99, saturation vs baseline (canary vs stable) |
| SLO burn rate | Fast burn alerts during rollout (chapter 18) |
| Business KPIs | Orders/minute, payment success rate unchanged |
| Logs | New error types after deploy |

### 20.2 Automated rollback triggers

```text
- Canary analysis fails (error rate/latency thresholds) → abort rollout, shift traffic back
- Readiness never succeeds within progress deadline → Kubernetes stops rollout (progressDeadlineSeconds)
- Post-deploy smoke fails → pipeline redeploys previous digest
- SLO fast-burn alert within N minutes of deploy → page + auto-rollback (if policy allows)
```

```text
Rollback ≠ always possible: schema contracts, data migrations, external side effects.
That's why expand/contract migrations and feature flags matter — prefer disabling the flag or rolling forward.
```

### 20.3 Deployment records

Every deployment emits an event (service, version/digest, environment, actor, time, change ID) to observability (deploy markers on dashboards) and to DORA metrics collection.

---

## 21. Change Management

| Change type (ITIL 4 practice) | Description | Approval |
|---|---|---|
| **Standard** | Pre-authorised, low-risk, repeatable (most automated deployments that pass the pipeline) | Pipeline gates = approval |
| **Normal** | Assessed and authorised per change (major migrations, infra changes, risky releases) | Peer/CAB review proportionate to risk |
| **Emergency** | Must be implemented ASAP (incident fixes) | Expedited, reviewed retrospectively |

```text
Modern approach: make the pipeline the change-management system —
PR (change request) + automated tests/scans (evidence) + approvals (CODEOWNERS/environment reviewers)
+ deployment record (implementation evidence) + monitoring (verification).
Avoid weekly CAB meetings for standard changes; DORA research associates heavyweight external approval with worse performance.
```

### Change freeze policy (example)

```text
- Freezes only for known high-risk periods (festival sales, financial year-end close), announced in advance
- During freezes: only emergency changes; feature flags remain operable
- Keep deploying small changes up to the freeze, not a big batch right before
```

---

## 22. Measuring Delivery

| Metric | Source |
|---|---|
| DORA five (lead time, deploy frequency, failed deployment recovery time, change fail rate, deployment rework rate) | VCS + CI/CD + incident tooling (chapter 01 §17) |
| Pipeline duration (p50/p95) and queue time | CI platform |
| Pipeline success rate and flaky failure rate | CI platform + test analytics |
| PR cycle time (open → merge) and review wait time | VCS analytics |
| Time from merge to production | Deploy records |
| Developer experience | Surveys (friction, satisfaction) — SPACE framework dimensions |

Tools: DORA's Four Keys-style pipelines, Sleuth, LinearB, Swarmia, Faros, GitLab Value Stream Analytics, Apache DevLake.

---

## 23. Runners and Build Infrastructure

| Option | Pros | Cons |
|---|---|---|
| Hosted runners (GitHub/GitLab/Bitbucket) | Zero maintenance, ephemeral, secure by default | Cost at scale; limited hardware; egress from shared IPs |
| Larger hosted runners / ARM runners | Faster builds, native ARM64 images | Cost |
| Self-hosted ephemeral (Actions Runner Controller on Kubernetes, GitLab Runner autoscaling, cloud VMs per job) | Control, private network access, cost at scale | You maintain and secure them |
| Self-hosted persistent | Caches warm | **Security risk** — state leaks between jobs; avoid for untrusted code |

```text
🔴 Never use self-hosted runners for public repositories' untrusted PRs
🔴 Ephemeral runners (one job per runner) for sensitive workloads
🟠 Runner groups per trust level (prod deploy runners isolated)
🟠 Patch runner images; restrict egress
```

---

## 24. CI/CD Cost Optimisation

```text
- Cancel superseded runs (concurrency groups)
- Affected-only builds; caching (dependencies, Docker layers, build caches)
- Right-size runners: bigger runners can be cheaper if they cut minutes significantly
- ARM runners where workloads support them
- Path filters: docs-only changes skip heavy jobs
- Schedule heavy suites (full regression, load tests) nightly instead of per PR
- Retention: prune old artifacts, caches, preview environments
- Track cost per pipeline/team; set budgets
```

---

## 25. Scenario Playbooks

### 25.1 Single VPS (startup) — GitHub Actions + SSH deploy
Build and test in CI → build image or artifact → push to registry → deploy job SSHes (deploy key restricted to a deploy user; `command=` restrictions in `authorized_keys`) → `docker compose pull && up -d` or PM2 reload → health check → rollback by redeploying previous tag. Server setup, Nginx, PM2, and backups: **39-vps-enterprise-setup-guide.md**.

### 25.2 Kubernetes with GitOps
CI builds/signs images → bot PR updates config repo digest → Argo CD syncs staging → automated tests → promotion PR to production → Argo Rollouts canary with Prometheus analysis → automatic abort on regression → `git revert` for rollback.

### 25.3 Serverless (AWS Lambda / Cloud Run / Azure Functions)
IaC (SAM/CDK/Terraform/OpenTofu) in the same repo → CI builds and tests → deploy to staging stack → integration tests → production with traffic shifting (Lambda aliases + CodeDeploy canary, Cloud Run revisions with traffic split) → alarms trigger automatic rollback.

### 25.4 Mobile apps
CI builds signed artifacts → internal track/TestFlight → staged rollout with crash/ANR gates (chapter 14 §19–20).

### 25.5 Regulated enterprise
Pipeline as the change record: linked ticket IDs, required approvals with segregation of duties, signed artifacts, evidence archive (test reports, scans, SBOMs, provenance), protected environments, audit log export to SIEM.

---

## 26. Checklists

### CI
- [ ] Pipeline as code; triggered on every PR/push; merge queue where busy
- [ ] PR feedback ≤ 15 minutes; caching configured; superseded runs cancelled
- [ ] Format/lint/type/unit/integration/contract tests; coverage on changed lines
- [ ] SAST, SCA, secret scanning, IaC/container scanning
- [ ] Main always green; broken main fixed immediately

### Build & artifacts
- [ ] Reproducible builds; pinned toolchains, base images, actions
- [ ] Immutable artifacts deployed by digest; SBOM + provenance + signature
- [ ] Registry scanning and retention policies

### CD
- [ ] Same artifact promoted through environments; config separated
- [ ] Migrations as a separate, idempotent step (expand/contract)
- [ ] Progressive delivery (canary/blue-green) with automated analysis
- [ ] Feature flags decouple deploy from release
- [ ] Post-deploy verification and automatic rollback
- [ ] Deployment events recorded (dashboards, DORA metrics)

### Security & governance
- [ ] OIDC to cloud; no long-lived keys; least-privilege job permissions
- [ ] Protected environments with reviewers; CODEOWNERS on pipeline files
- [ ] Untrusted input never interpolated into scripts; fork PRs without secrets
- [ ] Ephemeral runners for sensitive jobs
- [ ] Change types defined; emergency changes reviewed retrospectively

---

## 27. References

### Research and principles
- DORA capabilities: https://dora.dev/capabilities/
- DORA metrics: https://dora.dev/guides/dora-metrics/
- Continuous Delivery (Humble & Farley) site: https://continuousdelivery.com/
- Martin Fowler — Continuous Integration: https://martinfowler.com/articles/continuousIntegration.html
- OpenGitOps principles: https://opengitops.dev/
- CNCF Platforms white paper: https://tag-app-delivery.cncf.io/whitepapers/platforms/

### Platforms and tools
- GitHub Actions: https://docs.github.com/actions · Security hardening: https://docs.github.com/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions
- GitHub OIDC: https://docs.github.com/actions/security-for-github-actions/security-hardening-your-deployments/about-security-hardening-with-openid-connect
- GitHub artifact attestations: https://docs.github.com/actions/security-for-github-actions/using-artifact-attestations
- GitLab CI/CD: https://docs.gitlab.com/ci/ · Bitbucket Pipelines: https://support.atlassian.com/bitbucket-cloud/docs/get-started-with-bitbucket-pipelines/
- Argo CD: https://argo-cd.readthedocs.io/ · Argo Rollouts: https://argoproj.github.io/rollouts/ · Flux: https://fluxcd.io/ · Flagger: https://flagger.app/
- Backstage: https://backstage.io/
- OpenFeature: https://openfeature.dev/
- actionlint: https://github.com/rhysd/actionlint · zizmor: https://docs.zizmor.sh/
- SLSA: https://slsa.dev/ · Sigstore cosign: https://docs.sigstore.dev/

### Change management
- ITIL 4 change enablement (PeopleCert / AXELOS): https://www.peoplecert.org/
- Accelerate (Forsgren, Humble, Kim) — research basis for delivery performance

---

**Previous:** [15 — Desktop Engineering](./15-desktop-engineering.md) · **Next:** [17 — Cloud & Infrastructure](./17-cloud-and-infrastructure.md)