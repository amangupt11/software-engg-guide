# 🐘 PhpStorm — Enterprise PHP Development Environment

> **Production-grade, future-proof PhpStorm baseline for PHP, Laravel, Symfony, WordPress, APIs, databases, frontend tooling, testing, containers, and cloud-native development.**

## 🎯 Engineering Approach

The goal is **not to install as many plugins as possible**.

Evaluate PhpStorm capabilities in this order:

```text
Built-in / Bundled Capability
        ↓
Official JetBrains Plugin
        ↓
Trusted Community Plugin
        ↓
Project-specific Plugin
```

Prefer built-in and officially supported functionality whenever it already satisfies the requirement.

PhpStorm 2026.2 provides built-in support across PHP, Laravel, JavaScript/TypeScript, Tailwind CSS, SQL, Git, databases, Docker, testing, debugging, and more. citeturn0search0turn0search8

---

## 🏷️ Status Legend

| Status | Meaning |
|:---:|---|
| 🔴 **REQUIRED** | Recommended enterprise baseline |
| 🟠 **RECOMMENDED** | Strong addition for most projects |
| 🟡 **OPTIONAL** | Install when the project requires it |
| 🟢 **PERSONAL** | Developer preference / productivity |
| ⚪ **BUILT-IN** | Already provided by PhpStorm |

---

# 🧩 Core PhpStorm Matrix

| # | Capability / Plugin | Category | Status | Purpose |
|---:|---|---|:---:|---|
| 1 | **PHP** | Language | ⚪ | PHP language support |
| 2 | **Git** | Version Control | ⚪ | Git integration |
| 3 | **GitHub** | Collaboration | 🔴 | GitHub repositories and pull requests |
| 4 | **GitToolBox** | Git | 🟠 | Enhanced Git information and workflow |
| 5 | **GitHub Copilot** | AI | 🟠 | AI-assisted development |
| 6 | **JetBrains AI Assistant** | AI | 🟠 | JetBrains-native AI assistance |
| 7 | **SonarQube for IDE** | Security / Quality | 🔴 | Bugs, vulnerabilities and code smells |
| 8 | **Laravel** | Framework | ⚪ | Laravel development |
| 9 | **Symfony Support** | Framework | 🟠* | Symfony-specific development |
| 10 | **WordPress** | Framework / CMS | ⚪ | WordPress development |
| 11 | **PHPUnit** | Testing | ⚪ | Unit and integration testing |
| 12 | **Pest** | Testing | ⚪ | Modern PHP testing |
| 13 | **Behat** | BDD Testing | ⚪ | Behavior-driven testing |
| 14 | **Codeception** | Testing | ⚪ | Acceptance / functional testing |
| 15 | **Database Tools** | Database | ⚪ | SQL and database development |
| 16 | **Docker** | Containers | ⚪ | Docker and Compose |
| 17 | **Kubernetes** | Cloud Native | ⚪* | Kubernetes workflows |
| 18 | **HTTP Client** | API | ⚪ | REST / HTTP API testing |
| 19 | **JavaScript / TypeScript** | Frontend | ⚪ | Frontend development |
| 20 | **Tailwind CSS** | Frontend | ⚪ | Tailwind development |
| 21 | **Node.js / npm** | Frontend Tooling | ⚪ | JS build tooling |
| 22 | **Terraform / HCL** | Infrastructure | 🟠* | Infrastructure as Code |
| 23 | **PlantUML Integration** | Architecture | 🟡 | Architecture diagrams |
| 24 | **Ideolog** | Logs | 🟡 | Log analysis |
| 25 | **Key Promoter X** | Productivity | 🟢 | Learn IDE shortcuts |
| 26 | **Rainbow Brackets** | Productivity | 🟢 | Visual bracket matching |
| 27 | **String Manipulation** | Productivity | 🟢 | String transformation |
| 28 | **Translation** | Productivity | 🟢 | Translation inside IDE |

> `*` Project-dependent. Enable or install only when the technology is actually used.

---

# 🐘 PHP Core

