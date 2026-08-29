# 📦 Package Managers & Dependency CLI — Production Engineering Guide

> A production-grade command reference for language and platform package managers used in modern software engineering.

## Scope

This guide covers:

```text
JavaScript / TypeScript → npm
Java                  → Maven / Gradle
Python                → pip
C# / .NET             → NuGet / dotnet
PHP                   → Composer
Rust                  → Cargo
Go                    → Go Modules / go
Ruby                  → RubyGems / Bundler
Dart / Flutter        → dart pub / flutter pub
Swift                 → Swift Package Manager
```

> **Important:** Some ecosystems combine package management with build tooling. For example, Gradle is a build automation system with dependency management, and the Go `go` command manages modules, builds, tests, and installs. Use the project's declared tool and lock/version files rather than introducing a second package-management workflow.

---

# 1. Universal Package-Management Lifecycle

A production dependency lifecycle generally looks like:

```text
Discover
   ↓
Evaluate
   ↓
Add
   ↓
Resolve
   ↓
Lock
   ↓
Install
   ↓
Build
   ↓
Test
   ↓
Audit / Verify
   ↓
Update
   ↓
Publish
   ↓
Release
```

Typical dependency artifacts:

```text
Package manifest
        ↓
Dependency lock / resolution
        ↓
Package registry
        ↓
Local cache
        ↓
Installed dependencies
```

---

# 2. JavaScript / TypeScript — npm

## Check installation

```bash
node --version
npm --version
npm help
npm help install
```

## Initialize project

```bash
npm init
npm init -y
```

## Search packages

```bash
npm search express
npm view express
npm view express version
npm view express versions
npm view express dist-tags
```

## Install dependencies

```bash
npm install
npm install express
npm install express@latest
npm install express@5
```

Short form:

```bash
npm i express
```

Development dependency:

```bash
npm install --save-dev typescript
```

Optional dependency:

```bash
npm install --save-optional package-name
```

Exact version:

```bash
npm install --save-exact package-name@1.2.3
```

## Remove

```bash
npm uninstall express
```

Short form:

```bash
npm rm express
```

## Update

```bash
npm update
npm update express
```

Check outdated:

```bash
npm outdated
```

## Install from lockfile / CI

```bash
npm ci
```

Production dependency installation:

```bash
npm ci --omit=dev
```

> `npm ci` is designed for clean, reproducible installs from the lockfile and is generally preferred in CI when a compatible lockfile is committed.

## Run scripts

```bash
npm run
npm run dev
npm run build
npm test
npm start
```

## Execute binaries

```bash
npx eslint .
npx tsc --noEmit
```

## Audit

```bash
npm audit
npm audit fix
npm audit signatures
```

Review automated fixes before merging.

## Cache

```bash
npm cache verify
npm cache clean --force
```

Avoid unnecessary cache deletion in CI; caching can improve performance.

## Workspaces

```bash
npm install
npm run build --workspace=packages/api
npm install express --workspace=packages/api
npm run test --workspaces
```

## Publish

```bash
npm login
npm whoami
npm pack
npm publish
```

Scoped public package:

```bash
npm publish --access public
```

## Registry

```bash
npm config get registry
npm config set registry https://registry.npmjs.org/
npm view package-name --registry=https://registry.npmjs.org/
```

Official sources:

```text
https://docs.npmjs.com/
https://www.npmjs.com/
```

---

# 3. Java — Maven

Maven uses a project descriptor:

```text
pom.xml
```

## Check

```bash
mvn --version
mvn -version
```

## Help

```bash
mvn help:help
mvn help:effective-pom
mvn help:effective-settings
```

## Validate

```bash
mvn validate
```

## Compile

```bash
mvn compile
```

## Test

```bash
mvn test
```

## Package

```bash
mvn package
```

## Verify

```bash
mvn verify
```

## Install into local repository

```bash
mvn install
```

## Deploy to remote repository

```bash
mvn deploy
```

## Clean

```bash
mvn clean
```

## Common lifecycle

