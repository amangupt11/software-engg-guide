# 🪟 Windows 11 — Fresh Laptop Developer Setup

> A senior software engineer's setup guide for a **brand-new Windows 11 Pro** machine.
> Every tool is installed from its **official vendor source or WinGet**. No mirrors, no third-party bundlers.

**Author perspective:** Written from a 15+ years engineering perspective — installs are minimal, reproducible, and driven by the actual work you do (full-stack web, backend APIs, mobile, containers, cloud, databases).

**Golden rule:** *If a tool is not needed by a current project, don't install it. Install on demand, keep the baseline small.*

---

## 📚 Table of Contents

1. [Hardware Baseline](#1-hardware-baseline)
2. [First Boot — OS Hygiene](#2-first-boot--os-hygiene)
3. [Package Manager — WinGet](#3-package-manager--winget)
4. [Terminal & Shell Stack](#4-terminal--shell-stack)
5. [WSL2 + Ubuntu (Linux inside Windows)](#5-wsl2--ubuntu-linux-inside-windows)
6. [Git & GitHub](#6-git--github)
7. [Editors & IDEs](#7-editors--ides)
8. [Node.js / JavaScript / TypeScript](#8-nodejs--javascript--typescript)
9. [Python](#9-python)
10. [Java / JVM](#10-java--jvm)
11. [.NET](#11-net)
12. [PHP](#12-php)
13. [Go / Rust (optional)](#13-go--rust-optional)
14. [Android Development](#14-android-development)
15. [Databases](#15-databases)
16. [API & HTTP Tools](#16-api--http-tools)
17. [Docker & Kubernetes](#17-docker--kubernetes)
18. [Cloud CLIs](#18-cloud-clis)
19. [Browsers](#19-browsers)
20. [Security & Secrets](#20-security--secrets)
21. [Productivity Apps](#21-productivity-apps)
22. [Verification Script](#22-verification-script)
23. [Backup & Reproducibility](#23-backup--reproducibility)
24. [Final Checklist](#24-final-checklist)

---

## 1. Hardware Baseline

| Component | Minimum | Recommended for pro dev |
|---|---|---|
| CPU | Intel Core i5 / Ryzen 5 | Intel Core i7/i9 or Ryzen 7/9 |
| RAM | 16 GB | **32 GB** (containers + IDEs + DBs) |
| Storage | 512 GB NVMe SSD | **1 TB NVMe SSD** |
| GPU | Integrated | Dedicated (NVIDIA for AI/ML) |
| Display | FHD | 2K/4K, colour-accurate |

**Windows 11 Pro** is preferred over Home for developers: Hyper-V, WSL2 with full networking, BitLocker, Group Policy, Remote Desktop host.

Verify Windows edition and version:

```powershell
winver
```

---

## 2. First Boot — OS Hygiene

Before installing anything else, get the base OS clean and current.

### 2.1 Install every pending Windows update

```
Settings → Windows Update → Check for updates → Install all → Restart
```

Repeat until "You're up to date." appears. A fresh laptop is usually **several months behind**.

### 2.2 Enable core Windows features

Open **PowerShell as Administrator**:

```powershell
# Windows Subsystem for Linux
wsl --install

# Virtual machine platform (needed for WSL2 and Docker Desktop)
Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform -All -NoRestart

# Hyper-V (Pro/Enterprise only)
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All -NoRestart
```

Restart.

### 2.3 Enable Developer Mode

```
Settings → System → For developers → Developer Mode: On
```

Official docs: <https://learn.microsoft.com/windows/apps/get-started/enable-your-device-for-development>

### 2.4 Enable BitLocker (disk encryption)

```
Settings → Privacy & security → Device encryption → On
```

Back up the recovery key to your Microsoft account or a printed vault. Do this **before** you put source code on the machine.

### 2.5 (Optional) Create a Dev Drive

A ReFS-based volume optimised for source and build workloads (~30 % faster on many workflows).

```
Settings → System → Storage → Advanced storage settings → Disks & volumes → Create Dev Drive
```

Official docs: <https://learn.microsoft.com/windows/dev-drive/>

Recommended size: **150–250 GB**. Keep it separate from `C:` so re-installs don't nuke it.

---

## 3. Package Manager — WinGet

WinGet is Microsoft's official Windows package manager and ships with Windows 11. It is the correct way to install nearly every developer tool listed below.

Verify it works:

```powershell
winget --version
winget source list
```

Official docs: <https://learn.microsoft.com/windows/package-manager/winget/>

Update WinGet itself and its sources:

```powershell
winget upgrade Microsoft.AppInstaller
winget source update
```

> **Rule:** Prefer `winget install --id <exact.id>` over `winget install <name>`. IDs are deterministic; names are ambiguous.

---

## 4. Terminal & Shell Stack

| Tool | Official Source | Install |
|---|---|---|
| **Windows Terminal** | <https://learn.microsoft.com/windows/terminal/> | `winget install --id Microsoft.WindowsTerminal` |
| **PowerShell 7 (LTS)** | <https://learn.microsoft.com/powershell/scripting/install/install-powershell-on-windows> | `winget install --id Microsoft.PowerShell` |
| **Git Bash** *(comes with Git)* | <https://git-scm.com/download/win> | Installed in §6 |

Set PowerShell 7 as your default profile inside Windows Terminal (**Settings → Startup → Default profile → PowerShell**).

### 4.1 PowerShell execution policy

Allow signed scripts, not everything:

```powershell
Get-ExecutionPolicy -List
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

Never set `Bypass` as a blanket policy.

### 4.2 Nerd Font (for IDE/terminal glyphs)

Install from an official Nerd Fonts release:

- <https://www.nerdfonts.com/font-downloads> → **CascadiaCode Nerd Font** or **JetBrainsMono Nerd Font**

Set as Windows Terminal font: **Settings → Profile → Appearance → Font face**.

---

## 5. WSL2 + Ubuntu (Linux inside Windows)

WSL2 gives you a real Linux userland for Bash, systemd, containers, and Unix-native tooling.

Official docs: <https://learn.microsoft.com/windows/wsl/install>

```powershell
wsl --install -d Ubuntu-24.04
wsl --set-default-version 2
wsl --list --verbose
```

Inside Ubuntu, after first boot:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential curl git jq unzip zip \
                    ripgrep fd-find fzf htop tmux ca-certificates
```

**When to use Windows vs WSL:**

| Do on Windows | Do inside WSL Ubuntu |
|---|---|
| IDE GUIs (VS Code, Visual Studio, JetBrains) | Bash / Linux CLI |
| Browsers, Postman, Docker Desktop | Node/Python/Go server code the way it runs in prod |
| Windows-specific SDKs | Docker CLI, `kubectl`, cloud CLIs |
| Android Studio, Visual Studio | Terraform, Ansible, shell scripting |

Cross-editing: launch VS Code from WSL with `code .` — the **WSL** extension makes the whole editor run against the Linux side.

---

## 6. Git & GitHub

### 6.1 Install

```powershell
winget install --id Git.Git
winget install --id GitHub.cli
winget install --id GitHub.GitLFS
```

- Git — <https://git-scm.com/download/win>
- GitHub CLI — <https://cli.github.com/>
- Git LFS — <https://git-lfs.com/>
- Git Credential Manager (bundled with Git for Windows) — <https://github.com/git-ecosystem/git-credential-manager>

### 6.2 Identity

```bash
git config --global user.name  "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
git config --global core.autocrlf true          # Windows
git config --global core.editor "code --wait"
git config --global pull.rebase true
git config --global fetch.prune true
```

### 6.3 SSH key (Ed25519)

```powershell
ssh-keygen -t ed25519 -C "you@example.com"

Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
ssh-add $env:USERPROFILE\.ssh\id_ed25519

Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub | Set-Clipboard
```

Add the public key at <https://github.com/settings/keys> → **New SSH key**.

Test:

```bash
ssh -T git@github.com
```

### 6.4 GitHub CLI login

```powershell
gh auth login
gh auth status
```

Enable 2FA on your GitHub account before you write your first line of company code: <https://github.com/settings/security>.

---

## 7. Editors & IDEs

Install only what you actually use.

| Tool | Official Source | WinGet ID |
|---|---|---|
| **Visual Studio Code** | <https://code.visualstudio.com/> | `Microsoft.VisualStudioCode` |
| **Visual Studio 2022 Community/Pro** | <https://visualstudio.microsoft.com/downloads/> | `Microsoft.VisualStudio.2022.Community` |
| **IntelliJ IDEA Ultimate/Community** | <https://www.jetbrains.com/idea/download/> | `JetBrains.IntelliJIDEA.Ultimate` / `.Community` |
| **PyCharm Pro/Community** | <https://www.jetbrains.com/pycharm/download/> | `JetBrains.PyCharm.Professional` / `.Community` |
| **PhpStorm** | <https://www.jetbrains.com/phpstorm/download/> | `JetBrains.PhpStorm` |
| **WebStorm** | <https://www.jetbrains.com/webstorm/download/> | `JetBrains.WebStorm` |
| **Android Studio** | <https://developer.android.com/studio> | `Google.AndroidStudio` |
| **JetBrains Toolbox** *(manages all JetBrains IDEs)* | <https://www.jetbrains.com/toolbox-app/> | `JetBrains.Toolbox` |

**VS Code baseline extensions** (install via UI or `code --install-extension <id>`):

```
dbaeumer.vscode-eslint
esbenp.prettier-vscode
editorconfig.editorconfig
eamodio.gitlens
github.vscode-pull-request-github
github.vscode-github-actions
ms-azuretools.vscode-docker
ms-kubernetes-tools.vscode-kubernetes-tools
ms-vscode-remote.remote-wsl
ms-vscode-remote.remote-ssh
ms-vscode-remote.remote-containers
redhat.vscode-yaml
sonarsource.sonarlint-vscode
mikestead.dotenv
humao.rest-client
```

---

## 8. Node.js / JavaScript / TypeScript

**Do not** install Node.js directly from `winget install Node`. Use a version manager — different projects need different Node versions.

### 8.1 nvm-windows

Official: <https://github.com/coreybutler/nvm-windows/releases>

```powershell
winget install --id CoreyButler.NVMforWindows
```

Reload PowerShell, then:

```powershell
nvm install lts             # currently Node.js 24 LTS
nvm install 22              # keep the previous LTS for legacy projects
nvm use lts
node -v
npm -v
```

Node.js download page: <https://nodejs.org/en/download>

### 8.2 Enable Corepack (manages pnpm / Yarn)

```powershell
corepack enable
```

Official: <https://nodejs.org/api/corepack.html>

### 8.3 Global tools worth installing

Keep this list short — global installs age badly.

```powershell
npm install -g pnpm typescript ts-node
```

For everything else, prefer `npx` or the project's declared package manager.

---

## 9. Python

Use the **official python.org installer** and check "Add Python to PATH".

Official: <https://www.python.org/downloads/windows/>

```powershell
winget install --id Python.Python.3.12
```

Modern Python tooling — pick **one** and stick with it per project:

| Tool | Why | Install | Official |
|---|---|---|---|
| **uv** | Fastest, all-in-one (env + deps + runners) | `winget install --id astral-sh.uv` | <https://docs.astral.sh/uv/> |
| **Poetry** | Mature dependency + packaging | `pipx install poetry` | <https://python-poetry.org/> |
| **Hatch** | PEP-621 native, PyPA-endorsed | `pipx install hatch` | <https://hatch.pypa.io/> |

Install `pipx` for isolated CLI tools:

```powershell
python -m pip install --user pipx
python -m pipx ensurepath
```

Linters / formatters (installed per-project, not globally):

- **Ruff** — <https://docs.astral.sh/ruff/>
- **pytest** — <https://docs.pytest.org/>

---

## 10. Java / JVM

Use **Eclipse Temurin** (Adoptium) — the neutral, production-grade OpenJDK distribution.

Official: <https://adoptium.net/temurin/releases/>

Current LTS lines (as of Sep 2026):

| Version | Status | Recommended for |
|---|---|---|
| **JDK 25 LTS** | Newest LTS (Sep 2025, security support to Sep 2031) | New projects |
| **JDK 21 LTS** | Mature LTS (security support to Dec 2029) | Most enterprise codebases today |
| **JDK 17 LTS** | Older LTS (support to Oct 2027) | Legacy Spring Boot 2.x |

Install (pick one — you can have multiple JDKs and switch):

```powershell
winget install --id EclipseAdoptium.Temurin.25.JDK
winget install --id EclipseAdoptium.Temurin.21.JDK
```

Verify:

```powershell
java -version
javac -version
```

Set `JAVA_HOME` (System Properties → Environment Variables). Example:

```
JAVA_HOME = C:\Program Files\Eclipse Adoptium\jdk-21.x.x-hotspot
```

### 10.1 Build tools

Use the project's **wrappers** (`mvnw`, `gradlew`) whenever they exist. Install globals only for greenfield work:

- Maven — <https://maven.apache.org/download.cgi> — `winget install --id Apache.Maven`
- Gradle — <https://gradle.org/install/> — `winget install --id Gradle.Gradle`

---

## 11. .NET

Official: <https://dotnet.microsoft.com/download>

```powershell
winget install --id Microsoft.DotNet.SDK.8       # LTS (Nov 2023 → Nov 2026)
winget install --id Microsoft.DotNet.SDK.9       # STS
```

Verify:

```powershell
dotnet --info
dotnet --list-sdks
```

For Windows desktop / enterprise .NET work, also install Visual Studio 2022 (§7).

---

## 12. PHP

Only install if you actually work on PHP.

- PHP — <https://windows.php.net/download/>
- Composer — <https://getcomposer.org/download/>

```powershell
winget install --id PHP.PHP
winget install --id Composer.Composer
```

For Laravel/Symfony, PhpStorm (§7) is worth its price.

---

## 13. Go / Rust (optional)

| Tool | Official Source | WinGet ID |
|---|---|---|
| **Go** | <https://go.dev/dl/> | `GoLang.Go` |
| **Rust (rustup)** | <https://www.rust-lang.org/tools/install> | `Rustlang.Rustup` |

```powershell
winget install --id GoLang.Go
winget install --id Rustlang.Rustup
```

---

## 14. Android Development

Only install if you build mobile apps.

1. **Android Studio** — <https://developer.android.com/studio>
   ```powershell
   winget install --id Google.AndroidStudio
   ```
2. Inside Android Studio: **SDK Manager** → install
   - Android SDK Platform (latest API, e.g. 35+)
   - Android SDK Platform-Tools
   - Android SDK Build-Tools
   - Android Emulator
   - At least one system image
3. Set environment variables:
   ```
   ANDROID_HOME       = %LOCALAPPDATA%\Android\Sdk
   ANDROID_SDK_ROOT   = %LOCALAPPDATA%\Android\Sdk
   PATH              += %ANDROID_HOME%\platform-tools;%ANDROID_HOME%\emulator
   ```
4. Verify:
   ```powershell
   adb version
   ```

For **React Native**, also install JDK 17 or 21 (§10) and Node LTS (§8), then follow: <https://reactnative.dev/docs/environment-setup>.

---

## 15. Databases

Prefer **Docker containers** (§17) for local databases — they're disposable, versioned, and match production. Install native only when you need to.

| DB | Official Source | WinGet ID |
|---|---|---|
| **PostgreSQL** | <https://www.postgresql.org/download/windows/> | `PostgreSQL.PostgreSQL.17` |
| **MySQL** | <https://dev.mysql.com/downloads/mysql/> | `Oracle.MySQL` |
| **MongoDB Community** | <https://www.mongodb.com/try/download/community> | `MongoDB.Server` |
| **MongoDB Compass** *(GUI)* | <https://www.mongodb.com/products/tools/compass> | `MongoDB.Compass.Full` |
| **Redis** | Use WSL/Docker (native Windows unsupported) — <https://redis.io/downloads/> | (via WSL) |
| **SQLite** | <https://www.sqlite.org/download.html> | `SQLite.SQLite` |
| **DBeaver Community** *(cross-DB GUI)* | <https://dbeaver.io/download/> | `dbeaver.dbeaver` |

Redis on Windows: **run in WSL2 or Docker**, not the abandoned Microsoft port.

```bash
# inside WSL Ubuntu
sudo apt install redis-server -y
```

---

## 16. API & HTTP Tools

| Tool | Official Source | WinGet ID |
|---|---|---|
| **Postman** | <https://www.postman.com/downloads/> | `Postman.Postman` |
| **Insomnia** | <https://insomnia.rest/download> | `Insomnia.Insomnia` |
| **Bruno** *(Git-friendly)* | <https://www.usebruno.com/downloads> | `Bruno.Bruno` |
| **curl** | Already in Windows 11 | — |
| **HTTPie** | <https://httpie.io/cli> | `HTTPie.HTTPie` |
| **jq** | <https://jqlang.org/download/> | `jqlang.jq` |
| **yq** | <https://mikefarah.gitbook.io/yq/> | `MikeFarah.yq` |

---

## 17. Docker & Kubernetes

### 17.1 Docker Desktop

Official: <https://www.docker.com/products/docker-desktop/>

```powershell
winget install --id Docker.DockerDesktop
```

After install:

- Settings → General → **Use the WSL 2 based engine** ✅
- Settings → Resources → WSL Integration → enable your Ubuntu distro
- Verify: `docker version && docker compose version`

**Licensing note:** Docker Desktop requires a paid subscription for larger companies. Check <https://www.docker.com/pricing/> against your employer's headcount and revenue.

Free alternatives:
- **Rancher Desktop** — <https://rancherdesktop.io/>
- **Podman Desktop** — <https://podman-desktop.io/>

### 17.2 Kubernetes CLI tools

| Tool | Official Source | WinGet ID |
|---|---|---|
| **kubectl** | <https://kubernetes.io/docs/tasks/tools/> | `Kubernetes.kubectl` |
| **Helm** | <https://helm.sh/docs/intro/install/> | `Helm.Helm` |
| **kind** *(local K8s)* | <https://kind.sigs.k8s.io/> | `Kubernetes.kind` |
| **Minikube** | <https://minikube.sigs.k8s.io/> | `Kubernetes.minikube` |
| **k9s** *(TUI)* | <https://k9scli.io/> | `Derailed.k9s` |

---

## 18. Cloud CLIs

Install only the clouds you actually use.

| Cloud | Official Source | WinGet ID |
|---|---|---|
| **AWS CLI v2** | <https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html> | `Amazon.AWSCLI` |
| **Azure CLI** | <https://learn.microsoft.com/cli/azure/install-azure-cli-windows> | `Microsoft.AzureCLI` |
| **Google Cloud CLI** | <https://cloud.google.com/sdk/docs/install> | `Google.CloudSDK` |
| **Cloudflare Wrangler** | <https://developers.cloudflare.com/workers/wrangler/install-and-update/> | (via npm) |
| **Terraform** | <https://developer.hashicorp.com/terraform/install> | `HashiCorp.Terraform` |
| **OpenTofu** | <https://opentofu.org/docs/intro/install/> | `OpenTofu.Tofu` |

---

## 19. Browsers

| Browser | Official Source | WinGet ID |
|---|---|---|
| **Google Chrome** | <https://www.google.com/chrome/> | `Google.Chrome` |
| **Microsoft Edge** | Pre-installed | — |
| **Firefox Developer Edition** | <https://www.mozilla.org/firefox/developer/> | `Mozilla.Firefox.DeveloperEdition` |

Recommended: **Chrome or Edge** as primary debugging browser; **Firefox Developer Edition** as a compatibility second.

---

## 20. Security & Secrets

| Tool | Purpose | Official Source | WinGet ID |
|---|---|---|---|
| **1Password / Bitwarden** | Password manager | <https://1password.com/> / <https://bitwarden.com/> | `AgileBits.1Password` / `Bitwarden.Bitwarden` |
| **GnuPG (Gpg4win)** | Signing / encryption | <https://gnupg.org/download/> | `GnuPG.Gpg4win` |
| **OpenSSL** | TLS / certs | Comes with Git for Windows | — |
| **age** | Modern file encryption | <https://github.com/FiloSottile/age> | `FiloSottile.age` |
| **gitleaks** | Secret scanning | <https://github.com/gitleaks/gitleaks> | `gitleaks.gitleaks` |

**Rules:**
- Never commit `.env` files or private keys.
- Use SSH keys with a passphrase, protected by the OS keychain.
- Enable BitLocker (§2.4) so a stolen laptop cannot leak client code.

---

## 21. Productivity Apps

| Tool | Purpose | Official Source | WinGet ID |
|---|---|---|---|
| **Microsoft PowerToys** | Windows power-user utilities | <https://learn.microsoft.com/windows/powertoys/> | `Microsoft.PowerToys` |
| **7-Zip** | Archives | <https://www.7-zip.org/> | `7zip.7zip` |
| **Everything** | Instant file search | <https://www.voidtools.com/downloads/> | `voidtools.Everything` |
| **Notepad++** | Quick edits | <https://notepad-plus-plus.org/downloads/> | `Notepad++.Notepad++` |
| **ShareX** | Screenshots + GIFs | <https://getsharex.com/downloads> | `ShareX.ShareX` |
| **WinSCP** | SFTP / SCP client | <https://winscp.net/eng/download.php> | `WinSCP.WinSCP` |
| **Obsidian** | Local Markdown knowledge base | <https://obsidian.md/download> | `Obsidian.Obsidian` |
| **Slack / Teams / Zoom** | Team comms | <https://slack.com/> / <https://teams.microsoft.com/> / <https://zoom.us/> | `SlackTechnologies.Slack` / `Microsoft.Teams` / `Zoom.Zoom` |

---

## 22. Verification Script

After every install session, run this to confirm the toolchain is intact. Save as `verify-dev-env.ps1`:

```powershell
# ---- verify-dev-env.ps1 ----
$tools = @(
    @{ Name = "Git";        Cmd = "git --version" }
    @{ Name = "GitHub CLI"; Cmd = "gh --version" }
    @{ Name = "PowerShell"; Cmd = "pwsh -v" }
    @{ Name = "WSL";        Cmd = "wsl --version" }
    @{ Name = "Node";       Cmd = "node -v" }
    @{ Name = "npm";        Cmd = "npm -v" }
    @{ Name = "Python";     Cmd = "python --version" }
    @{ Name = "pip";        Cmd = "python -m pip --version" }
    @{ Name = "Java";       Cmd = "java -version" }
    @{ Name = "dotnet";     Cmd = "dotnet --version" }
    @{ Name = "Docker";     Cmd = "docker --version" }
    @{ Name = "kubectl";    Cmd = "kubectl version --client" }
    @{ Name = "Terraform";  Cmd = "terraform -version" }
    @{ Name = "AWS CLI";    Cmd = "aws --version" }
    @{ Name = "Azure CLI";  Cmd = "az --version" }
)

foreach ($t in $tools) {
    Write-Host "==> $($t.Name)" -ForegroundColor Cyan
    try   { Invoke-Expression $t.Cmd }
    catch { Write-Host "  NOT INSTALLED" -ForegroundColor Red }
    Write-Host ""
}
```

Run it:

```powershell
.\verify-dev-env.ps1
```

---

## 23. Backup & Reproducibility

You want a fresh machine to be reproducible in an afternoon, not a weekend.

### 23.1 Export the WinGet inventory

```powershell
winget export -o "$env:USERPROFILE\Documents\winget-export.json"
```

Restore later with:

```powershell
winget import -i "$env:USERPROFILE\Documents\winget-export.json" --accept-package-agreements --accept-source-agreements
```

Docs: <https://learn.microsoft.com/windows/package-manager/winget/export>

### 23.2 Files to back up

```
%USERPROFILE%\.ssh\
%USERPROFILE%\.gitconfig
%USERPROFILE%\.gitignore_global
%USERPROFILE%\Documents\PowerShell\Microsoft.PowerShell_profile.ps1
%APPDATA%\Code\User\settings.json
%APPDATA%\Code\User\keybindings.json
```

Keep them in a **private GitHub dotfiles repo** (no secrets) plus an encrypted backup for `.ssh` and `.gpg` keys.

### 23.3 System backup

- **File History** (built-in) → external SSD.
- **OneDrive / Google Drive** for `Documents`.
- SSH private keys → 1Password / Bitwarden secure attachments.

---

## 24. Final Checklist

Foundation
- [ ] Windows 11 Pro fully updated
- [ ] Developer Mode on
- [ ] BitLocker on, recovery key backed up
- [ ] Dev Drive created (optional)
- [ ] WinGet updated
- [ ] Windows Terminal + PowerShell 7 installed
- [ ] Execution policy set to `RemoteSigned`
- [ ] WSL2 + Ubuntu 24.04 LTS installed

Version control
- [ ] Git configured (identity, autocrlf, default branch)
- [ ] SSH Ed25519 key generated + added to GitHub
- [ ] `ssh -T git@github.com` succeeds
- [ ] GitHub CLI logged in
- [ ] GitHub 2FA enabled

Runtimes (install what you use)
- [ ] Node.js LTS via nvm-windows + Corepack enabled
- [ ] Python 3.12 + uv/pipx
- [ ] Eclipse Temurin JDK (25 or 21 LTS), JAVA_HOME set
- [ ] .NET SDK (LTS)
- [ ] Go / Rust / PHP if needed
- [ ] Android Studio + SDK + ANDROID_HOME set (mobile only)

Tools
- [ ] VS Code + core extensions
- [ ] Docker Desktop with WSL2 backend (or Rancher/Podman)
- [ ] kubectl / helm
- [ ] AWS/Azure/GCP CLI as needed
- [ ] Postman or Bruno
- [ ] DBeaver
- [ ] jq, yq, ripgrep, fzf, fd

Security
- [ ] Password manager installed & signed in
- [ ] gitleaks pre-commit hook set up
- [ ] `.env`, secrets never in Git
- [ ] Antivirus (Windows Defender is fine)

Reproducibility
- [ ] `winget export` saved
- [ ] Dotfiles repo pushed
- [ ] `.ssh` keys backed up encrypted
- [ ] First project cloned, built, and runs

---

## 🏁 Final Engineering Principle

> **A senior engineer's laptop is measured by how quickly it can be rebuilt from scratch — not by how much is installed on it.**

The target flow:

```
Fresh Windows 11 Pro
        ↓
Windows Update + Dev features
        ↓
WinGet baseline
        ↓
WSL2 Ubuntu
        ↓
Git + SSH + GitHub
        ↓
Language toolchains (only what you use)
        ↓
Editors / IDEs
        ↓
Docker + K8s + Cloud CLIs
        ↓
Project clone → build → test → run
        ↓
Backup + dotfiles repo
```

Everything else waits until a project needs it.