PhpStorm is designed as a complete PHP development environment.

```text
PHP
 ↓
PhpStorm
 ↓
Composer
 ↓
Framework / Application
 ↓
PHPUnit / Pest
 ↓
PHPStan / Psalm
 ↓
Docker
 ↓
CI/CD
 ↓
Production
```

PhpStorm provides intelligent code completion, refactoring, navigation, debugging, testing and code-quality integration. citeturn0search8turn0search11

---

# 📦 Composer

Composer should be treated as a core dependency-management tool.

```text
composer.json
      ↓
Composer
      ↓
composer.lock
      ↓
Reproducible Dependencies
```

Recommended principles:

```text
composer.json
composer.lock
Semantic Versioning
Dependency Audit
Automated Updates
```

Do not install a plugin merely to replace Composer's core functionality.

---

# 🚀 Laravel

Laravel is now built directly into PhpStorm.

Starting with PhpStorm 2025.3, Laravel support became built in, including Laravel-specific development assistance. citeturn0search12turn0search3

```text
PHP
 ↓
PhpStorm
 ↓
Laravel
 ├── Blade
 ├── Eloquent
 ├── Routing
 ├── Artisan
 ├── Queues
 ├── Events
 ├── Jobs
 └── API
```

PhpStorm 2026.2 adds a dedicated Laravel tool window for project information, Artisan commands, logs, errors, and Laravel Cloud workflows. citeturn0search0turn0search5

**Status:** ⚪ Built-in.

> Do not install Laravel Idea separately in current PhpStorm unless your environment specifically requires a separate plugin workflow. The Laravel support is now integrated and the Laravel Idea component ships with PhpStorm. citeturn0search12

---

# Symfony

Symfony support is available through the **Symfony Support** plugin.

PhpStorm's current framework documentation lists Symfony Support as a community-provided plugin, unlike Laravel support which is now built in. citeturn0search4turn0search10

```text
PHP
 ↓
PhpStorm
 ↓
Symfony Support
 ↓
Symfony
 ↓
Doctrine
 ↓
PHPUnit
```

**Status:** 🟠 Recommended when Symfony is used.

---

# WordPress

WordPress support is bundled with PhpStorm.

```text
PHP
 ↓
PhpStorm
 ↓
WordPress
 ├── Themes
 ├── Plugins
 ├── Hooks
 └── REST API
```

**Status:** ⚪ Built-in / bundled.

---

# 🧪 Testing

PhpStorm supports major PHP testing frameworks, including:

```text
PHPUnit
Pest
Behat
Codeception
phpspec
```

JetBrains documents built-in support for these testing tools. citeturn0search11

Recommended enterprise model:

```text
Unit Tests
    ↓
Integration Tests
    ↓
API Tests
    ↓
Acceptance / E2E Tests
    ↓
CI/CD
```

For Laravel:

```text
Pest / PHPUnit
     ↓
Feature Tests
     ↓
Unit Tests
```

---

# 🔍 Static Analysis & Code Quality

PhpStorm integrates with:

```text
PHPStan
Larastan
Psalm
PHP_CodeSniffer
PHP CS Fixer
Laravel Pint
Mess Detector
Qodana
```

JetBrains documents these as supported PHP code-quality tools. citeturn0search1turn0search7

Recommended architecture:

```text
PhpStorm
   ↓
Local Static Analysis
   ↓
CI Static Analysis
   ↓
Pull Request
   ↓
Merge
```

**Status:** 🔴 Required as an engineering practice.

---

# 🔐 SonarQube for IDE

Use SonarQube for IDE where it fits your organization's quality and security workflow.

Detect:

- Bugs
- Vulnerabilities
- Security hotspots
- Code smells
- Maintainability problems

**Status:** 🔴 Recommended enterprise baseline.

> IDE analysis complements—not replaces—CI security scanning, dependency auditing, secret scanning, SAST, code review, and security testing.

---

# 🗄️ Database Development

PhpStorm supports database development out of the box for many database systems.

Common environments include:

