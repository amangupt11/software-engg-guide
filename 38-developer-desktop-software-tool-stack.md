# 🖥️ Software Engineering Developer Workstation — Windows

> **Production-grade fresh Windows laptop software/tool inventory for a modern software engineer.**
>
> Scope: full-stack web, backend, frontend, mobile, desktop, databases, DevOps, cloud, containers, infrastructure, security, testing, and daily engineering operations.

> **Rule:** Install the tool itself from its official vendor/project website or official Microsoft package source. Do not use random download mirrors.

Microsoft currently provides an official Windows Developer Configurations project designed to bootstrap fresh Windows machines into developer workstations, including baseline tools and WSL/Ubuntu. citeturn0search3

---

# 1. Operating System

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **Windows 11 Pro** | Primary workstation OS | Enterprise-grade Windows development environment, Hyper-V, WSL, security and management features | [Microsoft Windows](https://www.microsoft.com/windows/) |

---

# 2. Windows Developer Platform

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **Windows Update** | OS maintenance | Security and platform updates | [Windows Update](https://support.microsoft.com/windows/windows-update) |
| **WinGet** | Software installation | Official Windows package-management CLI | [WinGet](https://learn.microsoft.com/windows/package-manager/winget/) |
| **Windows Developer Mode** | Development | Enables developer-oriented Windows capabilities | [Developer settings](https://learn.microsoft.com/windows/apps/get-started/enable-your-device-for-development) |
| **Microsoft PowerToys** | Productivity | Advanced Windows utilities for developers and power users | [PowerToys](https://learn.microsoft.com/windows/powertoys/) |
| **Dev Drive** | Development storage | Developer-optimized storage for source/build workloads | [Dev Drive](https://learn.microsoft.com/windows/dev-drive/) |

---

# 3. Terminal / Shell

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **Windows Terminal** | Terminal | Modern terminal for PowerShell, CMD and Linux | [Windows Terminal](https://apps.microsoft.com/detail/9n0dx20hk701) |
| **PowerShell 7** | Automation / administration | Modern cross-platform shell and automation language | [PowerShell](https://learn.microsoft.com/powershell/scripting/install/install-powershell-on-windows) |
| **Command Prompt** | Windows compatibility | Native Windows command-line environment | [Windows commands](https://learn.microsoft.com/windows-server/administration/windows-commands/windows-commands) |
| **Git Bash** | Unix-style shell | Bash/Git tooling on Windows | [Git for Windows](https://git-scm.com/download/win) |
| **Ubuntu on WSL** | Linux development | Native Linux userland/toolchain integrated with Windows | [WSL](https://learn.microsoft.com/windows/wsl/install) |

Microsoft's current WSL guidance recommends `wsl --install` for supported Windows versions and documents Ubuntu, Windows Terminal, Git, VS Code, Docker, and databases as part of a WSL development environment. citeturn0search0turn0search2

---

# 4. Version Control / Git

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **Git** | Source control | Distributed version control | [Git](https://git-scm.com/download/win) |
| **Git Credential Manager** | Authentication | Secure Git authentication and credential storage | [GCM](https://github.com/git-ecosystem/git-credential-manager) |
| **GitHub CLI (`gh`)** | GitHub automation | Issues, PRs, Actions and repository operations from terminal | [GitHub CLI](https://cli.github.com/) |
| **Git LFS** | Large files | Version large binary assets efficiently | [Git LFS](https://git-lfs.com/) |

---

# 5. Primary Code Editors / IDEs

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **Visual Studio Code** | Web / full-stack / general development | Lightweight extensible editor | [VS Code](https://code.visualstudio.com/download) |
| **IntelliJ IDEA** | Java / Kotlin | Enterprise JVM development | [IntelliJ IDEA](https://www.jetbrains.com/idea/download/) |
| **PyCharm** | Python | Python application development | [PyCharm](https://www.jetbrains.com/pycharm/download/) |
| **PhpStorm** | PHP | PHP / Laravel development | [PhpStorm](https://www.jetbrains.com/phpstorm/download/) |
| **Visual Studio** | C# / C++ / Windows | .NET, C++, Windows and enterprise development | [Visual Studio](https://visualstudio.microsoft.com/downloads/) |
| **Android Studio** | Android | Android SDK, emulator, Kotlin/Java and Gradle development | [Android Studio](https://developer.android.com/studio) |
| **Xcode** | iOS / macOS | Apple platform development | [Xcode](https://developer.apple.com/xcode/) |
| **DBeaver** | Database development | Cross-database SQL client | [DBeaver](https://dbeaver.io/download/) |

> **Windows limitation:** Xcode requires macOS. It is part of the engineering toolchain for Apple development but cannot be natively installed on a Windows laptop.

---

# 6. VS Code Core Tooling

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **VS Code** | General development | Primary extensible editor | [VS Code](https://code.visualstudio.com/download) |
| **VS Code Remote Development** | WSL / SSH / containers | Develop inside remote Linux and container environments | [Remote Development](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack) |
| **VS Code Dev Containers** | Container development | Reproducible development environments | [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) |
| **VS Code WSL** | Linux development | Run VS Code against WSL | [WSL extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl) |

Microsoft documents VS Code + WSL as a full development environment with Linux toolchains, debugging, testing and Git integration. citeturn0search1

---

# 7. AI Development

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **GitHub Copilot** | AI coding | Code completion, explanation, refactoring and development assistance | [GitHub Copilot](https://github.com/features/copilot) |
| **GitHub Copilot CLI** | Terminal AI | AI-assisted terminal workflows | [GitHub Copilot](https://github.com/features/copilot) |

---

# 8. JavaScript / TypeScript

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **Node.js LTS** | JavaScript runtime | Runtime for Node.js applications and tooling | [Node.js](https://nodejs.org/en/download/) |
| **npm** | Package management | Node.js package registry and dependency management | [npm](https://www.npmjs.com/) |
| **Corepack** | Package manager management | Manage supported package-manager versions where applicable | [Node.js Corepack](https://nodejs.org/api/corepack.html) |
| **pnpm** | Package management | Fast, disk-efficient Node dependency management | [pnpm](https://pnpm.io/installation) |
| **Yarn** | Package management | Alternative Node package manager | [Yarn](https://yarnpkg.com/getting-started/install) |
| **Bun** | JS runtime/tooling | JavaScript/TypeScript runtime, package manager and bundler | [Bun](https://bun.sh/) |
| **Deno** | JS/TS runtime | Secure JavaScript/TypeScript runtime and tooling | [Deno](https://docs.deno.com/runtime/) |

For production projects, select the package manager declared by the repository rather than installing or switching package managers arbitrarily.

Node's official download page currently distinguishes LTS and Current releases; prefer an LTS release for normal production workstation use unless a project explicitly requires another release line. citeturn0search10

---

# 9. Frontend Web Development

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **Google Chrome** | Web development | Primary browser and DevTools | [Chrome](https://www.google.com/chrome/) |
| **Microsoft Edge** | Web development | Chromium browser and DevTools | [Edge](https://www.microsoft.com/edge/download) |
| **Firefox Developer Edition** | Web development | Browser with advanced developer tooling | [Firefox Developer Edition](https://www.mozilla.org/firefox/developer/) |
| **Playwright** | Browser testing | Cross-browser end-to-end testing | [Playwright](https://playwright.dev/) |
| **Selenium** | Browser automation | Cross-browser automation/testing | [Selenium](https://www.selenium.dev/downloads/) |

---

# 10. Java / JVM

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **Eclipse Temurin JDK** | Java development | OpenJDK distribution | [Eclipse Temurin](https://adoptium.net/temurin/releases/) |
| **Oracle JDK** | Java development | Oracle's JDK distribution | [Oracle JDK](https://www.oracle.com/java/technologies/downloads/) |
| **Maven** | Java build/dependencies | Build lifecycle and dependency management | [Maven](https://maven.apache.org/download.cgi) |
| **Gradle** | JVM build/dependencies | Build automation and dependency management | [Gradle](https://gradle.org/install/) |
| **SDKMAN!** | JVM version management | Manage Java/JVM SDK versions, primarily in Unix/WSL environments | [SDKMAN!](https://sdkman.io/install/) |

Prefer project wrappers:

```text
mvnw / mvnw.cmd
gradlew / gradlew.bat
```

instead of relying on one global build-tool version.

---

# 11. Python

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **Python** | Python development | Python runtime | [Python](https://www.python.org/downloads/) |
| **pip** | Python packages | Python package installation | [pip](https://pip.pypa.io/) |
| **venv** | Environment isolation | Project-specific Python environments | [Python venv](https://docs.python.org/3/library/venv.html) |
| **uv** | Python tooling | Fast Python package/project/environment management | [uv](https://docs.astral.sh/uv/) |
| **Poetry** | Python dependency/project management | Dependency resolution and packaging | [Poetry](https://python-poetry.org/docs/) |
| **Ruff** | Python linting/formatting | Fast Python linting and formatting | [Ruff](https://docs.astral.sh/ruff/) |
| **pytest** | Python testing | Testing framework | [pytest](https://docs.pytest.org/) |

---

# 12. C / C++

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **Visual Studio** | Windows C/C++ | Microsoft C/C++ compiler and IDE | [Visual Studio](https://visualstudio.microsoft.com/downloads/) |
| **MSVC** | C/C++ compilation | Microsoft C/C++ toolchain | [MSVC](https://learn.microsoft.com/cpp/build/vscpp-step-0-installation) |
| **Windows SDK** | Windows development | Windows APIs, headers and libraries | [Windows SDK](https://developer.microsoft.com/windows/downloads/windows-sdk/) |
| **CMake** | Cross-platform builds | Build configuration and generation | [CMake](https://cmake.org/download/) |
| **Ninja** | Build execution | Fast build system | [Ninja](https://ninja-build.org/) |
| **LLVM / Clang** | C/C++ toolchain | Compiler, linker and tooling | [LLVM](https://llvm.org/) |
| **vcpkg** | C/C++ packages | C/C++ dependency management | [vcpkg](https://learn.microsoft.com/vcpkg/) |
| **Conan** | C/C++ packages | C/C++ dependency management | [Conan](https://conan.io/) |

---

# 13. C# / .NET

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **.NET SDK** | C# / .NET | Runtime, compiler and CLI | [.NET](https://dotnet.microsoft.com/download) |
| **Visual Studio** | .NET development | Full .NET IDE | [Visual Studio](https://visualstudio.microsoft.com/downloads/) |
| **JetBrains Rider** | .NET development | Cross-platform .NET IDE | [Rider](https://www.jetbrains.com/rider/download/) |
| **NuGet** | Dependencies | .NET package ecosystem | [NuGet](https://www.nuget.org/) |
| **dotnet CLI** | Build / test / publish | .NET command-line tooling | [.NET CLI](https://learn.microsoft.com/dotnet/core/tools/) |

---

# 14. PHP

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **PHP** | PHP development | PHP runtime | [PHP](https://www.php.net/downloads.php) |
| **Composer** | PHP dependencies | PHP dependency management | [Composer](https://getcomposer.org/download/) |
| **Laravel** | PHP backend | Enterprise web/API framework | [Laravel](https://laravel.com/docs) |
| **Symfony** | PHP backend | Enterprise PHP framework | [Symfony](https://symfony.com/doc/current/index.html) |

---

# 15. Android

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **Android Studio** | Android development | Official Android IDE | [Android Studio](https://developer.android.com/studio) |
| **Android SDK** | Android builds | Platform APIs and build tooling | [Android SDK](https://developer.android.com/tools) |
| **Android SDK Platform Tools** | Device debugging | `adb`, `fastboot` and device tooling | [Platform Tools](https://developer.android.com/tools/releases/platform-tools) |
| **Android Emulator** | Device simulation | Android virtual devices | [Android Emulator](https://developer.android.com/studio/run/emulator) |
| **Gradle** | Android builds | Android/JVM build system | [Gradle](https://gradle.org/) |
| **Kotlin** | Android language | Officially supported Android development language | [Kotlin](https://kotlinlang.org/) |

---

# 16. React Native / Cross-Platform Mobile

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **React Native** | Cross-platform mobile | Native Android/iOS applications using React | [React Native](https://reactnative.dev/) |
| **Expo** | React Native development | Development/build/deployment platform | [Expo](https://expo.dev/) |
| **Node.js** | React Native tooling | JavaScript runtime | [Node.js](https://nodejs.org/) |
| **Android Studio** | Android target | SDK/emulator/debugging | [Android Studio](https://developer.android.com/studio) |
| **JDK** | Android build | JVM runtime/toolchain required by Android build tooling | [Eclipse Temurin](https://adoptium.net/temurin/releases/) |

---

# 17. iOS / macOS Development

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **Xcode** | Apple development | Official IDE, SDKs, compiler, Simulator, signing and App Store tooling | [Xcode](https://developer.apple.com/xcode/) |
| **Swift** | Apple development | Swift programming language | [Swift](https://www.swift.org/install/) |
| **Swift Package Manager** | Swift dependencies | Official Swift dependency manager | [Swift Package Manager](https://www.swift.org/documentation/package-manager/) |

> **Windows:** Xcode/iOS Simulator cannot be installed natively on Windows. Use a supported Mac or a properly configured remote macOS build environment.

---

# 18. Databases

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **PostgreSQL** | Relational database | Production-grade SQL database | [PostgreSQL](https://www.postgresql.org/download/) |
| **MySQL** | Relational database | SQL database | [MySQL](https://dev.mysql.com/downloads/) |
| **MariaDB** | Relational database | MySQL-compatible database | [MariaDB](https://mariadb.org/download/) |
| **MongoDB Community Server** | Document database | NoSQL document database | [MongoDB](https://www.mongodb.com/try/download/community) |
| **MongoDB Compass** | MongoDB GUI | Database administration and exploration | [Compass](https://www.mongodb.com/products/tools/compass) |
| **Redis** | Cache / data store | In-memory data structure server | [Redis](https://redis.io/downloads/) |
| **SQLite** | Embedded database | Lightweight embedded SQL database | [SQLite](https://www.sqlite.org/download.html) |
| **DBeaver** | Database GUI | Cross-database SQL client | [DBeaver](https://dbeaver.io/download/) |

> For workstation databases, Docker containers are often preferable when the project already defines database versions through Docker Compose/dev containers.

---

# 19. API Development / Testing

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **Postman** | API development | REST/API testing and collaboration | [Postman](https://www.postman.com/downloads/) |
| **Insomnia** | API development | API client/testing | [Insomnia](https://insomnia.rest/download) |
| **Bruno** | API development | Git-friendly API client | [Bruno](https://www.usebruno.com/downloads) |
| **curl** | CLI HTTP | HTTP/API automation and troubleshooting | [curl](https://curl.se/download.html) |
| **HTTPie** | CLI/API | Human-friendly HTTP client | [HTTPie](https://httpie.io/cli) |

---

# 20. Containers

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **Docker Desktop** | Containers | Local Docker engine and desktop tooling | [Docker Desktop](https://www.docker.com/products/docker-desktop/) |
| **Docker Engine** | Linux containers | Container runtime | [Docker Engine](https://docs.docker.com/engine/install/) |
| **Docker Compose** | Multi-container apps | Local service orchestration | [Docker Compose](https://docs.docker.com/compose/) |
| **Podman** | Containers | Alternative container engine | [Podman](https://podman.io/get-started) |
| **BuildKit / Buildx** | Image builds | Modern Docker image build tooling | [Docker Build](https://docs.docker.com/build/) |

---

# 21. Kubernetes

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **kubectl** | Kubernetes administration | Kubernetes CLI | [kubectl](https://kubernetes.io/docs/tasks/tools/) |
| **Helm** | Kubernetes packages | Kubernetes application/package manager | [Helm](https://helm.sh/docs/intro/install/) |
| **kind** | Local Kubernetes | Kubernetes clusters using containers | [kind](https://kind.sigs.k8s.io/docs/user/quick-start/) |
| **Minikube** | Local Kubernetes | Local Kubernetes development | [Minikube](https://minikube.sigs.k8s.io/docs/start/) |
| **k9s** | Kubernetes terminal UI | Interactive Kubernetes management | [k9s](https://k9scli.io/) |

---

# 22. Cloud CLI Tools

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **AWS CLI** | AWS | AWS resource management | [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) |
| **Azure CLI** | Azure | Azure resource management | [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) |
| **Google Cloud CLI** | GCP | Google Cloud management | [Google Cloud CLI](https://cloud.google.com/sdk/docs/install) |
| **Cloudflare Wrangler** | Cloudflare | Workers / Pages / Cloudflare development | [Wrangler](https://developers.cloudflare.com/workers/wrangler/install-and-update/) |
| **Vercel CLI** | Vercel | Web deployment and management | [Vercel CLI](https://vercel.com/docs/cli) |

---

# 23. Infrastructure as Code

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **Terraform** | Infrastructure as Code | Provision cloud/infrastructure resources | [Terraform](https://developer.hashicorp.com/terraform/install) |
| **OpenTofu** | Infrastructure as Code | Open-source Terraform-compatible IaC | [OpenTofu](https://opentofu.org/docs/intro/install/) |
| **Ansible** | Configuration management | Automation and configuration | [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) |
| **Packer** | Image building | Machine image automation | [Packer](https://developer.hashicorp.com/packer/install) |
| **Pulumi** | Infrastructure as Code | Infrastructure using programming languages | [Pulumi](https://www.pulumi.com/docs/install/) |

Terraform's official CLI documentation covers commands such as `terraform init`, `plan`, `apply`, and `destroy`. citeturn0search11

---

# 24. CI/CD

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **GitHub Actions** | CI/CD | Repository-native automation | [GitHub Actions](https://docs.github.com/actions) |
| **GitLab Runner** | CI/CD | GitLab pipeline execution | [GitLab Runner](https://docs.gitlab.com/runner/install/) |
| **Jenkins** | CI/CD | Self-hosted automation server | [Jenkins](https://www.jenkins.io/download/) |
| **Argo CD** | Kubernetes CD | GitOps continuous delivery | [Argo CD](https://argo-cd.readthedocs.io/en/stable/overview/intro/) |

---

# 25. Observability / Troubleshooting

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **OpenTelemetry** | Observability | Vendor-neutral traces, metrics and logs instrumentation | [OpenTelemetry](https://opentelemetry.io/docs/) |
| **Prometheus** | Metrics | Metrics collection and monitoring | [Prometheus](https://prometheus.io/download/) |
| **Grafana** | Dashboards | Metrics/logs/traces visualization | [Grafana](https://grafana.com/grafana/download/) |
| **Loki** | Logs | Log aggregation | [Loki](https://grafana.com/oss/loki/) |
| **Jaeger** | Tracing | Distributed tracing | [Jaeger](https://www.jaegertracing.io/download/) |

---

# 26. Security / Secrets

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **GnuPG** | Cryptography | Encryption/signing | [GnuPG](https://gnupg.org/download/) |
| **OpenSSL** | TLS / certificates | Cryptographic and TLS tooling | [OpenSSL](https://www.openssl.org/source/) |
| **SOPS** | Secret files | Encrypted configuration/secrets | [SOPS](https://github.com/getsops/sops) |
| **Trivy** | Security scanning | Container, filesystem and dependency vulnerability scanning | [Trivy](https://trivy.dev/) |
| **Gitleaks** | Secret detection | Detect credentials in Git repositories | [Gitleaks](https://github.com/gitleaks/gitleaks) |

---

# 27. Code Quality / Security

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **SonarQube / SonarScanner** | Code quality | Static analysis and quality gates | [SonarQube](https://www.sonarsource.com/products/sonarqube/) |
| **Semgrep** | Static analysis | Code security and pattern analysis | [Semgrep](https://semgrep.dev/docs/getting-started/) |
| **ShellCheck** | Shell scripting | Shell static analysis | [ShellCheck](https://www.shellcheck.net/) |
| **Ruff** | Python | Linting/formatting | [Ruff](https://docs.astral.sh/ruff/) |
| **ESLint** | JS/TS | JavaScript/TypeScript linting | [ESLint](https://eslint.org/) |
| **Prettier** | Formatting | Consistent source formatting | [Prettier](https://prettier.io/) |

---

# 28. Build / Dependency / Package Tools

| Software / Tool | Ecosystem | Purpose | Official Download |
|---|---|---|---|
| **npm** | Node.js | Package management | [npm](https://www.npmjs.com/) |
| **Maven** | Java | Build/dependency management | [Maven](https://maven.apache.org/download.cgi) |
| **Gradle** | Java/Kotlin | Build/dependency management | [Gradle](https://gradle.org/install/) |
| **pip** | Python | Package management | [pip](https://pip.pypa.io/) |
| **NuGet** | .NET | Package management | [NuGet](https://www.nuget.org/) |
| **Composer** | PHP | Package management | [Composer](https://getcomposer.org/download/) |
| **Cargo** | Rust | Package/build management | [Cargo](https://doc.rust-lang.org/cargo/) |
| **Go Modules** | Go | Dependency management | [Go](https://go.dev/ref/mod) |
| **Bundler** | Ruby | Dependency management | [Bundler](https://bundler.io/) |
| **pub** | Dart/Flutter | Dependency management | [pub.dev](https://pub.dev/) |
| **Swift Package Manager** | Swift | Dependency management | [Swift](https://www.swift.org/documentation/package-manager/) |

---

# 29. Linux / WSL Developer Utilities

Recommended inside Ubuntu/WSL:

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **build-essential** | Native compilation | GCC, make and common build tools | [Ubuntu Packages](https://packages.ubuntu.com/) |
| **gcc / g++** | C/C++ | GNU compilers | [GCC](https://gcc.gnu.org/) |
| **make** | Build automation | Traditional build automation | [GNU Make](https://www.gnu.org/software/make/) |
| **CMake** | Build system | Cross-platform build generation | [CMake](https://cmake.org/download/) |
| **jq** | JSON CLI | JSON parsing/transformation | [jq](https://jqlang.org/download/) |
| **yq** | YAML/JSON CLI | Structured configuration processing | [yq](https://mikefarah.gitbook.io/yq/) |
| **ripgrep (`rg`)** | Search | Fast recursive text search | [ripgrep](https://github.com/BurntSushi/ripgrep) |
| **fd** | File search | Fast user-friendly file finder | [fd](https://github.com/sharkdp/fd) |
| **fzf** | CLI productivity | Fuzzy finder | [fzf](https://github.com/junegunn/fzf) |
| **bat** | File viewing | Modern `cat` replacement | [bat](https://github.com/sharkdp/bat) |
| **eza** | File listing | Modern `ls` replacement | [eza](https://github.com/eza-community/eza) |
| **zoxide** | Navigation | Smart directory jumper | [zoxide](https://github.com/ajeetdsouza/zoxide) |

---

# 30. Remote Development

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **OpenSSH** | Remote servers | Secure remote shell and file transfer | [OpenSSH](https://www.openssh.com/) |
| **VS Code Remote - SSH** | Remote development | Develop directly on remote machines | [Remote SSH](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh) |
| **VS Code Remote Development** | WSL / SSH / containers | Unified remote development | [Remote Development](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack) |
| **MobaXterm** | Windows remote administration | SSH, RDP, terminal and Unix utilities | [MobaXterm](https://mobaxterm.mobatek.net/download.html) |

---

# 31. File / Archive Utilities

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **7-Zip** | Archives | ZIP/7z/TAR extraction and compression | [7-Zip](https://www.7-zip.org/download.html) |
| **WinSCP** | File transfer | SFTP/SCP/FTP client | [WinSCP](https://winscp.net/eng/download.php) |
| **Everything** | File search | Extremely fast Windows file search | [Everything](https://www.voidtools.com/downloads/) |

---

# 32. Productivity / Workstation Utilities

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **ShareX** | Screenshots / capture | Developer-friendly screen capture and sharing | [ShareX](https://getsharex.com/downloads) |
| **Obsidian** | Knowledge management | Local Markdown-based engineering notes | [Obsidian](https://obsidian.md/download) |
| **Notepad++** | Quick editing | Lightweight text/source editor | [Notepad++](https://notepad-plus-plus.org/downloads/) |
| **Microsoft PowerToys** | Windows productivity | Window management, launcher, utilities and automation | [PowerToys](https://learn.microsoft.com/windows/powertoys/) |

---

# 33. Browser Engineering

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **Google Chrome** | Web debugging | Chromium DevTools | [Chrome](https://www.google.com/chrome/) |
| **Microsoft Edge** | Web debugging | Chromium DevTools and Windows integration | [Edge](https://www.microsoft.com/edge/download) |
| **Firefox Developer Edition** | Web debugging | Advanced browser development tools | [Firefox Developer Edition](https://www.mozilla.org/firefox/developer/) |

Recommended:

```text
Primary browser → Chrome or Edge
Secondary browser → Firefox Developer Edition
```

---

# 34. Desktop Development

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **Visual Studio** | Windows desktop | WinUI, WPF, WinForms, C++ | [Visual Studio](https://visualstudio.microsoft.com/downloads/) |
| **.NET SDK** | .NET desktop | Runtime and SDK | [.NET](https://dotnet.microsoft.com/download) |
| **Windows App SDK** | Windows apps | Modern Windows application platform | [Windows App SDK](https://learn.microsoft.com/windows/apps/windows-app-sdk/) |
| **Electron** | Cross-platform desktop | Desktop apps using web technologies | [Electron](https://www.electronjs.org/) |
| **Tauri** | Cross-platform desktop | Lightweight desktop apps with web UI and native backend | [Tauri](https://tauri.app/) |

---

# 35. Common CLI Utilities

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **curl** | HTTP | API/download/network testing | [curl](https://curl.se/download.html) |
| **wget** | Downloads | CLI file retrieval | [GNU Wget](https://www.gnu.org/software/wget/) |
| **jq** | JSON | CLI JSON processing | [jq](https://jqlang.org/) |
| **yq** | YAML | CLI YAML processing | [yq](https://mikefarah.gitbook.io/yq/) |
| **OpenSSL** | TLS | Certificate/TLS diagnostics | [OpenSSL](https://www.openssl.org/source/) |
| **netcat** | Networking | TCP/UDP troubleshooting | [Ncat](https://nmap.org/ncat/) |
| **Nmap** | Networking/security | Network discovery and port scanning | [Nmap](https://nmap.org/download.html) |

Use network/security scanning tools only on systems you are authorized to test.

---

# 36. Developer Configuration / Environment Management

| Software / Tool | Use Case | Purpose | Official Download |
|---|---|---|---|
| **Windows Developer Configurations** | Fresh-machine setup | Reproducible Windows developer workstation bootstrap | [Microsoft Developer Configurations](https://learn.microsoft.com/windows/dev-configs/) |
| **mise** | Runtime/tool versions | Manage multiple language/tool versions | [mise](https://mise.jdx.dev/) |
| **asdf** | Runtime/tool versions | Multi-language version management | [asdf](https://asdf-vm.com/) |
| **direnv** | Project environments | Automatically load environment variables per directory | [direnv](https://direnv.net/) |

---

# 37. Recommended Fresh Windows Baseline

For a senior full-stack engineer, the **core workstation** should start with:

```text
Windows 11 Pro
│
├── Windows Update
├── WinGet
├── PowerToys
│
├── Windows Terminal
├── PowerShell 7
├── WSL2
│   └── Ubuntu LTS
│
├── Git
├── Git Credential Manager
├── GitHub CLI
│
├── VS Code
├── IntelliJ IDEA
├── PyCharm
├── PhpStorm
├── Visual Studio
├── Android Studio
│
├── Node.js LTS
├── npm
├── Python
├── JDK
├── .NET SDK
├── PHP
│
├── Docker Desktop
├── kubectl
├── Helm
│
├── PostgreSQL
├── MySQL
├── MongoDB
├── Redis
└── DBeaver
```

---

# 38. Recommended Principle — Install on Windows vs WSL

Use **Windows** for:

```text
IDE GUI
Browser
Windows-specific development
Android Studio
Visual Studio
Desktop applications
Docker Desktop
Productivity applications
```

Use **WSL Ubuntu** for:

```text
Bash
Linux CLI
Node.js server tooling
Python tooling
Linux-native build tools
Terraform
Ansible
Shell scripting
Linux containers
Unix utilities
```

Microsoft explicitly documents WSL as a way to run Linux applications, utilities and Bash tools directly on Windows and recommends WSL + VS Code for Linux-oriented development workflows. citeturn0search1turn0search2

---

# 39. Do Not Install Everything

A production-grade workstation should **not** mean installing every available developer tool.

Use this rule:

```text
Required by current projects
        ↓
Install

Potentially useful
        ↓
Install only when needed

Duplicate functionality
        ↓
Choose one primary tool

Experimental
        ↓
Separate environment
```

For example:

```text
Node package manager:
npm / pnpm / Yarn
        ↓
Use the repository's declared package manager
```

Similarly:

```text
Java:
Maven OR Gradle
        ↓
Follow project build configuration
```

---

# 40. Fresh Laptop Golden Setup

```text
01. Windows Update
02. WinGet
03. Windows Terminal
04. PowerShell 7
05. WSL2 + Ubuntu
06. Git + Credential Manager
07. GitHub CLI
08. VS Code
09. Required IDEs
10. Node.js LTS
11. Python
12. JDK
13. .NET SDK
14. PHP + Composer when required
15. Android Studio when mobile development is required
16. Docker Desktop
17. Database tools
18. Cloud CLIs
19. Kubernetes tools
20. IaC tools
21. Security tools
22. Testing tools
23. Productivity utilities
24. Verify PATH / credentials / versions
25. Clone first project
26. Run build
27. Run tests
28. Run local services
29. Verify debugger
30. Verify container workflow
```

---

# 41. Official-Source Rule

Use:

```text
Vendor / project official website
        ↓
Official documentation
        ↓
Official installer / package manager
```

Avoid:

```text
Random download websites
Unofficial repackaged installers
Unknown GitHub binaries
Cracked software
Unverified package mirrors
```

For Windows software, **WinGet is a preferred installation mechanism when the official package is available there**; Microsoft documents WinGet as Windows' package-management CLI for discovering, installing, upgrading, removing and configuring applications. citeturn0search6turn0search4

---

# 🏁 Final Workstation Philosophy

A senior software-engineering laptop should be:

```text
Reproducible
Secure
Minimal
Version-controlled
Automatable
Observable
Maintainable
Project-independent
Easy to rebuild
```

The ideal target is:

```text
Fresh Windows Laptop
        ↓
Windows Developer Configuration
        ↓
Windows + WSL2
        ↓
Core Engineering Tools
        ↓
Language Toolchains
        ↓
Project-specific Dependencies
        ↓
CI/CD
        ↓
Production Engineering
```

**Do not optimize for the number of applications installed.**

Optimize for:

```text
Correct Tool
+
Correct Version
+
Official Source
+
Reproducible Setup
+
Secure Configuration
+
Project Compatibility
=
Production-grade Developer Workstation
```
