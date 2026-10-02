# 🧰 Terminal & CLI Engineering — Production Engineering Guide

> Two halves of the same craft. **Part A — the terminal as an engineering workstation**: terminal emulators, shells and prompts, multiplexers, dotfiles, modern CLI tools, SSH workflows, remote development, and safe production terminal habits. **Part B — building production-grade CLI tools**: design principles (Command Line Interface Guidelines), argument conventions, help, output for humans and machines, stdout/stderr, exit codes, colour and TTY detection (`NO_COLOR`), interactivity, configuration precedence and XDG paths, secrets, errors, signals, performance, frameworks per language with examples, TUIs, testing, cross-platform concerns, distribution (Homebrew, winget, Scoop, apt/rpm, npm/pipx/uv, cargo, go install), signing, versioning, updates, completions, man pages, telemetry, and security.
>
> This chapter connects the command references in this repository:
> **[25 Windows PowerShell](./25-windows-powershell.md)** · **[26 Windows Command Prompt](./26-windows-command-prompt.md)** · **[27 Linux Terminal](./27-linux-terminal.md)** · **[28 Shell Scripting](./28-shell-scripting.md)** · [37 Package managers](./37-package-managers.md) · [38 Workstation stack](./38-developer-desktop-software-tool-stack.md) · [41 Fresh setups](./41-(a)Windows-11-Fresh-Setup.md) · [42 Git & SSH](./42-Git-GitHub-GitLab-Bitbucket-Setup.md)

---

## 📚 Table of Contents