```text
PostgreSQL
MySQL
MariaDB
SQLite
MongoDB
Redis
SQL Server
Oracle
```

JetBrains states that PhpStorm supports MySQL, PostgreSQL, MongoDB, Redis, SQLite and many other DBMS dialects out of the box. citeturn0search6

**Status:** ⚪ Built-in / bundled.

---

# 🔌 API Development

PhpStorm includes an HTTP Client.

```text
HTTP Client
 ↓
REST
 ↓
JSON
 ↓
GraphQL
 ↓
WebSocket
```

Recommended repository structure:

```text
http/
├── auth.http
├── users.http
├── orders.http
└── health.http
```

Keep repeatable API requests version-controlled.

---

# 🎨 Frontend Development

PhpStorm also provides support for:

```text
JavaScript
TypeScript
React
Vue
Blade
Tailwind CSS
Vite
Node.js
npm
```

JetBrains specifically positions PhpStorm as an integrated environment for PHP plus Blade, JavaScript/TypeScript, Tailwind CSS and databases. citeturn0search3turn0search8

This means a Laravel full-stack developer generally does **not** need a separate frontend IDE.

---

# 🐳 Docker

Use the built-in Docker integration.

```text
PHP Application
      ↓
Dockerfile
      ↓
Docker Image
      ↓
Container
      ↓
Docker Compose
      ↓
Application Stack
```

PhpStorm 2026.2 adds improved Docker Compose editing, including service status and quick access to logs, database connections and browser endpoints. citeturn0search2

**Status:** ⚪ Built-in / bundled.

---

# ☸️ Kubernetes

For cloud-native PHP applications:

```text
PHP
 ↓
Docker
 ↓
Container Registry
 ↓
Kubernetes
 ├── Deployment
 ├── Service
 ├── ConfigMap
 ├── Secret
 └── Ingress
```

Use supported native tooling where available and keep Kubernetes-specific plugins project-dependent.

**Status:** 🟡 Project-dependent.

---

# 🐞 Debugging

For PHP debugging:

```text
PhpStorm
    ↓
Xdebug
    ↓
Breakpoints
    ↓
Step Over / Step Into
    ↓
Variables
    ↓
Call Stack
```

PhpStorm provides an integrated debugging interface for Xdebug. citeturn0search3

Recommended production workflow:

```text
Local
 ↓
Debug
 ↓
Test
 ↓
CI
 ↓
Staging
 ↓
Production
```

Do not enable verbose debugging behavior in production.

---

# 🌐 PHP Development Environment

A PHP application normally consists of:

```text
PHP Runtime
+
Web Server
+
Database
+
Dependencies
+
Debugger
```

Typical environments:

```text
Local PHP
Docker
Remote PHP
WSL
Linux Server
```

PhpStorm can integrate with local and remote PHP interpreters, web servers, databases, debugging engines, and command-line tools. citeturn0search17

---

# 🤖 AI-Assisted Development

Recommended options:

```text
GitHub Copilot
        +
JetBrains AI Assistant
        +
Coding Agents
```

PhpStorm 2026.2 expands native support for third-party AI providers and coding-agent workflows. citeturn0search0turn0search2

AI should assist with:

- Code generation
- Refactoring
- Tests
- Documentation
- Debugging
- Code explanation

AI should not replace:

- Architecture decisions
- Security review
- Code review
- Testing
- Performance validation
- Production verification

---

# 🌿 Git & GitHub

Core Git support is already provided by PhpStorm.

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
Security Checks
 ↓
Merge
```

GitHub integration provides repository and collaboration workflows.

---

# 🏗️ Infrastructure as Code

For Terraform projects:

```text
Terraform
 ↓
HCL
 ↓
AWS / Azure / GCP
 ↓
Infrastructure
```

**Status:** 🟠 Recommended when Terraform is part of the project.

---

# 📐 Architecture & Documentation

For version-controlled architecture diagrams:

```text
Architecture
 ↓
PlantUML
 ↓
