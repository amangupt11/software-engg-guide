# ☁️ Cloud & Infrastructure — Production Engineering Guide

> How to design, provision, secure, and run infrastructure for production systems: cloud service models and shared responsibility, choosing providers and regions (incl. India data residency), landing zones and account structure, networking, compute options, containers and **Kubernetes** (versions, Gateway API after the Ingress NGINX retirement, autoscaling, reliability), storage, Infrastructure as Code (Terraform/OpenTofu, Pulumi, CDK, Bicep, Crossplane) with state, modules, CI, drift, and policy as code, configuration management, multi-region patterns, FinOps cost management, sustainability, edge/CDN, hybrid and sovereign options, and reference setups from a single VPS to multi-region Kubernetes.
>
> Related: [04 Cloud-native architecture §13](./04-software-architecture.md#13-cloud-native-architecture) · [09 Cloud & Kubernetes security §17–18](./09-security-engineering.md) · [10 Managed databases](./10-database-engineering.md) · [16 CI/CD & GitOps](./16-devops-and-ci-cd.md) · [18 Observability](./18-observability-and-monitoring.md) · [20 Reliability & DR](./20-reliability-and-disaster-recovery.md) · [39 VPS enterprise setup](./39-vps-enterprise-setup-guide.md)

---

## 📚 Table of Contents

- [☁️ Cloud \& Infrastructure — Production Engineering Guide](#️-cloud--infrastructure--production-engineering-guide)
  - [📚 Table of Contents](#-table-of-contents)
  - [1. Principles](#1-principles)
  - [2. Service Models and Shared Responsibility](#2-service-models-and-shared-responsibility)
  - [3. Choosing Providers and Regions](#3-choosing-providers-and-regions)
    - [3.1 Provider selection factors](#31-provider-selection-factors)
    - [3.2 India regions (examples — verify current availability)](#32-india-regions-examples--verify-current-availability)
  - [4. Landing Zones and Account Structure](#4-landing-zones-and-account-structure)
  - [5. Networking](#5-networking)
    - [5.1 VPC/VNet design](#51-vpcvnet-design)
    - [5.2 Connectivity patterns](#52-connectivity-patterns)
    - [5.3 Load balancing](#53-load-balancing)
  - [6. Compute Options](#6-compute-options)
  - [7. Kubernetes in Production](#7-kubernetes-in-production)
    - [7.1 Versions and upgrades](#71-versions-and-upgrades)
    - [7.2 Cluster baseline](#72-cluster-baseline)
    - [7.3 Workload manifest (production-ready baseline)](#73-workload-manifest-production-ready-baseline)
    - [7.4 Packaging manifests](#74-packaging-manifests)
  - [8. Kubernetes Traffic — Gateway API and Ingress](#8-kubernetes-traffic--gateway-api-and-ingress)
    - [8.1 Ingress NGINX retirement](#81-ingress-nginx-retirement)
    - [8.2 Gateway API model](#82-gateway-api-model)
  - [9. Kubernetes Scaling and Reliability](#9-kubernetes-scaling-and-reliability)
  - [10. Storage](#10-storage)
  - [11. Infrastructure as Code](#11-infrastructure-as-code)
    - [IaC rules](#iac-rules)
  - [12. Terraform / OpenTofu in Practice](#12-terraform--opentofu-in-practice)
    - [12.1 Repository structure](#121-repository-structure)
    - [12.2 Backend and providers](#122-backend-and-providers)
    - [12.3 Module example](#123-module-example)
    - [12.4 CI workflow for IaC](#124-ci-workflow-for-iac)
    - [12.5 Safety rules](#125-safety-rules)
  - [13. Policy as Code and Drift](#13-policy-as-code-and-drift)
    - [Drift management](#drift-management)
  - [14. Configuration Management for VMs](#14-configuration-management-for-vms)
  - [15. Cloud Identity and Access](#15-cloud-identity-and-access)
  - [16. High Availability and Multi-Region](#16-high-availability-and-multi-region)
  - [17. Edge, CDN, and DNS](#17-edge-cdn-and-dns)
  - [18. FinOps — Cost Management](#18-finops--cost-management)
    - [18.1 Inform (visibility)](#181-inform-visibility)
    - [18.2 Optimize](#182-optimize)
    - [18.3 Operate](#183-operate)
  - [19. Sustainability](#19-sustainability)
  - [20. Hybrid, On-Prem, and Sovereign Cloud](#20-hybrid-on-prem-and-sovereign-cloud)
  - [21. Reference Setups](#21-reference-setups)
    - [21.1 Stage 1 — Single VPS (MVP)](#211-stage-1--single-vps-mvp)
    - [21.2 Stage 2 — Managed PaaS](#212-stage-2--managed-paas)
    - [21.3 Stage 3 — Kubernetes platform (multiple teams)](#213-stage-3--kubernetes-platform-multiple-teams)
    - [21.4 Stage 4 — Multi-region (high availability / residency)](#214-stage-4--multi-region-high-availability--residency)
  - [22. Checklists](#22-checklists)
    - [Foundation](#foundation)
    - [IaC](#iac)
    - [Kubernetes](#kubernetes)
    - [Resilience \& cost](#resilience--cost)
  - [23. References](#23-references)
    - [Cloud frameworks and docs](#cloud-frameworks-and-docs)
    - [Kubernetes](#kubernetes-1)
    - [IaC and policy](#iac-and-policy)
    - [FinOps](#finops)

---

## 1. Principles

| # | Principle |
|---:|---|
| 1 | **Everything as code** — infrastructure, policies, dashboards; no click-ops in production. |
| 2 | **Immutable infrastructure** — replace, don't patch in place; rebuild from code. |
| 3 | **Least privilege and zero trust** — identities, networks, and data access minimised. |
| 4 | **Design for failure** — multi-AZ by default; know your RPO/RTO. |
| 5 | **Managed services for undifferentiated work** — but understand their limits and exit costs. |
| 6 | **Cost is a design constraint** — tagging, budgets, and unit economics from day one. |
| 7 | **Environments are cattle** — any non-production environment can be destroyed and recreated. |
| 8 | **Boring and supported versions** — stay within provider and upstream support windows. |

---

## 2. Service Models and Shared Responsibility

| Model | You manage | Provider manages | Examples |
|---|---|---|---|
| **On-prem** | Everything | — | Own data centre |
| **IaaS** | OS, runtime, app, data, network config | Hardware, virtualisation, physical security | EC2, Compute Engine, Azure VMs, VPS providers |
| **CaaS / managed Kubernetes** | Workloads, cluster config, node pools (varies) | Control plane (and nodes in autopilot modes) | EKS, GKE (Standard/Autopilot), AKS |
| **PaaS / serverless containers** | App code and config | Runtime, scaling, OS | Cloud Run, App Runner, Azure Container Apps, App Service, Heroku-style platforms |
| **FaaS** | Function code | Everything else | Lambda, Cloud Functions/Cloud Run functions, Azure Functions |
| **SaaS** | Configuration, data, access | Application | Managed Postgres, auth providers, observability SaaS |

```text
Shared responsibility rule: the provider secures the cloud; YOU secure what you put in it —
identities, data, network configuration, workload hardening, and application security — at every model.
```

---

## 3. Choosing Providers and Regions

### 3.1 Provider selection factors

| Factor | Questions |
|---|---|
| Team skills and ecosystem | Which provider does the team know? Existing enterprise agreements? |
| Services needed | Managed Kubernetes, databases, AI/ML, analytics, messaging maturity |
| Regions and residency | Regions where users and regulators require data to live |
| Compliance | Certifications (ISO 27001, SOC 2, PCI DSS, MeitY empanelment for Indian government workloads) |
| Cost model | Pricing, egress charges, commitments, credits |
| Support | Support plans, partner ecosystem |
| Lock-in and exit | Proprietary services vs portable ones; data egress cost for migration |

### 3.2 India regions (examples — verify current availability)

| Provider | Regions in India |
|---|---|
| AWS | Mumbai (`ap-south-1`), Hyderabad (`ap-south-2`) |
| Microsoft Azure | Central India (Pune), South India (Chennai), West India (Mumbai) and newer regions as announced |
| Google Cloud | Mumbai (`asia-south1`), Delhi (`asia-south2`) |
| Oracle Cloud | Mumbai, Hyderabad |
| Others | Indian cloud/VPS providers and data-centre operators for local hosting needs |

```text
Region choice checklist:
- Latency to users (India-wide users: Mumbai/Delhi/Hyderabad regions; global users: add regions or CDN/edge)
- Data residency/localisation rules (e.g. RBI payment data localisation; sector/government requirements; DPDP cross-border rules)
- Service availability in the region (not all services/instance types launch everywhere)
- DR pairing: second region within the same country when residency requires
- Price differences between regions
```

---

## 4. Landing Zones and Account Structure

A **landing zone** is a pre-configured, secure, multi-account (or multi-subscription/project) foundation.

```text
Organisation / Management group / Org node
├── Security OU          → log archive account (immutable audit logs), security tooling account
├── Infrastructure OU    → networking hub (transit/shared VPC, DNS, egress firewall), shared services (CI runners, artifact registry)
├── Workloads OU
│   ├── prod             → one account/subscription/project per workload or team for blast-radius isolation
│   ├── staging
│   └── dev
├── Sandbox OU           → experimentation with budgets and auto-cleanup
└── Suspended OU         → decommissioned accounts
```

| Building block | AWS | Azure | Google Cloud |
|---|---|---|---|
| Hierarchy | Organizations, OUs, accounts | Management groups, subscriptions, resource groups | Organization, folders, projects |
| Guardrails | SCPs, RCPs, Control Tower controls | Azure Policy, Blueprints successor patterns | Organization Policy constraints |
| Landing zone tooling | Control Tower, Landing Zone Accelerator | Azure Landing Zones (CAF) | Cloud Foundation Fabric / setup checklist |
| Central audit | CloudTrail org trail | Activity Log + diagnostic settings to Log Analytics | Cloud Audit Logs aggregated sinks |
| Identity | IAM Identity Center | Entra ID | Cloud Identity / Workforce Identity Federation |

```text
🔴 Separate production from non-production at the account/subscription/project level
🔴 Central, immutable audit logging; break-glass access procedure
🔴 Guardrails: deny disabling audit logs, deny public storage, restrict regions (data residency), require encryption
🟠 Budgets and alerts per account; mandatory tags
🟠 Account vending automated (IaC) for new teams/workloads
```

---

## 5. Networking

### 5.1 VPC/VNet design

```text
Region: ap-south-1
VPC 10.20.0.0/16 (plan non-overlapping CIDRs across all VPCs, on-prem, and partner networks)
├── Public subnets   (per AZ, /24)  → load balancers, NAT gateways only
├── Private app      (per AZ, /20)  → compute, Kubernetes nodes/pods
├── Private data     (per AZ, /24)  → databases, caches (no internet route)
└── Endpoint subnet                 → private endpoints/interface endpoints for managed services
```

| Rule | Status |
|---|:---:|
| Plan an IP address management (IPAM) scheme; avoid overlapping CIDRs (painful to fix later) | 🔴 |
| Workloads in private subnets; only load balancers public | 🔴 |
| Databases unreachable from the internet | 🔴 |
| Private endpoints / VPC endpoints for cloud services (storage, secrets, registries) to avoid NAT costs and public paths | 🟠 |
| Egress control: NAT with logging, egress firewall/proxy for sensitive workloads | 🟠 |
| Kubernetes pod CIDR planning (VPC-native/CNI IP consumption) | 🟠 |
| IPv6 / dual-stack plan where providers and clients support it | 🟡 |

### 5.2 Connectivity patterns

| Need | Options |
|---|---|
| VPC ↔ VPC | Peering (small scale), transit gateway / hub-and-spoke / shared VPC (scale) |
| Cloud ↔ on-prem | Site-to-site VPN (quick), dedicated interconnect (Direct Connect, ExpressRoute, Cloud Interconnect) |
| Service exposure to other accounts/customers | PrivateLink / Private Service Connect / Private Link Service |
| Zero-trust access for humans | Identity-aware proxies (IAP, Verified Access, Entra Private Access), Tailscale/WireGuard-based meshes, bastion-less SSM/SSH via identity |

### 5.3 Load balancing

| Layer | Use |
|---|---|
| L4 (TCP/UDP) | Non-HTTP protocols, extreme throughput, static IPs |
| L7 (HTTP/HTTPS) | Path/host routing, TLS termination, WAF integration, HTTP/2 and HTTP/3 |
| Global / anycast | Multi-region failover, edge TLS termination |
| Internal | Service-to-service within VPC |

---

## 6. Compute Options

| Option | Choose when | Watch out for |
|---|---|---|
| **VPS / single VM** | Early-stage, predictable load, cost-sensitive (see **39**) | Single point of failure; manual ops |
| **VM groups with autoscaling** | Legacy apps, special OS needs | Image pipelines (Packer), patching |
| **Managed containers** (Cloud Run, App Runner, Azure Container Apps, ECS on Fargate) | Most stateless web services without needing Kubernetes control | Platform limits; per-request pricing at scale |
| **Managed Kubernetes** (EKS, GKE, AKS) | Many services, platform team, portability, advanced scheduling | Operational complexity; upgrade cadence |
| **Serverless functions** | Event-driven glue, bursty workloads | Cold starts, time limits, connection management |
| **Batch / HPC** | Large parallel jobs | Spot interruptions; job orchestration |
| **GPU instances / managed AI endpoints** | ML training/inference | Cost, capacity availability, quotas |

```text
Decision ladder: managed PaaS/serverless containers → managed Kubernetes → self-managed Kubernetes/VMs
Climb only when you need the control — each step adds operational burden.
```

---

## 7. Kubernetes in Production

### 7.1 Versions and upgrades

Kubernetes ships **about three minor releases per year** (v1.36 was released on 22 April 2026), and each minor release is supported upstream for roughly **14 months** (≈ 12 months of patches plus a maintenance period). Managed providers have their own support windows (standard and paid extended support).

```text
🔴 Stay within supported versions; plan an upgrade every ~4 months or at least twice a year
🔴 Check deprecated/removed APIs before upgrading (kubent, pluto, provider upgrade insights)
🟠 Upgrade order: control plane → add-ons (CNI, CSI, DNS, gateway controllers) → node pools (surge/blue-green)
🟠 Test upgrades in staging clusters with the same add-on versions
```

### 7.2 Cluster baseline

| Area | Baseline |
|---|---|
| Tenancy | Namespaces per team/app; ResourceQuotas and LimitRanges; separate clusters for prod vs non-prod (and for strong isolation needs) |
| Security | Pod Security Admission (`restricted`), RBAC least privilege, NetworkPolicies default-deny, image signature verification, secrets encryption (KMS) — chapter 09 §17 |
| Networking | CNI (VPC-native / Cilium), **Gateway API** for north-south traffic (§8), optional service mesh for mTLS and traffic policy |
| Certificates | cert-manager with ACME/private CA |
| DNS | external-dns syncing records |
| Secrets | External Secrets Operator / Secrets Store CSI with a cloud secret manager |
| Observability | OTel Collector (DaemonSet + gateway), Prometheus-compatible metrics, logs pipeline, kube-state-metrics (chapter 18) |
| Policy | Kyverno / OPA Gatekeeper / ValidatingAdmissionPolicy |
| GitOps | Argo CD or Flux (chapter 16 §9) |
| Backup | Velero (cluster resources + volume snapshots) or provider backup services |
| Node management | Karpenter (AWS and others) or Cluster Autoscaler; node auto-upgrades/auto-repair |

### 7.3 Workload manifest (production-ready baseline)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: orders
  namespace: orders
  labels: { app.kubernetes.io/name: orders, app.kubernetes.io/part-of: shopnow }
spec:
  replicas: 3
  revisionHistoryLimit: 5
  strategy:
    type: RollingUpdate
    rollingUpdate: { maxUnavailable: 0, maxSurge: 25% }
  selector: { matchLabels: { app.kubernetes.io/name: orders } }
  template:
    metadata:
      labels: { app.kubernetes.io/name: orders }
    spec:
      serviceAccountName: orders
      automountServiceAccountToken: false
      terminationGracePeriodSeconds: 40
      securityContext:
        runAsNonRoot: true
        seccompProfile: { type: RuntimeDefault }
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: topology.kubernetes.io/zone
          whenUnsatisfiable: DoNotSchedule
          labelSelector: { matchLabels: { app.kubernetes.io/name: orders } }
      containers:
        - name: orders
          image: ghcr.io/shopnow/orders@sha256:<digest>
          ports: [{ name: http, containerPort: 8080 }]
          envFrom: [{ configMapRef: { name: orders-config } }, { secretRef: { name: orders-secrets } }]
          resources:
            requests: { cpu: "250m", memory: "512Mi" }
            limits:   { memory: "768Mi" }          # memory limit set; CPU limit often omitted to avoid throttling (team policy)
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities: { drop: ["ALL"] }
          startupProbe:   { httpGet: { path: /health/live, port: http }, failureThreshold: 30, periodSeconds: 2 }
          livenessProbe:  { httpGet: { path: /health/live, port: http }, periodSeconds: 10 }
          readinessProbe: { httpGet: { path: /health/ready, port: http }, periodSeconds: 5 }
          lifecycle:
            preStop: { sleep: { seconds: 5 } }    # sleep action (newer Kubernetes); otherwise exec a sleep command
          volumeMounts: [{ name: tmp, mountPath: /tmp }]
      volumes: [{ name: tmp, emptyDir: {} }]
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata: { name: orders, namespace: orders }
spec:
  minAvailable: 2
  selector: { matchLabels: { app.kubernetes.io/name: orders } }
```

### 7.4 Packaging manifests

| Tool | Use |
|---|---|
| **Kustomize** | Overlays per environment, no templating language |
| **Helm** | Packaged, parameterised charts; third-party software |
| Jsonnet / CUE / cdk8s / Timoni | Programmatic config generation |

---

## 8. Kubernetes Traffic — Gateway API and Ingress

### 8.1 Ingress NGINX retirement

Kubernetes SIG Network and the Security Response Committee retired the community **Ingress NGINX** controller: best-effort maintenance ended in **March 2026**, after which it receives no further security fixes. Existing deployments keep running but are increasingly risky. The recommended path is **Gateway API** (or another maintained Ingress controller as an interim step).

> The **Ingress API** itself is not removed — only the community Ingress NGINX controller is retired. Note that this is distinct from F5/NGINX's commercial and open-source NGINX Ingress Controller products, which are separate projects.

```bash
# Check whether a cluster runs the retired controller
kubectl get pods --all-namespaces --selector app.kubernetes.io/name=ingress-nginx
```

### 8.2 Gateway API model

| Resource | Owner | Purpose |
|---|---|---|
| `GatewayClass` | Infrastructure provider | Type of gateway implementation |
| `Gateway` | Cluster/platform operator | Listeners (ports, hostnames, TLS) |
| `HTTPRoute` / `GRPCRoute` / `TLSRoute` | Application team | Routing rules to services |
| `ReferenceGrant` | Namespace owner | Allow cross-namespace references |

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata: { name: public, namespace: gateway }
spec:
  gatewayClassName: <your-implementation>      # e.g. a cloud provider class, Envoy Gateway, Istio, Cilium, Traefik, Kong, NGINX Gateway Fabric
  listeners:
    - name: https
      protocol: HTTPS
      port: 443
      hostname: "api.shopnow.example"
      tls:
        mode: Terminate
        certificateRefs: [{ name: api-shopnow-tls }]
      allowedRoutes: { namespaces: { from: Selector, selector: { matchLabels: { gateway-access: public } } } }
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata: { name: orders, namespace: orders }
spec:
  parentRefs: [{ name: public, namespace: gateway }]
  hostnames: ["api.shopnow.example"]
  rules:
    - matches: [{ path: { type: PathPrefix, value: /v1/orders } }]
      backendRefs: [{ name: orders, port: 80 }]
      timeouts: { request: 10s }
```

Migration aids: `ingress2gateway` (Kubernetes SIGs) converts Ingress resources (including common annotations) to Gateway API resources — review output carefully, since controller-specific annotations rarely map one-to-one.

---

## 9. Kubernetes Scaling and Reliability

| Mechanism | Purpose |
|---|---|
| **Resource requests** | Scheduling and bin-packing; base for HPA CPU utilisation |
| **HPA** (Horizontal Pod Autoscaler) | Scale replicas on CPU/memory/custom/external metrics |
| **KEDA** | Event-driven autoscaling (queue length, Kafka lag, cron), including scale-to-zero |
| **VPA** | Recommend/adjust requests (use in recommendation mode alongside HPA on CPU) |
| **Cluster Autoscaler / Karpenter** | Add/remove nodes based on pending pods |
| **PodDisruptionBudgets** | Keep minimum replicas during voluntary disruptions (upgrades, drains) |
| **Topology spread / anti-affinity** | Spread across zones/nodes |
| **Priority classes** | Protect critical workloads under pressure |

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata: { name: orders, namespace: orders }
spec:
  scaleTargetRef: { apiVersion: apps/v1, kind: Deployment, name: orders }
  minReplicas: 3
  maxReplicas: 30
  metrics:
    - type: Resource
      resource: { name: cpu, target: { type: Utilization, averageUtilization: 65 } }
  behavior:
    scaleUp:   { stabilizationWindowSeconds: 0,   policies: [{ type: Percent, value: 100, periodSeconds: 60 }] }
    scaleDown: { stabilizationWindowSeconds: 300, policies: [{ type: Percent, value: 20,  periodSeconds: 60 }] }
```

```text
Requests/limits guidance:
- Always set requests (CPU, memory) from measured usage; set memory limits to prevent node OOM cascades
- CPU limits can cause throttling of latency-sensitive services; many teams set requests only (policy decision)
- Right-size continuously (VPA recommendations, cost tools)
```

---

## 10. Storage

| Type | Use | Examples |
|---|---|---|
| **Block** | Databases, single-instance stateful apps | EBS, Persistent Disk/Hyperdisk, Azure Managed Disks; Kubernetes PVs via CSI |
| **File (NFS/SMB)** | Shared filesystems, legacy apps | EFS, Filestore, Azure Files |
| **Object** | Files, backups, data lakes, static assets | S3, GCS, Azure Blob, MinIO/compatible stores |
| **Archive** | Long-term retention | Glacier tiers, Archive storage classes |

```text
🔴 Encryption at rest (provider-managed or customer-managed keys) on all volumes and buckets
🔴 Block public access on buckets by default; access via IAM and signed URLs
🔴 Versioning + object lock (WORM) for backups and audit logs (ransomware protection)
🟠 Lifecycle policies: transition to cheaper tiers, expire temporary data
🟠 Snapshots scheduled and restore-tested (chapter 20)
🟠 Prefer managed databases over databases on Kubernetes unless you have strong operator expertise (chapter 10 §2.2)
```

---

## 11. Infrastructure as Code

| Tool | Language | Scope | Notes |
|---|---|---|---|
| **Terraform** (HashiCorp, now part of IBM) | HCL | Multi-cloud, huge provider ecosystem | Business Source Licence (BSL) since Aug 2023 for newer versions — review licence terms for your use |
| **OpenTofu** (Linux Foundation) | HCL | Open-source (MPL-2.0) fork of Terraform | Largely compatible with Terraform configs/providers; adds its own features (e.g. state encryption) |
| **Pulumi** | TypeScript, Python, Go, C#, Java, YAML | Multi-cloud | General-purpose languages, testing |
| **AWS CDK** / CloudFormation | TS/Python/Java/C#/Go → CloudFormation | AWS | Native AWS integration |
| **Bicep** / ARM | Bicep DSL | Azure | Native Azure |
| **Google Cloud Infrastructure Manager** / Config Connector | Terraform-based / Kubernetes CRDs | GCP | |
| **Crossplane** | Kubernetes CRDs + compositions | Multi-cloud via Kubernetes control plane | Platform-team self-service APIs |

### IaC rules

```text
🔴 All production infrastructure defined in code, reviewed via PRs, applied by pipelines
🔴 Remote state with locking and encryption; restricted access (state contains secrets/attributes)
🔴 Separate state per environment and per component (blast radius)
🔴 Pin provider and module versions; upgrade deliberately
🟠 Reusable, versioned modules with documented inputs/outputs
🟠 Plan output reviewed on every PR; apply only from main via CI with approvals for production
🟠 Policy as code checks before apply (§13); drift detection scheduled
```

---

## 12. Terraform / OpenTofu in Practice

### 12.1 Repository structure

```text
infra/
├── modules/                      # reusable modules (versioned; or separate repo/registry)
│   ├── network/
│   ├── eks-cluster/
│   └── postgres/
├── live/                         # environment instantiations (one state each)
│   ├── prod/
│   │   ├── network/main.tf
│   │   ├── cluster/main.tf
│   │   └── data/main.tf
│   └── staging/…
└── policies/                     # OPA/Conftest or Checkov custom policies
```

Tools for orchestration across many states: **Terragrunt**, Terramate, Atmos, or CI matrices.

### 12.2 Backend and providers

```hcl
terraform {
  required_version = ">= 1.9"
  required_providers {
    aws = { source = "hashicorp/aws", version = "~> 6.0" }   # pin to the major you have tested
  }
  backend "s3" {
    bucket       = "shopnow-tfstate-prod"
    key          = "network/terraform.tfstate"
    region       = "ap-south-1"
    encrypt      = true
    use_lockfile = true          # S3-native state locking in recent versions (older setups used DynamoDB)
  }
}

provider "aws" {
  region = "ap-south-1"
  default_tags {
    tags = { project = "shopnow", environment = "prod", owner = "platform-team", cost_center = "cc-142", managed_by = "terraform" }
  }
}
```

> Provider major versions and backend options evolve — confirm against the provider/backend docs for the versions you pin.

### 12.3 Module example

```hcl
module "orders_db" {
  source  = "../../modules/postgres"
  name    = "orders"
  engine_version         = "18"
  instance_class         = "db.r7g.large"
  multi_az               = true
  backup_retention_days  = 14
  deletion_protection    = true
  allowed_security_groups = [module.cluster.node_security_group_id]
}
```

### 12.4 CI workflow for IaC

```text
PR:    fmt -check → validate → tflint → checkov/trivy config → plan (per changed stack) → post plan summary to PR
Merge: plan again → manual approval for prod (environment protection) → apply → record outputs
Daily: drift detection (plan -detailed-exitcode) → alert on drift
```

```bash
terraform fmt -check -recursive
terraform init -input=false
terraform validate
terraform plan -input=false -out=tfplan
terraform show -json tfplan > tfplan.json   # feed to policy checks (Conftest/OPA)
terraform apply -input=false tfplan
# OpenTofu: same commands with `tofu`
```

### 12.5 Safety rules

```text
🔴 `prevent_destroy` / deletion protection on stateful resources (databases, buckets with data, KMS keys)
🔴 Review destructive plan actions explicitly (replace/destroy); block auto-apply when destroys are present
🟠 `moved` and `import` blocks for refactors instead of manual state surgery
🟠 Secrets never hard-coded in .tf files; reference secret managers; mark sensitive outputs
```

---

## 13. Policy as Code and Drift

| Layer | Tools | Example policies |
|---|---|---|
| IaC (pre-apply) | **Checkov**, Trivy (config), tfsec rules, **Conftest/OPA**, Sentinel (HCP Terraform), OpenTofu/Terraform test frameworks | No public buckets; encryption required; approved regions only; mandatory tags; no 0.0.0.0/0 SSH |
| Cloud org guardrails | SCPs, Azure Policy, GCP Org Policy | Deny region outside India for regulated data; deny disabling audit logs |
| Kubernetes admission | Kyverno, OPA Gatekeeper, ValidatingAdmissionPolicy | Signed images only; no privileged pods; resource requests required |
| Runtime posture | CSPM/CNAPP, Prowler | Detect drift and misconfigurations |

```rego
# Conftest policy: deny S3 buckets without public access block (illustrative)
package terraform.s3

import rego.v1

deny contains msg if {
  some rc in input.resource_changes
  rc.type == "aws_s3_bucket_public_access_block"
  rc.change.after.block_public_acls == false
  msg := sprintf("%s must block public ACLs", [rc.address])
}
```

### Drift management

```text
- Scheduled `plan` to detect drift; alert owners
- Root-cause manual changes (break-glass? missing IaC?) and codify them
- Prevent drift: least-privilege console access in prod; read-only by default
```

---

## 14. Configuration Management for VMs

| Need | Tools |
|---|---|
| Golden images | **Packer** (bake hardened images: OS patches, agents, CIS hardening) |
| Configuration | **Ansible** (agentless), cloud-init for first boot; Chef/Puppet/Salt in legacy estates |
| Patching | Managed patching services (SSM Patch Manager, Azure Update Manager, GCP VM Manager), unattended-upgrades on Ubuntu |
| Remote access | Session managers (SSM, IAP, Bastion) instead of open SSH where possible |

```yaml
# Ansible playbook excerpt — baseline hardening (illustrative)
- hosts: app_servers
  become: true
  tasks:
    - name: Ensure unattended security upgrades
      ansible.builtin.apt: { name: unattended-upgrades, state: present }
    - name: Disable SSH password authentication
      ansible.builtin.lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?PasswordAuthentication'
        line: 'PasswordAuthentication no'
      notify: restart ssh
  handlers:
    - name: restart ssh
      ansible.builtin.service: { name: ssh, state: restarted }
```

Hands-on single-server hardening (SSH, UFW, Fail2Ban/CrowdSec, kernel tuning): **39-vps-enterprise-setup-guide.md**; shell scripting: **27**, **28**.

---

## 15. Cloud Identity and Access

```text
🔴 Humans: SSO via central IdP (IAM Identity Center / Entra ID / Cloud Identity), MFA (phishing-resistant for admins), no IAM users with long-lived keys
🔴 Workloads: roles via workload identity (IRSA/EKS Pod Identity, GKE Workload Identity Federation, AKS Workload ID), instance profiles, managed identities
🔴 CI/CD: OIDC federation with tightly scoped trust conditions (chapter 16 §14)
🔴 Least privilege: start from minimal, use access analyzers/recommenders to trim unused permissions
🟠 Just-in-time elevated access for production with approval and session recording
🟠 Quarterly access reviews; remove dormant identities
🟠 Permission boundaries / deny policies as guardrails
```

Full security: chapter 09 §18.

---

## 16. High Availability and Multi-Region

| Pattern | RTO / RPO (typical) | Cost | Complexity |
|---|---|---|---|
| Single AZ | Hours / last backup | Lowest | Low — **not for production** |
| **Multi-AZ** (default) | Minutes / ~0 for sync-replicated DBs | + | Low–medium |
| Backup & restore to another region | Hours / backup interval | + | Low |
| Pilot light (core data replicated, minimal infra in DR region) | Tens of minutes–hours / minutes | ++ | Medium |
| Warm standby (scaled-down full stack in DR region) | Minutes / seconds–minutes | +++ | Medium–high |
| Active-active multi-region | Near-zero / near-zero (with conflict handling) | ++++ | High (data consistency, routing) |

```text
- Start with multi-AZ for everything in production
- Choose a DR pattern per service from business RTO/RPO (chapter 20), not uniformly
- Data residency may require both regions inside India (e.g. Mumbai + Hyderabad / Delhi)
- Test failover regularly — untested DR is not DR
- Beware of hidden single points: control planes, DNS, CI/CD, secrets, IdP, third-party APIs
```

---

## 17. Edge, CDN, and DNS

| Component | Practice |
|---|---|
| **CDN** (CloudFront, Cloud CDN/Media CDN, Azure Front Door, Cloudflare, Akamai, Fastly) | Cache static assets and cacheable APIs; TLS at edge; HTTP/3; origin shielding |
| **WAF & DDoS** | Managed rule sets (OWASP-based), rate limiting, bot management; provider DDoS protection |
| **Edge compute** | Redirects, auth checks, A/B routing, header manipulation, personalisation (Workers, CloudFront Functions/Lambda@Edge, Edge Functions) |
| **DNS** | Managed DNS with health-checked failover/latency routing; low TTLs for records you may fail over; DNSSEC where supported |
| **Certificates** | Automated issuance/renewal (ACM, Google-managed, Azure-managed, ACME/cert-manager); monitor expiry |
| **Origin protection** | Only accept traffic from CDN (origin auth headers, IP allow-lists, private origins) |

---

## 18. FinOps — Cost Management

The **FinOps Framework** (FinOps Foundation) organises cloud financial management into iterative phases — **Inform → Optimize → Operate** — with shared accountability between engineering, finance, and business.

### 18.1 Inform (visibility)

```text
🔴 Mandatory tags/labels: project, environment, owner/team, cost_center, service
🔴 Cost allocation per team/service/environment; showback or chargeback
🔴 Budgets and anomaly alerts per account/project
🟠 Unit economics: cost per order / per active user / per 1 000 API requests
🟠 Kubernetes cost allocation (OpenCost, Kubecost, provider tools) by namespace/label
🟠 FOCUS (FinOps Open Cost and Usage Specification) for normalised billing data across providers
```

### 18.2 Optimize

| Lever | Typical action |
|---|---|
| Rightsizing | Match instance/pod sizes to measured usage |
| Autoscaling & scheduling | Scale down nights/weekends for non-prod; scale-to-zero for dev |
| Commitments | Savings Plans / Reserved Instances / Committed Use Discounts for steady baseline |
| Spot / preemptible | Stateless, fault-tolerant workloads, CI runners, batch |
| ARM-based instances | Graviton/Axion/Cobalt-class instances often cheaper per performance |
| Storage tiering | Lifecycle rules; delete orphaned volumes/snapshots |
| Data transfer | Private endpoints, CDN offload, keep chatty services in the same AZ/region, compress |
| Managed service choice | Serverless for spiky; provisioned for steady |
| Logging/observability costs | Sampling, retention tiers, drop noisy logs, limit metric cardinality (chapter 18) |

### 18.3 Operate

```text
- Monthly cost review per team with action items
- Cost checks in PRs for IaC (Infracost) to show cost impact before merging
- Clean-up automation for sandboxes and preview environments
- Track commitment utilisation and coverage
```

---

## 19. Sustainability

```text
- Choose regions with lower carbon intensity where residency and latency allow; use provider carbon footprint tools
- Rightsize and autoscale (idle resources waste energy and money)
- Prefer efficient architectures (ARM instances, serverless for spiky loads, efficient data formats)
- Reduce data movement and storage of unneeded data (retention policies)
- AWS Well-Architected includes a Sustainability pillar; use its guidance in reviews
```

---

## 20. Hybrid, On-Prem, and Sovereign Cloud

| Need | Options |
|---|---|
| Regulatory/sovereignty requirements | In-country regions, sovereign cloud offerings, government community clouds (e.g. MeitY-empanelled services for Indian government workloads) |
| Latency/edge locations | Provider edge/outpost/stack offerings, on-prem Kubernetes |
| Existing data centres | Hybrid connectivity, consistent platforms (Kubernetes everywhere, Anthos/Arc/EKS Anywhere-style management) |
| Cost at very large steady scale | Some organisations repatriate stable workloads — model total cost including staff |

```text
Portability strategy: containers + Kubernetes + OpenTofu/Terraform + open data formats + OTel
reduce switching cost; but don't avoid every managed service — design exit plans for critical dependencies instead.
```

---

## 21. Reference Setups

### 21.1 Stage 1 — Single VPS (MVP)
One VPS (Ubuntu LTS) with Nginx, app processes (PM2/systemd/Docker Compose), PostgreSQL, Redis/Valkey, Prometheus/Grafana/Loki, daily off-site backups, Cloudflare in front. Full guide: **39-vps-enterprise-setup-guide.md**. Plan the move to managed services before the single server becomes a risk.

### 21.2 Stage 2 — Managed PaaS
```text
CDN/WAF → managed containers (Cloud Run / App Runner / Azure Container Apps), min 2 instances
        → managed PostgreSQL (HA, PITR) + managed Redis/Valkey
        → object storage; managed queue; secret manager
IaC: OpenTofu/Terraform; CI/CD: GitHub Actions with OIDC
Observability: OTel → managed backends
```

### 21.3 Stage 3 — Kubernetes platform (multiple teams)
```text
Landing zone (prod/non-prod accounts) → hub network → managed Kubernetes per environment (multi-AZ node pools, Karpenter/autoscaler)
Add-ons: Gateway API implementation, cert-manager, external-dns, External Secrets, Kyverno, OTel Collector, Argo CD, Argo Rollouts, Velero
Data: managed databases outside the cluster; Kafka (managed) for events
Platform: Backstage templates; golden paths; FinOps dashboards per namespace
```

### 21.4 Stage 4 — Multi-region (high availability / residency)
```text
Two regions (e.g. both within India for residency) → global DNS/CDN with health-based routing
Active-passive (warm standby) for stateful core; active-active for stateless read-heavy services
Cross-region DB replication; regular failover drills; regional cells to limit blast radius
```

---

## 22. Checklists

### Foundation
- [ ] Multi-account/subscription/project structure with prod isolation
- [ ] Central immutable audit logs; guardrail policies (regions, encryption, public access)
- [ ] SSO + MFA for humans; workload identity; CI OIDC; no long-lived keys
- [ ] IPAM plan; private subnets; private endpoints; controlled egress
- [ ] Budgets, anomaly alerts, mandatory tags

### IaC
- [ ] All infra in code; remote encrypted state with locking; per-env/per-component states
- [ ] Pinned providers/modules; reusable modules
- [ ] PR plan + policy checks + cost estimate; CI apply with approvals; drift detection
- [ ] Deletion protection on stateful resources

### Kubernetes
- [ ] Supported version; upgrade plan; deprecated API checks
- [ ] Gateway API (or maintained controller) — **no retired Ingress NGINX**
- [ ] Pod Security `restricted`, NetworkPolicies, RBAC, signed images, secrets via external store
- [ ] Requests/limits, HPA/KEDA, PDBs, topology spread, probes, graceful shutdown
- [ ] GitOps, progressive delivery, observability stack, backups (Velero)

### Resilience & cost
- [ ] Multi-AZ for production; DR pattern per service with tested failover
- [ ] CDN/WAF/DDoS protection; automated certificates; DNS failover
- [ ] Rightsizing, commitments, spot where suitable, storage lifecycle
- [ ] Unit cost metrics reviewed monthly

---

## 23. References

### Cloud frameworks and docs
- AWS Well-Architected: https://aws.amazon.com/architecture/well-architected/
- Azure Cloud Adoption Framework & Landing Zones: https://learn.microsoft.com/azure/cloud-adoption-framework/
- Google Cloud Architecture Framework: https://cloud.google.com/architecture/framework
- AWS shared responsibility model: https://aws.amazon.com/compliance/shared-responsibility-model/

### Kubernetes
- Kubernetes releases & version skew: https://kubernetes.io/releases/
- Kubernetes 1.36 release information: https://kubernetes.io/releases/1.36/
- Ingress NGINX retirement announcement: https://kubernetes.io/blog/2025/11/11/ingress-nginx-retirement/
- Gateway API: https://gateway-api.sigs.k8s.io/ · ingress2gateway: https://github.com/kubernetes-sigs/ingress2gateway
- Pod Security Standards: https://kubernetes.io/docs/concepts/security/pod-security-standards/
- Production best practices (Kubernetes docs): https://kubernetes.io/docs/setup/production-environment/
- KEDA: https://keda.sh/ · Karpenter: https://karpenter.sh/ · Velero: https://velero.io/ · cert-manager: https://cert-manager.io/ · External Secrets: https://external-secrets.io/

### IaC and policy
- OpenTofu: https://opentofu.org/docs/ · Terraform: https://developer.hashicorp.com/terraform/docs
- Pulumi: https://www.pulumi.com/docs/ · AWS CDK: https://docs.aws.amazon.com/cdk/ · Bicep: https://learn.microsoft.com/azure/azure-resource-manager/bicep/ · Crossplane: https://docs.crossplane.io/
- Terragrunt: https://terragrunt.gruntwork.io/ · TFLint: https://github.com/terraform-linters/tflint
- Checkov: https://www.checkov.io/ · Conftest: https://www.conftest.dev/ · Trivy: https://trivy.dev/
- Packer: https://developer.hashicorp.com/packer · Ansible: https://docs.ansible.com/
- Infracost: https://www.infracost.io/docs/

### FinOps
- FinOps Framework: https://www.finops.org/framework/
- FOCUS specification: https://focus.finops.org/
- OpenCost: https://www.opencost.io/

---

**Previous:** [16 — DevOps & CI/CD](./16-devops-and-ci-cd.md) · **Next:** [18 — Observability & Monitoring](./18-observability-and-monitoring.md)