**Part A — The Terminal Workstation**
1. [Terminal Concepts](#1-terminal-concepts)
2. [Terminal Emulators](#2-terminal-emulators)
3. [Shells and Prompts](#3-shells-and-prompts)
4. [Multiplexers and Session Management](#4-multiplexers-and-session-management)
5. [Dotfiles and Environment Management](#5-dotfiles-and-environment-management)
6. [Modern CLI Toolkit](#6-modern-cli-toolkit)
7. [Working with Structured Data in the Terminal](#7-working-with-structured-data-in-the-terminal)
8. [SSH Workflows](#8-ssh-workflows)
9. [Remote and Container-Based Development](#9-remote-and-container-based-development)
10. [Safe Production Terminal Habits](#10-safe-production-terminal-habits)

**Part B — Building CLI Tools**
11. [When to Build a CLI](#11-when-to-build-a-cli)
12. [CLI Design Principles](#12-cli-design-principles)
13. [Commands, Arguments, and Flags](#13-commands-arguments-and-flags)
14. [Help and Documentation](#14-help-and-documentation)
15. [Output — Humans and Machines](#15-output--humans-and-machines)
16. [Exit Codes and Errors](#16-exit-codes-and-errors)
17. [Colour, TTY Detection, and Accessibility](#17-colour-tty-detection-and-accessibility)
18. [Interactivity and Non-Interactive Use](#18-interactivity-and-non-interactive-use)
19. [Configuration, Environment, and Paths](#19-configuration-environment-and-paths)
20. [Secrets and Authentication in CLIs](#20-secrets-and-authentication-in-clis)
21. [Signals, Cancellation, and Long-Running Operations](#21-signals-cancellation-and-long-running-operations)
22. [Performance and Startup Time](#22-performance-and-startup-time)
23. [Frameworks by Language](#23-frameworks-by-language)
24. [Terminal UIs (TUIs)](#24-terminal-uis-tuis)
25. [Testing CLIs](#25-testing-clis)
26. [Cross-Platform Concerns](#26-cross-platform-concerns)
27. [Distribution and Packaging](#27-distribution-and-packaging)
28. [Versioning, Updates, and Compatibility](#28-versioning-updates-and-compatibility)
29. [Shell Completions and Man Pages](#29-shell-completions-and-man-pages)
30. [Telemetry and Privacy](#30-telemetry-and-privacy)
31. [CLI Security](#31-cli-security)
32. [Checklists](#32-checklists)
33. [References](#33-references)

---

# Part A — The Terminal Workstation

## 1. Terminal Concepts

| Term | Meaning |
|---|---|
| **Terminal emulator** | The GUI application that renders text and sends keystrokes (Windows Terminal, iTerm2, GNOME Terminal…) |
| **Shell** | The command interpreter running inside it (bash, zsh, fish, PowerShell, cmd) |
| **TTY / PTY** | (Pseudo-)terminal device connecting the emulator and processes; programs detect it to decide on colours/interactivity |
| **Standard streams** | stdin (0), stdout (1), stderr (2) |
| **Exit status** | Integer a process returns; `0` = success |
| **Environment variables** | Key-value context inherited by child processes |
| **PATH** | Ordered list of directories searched for commands |
| **Job control** | Foreground/background processes (`&`, `jobs`, `fg`, `bg`, `Ctrl+Z`) |

```text
Pipelines connect stdout → stdin:  producer | filter | consumer
stderr is separate so errors don't corrupt piped data:  cmd 2>errors.log | jq .
```

Shell-specific references: **27** (Linux/Bash), **28** (scripting), **25** (PowerShell — objects in the pipeline, not text), **26** (cmd.exe).

---

## 2. Terminal Emulators

| Platform | Options | Notes |
|---|---|---|
| Windows | **Windows Terminal** (default on Windows 11), WezTerm, Alacritty | Profiles for PowerShell 7, WSL distros, cmd, SSH — see **41(a)** |
| macOS | Terminal.app, **iTerm2**, WezTerm, Ghostty, Alacritty, Kitty | Setup: **41(b)** |
| Linux | GNOME Terminal/Console, Konsole, Kitty, Alacritty, WezTerm, Ghostty, foot (Wayland) | — |
| Cross-platform | WezTerm, Alacritty, Kitty (macOS/Linux), Ghostty (macOS/Linux) | GPU-accelerated; configuration as code |

Recommended settings:

```text
- A font with good glyph coverage and ligature options (and a Nerd Font variant if your prompt uses icons)
- Large scrollback; copy-on-select optional; bracketed paste enabled (protects against pasted multi-line commands executing instantly)
- Distinct colour scheme / tab colour for production profiles (visual safety cue — §10)
- Unicode/UTF-8 everywhere (Indic scripts and emoji render correctly)
```

---

## 3. Shells and Prompts

| Shell | Strengths | Typical use |
|---|---|---|
| **bash** | Ubiquitous, POSIX-ish, default on most Linux servers | Scripts (with `#!/usr/bin/env bash`), servers |
| **zsh** | Powerful completion, plugins; default on macOS | Interactive use on macOS/Linux |
| **fish** | Friendly defaults, autosuggestions, syntax highlighting | Interactive use (not POSIX — don't write portable scripts in it) |
| **PowerShell 7** (`pwsh`) | Object pipeline, cross-platform, rich Windows administration | Windows automation, cross-platform scripting (chapter 25) |
| **Nushell** | Structured data pipelines | Data-heavy interactive work |
| `sh` (dash/POSIX) | Minimal, fast | Portable scripts, containers |

```text
Rule of thumb: interactive shell = whatever makes you productive; script shell = bash or POSIX sh (Linux/macOS)
and PowerShell (Windows), with strict mode and ShellCheck/PSScriptAnalyzer (chapter 28, chapter 06 §20).
```

### Prompt essentials

| Show | Why |
|---|---|
| Current directory and Git branch/status | Context |
| Exit code of the last command (when non-zero) | Notice failures |
| Kubernetes context/namespace and cloud profile | **Safety** — know which cluster/account you are about to change |
| Python venv / Node version | Toolchain context |
| Hostname when in SSH sessions | Know you're on a remote machine |

Cross-shell prompt: **Starship** (fast, configurable, works in bash/zsh/fish/PowerShell).

```toml
# ~/.config/starship.toml (excerpt)
[kubernetes]
disabled = false
format = '[⎈ $context/$namespace]($style) '
[kubernetes.context_aliases]
"arn:aws:eks:ap-south-1:123456789012:cluster/prod" = "🔥PROD"
[aws]
format = '[☁ $profile($region)]($style) '
```

---

## 4. Multiplexers and Session Management

| Tool | Use |
|---|---|
| **tmux** | Persistent sessions on servers (survive SSH disconnects), splits, windows, scripting |
| GNU Screen | Legacy equivalent, often preinstalled |
| Zellij | Modern multiplexer with discoverable keybindings |
| Terminal-native splits/tabs | Windows Terminal, iTerm2, WezTerm panes for local work |

```bash
tmux new -s deploy            # new named session
tmux attach -t deploy         # reattach after disconnect
tmux ls                       # list sessions
# inside tmux: Ctrl+b d (detach), Ctrl+b % (vertical split), Ctrl+b " (horizontal split), Ctrl+b [ (scroll/copy mode)
```

```text
🟠 Run long remote operations (migrations, backfills, restores) inside tmux/screen so a dropped connection doesn't kill them
   — better still, run them as jobs (Kubernetes Job, systemd unit, CI job) with logs (chapter 13 §21)
```

---

## 5. Dotfiles and Environment Management

### 5.1 Dotfiles

```text
dotfiles/                 # Git repository (public or private — never commit secrets)
├── zsh/.zshrc
├── bash/.bashrc
├── git/.gitconfig        # see 42 for recommended Git config
├── starship/starship.toml
├── tmux/.tmux.conf
├── powershell/Microsoft.PowerShell_profile.ps1
└── install.sh / install.ps1
```

Managers: **chezmoi** (templating, secrets via password managers, cross-OS), GNU Stow, yadm, bare Git repo technique.

### 5.2 Toolchain and project environments

| Tool | Purpose |
|---|---|
| **mise** (formerly rtx) / asdf | Per-project runtime versions (Node, Python, Java, Go…) via `.tool-versions`/`mise.toml` |
| nvm / fnm / Volta | Node.js versions |
| pyenv / **uv** | Python versions and virtual environments (see **37**) |
| SDKMAN! | JVM toolchains |
| **direnv** | Load/unload env vars per directory (`.envrc`) — never commit real secrets |
| Dev Containers / Nix / Devbox | Reproducible full environments |

```toml
# mise.toml (project root)
[tools]
node = "24"
python = "3.13"
java = "temurin-25"
[env]
APP_ENV = "local"
```

---

## 6. Modern CLI Toolkit

These complement (not replace) the classic tools documented in **27**. Install via your package manager (**37**, **41**).

| Need | Classic | Modern alternative | Why |
|---|---|---|---|
| Search file contents | `grep -r` | **ripgrep** (`rg`) | Fast, respects `.gitignore` |
| Find files | `find` | **fd** | Simpler syntax, fast |
| Fuzzy finding | — | **fzf** | Interactive selection for files, history, branches |
| View files | `cat`, `less` | **bat** | Syntax highlighting, Git markers |
| List files | `ls` | **eza** | Git status, tree view |
| Jump directories | `cd` | **zoxide** (`z`) | Frecency-based navigation |
| Diff | `diff` | **delta**, difftastic | Readable Git diffs (configure as Git pager — 42) |
| HTTP | `curl` | **xh** / HTTPie | Friendly HTTP; keep `curl` for scripts |
| JSON | — | **jq** | Query/transform JSON |
| YAML/TOML/XML | — | **yq** | jq-like for YAML and more |
| Disk usage | `du` | **dust**, ncdu | Visual overview |
| Processes | `top` | **btop**, htop | Better overview |
| Benchmark commands | `time` | **hyperfine** | Statistical command benchmarking |
| Git TUI | — | lazygit, tig | Faster interactive Git |
| Kubernetes | `kubectl` | **k9s**, kubectx/kubens, stern | Cluster TUI, context switching, multi-pod logs |
| Cheat sheets | `man` | tldr | Example-driven help |

```bash
rg -n "Idempotency-Key" --type java            # where is the header used?
fd -e sql migrations | xargs -I{} wc -l {}     # count lines in migration files
git branch --sort=-committerdate | fzf | xargs git switch
hyperfine 'npm run build' 'pnpm run build'     # compare build tools
```

---

## 7. Working with Structured Data in the Terminal

```bash
# jq: extract fields, filter, reshape
curl -s https://api.shopnow.example/v1/orders?limit=50 -H "Authorization: Bearer $TOKEN" \
  | jq -r '.data[] | select(.status=="payment_failed") | [.id, .total.amountMinor] | @tsv'

# yq: edit YAML in place (e.g. bump an image digest in a Kustomize overlay)
yq -i '.images[0].digest = "sha256:abc123..."' overlays/staging/kustomization.yaml

# kubectl + jq: pods not Ready
kubectl get pods -A -o json | jq -r '.items[] | select([.status.conditions[]? | select(.type=="Ready" and .status!="True")] | length > 0) | "\(.metadata.namespace)/\(.metadata.name)"'
```

```powershell
# PowerShell works with objects natively (chapter 25)
Invoke-RestMethod "https://api.shopnow.example/v1/orders?limit=50" -Headers @{ Authorization = "Bearer $env:TOKEN" } |
  Select-Object -ExpandProperty data |
  Where-Object status -eq 'payment_failed' |
  Select-Object id, @{n='amount'; e={ $_.total.amountMinor }}
```

---

## 8. SSH Workflows

Key generation, agent setup, and multi-host configuration: **42 §3, §9**. Server hardening: **39**.

```sshconfig
# ~/.ssh/config (excerpt)
Host bastion-prod
  HostName bastion.prod.shopnow.example
  User ops
  IdentityFile ~/.ssh/id_ed25519
  IdentitiesOnly yes

Host app-prod-*
  ProxyJump bastion-prod          # hop through bastion without agent forwarding
  User deploy
  StrictHostKeyChecking yes

Host *
  ServerAliveInterval 30
  ControlMaster auto              # connection multiplexing (macOS/Linux)
  ControlPath ~/.ssh/cm-%r@%h:%p
  ControlPersist 10m
```

| Practice | Status |
|---|:---:|
| Ed25519 keys (or hardware-backed keys: FIDO2 `ed25519-sk`), passphrase-protected, in an agent | 🔴 |
| `ProxyJump` instead of agent forwarding (`ForwardAgent yes` lets a compromised host use your keys) | 🔴 |
| Verify host keys; distribute known_hosts or use SSH certificates | 🔴 |
| Prefer identity-aware access (SSM Session Manager, IAP, SSH certificates from a CA with short TTLs) over long-lived keys on servers | 🟠 |
| `scp` replaced by `rsync`/`sftp` for large or resumable transfers | 🟡 |

```bash
rsync -avz --partial --progress ./backup.dump app-prod-1:/var/backups/   # resumable transfer
ssh -L 5433:db.internal:5432 bastion-prod                                # local port forward to a private DB (temporary, audited)
```

---

## 9. Remote and Container-Based Development

| Approach | Tools | Use |
|---|---|---|
| Remote SSH editing | VS Code Remote-SSH, JetBrains Gateway | Powerful remote machines, large repos |
| WSL2 | Linux toolchain on Windows (see **41(a)**) | Windows developers targeting Linux |
| **Dev Containers** | `devcontainer.json` (VS Code, JetBrains, Codespaces, Dev Pod) | Reproducible, onboarding in minutes |
| Cloud dev environments | GitHub Codespaces, Gitpod/Ona, Coder | Standardised environments, no local setup |

```json
// .devcontainer/devcontainer.json (excerpt)
{
  "name": "orders-service",
  "image": "mcr.microsoft.com/devcontainers/java:25",
  "features": {
    "ghcr.io/devcontainers/features/node:1": { "version": "24" },
    "ghcr.io/devcontainers/features/docker-in-docker:2": {}
  },
  "postCreateCommand": "./gradlew dependencies",
  "customizations": { "vscode": { "extensions": ["vscjava.vscode-java-pack", "redhat.vscode-yaml"] } }
}
```

IDE extension recommendations: **30–36**.

---

## 10. Safe Production Terminal Habits

```text
🔴 Default to read-only access; elevate just-in-time for writes (chapter 21 §14)
🔴 Make production visually obvious: red prompt/tab, "PROD" in the prompt, k8s/cloud context displayed (§3)
🔴 Confirm the target before destructive commands (kubectl config current-context; aws sts get-caller-identity; hostname)
🔴 Prefer pipelines/GitOps/runbook automation over ad-hoc commands; record any manual production command in the incident/change log
🔴 Never paste secrets into commands that end up in shell history; use env vars from secret managers or prompts
🟠 Use --dry-run / plan modes first (kubectl --dry-run=server, terraform plan, rsync --dry-run)
🟠 Use tmux for long operations; set timeouts; log output (script/tee)
🟠 Separate shells/profiles for production; avoid having prod and staging contexts in the same session
🟠 HISTCONTROL=ignorespace (bash) so commands prefixed with a space aren't saved — and still avoid secrets on the command line
```

```bash
# Guard function for kubectl in production (illustrative)
kprod() {
  local ctx; ctx=$(kubectl config current-context)
  [[ "$ctx" == *prod* ]] || { echo "Not a prod context: $ctx" >&2; return 1; }
  read -r -p "Run on $ctx: kubectl $* ? [type 'yes'] " ans
  [[ "$ans" == "yes" ]] && kubectl "$@"
}
```

---

# Part B — Building CLI Tools

## 11. When to Build a CLI

| Build a CLI when | Prefer something else when |
|---|---|
| Developers/operators run a task repeatedly | Non-technical users → web UI |
| It must be scripted, automated, or used in CI | One-off task → a documented script in the repo |
| Wrapping an API for humans (like `gh`, `aws`, `kubectl`) | Complex visual workflows → GUI/TUI |
| Local development workflows (scaffolding, codegen, migrations) | Task runners suffice → Make, Task, just, npm scripts |

```text
Start with a script (chapter 28). Promote it to a real CLI when it gains users, flags, configuration, and support needs.
```

---

## 12. CLI Design Principles

Adapted from the **Command Line Interface Guidelines** (clig.dev) and long-standing UNIX conventions:

| # | Principle |
|---:|---|
| 1 | **Human-first, machine-friendly** — readable by default, with stable machine output (`--json`) for scripts. |
| 2 | **Composable** — read stdin, write stdout, errors to stderr, meaningful exit codes; play well in pipelines. |
| 3 | **Consistent** — follow conventions users already know (`-h/--help`, `--version`, `--`, `-v`, `-q`). |
| 4 | **Discoverable** — great help, examples, suggestions on typos ("did you mean…?"), shell completion. |
| 5 | **Say just enough** — no noise on success unless useful; clear progress for long tasks; quiet mode for scripts. |
| 6 | **Errors are documentation** — what went wrong, why, and how to fix it. |
| 7 | **Safe by default** — confirm destructive actions; `--dry-run`; never surprise users. |
| 8 | **Robust** — handle bad input, network failures, interrupts (Ctrl+C) gracefully; idempotent where possible. |
| 9 | **Fast** — start quickly; show feedback within ~100 ms. |
| 10 | **Stable** — treat flags, output formats, and exit codes as a public API (SemVer). |

---

## 13. Commands, Arguments, and Flags

### 13.1 Structure

```text
tool [global flags] <command> [<subcommand>] [flags] [--] [arguments]

shopctl orders list --status paid --limit 50
shopctl orders get ord_7f3kq2m9x1 -o json
shopctl deploy orders --env staging --dry-run
```

| Convention | Rule |
|---|---|
| Subcommands | noun-verb (`orders list`) or verb-noun (`get orders`) — pick one and be consistent (kubectl uses verb-noun, gh uses noun-verb) |
| Short flags | Single dash, single letter (`-o json`, `-v`); combinable (`-vq`) |
| Long flags | Double dash, kebab-case (`--dry-run`, `--output json` or `--output=json`) |
| `--` | Ends flag parsing; everything after is positional (`tool run -- --not-a-flag`) |
| Standard flags | `-h/--help`, `--version`, `-v/--verbose`, `-q/--quiet`, `-o/--output`, `-f/--force` (destructive), `-n/--dry-run` where unambiguous, `--no-color`, `--yes` (skip prompts) |
| Positional args | Few and obvious (IDs, file paths); use flags for anything optional or ambiguous |
| Booleans | `--feature` / `--no-feature` |
| Repeated values | `--label a --label b` or `--label a,b` (document which) |
| Stdin | Accept `-` as "read from stdin" for file arguments |
| Dangerous defaults | None: deleting/overwriting requires an explicit flag or confirmation |

POSIX Utility Syntax Guidelines and GNU long-option conventions are the baseline users expect on Linux/macOS; on Windows, PowerShell users expect `-Name` style parameters — for cross-platform tools, GNU-style flags are the norm.

---

## 14. Help and Documentation

```text
$ shopctl orders list --help
List orders for the current tenant.

Usage:
  shopctl orders list [flags]

Examples:
  # Orders that failed payment today
  shopctl orders list --status payment_failed --since 24h

  # Machine-readable output for scripts
  shopctl orders list --status paid -o json | jq '.[].id'

Flags:
  -s, --status string   Filter by status (created|payment_pending|payment_failed|paid|shipped|cancelled|refunded)
      --since duration  Only orders created within this duration (e.g. 1h, 24h, 7d)
  -l, --limit int       Maximum results (1–500) (default 50)
  -o, --output string   Output format: table|json|yaml (default "table")
  -h, --help            Help for list

Global Flags:
      --profile string  Config profile (default "default")
  -v, --verbose         Verbose output to stderr

Docs: https://docs.shopnow.example/cli/orders
```

```text
🔴 `-h`/`--help` on every command; `tool help <command>` also works
🔴 Lead with examples — the most-read part of help
🟠 Suggest corrections for typos ("Unknown command 'oders'. Did you mean 'orders'?")
🟠 Link to web docs for depth; generate a docs site and man pages from the same source (§29)
🟠 `tool` with no arguments shows concise help (not an error dump)
```

---

## 15. Output — Humans and Machines

### 15.1 Streams

```text
stdout → the program's primary output (data, results) — what pipes consume
stderr → logs, progress, warnings, errors, prompts — what humans read
Never print progress bars or log lines to stdout when it carries data.
```

### 15.2 Formats

| Mode | Use |
|---|---|
| Human (default when stdout is a TTY) | Aligned tables, colours, relative times ("3 min ago"), truncation |
| `-o json` / `--json` | Stable, documented schema for scripts; full values (no truncation); one JSON document or JSON Lines for streams |
| `-o yaml` | Config-like output |
| `-o name` / `--quiet` IDs only | Easy piping (`xargs`) |
| Plain (when stdout is not a TTY) | No colours or decorations; consider no truncation |

```text
🔴 Machine-readable output is a versioned contract — don't change field names/types in minor releases
🟠 Timestamps in RFC 3339 UTC in machine output; humans can get local time (e.g. IST) in table output
🟠 Paginate or stream large outputs; respect --limit
🟡 Offer --template / --jq style formatting like gh/kubectl for power users
```

### 15.3 Success output

```text
✓ Deployed orders@sha256:3f2a… to staging (canary 5%) in 42s
  Next: shopctl deploy promote orders --env staging
```

Short confirmation + next step; nothing at all is also fine for simple commands in quiet/script contexts.

---

## 16. Exit Codes and Errors

### 16.1 Exit codes

| Code | Meaning (common convention) |
|---:|---|
| 0 | Success |
| 1 | General failure |
| 2 | Misuse: invalid flags/arguments (used by many CLI frameworks and shells) |
| 3–63 | Tool-specific documented codes (e.g. 3 = not found, 4 = conflict, 5 = auth failure) |
| 64–78 | BSD `sysexits.h` conventions (64 usage, 65 data error, 69 unavailable, 75 temporary failure, 77 permission, 78 config) — optional |
| 124 | Timed out (used by `timeout` utility) |
| 126 / 127 | Command not executable / not found (shell) |
| 128 + N | Terminated by signal N (e.g. 130 = SIGINT/Ctrl+C, 143 = SIGTERM) |

```text
🔴 Document exit codes; keep them stable (they're part of the API)
🔴 Exit non-zero on any failure — including partial failure (document semantics)
🟠 Distinguish retryable (e.g. 75 temporary failure) from permanent errors for automation
```

### 16.2 Error messages

```text
❌ Error: 403
✅ Error: you don't have permission to deploy to production.
   Your role: developer (requires: deployer)
   Request access: https://access.shopnow.example/roles/deployer
   Run with --verbose for request details (trace ID: 4bf92f35…).
```

```text
- What happened, why (if known), what to do next; one line first, details after
- No stack traces by default (show with --verbose/--debug); include a trace/request ID for support
- Validate inputs early, before doing any work or side effects
- Suggest the exact command to fix things when possible
```

---

## 17. Colour, TTY Detection, and Accessibility

```text
Use colour and formatting only when:
  - stdout/stderr is a TTY (isatty), AND
  - the NO_COLOR environment variable is NOT set to a non-empty value (no-color.org convention), AND
  - TERM is not "dumb", AND
  - the user hasn't passed --no-color (or set the tool's config to disable it)
Allow forcing colour (--color=always, FORCE_COLOR / CLICOLOR_FORCE conventions) for CI logs that support it.
```

| Practice | Why |
|---|---|
| Never convey meaning by colour alone — add symbols/words ("✓ OK", "✗ FAILED") | Colour-blind users, monochrome terminals, screen readers |
| Keep emoji/symbols optional or ASCII-fallback | Some terminals/fonts and Windows consoles render poorly |
| Respect terminal width; wrap help text | Small terminals, screen magnification |
| Avoid excessive animation; disable spinners when not a TTY | Logs, screen readers, CI |
| Plain-text output mode | Screen-reader friendliness |

---

## 18. Interactivity and Non-Interactive Use

```text
🔴 Every interactive prompt has a flag equivalent (e.g. --yes, --env staging, --password-stdin)
🔴 Never prompt when stdin is not a TTY — fail with a clear message instead (CI must not hang)
🔴 Destructive operations: require confirmation interactively, or --yes/--force non-interactively;
    for very destructive actions, require typing the resource name
🟠 --dry-run shows exactly what would change
🟠 Detect CI (CI=true) to disable prompts, spinners, and colour unless forced
```

```text
$ shopctl db restore --env production --to "2026-10-02T03:40:00Z"
⚠ This will restore the PRODUCTION orders database to 2026-10-02T03:40:00Z.
  Writes after that time will be lost from the restored instance.
  Type the environment name to continue: production
```

---

## 19. Configuration, Environment, and Paths

### 19.1 Precedence (highest wins)

```text
1. Command-line flags
2. Environment variables (TOOL_*  e.g. SHOPCTL_PROFILE, SHOPCTL_API_URL)
3. Project-level config (./.shopctl.toml — found by walking up from the current directory)
4. User-level config (XDG config dir — below)
5. System-level config (/etc/shopctl/config.toml)
6. Built-in defaults
```

### 19.2 File locations

| OS | Config | Cache | State/data |
|---|---|---|---|
| Linux | `$XDG_CONFIG_HOME/tool` (default `~/.config/tool`) | `$XDG_CACHE_HOME/tool` (`~/.cache/tool`) | `$XDG_STATE_HOME`/`$XDG_DATA_HOME` (`~/.local/state`, `~/.local/share`) |
| macOS | `~/Library/Application Support/tool` (many CLIs also honour XDG paths — be consistent and documented) | `~/Library/Caches/tool` | `~/Library/Application Support/tool` |
| Windows | `%APPDATA%\tool` | `%LOCALAPPDATA%\tool\cache` | `%LOCALAPPDATA%\tool` |

```text
🔴 Never write into the current directory or home root without being asked
🟠 `tool config path` / `tool config view` commands help users debug configuration
🟠 Profiles for multiple environments/accounts (--profile, TOOL_PROFILE)
🟠 Validate config on load with clear errors (file, line, key)
```

---

## 20. Secrets and Authentication in CLIs

```text
🔴 Never accept secrets as plain command-line flags (visible in process lists and shell history) — use:
    prompts (no echo), --password-stdin / --token-stdin, environment variables, OS keychains, or credential helpers
🔴 Store tokens in the OS credential store (macOS Keychain, Windows Credential Manager, Linux Secret Service); fall back to
    a config file with 0600 permissions only if necessary, and warn
🔴 Prefer OAuth device authorization flow or browser-based login (Authorization Code + PKCE with a loopback redirect) for user auth
🔴 Short-lived tokens with refresh; `tool auth logout` revokes and deletes stored credentials
🟠 For CI: support workload identity/OIDC tokens and environment-provided tokens; document least-privilege scopes
🟠 Redact secrets in --verbose/--debug output
```

```bash
# Good patterns
echo "$REGISTRY_TOKEN" | shopctl registry login --token-stdin
shopctl auth login            # opens browser or prints device code + URL
```

---

## 21. Signals, Cancellation, and Long-Running Operations

```text
🔴 Ctrl+C (SIGINT): stop promptly, clean up temp files/locks, print what state things are in, exit 130
🔴 SIGTERM (CI cancellation, container stop): same graceful behaviour
🟠 Second Ctrl+C forces immediate exit
🟠 Long operations: progress indicator on stderr (TTY only), elapsed time, and ETA when knowable
🟠 Make operations resumable or idempotent (re-running after interruption is safe)
🟠 Server-side long operations: return an operation ID; `tool ops wait <id>` / `--wait` / `--no-wait` (chapter 11 §7.4)
🟠 Timeouts with sensible defaults and --timeout flag
```

```go
ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)
defer stop()
if err := run(ctx); err != nil {
    if errors.Is(err, context.Canceled) {
        fmt.Fprintln(os.Stderr, "Interrupted. No changes were applied.")
        os.Exit(130)
    }
    fmt.Fprintln(os.Stderr, "Error:", err)
    os.Exit(1)
}
```

---

## 22. Performance and Startup Time

| Language | Startup notes |
|---|---|
| Go, Rust | Native binaries, fast startup — ideal for frequently invoked CLIs |
| Node.js | Bundle the CLI (esbuild/tsup), lazy-load heavy modules; consider single-executable applications or compiled runtimes (Bun/Deno compile) for distribution |
| Python | Lazy imports in subcommands; avoid importing heavy libraries at module import time; package with uv/pipx |
| JVM | Slow startup unless using GraalVM native image (picocli supports it) or AOT/CDS caches |
| .NET | Native AOT or ReadyToRun for fast startup and single-file distribution |

```bash
hyperfine --warmup 3 'shopctl --version' 'shopctl orders list -o json --limit 1'
```

```text
- Target: `--help`/`--version` in well under ~100 ms; first feedback within ~100 ms for any command
- Cache expensive lookups (API discovery, auth tokens) in the cache dir with TTLs
- Parallelise independent network calls; show partial results early
```

---

## 23. Frameworks by Language

| Language | Frameworks | Notes |
|---|---|---|
| **Go** | **Cobra** (+ Viper for config), urfave/cli, Kong | Cobra powers kubectl, gh, Hugo; generates completions and docs |
| **Rust** | **clap** (derive API), argh | Excellent help, completions (clap_complete), man pages (clap_mangen) |
| **Python** | **Typer** (type hints, built on Click), Click, argparse (stdlib), Rich (output) | Distribute via `uv tool`/pipx |
| **Node.js/TS** | Commander, yargs, oclif (plugins, large CLIs), citty/clipanion; Ink (React for TUIs) | Bundle for fast startup |
| **Java/Kotlin** | **picocli** (GraalVM native image support), Clikt (Kotlin) | |
| **.NET** | System.CommandLine, Spectre.Console.Cli | Spectre.Console for rich output |
| **PowerShell** | Advanced functions/modules with parameter attributes (chapter 25) | `Get-Help`-based docs |
| **Shell** | `getopts` (POSIX), `getopt` (GNU) | For small tools (chapter 28) |

### 23.1 Go — Cobra

```go
var listCmd = &cobra.Command{
    Use:   "list",
    Short: "List orders for the current tenant",
    Example: `  shopctl orders list --status payment_failed --since 24h
  shopctl orders list -o json | jq '.[].id'`,
    Args: cobra.NoArgs,
    RunE: func(cmd *cobra.Command, _ []string) error {
        status, _ := cmd.Flags().GetString("status")
        limit, _ := cmd.Flags().GetInt("limit")
        if limit < 1 || limit > 500 {
            return fmt.Errorf("--limit must be between 1 and 500 (got %d)", limit)
        }
        orders, err := client.ListOrders(cmd.Context(), status, limit)
        if err != nil {
            return err
        }
        return output.Render(cmd.OutOrStdout(), outputFormat, orders)   // table/json/yaml
    },
}

func init() {
    listCmd.Flags().StringP("status", "s", "", "Filter by status")
    listCmd.Flags().IntP("limit", "l", 50, "Maximum results (1–500)")
    ordersCmd.AddCommand(listCmd)
}
```

### 23.2 Rust — clap (derive)

```rust
use clap::{Parser, Subcommand, ValueEnum};

#[derive(Parser)]
#[command(name = "shopctl", version, about = "ShopNow command-line tool")]
struct Cli {
    #[arg(long, global = true, env = "SHOPCTL_PROFILE", default_value = "default")]
    profile: String,
    #[command(subcommand)]
    command: Commands,
}

#[derive(Subcommand)]
enum Commands {
    /// List orders for the current tenant
    Orders {
        #[arg(short, long)]
        status: Option<String>,
        #[arg(short, long, default_value_t = 50, value_parser = clap::value_parser!(u16).range(1..=500))]
        limit: u16,
        #[arg(short, long, value_enum, default_value_t = Output::Table)]
        output: Output,
    },
}

#[derive(Clone, ValueEnum)]
enum Output { Table, Json, Yaml }
```

### 23.3 Python — Typer

```python
import json
import sys
from enum import Enum
import typer

app = typer.Typer(help="ShopNow command-line tool", no_args_is_help=True)
orders = typer.Typer(help="Work with orders")
app.add_typer(orders, name="orders")

class Output(str, Enum):
    table = "table"
    json = "json"

@orders.command("list")
def list_orders(
    status: str | None = typer.Option(None, "--status", "-s", help="Filter by status"),
    limit: int = typer.Option(50, "--limit", "-l", min=1, max=500, help="Maximum results"),
    output: Output = typer.Option(Output.table, "--output", "-o"),
) -> None:
    """List orders for the current tenant."""
    try:
        rows = client.list_orders(status=status, limit=limit)
    except AuthError as e:
        typer.echo(f"Error: {e}. Run `shopctl auth login`.", err=True)
        raise typer.Exit(code=5)
    if output is Output.json:
        json.dump(rows, sys.stdout)
    else:
        render_table(rows)

if __name__ == "__main__":
    app()
```

### 23.4 Node.js — Commander

```ts
#!/usr/bin/env node
import { Command, Option } from 'commander';

const program = new Command().name('shopctl').description('ShopNow command-line tool').version(VERSION);

program.command('orders')
  .command('list')
  .description('List orders for the current tenant')
  .option('-s, --status <status>', 'Filter by status')
  .addOption(new Option('-o, --output <format>', 'Output format').choices(['table', 'json']).default('table'))
  .option('-l, --limit <n>', 'Maximum results (1–500)', (v) => parseInt(v, 10), 50)
  .action(async (opts) => {
    const orders = await client.listOrders(opts);
    if (opts.output === 'json') process.stdout.write(JSON.stringify(orders) + '\n');
    else printTable(orders);
  });

program.parseAsync().catch((err) => { console.error(`Error: ${err.message}`); process.exit(1); });
```

---

## 24. Terminal UIs (TUIs)

| Framework | Language |
|---|---|
| **Bubble Tea** (+ Lip Gloss, Bubbles) | Go |
| **Ratatui** | Rust |
| **Textual** (+ Rich) | Python |
| Ink | Node.js (React) |
| Spectre.Console | .NET |
| tview | Go |

```text
- Use TUIs for interactive exploration (k9s, lazygit style); keep every capability available non-interactively too
- Support keyboard navigation with discoverable shortcuts (help panel), resizing, and NO_COLOR
- Restore terminal state on exit/crash (alternate screen, raw mode)
```

---

## 25. Testing CLIs

| Test type | Approach |
|---|---|
| Unit | Command handlers as functions with injected I/O (stdout/stderr writers, env, filesystem, clock) |
| Integration | Run the binary/entry point in a temp directory with a fake HOME/config; assert stdout, stderr, exit code |
| **Golden/snapshot tests** | Compare help text and output against checked-in files (update with an explicit flag) |
| API mocking | Local HTTP stubs (httptest, WireMock, MSW) for remote calls |
| Cross-platform | CI matrix on Linux, macOS, Windows (paths, line endings, shells) |
| Shell scripts | **bats-core** for Bash; Pester for PowerShell |
| Completions | Verify generated completion scripts load without errors in each shell |

```go
func TestOrdersList_JSON(t *testing.T) {
    srv := httptest.NewServer(fakeOrdersAPI(t))
    defer srv.Close()
    var out, errOut bytes.Buffer
    code := cli.Run([]string{"orders", "list", "-o", "json"}, cli.Env{
        Stdout: &out, Stderr: &errOut, Getenv: mapEnv{"SHOPCTL_API_URL": srv.URL, "NO_COLOR": "1"},
    })
    require.Equal(t, 0, code, errOut.String())
    golden.Assert(t, out.String(), "orders_list.json.golden")
}
```

```bash
# bats-core
@test "unknown command suggests a correction" {
  run shopctl oders
  [ "$status" -eq 2 ]
  [[ "$output" == *"Did you mean 'orders'?"* ]]
}
```

---

## 26. Cross-Platform Concerns

| Concern | Guidance |
|---|---|
| Paths | Use path libraries (`filepath`, `pathlib`, `path`); never hard-code `/` or `\`; handle spaces and Unicode in paths |
| Line endings | Write LF by default; accept CRLF on input; `.gitattributes` in repos (42 §6) |
| Shell differences | Don't shell out to `bash`/`grep`/`sed` from a cross-platform tool; implement in-language or detect platform |
| Executables | `.exe` suffix on Windows; `PATHEXT`; executable bit on Unix |
| Console encoding | UTF-8 output; on Windows, modern terminals support UTF-8 and VT sequences (enable virtual terminal processing) |
| Environment variables | Case-insensitive on Windows; document names in UPPER_SNAKE |
| Home and config dirs | OS-appropriate locations (§19.2) |
| File locking and permissions | Windows locks open files; `chmod` semantics differ |
| Signals | Windows has Ctrl+C/Ctrl+Break and console close events rather than full POSIX signals |
| PowerShell users | Provide examples for PowerShell quoting (`'single quotes'`, backtick escapes) as well as bash |

PowerShell and cmd specifics: **25**, **26**.

---

## 27. Distribution and Packaging

| Channel | Ecosystem | Notes |
|---|---|---|
| **GitHub/GitLab Releases** | All | Prebuilt binaries per OS/arch + checksums + signatures + SBOM; **GoReleaser** (Go), cargo-dist (Rust) automate this |
| **Homebrew** (tap or homebrew-core) | macOS, Linux | `brew install shopnow/tap/shopctl` |
| **winget** | Windows | Submit manifests to winget-pkgs (chapter 37, 41(a)) |
| **Scoop** / Chocolatey | Windows | Bucket/package manifests |
| **apt/deb, dnf/rpm** repositories | Linux | Signed repositories (nfpm builds packages) |
| **Snap / Flatpak** | Linux | Sandboxed; less common for dev CLIs |
| **npm** (`npx`, global) | Node.js | Ship bundled JS or platform-specific binary packages via optionalDependencies |
| **PyPI** (`uv tool install`, `pipx install`) | Python | Isolated environments for CLI tools |
| **cargo install** / **go install** | Rust / Go | Developer audiences |
| Container image | Any | `docker run ghcr.io/shopnow/shopctl` for CI usage |
| Internal | Artifactory/Nexus, internal Homebrew tap, MDM-managed installs | Enterprise distribution |

```yaml
# .goreleaser.yaml (excerpt)
builds:
  - main: ./cmd/shopctl
    env: [CGO_ENABLED=0]
    goos: [linux, darwin, windows]
    goarch: [amd64, arm64]
    ldflags: ["-s -w -X main.version={{.Version}} -X main.commit={{.Commit}}"]
archives:
  - formats: [tar.gz]
    format_overrides: [{ goos: windows, formats: [zip] }]
checksum: { name_template: checksums.txt }
sboms: [{ artifacts: archive }]
signs: [{ cmd: cosign, artifacts: checksum, args: ["sign-blob", "--yes", "--output-signature=${signature}", "${artifact}"] }]
brews:
  - repository: { owner: shopnow, name: homebrew-tap }
    homepage: https://docs.shopnow.example/cli
```

> Field names in GoReleaser configs change between major versions — validate with `goreleaser check`.

```text
🔴 Publish SHA-256 checksums and signatures (cosign/Sigstore, GPG for package repos); provenance attestations where possible
🔴 Code-sign Windows binaries and sign + notarise macOS binaries distributed outside package managers (chapter 15 §11)
🟠 Native ARM64 builds (Apple silicon, Windows on ARM, Graviton)
🟠 Installation docs per channel; a single "install" page
```

---

## 28. Versioning, Updates, and Compatibility

```text
🔴 SemVer: flags, subcommands, output schemas (JSON), config keys, and exit codes are the public API
🔴 `tool --version` prints version, commit, build date, (and OS/arch); `tool version -o json` for automation
🟠 Deprecate before removing: warn on stderr for deprecated flags/commands for at least one minor release, with replacement
🟠 Update notifications: check for new versions at most once a day, cache the result, never in CI/non-TTY, opt-out via env/config
🟠 Self-update only when installed via direct download (not when a package manager owns the install); verify signatures
🟠 Server compatibility: send client version to APIs; servers return clear "please upgrade" errors for unsupported versions
```

---

## 29. Shell Completions and Man Pages

```bash
# Generated by the framework (Cobra/clap/Typer/oclif…)
shopctl completion bash > /etc/bash_completion.d/shopctl
shopctl completion zsh  > "${fpath[1]}/_shopctl"
shopctl completion fish > ~/.config/fish/completions/shopctl.fish
shopctl completion powershell | Out-String | Invoke-Expression    # add to $PROFILE for persistence
```

```text
🟠 Package managers install completions automatically (Homebrew, deb/rpm) — include them in packages
🟠 Dynamic completion for resource names (orders, environments) with caching and short timeouts
🟠 Man pages generated from the same command definitions (cobra/doc, clap_mangen, click-man) and shipped in Unix packages
🟠 Web docs generated from command definitions to keep help, man pages, and site in sync
```

---

## 30. Telemetry and Privacy

```text
🔴 Opt-in (or at minimum clearly disclosed opt-out) with a first-run notice; honour DO_NOT_TRACK / tool-specific env vars
🔴 Never collect arguments that may contain secrets, file contents, paths with usernames, or personal data
🔴 Disabled automatically in CI unless explicitly enabled
🟠 Document exactly what is collected and why (command name, version, OS/arch, duration, success/failure)
🟠 Enterprise policy override via config/env
🟠 Comply with privacy law (DPDP/GDPR) for any personal data
```

---

## 31. CLI Security

| Risk | Mitigation |
|---|---|
| Secrets in arguments/history/logs | stdin/prompts/keychains; redaction (§20) |
| Command injection when shelling out | Avoid shells; pass argument arrays; validate inputs |
| Path traversal / unsafe file writes | Canonicalise paths; refuse to overwrite outside intended directories without confirmation |
| Insecure temp files | Use secure temp APIs (`os.CreateTemp`, `tempfile`, `mktemp`) with restrictive permissions |
| TLS bypass | Never disable certificate verification by default; if `--insecure` exists, warn loudly |
| Supply chain | Pinned dependencies, SBOM, signed releases, reproducible builds (chapter 09 §16) |
| Update hijacking | Signed update metadata/artifacts, HTTPS, pinned public keys |
| Config file tampering | Strict permissions on credential/config files (0600) and warnings when too open |
| Plugins/extensions | Explicit install, signature or source verification, documented trust model |
| `curl | sh` installers | Offer package-manager installs and checksum-verified downloads as the recommended path; if a script installer exists, keep it auditable and signed |

---

## 32. Checklists

### Terminal workstation
- [ ] Modern terminal with UTF-8, bracketed paste, readable font, production colour cue
- [ ] Prompt shows Git, exit code, Kubernetes context/namespace, cloud profile
- [ ] Dotfiles in Git (no secrets); bootstrap script per OS
- [ ] Toolchain versions per project (mise/asdf/uv/nvm); direnv without committed secrets
- [ ] SSH: Ed25519/FIDO2 keys with passphrase, ProxyJump, no agent forwarding, verified host keys
- [ ] tmux for long remote work; dry-run habits; production guard functions

### New CLI tool
- [ ] Consistent command structure; standard flags (`-h`, `--version`, `-v`, `-q`, `-o`, `--yes`, `--dry-run`, `--no-color`)
- [ ] Help with examples on every command; typo suggestions; no-arg shows help
- [ ] Data → stdout, logs/progress/errors → stderr; `--json` output with stable schema
- [ ] Documented, stable exit codes; actionable error messages with trace IDs
- [ ] Colour only on TTY and when `NO_COLOR` unset; no meaning by colour alone
- [ ] Never prompts without a TTY; flag equivalents for every prompt; confirmations for destructive actions
- [ ] Config precedence flags > env > project > user > system > defaults; OS-appropriate paths
- [ ] Secrets never as plain flags; OS keychain storage; device/browser login
- [ ] Graceful Ctrl+C/SIGTERM handling; resumable/idempotent operations; timeouts
- [ ] Fast startup measured (hyperfine)
- [ ] Tests: unit, integration with fake HOME, golden help/output, cross-OS CI
- [ ] Releases: multi-OS/arch binaries, checksums, signatures, SBOM; signed/notarised where applicable
- [ ] Package manager channels (Homebrew, winget/Scoop, apt/rpm, npm/PyPI/cargo as relevant)
- [ ] Shell completions and man pages generated; docs site from the same source
- [ ] SemVer with deprecation warnings; update checks opt-out and disabled in CI
- [ ] Telemetry opt-in/disclosed; no sensitive data collected

---

## 33. References

### Terminal and shell
- Command Line Interface Guidelines: https://clig.dev/
- POSIX Utility Syntax Guidelines (The Open Group Base Specifications, Chapter 12): https://pubs.opengroup.org/onlinepubs/9799919799/basedefs/V1_chap12.html
- GNU Coding Standards — Command-Line Interfaces: https://www.gnu.org/prep/standards/html_node/Command_002dLine-Interfaces.html
- NO_COLOR: https://no-color.org/
- XDG Base Directory Specification: https://specifications.freedesktop.org/basedir-spec/latest/
- Windows Terminal: https://learn.microsoft.com/windows/terminal/
- PowerShell: https://learn.microsoft.com/powershell/
- tmux: https://github.com/tmux/tmux/wiki · Starship: https://starship.rs/ · chezmoi: https://www.chezmoi.io/ · mise: https://mise.jdx.dev/ · direnv: https://direnv.net/
- ripgrep: https://github.com/BurntSushi/ripgrep · fd: https://github.com/sharkdp/fd · fzf: https://github.com/junegunn/fzf · bat: https://github.com/sharkdp/bat · jq: https://jqlang.org/ · yq: https://github.com/mikefarah/yq · hyperfine: https://github.com/sharkdp/hyperfine
- OpenSSH manual: https://www.openssh.com/manual.html
- Dev Containers specification: https://containers.dev/

### CLI frameworks and tooling
- Cobra: https://cobra.dev/ · clap: https://docs.rs/clap · Typer: https://typer.tiangolo.com/ · Click: https://click.palletsprojects.com/
- Commander.js: https://github.com/tj/commander.js · oclif: https://oclif.io/ · picocli: https://picocli.info/ · System.CommandLine: https://learn.microsoft.com/dotnet/standard/commandline/ · Spectre.Console: https://spectreconsole.net/
- Bubble Tea: https://github.com/charmbracelet/bubbletea · Ratatui: https://ratatui.rs/ · Textual: https://textual.textualize.io/ · Rich: https://rich.readthedocs.io/
- GoReleaser: https://goreleaser.com/ · cargo-dist: https://opensource.axo.dev/cargo-dist/ · nfpm: https://nfpm.goreleaser.com/
- bats-core: https://bats-core.readthedocs.io/ · Pester: https://pester.dev/
- Homebrew formula/cask docs: https://docs.brew.sh/ · winget manifests: https://learn.microsoft.com/windows/package-manager/package/ · Scoop: https://scoop.sh/
- sysexits.h (BSD exit codes): https://man.freebsd.org/cgi/man.cgi?query=sysexits

---

**Previous:** [28 — Shell Scripting](./28-shell-scripting.md) · **Next:** [30 — IDE & Developer Tooling](./30-ide-and-developer-tooling.md)