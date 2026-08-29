# 🚀 IntelliJ IDEA — Enterprise Java/Kotlin Development Environment

> **Production-grade, future-proof IntelliJ IDEA plugin and tooling baseline for enterprise Java/Kotlin development.**

---

## 🎯 Engineering Approach

The goal is **not to install as many plugins as possible**.

For IntelliJ IDEA, evaluate capabilities in this order:

```text
Built-in IDE Capability
        ↓
Bundled / Official JetBrains Plugin
        ↓
Trusted Marketplace Plugin
        ↓
Project-specific Plugin
```

Prefer built-in and official JetBrains functionality whenever it already satisfies the requirement.

---

## 🏷️ Status Legend

| Status | Meaning |
|:---:|---|
| 🔴 **REQUIRED** | Recommended enterprise baseline |
| 🟠 **RECOMMENDED** | Strong addition for most projects |
| 🟡 **OPTIONAL** | Install when the project requires it |
| 🟢 **PERSONAL** | Developer preference / productivity |
| ⚪ **BUILT-IN** | Already provided by IntelliJ IDEA |

---

# 🧩 Core IntelliJ IDEA Matrix

| # | Plugin / Capability | Category | Status | Purpose |
|---:|---|---|:---:|---|
| 1 | **Git** | Version Control | ⚪ | Git integration |
| 2 | **GitHub** | Git / Collaboration | 🔴 | GitHub authentication, repositories and pull requests |
| 3 | **GitToolBox** | Git | 🟠 | Enhanced Git information and workflow |
| 4 | **GitHub Copilot** | AI | 🟠 | AI-assisted development |
| 5 | **JetBrains AI Assistant** | AI | 🟠 | JetBrains-native AI assistance |
| 6 | **SonarQube for IDE** | Security / Quality | 🔴 | Bugs, vulnerabilities and code smells |
| 7 | **Spring** | Spring / Backend | 🔴* | Spring and Spring Boot development |
| 8 | **Lombok** | Java | 🔴* | Lombok annotations and IDE support |
| 9 | **MapStruct Support** | Java | 🟠 | MapStruct mapper development |
| 10 | **JPA Buddy** | Persistence | 🟠 | JPA / Hibernate development |
| 11 | **Docker** | Containers | ⚪ | Docker and Compose workflows |
| 12 | **Kubernetes** | Cloud Native | ⚪ | Kubernetes resources and clusters |
| 13 | **Database Tools** | Database | ⚪ | SQL and database development |
| 14 | **HTTP Client** | API | ⚪ | REST, GraphQL, WebSocket and HTTP testing |
| 15 | **.ignore** | Git | 🟡 | `.gitignore` and related ignore files |
| 16 | **PlantUML Integration** | Architecture | 🟡 | UML and architecture diagrams |
| 17 | **.env support** | Configuration | 🟡 | Environment configuration |
| 18 | **Terraform / HCL** | Infrastructure | 🟠* | Infrastructure as Code |
| 19 | **AWS Toolkit** | Cloud | 🟡* | AWS development |
| 20 | **Azure Toolkit** | Cloud | 🟡* | Azure development |
| 21 | **Google Cloud Code** | Cloud | 🟡* | Google Cloud development |
| 22 | **Ideolog** | Logs | 🟡 | Log analysis |
| 23 | **Rainbow Brackets** | Productivity | 🟢 | Visual bracket matching |
| 24 | **Key Promoter X** | Productivity | 🟢 | Learn IDE keyboard shortcuts |
| 25 | **String Manipulation** | Productivity | 🟢 | String transformation utilities |
| 26 | **Translation** | Productivity | 🟢 | Translation inside the IDE |

> `*` Project-dependent. Enable these only when the technology is actually used.

---

# ☕ Java Enterprise

IntelliJ IDEA provides extensive native Java development support.

```text
Java
 ↓
IntelliJ IDEA
 ↓
Maven / Gradle
 ↓
Spring Boot
 ↓
JPA / Hibernate
 ↓
PostgreSQL / MySQL
 ↓
JUnit / Mockito
 ↓
Docker
 ↓
Kubernetes
```

Core capabilities should preferably remain inside IntelliJ rather than being duplicated through unnecessary plugins.

---

# 🌱 Spring / Spring Boot

If the project uses Spring Boot:

```text
Spring Boot
 ↓
IntelliJ IDEA
 ↓
Spring
Spring MVC
Spring Security
Spring Data
Spring Cloud
JPA / Hibernate
```

**Status:** 🔴 Required for Spring-based projects.

---

# 🧩 Lombok

If the project uses Lombok:

```text
Lombok
 ↓
@Getter
@Setter
@Builder
@Data
@RequiredArgsConstructor
```

Use the Lombok plugin only when the codebase actually depends on Lombok.

**Status:** 🔴 Required for Lombok projects.

---

# 🔄 MapStruct

For projects using compile-time object mapping:

```text
DTO
 ↓
MapStruct
 ↓
Entity
```

MapStruct support improves navigation and development around mapper interfaces and generated implementations.

**Status:** 🟠 Recommended when MapStruct is used.

---

# 🗃️ JPA / Hibernate

For enterprise persistence:

```text
Java
 ↓
JPA
 ↓
Hibernate
 ↓
PostgreSQL / MySQL / SQL Server
```

JPA Buddy can improve entity, repository, mapping and persistence workflows.

**Status:** 🟠 Recommended for JPA-heavy applications.

---

# 🐳 Docker

IntelliJ IDEA provides integrated Docker support.

```text
Application
    ↓
Dockerfile
    ↓
Docker Image
    ↓
Container
    ↓
Docker Compose
```