```bash
mvn clean test
mvn clean package
mvn clean verify
mvn clean install
```

Typical lifecycle:

```text
validate
↓
compile
↓
test
↓
package
↓
verify
↓
install
↓
deploy
```

## Dependency tree

```bash
mvn dependency:tree
```

Verbose:

```bash
mvn dependency:tree -Dverbose
```

Specific dependency:

```bash
mvn dependency:tree -Dincludes=org.springframework
```

## Dependency analysis

```bash
mvn dependency:analyze
```

## Download dependencies

```bash
mvn dependency:go-offline
```

## Add dependency

Maven normally manages dependencies by editing `pom.xml`.

Typical structure:

```xml
<dependency>
    <groupId>org.example</groupId>
    <artifactId>example</artifactId>
    <version>1.0.0</version>
</dependency>
```

## Profiles

```bash
mvn test -Ptest
mvn package -Pproduction
```

## Skip tests

```bash
mvn package -DskipTests
```

Use carefully; skipping tests should not become the default production workflow.

## Debug

```bash
mvn -X
```

Quiet:

```bash
mvn -q
```

Offline:

```bash
mvn -o package
```

## Plugin information

```bash
mvn help:describe -Dplugin=org.apache.maven.plugins:maven-compiler-plugin
```

## Wrapper

If the project contains Maven Wrapper:

```bash
./mvnw test
```

Windows:

```cmd
mvnw.cmd test
```

Prefer the repository's wrapper for reproducible builds.

## Central repository

```text
https://central.sonatype.com/
```

Official Maven:

```text
https://maven.apache.org/
```

---

# 4. Java — Gradle

Gradle uses:

```text
build.gradle
build.gradle.kts
settings.gradle
settings.gradle.kts
gradle.properties
```

## Check

```bash
gradle --version
```

## Wrapper

Linux/macOS:

```bash
./gradlew --version
```

Windows:

```cmd
gradlew.bat --version
```

> Prefer the Gradle Wrapper in committed projects.

## Initialize

```bash
gradle init
```

Example:

```bash
gradle init --type java-library
```

## List tasks

```bash
./gradlew tasks
```

## Help

```bash
./gradlew help
./gradlew help --task build
```

## Projects

```bash
./gradlew projects
```

## Build

```bash
./gradlew build
```

## Clean

```bash
./gradlew clean
```

## Test

```bash
./gradlew test
```

## Check

```bash
./gradlew check
```

## Assemble

```bash
./gradlew assemble
```

## Run

```bash
./gradlew run
```

## Dependency tree

```bash
./gradlew dependencies
```

Specific configuration:

```bash
./gradlew dependencies --configuration runtimeClasspath
```

## Dependency insight

```bash
./gradlew dependencyInsight \
  --dependency spring-core \
  --configuration runtimeClasspath
```

## Refresh dependencies

```bash
./gradlew build --refresh-dependencies
```

## Offline

```bash
./gradlew build --offline
```

## Continue after failures

```bash
./gradlew build --continue
```

## Rerun tasks

```bash
./gradlew build --rerun-tasks
```

## Exclude task

```bash
./gradlew build -x test
```

Use only when intentionally required.

## Build cache

```bash
./gradlew build --build-cache
```

Disable:

```bash
./gradlew build --no-build-cache
```

## Dependency locking

```bash
./gradlew dependencies --write-locks
```

## Dependency verification

```bash
./gradlew --write-verification-metadata sha256
```

## Publishing

```bash
./gradlew publish
```

## Wrapper generation

```bash
gradle wrapper
```

Specific version:

```bash
gradle wrapper --gradle-version=9.7.1
```

Official:

```text
https://docs.gradle.org/current/userguide/command_line_interface.html
https://gradle.org/
```

---

# 5. Python — pip

Python packaging commonly uses:

```text
pip
venv
pyproject.toml
requirements.txt
```

## Check

```bash
python --version
python -m pip --version
```

Linux/macOS:

```bash
python3 --version
python3 -m pip --version
```

## Help

