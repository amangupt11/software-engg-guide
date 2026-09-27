# 🍎 macOS (Apple Silicon) — Fresh MacBook Developer Setup

> A senior software engineer's setup guide for a **brand-new MacBook** running **macOS 26 Tahoe** (or newer).
> Every tool is installed from its **official vendor source or Homebrew's official tap**. No mirrors, no random `.dmg`s from search results.

**Author perspective:** 15+ years of engineering. Installs are minimal, reproducible, and driven by real project needs (full-stack web, backend APIs, iOS/macOS apps, containers, cloud, databases).

**macOS version reference (Sep 2026):**
- **macOS 26 Tahoe** — current stable, last release supporting Intel Macs.
- **macOS 27 Golden Gate** — Apple Silicon only, launching ~Sep 2026.
- Confirm your version: **  → About This Mac**.

**Golden rule:** *If a tool is not needed by a current project, don't install it. Homebrew makes it cheap to add later.*

---

## 📚 Table of Contents

1. [Hardware Baseline](#1-hardware-baseline)
2. [First Boot — macOS Hygiene](#2-first-boot--macos-hygiene)
3. [Xcode Command Line Tools](#3-xcode-command-line-tools)
4. [Homebrew — The Package Manager](#4-homebrew--the-package-manager)
5. [Rosetta 2 (Apple Silicon only)](#5-rosetta-2-apple-silicon-only)
6. [Terminal & Shell Stack](#6-terminal--shell-stack)
7. [Git & GitHub](#7-git--github)
8. [Editors & IDEs](#8-editors--ides)
9. [Node.js / JavaScript / TypeScript](#9-nodejs--javascript--typescript)
10. [Python](#10-python)
11. [Java / JVM](#11-java--jvm)
12. [.NET](#12-net)
13. [PHP](#13-php)
14. [Go / Rust (optional)](#14-go--rust-optional)
15. [Xcode & Apple Platform Development](#15-xcode--apple-platform-development)
16. [Android Development](#16-android-development)
17. [Databases](#17-databases)
18. [API & HTTP Tools](#18-api--http-tools)
19. [Docker & Kubernetes](#19-docker--kubernetes)
20. [Cloud CLIs](#20-cloud-clis)
21. [Browsers](#21-browsers)
22. [Security & Secrets](#22-security--secrets)
23. [Productivity Apps](#23-productivity-apps)
24. [Verification Script](#24-verification-script)
25. [Backup & Reproducibility](#25-backup--reproducibility)
26. [Final Checklist](#26-final-checklist)

---

## 1. Hardware Baseline

| Component | Minimum | Recommended for pro dev |
|---|---|---|
| Chip | Apple M1 / M2 | **Apple M3/M4/M5 Pro or Max** |
| RAM (unified memory) | 16 GB | **32 GB** (48 GB+ for iOS + Android + containers) |
| Storage | 512 GB | **1 TB** (Xcode + simulators + Docker eats space fast) |
| Display | Built-in | 4K/5K external for long sessions |

> **Apple Silicon vs Intel:** macOS 26 Tahoe is the last version that runs on Intel Macs. New machines are all Apple Silicon (ARM64) — most native tools now ship arm64 binaries. Use **Rosetta 2** (§5) only for the shrinking set of x86-only tools.

Verify your chip:

```bash
uname -m       # arm64 = Apple Silicon, x86_64 = Intel
sysctl -n machdep.cpu.brand_string
```

---

## 2. First Boot — macOS Hygiene

Get the base OS clean and current before anything else.

### 2.1 Install every pending update

```
System Settings → General → Software Update → Update Now
```

Repeat until "Your Mac is up to date." A fresh Mac is often behind on point releases.

### 2.2 Enable FileVault (disk encryption)

```
System Settings → Privacy & Security → FileVault → Turn On
```

Store the recovery key in your Apple Account **and** in a printed vault or password manager. Do this **before** you put source code on the machine.

### 2.3 Enable Firewall

```
System Settings → Network → Firewall → On
```

### 2.4 Sign in to Apple Account & iCloud

Only sync **Documents, Desktop, Keychain, Contacts** to iCloud. Turn off **Photos** unless you actually want work photos synced.

### 2.5 Set up Time Machine

External SSD (≥ 2× your internal disk):

```
System Settings → General → Time Machine → Add Backup Disk
```

An unbacked-up MacBook is a countdown to disaster.

### 2.6 Show hidden files & real paths in Finder

```bash
defaults write com.apple.finder AppleShowAllFiles YES
defaults write com.apple.finder _FXShowPosixPathInTitle YES
killall Finder
```

### 2.7 Fix key-repeat for coders

```bash
defaults write -g ApplePressAndHoldEnabled -bool false     # enable proper key repeat in Vim/JetBrains
defaults write NSGlobalDomain KeyRepeat -int 2
defaults write NSGlobalDomain InitialKeyRepeat -int 15
```

Log out and back in for these to apply.

---

## 3. Xcode Command Line Tools

The prerequisite for Homebrew, Git, and most compilers.

```bash
xcode-select --install
```

Accept the license, wait for the download to finish. Verify:

```bash
xcode-select -p
# /Library/Developer/CommandLineTools
gcc --version
```

Official docs: <https://developer.apple.com/xcode/resources/>

---

## 4. Homebrew — The Package Manager

Homebrew is the de-facto macOS package manager and the correct way to install nearly every tool in this guide.

Official install (do **not** curl-pipe from any other source):

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Homepage & docs: <https://brew.sh>

After install, add Homebrew to your shell PATH (the installer prints the exact command; on Apple Silicon it's under `/opt/homebrew`):

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"

brew --version
brew doctor
```

**Two concepts:**

| | Formulae | Casks |
|---|---|---|
| What | CLI tools & libraries | GUI apps (`.app`) |
| Command | `brew install <name>` | `brew install --cask <name>` |
| Example | `brew install jq` | `brew install --cask visual-studio-code` |

---

## 5. Rosetta 2 (Apple Silicon only)

For the shrinking set of x86-only tools:

```bash
softwareupdate --install-rosetta --agree-to-license
```

You rarely need this in 2026 — Node, Python, JDK, Docker, Postgres, and JetBrains all ship native ARM64 builds. Install only when a specific tool asks for it.

---

## 6. Terminal & Shell Stack

macOS ships **zsh** as the default shell and **Terminal.app** as the default terminal. Both are fine; most engineers upgrade to a nicer terminal.

| Tool | Purpose | Official Source | Install |
|---|---|---|---|
| **iTerm2** | Better terminal | <https://iterm2.com> | `brew install --cask iterm2` |
| **Ghostty** | Fast GPU-accelerated terminal | <https://ghostty.org> | `brew install --cask ghostty` |
| **Warp** | Modern AI-native terminal | <https://www.warp.dev> | `brew install --cask warp` |
| **Oh My Zsh** | zsh framework | <https://ohmyz.sh> | see below |
| **Starship** | Cross-shell prompt | <https://starship.rs> | `brew install starship` |
| **tmux** | Terminal multiplexer | <https://github.com/tmux/tmux> | `brew install tmux` |

Install Oh My Zsh (official one-liner):

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

Install a Nerd Font for terminal glyphs:

```bash
brew install --cask font-jetbrains-mono-nerd-font
brew install --cask font-cascadia-code-nf
```

Nerd Fonts official: <https://www.nerdfonts.com>

Baseline CLI utilities (all official, all via Homebrew):

```bash
brew install \
  coreutils gnu-sed gnu-tar grep findutils \
  bat eza fd fzf ripgrep zoxide \
  jq yq htop tree wget curl httpie \
  git-delta gh
```

---

## 7. Git & GitHub

macOS's Xcode Git is fine; the Homebrew version stays current:

```bash
brew install git git-lfs gh
```

Official sources:
- Git — <https://git-scm.com/download/mac>
- GitHub CLI — <https://cli.github.com>
- Git LFS — <https://git-lfs.com>

### 7.1 Identity & sane defaults

```bash
git config --global user.name  "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
git config --global core.autocrlf input           # macOS/Linux
git config --global core.editor "code --wait"     # or "vim" / "nvim"
git config --global pull.rebase true
git config --global fetch.prune true
git config --global rerere.enabled true
```

### 7.2 SSH key (Ed25519)

```bash
ssh-keygen -t ed25519 -C "you@example.com"
```

Add the key to macOS Keychain (`~/.ssh/config`):

```
Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
```

Load and copy:

```bash
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
pbcopy < ~/.ssh/id_ed25519.pub
```

Add at <https://github.com/settings/keys> → **New SSH key**. Test:

```bash
ssh -T git@github.com
```

### 7.3 GitHub CLI login

```bash
gh auth login
gh auth status
```

Enable 2FA on GitHub before pushing company code: <https://github.com/settings/security>.

### 7.4 Sign commits with SSH (optional but recommended)

```bash
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true
```

Then upload the same key as a **signing key** on GitHub.

---

## 8. Editors & IDEs

Install only what you actually use.

| Tool | Official Source | Homebrew Cask |
|---|---|---|
| **Visual Studio Code** | <https://code.visualstudio.com> | `visual-studio-code` |
| **Xcode** | Mac App Store | (App Store) |
| **JetBrains Toolbox** *(manages all JetBrains IDEs)* | <https://www.jetbrains.com/toolbox-app/> | `jetbrains-toolbox` |
| **IntelliJ IDEA** | <https://www.jetbrains.com/idea/download/> | `intellij-idea` / `intellij-idea-ce` |
| **PyCharm** | <https://www.jetbrains.com/pycharm/download/> | `pycharm` / `pycharm-ce` |
| **PhpStorm** | <https://www.jetbrains.com/phpstorm/download/> | `phpstorm` |
| **WebStorm** | <https://www.jetbrains.com/webstorm/download/> | `webstorm` |
| **Android Studio** | <https://developer.android.com/studio> | `android-studio` |
| **Zed** | <https://zed.dev> | `zed` |
| **Neovim** | <https://neovim.io> | `neovim` (formula) |

VS Code baseline extensions:

```bash
code --install-extension dbaeumer.vscode-eslint
code --install-extension esbenp.prettier-vscode
code --install-extension editorconfig.editorconfig
code --install-extension eamodio.gitlens
code --install-extension github.vscode-pull-request-github
code --install-extension github.vscode-github-actions
code --install-extension ms-azuretools.vscode-docker
code --install-extension ms-kubernetes-tools.vscode-kubernetes-tools
code --install-extension ms-vscode-remote.remote-ssh
code --install-extension ms-vscode-remote.remote-containers
code --install-extension redhat.vscode-yaml
code --install-extension sonarsource.sonarlint-vscode
code --install-extension mikestead.dotenv
code --install-extension humao.rest-client
```

---

## 9. Node.js / JavaScript / TypeScript

Do **not** `brew install node` and pin yourself to one version. Use a version manager.

### 9.1 nvm (official)

Official: <https://github.com/nvm-sh/nvm>

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash

# Reload shell, then:
nvm install --lts        # currently Node.js 24 LTS
nvm install 22           # keep the previous LTS for legacy projects
nvm alias default lts/*
node -v
npm -v
```

Node.js download page: <https://nodejs.org/en/download>

### 9.2 Alternatives (any one, not all)

| Tool | Why | Install | Official |
|---|---|---|---|
| **fnm** | Fast Node manager written in Rust | `brew install fnm` | <https://github.com/Schniz/fnm> |
| **Volta** | Version pinned per project | `brew install volta` | <https://volta.sh> |
| **mise** | Multi-runtime version manager | `brew install mise` | <https://mise.jdx.dev> |

### 9.3 Enable Corepack (pnpm / Yarn)

```bash
corepack enable
```

Official: <https://nodejs.org/api/corepack.html>

### 9.4 Minimal global installs

```bash
npm install -g pnpm typescript ts-node
```

For everything else, prefer `npx` or the repo's declared package manager.

---

## 10. Python

Never rely on macOS's system Python for development.

Install a proper Python:

```bash
brew install python@3.12
```

Or, for multiple versions and full isolation:

```bash
brew install pyenv
pyenv install 3.12
pyenv global 3.12
```

- Python official — <https://www.python.org/downloads/macos/>
- pyenv — <https://github.com/pyenv/pyenv>

Modern tooling (pick **one** per project):

| Tool | Why | Install | Official |
|---|---|---|---|
| **uv** | Fastest, all-in-one | `brew install uv` | <https://docs.astral.sh/uv/> |
| **Poetry** | Mature dependency + packaging | `brew install poetry` | <https://python-poetry.org/> |
| **Hatch** | PEP-621 native, PyPA-endorsed | `brew install hatch` | <https://hatch.pypa.io/> |

Isolated CLI tools:

```bash
brew install pipx
pipx ensurepath
```

Linters / formatters (per-project):

- **Ruff** — <https://docs.astral.sh/ruff/>
- **pytest** — <https://docs.pytest.org/>

---

## 11. Java / JVM

Use **Eclipse Temurin** (Adoptium) — the neutral, production OpenJDK distribution.

Official: <https://adoptium.net/temurin/releases/>

Current LTS lines (as of Sep 2026):

| Version | Status | Recommended for |
|---|---|---|
| **JDK 25 LTS** | Newest LTS (Sep 2025, security to Sep 2031) | New projects |
| **JDK 21 LTS** | Mature LTS (security to Dec 2029) | Most enterprise codebases today |
| **JDK 17 LTS** | Older LTS (support to Oct 2027) | Legacy Spring Boot 2.x |

Install (pick one — or install several and switch):

```bash
brew tap homebrew/cask-versions       # for older LTS if needed
brew install --cask temurin@25
brew install --cask temurin@21
```

Verify:

```bash
/usr/libexec/java_home -V             # lists all installed JDKs
java -version
javac -version
```

Switch between JDKs cleanly with **jenv** or **SDKMAN!**:

```bash
# jenv
brew install jenv
jenv add /Library/Java/JavaVirtualMachines/temurin-21.jdk/Contents/Home

# OR SDKMAN! (also handles Maven, Gradle, Kotlin, Scala)
curl -s "https://get.sdkman.io" | bash
```

- jenv — <https://www.jenv.be>
- SDKMAN! — <https://sdkman.io>

Build tools — prefer the project's `mvnw` / `gradlew`. For greenfield:

```bash
brew install maven gradle
```

---

## 12. .NET

Official: <https://dotnet.microsoft.com/download>

```bash
brew install --cask dotnet-sdk
```

Verify:

```bash
dotnet --info
dotnet --list-sdks
```

For .NET on Mac, **JetBrains Rider** (§8) or **VS Code + C# Dev Kit** are the primary editors.

---

## 13. PHP

Homebrew keeps PHP current:

```bash
brew install php composer
```

- PHP — <https://www.php.net/downloads.php>
- Composer — <https://getcomposer.org/download/>

For Laravel/Symfony, **PhpStorm** (§8) is worth the license.

Local PHP dev environments:

- **Laravel Herd** — <https://herd.laravel.com> (free, native macOS)
- **Valet** — <https://laravel.com/docs/valet>

---

## 14. Go / Rust (optional)

| Tool | Official Source | Homebrew |
|---|---|---|
| **Go** | <https://go.dev/dl/> | `brew install go` |
| **Rust (rustup)** | <https://www.rust-lang.org/tools/install> | `brew install rustup-init && rustup-init` |

---

## 15. Xcode & Apple Platform Development

Only install if you build for iOS / iPadOS / macOS / watchOS / tvOS / visionOS.

### 15.1 Install Xcode

**From the Mac App Store** (only official channel): <https://apps.apple.com/app/xcode/id497799835>

After install:

```bash
sudo xcodebuild -license accept
sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
xcodebuild -version
```

### 15.2 Simulators

Open Xcode → **Settings → Platforms** → download the iOS / iPadOS / watchOS / visionOS simulators you need. They're huge (5–15 GB each) — download on-demand.

### 15.3 Command-line helpers

```bash
brew install cocoapods           # if the project still uses CocoaPods
brew install fastlane            # release automation
brew install swiftlint           # linting
brew install swiftformat         # formatting
brew install xcbeautify          # pretty xcodebuild output
```

Official links:
- CocoaPods — <https://cocoapods.org>
- Fastlane — <https://fastlane.tools>
- SwiftLint — <https://github.com/realm/SwiftLint>
- SwiftFormat — <https://github.com/nicklockwood/SwiftFormat>

### 15.4 Apple Developer Program

Enrol at <https://developer.apple.com/programs/> to publish to the App Store, use push notifications, and access some beta APIs.

---

## 16. Android Development

Only install if you build mobile apps for Android.

```bash
brew install --cask android-studio
```

Official: <https://developer.android.com/studio>

Inside Android Studio: **SDK Manager** → install
- Android SDK Platform (latest API, e.g. 35+)
- Android SDK Platform-Tools
- Android SDK Build-Tools
- Android Emulator + at least one system image

Set environment variables in `~/.zshrc`:

```bash
export ANDROID_HOME="$HOME/Library/Android/sdk"
export ANDROID_SDK_ROOT="$ANDROID_HOME"
export PATH="$PATH:$ANDROID_HOME/platform-tools:$ANDROID_HOME/emulator"
```

Verify:

```bash
adb version
```

For **React Native** / **Flutter**, also install JDK 17 or 21 (§11) and Node LTS (§9).

- React Native env — <https://reactnative.dev/docs/environment-setup>
- Flutter — <https://docs.flutter.dev/get-started/install/macos>

---

## 17. Databases

Prefer **Docker containers** (§19) for local databases — disposable, versioned, match production. Install native only when you must.

| DB | Official Source | Homebrew |
|---|---|---|
| **PostgreSQL** | <https://www.postgresql.org/download/macosx/> | `brew install postgresql@17` |
| **MySQL** | <https://dev.mysql.com/downloads/mysql/> | `brew install mysql` |
| **MariaDB** | <https://mariadb.org/download/> | `brew install mariadb` |
| **MongoDB Community** | <https://www.mongodb.com/try/download/community> | `brew tap mongodb/brew && brew install mongodb-community@8.0` |
| **MongoDB Compass** *(GUI)* | <https://www.mongodb.com/products/tools/compass> | `brew install --cask mongodb-compass` |
| **Redis** | <https://redis.io/downloads/> | `brew install redis` |
| **SQLite** | Comes with macOS | — |
| **DBeaver Community** *(cross-DB GUI)* | <https://dbeaver.io/download/> | `brew install --cask dbeaver-community` |
| **Postico** *(Postgres GUI)* | <https://eggerapps.at/postico2/> | `brew install --cask postico` |
| **TablePlus** *(commercial GUI)* | <https://tableplus.com> | `brew install --cask tableplus` |

Start / stop native services:

```bash
brew services start  postgresql@17
brew services stop   postgresql@17
brew services list
```

---

## 18. API & HTTP Tools

| Tool | Official Source | Homebrew Cask |
|---|---|---|
| **Postman** | <https://www.postman.com/downloads/> | `postman` |
| **Insomnia** | <https://insomnia.rest/download> | `insomnia` |
| **Bruno** *(Git-friendly)* | <https://www.usebruno.com/downloads> | `bruno` |
| **curl** | Built-in | — |
| **HTTPie** | <https://httpie.io/cli> | `brew install httpie` |
| **jq / yq** | See §6 | `brew install jq yq` |

---

## 19. Docker & Kubernetes

### 19.1 Docker Desktop

Official: <https://www.docker.com/products/docker-desktop/>

```bash
brew install --cask docker
```

Launch once from `/Applications/Docker.app` to accept the license and let it install the CLI helper. Then:

```bash
docker version
docker compose version
```

**Licensing note:** Docker Desktop requires a paid subscription for larger companies. Check <https://www.docker.com/pricing/> against your employer's headcount and revenue.

Free alternatives (all native ARM64):

| Tool | Official | Homebrew |
|---|---|---|
| **OrbStack** *(fastest on Apple Silicon)* | <https://orbstack.dev> | `brew install --cask orbstack` |
| **Rancher Desktop** | <https://rancherdesktop.io> | `brew install --cask rancher` |
| **Podman Desktop** | <https://podman-desktop.io> | `brew install --cask podman-desktop` |
| **colima** *(CLI-only)* | <https://github.com/abiosoft/colima> | `brew install colima` |

**OrbStack** is the most senior-engineer-friendly on Apple Silicon: 3× faster than Docker Desktop, built for macOS, one-click Kubernetes.

### 19.2 Kubernetes CLI tools

| Tool | Official Source | Homebrew |
|---|---|---|
| **kubectl** | <https://kubernetes.io/docs/tasks/tools/> | `brew install kubectl` |
| **Helm** | <https://helm.sh/docs/intro/install/> | `brew install helm` |
| **kind** | <https://kind.sigs.k8s.io/> | `brew install kind` |
| **Minikube** | <https://minikube.sigs.k8s.io/> | `brew install minikube` |
| **k9s** *(TUI)* | <https://k9scli.io/> | `brew install k9s` |
| **kubectx / kubens** | <https://github.com/ahmetb/kubectx> | `brew install kubectx` |

---

## 20. Cloud CLIs

Install only the clouds you actually use.

| Cloud | Official Source | Homebrew |
|---|---|---|
| **AWS CLI v2** | <https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html> | `brew install awscli` |
| **Azure CLI** | <https://learn.microsoft.com/cli/azure/install-azure-cli-macos> | `brew install azure-cli` |
| **Google Cloud CLI** | <https://cloud.google.com/sdk/docs/install> | `brew install --cask google-cloud-sdk` |
| **Cloudflare Wrangler** | <https://developers.cloudflare.com/workers/wrangler/install-and-update/> | `npm install -g wrangler` |
| **Terraform** | <https://developer.hashicorp.com/terraform/install> | `brew tap hashicorp/tap && brew install hashicorp/tap/terraform` |
| **OpenTofu** | <https://opentofu.org/docs/intro/install/> | `brew install opentofu` |
| **Ansible** | <https://docs.ansible.com/> | `brew install ansible` |

---

## 21. Browsers

| Browser | Official Source | Homebrew Cask |
|---|---|---|
| **Google Chrome** | <https://www.google.com/chrome/> | `google-chrome` |
| **Firefox Developer Edition** | <https://www.mozilla.org/firefox/developer/> | `firefox@developer-edition` |
| **Safari** | Built-in | — |
| **Microsoft Edge** | <https://www.microsoft.com/edge/download> | `microsoft-edge` |
| **Arc** *(optional, developer-friendly)* | <https://arc.net> | `arc` |

Recommended: **Chrome or Arc** primary, **Safari** for Apple-platform testing, **Firefox Dev Edition** for cross-browser QA.

---

## 22. Security & Secrets

| Tool | Purpose | Official Source | Homebrew |
|---|---|---|---|
| **1Password / Bitwarden** | Password manager | <https://1password.com> / <https://bitwarden.com> | `--cask 1password` / `--cask bitwarden` |
| **GnuPG** | Signing / encryption | <https://gnupg.org/download/> | `brew install gnupg` |
| **OpenSSL** *(modern)* | TLS / certs | <https://www.openssl.org> | `brew install openssl@3` |
| **age** | Modern file encryption | <https://github.com/FiloSottile/age> | `brew install age` |
| **gitleaks** | Secret scanning | <https://github.com/gitleaks/gitleaks> | `brew install gitleaks` |
| **mkcert** | Local HTTPS certs | <https://github.com/FiloSottile/mkcert> | `brew install mkcert` |

**Rules:**
- Never commit `.env` or private keys.
- SSH keys with a passphrase, stored in **macOS Keychain** (§7.2).
- **FileVault on** (§2.2) — a stolen MacBook must be a paperweight.
- **Find My Mac** on: **System Settings → your name → iCloud → Find My Mac → On**.

---

## 23. Productivity Apps

| Tool | Purpose | Official Source | Homebrew Cask |
|---|---|---|---|
| **Raycast** | Spotlight replacement + launcher + snippets | <https://www.raycast.com> | `raycast` |
| **Rectangle** | Window snapping (free) | <https://rectangleapp.com> | `rectangle` |
| **AltTab** | Windows-style Alt-Tab | <https://alt-tab-macos.netlify.app> | `alt-tab` |
| **Maccy** | Clipboard history | <https://maccy.app> | `maccy` |
| **CleanShot X** | Screenshots & GIFs | <https://cleanshot.com> | `cleanshot` |
| **The Unarchiver** | Archives | <https://theunarchiver.com> | `the-unarchiver` |
| **Obsidian** | Local Markdown notes | <https://obsidian.md/download> | `obsidian` |
| **Notion** | Team docs | <https://www.notion.so/desktop> | `notion` |
| **Slack / Teams / Zoom** | Team comms | <https://slack.com> / <https://teams.microsoft.com> / <https://zoom.us> | `slack` / `microsoft-teams` / `zoom` |
| **Cyberduck / Transmit** | SFTP client | <https://cyberduck.io> / <https://panic.com/transmit/> | `cyberduck` / `transmit` |
| **Stats** | Menu-bar system monitor | <https://github.com/exelban/stats> | `stats` |

---

## 24. Verification Script

Save as `verify-dev-env.sh` and run after each install session:

```bash
#!/usr/bin/env bash
# ---- verify-dev-env.sh ----

check() {
  local name="$1"; shift
  printf "==> %-14s " "$name"
  if command -v "$1" >/dev/null 2>&1; then
    "$@" 2>&1 | head -n 1
  else
    printf "NOT INSTALLED\n"
  fi
}

check "Git"        git --version
check "GitHub CLI" gh --version
check "Homebrew"   brew --version
check "Node"       node --version
check "npm"        npm --version
check "Python"     python3 --version
check "pip"        python3 -m pip --version
check "Java"       java -version
check "dotnet"     dotnet --version
check "PHP"        php --version
check "Docker"     docker --version
check "kubectl"    kubectl version --client
check "Terraform"  terraform -version
check "AWS CLI"    aws --version
check "Azure CLI"  az --version
check "gcloud"     gcloud --version
check "Xcode"      xcodebuild -version
check "adb"        adb version
```

```bash
chmod +x verify-dev-env.sh
./verify-dev-env.sh
```

---

## 25. Backup & Reproducibility

Rebuilding a Mac should be an afternoon, not a weekend.

### 25.1 Export the Homebrew inventory

Homebrew's official reproducibility tool is a **Brewfile**:

```bash
brew bundle dump --file="$HOME/Brewfile" --force --describe
```

Docs: <https://github.com/Homebrew/homebrew-bundle>

Restore on any Mac with:

```bash
brew bundle --file="$HOME/Brewfile"
```

Commit the `Brewfile` to your **dotfiles** repo.

### 25.2 Files to back up

```
~/.ssh/
~/.gitconfig
~/.gitignore_global
~/.zshrc
~/.zprofile
~/.config/       (Neovim, Starship, etc.)
~/Library/Application Support/Code/User/settings.json
~/Library/Application Support/Code/User/keybindings.json
```

Keep them in a **private GitHub dotfiles repo** (no secrets) plus an encrypted backup for `.ssh` and `.gnupg`.

### 25.3 System backup

- **Time Machine** (§2.5) → external SSD, hourly.
- **iCloud Drive** for `Documents`/`Desktop` if the free tier is enough.
- **SSH & GPG private keys** → 1Password / Bitwarden secure attachments.

### 25.4 Migration Assistant

When you buy a new Mac, Migration Assistant (in `/Applications/Utilities`) can pull the whole environment from Time Machine or another Mac in one pass. Combined with the Brewfile above, you get repeatable, verifiable rebuilds.

---

## 26. Final Checklist

Foundation
- [ ] macOS fully updated (26 Tahoe or newer)
- [ ] FileVault on, recovery key saved
- [ ] Firewall on
- [ ] Find My Mac on
- [ ] Time Machine configured with external SSD
- [ ] Apple Account signed in
- [ ] Xcode Command Line Tools installed
- [ ] Homebrew installed, `brew doctor` clean
- [ ] Rosetta 2 installed (only if needed)

Shell
- [ ] iTerm2 / Ghostty / Warp installed
- [ ] Nerd Font installed and configured
- [ ] Oh My Zsh or Starship prompt
- [ ] CLI baseline: jq, yq, ripgrep, fd, fzf, bat, eza

Version control
- [ ] Git configured (identity, autocrlf input, default branch main)
- [ ] SSH Ed25519 key generated + stored in Keychain
- [ ] Key added to GitHub, `ssh -T git@github.com` succeeds
- [ ] GitHub CLI logged in
- [ ] GitHub 2FA enabled
- [ ] Commit signing configured (optional)

Runtimes (install what you use)
- [ ] Node.js LTS via nvm/fnm/mise + Corepack enabled
- [ ] Python 3.12 + uv/pipx
- [ ] Eclipse Temurin JDK (25 or 21 LTS), `java_home -V` shows it
- [ ] .NET SDK (LTS)
- [ ] Go / Rust / PHP if needed
- [ ] Xcode installed (Apple platform work)
- [ ] Android Studio + SDK + ANDROID_HOME set (mobile only)

Tools
- [ ] VS Code + core extensions
- [ ] Docker Desktop **or** OrbStack / Rancher / Podman
- [ ] kubectl / helm / k9s
- [ ] AWS/Azure/GCP CLI as needed
- [ ] Postman or Bruno
- [ ] DBeaver / Postico / TablePlus

Security
- [ ] Password manager installed & signed in
- [ ] gitleaks pre-commit hook set up
- [ ] `.env`, secrets never in Git
- [ ] mkcert set up for local HTTPS

Reproducibility
- [ ] `Brewfile` generated and committed
- [ ] Dotfiles repo pushed
- [ ] `~/.ssh` keys backed up encrypted
- [ ] First project cloned, built, and runs

---

## 🏁 Final Engineering Principle

> **A senior engineer's Mac is measured by how quickly it can be rebuilt from scratch — not by how much is installed on it.**

Target flow:

```
Fresh MacBook (macOS 26 Tahoe / 27 Golden Gate)
        ↓
Software Update + FileVault + Time Machine
        ↓
Xcode Command Line Tools
        ↓
Homebrew
        ↓
Git + SSH + GitHub (keychain-integrated)
        ↓
Language toolchains (only what you use)
        ↓
Editors / IDEs
        ↓
OrbStack / Docker + K8s + Cloud CLIs
        ↓
Project clone → build → test → run
        ↓
Brewfile + dotfiles committed
```

Everything else waits until a project needs it.
