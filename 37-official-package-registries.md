# 📚 Official Package Registries & Dependency Sources

> **Production-grade reference for finding, evaluating, installing, and publishing packages using the official ecosystem registry or repository.**

---

## 🎯 Purpose

For every language ecosystem, distinguish between:

```text
Language
   ↓
Runtime / SDK
   ↓
Package Manager / Build Tool
   ↓
Official Registry / Repository
   ↓
Package Search
   ↓
Dependency Resolution
   ↓
Lock / Reproducible Build
```

### Important

An **official registry/repository** is the source intended by the ecosystem for package distribution.

A third-party website may provide better search, statistics, dependency graphs, or convenience, but it should not automatically be treated as the authoritative package source.

For example:

```text
Java
 ├── Official → Maven Central
 └── Discovery → MVNRepository
```

`mvnrepository.com` is useful for package discovery, but it is **not Maven Central**.

---

# 🌎 Official Registry Matrix

| # | Language / Platform | Package Manager / Build Tool | Official Registry / Repository | Package Search |
|---:|---|---|---|---|
| 1 | **JavaScript / TypeScript** | npm | **npm Registry** | [npm](https://www.npmjs.com/) |
| 2 | **Java** | Maven | **Maven Central** | [Maven Central](https://central.sonatype.com/) |
| 3 | **Java** | Gradle | **Maven Central** | [Maven Central](https://central.sonatype.com/) |
| 4 | **Java / Gradle Plugins** | Gradle | **Gradle Plugin Portal** | [Gradle Plugins](https://plugins.gradle.org/) |
| 5 | **Kotlin** | Gradle / Maven | **Maven Central** | [Maven Central](https://central.sonatype.com/) |
| 6 | **Kotlin / Android** | Gradle | **Google Maven Repository** | [Google Maven](https://maven.google.com/) |
| 7 | **Python** | pip / uv | **PyPI** | [PyPI](https://pypi.org/) |
| 8 | **C# / .NET** | NuGet / dotnet | **NuGet Gallery** | [NuGet](https://www.nuget.org/) |
| 9 | **PHP** | Composer | **Packagist** | [Packagist](https://packagist.org/) |
| 10 | **Dart / Flutter** | Dart Pub | **pub.dev** | [pub.dev](https://pub.dev/) |
| 11 | **Rust** | Cargo | **crates.io** | [crates.io](https://crates.io/) |
| 12 | **Go** | Go Modules | **Go Module Proxy / pkg.go.dev** | [pkg.go.dev](https://pkg.go.dev/) |
| 13 | **Ruby** | Bundler / RubyGems | **RubyGems.org** | [RubyGems](https://rubygems.org/) |
| 14 | **Swift** | Swift Package Manager | **Swift Package Index / Git repositories** | [Swift Package Index](https://swiftpackageindex.com/) |
| 15 | **Scala** | sbt / Gradle | **Maven Central** | [Maven Central](https://central.sonatype.com/) |
| 16 | **C / C++** | vcpkg | **vcpkg Registry** | [vcpkg](https://vcpkg.io/) |
| 17 | **C / C++** | Conan | **ConanCenter** | [ConanCenter](https://conan.io/center) |
| 18 | **R** | R Packages | **CRAN** | [CRAN](https://cran.r-project.org/) |
| 19 | **Elixir** | Mix | **Hex** | [Hex](https://hex.pm/) |

---

# 🟨 JavaScript / TypeScript

### Package Manager

```text
npm
```

### Official Registry

```text
npm Registry
```

The npm CLI uses the public npm registry by default and supports configurable private/compatible registries. citeturn0search0

### Search / Browse

**Official:**

[npmjs.com](https://www.npmjs.com/)

### Package installation

```bash
npm install express
```

### Typical project

```text
package.json
package-lock.json
```

### Alternatives

```text
pnpm
Yarn
```

---

# ☕ Java — Maven

### Package Manager / Build Tool

```text
Maven
```

### Official Repository

```text
Maven Central
```

### Search / Browse

**Official:**

[central.sonatype.com](https://central.sonatype.com/)

### Typical dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### Project file

```text
pom.xml
```

### CLI

```bash
mvn dependency:tree
mvn clean install
```

---

# ☕ Java — Gradle

### Build / Dependency Tool

```text
Gradle
```

### Primary Repository

```text
Maven Central
```

### Plugin Repository

```text
Gradle Plugin Portal
```

### Search

[Maven Central](https://central.sonatype.com/)

[Gradle Plugin Portal](https://plugins.gradle.org/)

### Typical project

```text
build.gradle
build.gradle.kts
settings.gradle
settings.gradle.kts
```

### CLI

```bash
./gradlew dependencies
./gradlew build
```

---

# 🟣 Kotlin

### Build / Dependency Tool

```text
Gradle
```

### Repositories

```text
Maven Central
Google Maven Repository
```

### Search

[Maven Central](https://central.sonatype.com/)

[Google Maven](https://maven.google.com/)

For Android projects, Google's Maven repository is especially important for Android and Google libraries.

---

# 🐍 Python

### Package Managers

```text
uv
pip
```

### Alternatives

```text
Poetry
Hatch
Conda
```

### Official Registry

```text
PyPI
```

### Search

[PyPI](https://pypi.org/)

### CLI

```bash
python -m pip install requests
```

or:

```bash
uv add requests
```

### Typical project

```text
pyproject.toml
uv.lock
```

---

# 🔵 C# / .NET

### Package System

```text
NuGet
```

### CLI

```text
dotnet
```

### Official Registry

```text
NuGet Gallery
```

### Search

[nuget.org](https://www.nuget.org/)

### CLI

```bash
dotnet add package Newtonsoft.Json
dotnet restore
```

### Typical project

```text
Project.csproj
```

---

# 🐘 PHP

### Package Manager

```text
Composer
```

### Official Registry

```text
Packagist
```

### Search

[packagist.org](https://packagist.org/)

### CLI

```bash
composer require laravel/framework
```

### Typical project

```text
composer.json
composer.lock
```

---

# 🦋 Dart / Flutter

### Package Manager

```text
Dart Pub
```

### Official Registry

```text
pub.dev
```

### Search

[pub.dev](https://pub.dev/)

### CLI

```bash
dart pub add http
```

or:

```bash
flutter pub add http
```

### Typical project

```text
pubspec.yaml
pubspec.lock
```

---

# 🦀 Rust

### Package Manager

```text
Cargo
```

### Official Registry

```text
crates.io
```

### Search

[crates.io](https://crates.io/)

### CLI

```bash
cargo add serde
cargo build
cargo update
```

### Typical project

```text
Cargo.toml
Cargo.lock
```

---

# 🐹 Go

### Dependency System

```text
Go Modules
```

### Official Ecosystem Services

```text
Go Module Proxy
pkg.go.dev
```

### Search / Documentation

[pkg.go.dev](https://pkg.go.dev/)

### CLI

```bash
go get github.com/example/package
go mod tidy
```

### Typical project

```text
go.mod
go.sum
```

Go modules can resolve modules through the Go module proxy or other configured module sources.

---

# 💎 Ruby

### Package Manager

```text
Bundler
```

### Registry

```text
RubyGems.org
```

### Search

[rubygems.org](https://rubygems.org/)

### CLI

```bash
bundle add rails
bundle install
```

### Typical project

```text
Gemfile
Gemfile.lock
```

---

# 🍎 Swift

### Package Manager

```text
Swift Package Manager
```

### Package Discovery

```text
Swift Package Index
+
Git repositories
```

### Search

[Swift Package Index](https://swiftpackageindex.com/)

Swift Package Manager also supports package registries implementing the Swift package registry specification. citeturn0search7

### Typical project

```text
Package.swift
```

### CLI

```bash
swift package resolve
swift build
swift test
```

---

# 🟢 Scala

### Build / Dependency Tool

```text
sbt
```

### Repository

```text
Maven Central
```

### Search

[Maven Central](https://central.sonatype.com/)

Scala libraries are commonly published as Maven-compatible artifacts.

---

# ⚙️ C / C++

C/C++ does not have one universal package registry comparable to npm or PyPI.

### Common package managers

```text
vcpkg
Conan
```

### Registries

[vcpkg](https://vcpkg.io/)

[ConanCenter](https://conan.io/center)

Typical architecture:

```text
C / C++
    ↓
CMake
    ↓
vcpkg / Conan
    ↓
Libraries
    ↓
Compiler
```

---

# 📊 R

### Package System

```text
R Packages
```

### Official Registry

```text
CRAN
```

### Search

[CRAN](https://cran.r-project.org/)

### CLI

```r
install.packages("ggplot2")
```

For reproducible environments, `renv` can be used to capture project dependencies.

---

# 🟣 Elixir

### Build / Dependency Tool

```text
Mix
```

### Registry

```text
Hex
```

### Search

[hex.pm](https://hex.pm/)

### CLI

```bash
mix deps.get
mix deps.update --all
```

### Typical project

```text
mix.exs
mix.lock
```

---

# 🔎 Official vs Third-Party Discovery

This distinction is important.

## Official source

```text
npm
Maven Central
PyPI
NuGet
Packagist
pub.dev
crates.io
RubyGems
```

## Third-party discovery tools

Examples:

```text
MVNRepository
Libraries.io
deps.dev
Snyk Advisor
```

Third-party tools can be excellent for:

- Package comparison
- Dependency graphs
- Popularity
- Maintenance information
- Vulnerability information
- Cross-registry discovery

But they should **not automatically replace the official registry as the source of truth**.

---

# ☕ Java Example — Maven Central vs MVNRepository

### Official

```text
Maven
  ↓
Maven Central
  ↓
central.sonatype.com
```

### Third-party discovery

```text
Maven
  ↓
MVNRepository
  ↓
mvnrepository.com
```

Use Maven Central when you need the authoritative artifact repository.

Use MVNRepository when you want convenient dependency discovery, version comparison, or copy/paste dependency information.

---

# 🏢 Enterprise / Private Registries

Production organizations may use private registries in addition to public registries.

Examples:

```text
JFrog Artifactory
Sonatype Nexus
GitHub Packages
GitLab Package Registry
AWS CodeArtifact
Azure Artifacts
Google Artifact Registry
```

Architecture:

```text
Developer
    ↓
Enterprise Registry
    ↓
Approved / Cached Packages
    ↓
Public Registry
```

Use private registries for:

- Internal packages
- Dependency proxying
- Approved third-party packages
- Access control
- Artifact retention
- Compliance
- Supply-chain security

---

# 🔐 Dependency Security

Finding a package is only the first step.

Before adding a production dependency:

```text
Package
  ↓
Publisher
  ↓
Maintenance
  ↓
Version
  ↓
License
  ↓
Security
  ↓
Transitive Dependencies
  ↓
Performance
  ↓
Compatibility
  ↓
Approval
```

Recommended controls:

```text
Dependency Scanning
License Scanning
Secret Scanning
SAST
SBOM
Lock Files
Private Registry
CI/CD Security Gates
```

---

# 🔒 Lock Files

Common lock/resolution files:

| Ecosystem | Lock / Resolution File |
|---|---|
| npm | `package-lock.json` |
| pnpm | `pnpm-lock.yaml` |
| Yarn | `yarn.lock` |
| Python / uv | `uv.lock` |
| PHP | `composer.lock` |
| Rust | `Cargo.lock` |
| Go | `go.sum` |
| Ruby | `Gemfile.lock` |
| Dart | `pubspec.lock` |
| Gradle | Dependency locking / version catalogs |
| .NET | `packages.lock.json` when enabled |
| Swift | Package resolution state |

---

# 🧱 Reproducible Dependency Workflow

```text
Official Registry
       ↓
Package Search
       ↓
Dependency Evaluation
       ↓
Add Dependency
       ↓
Lock / Resolve
       ↓
Build
       ↓
Test
       ↓
Security Scan
       ↓
Code Review
       ↓
CI/CD
       ↓
Production
```

---

# 🚫 Anti-Patterns

```text
❌ Treating a third-party index as the official registry
❌ Installing packages without checking the publisher
❌ Ignoring lock files
❌ Mixing package managers unnecessarily
❌ Blindly copying dependency versions
❌ Using abandoned packages without justification
❌ Ignoring transitive dependencies
❌ Ignoring licenses
❌ Ignoring vulnerabilities
❌ Installing packages globally for application dependencies
❌ Using untrusted registries
```

---

# 🏆 Enterprise Standard

> **Official registry first. Third-party discovery second.**

Recommended decision process:

```text
1. Find package
       ↓
2. Check official registry
       ↓
3. Verify publisher / maintainer
       ↓
4. Review versions
       ↓
5. Review dependencies
       ↓
6. Check security
       ↓
7. Check license
       ↓
8. Add dependency
       ↓
9. Lock resolution
       ↓
10. Validate in CI/CD
```

---

# 📚 Quick Reference

| Ecosystem | Official Search |
|---|---|
| JavaScript / TypeScript | [npm](https://www.npmjs.com/) |
| Java / Kotlin | [Maven Central](https://central.sonatype.com/) |
| Gradle Plugins | [Gradle Plugin Portal](https://plugins.gradle.org/) |
| Android | [Google Maven](https://maven.google.com/) |
| Python | [PyPI](https://pypi.org/) |
| .NET | [NuGet](https://www.nuget.org/) |
| PHP | [Packagist](https://packagist.org/) |
| Dart / Flutter | [pub.dev](https://pub.dev/) |
| Rust | [crates.io](https://crates.io/) |
| Go | [pkg.go.dev](https://pkg.go.dev/) |
| Ruby | [RubyGems](https://rubygems.org/) |
| Swift | [Swift Package Index](https://swiftpackageindex.com/) |
| C / C++ | [vcpkg](https://vcpkg.io/) |
| C / C++ | [ConanCenter](https://conan.io/center) |
| R | [CRAN](https://cran.r-project.org/) |
| Elixir | [Hex](https://hex.pm/) |

---

# ✅ Final Principle

> **The package manager installs and resolves dependencies; the registry is where the ecosystem publishes and distributes packages; the lock/resolution mechanism makes the dependency graph reproducible.**

Keep these concepts separate in enterprise engineering documentation:

```text
Language
   ↓
Runtime / SDK
   ↓
Package Manager
   ↓
Build Tool
   ↓
Official Registry
   ↓
Dependency Lock
   ↓
Security / License Controls
   ↓
CI/CD
   ↓
Production
```
