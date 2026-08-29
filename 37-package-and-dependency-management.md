# 📦 Package & Dependency Management

> **Production-grade, future-proof guide to language runtimes, SDKs, package managers, build systems, registries, dependency locking, security, and reproducible builds.**

---

# 🎯 Purpose

Every production software project depends on external libraries, frameworks, plugins, SDKs, or internal packages.

A professional dependency-management strategy should answer:

```text
What runtime / SDK do we use?
        ↓
What package manager / build tool do we use?
        ↓
Where are packages published?
        ↓
How are dependencies declared?
        ↓
How are versions locked?
        ↓
How are dependencies updated?
        ↓
How are vulnerabilities detected?
        ↓
How are builds reproduced?
```

---

# 🧩 The Dependency Management Model

Do not treat **runtime**, **package manager**, **build system**, and **registry** as the same thing.

```text
Runtime / SDK
      ↓
Package Manager
      ↓
Build System
      ↓
Package Registry / Repository
      ↓
Dependency Locking
      ↓
Security / License / Supply-Chain Controls
```

Some ecosystems combine several of these responsibilities.

---

# 🌎 Language → Package Manager → Registry Matrix

| Language / Platform | Runtime / SDK | Primary Package / Dependency Tool | Strong Alternative | Primary Registry / Repository |
|---|---|---|---|---|
| **JavaScript / TypeScript** | Node.js | **npm** | pnpm, Yarn | **npm Registry** |
| **Java** | JDK | **Maven** | Gradle | **Maven Central** |
| **Kotlin** | JDK / Kotlin | **Gradle** | Maven | **Maven Central** |
| **Python** | Python | **uv / pip** | Poetry, Hatch | **PyPI** |
| **C# / .NET** | .NET SDK | **dotnet / NuGet** | Paket | **NuGet Gallery** |
| **PHP** | PHP | **Composer** | — | **Packagist** |
| **Dart / Flutter** | Dart SDK | **Dart Pub** | — | **pub.dev** |
| **Swift** | Swift SDK | **Swift Package Manager** | CocoaPods* | **Swift Package Index / Git repositories** |
| **Rust** | Rust toolchain | **Cargo** | — | **crates.io** |
| **Go** | Go SDK | **Go Modules** | — | **Module Proxy / source repositories** |
| **Ruby** | Ruby | **Bundler** | — | **RubyGems.org** |
| **Scala** | JDK / Scala | **sbt** | Mill | **Maven Central / Scala repositories** |
| **C / C++** | Compiler / SDK | **CMake + package manager** | Meson | vcpkg / Conan / system repositories |
| **R** | R | **R packages / R CMD** | renv | CRAN |
| **Elixir** | Erlang/OTP | **Mix** | — | Hex.pm |

> `*` CocoaPods remains relevant to existing Apple projects, but Swift Package Manager should generally be evaluated first for modern Swift projects.

---

# 🟨 JavaScript / TypeScript

## Runtime

```text
Node.js
```

## Primary Package Manager

```text
npm
```

## Alternatives

```text
pnpm
Yarn
```

## Registry

```text
npm Registry
```

npm uses the public npm registry by default, while npm can also be configured to use compatible private registries. citeturn0search17

## Typical project

```text
project/
├── package.json
├── package-lock.json
├── src/
└── node_modules/
```

### Production workflow

```text
Node.js
   ↓
npm / pnpm
   ↓
package.json
   ↓
Lockfile
   ↓
Install
   ↓
Build
   ↓
Test
   ↓
CI/CD
```

### Important files

| File | Purpose |
|---|---|
| `package.json` | Project metadata and dependency declarations |
| `package-lock.json` | npm dependency resolution lock |
| `pnpm-lock.yaml` | pnpm dependency lock |
| `yarn.lock` | Yarn dependency lock |
| `.npmrc` | npm configuration / registry settings |

### Engineering rule

Use **one primary package manager per repository**.

Do not mix:

```text
npm
+
pnpm
+
Yarn
```

without a documented reason.

---

# ☕ Java

## Runtime

```text
JDK
```

## Primary Build / Dependency Tool

```text
Maven
```

## Alternative

```text
Gradle
```

## Primary Registry

```text
Maven Central
```

Maven Central is the central repository used for Maven artifacts and is also consumed by projects using other build tools. citeturn0search10

### Maven