**Status:** ⚪ Built-in / bundled capability.

Avoid installing duplicate third-party Docker plugins when the native capability satisfies the project requirements.

---

# ☸️ Kubernetes

Use IntelliJ's Kubernetes support for cloud-native projects.

```text
Application
    ↓
Container
    ↓
Docker Image
    ↓
Kubernetes
    ├── Deployment
    ├── Service
    ├── ConfigMap
    ├── Secret
    └── Ingress
```

**Status:** ⚪ Built-in / bundled capability.

---

# 🗄️ Database Development

IntelliJ provides database tooling for supported database systems.

Typical environments:

```text
PostgreSQL
MySQL
MariaDB
SQL Server
Oracle
SQLite
```

Use the built-in database tools before adding another database plugin.

**Status:** ⚪ Built-in / bundled capability.

---

# 🔌 API Development

IntelliJ's HTTP Client can be used for API development and testing.

```text
HTTP Client
 ↓
REST
 ↓
GraphQL
 ↓
WebSocket
 ↓
API Testing
```

Example project structure:

```text
http/
├── auth.http
├── users.http
├── orders.http
└── health.http
```

**Status:** ⚪ Built-in.

---

# 🔐 Security & Code Quality

## SonarQube for IDE

Use local static analysis to identify:

- Bugs
- Vulnerabilities
- Security hotspots
- Code smells
- Maintainability issues

**Status:** 🔴 Required for an enterprise-quality baseline.

> IDE analysis complements—not replaces—CI/CD security scanning, dependency auditing, SAST, secrets management and peer review.

---

# 🌿 Git & GitHub

Recommended workflow:

```text
Git
 ↓
Feature Branch
 ↓
Commit
 ↓
Pull Request
 ↓
Code Review
 ↓
CI
 ↓
Merge
```

Core Git integration is already provided by IntelliJ IDEA.

GitHub integration should be used for repositories and pull-request workflows.

---

# 🤖 AI-Assisted Development

Recommended options:

```text
GitHub Copilot
        +
JetBrains AI Assistant
```

AI should be treated as an engineering assistant.

It should **not** replace:

- Code review
- Architecture decisions
- Security analysis
- Testing
- Performance analysis
- Production validation

---

# 🏗️ Infrastructure as Code

For Terraform/HCL projects:

```text
Terraform
 ↓
HCL
 ↓
Cloud Provider
 ↓
AWS / Azure / GCP
```

**Status:** 🟠 Recommended when Terraform is part of the engineering environment.

---

# ☁️ Cloud Development

Install cloud-specific tooling only when required.

| Cloud | Tooling |
|---|---|
| AWS | AWS Toolkit |
| Azure | Azure Toolkit |
| Google Cloud | Google Cloud Code |

**Status:** 🟡 Project-dependent.

---

# 📐 Architecture & Documentation

For architecture diagrams:

```text
Architecture
 ↓
UML
 ↓
PlantUML
```

PlantUML integration is useful when diagrams are stored alongside source code.

**Status:** 🟡 Optional.

---

# 🧹 Plugin Selection Rules

### Rule 1 — Prefer built-in functionality

```text
IntelliJ built-in
      ↓
Use it
```

Do not install a Marketplace plugin simply because it provides a feature that IntelliJ already supports.

### Rule 2 — Add plugins for real project requirements

```text
Project Technology
      ↓
Required tooling
      ↓
Install plugin
```

### Rule 3 — Avoid duplicate functionality

```text
Two formatters
      ↓
Conflict risk
```

```text
Two competing Git tools
      ↓
Workflow complexity
```

Use a clear primary tool for each responsibility.

### Rule 4 — Review plugin maintenance

Before adopting a third-party plugin, evaluate:

- Publisher reputation
- Last release
- Compatibility with current IntelliJ versions
- Marketplace adoption
- Issue activity
- Security implications
- License
- Long-term maintenance

### Rule 5 — Keep the baseline small

A production-grade IDE is not defined by the number of plugins installed.

---

# 🏆 Enterprise Baseline

For a professional Java/Spring enterprise developer:

```text
IntelliJ IDEA
│
├── Git                         [Built-in]
├── GitHub                      [Required]
├── Spring                      [Required for Spring]
├── Lombok                      [Project-dependent]
├── SonarQube for IDE           [Required]
├── GitToolBox                  [Recommended]
├── GitHub Copilot              [Recommended]
├── JetBrains AI Assistant      [Recommended]
├── MapStruct Support           [Recommended]
├── JPA Buddy                   [Recommended]
├── Docker                      [Built-in]
├── Kubernetes                  [Built-in]
├── Database Tools              [Built-in]
├── HTTP Client                 [Built-in]
└── Terraform / HCL             [Project-dependent]
```

---

# 🔄 Production Engineering Workflow

```text
IntelliJ IDEA
      ↓
Code
      ↓
Format
      ↓
Static Analysis
      ↓
Unit Tests
      ↓
Integration Tests
      ↓
Build
      ↓
Security Scan
      ↓
Git Commit
      ↓
Pull Request
      ↓
CI/CD
      ↓
Docker
      ↓
Kubernetes
      ↓
Production
```

---

# ✅ Final Engineering Principle

> **Use IntelliJ IDEA as the integrated development environment, but keep the software independent of the IDE.**

A senior engineer should understand and be able to operate the underlying toolchain:

```text
Java
Maven / Gradle
Git
JUnit
Mockito
Spring Boot
Docker
Kubernetes
SQL
HTTP
Linux
CI/CD
Cloud
```

The IDE should improve productivity, navigation, debugging, refactoring and development quality—not become a dependency of the application itself.