Markdown / Source Control
```

**Status:** 🟡 Optional.

---

# 📝 Environment Configuration

PhpStorm provides `.env` support for navigation, validation and completion in environments such as Laravel projects. citeturn0search3

Recommended:

```text
.env
.env.example
.env.testing
.env.production
```

Never commit real secrets.

---

# 📋 Production Logging & Observability

IDE log viewers are useful during development, but production observability should be centralized.

Recommended architecture:

```text
PHP Application
      ↓
Structured Logs
      ↓
OpenTelemetry / Log Pipeline
      ↓
Centralized Observability
      ↓
Metrics + Logs + Traces
```

Possible platforms:

```text
Grafana
Loki
ELK / Elastic
Cloud-native observability
Sentry
```

---

# ⚠️ Plugin Maintenance Policy

Before adding any third-party PhpStorm plugin, evaluate:

- Publisher reputation
- Marketplace status
- Last release
- IDE compatibility
- GitHub/project activity
- Issue activity
- Security implications
- License
- Community adoption
- Long-term maintenance
- Whether PhpStorm now provides the feature natively

This is especially important because PhpStorm's built-in capabilities continue to expand. Laravel support, for example, moved into PhpStorm itself starting in 2025.3. citeturn0search12

---

# 🧹 Plugin Selection Rules

### Rule 1 — Prefer built-in functionality

```text
PhpStorm native capability
        ↓
Use it
```

### Rule 2 — Framework-specific plugin only when necessary

```text
Laravel
   ↓
Built-in
```

```text
Symfony
   ↓
Symfony Support plugin
```

### Rule 3 — Avoid duplicate functionality

```text
Multiple formatters
        ↓
Conflict risk
```

Use one clearly defined formatting and quality workflow.

### Rule 4 — Keep tooling reproducible

Every developer should be able to reproduce the environment using:

```text
PHP
Composer
Node.js
npm / pnpm
Docker
Git
CLI tools
```

The IDE should improve productivity, not become a hidden dependency.

---

# 🏆 Enterprise PHP Baseline

For a professional PHP/Laravel developer:

```text
PhpStorm
│
├── PHP                         [Built-in]
├── Git                         [Built-in]
├── GitHub                      [Required]
├── Laravel                     [Built-in]
├── Symfony Support             [Project-dependent]
├── WordPress                   [Built-in]
├── PHPUnit / Pest              [Built-in]
├── PHPStan / Larastan          [Recommended]
├── Psalm                       [Project-dependent]
├── PHP CS Fixer / Pint         [Recommended]
├── SonarQube for IDE           [Recommended]
├── Database Tools              [Built-in]
├── HTTP Client                 [Built-in]
├── JavaScript / TypeScript     [Built-in]
├── Tailwind CSS                [Built-in]
├── Docker                      [Built-in]
├── Xdebug                      [Required for debugging]
├── Kubernetes                  [Project-dependent]
├── GitHub Copilot              [Recommended]
├── JetBrains AI Assistant      [Recommended]
└── Terraform / HCL             [Project-dependent]
```

---

# 🔄 Production PHP Workflow

```text
PhpStorm
    ↓
PHP
    ↓
Composer
    ↓
Code Style
    ↓
PHPStan / Psalm
    ↓
PHPUnit / Pest
    ↓
Security Scan
    ↓
Build
    ↓
Docker
    ↓
CI/CD
    ↓
Container Registry
    ↓
Kubernetes / Cloud
    ↓
Production
    ↓
Logs + Metrics + Traces
```

---

# ✅ Final Engineering Principle

> **Use PhpStorm as the development environment, not as a dependency of the application.**

A senior PHP engineer should understand the underlying toolchain:

```text
PHP
Composer
Laravel / Symfony
PHPUnit / Pest
PHPStan / Psalm
PHP CS Fixer / Pint
Git
Linux
Docker
SQL
HTTP
CI/CD
Kubernetes
Cloud
Observability
```

The IDE should provide productivity, navigation, refactoring, debugging, testing and framework assistance while the application remains reproducible from the command line and CI/CD environment.