```text
pom.xml
    ↓
Maven
    ↓
Maven Central
    ↓
Dependencies
    ↓
Compile
    ↓
Test
    ↓
Package
```

### Gradle

```text
build.gradle
build.gradle.kts
        ↓
Gradle
        ↓
Maven Central / Google / private repositories
        ↓
Dependencies
```

Gradle provides dependency declaration, resolution, conflict management, dependency locking, and version catalogs. citeturn0search0turn0search1

### Gradle version catalog

```text
gradle/
└── libs.versions.toml
```

This provides centralized dependency and plugin version management. citeturn0search1

### Enterprise rule

For a new Java project:

```text
Maven OR Gradle
       ↓
Choose one
       ↓
Standardize
```

Do not use both as competing build systems unless the architecture genuinely requires it.

---

# 🟣 Kotlin

## Runtime / SDK

```text
JDK
+
Kotlin
```

## Primary Build Tool

```text
Gradle
```

## Registry

```text
Maven Central
```

Typical Android/Kotlin project:

```text
Kotlin
   ↓
Gradle
   ↓
Maven Central / Google
   ↓
Dependencies
   ↓
Build
```

For Android:

```text
Kotlin
 ↓
Gradle
 ↓
Android Gradle Plugin
 ↓
Google Maven Repository
 ↓
Maven Central
```

---

# 🐍 Python

## Runtime

```text
Python
```

## Primary Modern Dependency Tool

```text
uv
```

## Standard Installer

```text
pip
```

## Alternatives

```text
Poetry
Hatch
Conda
```

## Registry

```text
PyPI
```

Recommended modern project:

```text
pyproject.toml
       ↓
uv
       ↓
Lockfile
       ↓
Virtual Environment
       ↓
Tests
       ↓
Build
```

### Engineering rule

Prefer a reproducible project configuration using:

```text
pyproject.toml
+
lock file
+
isolated environment
```

Avoid relying on undocumented globally installed packages.

---

# 🔵 C# / .NET

## Runtime / SDK

```text
.NET SDK
```

## Primary Package System

```text
dotnet CLI
+
NuGet
```

## Registry

```text
NuGet Gallery
```

NuGet is the package manager for .NET and NuGet.org is its central public package repository. citeturn0search13

Typical project:

```text
project/
├── project.csproj
├── Program.cs
└── ...
```

Dependency:

```text
dotnet add package PackageName
```

Production flow:

```text
.NET SDK
   ↓
dotnet
   ↓
NuGet
   ↓
Restore
   ↓
Build
   ↓
Test
   ↓
Publish
```

---

# 🐘 PHP

## Runtime

```text
PHP
```

## Package Manager

```text
Composer
```

## Primary Registry

```text
Packagist
```

Typical project:

```text
composer.json
composer.lock
```

Workflow:

```text
PHP
 ↓
Composer
 ↓
Packagist
 ↓
Dependencies
 ↓
Autoloading
 ↓
Test
 ↓
Build / Deploy
```

Use `composer.lock` for reproducible application deployments.

---

# 🦋 Dart / Flutter

## SDK

```text
Dart SDK
Flutter SDK
```

## Package Manager

```text
Dart Pub
```

## Registry

```text
pub.dev
```

Typical project:

```text
pubspec.yaml
pubspec.lock
```

Workflow:

```text
Flutter
 ↓
Dart Pub
 ↓
pubspec.yaml
 ↓
pubspec.lock
 ↓
Build
```

For applications, commit the appropriate lock file so dependency resolution is reproducible.

---

# 🍎 Swift

## SDK

```text
Swift SDK
Xcode
```

## Primary Package Manager

```text
Swift Package Manager
```

## Package Discovery

```text
Swift Package Index
+
Git repositories
```

Typical project:

```text
Package.swift
```

Workflow:

```text
Swift
 ↓
Swift Package Manager
 ↓
Package.swift
 ↓
Dependency Resolution
 ↓
Build
 ↓
Test
```

For modern Swift projects, evaluate Swift Package Manager before adding another dependency-management system.

---

# 🦀 Rust

## Toolchain

```text
Rust
rustup
```

## Package Manager / Build Tool

```text
Cargo
```

## Registry

```text
crates.io
```

Typical project:

```text
Cargo.toml
Cargo.lock
src/
```

Workflow:

```text
Rust
 ↓
Cargo
 ↓
crates.io
 ↓
Cargo.lock
 ↓
Build
 ↓
Test
```