```bash
python -m pip --help
python -m pip install --help
```

## Create virtual environment

```bash
python -m venv .venv
```

Linux/macOS:

```bash
source .venv/bin/activate
```

Windows:

```cmd
.venv\Scripts\activate
```

Deactivate:

```bash
deactivate
```

## Search / inspect

Modern pip does not provide the old `pip search` command against PyPI.

Use PyPI:

```text
https://pypi.org/
```

Inspect installed package:

```bash
python -m pip show requests
```

## Install

```bash
python -m pip install requests
```

Specific:

```bash
python -m pip install requests==2.32.0
```

Range:

```bash
python -m pip install "requests>=2.30,<3"
```

## Install requirements

```bash
python -m pip install -r requirements.txt
```

## Install editable package

```bash
python -m pip install -e .
```

Development editable:

```bash
python -m pip install -e ".[dev]"
```

## Upgrade

```bash
python -m pip install --upgrade requests
```

## Uninstall

```bash
python -m pip uninstall requests
```

## List

```bash
python -m pip list
```

Outdated:

```bash
python -m pip list --outdated
```

## Freeze

```bash
python -m pip freeze
```

Export:

```bash
python -m pip freeze > requirements.txt
```

## Check dependency consistency

```bash
python -m pip check
```

## Download packages

```bash
python -m pip download requests
```

## Build package

```bash
python -m pip install build
python -m build
```

## Publish

```bash
python -m pip install twine
python -m twine upload dist/*
```

Official:

```text
https://packaging.python.org/
https://pip.pypa.io/
https://pypi.org/
```

---

# 6. C# / .NET — NuGet

NuGet is the .NET package ecosystem.

Common project files:

```text
*.csproj
*.fsproj
*.sln
Directory.Packages.props
packages.lock.json
```

## .NET CLI

Check:

```bash
dotnet --info
dotnet --version
```

## Add package

```bash
dotnet add package Newtonsoft.Json
```

Specific version:

```bash
dotnet add package Newtonsoft.Json --version 13.0.3
```

Development package:

```bash
dotnet add package PackageName --version 1.0.0
```

## Remove

```bash
dotnet remove package Newtonsoft.Json
```

## List

```bash
dotnet list package
```

Project:

```bash
dotnet list MyApp.csproj package
```

Outdated:

```bash
dotnet list package --outdated
```

## Restore

```bash
dotnet restore
```

## Build

```bash
dotnet build
```

## Test

```bash
dotnet test
```

## Pack

```bash
dotnet pack
```

## Publish

```bash
dotnet publish
```

## NuGet CLI

```bash
nuget help
nuget restore MySolution.sln
nuget install PackageName
nuget pack MyPackage.nuspec
nuget push MyPackage.nupkg
```

## NuGet sources

```bash
dotnet nuget list source
dotnet nuget add source <URL> --name <Name>
dotnet nuget remove source <Name>
dotnet nuget disable source <Name>
dotnet nuget enable source <Name>
```

## Cache

```bash
dotnet nuget locals all --list
dotnet nuget locals all --clear
```

## Package lock

```bash
dotnet restore --locked-mode
```

Official:

```text
https://learn.microsoft.com/nuget/
https://www.nuget.org/
https://learn.microsoft.com/dotnet/core/tools/
```

---

# 7. PHP — Composer

Composer uses:

```text
composer.json
composer.lock
vendor/
```

## Check

```bash
composer --version
composer diagnose
```

## Initialize

```bash
composer init
```

## Search

```bash
composer search laravel
```

## Require

```bash
composer require laravel/framework
```

Development:

```bash
composer require --dev phpunit/phpunit
```

Specific:

```bash
composer require laravel/framework:^12.0
```

## Remove

```bash
composer remove laravel/framework
```

## Install

```bash
composer install
```

Production:

```bash
composer install --no-dev --optimize-autoloader
```

## Update

```bash
composer update
```

Specific:

```bash
composer update laravel/framework
```

## Update without changing unrelated dependencies

