# 🚀 Git & GitHub Professional Setup Guide

> A complete guide for configuring a new development machine for professional software development.

---

## 📚 Table of Contents

- [Prerequisites](#-prerequisites)
- [Install Git](#-install-git)
- [Configure Git](#-configure-git)
- [SSH Setup](#-ssh-setup)
- [GitHub Authentication](#-github-authentication)
- [Git Configuration](#-git-configuration)
- [GitHub CLI](#-github-cli)
- [Developer Tools](#-developer-tools)
- [VS Code Extensions](#-recommended-vs-code-extensions)
- [Folder Structure](#-recommended-folder-structure)
- [Create Your First Repository](#-create-your-first-repository)
- [Daily Workflow](#-daily-git-workflow)
- [Backup Configuration](#-backup-your-configuration)
- [Best Practices](#-professional-best-practices)
- [Final Checklist](#-final-checklist)

---

# 📋 Prerequisites

- Windows 11 / macOS / Linux
- Administrator access
- GitHub account
- Internet connection

---

# 📦 Install Git

## Windows

Download the latest version:

https://git-scm.com/download/win

## macOS

```bash
brew install git
```

## Ubuntu

```bash
sudo apt update
sudo apt install git
```

### Verify

```bash
git --version
```

Expected:

```text
git version 2.50.1
```

---

# ⚙️ Configure Git

## Configure Identity

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

Verify:

```bash
git config --global --list
```

---

## Default Branch

```bash
git config --global init.defaultBranch main
```

---

## Colored Output

```bash
git config --global color.ui auto
```

---

## Line Endings

### Windows

```bash
git config --global core.autocrlf true
```

### macOS / Linux

```bash
git config --global core.autocrlf input
```

---

## VS Code as Default Editor

```bash
git config --global core.editor "code --wait"
```

---

## VS Code Merge Tool

```bash
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd "code --wait $MERGED"
```

---

# 🔐 SSH Setup

## Generate SSH Key

```bash
ssh-keygen -t ed25519 -C "you@example.com"
```

Generated files:

```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

---

## Start SSH Agent

### Windows

```powershell
Get-Service ssh-agent | Set-Service -StartupType Automatic

Start-Service ssh-agent

ssh-add $env:USERPROFILE\.ssh\id_ed25519
```

### macOS / Linux

```bash
eval "$(ssh-agent -s)"

ssh-add ~/.ssh/id_ed25519
```

---

## Copy Public Key

Windows

```powershell
Get-Content ~/.ssh/id_ed25519.pub
```

Linux/macOS

```bash
cat ~/.ssh/id_ed25519.pub
```

---

# 🐙 GitHub Authentication

## Add SSH Key

1. GitHub → Settings
2. SSH and GPG Keys
3. New SSH Key
4. Paste Key
5. Save

---

## Test Connection

```bash
ssh -T git@github.com
```

Expected

```text
Hi username!
You've successfully authenticated.
```

---

## Clone Using SSH

```bash
git clone git@github.com:username/repository.git
```

---

# ⚡ Git Configuration

## Credential Manager

```bash
git config --global credential.helper manager
```

---

## Global Git Ignore

Create

```text
~/.gitignore_global
```

Example

```gitignore
.DS_Store
Thumbs.db
*.log
node_modules/
.env
coverage/
dist/
build/
```

Configure

```bash
git config --global core.excludesfile ~/.gitignore_global
```

---

## Useful Aliases

```bash
git config --global alias.st status
git config --global alias.br branch
git config --global alias.co checkout
git config --global alias.ci commit
git config --global alias.last "log -1 HEAD"
git config --global alias.lg "log --oneline --graph --decorate --all"
git config --global alias.undo "reset HEAD~1"
```

---

## Other Recommended Settings

```bash
git config --global pull.rebase true
git config --global fetch.prune true
git config --global push.default simple
```

---

# 💻 GitHub CLI

## Install

### Windows

```powershell
winget install GitHub.cli
```

### macOS

```bash
brew install gh
```

### Ubuntu

```bash
sudo apt install gh
```

---

## Login

```bash
gh auth login
```

Verify

```bash
gh auth status
```

---

# 🛠 Developer Tools

- Visual Studio Code
- Docker Desktop
- Node.js (LTS)
- Python
- Java
- Postman
- Insomnia
- Windows Terminal
- PowerShell 7
- WSL2
- Fira Code
- JetBrains Mono

---

# 🧩 Recommended VS Code Extensions

## Git

- GitLens
- GitHub Pull Requests
- Git History
- GitHub Actions

## JavaScript / React

- ESLint
- Prettier
- npm Intellisense
- ES7 React Snippets
- React Native Tools

## Web

- Live Server
- Live Preview
- Auto Rename Tag
- Auto Close Tag
- Path Intellisense

## CSS

- Tailwind CSS IntelliSense

## Markdown

- Markdown All in One

## Utilities

- Error Lens
- Better Comments
- Bookmarks
- Peacock
- Material Icon Theme

---

# 📁 Recommended Folder Structure

## Windows

```text
C:\
└── Dev
    ├── Personal
    ├── Work
    ├── OpenSource
    └── Learning
```

## macOS / Linux

```text
~/Developer
├── Personal
├── Work
├── OpenSource
└── Learning
```

---

# 🚀 Create Your First Repository

```bash
mkdir my-project

cd my-project

git init

echo "# My Project" > README.md

git add .

git commit -m "Initial commit"

git remote add origin git@github.com:username/my-project.git

git push -u origin main
```

---

# 🔄 Daily Git Workflow

```bash
git pull

git checkout -b feature/new-feature

# Make changes

git add .

git commit -m "Add new feature"

git push origin feature/new-feature
```

After merging:

```bash
git checkout main

git pull

git branch -d feature/new-feature
```

---

# 💾 Backup Your Configuration

```text
~/.gitconfig
~/.gitignore_global
~/.ssh/
```

Export settings:

```bash
git config --global --list
```

---

# ✅ Professional Best Practices

- Use SSH instead of HTTPS.
- Enable GitHub 2FA.
- Use Pull Requests.
- Never commit secrets.
- Write meaningful commit messages.
- Use feature branches.
- Keep Git updated.
- Review changes before pushing.
- Backup your SSH keys.

---

# ✅ Final Checklist

| Task                     | Status |
| ------------------------ | :----: |
| Git Installed            |   ☐   |
| Git Configured           |   ☐   |
| SSH Configured           |   ☐   |
| GitHub CLI Installed     |   ☐   |
| VS Code Installed        |   ☐   |
| Extensions Installed     |   ☐   |
| First Repository Created |   ☐   |
| Backup Completed         |   ☐   |

---

# 🎉 Congratulations!

Your development machine is now configured with a **professional Git & GitHub workflow** that's suitable for personal projects, 77383 82822 open source, and enterprise software development.