Cargo handles dependency management and builds for Rust projects.

---

# 🐹 Go

## SDK

```text
Go
```

## Dependency System

```text
Go Modules
```

## Tools

```text
go
go get
go mod
```

Go officially recommends modules for dependency management; module dependencies are recorded in `go.mod`. citeturn0search2turn0search3

Typical project:

```text
go.mod
go.sum
```

Workflow:

```text
Go
 ↓
Go Modules
 ↓
go.mod
 ↓
go.sum
 ↓
go mod tidy
 ↓
Build
 ↓
Test
```

Go's ecosystem is decentralized: modules can be retrieved from repositories or module proxies, while `pkg.go.dev` provides package discovery and documentation. citeturn0search8turn0search11

---

# 💎 Ruby

## Runtime

```text
Ruby
```

## Dependency Manager

```text
Bundler
```

## Registry

```text
RubyGems.org
```

Bundler manages application dependencies across machines in a repeatable way. citeturn0search7

Typical project:

```text
Gemfile
Gemfile.lock
```

Workflow:

```text
Ruby
 ↓
Bundler
 ↓
RubyGems
 ↓
Gemfile
 ↓
Gemfile.lock
 ↓
Test
 ↓
Deploy
```

---

# 🟢 Scala

## Runtime

```text
JDK
Scala
```

## Primary Build Tool

```text
sbt
```

## Alternative

```text
Mill
```

Typical ecosystem:

```text
Scala
 ↓
sbt
 ↓
Maven Central / Scala repositories
 ↓
Dependencies
 ↓
Compile
 ↓
Test
```

---

# ⚙️ C / C++

C/C++ is different from managed ecosystems.

There is no single universally dominant package manager.

Typical stack:

```text
Compiler
 ↓
CMake
 ↓
Package Manager
 ↓
Dependency
```

Common choices:

```text
vcpkg
Conan
system package manager
```

Build systems:

```text
CMake
Meson
Make
Ninja
```

Example:

```text
C++
 ↓
CMake
 ↓
vcpkg / Conan
 ↓
Libraries
 ↓
Compiler
 ↓
Binary
```

### Enterprise rule

Standardize the combination:

```text
Compiler
+
Build System
+
Package Manager
```

rather than standardizing only one component.

---

# 📊 R

## Runtime

```text
R
```

## Package System

```text
R Packages
```

## Repository

```text
CRAN
```

For reproducible projects, `renv` can be used to isolate and lock package environments.

---

# 🟣 Elixir

## Runtime

```text
Erlang/OTP
Elixir
```

## Build / Dependency Tool

```text
Mix
```

## Registry

```text
Hex.pm
```

Typical project:

```text
mix.exs
mix.lock
```

Workflow:

```text
Elixir
 ↓
Mix
 ↓
Hex
 ↓
Dependencies
 ↓
Compile
 ↓
Test
```

---

# 🔐 Dependency Security

Dependency management is also **software supply-chain security**.

Every dependency introduces risk:

```text
Application
    ↓
Direct Dependencies
    ↓
Transitive Dependencies
    ↓
Build Tools
    ↓
Registries
```

Security controls should include:

```text
Dependency Audit
       ↓
Vulnerability Scanning
       ↓
License Scanning
       ↓
Lock Files
       ↓
Trusted Registries
       ↓
Private Registry Controls
       ↓
CI Security Gates
```

---

# 🔒 Lock Files

Lock files capture the resolved dependency graph.

Common examples:

| Ecosystem | Lock File |
|---|---|
| Node / npm | `package-lock.json` |
| pnpm | `pnpm-lock.yaml` |
| Yarn | `yarn.lock` |
| Python / uv | `uv.lock` |
| Java / Gradle | Dependency locking / version catalogs |
| .NET | `packages.lock.json` where enabled |
| PHP | `composer.lock` |
| Dart | `pubspec.lock` |
| Rust | `Cargo.lock` |
| Go | `go.sum` |
| Ruby | `Gemfile.lock` |
| Swift | Package resolution state |

### Production principle

> **Application builds should be reproducible.**

A developer machine should not silently resolve a different dependency graph than CI or production.

---

# 🏢 Private Package Registries

Enterprise organizations frequently need private package repositories.

Architecture:

```text
Developer
    ↓
Private Registry
    ↓
Approved Dependencies
    ↓
Public Registry
```