```bash
composer update laravel/framework --with-dependencies
```

Use carefully; understand the dependency graph.

## Validate

```bash
composer validate
```

## Diagnose

```bash
composer diagnose
```

## Show dependencies

```bash
composer show
```

Specific:

```bash
composer show laravel/framework
```

## Why dependency exists

```bash
composer why package/name
```

Reverse dependency:

```bash
composer why-not package/name
```

## Dump autoload

```bash
composer dump-autoload
```

Optimized:

```bash
composer dump-autoload -o
```

## Clear cache

```bash
composer clear-cache
```

## Scripts

```bash
composer run-script test
composer run test
```

## Validate lock consistency

```bash
composer validate
composer install
```

## Package repository

```text
https://packagist.org/
```

Official Composer:

```text
https://getcomposer.org/
```

---

# 8. Rust — Cargo

Cargo is Rust's package manager and build tool.

Manifest:

```text
Cargo.toml
Cargo.lock
```

## Check

```bash
cargo --version
cargo --list
cargo --help
```

## New package

Binary:

```bash
cargo new myapp
```

Library:

```bash
cargo new --lib mylib
```

Initialize existing directory:

```bash
cargo init
```

## Add dependency

```bash
cargo add serde
```

Specific:

```bash
cargo add serde@1.0
```

Development dependency:

```bash
cargo add --dev criterion
```

## Remove

```bash
cargo remove serde
```

## Build

```bash
cargo build
```

Release:

```bash
cargo build --release
```

## Check

```bash
cargo check
```

## Test

```bash
cargo test
```

## Run

```bash
cargo run
```

Release:

```bash
cargo run --release
```

## Clean

```bash
cargo clean
```

## Update

```bash
cargo update
```

Specific:

```bash
cargo update -p serde
```

## Dependency tree

```bash
cargo tree
```

## Audit

With cargo-audit installed:

```bash
cargo audit
```

## Format

```bash
cargo fmt
```

Check formatting:

```bash
cargo fmt --check
```

## Lint

```bash
cargo clippy
```

## Documentation

```bash
cargo doc
```

Open:

```bash
cargo doc --open
```

## Package

```bash
cargo package
```

## Publish

```bash
cargo publish
```

## Fetch

```bash
cargo fetch
```

Official:

```text
https://doc.rust-lang.org/cargo/
https://crates.io/
```

---

# 9. Go — Go Modules

Go uses the `go` command for module dependency management, building, testing, and installation.

Manifest:

```text
go.mod
go.sum
```

## Check

```bash
go version
go env
```

## Initialize module

```bash
go mod init example.com/myapp
```

## Add dependency

```bash
go get github.com/gin-gonic/gin
```

Specific:

```bash
go get github.com/gin-gonic/gin@v1.10.0
```

## Update dependencies

```bash
go get -u ./...
```

Patch-oriented:

```bash
go get -u=patch ./...
```

## Remove unused / add required

```bash
go mod tidy
```

## Download

```bash
go mod download
```

## Verify

```bash
go mod verify
```

## Vendor

```bash
go mod vendor
```

## List modules

```bash
go list -m all
```

## Why dependency is present

```bash
go mod graph
```

## Build

```bash
go build ./...
```

## Test

```bash
go test ./...
```

## Install executable

```bash
go install ./cmd/myapp
```

## Format

```bash
gofmt -w .
```

## Vulnerability scan

With official Go vulnerability tooling:

```bash
govulncheck ./...
```

## Package information

```bash
go doc package
```

Official:

```text
https://go.dev/
https://go.dev/ref/mod
https://pkg.go.dev/
```

---

# 10. Ruby — RubyGems / Bundler

Common files:

```text
Gemfile
Gemfile.lock
*.gemspec
```

## Check

```bash
ruby --version
gem --version
bundle --version
```

## Search

```bash
gem search rails
```

## Install gem

```bash
gem install rails
```

Specific:

```bash
gem install rails -v 8.0.0
```

## Uninstall

```bash
gem uninstall rails
```

## List

