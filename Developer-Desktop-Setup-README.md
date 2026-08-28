# Developer Desktop Setup Guide

A complete Windows development environment setup checklist.

Purpose:
- React Native development
- Web development
- Backend development
- Cloud development
- Database development
- Software engineering workstation setup

Last Updated:
2026

---

# 1. Hardware Recommendation

Recommended minimum:

CPU:
- Intel Core i7 / i9
- AMD Ryzen 7 / Ryzen 9

RAM:
- Minimum: 16GB
- Recommended: 32GB

Storage:
- NVMe SSD
- Minimum 512GB
- Recommended 1TB+

GPU:
- Optional
- NVIDIA GPU recommended for AI/ML workloads

---

# 2. Operating System

Install:

## Windows 11 Pro 64-bit

Enable:

- Hyper-V
- Virtual Machine Platform
- Windows Subsystem for Linux (WSL2)

Check Windows version:

```powershell
winver
```

---

# 3. Windows Updates

Run:

Settings

→ Windows Update

Install all updates.

Restart.

---

# 4. Install Windows Terminal

Install:

```powershell
winget install Microsoft.WindowsTerminal
```

Recommended shells:

- PowerShell 7
- WSL Ubuntu
- Git Bash

---

# 5. Install PowerShell 7 LTS

Install:

```powershell
winget install Microsoft.PowerShell
```

Verify:

```powershell
pwsh --version
```

---

# 6. Configure PowerShell Execution Policy

Check:

```powershell
Get-ExecutionPolicy -List
```

Recommended:

```
CurrentUser RemoteSigned
LocalMachine RemoteSigned
```

Set:

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

---

# 7. Install Git

Install:

```powershell
winget install Git.Git
```

Verify:

```bash
git --version
```

Configure:

```bash
git config --global user.name "Your Name"

git config --global user.email "your@email.com"
```

---

# 8. Install Visual Studio Code

Install:

```powershell
winget install Microsoft.VisualStudioCode
```

Recommended Extensions:

## General

- Prettier
- ESLint
- GitLens

## React Native

- React Native Tools
- ES7 React Snippets

## Backend

- Docker
- REST Client

---

# 9. Install Node.js Environment

## Install nvm-windows

Download:

https://github.com/coreybutler/nvm-windows/releases


Verify:

```powershell
nvm version
```

---

# Node Versions

Recommended:

| Version | Purpose |
|-|-|
| Node 22 LTS | Main development |
| Node 20 LTS | Legacy projects |
| Node 24 | Testing |

Install:

```powershell
nvm install 22
```

Use:

```powershell
nvm use 22
```

Verify:

```powershell
node -v

npm -v
```

---

# 10. Install Java

## React Native Android Development

Recommended:

Java:

```
JDK 17 LTS
```

Install:

Oracle JDK 17
or
Eclipse Temurin JDK 17


Verify:

```powershell
java -version

javac -version
```

Expected:

```
java version "17.x.x"
```

---

# JAVA_HOME

Set:

```
JAVA_HOME=C:\Program Files\Java\jdk-17
```

Verify:

```powershell
echo $env:JAVA_HOME
```

---

# 11. Install Android Studio

Install:

Android Studio Latest Stable


Install components:

- Android SDK
- Android SDK Platform Tools
- Android Emulator
- Android Virtual Device


Recommended:

Android API:

```
35+
```


Set:

```
ANDROID_HOME
```

Example:

```
C:\Users\<username>\AppData\Local\Android\Sdk
```

---

# 12. React Native Setup

Recommended Stack:

```
React Native Latest Stable
Node.js 22 LTS
JDK 17 LTS
Android Studio Latest
```

Install CLI:

```bash
npm install -g react-native-cli
```

Create project:

```bash
npx react-native@latest init MyApp
```

Run:

```bash
npx react-native run-android
```

Check:

```bash
npx react-native doctor
```

---

# 13. Install Python

Recommended:

Python 3.12


Install:

```powershell
winget install Python.Python.3.12
```

Verify:

```powershell
python --version
```

Used for:

- Automation
- AI tools
- Scripts
- Backend

---

# 14. Install Docker Desktop

Install:

```powershell
winget install Docker.DockerDesktop
```

Enable:

- WSL2 backend


Verify:

```powershell
docker --version
```

---

# 15. Database Setup

## PostgreSQL

Install:

PostgreSQL 17


## MySQL

Install:

MySQL 8.4 LTS


## MongoDB

Install:

MongoDB 8


Database GUI:

- DBeaver
- MongoDB Compass

---

# 16. API Testing Tools

Install:

Postman


Verify:

API testing works.

Alternative:

Insomnia

---

# 17. Cloud Tools

## AWS CLI

Install:

```powershell
winget install Amazon.AWSCLI
```


## Azure CLI

Install:

```powershell
winget install Microsoft.AzureCLI
```


## Google Cloud CLI

Install:

Google Cloud SDK

---

# 18. WSL2 Setup

Install:

```powershell
wsl --install
```

Recommended Linux:

Ubuntu 24.04 LTS


Verify:

```powershell
wsl --version
```

---

# 19. Useful Developer Applications

Install:

## Browser

- Google Chrome
- Microsoft Edge
- Firefox Developer Edition


## Utilities

- 7-Zip
- Everything Search
- Notepad++
- ShareX


## Communication

- Teams
- Slack
- Discord

---

# 20. Environment Verification

Run:

## Git

```powershell
git --version
```


## Node

```powershell
node -v
npm -v
```


## Java

```powershell
java -version
```


## Python

```powershell
python --version
```


## Docker

```powershell
docker --version
```


## Android

```powershell
adb version
```

---

# Final Development Stack

```
Windows 11 Pro
|
├── PowerShell 7 LTS
├── Windows Terminal
├── WSL2 Ubuntu 24.04 LTS
|
├── VS Code
├── Git
|
├── Node.js 22 LTS
├── nvm-windows
|
├── React Native Latest Stable
├── Android Studio
├── Android SDK
├── JDK 17 LTS
|
├── Python 3.12
|
├── Docker Desktop
|
├── PostgreSQL 17
├── MySQL 8.4 LTS
├── MongoDB
|
├── Postman
├── AWS CLI
├── Azure CLI
```

---

# First Day Checklist

[ ] Windows updates completed

[ ] Install drivers

[ ] Install Terminal

[ ] Install PowerShell 7

[ ] Install Git

[ ] Install VS Code

[ ] Install nvm-windows

[ ] Install Node LTS

[ ] Install JDK 17

[ ] Install Android Studio

[ ] Configure Android SDK

[ ] Install Docker

[ ] Install databases

[ ] Configure GitHub SSH

[ ] Clone projects

[ ] Run test build


END
