# 📏 Coding Standards — Production Engineering Guide

> Organisation-wide coding standards that make code readable, consistent, secure, and reviewable — and the tooling that enforces them automatically: universal rules for every language, an enforcement pipeline (EditorConfig → formatter → linter → type checker → pre-commit → CI), per-language standards with official style guides and ready-to-use configs, naming conventions, documentation comments, secure-coding rules (CWE / OWASP), performance rules, test-code standards, rules for AI-generated code, and how to roll standards out to legacy code.
>
> Related: [05 Software Design](./05-software-design.md) · [07 Version Control & Code Review](./07-version-control.md) · [08 Testing](./08-testing-and-quality.md) · [09 Security](./09-security-engineering.md) · IDE setup: [30](./30-ide-and-developer-tooling.md)–[36](./36-xcode-extensions.md) · Shell: [28](./28-shell-scripting.md)

---

## 📚 Table of Contents

1. [Why Standards Matter](#1-why-standards-matter)
2. [Principles](#2-principles)
3. [Universal Rules (All Languages)](#3-universal-rules-all-languages)
4. [Naming Conventions](#4-naming-conventions)
5. [Functions, Classes, and Files](#5-functions-classes-and-files)
6. [Comments and Documentation](#6-comments-and-documentation)
7. [The Enforcement Pipeline](#7-the-enforcement-pipeline)
8. [JavaScript / TypeScript](#8-javascript--typescript)
9. [Java](#9-java)
10. [Kotlin](#10-kotlin)
11. [Python](#11-python)
12. [C# / .NET](#12-c--net)
13. [Go](#13-go)
14. [Rust](#14-rust)
15. [PHP](#15-php)
16. [Swift](#16-swift)
17. [Dart / Flutter](#17-dart--flutter)
18. [C / C++](#18-c--c)
19. [SQL](#19-sql)
20. [Shell, Dockerfiles, IaC, YAML, Markdown](#20-shell-dockerfiles-iac-yaml-markdown)
21. [Secure Coding Standards](#21-secure-coding-standards)
22. [Performance Coding Rules](#22-performance-coding-rules)
23. [Test Code Standards](#23-test-code-standards)
24. [AI-Generated Code Standards](#24-ai-generated-code-standards)
25. [Dependencies](#25-dependencies)
26. [Exceptions, Waivers, and Legacy Code](#26-exceptions-waivers-and-legacy-code)
27. [Checklists](#27-checklists)
28. [References](#28-references)

---

## 1. Why Standards Matter

| Without standards | With enforced standards |
|---|---|
| Reviews argue about formatting | Reviews focus on design and correctness |
| Every file looks different | Any engineer can read any file |
| Bugs hide in inconsistent patterns | Linters catch whole bug classes automatically |
| Security depends on individual memory | Secure patterns are the default and checked |
| Onboarding takes weeks | Conventions are documented and tool-enforced |

> **Code is read far more often than it is written.** Optimise for the reader — including you, in six months, during an incident.

---

## 2. Principles

| # | Principle |
|---:|---|
| 1 | **Automate everything automatable.** If a rule can be checked by a tool, it MUST be enforced by a tool, not by reviewers. |
| 2 | **Follow the official/community standard for each language** before inventing house rules. |
| 3 | **Formatters are not negotiable.** Accept the formatter's output; don't fight it. |
| 4 | **Consistency within a codebase beats personal preference.** |
| 5 | **Clarity over cleverness.** |
| 6 | **Fail the build, not the reviewer.** Standards violations block merge in CI. |
| 7 | **Standards evolve via pull requests** to the standards themselves, with rationale. |
| 8 | **New code meets the standard; old code improves when touched** (§26). |

---

## 3. Universal Rules (All Languages)

### 3.1 Readability

| Rule | Status |
|---|:---:|
| Use the language's standard formatter with default settings (or a checked-in config) | 🔴 |
| Names reveal intent; no abbreviations except well-known ones (`id`, `url`, `http`) | 🔴 |
| One concept, one name across the codebase (don't mix `customer`/`client`/`user` for the same thing) | 🔴 |
| No magic numbers/strings — use named constants or enums | 🔴 |
| Prefer early returns (guard clauses) over deep nesting | 🟠 |
| Maximum nesting depth ~3; extract functions beyond that | 🟠 |
| Keep line length per formatter default (e.g. 80–120) | 🟠 |
| Positive conditionals (`if isValid`) over double negatives (`if !isNotValid`) | 🟠 |
| No commented-out code — Git remembers | 🔴 |
| No `TODO` without a ticket: `// TODO(PAY-123): remove after migration` | 🟠 |

```python
# ❌ Nested and unclear
def process(o):
    if o:
        if o.status == 3:
            if o.total > 0:
                ship(o)

# ✅ Guard clauses, named constants
def ship_if_ready(order: Order | None) -> None:
    if order is None:
        return
    if order.status is not OrderStatus.PAID:
        return
    if order.total.is_zero():
        return
    ship(order)
```

### 3.2 Correctness

| Rule | Status |
|---|:---:|
| Enable the strictest practical compiler/type-checker settings | 🔴 |
| Treat warnings as errors in CI (with a curated rule set) | 🔴 |
| Handle every error path; no empty catch blocks | 🔴 |
| Use immutable data by default (`const`, `final`, `val`, `readonly`, frozen dataclasses) | 🟠 |
| Avoid null where the language offers alternatives (Optional, nullable types, sum types) | 🟠 |
| Use UTC, RFC 3339, decimal/integer money (chapter 01 §11) | 🔴 |
| Close resources deterministically (`try-with-resources`, `using`, `with`, `defer`) | 🔴 |

### 3.3 Security (summary; details §21)

| Rule | Status |
|---|:---:|
| Never hard-code secrets, keys, tokens, or passwords | 🔴 |
| Parameterised queries only | 🔴 |
| Validate untrusted input at the boundary; encode output for its context | 🔴 |
| No `eval`/dynamic code execution on untrusted data | 🔴 |
| Never log secrets or unnecessary personal data | 🔴 |

### 3.4 File hygiene

```text
🔴 UTF-8 encoding, LF line endings in repo (except Windows-only scripts), final newline
🔴 No trailing whitespace
🔴 Licence/copyright headers only if your organisation requires them (be consistent)
🟠 One public top-level type per file in Java/C#/Kotlin (convention)
```

`.gitattributes` for line endings: see **42-Git-GitHub-GitLab-Bitbucket-Setup.md §6**.

---

## 4. Naming Conventions

### 4.1 Case styles by language

| Element | JS/TS | Java / Kotlin | Python | C# | Go | Rust | PHP | Swift |
|---|---|---|---|---|---|---|---|---|
| Variables | `camelCase` | `camelCase` | `snake_case` | `camelCase` | `camelCase` | `snake_case` | `$camelCase` | `camelCase` |
| Functions / methods | `camelCase` | `camelCase` | `snake_case` | `PascalCase` | `camelCase` / `PascalCase` (exported) | `snake_case` | `camelCase` | `camelCase` |
| Classes / types | `PascalCase` | `PascalCase` | `PascalCase` | `PascalCase` | `PascalCase` (exported) | `PascalCase` | `PascalCase` | `PascalCase` |
| Interfaces | `PascalCase` (no `I` prefix) | `PascalCase` | `PascalCase` (Protocol) | `IPascalCase` | `PascalCase`, often `-er` (`Reader`) | `PascalCase` (trait) | `PascalCase` + `Interface` suffix (PSR) | `PascalCase` (protocol) |
| Constants | `UPPER_SNAKE` or `camelCase` | `UPPER_SNAKE` | `UPPER_SNAKE` | `PascalCase` | `PascalCase`/`camelCase` | `UPPER_SNAKE` | `UPPER_SNAKE` | `camelCase` |
| Enum members | `PascalCase` | `UPPER_SNAKE` | `UPPER_SNAKE` | `PascalCase` | `PascalCase` consts | `PascalCase` | `PascalCase` (8.1 enums) | `camelCase` |
| Files | `kebab-case.ts` or `PascalCase.tsx` (components) | `PascalCase.java` | `snake_case.py` | `PascalCase.cs` | `snake_case.go` | `snake_case.rs` | `PascalCase.php` | `PascalCase.swift` |
| Packages / modules | `kebab-case` | `com.company.lower` | `lower_snake` | `Company.Product.Module` | `lowercase` (short, no underscores) | `snake_case` crates | `Vendor\Package` | `PascalCase` modules |

### 4.2 Naming guidance

| Kind | Guideline | ✅ | ❌ |
|---|---|---|---|
| Booleans | Predicate form | `isActive`, `hasAccess`, `canRetry` | `active`, `flag`, `check` |
| Functions | Verb phrase | `calculateTax`, `sendInvoice` | `tax`, `invoiceStuff` |
| Collections | Plural | `orders`, `lineItems` | `orderList`, `data` |
| Units | Include unit when ambiguous | `timeoutMs`, `sizeBytes`, `amountMinor` | `timeout`, `size`, `amount` |
| Time | Indicate meaning | `createdAt`, `expiresAt` | `date`, `time` |
| Avoid noise words | — | `Customer` | `CustomerData`, `CustomerInfo`, `CustomerManager` |
| Domain language | Use the business's words | `Invoice`, `GSTIN` | Invented synonyms |

### 4.3 API, database, and event naming

| Artifact | Convention |
|---|---|
| REST paths | plural nouns, kebab-case: `/v1/purchase-orders/{id}/line-items` |
| JSON fields | `camelCase` (most common) or `snake_case` — pick one org-wide |
| DB tables | `snake_case`, plural or singular consistently: `purchase_orders` |
| DB columns | `snake_case`; FKs `<entity>_id`; timestamps `created_at`, `updated_at` |
| Indexes | `ix_<table>_<cols>`, unique `ux_…`, FK `fk_<table>_<ref>` |
| Events | past tense, domain-prefixed: `orders.order_placed.v1` |
| Env vars | `UPPER_SNAKE`, app-prefixed: `SHOP_DATABASE_URL` |
| Feature flags | `kebab-case` with area prefix: `checkout-payment-retry` |
| Metrics | OpenTelemetry/Prometheus conventions: `http_server_request_duration_seconds` |

---

## 5. Functions, Classes, and Files

| Guideline | Typical threshold (configure in linters) |
|---|---|
| Function length | Aim ≤ ~30–40 lines; refactor beyond ~60 |
| Parameters | ≤ 3–4; otherwise a parameter object |
| Cyclomatic complexity | ≤ 10 per function (warn), ≤ 15 (error) |
| Cognitive complexity (Sonar) | ≤ 15 per function |
| Class length | Warn beyond ~300–500 lines |
| File length | Warn beyond ~500–800 lines |
| Boolean parameters | Avoid; use two functions or an enum/options object |
| Side effects | Function names reveal them (`saveOrder`, not `getOrder` that also writes) |
| Command–query separation | A function either changes state or returns data, rarely both |

> Thresholds are heuristics. Exceed them deliberately with a comment explaining why, not by accident.

```typescript
// ❌ Boolean trap
createUser(email, true, false);
// ✅ Options object with names
createUser({ email, sendWelcomeEmail: true, isAdmin: false });
```

---

## 6. Comments and Documentation

### 6.1 What to comment

| Comment | Example |
|---|---|
| ✅ **Why** (intent, constraints, trade-offs) | `// PSP returns 200 with error body for some declines; see PSP docs §4.2` |
| ✅ Non-obvious algorithms, invariants, units | `// amounts are in paise (1/100 INR)` |
| ✅ Public API contracts | Docstrings / JSDoc / KDoc / Javadoc |
| ✅ Warnings and links | `// Do not reorder: migration 0042 depends on this enum order` |
| ❌ **What** the code obviously does | `i++ // increment i` |
| ❌ Commented-out code | Delete it |
| ❌ Changelog in comments | Use Git history |

### 6.2 Documentation comment formats

| Language | Format | Generator |
|---|---|---|
| Java | Javadoc | `javadoc` |
| Kotlin | KDoc | Dokka |
| TypeScript / JS | TSDoc / JSDoc | TypeDoc |
| Python | Docstrings (PEP 257; Google or NumPy style) | Sphinx, MkDocs + mkdocstrings |
| C# | XML doc comments `///` | DocFX |
| Go | Doc comments preceding declarations | `go doc`, pkg.go.dev |
| Rust | `///` and `//!` (Markdown) | `rustdoc` |
| PHP | PHPDoc | phpDocumentor |
| Swift | `///` Markdown (DocC) | DocC |
| Dart | `///` | `dart doc` |

### 6.3 Example (Python, Google style)

```python
def calculate_gst(amount: Decimal, rate: Decimal) -> Decimal:
    """Calculate GST for a taxable amount.

    Args:
        amount: Taxable amount in rupees. Must be non-negative.
        rate: GST rate as a fraction (e.g. Decimal("0.18")).

    Returns:
        GST amount rounded half-up to 2 decimal places.

    Raises:
        ValueError: If amount or rate is negative.
    """
```

### 6.4 Repository-level documentation (minimum)

```text
README.md          what it is, how to run, test, deploy; owners; links
CONTRIBUTING.md    branch/commit/PR conventions, local setup, standards
CODEOWNERS         review ownership (chapter 07)
SECURITY.md        how to report vulnerabilities
CHANGELOG.md       release history (Keep a Changelog)
docs/adr/          architecture decisions
.editorconfig      editor basics
```

Details: chapter **22**.

---

## 7. The Enforcement Pipeline

```text
Editor (EditorConfig + IDE plugins, format on save)
   ↓
Pre-commit hooks (format, lint changed files, secret scan)  — fast, local
   ↓
CI on every PR (format check, full lint, type check, tests, SAST, SCA, coverage)  — authoritative
   ↓
Branch protection: required checks must pass before merge
```

| Stage | Purpose | Must be |
|---|---|---|
| Editor | Instant feedback | Convenient |
| Pre-commit | Catch issues before push | Fast (< ~10 s) |
| **CI** | **Source of truth** — nobody can bypass | Complete, reproducible |

### 7.1 EditorConfig (all repos)

```ini
# .editorconfig — https://editorconfig.org
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
indent_style = space
indent_size = 2

[*.{java,kt,kts,py,cs,php,swift,rs}]
indent_size = 4

[*.go]
indent_style = tab

[Makefile]
indent_style = tab

[*.{cmd,bat}]
end_of_line = crlf

[*.md]
trim_trailing_whitespace = false
```

### 7.2 pre-commit framework (polyglot)

```yaml
# .pre-commit-config.yaml — https://pre-commit.com
# Pin `rev` to current release tags; run `pre-commit autoupdate` regularly.
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v5.0.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-json
      - id: check-merge-conflict
      - id: check-added-large-files
        args: ["--maxkb=500"]
      - id: detect-private-key
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.24.0
    hooks:
      - id: gitleaks
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.12.0
    hooks:
      - id: ruff-check
        args: [--fix]
      - id: ruff-format
  - repo: https://github.com/shellcheck-py/shellcheck-py
    rev: v0.10.0.1
    hooks:
      - id: shellcheck
```

> The `rev` values above are examples. Always run `pre-commit autoupdate` to pin the latest releases, and verify hook IDs against each project's README.

JavaScript projects often use **Husky + lint-staged** or **Lefthook** instead. Hooks setup: **42 §7**.

### 7.3 CI quality job (GitHub Actions example)

```yaml
name: quality
on: [pull_request]
jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 24, cache: npm }
      - run: npm ci
      - run: npm run format:check     # prettier --check . (or biome format)
      - run: npm run lint             # eslint . --max-warnings=0
      - run: npm run typecheck        # tsc --noEmit
      - run: npm test -- --coverage
```

> Pin actions to a full commit SHA in production pipelines (supply-chain hardening — chapter 16).

### 7.4 Platform-wide quality tools

| Tool | Role |
|---|---|
| **SonarQube / SonarQube Cloud** | Multi-language quality + security gate, PR decoration |
| **SonarQube for IDE** | Same rules in the editor (see 31–34) |
| **CodeQL** | Semantic security analysis (GitHub) |
| **Semgrep** | Fast, customisable pattern-based SAST |
| **Codacy / DeepSource** | Hosted analysis alternatives |

---

## 8. JavaScript / TypeScript

| Item | Standard |
|---|---|
| Language | **TypeScript** for all new code; `strict: true` |
| Runtime | Node.js **Active LTS** (Node 24 at the time of writing — see **37**/**41** for install) |
| Compiler | **TypeScript 7.0** (native Go-based compiler, GA July 2026) is the performance path; TypeScript 6.x remains for toolchains that depend on the compiler's programmatic API (e.g. Vue, Svelte, Astro, Angular template type-checking) until a stable API ships in 7.x — verify your stack's support before switching |
| Formatter | **Prettier** (or **Biome**) |
| Linter | **ESLint** with flat config + **typescript-eslint** (or Biome lint) |
| Style references | typescript-eslint recommended/strict configs; Google TypeScript Style Guide |
| Tests | Vitest / Jest; Playwright for E2E |

### 8.1 `tsconfig.json` baseline

```json
{
  "compilerOptions": {
    "target": "ES2023",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "noFallthroughCasesInSwitch": true,
    "exactOptionalPropertyTypes": true,
    "useUnknownInCatchVariables": true,
    "forceConsistentCasingInFileNames": true,
    "skipLibCheck": true,
    "isolatedModules": true,
    "verbatimModuleSyntax": true
  }
}
```

> Set `target`/`module` to match your runtime and bundler; frontend frameworks usually ship their own recommended base config.

### 8.2 ESLint flat config (`eslint.config.js`)

```javascript
// @ts-check
import eslint from '@eslint/js';
import tseslint from 'typescript-eslint';

export default tseslint.config(
  eslint.configs.recommended,
  ...tseslint.configs.strictTypeChecked,
  ...tseslint.configs.stylisticTypeChecked,
  {
    languageOptions: {
      parserOptions: { projectService: true, tsconfigRootDir: import.meta.dirname },
    },
    rules: {
      '@typescript-eslint/no-floating-promises': 'error',
      '@typescript-eslint/no-misused-promises': 'error',
      '@typescript-eslint/consistent-type-imports': 'error',
      'no-console': ['warn', { allow: ['warn', 'error'] }],
      eqeqeq: ['error', 'always'],
      complexity: ['warn', 10],
    },
  },
  { ignores: ['dist/', 'coverage/', 'node_modules/'] },
);
```

### 8.3 `package.json` scripts

```json
{
  "scripts": {
    "format": "prettier --write .",
    "format:check": "prettier --check .",
    "lint": "eslint . --max-warnings=0",
    "typecheck": "tsc --noEmit",
    "test": "vitest run"
  }
}
```

### 8.4 TS rules of thumb

```text
🔴 No `any` (use `unknown` + narrowing); no non-null assertions `!` without justification
🔴 Always await or explicitly handle promises (no floating promises)
🔴 Validate external data with a schema (zod, valibot, ajv) — types are erased at runtime
🟠 Prefer `type`/`interface` consistently (team choice); discriminated unions for variants
🟠 Named exports over default exports (better refactoring and grep-ability)
🟠 `const` by default; `let` only when reassigned; never `var`
🟡 Barrel files (`index.ts`) sparingly — they can slow builds and create cycles
```

---

## 9. Java

| Item | Standard |
|---|---|
| Version | Latest **LTS** — Java 25 (Sept 2025) for new projects; Java 21 acceptable; plan migration off 17 and older |
| Style guide | **Google Java Style Guide** (common enterprise default) |
| Formatter | `google-java-format` or Spotless (wrapping it, or Palantir Java Format) |
| Linters / analysis | Checkstyle (style), PMD, SpotBugs (+ FindSecBugs), Error Prone, SonarQube |
| Nullness | JSpecify annotations + NullAway / IDE inspections |
| Build | Gradle (Kotlin DSL) or Maven — see **37** |

### 9.1 Spotless (Gradle Kotlin DSL)

```kotlin
plugins {
    java
    id("com.diffplug.spotless") version "<latest>"
}

spotless {
    java {
        googleJavaFormat()
        removeUnusedImports()
        importOrder()
        target("src/**/*.java")
    }
}
// CI: ./gradlew spotlessCheck check
```

### 9.2 Java rules of thumb

```text
🔴 Use records for immutable data carriers; sealed interfaces for closed hierarchies
🔴 try-with-resources for every AutoCloseable
🔴 Never catch Exception/Throwable broadly unless at a top-level boundary that logs and rethrows/translates
🔴 BigDecimal (with explicit RoundingMode) or long minor units for money; java.time for dates (never java.util.Date)
🟠 Optional for return values only; never for fields/parameters
🟠 Constructor injection; final fields
🟠 Prefer virtual threads (Java 21+) for blocking I/O-heavy workloads where frameworks support them
🟡 Lombok only per team decision (records and IDE generation often suffice)
```

---

## 10. Kotlin

| Item | Standard |
|---|---|
| Style guide | **Kotlin Coding Conventions** (kotlinlang.org); Android Kotlin style guide for Android |
| Formatter / linter | **ktlint** (official style) and/or **detekt** (static analysis, complexity) |
| Plugins | `org.jlleitschuh.gradle.ktlint` or Spotless with ktlint; detekt Gradle plugin |

```text
🔴 val over var; immutable collections by default
🔴 Avoid !! — use safe calls, requireNotNull with message, or redesign
🔴 Coroutines: structured concurrency (no GlobalScope in production code); inject dispatchers
🟠 Sealed interfaces + when-expressions for exhaustive handling
🟠 Data classes for value types; value classes for type-safe IDs (`@JvmInline value class OrderId(val v: String)`)
🟠 Explicit return types on public API
```

---

## 11. Python

| Item | Standard |
|---|---|
| Version | Latest stable CPython supported by your dependencies (3.13/3.14 era); pin via `.python-version` |
| Style guide | **PEP 8** (style), **PEP 257** (docstrings), **PEP 484+** (type hints) |
| Formatter + linter | **Ruff** (`ruff format` + `ruff check`) — replaces Black, isort, Flake8 and many plugins |
| Type checker | **mypy** (strict) or **Pyright** |
| Project/deps | `pyproject.toml`; **uv** or Poetry (see **37**) |
| Tests | pytest |

### 11.1 `pyproject.toml` baseline

```toml
[project]
name = "orders-service"
requires-python = ">=3.12"

[tool.ruff]
line-length = 100
target-version = "py312"

[tool.ruff.lint]
select = [
  "E", "W",   # pycodestyle
  "F",        # pyflakes
  "I",        # isort
  "B",        # flake8-bugbear
  "UP",       # pyupgrade
  "SIM",      # simplify
  "S",        # bandit (security)
  "N",        # pep8-naming
  "C90",      # mccabe complexity
  "PT",       # pytest style
  "RUF",
]
ignore = ["S101"]  # allow assert in tests (scope via per-file-ignores in real config)

[tool.ruff.lint.mccabe]
max-complexity = 10

[tool.ruff.lint.per-file-ignores]
"tests/**" = ["S101"]

[tool.mypy]
strict = true
warn_unreachable = true
```

### 11.2 Python rules of thumb

```text
🔴 Type hints on all public functions; mypy/pyright strict in CI for new code
🔴 No mutable default arguments (def f(x=[]))
🔴 Decimal for money; zoneinfo/UTC-aware datetimes (never naive datetimes in business logic)
🔴 Never use pickle/yaml.load (unsafe) on untrusted data — use yaml.safe_load, JSON
🟠 dataclasses (frozen=True) or Pydantic models for structured data
🟠 pathlib over os.path; f-strings over % / .format (except logging: use lazy %-style args)
🟠 Context managers for resources; explicit encoding="utf-8" when opening text files
```

---

## 12. C# / .NET

| Item | Standard |
|---|---|
| Version | Latest **LTS** — .NET 10 (Nov 2025) for new projects |
| Conventions | Microsoft **C# coding conventions** and .NET design guidelines |
| Formatter | `dotnet format` driven by `.editorconfig` |
| Analyzers | Built-in .NET analyzers (`AnalysisLevel=latest-recommended`), StyleCop.Analyzers, SonarAnalyzer.CSharp |
| Nullability | `<Nullable>enable</Nullable>` |

### 12.1 `Directory.Build.props`

```xml
<Project>
  <PropertyGroup>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
    <AnalysisLevel>latest-recommended</AnalysisLevel>
    <EnforceCodeStyleInBuild>true</EnforceCodeStyleInBuild>
    <GenerateDocumentationFile>true</GenerateDocumentationFile>
  </PropertyGroup>
</Project>
```

### 12.2 `.editorconfig` excerpts for C#

```ini
[*.cs]
dotnet_sort_system_directives_first = true
csharp_style_namespace_declarations = file_scoped:warning
csharp_style_var_when_type_is_apparent = true:suggestion
dotnet_style_readonly_field = true:warning
dotnet_diagnostic.CA2007.severity = none   # ConfigureAwait — not needed in ASP.NET Core apps
```

```text
🔴 async all the way; never .Result / .Wait() on tasks; pass CancellationToken
🔴 using / await using for IDisposable / IAsyncDisposable
🟠 records for immutable DTOs; primary constructors where they aid clarity
🟠 ILogger with message templates (structured logging), not string interpolation
```

CI: `dotnet format --verify-no-changes && dotnet build -warnaserror && dotnet test`

---

## 13. Go

| Item | Standard |
|---|---|
| Style | **Effective Go**, **Go Code Review Comments**, Google Go Style Guide |
| Formatter | `gofmt` / `goimports` (non-negotiable) |
| Vet / lint | `go vet`, **staticcheck**, **golangci-lint** (aggregator) |
| Vulnerabilities | `govulncheck` |

### 13.1 `.golangci.yml` (sketch — verify against your golangci-lint major version)

```yaml
version: "2"
linters:
  enable:
    - errcheck
    - govet
    - staticcheck
    - ineffassign
    - unused
    - gosec
    - revive
    - bodyclose
    - contextcheck
    - errorlint
    - gocyclo
  settings:
    gocyclo:
      min-complexity: 15
formatters:
  enable:
    - gofmt
    - goimports
```

```text
🔴 Check every error; wrap with %w and context; compare with errors.Is/As
🔴 context.Context as first parameter for I/O; respect cancellation
🔴 No goroutine without a defined lifetime (errgroup, WaitGroup, context cancellation)
🟠 Accept interfaces, return concrete types; small interfaces defined by the consumer
🟠 Use internal/ to hide packages; avoid package names like util, common, misc
🟠 Table-driven tests
```

CI: `gofmt -l . | (! grep .) && go vet ./... && golangci-lint run && go test -race ./... && govulncheck ./...`

---

## 14. Rust

| Item | Standard |
|---|---|
| Style | Rust Style Guide; **Rust API Guidelines** for libraries |
| Formatter | `rustfmt` (`cargo fmt`) |
| Linter | **Clippy** (`cargo clippy -- -D warnings`) |
| Supply chain | `cargo audit`, `cargo deny` |

```toml
# Cargo.toml workspace lints
[workspace.lints.rust]
unsafe_code = "forbid"          # relax per-crate with justification where unsafe is required
[workspace.lints.clippy]
pedantic = { level = "warn", priority = -1 }
unwrap_used = "warn"
expect_used = "warn"
```

```text
🔴 No unwrap()/expect() in library or request paths — propagate with ? and typed errors
🔴 Every unsafe block has a // SAFETY: comment explaining invariants
🟠 thiserror for library errors; anyhow for application binaries
🟠 Prefer borrowing over cloning; clone deliberately
```

---

## 15. PHP

| Item | Standard |
|---|---|
| Version | Latest supported PHP 8.x (check php.net supported versions) |
| Style | **PER Coding Style** (PHP-FIG; evolution of PSR-12), PSR-4 autoloading |
| Formatter / fixer | PHP-CS-Fixer or PHP_CodeSniffer (`phpcs`/`phpcbf`); Laravel Pint for Laravel |
| Static analysis | **PHPStan** or **Psalm** (high level, e.g. PHPStan level 8+ for new code) |
| Architecture rules | Deptrac |

```php
<?php

declare(strict_types=1);

namespace App\Billing;

final readonly class Money
{
    public function __construct(
        public int $amountMinor,
        public Currency $currency,
    ) {
        if ($amountMinor < 0) {
            throw new \InvalidArgumentException('Amount cannot be negative');
        }
    }
}
```

```text
🔴 declare(strict_types=1) in every file
🔴 Typed properties, parameters, and return types everywhere
🔴 Prepared statements / query builder bindings only
🟠 final classes by default; readonly properties/classes for value objects
🟠 Enums (8.1+) instead of class constants for closed sets
```

IDE setup: **34-phpstorm-extensions.md**.

---

## 16. Swift

| Item | Standard |
|---|---|
| Style | **Swift API Design Guidelines** (swift.org); Google Swift Style Guide as supplement |
| Formatter | `swift-format` (Apple, ships with recent toolchains) or SwiftFormat |
| Linter | **SwiftLint** |
| Concurrency | Swift 6 language mode with strict concurrency checking for new code |

```text
🔴 Clarity at the point of use; argument labels read as phrases
🔴 No force unwraps (!) or force try (try!) outside tests and provably-safe cases
🟠 struct and enum by default; class when reference semantics are needed
🟠 Sendable correctness; actors for shared mutable state; @MainActor for UI
🟠 Access control explicit: private by default, internal/public deliberately
```

IDE setup: **36-xcode-extensions.md**.

---

## 17. Dart / Flutter

| Item | Standard |
|---|---|
| Style | **Effective Dart** |
| Formatter | `dart format` |
| Linter | `flutter_lints` / `lints` package (recommended set), stricter custom rules in `analysis_options.yaml` |

```yaml
# analysis_options.yaml
include: package:flutter_lints/flutter.yaml
analyzer:
  language:
    strict-casts: true
    strict-inference: true
    strict-raw-types: true
linter:
  rules:
    - prefer_const_constructors
    - avoid_print
    - always_declare_return_types
    - unawaited_futures
```

---

## 18. C / C++

| Item | Standard |
|---|---|
| Guidelines | **C++ Core Guidelines** (Stroustrup & Sutter); SEI **CERT C / CERT C++**; **MISRA C/C++** for safety-critical/embedded |
| Formatter | `clang-format` (with checked-in `.clang-format`) |
| Static analysis | `clang-tidy`, cppcheck, compiler warnings (`-Wall -Wextra -Wpedantic -Werror`) |
| Runtime checks | AddressSanitizer, UndefinedBehaviorSanitizer, ThreadSanitizer in CI |
| Memory safety | Prefer RAII, smart pointers, `std::span`, bounds-checked access; consider memory-safe languages for new components where feasible |

```text
🔴 No raw owning pointers (use std::unique_ptr / std::shared_ptr)
🔴 No unchecked buffer operations (strcpy, sprintf, gets)
🔴 Sanitizer builds in CI for tests
🟠 const-correctness; prefer references over pointers for non-null parameters
```

---

## 19. SQL

| Rule | Example |
|---|---|
| Keywords UPPERCASE (or consistently lowercase — pick one) | `SELECT`, `FROM`, `WHERE` |
| `snake_case` identifiers; no reserved words as names | `order_items` |
| Explicit column lists; never `SELECT *` in application code | `SELECT id, status FROM orders` |
| Explicit `JOIN … ON`; never implicit comma joins | |
| Qualify columns in multi-table queries | `o.id`, `c.email` |
| One statement per migration step; migrations are forward-only and reviewed | |
| Always parameterised from application code | `WHERE id = $1` |
| Formatter / linter | **SQLFluff** (dialect-aware) |

```sql
SELECT
    o.id,
    o.status,
    c.email,
    SUM(oi.quantity * oi.unit_price_minor) AS total_minor
FROM orders AS o
JOIN customers AS c ON c.id = o.customer_id
JOIN order_items AS oi ON oi.order_id = o.id
WHERE o.created_at >= $1
  AND o.tenant_id = $2
GROUP BY o.id, o.status, c.email
ORDER BY o.id
LIMIT 100;
```

Database engineering standards (indexes, migrations, expand/contract): chapter **10**.

---

## 20. Shell, Dockerfiles, IaC, YAML, Markdown

| File type | Formatter | Linter | Key rules |
|---|---|---|---|
| Bash / sh | `shfmt` | **ShellCheck** | `set -Eeuo pipefail`, quote variables, `mktemp`, traps — see **28-shell-scripting.md** |
| PowerShell | — | **PSScriptAnalyzer** | Approved verbs, full cmdlet names in scripts — see **25** |
| Dockerfile | — | **hadolint** | Pin base image digests/tags, non-root `USER`, multi-stage builds, no secrets in layers |
| Terraform / OpenTofu | `terraform fmt` / `tofu fmt` | **TFLint**, Checkov / Trivy (misconfig) | Remote state with locking, pinned provider versions, modules |
| Kubernetes YAML | — | kubeconform, kube-linter, Checkov | Resource requests/limits, probes, non-root, read-only root FS |
| GitHub Actions | — | **actionlint**, zizmor | Pin actions by SHA, least-privilege `permissions:` |
| YAML (general) | Prettier | **yamllint** | 2-space indent, no tabs, quote ambiguous strings (`"on"`, `"yes"`, `"0123"`) |
| Markdown | Prettier | **markdownlint** | One H1, ordered headings, fenced code with language |
| JSON | Prettier | `jq` / schema validation | No comments unless JSONC is explicitly supported |

### Dockerfile baseline

```dockerfile
# syntax=docker/dockerfile:1
FROM node:24-bookworm-slim AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build && npm prune --omit=dev

FROM node:24-bookworm-slim
ENV NODE_ENV=production
WORKDIR /app
COPY --from=build --chown=node:node /app/dist ./dist
COPY --from=build --chown=node:node /app/node_modules ./node_modules
USER node
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

> Pin base images by digest (`@sha256:…`) in production and let Renovate/Dependabot update them.

---

## 21. Secure Coding Standards

Map coding rules to recognised weakness lists so findings are traceable:

- **CWE Top 25 Most Dangerous Software Weaknesses** (MITRE, updated annually): https://cwe.mitre.org/top25/
- **OWASP Top 10** (web), **OWASP API Security Top 10**, **OWASP Top 10 for LLM Applications**
- **OWASP ASVS** for verification requirements; **OWASP Cheat Sheet Series** for implementation

| Weakness class | Coding rule | Detected by |
|---|---|---|
| Injection (SQL, OS command, LDAP) — CWE-89, CWE-78 | Parameterised queries; no shell with untrusted input; allow-lists | SAST (Semgrep, CodeQL), linters |
| Cross-site scripting — CWE-79 | Framework auto-escaping; no `innerHTML`/`dangerouslySetInnerHTML` with untrusted data; CSP | SAST, ESLint security plugins |
| Broken access control / IDOR — CWE-862/863 | Authorise every resource access server-side by owner/tenant | Code review, tests, DAST |
| Path traversal — CWE-22 | Canonicalise and validate paths against a base dir | SAST |
| SSRF — CWE-918 | Allow-list outbound hosts; block internal ranges/metadata endpoints | SAST, review |
| Deserialisation of untrusted data — CWE-502 | Avoid native deserialisers on untrusted input; use JSON with schemas | SAST |
| Hard-coded credentials — CWE-798 | Secret manager; gitleaks/trufflehog in pre-commit and CI | Secret scanning |
| Use of weak crypto — CWE-327 | Approved libraries & algorithms only (e.g. AES-GCM, SHA-256+, Argon2id/bcrypt for passwords) | SAST, review |
| Missing input validation — CWE-20 | Schema validation at boundaries | Review, tests |
| Out-of-bounds write/read — CWE-787/125 | Memory-safe languages or sanitisers + bounds checks | Sanitisers, fuzzing |
| Sensitive data in logs — CWE-532 | Redaction utilities; logging allow-lists | Review, log scanning |

Full secure SDLC: chapter **09**.

---

## 22. Performance Coding Rules

```text
🔴 No N+1 queries — fetch in batches/joins; check ORM query logs in tests
🔴 Paginate all list endpoints and queries (no unbounded SELECTs)
🔴 Timeouts on every network call
🟠 Avoid O(n²) work on user-controlled sizes
🟠 Stream large payloads/files instead of loading into memory
🟠 Reuse clients/connection pools; don't create HTTP/DB clients per request
🟠 Measure before optimising (profilers, benchmarks); keep the benchmark in the repo
🟡 Precompile regexes used in hot paths; avoid excessive allocations in tight loops
```

Benchmarks and profiling: chapter **19**; load testing: **40**.

---

## 23. Test Code Standards

Test code is production code for your confidence — same quality bar.

| Rule | Status |
|---|:---:|
| Test names describe behaviour: `rejects_order_when_cart_is_empty` / `"rejects order when cart is empty"` | 🔴 |
| Arrange–Act–Assert (or Given–When–Then) structure | 🔴 |
| One behaviour per test; multiple asserts allowed if they verify one behaviour | 🟠 |
| No logic (loops/conditionals) in tests — use parameterised/table-driven tests | 🟠 |
| Deterministic: inject clock/random; no sleeps; no dependency on test order | 🔴 |
| No real external services in unit tests; use Testcontainers for integration tests | 🔴 |
| Test data builders/factories instead of copy-pasted fixtures | 🟠 |
| Flaky tests are quarantined and fixed within an agreed SLA, not ignored | 🔴 |
| Coverage: no decrease on changed lines; treat coverage as a signal, not a target | 🟠 |

```typescript
describe('Money.add', () => {
  it('adds amounts in the same currency', () => {
    // Arrange
    const a = Money.of(1000n, 'INR');
    const b = Money.of(250n, 'INR');
    // Act
    const sum = a.add(b);
    // Assert
    expect(sum.equals(Money.of(1250n, 'INR'))).toBe(true);
  });

  it('rejects adding different currencies', () => {
    expect(() => Money.of(1n, 'INR').add(Money.of(1n, 'USD'))).toThrow('Currency mismatch');
  });
});
```

Full testing standards and STLC: chapter **08**.

---

## 24. AI-Generated Code Standards

| Rule | Status |
|---|:---:|
| AI-generated code meets **every** standard in this document; no exemptions | 🔴 |
| The committer owns the code — understands it, can explain it in review, and is accountable for it | 🔴 |
| Same review, CI, SAST, SCA, and test gates as human-written code | 🔴 |
| Verify every new dependency suggested by AI exists, is the intended package, is maintained, and has an acceptable licence | 🔴 |
| Do not paste secrets, credentials, or restricted/customer data into prompts | 🔴 |
| Use only organisation-approved AI tools for the data classification of the code | 🔴 |
| Keep AI-assisted PRs small and focused; split large generated diffs | 🟠 |
| Ask the tool for tests, then verify the tests actually fail when the code is wrong | 🟠 |
| Provide repository context files (conventions, architecture notes) so assistants follow house standards | 🟠 |
| Disclose substantial AI generation in the PR description if your organisation's policy requires it | 🟡 |

---

## 25. Dependencies

| Rule | Status |
|---|:---:|
| Lockfiles committed (`package-lock.json`, `pnpm-lock.yaml`, `uv.lock`, `poetry.lock`, `Cargo.lock` for apps, `go.sum`, `composer.lock`, `Gemfile.lock`) | 🔴 |
| Reproducible installs in CI (`npm ci`, `uv sync --frozen`, `dotnet restore --locked-mode`) | 🔴 |
| Automated update PRs (Dependabot / Renovate) with CI | 🔴 |
| SCA scanning blocks known critical/high vulns without accepted risk | 🔴 |
| New dependency checklist: need, maintenance activity, licence, security history, size, transitive deps | 🟠 |
| Approved licence list (e.g. MIT, Apache-2.0, BSD); copyleft reviewed by legal | 🟠 |
| Prefer standard library over tiny dependencies | 🟠 |

Package manager commands for every ecosystem: **37-package-managers.md**, registries: **37-official-package-registries.md**.

---

## 26. Exceptions, Waivers, and Legacy Code

### 26.1 Suppressing a rule

```typescript
// eslint-disable-next-line @typescript-eslint/no-explicit-any -- third-party SDK has no types (PAY-311)
```

```python
result = subprocess.run(cmd, check=True)  # noqa: S603 -- cmd is a fixed allow-listed list, no user input
```

Rules:

```text
🔴 Suppress at the narrowest scope (line, not file or project)
🔴 Always include a reason and, where applicable, a ticket
🟠 Review suppressions periodically; count them as a quality metric
```

### 26.2 Adopting standards in a legacy codebase

```text
1. Add formatter + linter with the target configuration
2. Generate a BASELINE of existing violations (many tools support this:
   detekt baseline, PHPStan baseline, Sonar "new code" period, ESLint via bulk suppressions or
   eslint-plugin-baseline-style tooling, Ruff per-file ignores)
3. CI enforces: no NEW violations; changed files must not get worse
4. Format the whole repo in ONE dedicated commit; add it to .git-blame-ignore-revs
5. Boy-scout rule: fix violations in files you touch
6. Burn down the baseline with scheduled tech-debt time (chapter 01 §18)
```

```bash
# .git-blame-ignore-revs — keep blame useful after mass reformatting
echo "<sha-of-format-commit>  # Apply Prettier to entire repo" >> .git-blame-ignore-revs
git config blame.ignoreRevsFile .git-blame-ignore-revs
```

### 26.3 Changing the standard

```text
Propose via PR to this document (or the shared config package) → discussion → approval by
tech leads → publish new shared config version → repos upgrade via Renovate/Dependabot.
```

Share configs as packages (e.g. `@company/eslint-config`, a Gradle convention plugin, a shared `ruff.toml`) so every repo upgrades consistently.

---

## 27. Checklists

### New repository
- [ ] `.editorconfig`, `.gitattributes`, `.gitignore`
- [ ] Formatter and linter configured from the shared org config
- [ ] Strict compiler/type-checker settings
- [ ] Pre-commit hooks (format, lint, secret scan)
- [ ] CI quality job: format check, lint (zero warnings), type check, tests, SAST, SCA
- [ ] Branch protection requires quality checks
- [ ] README, CONTRIBUTING, CODEOWNERS, SECURITY.md
- [ ] Dependency update bot enabled

### Pull request (author self-check)
- [ ] Formatter run; linter clean; no new suppressions without reason
- [ ] Names express intent; no magic values
- [ ] Errors handled; resources closed
- [ ] No secrets, no sensitive data in logs
- [ ] Inputs validated; queries parameterised; authorisation checked
- [ ] Tests added/updated, deterministic, readable
- [ ] Public APIs documented
- [ ] AI-generated portions reviewed and understood

---

## 28. References

### Cross-language
- EditorConfig: https://editorconfig.org/
- pre-commit: https://pre-commit.com/
- SonarQube rules: https://rules.sonarsource.com/
- Semgrep: https://semgrep.dev/ · CodeQL: https://codeql.github.com/
- CWE Top 25: https://cwe.mitre.org/top25/
- OWASP Cheat Sheet Series: https://cheatsheetseries.owasp.org/
- SEI CERT Coding Standards: https://wiki.sei.cmu.edu/confluence/display/seccode

### Language standards
- TypeScript: https://www.typescriptlang.org/ · typescript-eslint: https://typescript-eslint.io/ · ESLint: https://eslint.org/ · Prettier: https://prettier.io/ · Biome: https://biomejs.dev/
- TypeScript 7.0 release coverage: https://adtmag.com/articles/2026/07/10/microsoft-bets-typescript-future-on-a-native-compiler.aspx
- Google style guides (Java, TypeScript, Go, Swift, C++…): https://google.github.io/styleguide/
- Kotlin coding conventions: https://kotlinlang.org/docs/coding-conventions.html · ktlint: https://pinterest.github.io/ktlint/ · detekt: https://detekt.dev/
- PEP 8: https://peps.python.org/pep-0008/ · Ruff: https://docs.astral.sh/ruff/ · mypy: https://mypy.readthedocs.io/
- C# coding conventions: https://learn.microsoft.com/dotnet/csharp/fundamentals/coding-style/coding-conventions
- Effective Go: https://go.dev/doc/effective_go · Go Code Review Comments: https://go.dev/wiki/CodeReviewComments · golangci-lint: https://golangci-lint.run/
- Rust API Guidelines: https://rust-lang.github.io/api-guidelines/ · Clippy: https://doc.rust-lang.org/clippy/
- PHP-FIG PER Coding Style: https://www.php-fig.org/per/coding-style/ · PHPStan: https://phpstan.org/
- Swift API Design Guidelines: https://www.swift.org/documentation/api-design-guidelines/ · SwiftLint: https://github.com/realm/SwiftLint
- Effective Dart: https://dart.dev/effective-dart
- C++ Core Guidelines: https://isocpp.github.io/CppCoreGuidelines/
- SQLFluff: https://sqlfluff.com/
- ShellCheck: https://www.shellcheck.net/ · hadolint: https://github.com/hadolint/hadolint · TFLint: https://github.com/terraform-linters/tflint · actionlint: https://github.com/rhysd/actionlint

---

**Previous:** [05 — Software Design](./05-software-design.md) · **Next:** [07 — Version Control](./07-version-control.md)