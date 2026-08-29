# IDE & Developer Tooling Guide

## Industry-Grade IDE and Development Environment Matrix

This document provides an industry-grade mapping of programming
languages and platforms to recommended primary IDEs and strong
alternatives.

> **Selection principle:** The primary IDE is chosen based on
> professional ecosystem support, tooling maturity, debugging, testing,
> build integration, framework support, platform integration, and
> developer/community adoption.
> Note: **Community Edition downlaod**

## Language → IDE Matrix

| Language | Primary IDE | Strong Alternative |
|---|---|---|
| **JavaScript / TypeScript** | **VS Code** | WebStorm |
| **Java** | **IntelliJ IDEA** | Eclipse |
| **Python** | **PyCharm** | VS Code |
| **C#** | **Visual Studio** | Rider |
| **C / C++** | **Visual Studio / CLion** | VS Code |
| **PHP** | **PhpStorm** | VS Code |
| **Kotlin / Java (Android)** | **Android Studio** | IntelliJ IDEA |
| **Swift (iOS / macOS)** | **Xcode** | VS Code* |

\* Xcode is the required Apple development environment for official Apple-platform build, signing, Simulator, and App Store workflows.                                                                
  -----------------------------------------------------------------------------

\* For Apple platform development, Xcode remains the required
first-party development environment for Apple's build, signing,
simulator, profiling, and App Store submission workflows.

------------------------------------------------------------------------

# Recommended Technology Stacks

## JavaScript / TypeScript

``` text
JavaScript / TypeScript
        ↓
      VS Code
        ↓
React / Next.js / Angular / Vue
        ↓
Node.js / NestJS / Express
        ↓
REST / GraphQL
        ↓
PostgreSQL / MySQL
```

**Primary IDE:** VS Code\
**Alternative:** WebStorm

------------------------------------------------------------------------

## Java Enterprise

``` text
Java
 ↓
IntelliJ IDEA
 ↓
Spring Boot / Spring
 ↓
Hibernate / JPA
 ↓
PostgreSQL / MySQL
 ↓
Maven / Gradle
```

**Primary IDE:** IntelliJ IDEA\
**Alternative:** Eclipse

------------------------------------------------------------------------

## Python

``` text
Python
 ↓
PyCharm
 ├── Django
 ├── FastAPI
 └── Flask
 ↓
SQLAlchemy / Django ORM
 ↓
PostgreSQL
 ↓
Pytest
```

### Framework selection

  Requirement                         Recommended Technology
  ----------------------------------- ------------------------
  Full-featured web application       Django
  Modern REST API / Microservices     FastAPI
  Lightweight web application / API   Flask
  ORM / Database Toolkit              SQLAlchemy
  Background / Distributed Tasks      Celery

**Primary IDE:** PyCharm\
**Alternative:** VS Code

------------------------------------------------------------------------

## C# / .NET

``` text
C#
 ↓
Visual Studio / Rider
 ↓
.NET
 ├── ASP.NET Core
 ├── Blazor
 ├── .NET MAUI
 ├── WPF
 └── WinUI
 ↓
Entity Framework Core
 ↓
SQL Server / PostgreSQL
```

**Primary IDE:** Visual Studio\
**Alternative:** JetBrains Rider

------------------------------------------------------------------------

## PHP

``` text
PHP
 ↓
PhpStorm
 ↓
Laravel / Symfony
 ↓
Eloquent / Doctrine
 ↓
MySQL / PostgreSQL
 ↓
PHPUnit / Pest
```

**Primary IDE:** PhpStorm\
**Alternative:** VS Code

------------------------------------------------------------------------

## Android

``` text
Android
 ↓
Android Studio
 ↓
Kotlin
 ↓
Jetpack Compose
 ↓
Android SDK
 ↓
Gradle
 ↓
JUnit / Espresso
```

**Primary IDE:** Android Studio\
**Alternative:** IntelliJ IDEA

------------------------------------------------------------------------

## iOS / macOS

``` text
Apple Platforms
 ↓
Xcode
 ↓
Swift
 ↓
SwiftUI / UIKit
 ↓
Apple SDKs
 ↓
Swift Concurrency
 ↓
XCTest
```

**Primary IDE:** Xcode

------------------------------------------------------------------------

## Cross-platform Mobile

``` text
Dart
 ↓
VS Code / Android Studio
 ↓
Flutter
 ↓
Android + iOS + Web + Desktop
```

**Primary IDE:** VS Code / Android Studio\
**Alternative:** IntelliJ IDEA

------------------------------------------------------------------------

## C / C++

``` text
C / C++
 ↓
Visual Studio / CLion
 ↓
CMake
 ↓
GDB / LLDB
 ↓
GoogleTest
 ↓
Sanitizers / Valgrind
```

**Primary IDE:** Visual Studio or CLion\
**Alternative:** VS Code

------------------------------------------------------------------------

## Go

``` text
Go
 ↓
GoLand
 ↓
Gin / Fiber / Echo / Chi
 ↓
PostgreSQL / Redis
 ↓
Docker / Kubernetes
```

**Primary IDE:** GoLand\
**Alternative:** VS Code

------------------------------------------------------------------------

## Rust

``` text
Rust
 ↓
RustRover
 ↓
Cargo
 ↓
Tokio
 ↓
Axum / Actix Web
 ↓
PostgreSQL
```

**Primary IDE:** RustRover\
**Alternatives:** VS Code, Zed, Neovim