Examples of enterprise repository platforms:

```text
JFrog Artifactory
Sonatype Nexus
GitHub Packages
GitLab Package Registry
AWS CodeArtifact
Azure Artifacts
Google Artifact Registry
```

Use private registries for:

- Internal libraries
- Approved third-party packages
- Security controls
- Dependency proxying
- Artifact retention
- Access control
- Compliance

---

# 🔄 Dependency Update Strategy

Do not update all dependencies blindly.

Recommended workflow:

```text
Dependency Update
       ↓
Review Release Notes
       ↓
Security / Breaking Changes
       ↓
Update Branch
       ↓
Build
       ↓
Unit Tests
       ↓
Integration Tests
       ↓
Security Scan
       ↓
Code Review
       ↓
Merge
```

---

# 🧮 Versioning Strategy

Prefer explicit version policies.

Examples:

```text
Exact Version
Semantic Version Range
Version Catalog
Lock File
Dependency Constraints
```

For enterprise projects:

```text
Declared Version
      +
Resolved Version
      +
Lock / Constraint
```

should be intentional.

---

# 🧱 Reproducible Builds

A production-grade dependency system should allow:

```text
Developer Machine
       ↓
Same Source
       ↓
Same Dependency Resolution
       ↓
Same Build Inputs
       ↓
Same Artifact
```

This requires control over:

```text
Runtime Version
Package Manager Version
Dependency Versions
Lock Files
Build Tool Version
OS / Container
Environment
```

---

# 🚫 Dependency Anti-Patterns

Avoid:

```text
❌ Installing packages globally and relying on them
❌ Mixing package managers in one repository
❌ Ignoring lock files
❌ Using abandoned libraries without justification
❌ Blind dependency upgrades
❌ Committing secrets in package configuration
❌ Depending on untrusted registries
❌ Adding libraries for trivial functionality
❌ Ignoring transitive dependencies
❌ Ignoring licenses
❌ Ignoring security advisories
```

---

# 🏆 Enterprise Dependency Governance

A mature engineering organization should define:

```text
Approved Languages
        ↓
Approved Runtime Versions
        ↓
Approved Package Managers
        ↓
Approved Registries
        ↓
Dependency Policy
        ↓
Security Policy
        ↓
License Policy
        ↓
Update Policy
        ↓
CI/CD Enforcement
```

Example:

```text
Java
 ↓
JDK 21 / approved LTS
 ↓
Maven
 ↓
Maven Central + Private Nexus
 ↓
Dependency Scan
 ↓
License Scan
 ↓
CI Gate
```

---

# 🔄 Universal Dependency Lifecycle

```text
Discover
   ↓
Evaluate
   ↓
Approve
   ↓
Add
   ↓
Lock
   ↓
Build
   ↓
Test
   ↓
Scan
   ↓
Release
   ↓
Monitor
   ↓
Update
   ↓
Remove
```

---

# 🧠 Final Engineering Principles

### 1. One primary package manager per project

```text
Repository
    ↓
One standard
    ↓
Reproducible workflow
```

### 2. Prefer official registries

Use the ecosystem's primary trusted registry whenever appropriate.

### 3. Lock application dependencies

Reproducibility is more important than convenience.

### 4. Minimize dependency count

Every dependency creates:

```text
Maintenance
Security
License
Upgrade
Build
Supply-chain
```

responsibility.

### 5. Treat dependencies as production code

A dependency should be evaluated for:

```text
Security
Quality
Maintenance
License
Performance
Compatibility
Community
```

### 6. Make dependency management part of CI/CD

```text
Commit
 ↓
Dependency Resolution
 ↓
Build
 ↓
Test
 ↓
Security Scan
 ↓
License Scan
 ↓
Artifact
```

---

# 📚 Official Ecosystem Sources

Use the official ecosystem documentation and registries as the primary source of truth:

- npm Registry / npm documentation
- Maven Central / Maven documentation
- Gradle documentation
- PyPI / Python Packaging documentation
- NuGet
- Composer / Packagist
- Swift Package Manager / Swift Package Index
- crates.io / Cargo
- Go Modules / pkg.go.dev
- RubyGems / Bundler
- pub.dev / Dart documentation

> **Version note:** Package managers, registries, build tools, and language runtimes evolve independently. Before standardizing an enterprise toolchain, verify compatibility between the runtime, package manager, build system, plugins, CI environment, and dependency registry.