```bash
gem list
```

## Info

```bash
gem info rails
```

## Bundle install

```bash
bundle install
```

## Add dependency

```bash
bundle add rails
```

Development:

```bash
bundle add rspec --group development,test
```

## Remove

```bash
bundle remove rails
```

## Update

```bash
bundle update
```

Specific:

```bash
bundle update rails
```

## Check outdated

```bash
bundle outdated
```

## Show dependencies

```bash
bundle list
```

## Execute within bundle

```bash
bundle exec ruby app.rb
```

## Check consistency

```bash
bundle check
```

## Package

```bash
gem build mygem.gemspec
```

## Publish

```bash
gem push mygem-1.0.0.gem
```

Official:

```text
https://bundler.io/
https://guides.rubygems.org/
https://rubygems.org/
```

---

# 11. Dart / Flutter — pub

Dart uses `dart pub`.

Flutter applications commonly use:

```bash
flutter pub
```

Manifest:

```text
pubspec.yaml
pubspec.lock
```

## Check

```bash
dart --version
flutter --version
```

## Get dependencies

Dart:

```bash
dart pub get
```

Flutter:

```bash
flutter pub get
```

## Add

```bash
dart pub add http
```

Flutter:

```bash
flutter pub add http
```

Dev dependency:

```bash
dart pub add --dev test
```

## Remove

```bash
dart pub remove http
```

## Upgrade

```bash
dart pub upgrade
```

## Downgrade

```bash
dart pub downgrade
```

## Outdated

```bash
dart pub outdated
```

## Dependency tree

```bash
dart pub deps
```

## Cache

```bash
dart pub cache
```

## Global packages

```bash
dart pub global activate package_name
dart pub global deactivate package_name
dart pub global list
```

## Publish

```bash
dart pub publish
```

## Unpack

```bash
dart pub unpack package_name
```

Official:

```text
https://dart.dev/tools/pub
https://pub.dev/
```

---

# 12. Swift — Swift Package Manager

Swift Package Manager is integrated with the Swift toolchain.

Manifest:

```text
Package.swift
Package.resolved
```

## Check

```bash
swift --version
```

## Initialize

Executable:

```bash
swift package init --type executable
```

Library:

```bash
swift package init --type library
```

## Build

```bash
swift build
```

Release:

```bash
swift build -c release
```

## Test

```bash
swift test
```

## Run

```bash
swift run
```

## Clean

```bash
swift package clean
```

## Resolve dependencies

```bash
swift package resolve
```

## Show dependencies

```bash
swift package show-dependencies
```

## Update dependencies

```bash
swift package update
```

## Reset package state

```bash
swift package reset
```

## Add package dependency

Edit `Package.swift`, then:

```bash
swift package resolve
```

## Archive / release workflows

Swift package publishing normally involves source control tags and a package repository rather than a central command equivalent to `npm publish`.

Official:

```text
https://www.swift.org/documentation/package-manager/
https://github.com/swiftlang/swift-package-manager
https://swiftpackageindex.com/
```

---

# 13. Package Manager Quick Reference

| Ecosystem | Primary Tool | Manifest / Lock | Registry |
|---|---|---|---|
| JavaScript / TypeScript | npm | `package.json` / `package-lock.json` | npm Registry |
| Java | Maven | `pom.xml` | Maven Central |
| Java | Gradle | `build.gradle*` / lock files | Maven Central / other repositories |
| Python | pip | `pyproject.toml` / requirements | PyPI |
| C# / .NET | NuGet / dotnet | `*.csproj` / lock support | NuGet.org |
| PHP | Composer | `composer.json` / `composer.lock` | Packagist |
| Rust | Cargo | `Cargo.toml` / `Cargo.lock` | crates.io |
| Go | Go Modules | `go.mod` / `go.sum` | Go module ecosystem |
| Ruby | Bundler | `Gemfile` / `Gemfile.lock` | RubyGems |
| Dart / Flutter | pub | `pubspec.yaml` / `pubspec.lock` | pub.dev |
| Swift | SwiftPM | `Package.swift` / `Package.resolved` | Swift Package ecosystem |