------------------------------------------------------------------------

## Ruby

``` text
Ruby
 ↓
RubyMine
 ↓
Ruby on Rails
 ↓
PostgreSQL
 ↓
RSpec
```

**Primary IDE:** RubyMine\
**Alternative:** VS Code

------------------------------------------------------------------------

## Scala

``` text
Scala
 ↓
IntelliJ IDEA
 ↓
Akka / ZIO / Play Framework
 ↓
sbt
 ↓
PostgreSQL / Kafka
```

**Primary IDE:** IntelliJ IDEA\
**Alternative:** Metals + VS Code

------------------------------------------------------------------------

## SQL / Database Engineering

``` text
SQL
 ↓
DataGrip / DBeaver
 ↓
PostgreSQL
MySQL
SQL Server
Oracle
MariaDB
SQLite
```

**Primary IDE:** DataGrip\
**Community alternative:** DBeaver

Database engineers should also be comfortable with vendor-native CLI
tools and Linux terminals.

------------------------------------------------------------------------

## R / Statistics

``` text
R
 ↓
RStudio
 ↓
tidyverse
 ↓
Shiny
 ↓
Quarto / R Markdown
```

**Primary IDE:** RStudio\
**Alternative:** VS Code

------------------------------------------------------------------------

# Terminal and Server Development

## Linux / Server

``` text
Linux
 ↓
Terminal
 ↓
Bash / Zsh
 ↓
Neovim / Vim
 ↓
SSH
 ↓
Git
 ↓
Docker
 ↓
Kubernetes
 ↓
Cloud CLI
```

**Primary development environment:** VS Code\
**Terminal-first alternative:** Neovim

Production engineers should be comfortable working without a graphical
IDE.

------------------------------------------------------------------------

# Terminal-first Development

For developers who prefer terminal-based development:

``` text
Terminal
   ↓
Neovim
   ↓
Git
   ↓
Language Server Protocol (LSP)
   ↓
Debugger
   ↓
Build Tool
   ↓
Test Runner
```

Recommended tools include:

-   Neovim
-   Vim
-   tmux
-   Git
-   ripgrep (`rg`)
-   fd
-   fzf
-   curl
-   jq
-   make
-   CMake
-   language-specific CLI tools

------------------------------------------------------------------------

# Lightweight Editor

## Zed

Zed is a modern, high-performance code editor suitable for developers
who want a lightweight environment with modern editing and collaboration
capabilities.

``` text
Zed
 ↓
LSP
 ↓
Git
 ↓
Language tooling
 ↓
Terminal
```

**Primary use:** Lightweight modern development\
**Alternative:** VS Code

------------------------------------------------------------------------

# IDE Selection Principles

When selecting an IDE for enterprise software development, evaluate:

1.  Language support
2.  Framework support
3.  Debugger quality
4.  Test runner integration
5.  Build system integration
6.  Package manager integration
7.  Git integration
8.  Database tooling
9.  Docker / container integration
10. Kubernetes integration
11. Cloud tooling
12. Static analysis
13. Code navigation
14. Refactoring support
15. Profiling and performance tooling
16. Remote development
17. Plugin ecosystem
18. Community adoption
19. Enterprise support
20. Security and update lifecycle

------------------------------------------------------------------------

# IDE vs Code Editor vs Terminal

These should not be treated as exactly the same thing.

  -------------------------------------------------------------------------
  Category                Examples                Primary Purpose
  ----------------------- ----------------------- -------------------------
  Full IDE                IntelliJ IDEA, Visual   Complete application
                          Studio, PyCharm, Xcode  development

  Code Editor             VS Code, Zed            Flexible code editing +
                                                  extensions

  Terminal Editor         Neovim, Vim             Keyboard/terminal-first
                                                  development

  Database IDE            DataGrip, DBeaver       Database development

  Platform IDE            Android Studio, Xcode   Platform-specific
                                                  development
  -------------------------------------------------------------------------

A senior engineer should generally be able to work across all three
environments:

``` text
IDE
 ↓
Code Editor
 ↓
Terminal
```

The IDE provides productivity and integrated tooling; the terminal
provides portability, automation, remote-server access, CI/CD
compatibility, and production troubleshooting.

------------------------------------------------------------------------

# Recommended Enterprise Standard

For a multi-language engineering organization, a practical standard can
be:

``` text
General Development
    → VS Code

Java
    → IntelliJ IDEA

Python
    → PyCharm

C# / .NET
    → Visual Studio / Rider

PHP
    → PhpStorm

Android
    → Android Studio

iOS / macOS
    → Xcode

C / C++
    → Visual Studio / CLion

Go
    → GoLand

Rust
    → RustRover

Ruby
    → RubyMine

Scala
    → IntelliJ IDEA

Database
    → DataGrip / DBeaver

Linux / Production
    → Terminal + Neovim/Vim
```

## Engineering Principle

> **An IDE is a productivity tool, not a dependency of the software
> itself.**

A professional software engineer should understand the underlying
compiler/interpreter, build system, package manager, test runner,
debugger, version-control system, and command-line tooling instead of
depending entirely on IDE buttons.

This makes developers effective in:

-   Local development
-   Remote development
-   CI/CD
-   Docker containers
-   Kubernetes
-   Cloud environments
-   Production servers
-   Incident troubleshooting
-   Automated builds
-   Infrastructure environments
