# 🖥️ Desktop Engineering — Production Engineering Guide

> How to build, sign, distribute, update, and operate desktop applications for Windows, macOS, and Linux: platform landscape, choosing a technology (native WinUI/WPF, SwiftUI/AppKit, Electron, Tauri, .NET MAUI/Avalonia, Qt, Flutter, Compose Multiplatform), application architecture and process models, Electron and Tauri security, local data and secure storage, OS integration, packaging and installers, **code signing and notarisation** (including 2026 certificate rules and Azure Artifact Signing), distribution channels, auto-update, performance, accessibility, testing, CI/CD, crash reporting, enterprise deployment, and licensing.
>
> Related: [05 Design](./05-software-design.md) · [06 Coding standards](./06-coding-standards.md) · [09 Security](./09-security-engineering.md) · [12 Frontend (for web-tech desktop apps)](./12-frontend-engineering.md) · Workstation setup: [38](./38-developer-desktop-software-tool-stack.md), [41(a) Windows 11](./41-(a)Windows-11-Fresh-Setup.md), [41(b) macOS](./41-(b)macOS-Tahoe-Fresh-Setup.md) · Package managers: [37](./37-package-managers.md)

---

## 📚 Table of Contents

- [🖥️ Desktop Engineering — Production Engineering Guide](#️-desktop-engineering--production-engineering-guide)
  - [📚 Table of Contents](#-table-of-contents)
  - [1. Principles](#1-principles)
  - [2. Platform Landscape](#2-platform-landscape)
    - [Support policy template](#support-policy-template)
  - [3. Choosing a Technology](#3-choosing-a-technology)
    - [Decision guide](#decision-guide)
  - [4. Application Architecture](#4-application-architecture)
    - [4.1 Layering (all technologies)](#41-layering-all-technologies)
    - [4.2 Process models for web-tech desktop apps](#42-process-models-for-web-tech-desktop-apps)
  - [5. Electron Engineering and Security](#5-electron-engineering-and-security)
    - [5.1 Electron security checklist (from Electron's official security guidance)](#51-electron-security-checklist-from-electrons-official-security-guidance)
    - [5.2 Electron tooling](#52-electron-tooling)
  - [6. Tauri Engineering and Security](#6-tauri-engineering-and-security)
  - [7. Native and .NET Desktop Notes](#7-native-and-net-desktop-notes)
  - [8. Local Data and Secure Storage](#8-local-data-and-secure-storage)
    - [8.1 Standard locations](#81-standard-locations)
    - [8.2 Secure storage](#82-secure-storage)
  - [9. OS Integration](#9-os-integration)
  - [10. Packaging and Installers](#10-packaging-and-installers)
    - [10.1 Formats](#101-formats)
    - [10.2 Installer rules](#102-installer-rules)
  - [11. Code Signing and Notarisation](#11-code-signing-and-notarisation)
    - [11.1 Windows (Authenticode)](#111-windows-authenticode)
    - [11.2 macOS (Developer ID + notarisation)](#112-macos-developer-id--notarisation)
    - [11.3 Linux](#113-linux)
    - [11.4 Signing rules](#114-signing-rules)
  - [12. Distribution Channels](#12-distribution-channels)
  - [13. Auto-Update](#13-auto-update)
    - [13.1 Options](#131-options)
    - [13.2 Update rules](#132-update-rules)
  - [14. Performance](#14-performance)
  - [15. UX Conventions per Platform](#15-ux-conventions-per-platform)
  - [16. Accessibility](#16-accessibility)
  - [17. Localisation](#17-localisation)
  - [18. Offline, Sync, and Networking](#18-offline-sync-and-networking)
  - [19. Testing](#19-testing)
  - [20. CI/CD for Desktop](#20-cicd-for-desktop)
  - [21. Crash Reporting and Telemetry](#21-crash-reporting-and-telemetry)
  - [22. Enterprise Deployment](#22-enterprise-deployment)
  - [23. Licensing and Activation](#23-licensing-and-activation)
  - [24. Reference Snippets by Stack](#24-reference-snippets-by-stack)
    - [24.1 WPF / WinUI with CommunityToolkit.Mvvm (C#)](#241-wpf--winui-with-communitytoolkitmvvm-c)
    - [24.2 macOS SwiftUI document-style app (Swift)](#242-macos-swiftui-document-style-app-swift)
    - [24.3 Qt 6 / QML (C++ backend object)](#243-qt-6--qml-c-backend-object)
    - [24.4 Electron renderer using the preload API (TypeScript)](#244-electron-renderer-using-the-preload-api-typescript)
    - [24.5 Tauri frontend invoking a command (TypeScript)](#245-tauri-frontend-invoking-a-command-typescript)
  - [25. Scenario Playbooks](#25-scenario-playbooks)
    - [25.1 Internal Windows LOB app for a bank branch network](#251-internal-windows-lob-app-for-a-bank-branch-network)
    - [25.2 Cross-platform SaaS companion app (Windows, macOS, Linux)](#252-cross-platform-saas-companion-app-windows-macos-linux)
    - [25.3 Developer tool distributed to engineers](#253-developer-tool-distributed-to-engineers)
    - [25.4 Point-of-sale / kiosk app](#254-point-of-sale--kiosk-app)
    - [25.5 Migrating a legacy WinForms/WPF app](#255-migrating-a-legacy-winformswpf-app)
  - [26. Checklists](#26-checklists)
    - [Technology \& architecture](#technology--architecture)
    - [Build, sign, ship](#build-sign-ship)
    - [Quality \& operations](#quality--operations)
  - [27. References](#27-references)
    - [Platforms](#platforms)
    - [Frameworks](#frameworks)
    - [Distribution](#distribution)
    - [Security \& accessibility](#security--accessibility)

---

## 1. Principles

| # | Principle |
|---:|---|
| 1 | **Unsigned software is broken software** — users see warnings, OSes block it, enterprises refuse it. Signing is part of the build. |
| 2 | **Updates are a security feature** — a signed, reliable auto-update channel is mandatory for anything connected to the internet. |
| 3 | **Respect the platform** — menus, shortcuts, file locations, dark mode, accessibility APIs, and installation conventions differ per OS. |
| 4 | **The user's machine is not your server** — anything on it can be inspected or modified; enforce licences and security server-side where it matters. |
| 5 | **Least privilege** — no admin rights unless strictly needed; per-user installs by default. |
| 6 | **Startup time and memory matter** — desktop users run many apps at once. |
| 7 | **Support versions deliberately** — publish supported OS versions and an end-of-support policy. |

---

## 2. Platform Landscape

| Platform | Status notes (verify before planning) |
|---|---|
| **Windows 11** | Primary Windows target. **Windows 10 reached end of support on 14 Oct 2025** (Extended Security Updates available for some customers) — decide whether to keep supporting it based on your users. ARM64 Windows devices are increasingly common — ship native ARM64 builds where possible. |
| **macOS** | Annual releases; apps must run on Apple silicon (arm64) and, if supporting older Macs, Intel (universal binaries). Apple's toolchain requirements for Mac App Store uploads follow the same yearly Xcode/SDK baseline as iOS (chapter 14 §2.2). |
| **Linux** | Fragmented: Ubuntu/Debian, Fedora/RHEL, Arch, etc.; X11 and **Wayland** (Wayland is the default session on major distributions); universal formats (Flatpak, Snap, AppImage) reduce fragmentation. |

### Support policy template

```text
Windows: Windows 11 (x64, ARM64); Windows 10 22H2 until <date> (best effort)
macOS:   current + two previous major versions; Apple silicon + Intel (universal) until <date>
Linux:   Ubuntu LTS (current, previous), Fedora (current); Flatpak build for others
Review annually; announce drops ≥ 3 months ahead in-app.
```

---

## 3. Choosing a Technology

| Technology | Language / UI | Platforms | Strengths | Trade-offs |
|---|---|---|---|---|
| **WinUI 3 / Windows App SDK** | C#, C++ / XAML | Windows | Modern native Windows UI, Fluent design | Windows-only |
| **WPF** | C# / XAML | Windows | Mature, huge ecosystem, enterprise | Windows-only; older UI stack (still supported) |
| **WinForms** | C# | Windows | Simple LOB apps, legacy | Dated UI |
| **SwiftUI / AppKit** | Swift | macOS | Best Mac integration | macOS-only |
| **Electron** | JS/TS + HTML/CSS (Chromium + Node.js) | Win/macOS/Linux | Web skills, consistent rendering, vast ecosystem | Large bundles (~100 MB+), memory use, must track Chromium security updates |
| **Tauri 2** | Rust core + web UI in OS WebView | Win/macOS/Linux (+ mobile) | Small bundles, lower memory, capability-based security | WebView differences per OS (WebView2/WKWebView/WebKitGTK), Rust for native code |
| **.NET MAUI** | C# / XAML | Windows, macOS (Mac Catalyst), mobile | Shared .NET codebase incl. mobile | Desktop polish varies by platform |
| **Avalonia** | C# / XAML | Win/macOS/Linux | WPF-like, truly cross-platform, consistent rendering | Smaller ecosystem than WPF |
| **Uno Platform** | C# / WinUI XAML | Win/macOS/Linux/web/mobile | WinUI API surface everywhere | Complexity |
| **Qt 6** | C++ / QML (Python via PySide6) | Win/macOS/Linux/embedded | Performance, native look, long track record | Licensing (LGPL/GPL/commercial) considerations; C++ complexity |
| **Flutter desktop** | Dart | Win/macOS/Linux | Shared UI with mobile | Desktop ecosystem smaller |
| **Compose Multiplatform (desktop)** | Kotlin | Win/macOS/Linux | Shared Kotlin UI with Android | JVM runtime size; maturing |
| **JavaFX** | Java/Kotlin | Win/macOS/Linux | JVM shops | Smaller community |
| **PWA (installable web app)** | Web | Any modern browser | No installer, instant updates | Limited OS integration |

### Decision guide

```text
Windows-only enterprise LOB app?                      → WPF or WinUI 3 (.NET LTS)
Mac-first, premium UX?                                → SwiftUI/AppKit
Cross-platform, existing web team, rich UI?           → Electron (proven) or Tauri 2 (smaller, more security-constrained)
Cross-platform, C# team?                              → Avalonia (or MAUI if mobile sharing is key)
Cross-platform, performance-critical / embedded?      → Qt 6
Sharing UI with mobile apps?                          → Flutter or Compose Multiplatform
Simple tool that could be a web app?                  → PWA first
```

Electron vs Tauri in one line: Electron trades disk/memory footprint for UI consistency and ecosystem depth; Tauri trades rendering consistency (per-OS WebViews) for much smaller binaries and a stricter security model.

---

## 4. Application Architecture

### 4.1 Layering (all technologies)

```text
UI (views, view models / state)  →  Application services (use cases)  →  Domain
                                          ↓
                         Infrastructure: local DB, file system, OS APIs, network clients, updater
```

- **MVVM** is the norm for XAML stacks (CommunityToolkit.Mvvm, ReactiveUI) and maps well to SwiftUI/Compose.
- Keep OS-specific code behind interfaces (file dialogs, notifications, secure storage) for testability and portability.

### 4.2 Process models for web-tech desktop apps

```text
ELECTRON
┌───────────────────────────┐     IPC (contextBridge-exposed, validated)     ┌────────────────────────┐
│ Main process (Node.js)    │◄──────────────────────────────────────────────►│ Renderer (Chromium)     │
│ app lifecycle, windows,   │                                                 │ UI only, sandboxed,     │
│ OS APIs, file system,     │       Preload script (isolated world)           │ no Node.js access       │
│ updater, native modules   │◄── exposes a minimal, typed API ──────────────►│                         │
└───────────────────────────┘                                                 └────────────────────────┘

TAURI
┌───────────────────────────┐   commands/events (permission-checked via capabilities)   ┌─────────────────┐
│ Rust core                 │◄────────────────────────────────────────────────────────────►│ OS WebView UI    │
│ commands, plugins, state  │                                                              │ (WebView2 /      │
└───────────────────────────┘                                                              │ WKWebView /      │
                                                                                           │ WebKitGTK)       │
                                                                                           └─────────────────┘
```

Treat the renderer/webview like an **untrusted browser tab**: it must not have direct OS or file-system power; privileged operations live in the main process/core and are exposed through narrow, validated interfaces.

---

## 5. Electron Engineering and Security

### 5.1 Electron security checklist (from Electron's official security guidance)

| # | Control | Status |
|---:|---|:---:|
| 1 | Only load secure content (HTTPS; local bundled files) | 🔴 |
| 2 | `nodeIntegration: false` in renderers (default) | 🔴 |
| 3 | `contextIsolation: true` (default) | 🔴 |
| 4 | Enable process **sandbox** for renderers (default in recent versions) | 🔴 |
| 5 | Use `contextBridge` to expose a minimal API from preload — never expose `ipcRenderer` or Node APIs wholesale | 🔴 |
| 6 | Validate the **sender** of every IPC message and validate all IPC arguments | 🔴 |
| 7 | Define a strict **Content Security Policy** | 🔴 |
| 8 | Don't disable `webSecurity`; don't enable `allowRunningInsecureContent` or experimental features | 🔴 |
| 9 | Restrict navigation (`will-navigate`) and new windows (`setWindowOpenHandler`) to allow-listed origins | 🔴 |
| 10 | Don't use `shell.openExternal` with untrusted URLs (validate scheme/host) | 🔴 |
| 11 | Handle permission requests (`setPermissionRequestHandler`) — deny by default | 🔴 |
| 12 | Avoid `<webview>`; if used, verify options and attributes | 🟠 |
| 13 | Use **Electron Fuses** to disable unneeded capabilities in production builds (e.g. `RunAsNode`, Node CLI inspect args) and enable ASAR integrity validation where supported | 🟠 |
| 14 | Keep **Electron up to date** — each major bundles a Chromium version; only recent majors receive security fixes | 🔴 |
| 15 | Custom protocols instead of `file://` for loading local content | 🟠 |

```ts
// main.ts (illustrative)
import { app, BrowserWindow, ipcMain, shell } from 'electron';
import path from 'node:path';

function createWindow() {
  const win = new BrowserWindow({
    width: 1200, height: 800,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js'),
      contextIsolation: true,
      sandbox: true,
      nodeIntegration: false,
    },
  });
  win.webContents.setWindowOpenHandler(({ url }) => {
    if (new URL(url).origin === 'https://help.shopnow.example') shell.openExternal(url);
    return { action: 'deny' };
  });
  win.webContents.on('will-navigate', (e) => e.preventDefault());
  win.loadFile(path.join(__dirname, 'renderer/index.html'));
}

ipcMain.handle('orders:export', async (event, rawRange: unknown) => {
  if (!isTrustedSender(event.senderFrame)) throw new Error('Untrusted IPC sender');
  const range = ExportRangeSchema.parse(rawRange);          // validate arguments (zod)
  return exportOrders(range);
});

app.whenReady().then(createWindow);
```

```ts
// preload.ts — minimal, typed surface
import { contextBridge, ipcRenderer } from 'electron';
contextBridge.exposeInMainWorld('api', {
  exportOrders: (range: { from: string; to: string }) => ipcRenderer.invoke('orders:export', range),
});
```

### 5.2 Electron tooling

| Need | Tools |
|---|---|
| Scaffolding, packaging, makers, publishing | **Electron Forge** (officially maintained) |
| Packaging + auto-update alternative | **electron-builder** (+ electron-updater) |
| Windows signing | `@electron/windows-sign` (used by Forge/packager), electron-builder signing options incl. Azure Artifact Signing |
| macOS signing/notarisation | `@electron/osx-sign`, `@electron/notarize` |
| Testing | Playwright (Electron support), unit tests with Vitest/Jest |

---

## 6. Tauri Engineering and Security

| Control | Practice |
|---|---|
| **Capabilities & permissions** | Tauri 2 uses capability files to grant specific commands/plugins to specific windows/webviews — grant the minimum |
| Commands | Validate all inputs in Rust command handlers; return typed errors |
| Scopes | Restrict file-system and shell plugin scopes to explicit paths/commands |
| CSP | Configure a strict CSP in `tauri.conf.json` |
| Isolation pattern | Optional isolation application that intercepts IPC messages for validation |
| Updater | Built-in updater requires signed update artifacts (public key embedded in the app) |
| WebView differences | Test on WebView2 (Windows), WKWebView (macOS), WebKitGTK (Linux) — feature support differs |

```json
// src-tauri/capabilities/main.json (illustrative)
{
  "identifier": "main-window",
  "windows": ["main"],
  "permissions": [
    "core:default",
    "dialog:allow-save",
    { "identifier": "fs:allow-write-text-file", "allow": [{ "path": "$DOWNLOAD/*" }] }
  ]
}
```

```rust
#[tauri::command]
async fn export_orders(from: String, to: String, state: tauri::State<'_, AppState>) -> Result<String, AppError> {
    let range = ExportRange::parse(&from, &to)?;   // validate input
    state.orders.export(range).await
}
```

---

## 7. Native and .NET Desktop Notes

| Stack | Guidance |
|---|---|
| **.NET (WPF, WinUI 3, Avalonia, MAUI)** | Target the current .NET LTS; MVVM with CommunityToolkit.Mvvm; `Microsoft.Extensions.Hosting` for DI/config/logging in desktop apps; async I/O off the UI thread (`Dispatcher`/`DispatcherQueue` for UI updates); trimming/ReadyToRun/Native AOT (where supported) for startup and size |
| **WinUI 3 / Windows App SDK** | Packaged (MSIX) or unpackaged deployment; keep the Windows App SDK runtime updated |
| **SwiftUI / AppKit (macOS)** | App Sandbox for Mac App Store; hardened runtime for Developer ID; Swift concurrency with `@MainActor` for UI; `NSDocument`/`DocumentGroup` for document apps |
| **Qt 6** | QML for modern UIs; `QThread`/`QtConcurrent` for background work; deploy with `windeployqt`/`macdeployqt`; respect LGPL obligations if dynamically linking open-source Qt |

---

## 8. Local Data and Secure Storage

### 8.1 Standard locations

| Data | Windows | macOS | Linux (XDG) |
|---|---|---|---|
| App data (per user) | `%APPDATA%\Vendor\App` (roaming) or `%LOCALAPPDATA%\Vendor\App` | `~/Library/Application Support/<bundle-id>` | `$XDG_DATA_HOME/app` (`~/.local/share/app`) |
| Config | `%APPDATA%\…` | `~/Library/Application Support` / Preferences (via `UserDefaults`) | `$XDG_CONFIG_HOME/app` (`~/.config/app`) |
| Cache | `%LOCALAPPDATA%\…\Cache` | `~/Library/Caches/<bundle-id>` | `$XDG_CACHE_HOME/app` (`~/.cache/app`) |
| Logs | `%LOCALAPPDATA%\…\Logs` | `~/Library/Logs/<app>` | `$XDG_STATE_HOME/app` or cache dir |
| User documents | User-chosen locations (file dialogs) | Same | Same |

Never write to the install directory (`Program Files`, `/Applications`) at runtime.

### 8.2 Secure storage

| OS | API |
|---|---|
| Windows | Credential Manager / DPAPI (`ProtectedData`), Windows Hello for user verification |
| macOS | **Keychain** (Keychain Services), Secure Enclave-backed keys |
| Linux | Secret Service API (GNOME Keyring, KWallet) via libsecret |
| Cross-platform libs | Electron `safeStorage`, `keytar`-style libraries (check maintenance), Rust `keyring` crate, .NET wrappers |

```text
🔴 Tokens and secrets only in OS secure storage — never plaintext config files
🔴 Local databases (SQLite) containing personal data: consider encryption (SQLCipher) and OS file permissions
🟠 Schema migrations for local DBs; users skip versions
🟠 Clear local personal data on sign-out / uninstall where appropriate
```

---

## 9. OS Integration

| Feature | Notes |
|---|---|
| File associations & protocol handlers (`myapp://`) | Register in installer/manifest; validate all parameters (untrusted input) |
| Single-instance enforcement | Forward arguments to the running instance |
| System tray / menu bar | Platform-appropriate; don't hide the app with no way to quit |
| Notifications | Native notification APIs (Windows toast, macOS UserNotifications, Linux libnotify/portals) |
| Start at login | Opt-in only; Windows Startup/registry via installer or MSIX startup task; macOS `SMAppService` login items |
| Drag & drop, clipboard | Sanitise inputs; respect formats |
| Native menus & shortcuts | macOS app menu conventions; Windows/Linux in-window menus or ribbons |
| High-DPI & multi-monitor | Per-monitor DPI awareness (Windows), Retina assets, fractional scaling on Linux/Wayland |
| Sandboxed portals (Linux) | XDG Desktop Portals for file pickers, screenshots under Flatpak/Wayland |
| Power/sleep events | Pause sync, handle resume (network reconnect, token refresh) |

---

## 10. Packaging and Installers

### 10.1 Formats

| OS | Format | Notes |
|---|---|---|
| Windows | **MSIX** | Modern packaging: clean install/uninstall, containerised file/registry writes, App Installer updates, Store-compatible |
| Windows | **MSI** (WiX Toolset) | Enterprise standard for deployment via Intune/SCCM/GPO |
| Windows | NSIS / Inno Setup | Flexible EXE installers |
| Windows | Squirrel.Windows | Per-user installs with delta updates (common with Electron) |
| macOS | `.app` in **DMG** (drag-to-Applications) or **PKG** | PKG for enterprise/MDM and when installing helpers |
| Linux | **deb**, **rpm** | Distro packages; repository hosting for updates |
| Linux | **Flatpak** (Flathub), **Snap**, **AppImage** | Cross-distro; Flatpak/Snap provide sandboxing and update channels |

### 10.2 Installer rules

```text
🔴 Per-user install by default (no admin); per-machine option for enterprise
🔴 Clean uninstall (remove binaries; ask before deleting user data)
🔴 Silent install/uninstall switches for enterprise (/quiet, /S, msiexec /qn) documented
🟠 Architecture-specific builds: x64 and ARM64 (Windows), universal or separate arm64/x64 (macOS), x86_64/aarch64 (Linux)
🟠 Prerequisites bundled or detected (WebView2 runtime for Tauri/WebView2 apps, .NET runtime unless self-contained)
```

---

## 11. Code Signing and Notarisation

### 11.1 Windows (Authenticode)

| Topic | Current state |
|---|---|
| Why | Unsigned or newly signed apps trigger **Microsoft Defender SmartScreen** warnings; signing establishes publisher identity and builds reputation |
| Key storage | Since mid-2023, CA/Browser Forum rules require code-signing private keys to be generated and stored in **hardware** (HSM, hardware token, or cloud HSM-backed service) — exportable PFX files are no longer issued for new certificates |
| Certificate validity | From **1 March 2026**, publicly trusted code-signing certificates have a maximum validity of **460 days** — plan for more frequent renewals |
| Algorithms | SHA-256 signatures (SHA-1 no longer supported) |
| Timestamping | Always timestamp (RFC 3161) so signatures stay valid after certificate expiry (unless revoked) |
| **Azure Artifact Signing** (formerly Trusted Signing) | Microsoft's fully managed signing service — **generally available since Jan 2026** in the US, Canada, and Europe, with eligibility requirements; integrates with SignTool, GitHub Actions, Azure DevOps; Electron tooling supports it. Check whether your organisation/region is eligible |
| Alternatives | OV/EV certificates from public CAs with cloud signing (e.g. CA-hosted key lockers) or cloud KMS/HSM integrations (AWS/Azure/Google KMS with jsign/SignTool plugins) |

```powershell
# SignTool with timestamping (key in a hardware/cloud-backed provider)
signtool sign /fd SHA256 /tr http://timestamp.digicert.com /td SHA256 /n "ShopNow Technologies Pvt Ltd" .\ShopNowSetup.exe
signtool verify /pa /v .\ShopNowSetup.exe
```

> Use your CA's or signing service's documented timestamp URL; the one above is an example.

### 11.2 macOS (Developer ID + notarisation)

```text
1. Sign every binary, framework, and helper with a Developer ID Application certificate
2. Enable the Hardened Runtime; request only needed entitlements
3. Notarise with Apple's notary service (xcrun notarytool) — Apple scans and issues a ticket
4. Staple the ticket (xcrun stapler staple) so Gatekeeper can verify offline
5. Distribute (DMG/PKG — sign the PKG with a Developer ID Installer certificate)
```

```bash
codesign --force --options runtime --timestamp --entitlements entitlements.plist \
  --sign "Developer ID Application: ShopNow Technologies (TEAMID1234)" "ShopNow.app"
ditto -c -k --keepParent "ShopNow.app" ShopNow.zip
xcrun notarytool submit ShopNow.zip --keychain-profile "notary-profile" --wait
xcrun stapler staple "ShopNow.app"
spctl --assess --type execute --verbose "ShopNow.app"     # Gatekeeper check
```

Mac App Store apps instead use Apple Distribution certificates, the **App Sandbox**, and App Store review.

### 11.3 Linux

Sign repository metadata/packages with GPG (APT/YUM repos), publish checksums (SHA-256) and signatures for direct downloads; Flathub/Snap Store handle signing within their ecosystems.

### 11.4 Signing rules

```text
🔴 Signing happens in CI with keys in HSM/cloud signing services; never on developer laptops for releases
🔴 Restrict who/what can trigger signing (protected branches, environments with approvals, OIDC-scoped identities)
🔴 Sign everything you ship: executables, DLLs/dylibs, installers, update packages
🟠 Monitor certificate expiry (460-day maximum validity) and renew early
🟠 Keep an audit log of signing operations
```

---

## 12. Distribution Channels

| Channel | Pros | Cons |
|---|---|---|
| Direct download (website) | Full control, any business model | You own signing, updates, reputation |
| **Microsoft Store** | Discovery, Store-managed updates, trust; accepts MSIX and Win32 installers | Policies, review |
| **winget** (Windows Package Manager) | Developer/IT-friendly install & upgrade (`winget install Vendor.App`) — manifests submitted to the community repository | Requires stable URLs and hashes per version |
| **Mac App Store** | Trust, discovery, managed updates | Sandbox restrictions, review, platform fees |
| **Homebrew Cask** | Developer audience on macOS | Community-maintained casks |
| Flathub / Snap Store | Linux reach, sandboxing, updates | Packaging effort per store |
| Enterprise (Intune, Configuration Manager, Jamf, Kandji, Workspace ONE) | Managed deployment | MSI/MSIX/PKG + documentation |

Package manager usage for developers: **37-package-managers.md**, Windows and macOS setup flows: **41(a)**, **41(b)**.

---

## 13. Auto-Update

### 13.1 Options

| Stack | Updater |
|---|---|
| Electron | Built-in `autoUpdater` (Squirrel on Windows/macOS) with update servers (e.g. update.electronjs.org for public GitHub releases), **electron-updater** (electron-builder) |
| Tauri | Built-in updater plugin with signed artifacts |
| macOS native | **Sparkle** (EdDSA-signed appcasts) |
| Windows MSIX | App Installer file with update settings; Store updates |
| .NET | Velopack (successor to Squirrel-style updating), ClickOnce (legacy), MSIX |
| Linux | Package repositories, Flatpak/Snap channels, AppImageUpdate |

### 13.2 Update rules

```text
🔴 Updates are downloaded over HTTPS AND verified by signature before installation (code signature and/or update-manifest signature)
🔴 Update server/feed is protected like production infrastructure (compromise = malware distribution)
🔴 Never auto-downgrade without explicit intent; protect against rollback attacks where supported
🟠 Staged rollouts (percentage of users), with ability to pause and roll back (publish a newer fixed version)
🟠 Delta updates for large apps; background download; restart prompts that respect user work
🟠 Release channels: stable, beta, nightly (opt-in)
🟠 Minimum supported version enforcement for security-critical fixes (server-side check + forced update prompt)
```

---

## 14. Performance

| Area | Practice |
|---|---|
| Startup | Show a window fast; lazy-load heavy modules; defer non-critical initialisation; measure cold/warm start per OS |
| Memory | Electron: minimise renderers/BrowserViews, avoid memory leaks in long-running sessions; .NET: watch large object heap; profile regularly |
| UI thread | Never block the UI thread with I/O or CPU work — background threads/workers/Rust commands |
| Rendering | GPU acceleration where appropriate; virtualised lists for large data; avoid layout thrash in web UIs |
| Bundle size | Prune dependencies; ASAR packaging; tree shaking; strip debug symbols (upload separately for crash reporting) |
| Battery | Throttle background timers and polling; respect power-saving modes |
| Profiling | Chrome DevTools (renderer), Electron `contentTracing`, Windows Performance Analyzer/PerfView/dotnet-trace, Instruments (macOS), `perf`/heaptrack (Linux) |

---

## 15. UX Conventions per Platform

| Convention | Windows | macOS | Linux (GNOME/KDE) |
|---|---|---|---|
| Menu location | In window (or ribbon) | Global menu bar with app menu | In window / header bar (GNOME) |
| Primary modifier | Ctrl | ⌘ Command | Ctrl |
| Quit | Close last window often exits (app-dependent) | App stays running after closing windows (document apps) — `⌘Q` quits | App-dependent |
| Settings | "Settings"/"Options" | "Settings…" in app menu (`⌘,`) | "Preferences"/"Settings" |
| Window controls | Right side | Left side (traffic lights) | Varies |
| Design language | Fluent | Human Interface Guidelines | GNOME HIG / KDE HIG |
| Dark mode, accent colours | Follow system | Follow system | Follow system/portal settings |

Use platform keyboard shortcuts (`Ctrl/⌘+C`, `Ctrl/⌘+Z`, etc.) and native file dialogs.

---

## 16. Accessibility

| OS | Accessibility API | Screen readers |
|---|---|---|
| Windows | UI Automation (UIA) | Narrator, NVDA, JAWS |
| macOS | NSAccessibility | VoiceOver |
| Linux | AT-SPI | Orca |

```text
🔴 Full keyboard operation; visible focus; logical tab order
🔴 Accessible names/roles for custom controls (native controls provide them; custom-drawn UIs must implement them)
🔴 Respect system text scaling, high-contrast themes (Windows contrast themes / forced colours), reduced motion
🟠 Web-tech apps: follow chapter 12 §12 (Chromium exposes web accessibility tree to OS APIs)
🟠 Test with Accessibility Insights for Windows, Accessibility Inspector (macOS), Orca/Accerciser (Linux)
```

---

## 17. Localisation

Same principles as chapter 12 §15 and chapter 14 §17: externalised strings, ICU plurals, locale-aware formatting via OS/ICU APIs, RTL layouts, fonts for Indic scripts, localised installers and store listings, and per-app language selection where the platform supports it.

---

## 18. Offline, Sync, and Networking

```text
🔴 Assume intermittent connectivity (laptops sleep, roam, lose VPN)
🔴 Respect system proxy settings and corporate TLS inspection (use OS trust store; allow enterprise CA configuration)
🔴 Timeouts and retries on network calls; reconnect after resume
🟠 Local-first patterns with sync and conflict resolution (chapter 14 §8 applies)
🟠 Support offline licence grace periods (§23)
```

---

## 19. Testing

| Level | Tools |
|---|---|
| Unit | xUnit/NUnit/MSTest (.NET), XCTest/Swift Testing (macOS), Vitest/Jest (Electron/web UI), `cargo test` (Tauri core), Qt Test |
| UI automation — web-tech | **Playwright** (Electron support), WebdriverIO, Tauri WebDriver support |
| UI automation — Windows native | Appium Windows driver / FlaUI (UIA-based) |
| UI automation — macOS native | XCUITest |
| Visual regression | Screenshot comparison per OS/theme |
| Installer/update tests | Install, upgrade from N-1/N-2, uninstall, silent install on clean VMs; signature verification checks |
| OS matrix | CI on Windows (x64 + ARM64 where possible), macOS (arm64 + x64 if supported), Linux (X11 + Wayland sessions for GUI tests where feasible) |

```text
Release smoke test (per OS): clean install → launch → login → core workflow → update to next build → uninstall
```

---

## 20. CI/CD for Desktop

```yaml
# GitHub Actions matrix (illustrative)
name: desktop-release
on: { push: { tags: ['desktop/v*'] } }
permissions: { contents: write, id-token: write }
jobs:
  build:
    strategy:
      matrix:
        include:
          - os: windows-latest
          - os: macos-latest
          - os: ubuntu-latest
    runs-on: ${{ matrix.os }}
    environment: release-signing          # protected environment with approvals and signing secrets/OIDC
    steps:
      - uses: actions/checkout@<full-commit-sha>
      - uses: actions/setup-node@<full-commit-sha>
        with: { node-version: 24 }
      - run: npm ci
      - run: npm test
      - run: npm run make                  # Electron Forge: package + sign (+ notarise on macOS via config)
      - uses: actions/upload-artifact@<full-commit-sha>
        with: { name: dist-${{ matrix.os }}, path: out/make/** }
```

```text
🔴 Reproducible builds from tagged commits; artifacts + checksums + SBOM attached to releases (chapter 09 §16)
🔴 Signing/notarisation automated in protected CI environments
🟠 Publish to update feed only after smoke tests pass on signed artifacts
🟠 Staged release: beta channel → stable percentage rollout
```

---

## 21. Crash Reporting and Telemetry

| Need | Tools |
|---|---|
| Native crash dumps | Crashpad/Breakpad (Electron uses Crashpad), Windows Error Reporting, macOS crash reports |
| Aggregation | Sentry, BugSplat, Backtrace, Raygun, Bugsnag |
| Symbols | Upload PDBs (Windows), dSYMs (macOS), debug symbols (Linux) per release — keep them out of shipped binaries |
| Usage telemetry | Opt-in or consent-based per privacy law; anonymised; documented in privacy policy |

```text
🔴 Consent and transparency for telemetry (DPDP/GDPR); enterprise policy to disable telemetry
🔴 Scrub file paths/usernames and document contents from crash reports
🟠 Track crash-free sessions per version/OS; alert on regressions after rollout
```

---

## 22. Enterprise Deployment

| Requirement | Practice |
|---|---|
| Silent install/uninstall | MSI/MSIX/PKG with documented switches |
| Managed configuration | Windows Group Policy (ADMX templates) or registry policies; macOS configuration profiles (managed preferences); Linux system-wide config files |
| Update control | Ability to disable auto-update and deploy versions via MDM/SCCM; or managed update channels |
| Proxy/TLS | System proxy, PAC files, enterprise root CAs honoured |
| SSO | OIDC/SAML via system browser; Windows integrated auth / Kerberos where needed; macOS Platform SSO/enterprise SSO extensions |
| Security reviews | Provide SBOM, signing details, network endpoints list, data flow documentation |
| Least privilege | No admin rights required for normal use; no services running as SYSTEM/root unless essential |

---

## 23. Licensing and Activation

```text
- Licence validation on the server where possible (accounts/subscriptions) — client-side checks can be bypassed
- Signed licence files (asymmetric signatures) for offline activation; public key in the app, private key on the server
- Grace periods for offline use; clear messaging when licences expire
- Device limits enforced server-side; allow users to manage/deactivate devices
- Respect open-source licence obligations of bundled components (e.g. LGPL Qt, Chromium/Electron notices) — ship third-party notices
- Don't collect excessive hardware identifiers (privacy); document what is used
```

---

## 24. Reference Snippets by Stack

### 24.1 WPF / WinUI with CommunityToolkit.Mvvm (C#)

```csharp
public partial class OrdersViewModel : ObservableObject
{
    private readonly IOrdersService _orders;
    public OrdersViewModel(IOrdersService orders) => _orders = orders;

    [ObservableProperty] private bool isBusy;
    [ObservableProperty] private string? errorMessage;
    public ObservableCollection<OrderRow> Orders { get; } = new();

    [RelayCommand]
    private async Task LoadAsync(CancellationToken ct)
    {
        IsBusy = true; ErrorMessage = null;
        try
        {
            var items = await _orders.GetRecentAsync(ct);      // I/O off the UI thread (async)
            Orders.Clear();
            foreach (var o in items) Orders.Add(OrderRow.From(o));
        }
        catch (OperationCanceledException) { }
        catch (Exception ex) { ErrorMessage = "Could not load orders."; Log.Error(ex, "Load failed"); }
        finally { IsBusy = false; }
    }
}
```

```csharp
// App startup with Generic Host: DI, configuration, logging for desktop apps
var host = Host.CreateDefaultBuilder()
    .ConfigureServices(s =>
    {
        s.AddSingleton<IOrdersService, OrdersService>();
        s.AddHttpClient<IOrdersService, OrdersService>(c => c.Timeout = TimeSpan.FromSeconds(10));
        s.AddTransient<OrdersViewModel>();
        s.AddSingleton<MainWindow>();
    })
    .Build();
```

### 24.2 macOS SwiftUI document-style app (Swift)

```swift
@main
struct ShopNowAdminApp: App {
    @State private var model = AppModel()
    var body: some Scene {
        WindowGroup { ContentView().environment(model) }
            .commands {
                CommandGroup(after: .newItem) {
                    Button("Export Orders…") { model.exportOrders() }
                        .keyboardShortcut("e", modifiers: [.command, .shift])
                }
            }
        Settings { SettingsView().environment(model) }      // standard ⌘, settings window
        MenuBarExtra("ShopNow", systemImage: "cart") { MenuBarView().environment(model) }
    }
}
```

### 24.3 Qt 6 / QML (C++ backend object)

```cpp
class OrdersModel : public QAbstractListModel {
    Q_OBJECT
    QML_ELEMENT
public:
    Q_INVOKABLE void refresh();               // runs network I/O asynchronously (QNetworkAccessManager)
    int rowCount(const QModelIndex& = {}) const override { return static_cast<int>(m_rows.size()); }
    QVariant data(const QModelIndex& index, int role) const override;
private:
    std::vector<OrderRow> m_rows;
};
```

### 24.4 Electron renderer using the preload API (TypeScript)

```ts
declare global {
  interface Window { api: { exportOrders(range: { from: string; to: string }): Promise<string> } }
}

async function onExportClick() {
  try {
    const path = await window.api.exportOrders({ from: '2026-09-01', to: '2026-09-30' });
    showToast(`Exported to ${path}`);
  } catch {
    showToast('Export failed. Please try again.');
  }
}
```

### 24.5 Tauri frontend invoking a command (TypeScript)

```ts
import { invoke } from '@tauri-apps/api/core';
const path = await invoke<string>('export_orders', { from: '2026-09-01', to: '2026-09-30' });
```

---

## 25. Scenario Playbooks

### 25.1 Internal Windows LOB app for a bank branch network
WPF or WinUI 3 on .NET LTS; MVVM; SSO with the enterprise IdP (system browser, PKCE); MSI built with WiX; Authenticode signing via cloud HSM; deployed silently through Intune/Configuration Manager; Group Policy (ADMX) for server URLs and telemetry opt-out; auto-update disabled in favour of managed deployment; offline queue for branch connectivity drops; audit logging to a central service.

### 25.2 Cross-platform SaaS companion app (Windows, macOS, Linux)
Electron (Forge) or Tauri 2; shared React UI with the web app; strict IPC surface; Windows signing via Azure Artifact Signing (if eligible) or CA cloud signing; macOS notarisation in CI; Linux AppImage + Flatpak; signed auto-update with beta/stable channels and staged rollout; Sentry crash reporting with symbols; telemetry consent on first run.

### 25.3 Developer tool distributed to engineers
Tauri or native (Go/Rust CLI + optional GUI); distribution via winget, Homebrew Cask, and deb/rpm repositories; signed binaries and checksums; SBOM on GitHub Releases; auto-update via package managers rather than in-app updater; minimal telemetry, opt-in.

### 25.4 Point-of-sale / kiosk app
Native or Avalonia/Qt for hardware integration (printers, scanners, cash drawers); locked-down OS (Windows assigned access / kiosk mode); offline-first local database with reliable sync and idempotent transactions; remote monitoring and managed updates during off-hours; tamper-evident logs.

### 25.5 Migrating a legacy WinForms/WPF app
Characterisation tests → upgrade to current .NET LTS (upgrade assistant) → MSIX packaging → introduce MVVM and DI incrementally → replace risky components (direct DB access) with API calls → evaluate cross-platform move (Avalonia/Uno/web) only with a clear business driver.

---

## 26. Checklists

### Technology & architecture
- [ ] Technology choice recorded in ADR (team skills, platforms, footprint, security model)
- [ ] Layered architecture; OS-specific code behind interfaces
- [ ] Electron: context isolation, sandbox, no Node in renderer, validated IPC, strict CSP, navigation restrictions, fuses
- [ ] Tauri: minimal capabilities, scoped plugins, validated commands, CSP
- [ ] Secrets in OS secure storage; app data in standard per-user locations

### Build, sign, ship
- [ ] Builds per OS/architecture in CI from tags
- [ ] Windows Authenticode signing with HSM/cloud-backed keys (or Azure Artifact Signing) + timestamping
- [ ] Certificate renewal tracked (≤ 460-day validity)
- [ ] macOS Developer ID signing + hardened runtime + notarisation + stapling
- [ ] Linux packages/repos signed; checksums published
- [ ] Signed, verified auto-updates; staged rollout; channels
- [ ] Installers: per-user default, clean uninstall, silent switches

### Quality & operations
- [ ] Unit + UI automation + installer/update tests on all supported OSes
- [ ] Accessibility checked with platform tools and screen readers
- [ ] Crash reporting with symbols; telemetry with consent
- [ ] Startup time and memory budgets tracked per release
- [ ] Enterprise: managed config, proxy support, SSO, documentation, SBOM
- [ ] Supported-OS policy published; end-of-support notices in-app

---

## 27. References

### Platforms
- Windows App SDK / WinUI 3: https://learn.microsoft.com/windows/apps/windows-app-sdk/
- WPF: https://learn.microsoft.com/dotnet/desktop/wpf/
- MSIX: https://learn.microsoft.com/windows/msix/
- Windows 10 end of support: https://learn.microsoft.com/lifecycle/products/windows-10-home-and-pro
- Azure Artifact Signing (GA announcement): https://techcommunity.microsoft.com/blog/microsoft-security-blog/-/4482789
- SignTool: https://learn.microsoft.com/windows/win32/seccrypto/signtool
- Apple — Notarizing macOS software: https://developer.apple.com/documentation/security/notarizing-macos-software-before-distribution
- Apple Human Interface Guidelines: https://developer.apple.com/design/human-interface-guidelines/
- XDG Base Directory Specification: https://specifications.freedesktop.org/basedir-spec/latest/
- Flatpak: https://docs.flatpak.org/ · Snapcraft: https://snapcraft.io/docs

### Frameworks
- Electron security: https://www.electronjs.org/docs/latest/tutorial/security
- Electron code signing: https://www.electronjs.org/docs/latest/tutorial/code-signing
- Electron Fuses: https://www.electronjs.org/docs/latest/tutorial/fuses
- Electron Forge: https://www.electronforge.io/ · electron-builder: https://www.electron.build/
- Tauri 2: https://v2.tauri.app/ · Tauri security & capabilities: https://v2.tauri.app/security/
- Avalonia: https://docs.avaloniaui.net/ · .NET MAUI: https://learn.microsoft.com/dotnet/maui/
- Qt 6: https://doc.qt.io/ · Flutter desktop: https://docs.flutter.dev/platform-integration/desktop
- Compose Multiplatform: https://www.jetbrains.com/compose-multiplatform/
- Sparkle: https://sparkle-project.org/ · Velopack: https://velopack.io/

### Distribution
- Microsoft Store for developers: https://learn.microsoft.com/windows/apps/publish/
- winget manifests: https://learn.microsoft.com/windows/package-manager/package/
- Homebrew Cask: https://docs.brew.sh/Cask-Cookbook
- Flathub: https://docs.flathub.org/

### Security & accessibility
- CA/Browser Forum Code Signing Baseline Requirements: https://cabforum.org/working-groups/code-signing/
- Accessibility Insights for Windows: https://accessibilityinsights.io/
- OWASP Desktop App Security Top 10 (project page): https://owasp.org/www-project-desktop-app-security-top-10/

---

**Previous:** [14 — Mobile Engineering](./14-mobile-engineering.md) · **Next:** [16 — DevOps & CI/CD](./16-devops-and-ci-cd.md)