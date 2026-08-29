# 🚀 Visual Studio Code Extensions

> **Production-grade VS Code baseline for Full Stack JavaScript / TypeScript development.**

## 🎯 Extension Strategy

The goal is **not to install as many extensions as possible**.

A professional VS Code environment should use:
- A small, stable core
- Project-specific extensions only when required
- Well-maintained extensions
- No duplicate functionality
- Minimal configuration conflicts
- Strong support for testing, security, containers, APIs, databases, and remote development

### Status Legend

| Status | Meaning |
|:---:|---|
| 🔴 **REQUIRED** | Recommended baseline for professional development |
| 🟠 **RECOMMENDED** | Strong addition for most teams/projects |
| 🟡 **OPTIONAL** | Install when the project requires it |
| 🟢 **PERSONAL** | Developer preference / productivity |
| ⚪ **BUILT-IN** | VS Code already provides the capability |

---

# 🧩 Core Engineering Extensions

| Extension | Purpose | Status |
|---|---|:---:|
| **GitLens** | Advanced Git history, blame, repositories and code ownership | 🔴 |
| **GitHub Actions** | GitHub Actions workflow support | 🔴 |
| **GitHub Pull Requests** | Pull request and issue workflows | 🔴 |
| **ESLint** | JavaScript / TypeScript linting | 🔴 |
| **Prettier - Code formatter** | Consistent code formatting | 🔴 |
| **EditorConfig** | Consistent editor configuration across tools | 🔴 |
| **Error Lens** | Inline diagnostics and errors | 🟠 |
| **Code Spell Checker** | Detect spelling mistakes in code and documentation | 🟠 |
| **Path Intellisense** | File-path autocomplete | 🟠 |
| **DotENV** | `.env` syntax highlighting and support | 🔴 |

---

# ⚛️ JavaScript / TypeScript

| Extension | Purpose | Status |
|---|---|:---:|
| **ESLint** | Static analysis and code-quality rules | 🔴 |
| **Prettier - Code formatter** | Formatting | 🔴 |
| **Path Intellisense** | Import/file path completion | 🟠 |
| **npm Intellisense** | npm module import completion | 🟡 |
| **Import Cost** | Shows estimated import size | 🟡 |

> VS Code already includes strong JavaScript and TypeScript language support. Avoid duplicate language-service extensions.

---

# ⚛️ React / React Native

| Extension | Purpose | Status |
|---|---|:---:|
| **React Native Tools** | React Native debugging and development | 🟡 |
| **React snippets** | React code snippets | 🟡 |

> Avoid multiple overlapping React snippet extensions. Choose one maintained extension if needed.

---

# 🎨 HTML / CSS / UI

| Extension | Purpose | Status |
|---|---|:---:|
| **Auto Rename Tag** | Automatically rename paired HTML/JSX tags | 🟠 |
| **Auto Close Tag** | Automatically close HTML/JSX tags | 🟠 |
| **Tailwind CSS IntelliSense** | Tailwind autocomplete and diagnostics | 🟡 |
| **CSS Peek** | Navigate between markup and CSS definitions | 🟡 |
| **Live Preview** | Preview web pages inside VS Code | 🟡 |
| **Live Server** | Local development server | 🟡 |
| **SVG Previewer** | Preview SVG files | 🟢 |

---

# 🧪 Testing

| Extension | Purpose | Status |
|---|---|:---:|
| **Vitest** | Unit/component testing for Vitest projects | 🔴 |
| **Playwright Test** | End-to-end and browser testing | 🔴 |
| **Jest** | Jest integration when required | 🟡 |

Recommended model:

```text
Unit Tests
    ↓
Integration Tests
    ↓
API Tests
    ↓
E2E Tests
    ↓
CI/CD
```

---

# 🔌 API Development

| Extension | Purpose | Status |
|---|---|:---:|
| **REST Client** | Execute HTTP requests from `.http` / `.rest` files | 🔴 |
| **GraphQL** | GraphQL syntax and development support | 🟡 |

---

# 🗄️ Database

| Extension | Purpose | Status |
|---|---|:---:|
| **SQLTools** | Database query and connection tooling | 🟠 |
| **SQLTools PostgreSQL/Cockroach Driver** | PostgreSQL/CockroachDB support | 🟡 |
| **PostgreSQL** | PostgreSQL development support | 🟡 |
| **SQLite** | SQLite development support | 🟡 |
| **Prisma** | Prisma schema and ORM tooling | 🟡 |
| **MongoDB** | MongoDB development support | 🟡 |

> Install database extensions according to the actual database technology used by the project.

---

# 🐳 Containers & Cloud Native

| Extension | Purpose | Status |
|---|---|:---:|
| **Docker** | Dockerfile, image and container workflows | 🔴 |
| **Dev Containers** | Reproducible containerized development environments | 🔴 |
| **Remote - SSH** | Remote development over SSH | 🔴 |
| **Remote Explorer** | Manage remote development targets | 🟠 |
| **Kubernetes** | Kubernetes resource and cluster workflows | 🔴 |
| **YAML** | YAML syntax and validation | 🔴 |