---

# 14. Production Installation Rules

## JavaScript / TypeScript

Prefer:

```bash
npm ci
```

in CI when `package-lock.json` is authoritative.

## Java / Maven

Prefer the repository's:

```text
mvnw
mvnw.cmd
```

when Maven Wrapper is committed.

## Java / Gradle

Prefer:

```bash
./gradlew
gradlew.bat
```

rather than a globally installed Gradle version.

## Python

Use an isolated environment:

```bash
python -m venv .venv
```

and install declared dependencies.

## .NET

Use:

```bash
dotnet restore
dotnet build
dotnet test
```

with repository-defined package configuration.

## PHP

Production:

```bash
composer install --no-dev --optimize-autoloader
```

Use `composer.lock` for deterministic application installs.

## Rust

Commit:

```text
Cargo.lock
```

for applications and use reproducible Cargo workflows.

## Go

Commit:

```text
go.mod
go.sum
```

and run:

```bash
go mod tidy
```

as part of dependency maintenance.

## Ruby

Commit:

```text
Gemfile.lock
```

for applications and use:

```bash
bundle install
```

## Dart / Flutter

Use:

```bash
dart pub get
```

or:

```bash
flutter pub get
```

with the project's dependency files.

---

# 15. Dependency Update Strategy

Do not blindly execute:

```text
update everything
```

Production workflow:

```text
Inventory
↓
Review release notes
↓
Check compatibility
↓
Check security advisories
↓
Update dependency
↓
Resolve lockfile
↓
Run tests
↓
Build
↓
Security scan
↓
Review
↓
Release
```

---

# 16. Dependency Security

Check for:

```text
Known vulnerabilities
Malicious packages
Typosquatting
Unmaintained packages
Abandoned projects
License conflicts
Transitive vulnerabilities
Compromised maintainers
Unexpected install scripts
```

Useful ecosystem tools include:

```text
npm audit
pip audit
dotnet / NuGet vulnerability tooling
cargo audit
govulncheck
bundle-audit
composer audit
```

Use the official ecosystem's current security tooling and advisories.

---

# 17. Lockfiles

Lockfiles provide a resolved dependency graph.

Examples:

```text
npm          package-lock.json
Python       project-specific lock strategy
Maven        dependency versions in pom / dependency management
Gradle       dependency locking where enabled
.NET         packages.lock.json where enabled
Composer     composer.lock
Rust         Cargo.lock
Go           go.sum
Ruby         Gemfile.lock
Dart         pubspec.lock
Swift        Package.resolved
```

For production applications, review lockfile changes as part of code review.

---

# 18. Dependency Tree / Graph

Always know your transitive dependencies.

Examples:

```bash
npm ls
mvn dependency:tree
./gradlew dependencies
python -m pip list
dotnet list package
composer show
cargo tree
go mod graph
bundle list
dart pub deps
swift package show-dependencies
```

---

# 19. Reproducible Builds

A production build should minimize:

```text
"works on my machine"
```

Use:

```text
Pinned / controlled tool versions
Lockfiles
Wrappers
Immutable CI environments
Verified registries
Dependency checks
Repeatable commands
```

Example:

```text
Source
 ↓
Lockfile
 ↓
Exact dependency graph
 ↓
Controlled toolchain
 ↓
Build
 ↓
Artifact
```

---

# 20. Package Registry Sources

## npm

https://www.npmjs.com/

https://docs.npmjs.com/

## Maven Central

https://central.sonatype.com/

Maven:

https://maven.apache.org/

## Gradle

https://gradle.org/

https://docs.gradle.org/

## PyPI

https://pypi.org/

Python Packaging:

https://packaging.python.org/

## NuGet

https://www.nuget.org/

Microsoft NuGet:

https://learn.microsoft.com/nuget/

## Packagist

https://packagist.org/

Composer:

https://getcomposer.org/

## crates.io

https://crates.io/

Cargo:

https://doc.rust-lang.org/cargo/

## Go Packages

https://pkg.go.dev/

Go Modules:

https://go.dev/ref/mod

## RubyGems

https://rubygems.org/

Bundler:

https://bundler.io/

## pub.dev

https://pub.dev/

Dart pub:

https://dart.dev/tools/pub

## Swift Package Index

https://swiftpackageindex.com/

Swift Package Manager:

https://www.swift.org/documentation/package-manager/

---

# 21. Enterprise Package Registries

Production organizations commonly use internal/private registries.

Examples:

```text
npm private registry
Maven repository manager
Gradle repository
Private PyPI
Private NuGet feed
Private Composer repository
Private Cargo registry
Private Go module proxy
Private RubyGems server
Private Dart package repository
```

Common repository platforms include:

```text
JFrog Artifactory
Sonatype Nexus Repository
GitHub Packages
GitLab Package Registry
Azure Artifacts
AWS CodeArtifact
Google Artifact Registry
```

Use organization-approved registries for proprietary packages.

---

# 22. Repository Configuration

Production teams commonly configure:

```text
Public registry
Private registry
Authentication
Proxy
TLS
Trusted certificates
Repository priority
Cache
Package retention
Access control
```

Never commit registry credentials.

Use:

```text
CI/CD secret stores
Credential managers
Environment injection
OIDC/workload identity
Short-lived credentials
```

where supported.

---

# 23. CI/CD Package Workflow

Generic:

```text
Checkout
↓
Validate manifest
↓
Restore dependencies
↓
Use lockfile
↓
Cache safely
↓
Lint
↓
Test
↓
Security scan
↓
Build
↓
Package
↓
Publish artifact
↓
Deploy
```

Examples:

```bash
npm ci
npm test
npm run build
```

```bash
./mvnw clean verify
```

```bash
./gradlew clean build
```

```bash
python -m pip install -r requirements.txt
pytest
```

```bash
dotnet restore
dotnet test
dotnet publish
```

```bash
composer install --no-dev --optimize-autoloader
```

```bash
cargo test
cargo build --release
```

```bash
go mod download
go test ./...
go build ./...
```

```bash
bundle install
bundle exec rake test
```

```bash
flutter pub get
flutter test
flutter build
```

---

# 24. Production Anti-Patterns

Avoid:

```text
Installing random package versions
Ignoring lockfile changes
Using global packages as application dependencies
Running dependency updates directly in production
Disabling security checks to make builds pass
Committing registry credentials
Using untrusted package mirrors
Blindly accepting major-version upgrades
Ignoring transitive dependencies
Ignoring license requirements
```

---

# 25. Senior Engineer Dependency Checklist

```text
[ ] Correct package manager selected
[ ] Official registry identified
[ ] Manifest committed
[ ] Lockfile strategy defined
[ ] Tool version controlled
[ ] Dependencies reviewed
[ ] Transitive dependencies understood
[ ] Security scanning enabled
[ ] License policy checked
[ ] Private registry configured where required
[ ] Credentials protected
[ ] CI uses reproducible install
[ ] Builds are repeatable
[ ] Dependency updates are reviewed
[ ] Rollback strategy exists
[ ] Production artifact is immutable
```

---

# 26. Official Documentation Principle

> **The package manager's official documentation and registry should be the primary source of truth.**

Use third-party sites for discovery and community information, but verify:

```text
Command syntax
Version behavior
Security guidance
Configuration
Deprecations
Publishing requirements
```

against official documentation before adopting a production workflow.

---

# 🏁 Final Production Principle

Package management is part of the software supply chain.

```text
Developer
   ↓
Manifest
   ↓
Package Registry
   ↓
Dependency Resolution
   ↓
Lockfile
   ↓
Verification
   ↓
Build
   ↓
Security
   ↓
Artifact
   ↓
Deployment
```

A production-grade dependency strategy should be:

```text
Reproducible
Secure
Auditable
Version-controlled
Reviewable
Automated
Recoverable
Maintainable
```
