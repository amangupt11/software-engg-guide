# 🌿 Git, GitHub, GitLab & Bitbucket — Professional Setup & Workflow Guide

> A senior software engineer's reference for installing, configuring, and working across the three major Git hosts, from a single laptop, safely and reproducibly.
>
> **Every download link is official.** No mirrors, no repackaged installers.

**Author perspective:** 15+ years of engineering. Focus on real-world workflows: multi-host identities, SSH per-host keys, PR/MR/Pipeline workflows, commit signing, secret scanning, and being able to walk into any team using any of the three platforms.

---

## 📚 Table of Contents

**Part A — Git Core**
1. [Install Git — Official Sources](#1-install-git--official-sources)
2. [First-Time Git Configuration](#2-first-time-git-configuration)
3. [SSH Keys (Ed25519)](#3-ssh-keys-ed25519)
4. [Commit Signing (SSH / GPG)](#4-commit-signing-ssh--gpg)
5. [Useful Global Git Config](#5-useful-global-git-config)
6. [Global `.gitignore` & `.gitattributes`](#6-global-gitignore--gitattributes)
7. [Git Hooks & Pre-commit](#7-git-hooks--pre-commit)

**Part B — Platform Comparison**
8. [GitHub vs GitLab vs Bitbucket — At a Glance](#8-github-vs-gitlab-vs-bitbucket--at-a-glance)
9. [Multi-Host SSH Configuration](#9-multi-host-ssh-configuration)

**Part C — GitHub**
10. [GitHub Account Setup](#10-github-account-setup)
11. [GitHub CLI (`gh`) — Official](#11-github-cli-gh--official)
12. [GitHub Pull Request Workflow](#12-github-pull-request-workflow)
13. [GitHub Actions — CI/CD Basics](#13-github-actions--cicd-basics)
14. [GitHub Security](#14-github-security)

**Part D — GitLab**
15. [GitLab Account Setup](#15-gitlab-account-setup)
16. [GitLab CLI (`glab`) — Official](#16-gitlab-cli-glab--official)
17. [GitLab Merge Request Workflow](#17-gitlab-merge-request-workflow)
18. [GitLab CI/CD Basics](#18-gitlab-cicd-basics)
19. [GitLab Security](#19-gitlab-security)

**Part E — Bitbucket**
20. [Bitbucket Account Setup](#20-bitbucket-account-setup)
21. [Bitbucket CLI Reality Check](#21-bitbucket-cli-reality-check)
22. [Bitbucket Pull Request Workflow](#22-bitbucket-pull-request-workflow)
23. [Bitbucket Pipelines — CI/CD Basics](#23-bitbucket-pipelines--cicd-basics)
24. [Bitbucket Security](#24-bitbucket-security)

**Part F — Daily Workflows**
25. [Feature Branch Workflow](#25-feature-branch-workflow)
26. [Common Commands Cheat Sheet](#26-common-commands-cheat-sheet)
27. [Rescue Recipes (Undo, Fix, Recover)](#27-rescue-recipes-undo-fix-recover)

**Part G — Reference**
28. [Commit Message Conventions](#28-commit-message-conventions)
29. [Final Checklist](#29-final-checklist)

---

# PART A — GIT CORE

## 1. Install Git — Official Sources

Use the official installer for your OS. Verify signatures where available.

| Platform | Official Source | Command |
|---|---|---|
| **Windows** | <https://git-scm.com/download/win> | `winget install --id Git.Git` |
| **macOS** | <https://git-scm.com/download/mac> | `brew install git` |
| **Ubuntu / Debian** | <https://git-scm.com/download/linux> | `sudo apt install git` |
| **Fedora / RHEL** | <https://git-scm.com/download/linux> | `sudo dnf install git` |
| **Arch** | <https://git-scm.com/download/linux> | `sudo pacman -S git` |

Also install:

- **Git LFS** — large-file storage — <https://git-lfs.com>
- **Git Credential Manager** — bundled with Git for Windows; separate on macOS/Linux — <https://github.com/git-ecosystem/git-credential-manager>

Verify:

```bash
git --version           # 2.50+ recommended
git lfs --version
```

---

## 2. First-Time Git Configuration

Three levels of Git config exist — `--system`, `--global`, `--local`. **`--global`** lives in `~/.gitconfig` and is what you configure on a new machine.

```bash
git config --global user.name  "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
git config --global core.editor "code --wait"            # or "vim" / "nvim"
git config --global pull.rebase true
git config --global fetch.prune true
git config --global rerere.enabled true
git config --global push.autoSetupRemote true
git config --global push.default simple
git config --global diff.colorMoved zebra
```

Line endings — **critical** for cross-platform teams:

```bash
# Windows
git config --global core.autocrlf true

# macOS / Linux
git config --global core.autocrlf input
```

Verify:

```bash
git config --global --list
```

### 2.1 Per-project identity override

If you contribute to work + personal repos with **different email addresses**, use `includeIf`:

```
# ~/.gitconfig
[user]
    name  = Your Name
    email = personal@example.com

[includeIf "gitdir:~/work/"]
    path = ~/.gitconfig-work

# ~/.gitconfig-work
[user]
    email = you@company.com
    signingkey = ~/.ssh/id_ed25519_work.pub
```

Clone company repos under `~/work/` and Git auto-switches your identity.

---

## 3. SSH Keys (Ed25519)

**Ed25519** is the current standard — small, fast, secure. Do **not** use RSA-1024 or DSA.

### 3.1 Generate a key per host (recommended)

Best practice: one key per host so revoking access on one platform doesn't affect the others.

```bash
ssh-keygen -t ed25519 -C "you@example.com — github"      -f ~/.ssh/id_ed25519_github
ssh-keygen -t ed25519 -C "you@example.com — gitlab"      -f ~/.ssh/id_ed25519_gitlab
ssh-keygen -t ed25519 -C "you@example.com — bitbucket"   -f ~/.ssh/id_ed25519_bitbucket
```

Set correct permissions:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519_*
chmod 644 ~/.ssh/id_ed25519_*.pub
```

### 3.2 Add each public key to the correct host

- **GitHub** — <https://github.com/settings/keys> → *New SSH key*
- **GitLab** — <https://gitlab.com/-/profile/keys> → *Add new key*
- **Bitbucket** — <https://bitbucket.org/account/settings/ssh-keys/> → *Add key*

Copy the public key:

```bash
# macOS
pbcopy < ~/.ssh/id_ed25519_github.pub

# Windows PowerShell
Get-Content ~\.ssh\id_ed25519_github.pub | Set-Clipboard

# Linux (Wayland / X11)
wl-copy < ~/.ssh/id_ed25519_github.pub    # or: xclip -sel clip < ...
```

### 3.3 Load into the SSH agent

**Linux / macOS:**

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519_github
ssh-add ~/.ssh/id_ed25519_gitlab
ssh-add ~/.ssh/id_ed25519_bitbucket
```

**macOS (Keychain integration)** — see §9 for the `~/.ssh/config` entry.

**Windows PowerShell:**

```powershell
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
ssh-add $env:USERPROFILE\.ssh\id_ed25519_github
```

### 3.4 Test each host

```bash
ssh -T git@github.com          # → "Hi <username>! You've successfully authenticated..."
ssh -T git@gitlab.com          # → "Welcome to GitLab, @<username>!"
ssh -T git@bitbucket.org       # → "authenticated via ssh key..."
```

If a test fails, force the specific key:

```bash
ssh -T -i ~/.ssh/id_ed25519_github git@github.com
```

Official SSH docs per host:
- GitHub — <https://docs.github.com/authentication/connecting-to-github-with-ssh>
- GitLab — <https://docs.gitlab.com/user/ssh.html>
- Bitbucket Cloud — <https://support.atlassian.com/bitbucket-cloud/docs/set-up-personal-ssh-keys-on-macos/>

---

## 4. Commit Signing (SSH / GPG)

Signed commits prove *you* authored them. All three hosts show a "Verified" badge next to signed commits.

### 4.1 SSH commit signing (simplest — same key you already use)

```bash
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519_github.pub
git config --global commit.gpgsign true
git config --global tag.gpgsign true
```

Then upload the **same public key** as a **signing key** on each host:

- GitHub — <https://github.com/settings/keys> → *New SSH key* → **Key type: Signing Key**
- GitLab — <https://gitlab.com/-/profile/keys> (accepts SSH signing keys since 15.7)
- Bitbucket — Bitbucket Cloud verifies **GPG** signatures, not SSH. See §4.2 for GPG if signing is mandatory there.

### 4.2 GPG commit signing (works everywhere including Bitbucket)

Install GnuPG:

- macOS — `brew install gnupg`
- Windows — `winget install --id GnuPG.Gpg4win` — <https://gnupg.org/download/>
- Linux — `sudo apt install gnupg`

Generate a key:

```bash
gpg --full-generate-key
# Choose: (1) RSA and RSA
#         4096 bits
#         Expire in 2y
#         Real name + same email as your Git config
```

Get the key ID:

```bash
gpg --list-secret-keys --keyid-format=long
# sec   rsa4096/ABC123DEF4567890 2026-09-27
```

Configure Git:

```bash
git config --global user.signingkey ABC123DEF4567890
git config --global commit.gpgsign true
git config --global gpg.format openpgp
```

Export public key and upload:

```bash
gpg --armor --export ABC123DEF4567890
```

Upload to:
- GitHub — <https://github.com/settings/gpg/new>
- GitLab — <https://gitlab.com/-/profile/gpg_keys>
- Bitbucket — <https://bitbucket.org/account/settings/gpg-keys/>

Verify a signed commit locally:

```bash
git log --show-signature -1
```

---

## 5. Useful Global Git Config

Practical aliases and defaults that a senior engineer sets once and forgets:

```bash
# Aliases
git config --global alias.s   "status -sb"
git config --global alias.co  "checkout"
git config --global alias.cob "checkout -b"
git config --global alias.br  "branch"
git config --global alias.ci  "commit"
git config --global alias.cm  "commit -m"
git config --global alias.ca  "commit --amend --no-edit"
git config --global alias.last "log -1 HEAD --stat"
git config --global alias.lg  "log --oneline --graph --decorate --all -30"
git config --global alias.undo "reset --soft HEAD~1"
git config --global alias.unstage "reset HEAD --"
git config --global alias.wip "commit -am 'wip'"
git config --global alias.aliases "config --get-regexp ^alias\\."

# Better diff / merge
git config --global merge.conflictstyle zdiff3
git config --global diff.algorithm histogram
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'

# Safety
git config --global transfer.fsckObjects true
git config --global fetch.fsckObjects true
git config --global receive.fsckObjects true

# Faster operations on large repos
git config --global core.preloadindex true
git config --global core.fscache true          # Windows
git config --global gc.auto 256
```

Prefer **`git-delta`** for gorgeous diffs — <https://github.com/dandavison/delta>:

```bash
brew install git-delta          # macOS/Linux
winget install --id dandavison.delta

git config --global core.pager "delta"
git config --global delta.side-by-side true
git config --global delta.line-numbers true
git config --global delta.syntax-theme "GitHub"
```

---

## 6. Global `.gitignore` & `.gitattributes`

### 6.1 Global `.gitignore`

For OS/IDE noise that never belongs in any repo:

```bash
# ~/.gitignore_global
.DS_Store
Thumbs.db
Desktop.ini
$RECYCLE.BIN/

.idea/
.vscode/
*.iml
*.swp
*.swo
*~

.env
.env.local
*.pem
*.key
```

Wire it up:

```bash
git config --global core.excludesfile ~/.gitignore_global
```

Community templates: <https://github.com/github/gitignore>

### 6.2 `.gitattributes` (per repo)

Prevents CRLF chaos in cross-platform teams:

```
# .gitattributes
* text=auto eol=lf

# Force LF for shell scripts and Makefiles
*.sh   text eol=lf
Makefile text eol=lf

# Force CRLF for Windows batch
*.bat  text eol=crlf
*.cmd  text eol=crlf

# Binary
*.png  binary
*.jpg  binary
*.pdf  binary
*.zip  binary
```

---

## 7. Git Hooks & Pre-commit

**`pre-commit`** (<https://pre-commit.com>) is the community standard framework — YAML-configured, language-agnostic, runs locally and in CI.

Install:

```bash
brew install pre-commit                # macOS
pip install pre-commit                 # cross-platform
winget install --id python-poetry.pre-commit    # Windows (or pipx)
```

Add `.pre-commit-config.yaml` to your repo:

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
      - id: check-merge-conflict

  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.4
    hooks:
      - id: gitleaks

  - repo: https://github.com/pre-commit/mirrors-prettier
    rev: v4.0.0-alpha.8
    hooks:
      - id: prettier
```

Enable:

```bash
pre-commit install
pre-commit run --all-files
```

Every future `git commit` runs the hooks. This catches secrets, whitespace, syntax, and formatting before they reach the remote.

---

# PART B — PLATFORM COMPARISON

## 8. GitHub vs GitLab vs Bitbucket — At a Glance

| Feature | GitHub | GitLab | Bitbucket Cloud |
|---|---|---|---|
| **Owner** | Microsoft | GitLab Inc. | Atlassian |
| **Free private repos** | Unlimited (Free tier) | Unlimited (Free tier) | Unlimited (Free tier, max 5 users) |
| **Native CI/CD** | GitHub Actions | GitLab CI/CD | Bitbucket Pipelines |
| **PR / MR nomenclature** | Pull Request (PR) | Merge Request (MR) | Pull Request (PR) |
| **Issue tracking** | GitHub Issues, Projects (v2) | GitLab Issues, Epics, Iterations | Jira integration (native) |
| **Container registry** | GitHub Container Registry (ghcr.io) | GitLab Container Registry | (via external services) |
| **Package registry** | GitHub Packages (npm, Maven, NuGet, RubyGems, Docker) | GitLab Package Registry (npm, Maven, PyPI, Composer, Conan, etc.) | Limited |
| **Official CLI** | ✅ `gh` (first-party, MIT) | ✅ `glab` (first-party, MIT) | ❌ **No first-party CLI** — see §21 |
| **DevSecOps built-in** | CodeQL, Dependabot, secret scanning | SAST, DAST, dependency scanning, container scanning | Snyk integration, Atlassian security add-ons |
| **Self-hosted option** | GitHub Enterprise Server (paid) | GitLab Self-Managed (free CE + paid EE) | Bitbucket Data Center (paid) |
| **Best fit for** | Open source, large ecosystem, GitHub Actions users | End-to-end DevSecOps in one product | Teams already on Atlassian (Jira/Confluence) |

**Key strategic notes:**

- **GitHub** dominates open source and has the strongest AI/dev tooling ecosystem (Copilot, Codespaces, Actions marketplace).
- **GitLab** is the most complete single-vendor DevSecOps platform out of the box.
- **Bitbucket** is the natural choice for shops already invested in Jira/Confluence. **Atlassian retired Bitbucket Server** on 15 February 2024; only **Bitbucket Cloud** and **Bitbucket Data Center** remain — <https://www.atlassian.com/migration/assess/journey-to-cloud>.

---

## 9. Multi-Host SSH Configuration

Working with all three hosts at once is normal in consulting or contract work. Use `~/.ssh/config` to keep identities separate.

Create `~/.ssh/config`:

```
# ~/.ssh/config

# ------------------------------------------------------------
# Global defaults
# ------------------------------------------------------------
Host *
    AddKeysToAgent yes
    ServerAliveInterval 60
    ServerAliveCountMax 10
    HashKnownHosts yes
    # macOS only — stores passphrases in Keychain
    UseKeychain yes

# ------------------------------------------------------------
# GitHub — personal
# ------------------------------------------------------------
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_github
    IdentitiesOnly yes

# ------------------------------------------------------------
# GitHub — work (separate account, same host)
# ------------------------------------------------------------
Host github.com-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_github_work
    IdentitiesOnly yes

# ------------------------------------------------------------
# GitLab.com
# ------------------------------------------------------------
Host gitlab.com
    HostName gitlab.com
    User git
    IdentityFile ~/.ssh/id_ed25519_gitlab
    IdentitiesOnly yes

# ------------------------------------------------------------
# Self-managed GitLab
# ------------------------------------------------------------
Host gitlab.company.internal
    HostName gitlab.company.internal
    User git
    IdentityFile ~/.ssh/id_ed25519_gitlab_work
    IdentitiesOnly yes
    Port 22

# ------------------------------------------------------------
# Bitbucket Cloud
# ------------------------------------------------------------
Host bitbucket.org
    HostName bitbucket.org
    User git
    IdentityFile ~/.ssh/id_ed25519_bitbucket
    IdentitiesOnly yes
```

Permissions:

```bash
chmod 600 ~/.ssh/config
```

### 9.1 Cloning with a specific identity

Standard clone uses the default host match:

```bash
git clone git@github.com:acme/backend.git
```

To clone a **work** GitHub repo with the second identity, use the alias defined in the config:

```bash
git clone git@github.com-work:acme-corp/internal-api.git
```

`IdentitiesOnly yes` is essential — without it, `ssh-agent` will try every loaded key in order, and GitHub may reject the request (`Too many authentication failures`).

---

# PART C — GITHUB

## 10. GitHub Account Setup

1. Create/sign in at <https://github.com>.
2. Verify your email.
3. **Enable Two-Factor Authentication** at <https://github.com/settings/security> — TOTP (authenticator app) or a hardware security key.
4. Add SSH & Signing keys — §3, §4.
5. Set your profile visibility, timezone, and preferred email.
6. Configure **Notification preferences**: <https://github.com/settings/notifications>.

**Personal Access Tokens (PATs)** — needed for HTTPS clones, CI, and API scripts. Prefer **fine-grained tokens** (per-repo, per-scope, expiring) over classic tokens:

- <https://github.com/settings/personal-access-tokens>

---

## 11. GitHub CLI (`gh`) — Official

Official source: <https://cli.github.com>

Install:

| Platform | Command |
|---|---|
| **macOS** | `brew install gh` |
| **Windows** | `winget install --id GitHub.cli` |
| **Ubuntu / Debian** | See the official apt instructions at <https://github.com/cli/cli/blob/trunk/docs/install_linux.md> |
| **Fedora** | `sudo dnf install gh` |
| **Arch** | `sudo pacman -S github-cli` |

### 11.1 Login

```bash
gh auth login
# Choose: GitHub.com → SSH → your key → browser login → done
gh auth status
```

### 11.2 Everyday commands

```bash
# Repos
gh repo create acme/new-service --private --clone
gh repo clone acme/backend
gh repo view --web

# Issues
gh issue list --assignee @me
gh issue create --title "Bug: 500 on /users" --body "Steps to reproduce…"
gh issue view 42 --web

# Pull requests
gh pr create --fill --draft
gh pr list --state open --author @me
gh pr checkout 123               # check out a PR locally
gh pr review 123 --approve
gh pr merge 123 --squash --delete-branch

# Actions
gh run list --limit 10
gh run watch                     # live-tail the current workflow
gh workflow run deploy.yml -f environment=staging

# Releases
gh release create v1.2.0 --notes "..." ./dist/*.tgz
```

Docs: <https://cli.github.com/manual/>

---

## 12. GitHub Pull Request Workflow

Standard trunk-based flow that scales from 2 to 200 engineers:

```bash
# 1. Sync
git checkout main
git pull

# 2. Branch (naming: type/scope-summary)
git checkout -b feat/auth-jwt-refresh

# 3. Work + commit (Conventional Commits — §28)
git add -p
git commit -m "feat(auth): add refresh-token rotation"

# 4. Push
git push -u origin feat/auth-jwt-refresh

# 5. Open PR from terminal
gh pr create --fill --draft
gh pr ready              # when done drafting

# 6. Address review
git commit --fixup=<sha>
git rebase -i --autosquash main
git push --force-with-lease

# 7. Merge (or let GitHub squash)
gh pr merge --squash --delete-branch

# 8. Clean up local
git checkout main
git pull
git branch -d feat/auth-jwt-refresh
```

**Rules:**
- Never `git push --force`; use `--force-with-lease`.
- Never commit directly to `main` — enforce with branch protection: <https://docs.github.com/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches>.
- Every PR requires: passing CI, ≥1 review, up-to-date branch.

---

## 13. GitHub Actions — CI/CD Basics

Docs: <https://docs.github.com/actions>

Minimal Node.js CI at `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  test:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '24'
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
      - run: npm test
      - run: npm run build
```

Secrets live in **Settings → Secrets and variables → Actions**. Reference as `${{ secrets.NAME }}`.

Marketplace: <https://github.com/marketplace?type=actions>

---

## 14. GitHub Security

Turn these on for every repository with real code:

- **Dependabot alerts + version updates** — <https://docs.github.com/code-security/dependabot>
- **Secret scanning** (free on public repos, part of GHAS on private) — <https://docs.github.com/code-security/secret-scanning>
- **Code scanning with CodeQL** — <https://docs.github.com/code-security/code-scanning>
- **Branch protection rules** — require PRs, reviews, status checks, linear history
- **Required signed commits** — Settings → Branches → Require signed commits
- **CODEOWNERS file** at `.github/CODEOWNERS` — auto-request reviews from the right people

Example `.github/dependabot.yml`:

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10

  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "monthly"
```

---

# PART D — GITLAB

## 15. GitLab Account Setup

1. Sign up at <https://gitlab.com>. Self-managed instances live at your company's URL (e.g. `gitlab.company.internal`).
2. Verify your email.
3. **Enable 2FA** — <https://gitlab.com/-/profile/two_factor_auth> — TOTP or WebAuthn.
4. Add SSH & signing keys — §3, §4.
5. Configure notification preferences: <https://gitlab.com/-/profile/notifications>.

**Personal Access Tokens** (for CI, API, Container Registry):

- <https://gitlab.com/-/user_settings/personal_access_tokens>

Scopes to know: `api`, `read_api`, `read_repository`, `write_repository`, `read_registry`, `write_registry`.

**Project / Group / Deploy tokens** are separate mechanisms; use them for CI or read-only automation instead of your personal PAT.

---

## 16. GitLab CLI (`glab`) — Official

Official source: <https://gitlab.com/gitlab-org/cli>

`glab` is GitLab's first-party CLI (transferred from community maintainer to GitLab Inc. in 2022) and is inspired by GitHub's `gh`.

### 16.1 Install

| Platform | Command |
|---|---|
| **macOS** | `brew install glab` |
| **Linux (Homebrew supported)** | `brew install glab` |
| **Windows** | `winget install --id gitlab.gitlab-cli` — or via WSL Homebrew |
| **Snap** | `sudo snap install glab` — publisher: GitLab, Inc. |
| **Binary** | Downloads at <https://gitlab.com/gitlab-org/cli/-/releases> |

### 16.2 Login

```bash
glab auth login
# Choose: GitLab.com (or self-managed) → browser OAuth → done
glab auth status
```

For self-managed GitLab:

```bash
glab auth login --hostname gitlab.company.internal
```

`glab` supports multiple hostnames simultaneously and auto-detects the right one from your `git remote`.

### 16.3 Everyday commands

```bash
# Repos
glab repo create acme/new-service --private
glab repo clone acme/backend
glab repo view --web

# Issues
glab issue list --assignee=@me
glab issue create --title "Bug" --description "..."
glab issue view 42 --web

# Merge Requests
glab mr create --fill --draft
glab mr list --reviewer=@me
glab mr checkout 123
glab mr approve 123
glab mr merge 123 --squash --remove-source-branch

# CI/CD
glab ci status
glab ci view                     # opens pipeline in browser
glab ci trace                    # tails the running job
glab ci lint                     # validate .gitlab-ci.yml locally

# Releases
glab release create v1.2.0 --notes "..."
```

Docs: <https://gitlab.com/gitlab-org/cli/-/blob/main/docs/source/index.md>

---

## 17. GitLab Merge Request Workflow

Same shape as GitHub, different vocabulary:

```bash
git checkout main && git pull
git checkout -b feat/auth-jwt-refresh

git commit -m "feat(auth): add refresh-token rotation"
git push -u origin feat/auth-jwt-refresh

glab mr create --fill --draft
glab mr ready

# after review approval + green pipeline
glab mr merge --squash --remove-source-branch
```

**Approval rules** (Premium/Ultimate) let you require CODEOWNERS, minimum approvers, or scoped approvals per file path. Free tier gets basic "Approvals required" rules.

Protected branches: <https://docs.gitlab.com/user/project/protected_branches.html>

---

## 18. GitLab CI/CD Basics

GitLab reads `.gitlab-ci.yml` at the repo root. Docs: <https://docs.gitlab.com/ci/>.

Minimal Node.js pipeline:

```yaml
stages: [install, test, build, deploy]

image: node:24-alpine

cache:
  key:
    files: [package-lock.json]
  paths: [node_modules/]

install:
  stage: install
  script: npm ci
  artifacts:
    paths: [node_modules/]
    expire_in: 1 hour

test:
  stage: test
  needs: [install]
  script:
    - npm run lint
    - npm test -- --coverage
  coverage: '/Lines\s*:\s*(\d+\.\d+)%/'

build:
  stage: build
  needs: [install, test]
  script: npm run build
  artifacts:
    paths: [dist/]
    expire_in: 1 week

deploy_prod:
  stage: deploy
  needs: [build]
  script: ./scripts/deploy.sh
  environment:
    name: production
    url: https://api.example.com
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
      when: manual
```

Validate locally without pushing:

```bash
glab ci lint
```

CI/CD variables (secrets) — **Settings → CI/CD → Variables**. Mark them **Masked** and **Protected**.

---

## 19. GitLab Security

Out-of-the-box scanning (Free tier has basics; Ultimate has all):

- **SAST** (static app security testing) — <https://docs.gitlab.com/user/application_security/sast/>
- **Dependency scanning** — <https://docs.gitlab.com/user/application_security/dependency_scanning/>
- **Container scanning** — <https://docs.gitlab.com/user/application_security/container_scanning/>
- **Secret detection** — <https://docs.gitlab.com/user/application_security/secret_detection/>
- **DAST** (Ultimate) — <https://docs.gitlab.com/user/application_security/dast/>

Enable in `.gitlab-ci.yml`:

```yaml
include:
  - template: Jobs/SAST.gitlab-ci.yml
  - template: Jobs/Dependency-Scanning.gitlab-ci.yml
  - template: Jobs/Secret-Detection.gitlab-ci.yml
```

Also enable **push rules** on protected branches to reject unsigned commits, oversized files, or forbidden filenames.

---

# PART E — BITBUCKET

## 20. Bitbucket Account Setup

1. Sign up at <https://bitbucket.org>.
2. Verify email.
3. **Enable Two-Step Verification** — <https://bitbucket.org/account/settings/two-step-verification/>.
4. Add SSH & (optional) GPG keys — §3, §4.
5. Connect to Jira / Confluence at **Personal settings → Integrations** if applicable.

### 20.1 App passwords (not passwords)

Bitbucket **does not** accept your account password for Git over HTTPS. You must generate an **App Password** with the scopes you need:

- <https://bitbucket.org/account/settings/app-passwords/>

Scopes to know: `repository:read/write`, `pullrequest:read/write`, `pipeline:read/write`, `account:read`.

Use the app password as the "password" in HTTPS clones or CI systems.

### 20.2 Workspaces

Bitbucket organises everything under a **workspace** (`<workspace-id>/<repo-slug>`). Every user has a personal workspace by default. Teams create shared workspaces.

Docs: <https://support.atlassian.com/bitbucket-cloud/docs/workspaces/>

### 20.3 Bitbucket Server / Data Center note

**Atlassian retired Bitbucket Server on 15 February 2024** — <https://www.atlassian.com/software/bitbucket/server>. Only **Bitbucket Cloud** (SaaS) and **Bitbucket Data Center** (self-managed, enterprise-tier) remain. If you're joining a company still on Server, plan for migration.

---

## 21. Bitbucket CLI Reality Check

> **Atlassian does not ship a first-party CLI comparable to `gh` or `glab`.**
> Do not assume there is one and do not install unknown "bitbucket cli" binaries from search results.

Your realistic options:

### 21.1 Just use plain `git` + the browser

For 90% of workflows, `git push` + creating the PR in the browser is fine. Bitbucket sends the PR URL as `git push` output.

### 21.2 Atlassian REST API + `curl` / scripts

Official REST API: <https://developer.atlassian.com/cloud/bitbucket/rest/intro/>

Example — list your open PRs:

```bash
curl -u "$BB_USER:$BB_APP_PASSWORD" \
     "https://api.bitbucket.org/2.0/repositories/{workspace}/{repo_slug}/pullrequests?state=OPEN"
```

Bitbucket's REST API is the *actual* portable automation surface.

### 21.3 Commercial partner CLI — Appfire ACLI

The **Atlassian CLI (ACLI) for Bitbucket** is built by **Appfire**, a verified Atlassian partner, and sold through the Atlassian Marketplace. It's paid but production-grade for admin/automation tasks.

- Marketplace listing: <https://marketplace.atlassian.com/plugins/org.swift.bitbucket.cli>
- Bitbucket Cloud version: <https://marketplace.atlassian.com/apps/1211199/bitbucket-cloud-command-line-interface-cli>

If your organisation already licenses Appfire ACLI for Jira or Confluence, using it for Bitbucket is straightforward.

### 21.4 Community CLIs

**Not officially supported.** Evaluate maintenance, licence, and security carefully before adopting. Examples:

- `bitbucket-cloud-cli` (Python, PyPI) — <https://pypi.org/project/bitbucket-cloud-cli/> — install with `pipx install bitbucket-cloud-cli`. Command name: `bb`.
- `iamngoni/bitbucket-cli` (Rust) — <https://github.com/iamngoni/bitbucket-cli>.

Treat them as convenience wrappers around the REST API, not as officially blessed tooling.

### 21.5 IDE integration

JetBrains IDEs and VS Code have **first-class Bitbucket integration** via extensions:

- **VS Code — Atlassian for VS Code** — <https://marketplace.visualstudio.com/items?itemName=Atlassian.atlascode>
- **JetBrains — Bitbucket integration** is bundled

These give you PR review, pipeline status, and Jira issue linking without leaving the editor. For most Bitbucket users, this is the pragmatic "CLI substitute."

---

## 22. Bitbucket Pull Request Workflow

```bash
git checkout main && git pull
git checkout -b feat/auth-jwt-refresh

git commit -m "feat(auth): add refresh-token rotation"
git push -u origin feat/auth-jwt-refresh
# Bitbucket prints a "Create pull request" URL — open it
```

In the Bitbucket UI:

1. **Create pull request** → set title, description, reviewers.
2. Link a Jira issue by putting the key in the branch name or PR title (e.g. `PROJ-123`) — Bitbucket auto-links it.
3. Reviewers approve → CI passes → **Merge** (choose Merge commit / Squash / Fast-forward).

**Branch permissions** — protect `main` under **Repository settings → Branch permissions**:

- Prevent direct pushes
- Require minimum approvers
- Require successful builds

Docs: <https://support.atlassian.com/bitbucket-cloud/docs/set-branch-permissions/>

---

## 23. Bitbucket Pipelines — CI/CD Basics

Bitbucket reads `bitbucket-pipelines.yml` at the repo root. Docs: <https://support.atlassian.com/bitbucket-cloud/docs/get-started-with-bitbucket-pipelines/>.

Minimal Node.js pipeline:

```yaml
image: node:24-alpine

definitions:
  caches:
    npm: node_modules

pipelines:
  default:
    - step:
        name: Install & test
        caches: [npm]
        script:
          - npm ci
          - npm run lint
          - npm test
          - npm run build
        artifacts:
          - dist/**

  branches:
    main:
      - step:
          name: Deploy to production
          deployment: production
          trigger: manual
          script:
            - ./scripts/deploy.sh
```

**Repository variables** (secrets) live at **Repository settings → Repository variables**. Mark them **Secured**.

Bitbucket Pipelines uses "build minutes" from your plan — <https://www.atlassian.com/software/bitbucket/pricing>.

---

## 24. Bitbucket Security

Turn on for real projects:

- **Two-Step Verification** at the workspace level
- **IP allowlist** (Premium)
- **Required 2FA for workspace members** (Premium)
- **Branch permissions** on protected branches
- **Secret scanning** — <https://support.atlassian.com/bitbucket-cloud/docs/secret-scanning/>
- **Snyk integration** for dependency / container scanning — <https://snyk.io/docs/snyk-cli-snyk-for-bitbucket-cloud/>
- **Audit logs** — <https://support.atlassian.com/bitbucket-cloud/docs/view-and-download-audit-logs/>

---

# PART F — DAILY WORKFLOWS

## 25. Feature Branch Workflow

Works identically across all three hosts:

```
main  ●────●────●────●────●────●────●────● (protected)
              \        /         \        /
               ●──●──●            ●──●──●      (feat branches)
               PR/MR merged        PR/MR merged
```

Rules:

1. `main` is always deployable.
2. All work happens on short-lived branches (`feat/…`, `fix/…`, `chore/…`, `docs/…`).
3. Merges via PR/MR only. Require CI + review.
4. Prefer **squash-merge** for clean history; **merge commit** if you need the branch history preserved; **rebase** if you want a fully linear log.
5. Delete the branch after merge.

Branch naming:

```
feat/PROJ-123-user-profile
fix/PROJ-456-crash-on-null-token
chore/upgrade-node-24
docs/onboarding-guide
```

---

## 26. Common Commands Cheat Sheet

```bash
# ---------- Inspect ----------
git status -sb                          # short status
git log --oneline --graph --decorate --all -30
git diff                                # unstaged
git diff --cached                       # staged
git diff main..HEAD                     # branch vs main
git blame path/to/file                  # who changed each line
git show <sha>                          # inspect a commit
git shortlog -sn --all                  # authors by commit count

# ---------- Stage & commit ----------
git add -p                              # interactive hunks
git commit -m "feat(x): summary"
git commit --amend --no-edit            # tweak last commit
git commit --fixup=<sha>                # for later autosquash

# ---------- Branch ----------
git switch main                         # modern alternative to checkout
git switch -c feat/new-thing
git branch -d feat/done                 # delete merged
git branch -D feat/broken               # force delete
git branch --show-current

# ---------- Sync ----------
git fetch --all --prune
git pull --rebase
git push -u origin HEAD                 # first push of a new branch
git push --force-with-lease             # safer than --force

# ---------- Rebase ----------
git rebase -i main                      # interactive rebase onto main
git rebase -i --autosquash main         # squashes fixup! commits
git rebase --continue / --abort

# ---------- Stash ----------
git stash push -m "wip: refactor"
git stash list
git stash pop
git stash apply stash@{2}

# ---------- Tag & release ----------
git tag -a v1.2.0 -m "Release 1.2.0"
git push --tags
git tag -d v1.2.0                       # local delete
git push origin --delete v1.2.0         # remote delete

# ---------- Cherry-pick ----------
git cherry-pick <sha>                   # bring a commit into current branch
git cherry-pick --continue / --abort

# ---------- Worktrees (multiple branches at once) ----------
git worktree add ../myrepo-hotfix hotfix/urgent
git worktree list
git worktree remove ../myrepo-hotfix
```

---

## 27. Rescue Recipes (Undo, Fix, Recover)

### 27.1 "I committed to the wrong branch"

```bash
git log --oneline -3                    # note the last-commit sha
git reset --hard HEAD~1                 # remove from current branch
git switch correct-branch
git cherry-pick <sha>
```

### 27.2 "I want to undo the last commit but keep the changes"

```bash
git reset --soft HEAD~1                 # keep index + working tree
```

### 27.3 "I need to unstage a file"

```bash
git restore --staged path/to/file       # modern
git reset HEAD -- path/to/file          # traditional
```

### 27.4 "I want to discard local changes to a file"

```bash
git restore path/to/file
```

### 27.5 "I force-pushed and lost my work"

```bash
git reflog                              # find the lost commit sha
git reset --hard <sha>                  # restore
```

`git reflog` is the ultimate escape hatch — Git keeps ref-updates locally for 90 days by default.

### 27.6 "I need to remove a secret that was committed"

Do **all** of these:

1. **Rotate the secret immediately.** Assume it's compromised — the internet has indexed it.
2. Rewrite history with **`git filter-repo`** (official successor to `filter-branch`) — <https://github.com/newren/git-filter-repo>:

   ```bash
   pip install git-filter-repo
   git filter-repo --path secrets.env --invert-paths
   git push --force origin main
   ```
3. Force all collaborators to re-clone.
4. Enable **secret scanning** on the host (GitHub / GitLab / Bitbucket) to catch the next one.

### 27.7 "I want to merge just one file from another branch"

```bash
git checkout other-branch -- path/to/file
```

### 27.8 "I need to see who deleted a file"

```bash
git log --diff-filter=D --summary -- path/to/file
```

---

# PART G — REFERENCE

## 28. Commit Message Conventions

Adopt **Conventional Commits** — the de-facto standard used by every major automation tool:

Spec: <https://www.conventionalcommits.org>

```
<type>(<scope>): <short imperative summary>

<optional body — what and why, not how>

<optional footer — BREAKING CHANGE, refs, sign-off>
```

Common types:

| Type | Meaning |
|---|---|
| `feat` | New feature (SemVer minor) |
| `fix` | Bug fix (SemVer patch) |
| `docs` | Documentation only |
| `style` | Formatting, whitespace, no code change |
| `refactor` | Code change that is neither a fix nor a feature |
| `perf` | Performance improvement |
| `test` | Adding or updating tests |
| `build` | Build system or external dependencies |
| `ci` | CI configuration |
| `chore` | Housekeeping (versioning, deps, tooling) |
| `revert` | Reverts a previous commit |

Examples:

```
feat(auth): add refresh-token rotation
fix(api): return 400 instead of 500 on empty body
docs(readme): document Node.js 24 as the LTS baseline
refactor(db)!: rename users.email → users.email_address

BREAKING CHANGE: any consumer of the users table must update its column reference.
Refs PROJ-1234
```

Enforce it with **commitlint** (<https://commitlint.js.org>) or **committed** (Rust). Automate releases with **semantic-release** (<https://semantic-release.gitbook.io>) or **release-please** (<https://github.com/googleapis/release-please>).

---

## 29. Final Checklist

Git core
- [ ] Git 2.50+ installed from an official source
- [ ] Git LFS installed
- [ ] `user.name`, `user.email`, `init.defaultBranch=main` set
- [ ] `pull.rebase=true`, `fetch.prune=true`, `push.autoSetupRemote=true`
- [ ] `core.autocrlf` correct for the OS
- [ ] Global `.gitignore_global` wired via `core.excludesfile`
- [ ] Delta pager installed (optional but recommended)
- [ ] pre-commit framework installed and hooks configured

Identity & security
- [ ] Ed25519 SSH keys per host, in `~/.ssh/id_ed25519_<host>`
- [ ] `~/.ssh/config` with per-host `IdentityFile` + `IdentitiesOnly yes`
- [ ] Public keys uploaded to each platform
- [ ] Commit signing (SSH or GPG) configured and verified
- [ ] `ssh -T git@github.com`, `git@gitlab.com`, `git@bitbucket.org` all succeed
- [ ] 2FA enabled on GitHub, GitLab, Bitbucket
- [ ] gitleaks pre-commit hook installed

GitHub
- [ ] `gh` CLI installed and `gh auth status` shows logged-in
- [ ] SSH & signing keys added
- [ ] Fine-grained PATs generated where needed
- [ ] Dependabot + secret scanning + CodeQL enabled per repo
- [ ] Branch protection rules on `main`
- [ ] CODEOWNERS in place

GitLab
- [ ] `glab` CLI installed and `glab auth status` shows logged-in
- [ ] SSH & signing keys added
- [ ] Personal Access Token with scopes limited to actual need
- [ ] `glab ci lint` used before pushing `.gitlab-ci.yml`
- [ ] Protected branches + approvals configured
- [ ] SAST / Dependency / Secret Detection templates included

Bitbucket
- [ ] SSH keys added
- [ ] App Password generated for HTTPS / CI (scoped)
- [ ] 2FA on
- [ ] Workspace variables marked **Secured**
- [ ] Branch permissions on `main`
- [ ] Jira integration linked (if applicable)
- [ ] IDE plugin (Atlascode for VS Code / built-in JetBrains) installed as CLI substitute

Reproducibility
- [ ] `~/.gitconfig`, `~/.gitignore_global`, `~/.ssh/config` all in a private dotfiles repo
- [ ] `.ssh/id_ed25519_*` private keys backed up encrypted (age / GPG / password manager)
- [ ] Documented list of hosts + accounts for every project you work on

---

## 🏁 Final Principle

> **The three hosts differ in features and pricing, but Git itself is the same everywhere. Master the CLI, keep identities separated by host, sign your commits, and let the platforms compete on the rest.**

Sequence for any new environment:

```
Install Git → configure identity → generate per-host SSH keys
        ↓
Add keys to each host + enable 2FA
        ↓
Enable commit signing
        ↓
Install `gh` and/or `glab`
        ↓
Set up per-host ~/.ssh/config
        ↓
Install pre-commit + gitleaks
        ↓
Clone → branch → commit → PR/MR → merge → repeat
        ↓
Back up dotfiles + encrypted keys
```

Everything else — branching strategies, CI shape, review policy — is a team decision. The setup above is the neutral foundation you can drop into any team using any host on day one.
