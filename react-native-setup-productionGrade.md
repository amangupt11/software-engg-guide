# Pinaak

> Production-oriented React Native mobile application built with the **React Native Community CLI**.

[![React Native](https://img.shields.io/badge/React%20Native-0.86.2-61DAFB?logo=react)](https://reactnative.dev/)
[![React](https://img.shields.io/badge/React-19.2.3-61DAFB?logo=react)](https://react.dev/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript)](https://www.typescriptlang.org/)
[![Android](https://img.shields.io/badge/Android-API%2035-3DDC84?logo=android)](https://developer.android.com/)
[![Platform](https://img.shields.io/badge/Platform-Android%20%7C%20iOS-lightgrey)](https://reactnative.dev/)

---

## Table of Contents

- [1. Project Overview](#1-project-overview)
- [2. Technology Baseline](#2-technology-baseline)
- [3. Engineering Philosophy](#3-engineering-philosophy)
- [4. Prerequisites](#4-prerequisites)
- [5. New Developer Setup](#5-new-developer-setup)
- [6. Verify the Environment](#6-verify-the-environment)
- [7. Android SDK Setup](#7-android-sdk-setup)
- [8. Android Emulator Setup](#8-android-emulator-setup)
- [9. Install Pinaak](#9-install-pinaak)
- [10. Run Pinaak](#10-run-pinaak)
- [11. Validate the Project](#11-validate-the-project)
- [12. Project Structure](#12-project-structure)
- [13. Architecture Rules](#13-architecture-rules)
- [14. Dependency Policy](#14-dependency-policy)
- [15. Development Workflow](#15-development-workflow)
- [16. Git & Pull Requests](#16-git--pull-requests)
- [17. React Native Upgrade Policy](#17-react-native-upgrade-policy)
- [18. Troubleshooting](#18-troubleshooting)
- [19. Security](#19-security)
- [20. Definition of Done](#20-definition-of-done)
- [21. Official References](#21-official-references)

---

# 1. Project Overview

**Pinaak** is a long-lived React Native application intended to be developed with a strong focus on:

- reliability
- maintainability
- reproducible development environments
- conservative dependency management
- clear architecture
- testability
- predictable upgrades
- Android and iOS compatibility

Pinaak uses **React Native Community CLI**.

> **This is not an Expo project.**

## Current status

The project baseline was established on **2026-07-28**.

The initial goal is deliberately simple:

```text
Clean machine
    ↓
Correct development environment
    ↓
Install dependencies
    ↓
Start Android emulator
    ↓
Start Metro
    ↓
Build Pinaak
    ↓
Run Pinaak successfully
    ↓
Only then begin application development
```

---

# 2. Technology Baseline

| Layer | Version / Choice |
|---|---|
| Framework | React Native Community CLI |
| React Native | `0.86.2` |
| React | `19.2.3` |
| Architecture | New Architecture |
| JavaScript Engine | Hermes |
| Language | TypeScript |
| Node.js reference | `24.18.0` |
| npm reference | `11.16.0` |
| Minimum Node declared by project | `>= 22.11.0` |
| Java | JDK `17.0.20` |
| Android | Android 15 / API 35 |
| Android Build Tools | `36.0.0` |
| Android Emulator | Pixel 9 / API 35 / x86_64 |
| Package Manager | npm |
| Development OS reference | Windows 11 x64 |

### Important

The versions in `package.json` and the committed `package-lock.json` are authoritative for JavaScript dependencies.

Do not change the baseline merely because a newer version exists.

---

# 3. Engineering Philosophy

## 3.1 Stable over newest

We prefer:

```text
stable
+ supported
+ compatible
+ maintained
+ tested
```

over:

```text
latest
+ unverified
```

React Native does **not** follow a traditional enterprise "LTS" model. It uses a rolling release/support model with actively supported minor versions.

Therefore, Pinaak should stay on a **stable, actively supported React Native release series** and upgrade deliberately.

## 3.2 Minimal dependencies

Every dependency creates additional:

- maintenance cost
- security surface
- build complexity
- compatibility risk
- upgrade work
- bundle-size impact

Do not add a package just because a tutorial uses it.

## 3.3 Reproducibility

The lockfile is part of the project.

For clean installations:

```powershell
npm ci
```

Do not casually delete `package-lock.json`.

## 3.4 Native compatibility matters

Before adding any native package, verify:

- React Native `0.86.x` compatibility
- New Architecture support
- Android support
- iOS support
- active maintenance
- security history
- license
- build impact

## 3.5 No random upgrades

Do not independently upgrade:

- React
- React Native
- Community CLI
- Metro
- Babel
- Jest
- TypeScript
- Gradle
- Android Gradle Plugin
- Kotlin
- native libraries

without first reviewing compatibility.

---

# 4. Prerequisites

A new developer should install:

- Git
- Node.js
- npm
- JDK 17
- Android Studio
- Android SDK Platform 35
- Android SDK Build-Tools 36.0.0
- Android SDK Platform-Tools
- Android SDK Command-line Tools
- Android Emulator

For iOS development:

- macOS
- Xcode
- CocoaPods
- iOS Simulator

Follow the current official React Native environment guide for platform-specific requirements.

---

# 5. New Developer Setup

## Step 1 — Clone the repository

Replace the repository URL with the actual Pinaak GitHub URL:

```powershell
git clone <REPOSITORY_URL>
cd pinaak
```

Check:

```powershell
git status
```

---

## Step 2 — Check Node

```powershell
node -v
npm -v
```

Reference development machine:

```text
Node: v24.18.0
npm: 11.16.0
```

The project currently declares:

```json
"engines": {
  "node": ">= 22.11.0"
}
```

Do not switch Node versions randomly. If the project baseline changes, update this document and the project configuration together.

---

## Step 3 — Check Java

Run:

```powershell
java --version
javac --version
$env:JAVA_HOME
```

Reference:

```text
Java: 17.0.20 LTS
JAVA_HOME=C:\Program Files\Java\jdk-17.0.20
```

Validate:

```powershell
Test-Path "$env:JAVA_HOME\bin\java.exe"
```

Expected:

```text
True
```

Also inspect:

```powershell
where.exe java
```

The intended JDK should be used first.

---

# 6. Verify the Environment

From the project root:

```powershell
npx react-native doctor
```

Then:

```powershell
npx react-native info
```

Both commands are important when onboarding a new developer.

## Check Android SDK

```powershell
$env:ANDROID_HOME
```

Reference:

```text
C:\Users\<USERNAME>\AppData\Local\Android\Sdk
```

Check ADB:

```powershell
adb --version
```

Check emulator:

```powershell
emulator -version
```

Check available AVDs:

```powershell
emulator -list-avds
```

Check connected devices:

```powershell
adb devices
```

Expected after starting the emulator:

```text
List of devices attached
emulator-5554    device
```

---

# 7. Android SDK Setup

Open Android Studio:

```text
Settings
  ↓
Languages & Frameworks
  ↓
Android SDK
```

## SDK Platforms

Enable:

```text
Android 15
Android SDK Platform 35
```

## SDK Tools

Enable:

```text
Android SDK Build-Tools 36.0.0
Android SDK Command-line Tools (latest)
Android SDK Platform-Tools
Android Emulator
```

The reference environment contains:

```text
Android SDK
├── build-tools
│   └── 36.0.0
├── emulator
├── platform-tools
├── platforms
│   └── android-35
└── ...
```

Do not remove other SDK versions unless you have a specific reason.

---

# 8. Android Emulator Setup

## Recommended development emulator

```text
Device: Pixel 9
Android: 15
API: 35
Architecture: x86_64
```

In Android Studio:

```text
Device Manager
    ↓
Add Device
    ↓
Phone
    ↓
Pixel 9
    ↓
Android 15 / API 35
```

Start the emulator.

Then:

```powershell
adb devices
```

Wait until the device reports:

```text
device
```

Do not build while the emulator is still booting.

---

# 9. Install Pinaak

From:

```text
C:\Projects\pinaak
```

or wherever you cloned the project:

```powershell
npm ci
```

### Why `npm ci`?

`npm ci` installs exactly from the lockfile and is preferred for:

- clean environments
- CI
- reproducible installations
- onboarding

Avoid using:

```powershell
npm install
```

just to "refresh" dependencies.

Use `npm install` only when intentionally changing dependencies.

---

# 10. Run Pinaak

## Terminal 1 — Start Metro

From the project root:

```powershell
npm start
```

Keep this terminal running.

## Terminal 2 — Build Android

Open another PowerShell:

```powershell
cd C:\Projects\pinaak
```

Then:

```powershell
npm run android
```

Expected flow:

```text
React Native
    ↓
Metro
    ↓
Gradle
    ↓
Android build
    ↓
APK installed
    ↓
Pinaak starts on Pixel 9
```

The first native build may take longer because Gradle needs to download and prepare dependencies.

---

# 11. Validate the Project

Run:

```powershell
npm run lint
```

Then:

```powershell
npm test
```

Then:

```powershell
npx react-native doctor
```

Then:

```powershell
npx react-native info
```

Then:

```powershell
npm run android
```

## First-day checklist

- [ ] Repository cloned
- [ ] Node installed
- [ ] npm works
- [ ] JDK 17 installed
- [ ] `JAVA_HOME` configured
- [ ] Android Studio installed
- [ ] Android SDK Platform 35 installed
- [ ] Build Tools 36.0.0 installed
- [ ] Platform-Tools installed
- [ ] Emulator installed
- [ ] Pixel 9 API 35 AVD created
- [ ] Emulator boots
- [ ] `adb devices` reports `device`
- [ ] `npm ci` succeeds
- [ ] `npx react-native doctor` is clean or understood
- [ ] Metro starts
- [ ] Android build succeeds
- [ ] Pinaak launches
- [ ] lint passes
- [ ] tests pass

---

# 12. Project Structure

The initial Community CLI project is intentionally kept close to the official React Native structure.

As the application grows, use:

```text
pinaak/
│
├── android/
├── ios/
├── src/
│   ├── app/
│   ├── features/
│   ├── components/
│   ├── navigation/
│   ├── services/
│   ├── state/
│   ├── hooks/
│   ├── utils/
│   ├── constants/
│   ├── types/
│   └── config/
│
├── __tests__/
│
├── App.tsx
├── package.json
├── package-lock.json
├── tsconfig.json
├── babel.config.js
├── metro.config.js
└── jest.config.js
```

This is the intended architecture as the application grows. Do not create empty folders just to match this diagram.

---

# 13. Architecture Rules

## 13.1 Feature-first organization

Feature-specific code should live inside the feature.

Prefer:

```text
src/features/auth/
├── components/
├── hooks/
├── screens/
├── services/
├── types.ts
└── index.ts
```

over creating giant global directories for every concept.

## 13.2 Shared components

Use:

```text
src/components/
```

only for components genuinely shared by multiple features.

## 13.3 Services

External communication belongs behind service boundaries:

```text
src/services/
├── api/
├── storage/
└── analytics/
```

Do not put API calls directly into reusable UI components.

## 13.4 State

Separate state by purpose:

```text
Local UI state
       ↓
Server/cache state
       ↓
Session/auth state
       ↓
Durable application state
```

Do not put everything into a single global store.

## 13.5 TypeScript

Prefer:

```ts
type User = {
  id: string;
  name: string;
};
```

Avoid:

```ts
const data: any = ...
```

Use explicit types and typed boundaries.

## 13.6 Native code

Platform-specific code belongs in:

```text
android/
ios/
```

Native modules should be isolated behind small TypeScript interfaces where practical.

---

# 14. Dependency Policy

Before installing a dependency, answer:

### Is it actually needed?

Can the requirement be solved using:

- React Native
- platform APIs
- existing project code
- a small utility

If yes, prefer the existing solution.

### Is it maintained?

Check:

- recent releases
- open issues
- maintainer activity
- release cadence
- project health

### Is it compatible?

Check:

```text
React Native 0.86.x
New Architecture
Android
iOS
```

### Does it contain native code?

If yes, evaluate more carefully.

### Security

Check:

- known vulnerabilities
- dependency history
- transitive dependencies

### License

Confirm the license is acceptable for Pinaak.

---

## Dependency installation rule

Do not routinely use:

```powershell
npm install <package>@latest
```

Instead:

```text
Requirement
   ↓
Research package
   ↓
Check maintenance
   ↓
Check RN compatibility
   ↓
Check New Architecture
   ↓
Check Android/iOS
   ↓
Check security/license
   ↓
Install intentionally
   ↓
Test
```

---

# 15. Development Workflow

## Start work

```powershell
git switch main
git pull --ff-only
git switch -c feature/<short-description>
```

Install dependencies if needed:

```powershell
npm ci
```

Start Metro:

```powershell
npm start
```

Run Android:

```powershell
npm run android
```

Run lint:

```powershell
npm run lint
```

Run tests:

```powershell
npm test
```

---

## Useful commands

### Metro

```powershell
npm start
```

Reset Metro cache only when necessary:

```powershell
npm start -- --reset-cache
```

### Android

```powershell
npm run android
```

### Device

```powershell
adb devices
```

### AVDs

```powershell
emulator -list-avds
```

### Diagnostics

```powershell
npx react-native doctor
npx react-native info
```

---

# 16. Git & Pull Requests

## Branch naming

Use:

```text
feature/<name>
fix/<name>
refactor/<name>
chore/<name>
docs/<name>
```

Examples:

```text
feature/login-screen
fix/auth-token-expiry
refactor/api-client
chore/update-tooling
docs/android-setup
```

## Commit messages

Use clear, action-oriented messages:

```text
feat: add login screen
fix: handle expired auth session
refactor: isolate api client
test: add auth service coverage
docs: update Android setup
chore: update supported tooling
```

## Pull request requirements

Every PR should explain:

### What

What changed?

### Why

Why was the change needed?

### How

How was it implemented?

### Testing

For example:

```text
npm run lint
npm test
npm run android
```

For UI changes, include screenshots or a short recording where useful.

## Native changes

If modifying:

```text
android/
ios/
Gradle
Kotlin
Java
Manifest
Info.plist
Podfile
native modules
```

explain why in the PR.

Treat native changes as higher-risk changes.

---

# 17. React Native Upgrade Policy

## Important: React Native is not traditional LTS

React Native uses a rolling support model.

Do not select an old version merely because someone calls it "LTS".

Instead:

```text
Stable
+
Actively supported
+
Compatible
+
Tested
```

is the Pinaak standard.

## Current baseline

```text
React Native: 0.86.2
React: 19.2.3
Community CLI: 20.1.0
```

These were generated together by the React Native Community CLI template.

Treat them as a coordinated baseline.

## Before upgrading React Native

### 1. Read release notes

Review the official release information.

### 2. Check support status

Confirm the target version is supported.

### 3. Check Upgrade Helper

Review native project changes using:

```text
https://react-native-community.github.io/upgrade-helper/
```

### 4. Audit dependencies

Check:

- navigation
- storage
- networking
- native modules
- testing
- build tooling

### 5. Create an upgrade branch

```powershell
git switch -c chore/upgrade-react-native-0.xx
```

### 6. Upgrade intentionally

Do not manually modify dozens of native files from memory or random tutorials.

### 7. Validate

```powershell
npm ci
npm run lint
npm test
npx react-native doctor
npm run android
```

Also test iOS on macOS.

### 8. Test release builds

A debug build passing is not sufficient.

### 9. Update documentation

Update this README whenever the baseline changes.

---

# 18. Troubleshooting

## `emulator` is not recognized

If:

```powershell
emulator -version
```

fails, add:

```text
%LOCALAPPDATA%\Android\Sdk\emulator
```

to PATH.

Open a new PowerShell afterward.

---

## `adb` is not recognized

Add:

```text
%LOCALAPPDATA%\Android\Sdk\platform-tools
```

to PATH.

Then open a new terminal.

---

## `JAVA_HOME` is empty

Run:

```powershell
$env:JAVA_HOME
```

Set it to your JDK 17 installation.

Example:

```text
C:\Program Files\Java\jdk-17.0.20
```

Then verify:

```powershell
Test-Path "$env:JAVA_HOME\bin\java.exe"
```

Expected:

```text
True
```

---

## Wrong Java version

Run:

```powershell
java --version
where.exe java
```

Pinaak's reference setup uses JDK 17.

Do not randomly uninstall Java versions. Fix PATH/JAVA_HOME deliberately.

---

## Android device not detected

Run:

```powershell
adb devices
```

If it reports:

```text
unauthorized
```

unlock the emulator/device and approve the connection if prompted.

If it reports:

```text
offline
```

restart the emulator or ADB.

---

## Android API missing

Check:

```powershell
Get-ChildItem "$env:ANDROID_HOME\platforms"
```

The reference setup requires:

```text
android-35
```

Install Android 15 / API 35 from Android Studio SDK Manager.

---

## Build Tools missing

Check:

```powershell
Get-ChildItem "$env:ANDROID_HOME\build-tools"
```

Reference:

```text
36.0.0
```

---

## Metro problem

Try:

```powershell
npm start -- --reset-cache
```

Only do this when stale Metro state is a plausible cause.

---

## Gradle problem

First read the actual error.

Do not immediately delete caches.

If the error clearly points to stale native build state:

```powershell
cd android
.\gradlew clean
cd ..
npm run android
```

---

## Dependency caused the failure

If the build worked before installing a dependency and fails afterward:

1. inspect the dependency
2. verify RN 0.86 compatibility
3. verify New Architecture support
4. check Android/iOS requirements
5. inspect Gradle/CocoaPods errors
6. consider reverting the dependency change

Do **not** disable New Architecture as the first workaround.

---

## Emulator is slow

Check:

- CPU usage
- available RAM
- hardware virtualization
- hypervisor configuration
- graphics configuration
- emulator architecture
- background applications

Do not blindly disable Hyper-V because an old tutorial recommends it.

---

## "Works on my machine"

Compare:

```powershell
node -v
npm -v
java --version
$env:JAVA_HOME
$env:ANDROID_HOME
adb --version
npx react-native info
```

Also compare:

```text
package.json
package-lock.json
Android SDK
JDK
Node
native build tooling
```

---

# 19. Security

Never commit:

```text
.env
.env.*
API keys
access tokens
refresh tokens
passwords
private keys
keystores
signing certificates
service-account JSON
production credentials
```

## Mobile security rule

Never assume a secret embedded inside a mobile application is private.

Anything shipped to a client may potentially be extracted.

Sensitive operations should be protected by a trusted backend when appropriate.

## Dependency security

Before adding a dependency, check:

- maintenance activity
- known vulnerabilities
- transitive dependencies
- native code
- license
- React Native compatibility

## Security vulnerability reporting

**Do not open a public GitHub issue for a security vulnerability.**

Use GitHub's private vulnerability reporting / Security Advisories when configured for the repository.

---

# 20. Definition of Done

A change is complete when applicable:

- [ ] implementation is complete
- [ ] TypeScript is valid
- [ ] lint passes
- [ ] tests pass
- [ ] Android build passes
- [ ] iOS build passes
- [ ] no secrets are committed
- [ ] dependency changes are justified
- [ ] native changes are documented
- [ ] documentation is updated when behavior/tooling changes
- [ ] PR is reviewable

---

# 21. Official References

Always prefer official documentation over random tutorials.

### React Native

- React Native Documentation  
  https://reactnative.dev/docs/getting-started

- Environment Setup  
  https://reactnative.dev/docs/set-up-your-environment

- React Native Versions  
  https://reactnative.dev/versions

- Releases  
  https://reactnative.dev/releases/

- Versioning Policy  
  https://reactnative.dev/releases/versioning-policy

- Branches  
  https://reactnative.dev/releases/branches

- React Native 0.86  
  https://reactnative.dev/blog/2026/06/11/react-native-0.86

### Upgrade

- React Native Upgrade Helper  
  https://react-native-community.github.io/upgrade-helper/

### React

- React Documentation  
  https://react.dev/

### Node.js

- Node.js  
  https://nodejs.org/

### Android

- Android Studio  
  https://developer.android.com/studio

- Android Developers  
  https://developer.android.com/

- Android Emulator  
  https://developer.android.com/studio/run/emulator

### TypeScript

- TypeScript  
  https://www.typescriptlang.org/

---

# Pinaak Baseline Summary

```text
┌───────────────────────────────────────────────┐
│                  PINAAK                       │
├───────────────────────────────────────────────┤
│ React Native Community CLI                    │
│ React Native             0.86.2              │
│ React                    19.2.3              │
│ Architecture             New Architecture    │
│ JavaScript Engine        Hermes              │
│ Language                 TypeScript          │
│ Node.js                  24.18.0 reference   │
│ npm                      11.16.0 reference   │
│ Java                     JDK 17.0.20         │
│ Android                  API 35              │
│ Build Tools              36.0.0              │
│ Emulator                 Pixel 9 / API 35   │
│ Package Manager          npm                 │
└───────────────────────────────────────────────┘
```

## Golden Rule

> **Do not change the technology baseline without a reason, research, testing, and documentation.**

Pinaak is built for the long term.

**Stable. Maintainable. Reproducible. Deliberate.**