---

# 🔐 Security & Code Quality

| Extension | Purpose | Status |
|---|---|:---:|
| **SonarQube for IDE** | Detect bugs, vulnerabilities and code smells | 🔴 |

> Security extensions complement—not replace—CI security scanning, dependency auditing, secret management, and code review.

---

# 📝 Documentation

| Extension | Purpose | Status |
|---|---|:---:|
| **Markdown All in One** | Markdown editing, navigation and formatting | 🟠 |
| **Code Spell Checker** | Spelling validation | 🟠 |
| **Office Viewer** | Office document viewing | 🟢 |
| **PDF Viewer** | PDF viewing | 🟢 |
| **PPTX Viewer** | PowerPoint viewing | 🟢 |

---

# 🌿 Git & GitHub

| Extension | Purpose | Status |
|---|---|:---:|
| **GitLens** | Git history, blame and repository insights | 🔴 |
| **GitHub Actions** | Workflow authoring and inspection | 🔴 |
| **GitHub Pull Requests** | Pull request and issue management | 🔴 |
| **GitHub Copilot** | AI-assisted development | 🟠 |
| **GitHub Copilot Chat** | AI-assisted coding and explanation | 🟠 |

---

# 🤖 AI-Assisted Development

| Extension | Purpose | Status |
|---|---|:---:|
| **GitHub Copilot** | Code generation and development assistance | 🟠 |
| **GitHub Copilot Chat** | Conversational development assistance | 🟠 |

> AI tools are developer assistance, not a replacement for code review, testing, security analysis, architecture decisions, or engineering judgment.

---

# 🎨 UI, Icons & Themes

| Extension | Purpose | Status |
|---|---|:---:|
| **Material Icon Theme** | File and folder icons | 🟢 |
| **GitHub Theme** | GitHub-style editor theme | 🟢 |
| **Peacock** | Workspace color identification | 🟢 |
| **Bookmarks** | Code navigation bookmarks | 🟢 |
| **Todo Tree** | TODO/FIXME tracking | 🟡 |

---

# 🧰 Developer Utilities

| Extension | Purpose | Status |
|---|---|:---:|
| **Better Comments** | Comment highlighting | 🟢 |
| **Regex Previewer** | Regex development and testing | 🟢 |
| **Image Preview** | Image preview and dimensions | 🟢 |

---

# 🏗️ Monorepo & Architecture

| Extension | Purpose | Status |
|---|---|:---:|
| **Nx Console** | Nx monorepo development | 🟡 |

Install Nx Console only when the project uses Nx.

---

# 🖥️ Remote & Linux Development

Recommended extensions:

- **Remote - SSH**
- **Dev Containers**
- **Remote Explorer**

Terminal skills should include:

```text
Bash
Zsh
Git
SSH
Docker CLI
kubectl
Cloud CLI
Neovim / Vim
```

---

# 🧹 Avoid Redundant Extensions

### React snippets

```text
❌ Multiple React snippet packs
        ↓
Possible duplicate snippets
        ↓
Confusing autocomplete
```

Prefer:

```text
✅ One maintained snippet extension
```

### Formatting

Use one primary formatter per language/project.

```text
Prettier
    +
ESLint
    ↓
Formatting + code quality
```

Avoid competing formatters modifying the same files.

---

# 🏆 Recommended Enterprise Baseline

For a professional Full Stack JavaScript/TypeScript developer:

```text
Git
GitHub
ESLint
Prettier
EditorConfig
TypeScript
Docker
Dev Containers
Remote SSH
Kubernetes
YAML
REST Client
Vitest
Playwright
SonarQube
Markdown
```

Project-specific:

```text
React
React Native
Tailwind
GraphQL
Prisma
PostgreSQL
MongoDB
Nx
Jest
```

Personal productivity:

```text
Themes
Icons
Bookmarks
Todo Tree
Regex Previewer
```

---

# ✅ Engineering Principles

### 1. Prefer fewer, better extensions

More extensions do not automatically mean a better development environment.

### 2. Prefer project-specific configuration

```text
.vscode/
├── settings.json
├── extensions.json
└── tasks.json
```

### 3. Keep formatting deterministic

```text
Developer A
Developer B
Developer C
      ↓
Same formatter
      ↓
Same result
```

### 4. Automate quality checks

```text
Commit
  ↓
Lint
  ↓
Type Check
  ↓
Unit Tests
  ↓
Integration Tests
  ↓
Security Checks
  ↓
Build
  ↓
CI/CD
```

### 5. Never depend entirely on the IDE

A senior engineer should be able to work through:

```text
IDE
Code Editor
Terminal
Git
CLI
CI/CD
Remote Server
Container
```

> **The IDE is a productivity layer over the underlying engineering toolchain.**
