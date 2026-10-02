# 📱 Mobile Engineering — Production Engineering Guide

> How to build, ship, and operate production mobile apps on Android and iOS: current store requirements, choosing native vs cross-platform (Kotlin/Jetpack Compose, Swift/SwiftUI, Kotlin Multiplatform, Flutter, React Native), app architecture, modularisation, networking and offline-first data, concurrency, performance (startup, jank, ANRs, battery, size), security (OWASP MASVS), privacy and store disclosures, authentication and passkeys, push notifications, deep links, accessibility, localisation, testing, CI/CD and signing, release management (staged rollouts, TestFlight, phased release), remote config and forced updates, observability, and store compliance.
>
> Related: [05 Design](./05-software-design.md) · [06 Kotlin/Swift/Dart standards](./06-coding-standards.md) · [08 Mobile testing](./08-testing-and-quality.md) · [09 Security](./09-security-engineering.md) · [11 API design](./11-api-and-integration.md) · IDEs: [35 Android Studio](./35-android-studio-extensions.md) · [36 Xcode](./36-xcode-extensions.md) · Setup: [41(b) macOS](./41-(b)macOS-Tahoe-Fresh-Setup.md)

---

## 📚 Table of Contents

- [📱 Mobile Engineering — Production Engineering Guide](#-mobile-engineering--production-engineering-guide)
  - [📚 Table of Contents](#-table-of-contents)
  - [1. Principles](#1-principles)
  - [2. Platform and Store Requirements (2026)](#2-platform-and-store-requirements-2026)
    - [2.1 Google Play](#21-google-play)
    - [2.2 Apple App Store](#22-apple-app-store)
    - [2.3 OS support policy (define yours)](#23-os-support-policy-define-yours)
  - [3. Choosing an Approach](#3-choosing-an-approach)
    - [Decision guide](#decision-guide)
  - [4. App Architecture](#4-app-architecture)
    - [4.1 Layers (Android's official architecture guidance; equally applicable to iOS)](#41-layers-androids-official-architecture-guidance-equally-applicable-to-ios)
    - [4.2 Unidirectional data flow (UDF)](#42-unidirectional-data-flow-udf)
    - [4.3 Architecture patterns by stack](#43-architecture-patterns-by-stack)
  - [5. Modularisation and Project Structure](#5-modularisation-and-project-structure)
    - [5.1 Android multi-module (example)](#51-android-multi-module-example)
    - [5.2 iOS structure](#52-ios-structure)
  - [6. UI Development](#6-ui-development)
  - [7. Networking](#7-networking)
    - [Certificate pinning — decide carefully](#certificate-pinning--decide-carefully)
  - [8. Offline-First Data and Sync](#8-offline-first-data-and-sync)
    - [8.1 Pattern](#81-pattern)
    - [8.2 Sync rules](#82-sync-rules)
  - [9. Concurrency and Background Work](#9-concurrency-and-background-work)
  - [10. Performance](#10-performance)
    - [10.1 Key metrics](#101-key-metrics)
    - [10.2 Android performance tools and techniques](#102-android-performance-tools-and-techniques)
    - [10.3 iOS performance techniques](#103-ios-performance-techniques)
  - [11. Mobile Security (OWASP MASVS)](#11-mobile-security-owasp-masvs)
    - [Key rules](#key-rules)
  - [12. Privacy and Store Disclosures](#12-privacy-and-store-disclosures)
  - [13. Authentication](#13-authentication)
  - [14. Push Notifications](#14-push-notifications)
  - [15. Deep Links and App Links](#15-deep-links-and-app-links)
  - [16. Accessibility](#16-accessibility)
  - [17. Localisation](#17-localisation)
  - [18. Testing](#18-testing)
  - [19. Build, Signing, and CI/CD](#19-build-signing-and-cicd)
    - [19.1 Signing](#191-signing)
    - [19.2 Versioning](#192-versioning)
    - [19.3 CI/CD pipeline](#193-cicd-pipeline)
  - [20. Release Management](#20-release-management)
    - [20.1 Release train](#201-release-train)
    - [20.2 Rollout gates](#202-rollout-gates)
    - [20.3 OTA updates (React Native / Flutter code push)](#203-ota-updates-react-native--flutter-code-push)
  - [21. Remote Config, Feature Flags, and Forced Updates](#21-remote-config-feature-flags-and-forced-updates)
  - [22. Observability](#22-observability)
  - [23. Store Compliance Essentials](#23-store-compliance-essentials)
  - [24. In-App Purchases and Payments](#24-in-app-purchases-and-payments)
  - [25. Mobile Backend Considerations](#25-mobile-backend-considerations)
  - [26. Scenario Playbooks](#26-scenario-playbooks)
    - [26.1 New consumer app (India-first, low-end devices)](#261-new-consumer-app-india-first-low-end-devices)
    - [26.2 Banking / fintech app](#262-banking--fintech-app)
    - [26.3 Existing web company adding mobile](#263-existing-web-company-adding-mobile)
    - [26.4 Enterprise internal app](#264-enterprise-internal-app)
  - [27. Checklists](#27-checklists)
    - [Architecture \& code](#architecture--code)
    - [Quality \& performance](#quality--performance)
    - [Release](#release)
  - [28. References](#28-references)
    - [Android](#android)
    - [Apple](#apple)
    - [Cross-platform](#cross-platform)
    - [Security and tooling](#security-and-tooling)

---

## 1. Principles

| # | Principle |
|---:|---|
| 1 | **You can't hotfix instantly.** Store review and user update lag mean every release lives in the wild for months — design for backward compatibility and remote kill switches. |
| 2 | **Old versions call your APIs for years.** APIs must stay compatible with supported app versions (chapter 11 §11). |
| 3 | **Devices are constrained and diverse.** Low-end Android phones, flaky networks, small batteries — test on them. |
| 4 | **The app is untrusted.** Anything shipped in the binary (keys, logic, endpoints) can be extracted; the server enforces security. |
| 5 | **Offline is a normal state**, not an error. |
| 6 | **Privacy is a store requirement and a law** — disclosures, consent, minimal permissions. |
| 7 | **Measure crashes, ANRs, startup, and jank in production** — store visibility depends on them. |

---

## 2. Platform and Store Requirements (2026)

Store requirements change every year. Verify current rules on the official pages before each release cycle.

### 2.1 Google Play

| Requirement | Detail |
|---|---|
| **Target API level** | From **31 Aug 2026**, new apps and app updates must target **Android 16 (API level 36)** or higher (Wear OS and Android Automotive OS: API 35; Android TV and Android XR: API 34). Existing apps must target at least Android 15 (API 35) to remain available to new users on newer Android versions. |
| App format | Android App Bundle (AAB) for new apps; **Play App Signing** |
| Data safety section | Required disclosure of data collection, sharing, and security practices |
| Account deletion | Apps with account creation must offer in-app and web account deletion paths |
| Policies | Permissions (e.g. restricted SMS/Call Log, background location), payments, families, financial services declarations where applicable |
| Quality signals | Android vitals bad-behaviour thresholds (user-perceived crash rate and ANR rate) affect visibility |

### 2.2 Apple App Store

| Requirement | Detail |
|---|---|
| **Build toolchain** | Since **28 April 2026**, apps uploaded to App Store Connect must be built with **Xcode 26 or later** using the iOS 26 / iPadOS 26 (or tvOS/visionOS/watchOS 26) SDKs. Apple raises this baseline annually — expect the next Xcode requirement in spring of the following year. |
| Privacy | App Privacy details ("nutrition labels"); **privacy manifests** (`PrivacyInfo.xcprivacy`) declaring data use and **required-reason APIs**, including for third-party SDKs |
| Tracking | App Tracking Transparency (ATT) permission before cross-app tracking |
| Account deletion | Apps that support account creation must allow account deletion from within the app |
| Sign in with Apple | Required in some cases when offering third-party/social login (check guideline 4.8) |
| Review guidelines | App Store Review Guidelines (payments, content, privacy, design) |

### 2.3 OS support policy (define yours)

```text
Android: minSdk chosen from your user analytics and library requirements (many apps choose API 24–26+)
iOS:     support the current and previous major iOS versions (often N-1 or N-2) based on analytics
Review annually after OS releases; drop old versions deliberately with in-app messaging ahead of time
```

---

## 3. Choosing an Approach

| Approach | Language / UI | Strengths | Trade-offs | Fit |
|---|---|---|---|---|
| **Native Android** | Kotlin + **Jetpack Compose** | Best platform integration, performance, newest APIs | Separate iOS codebase | Platform-critical apps, large teams per platform |
| **Native iOS** | Swift + **SwiftUI** (+ UIKit where needed) | Best Apple integration and UX fidelity | Separate Android codebase | Same as above |
| **Kotlin Multiplatform (KMP)** | Shared Kotlin business logic; native UI or **Compose Multiplatform** UI | Share logic while keeping native UX; incremental adoption | iOS team must accept Kotlin; tooling maturing | Teams with strong Android/Kotlin skills sharing domain/data layers |
| **Flutter** | Dart, own rendering engine | Single UI codebase, consistent look, fast iteration | Non-native widgets; platform channels for native features | Product apps wanting one UI across platforms |
| **React Native** (New Architecture; **Expo** framework) | TypeScript/React, native components | Web/React skills, OTA updates for JS (within store rules), large ecosystem | Native modules for advanced features; upgrade cadence | Teams with React expertise; content/commerce apps |
| **Hybrid web** (Capacitor/Ionic) | Web app in a native shell | Reuse web app | Web-like performance/UX | Internal tools, content apps |
| **PWA** | Web | No store, instant updates | Limited device APIs, iOS constraints | Lightweight experiences |

### Decision guide

```text
Need deep platform features (camera pipelines, AR, widgets, watch, CarPlay/Android Auto) and best UX? → Native (± KMP for shared logic)
One small team, two platforms, standard business UI?                                             → Flutter or React Native (Expo)
Existing native apps, want to reduce duplicated logic?                                           → KMP for networking/data/domain
Existing React web team, content/commerce app?                                                   → React Native (Expo)
```

Record the decision in an ADR, including hiring, long-term maintenance, and native-module risk.

---

## 4. App Architecture

### 4.1 Layers (Android's official architecture guidance; equally applicable to iOS)

```text
┌──────────────────────────────┐
│ UI layer                     │  Screens (Compose/SwiftUI), state holders (ViewModel / @Observable models)
│  - UI state (immutable)      │  Unidirectional data flow: events up, state down
├──────────────────────────────┤
│ Domain layer (optional)      │  Use cases / interactors for complex or reused business logic
├──────────────────────────────┤
│ Data layer                   │  Repositories (single source of truth), data sources (network, DB, prefs)
└──────────────────────────────┘
```

### 4.2 Unidirectional data flow (UDF)

```kotlin
// Android — ViewModel exposes immutable UI state as StateFlow
data class OrderUiState(
    val isLoading: Boolean = true,
    val order: OrderUi? = null,
    val error: UserMessage? = null,
)

@HiltViewModel
class OrderViewModel @Inject constructor(
    private val repo: OrderRepository,
    savedStateHandle: SavedStateHandle,
) : ViewModel() {
    private val orderId: String = checkNotNull(savedStateHandle["orderId"])

    val uiState: StateFlow<OrderUiState> = repo.observeOrder(orderId)   // DB is the single source of truth
        .map { OrderUiState(isLoading = false, order = it.toUi()) }
        .catch { emit(OrderUiState(isLoading = false, error = it.toUserMessage())) }
        .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5_000), OrderUiState())

    fun onRetryPayment(method: PaymentMethod) = viewModelScope.launch {
        repo.retryPayment(orderId, method)          // refresh flows back through observeOrder
    }
}

@Composable
fun OrderRoute(viewModel: OrderViewModel = hiltViewModel()) {
    val state by viewModel.uiState.collectAsStateWithLifecycle()
    OrderScreen(state = state, onRetry = viewModel::onRetryPayment)
}
```

```swift
// iOS — Observation framework (iOS 17+) with SwiftUI
@MainActor
@Observable
final class OrderViewModel {
    private(set) var state: OrderState = .loading
    private let repository: OrderRepository
    private let orderID: String

    init(orderID: String, repository: OrderRepository) {
        self.orderID = orderID
        self.repository = repository
    }

    func load() async {
        do { state = .loaded(try await repository.order(id: orderID)) }
        catch { state = .failed(UserMessage(error)) }
    }

    func retryPayment(method: PaymentMethod) async {
        do { try await repository.retryPayment(orderID: orderID, method: method); await load() }
        catch { state = .failed(UserMessage(error)) }
    }
}

struct OrderView: View {
    @State var viewModel: OrderViewModel
    var body: some View {
        content.task { await viewModel.load() }
    }
    @ViewBuilder private var content: some View {
        switch viewModel.state {
        case .loading: ProgressView()
        case .loaded(let order): OrderDetail(order: order) { method in Task { await viewModel.retryPayment(method: method) } }
        case .failed(let message): ErrorView(message: message) { Task { await viewModel.load() } }
        }
    }
}
```

### 4.3 Architecture patterns by stack

| Stack | Common patterns / libraries |
|---|---|
| Android | MVVM/MVI with ViewModel + StateFlow; Hilt (DI); Navigation Compose; Room; DataStore; WorkManager |
| iOS | MVVM with Observation; The Composable Architecture (TCA) for strict UDF; Swift Package Manager modules; SwiftData/Core Data/GRDB |
| KMP | Shared repositories/use cases; Ktor client; SQLDelight/Room (KMP); Koin; platform UIs |
| Flutter | BLoC/Cubit, Riverpod, Provider; go_router; drift/Isar/sqflite |
| React Native | React Query/TanStack Query + Zustand/Redux Toolkit; Expo Router; WatermelonDB/op-sqlite/MMKV |

---

## 5. Modularisation and Project Structure

### 5.1 Android multi-module (example)

```text
app/                              # application module: DI graph, navigation host
build-logic/                      # Gradle convention plugins (shared config)
core/
  ├── designsystem/               # theme, components
  ├── network/                    # HTTP client, interceptors, auth
  ├── database/                   # Room DB, DAOs
  ├── data/                       # repositories
  ├── model/                      # shared models
  └── testing/                    # test utilities, fakes
feature/
  ├── checkout/
  ├── orders/
  └── profile/
gradle/libs.versions.toml         # version catalog
```

Benefits: faster incremental builds, enforced boundaries, ownership per feature. Use **version catalogs** and **convention plugins**; keep feature modules independent (communicate via navigation contracts and core modules).

### 5.2 iOS structure

```text
App/                       # app target: composition root, app lifecycle
Packages/
  ├── DesignSystem/        # Swift Package
  ├── Networking/
  ├── Persistence/
  ├── OrdersFeature/
  └── CheckoutFeature/
Tests/
```

Local Swift packages give module boundaries and faster builds; use `swift-tools-version` consistent with the required Xcode.

---

## 6. UI Development

| Topic | Android (Compose) | iOS (SwiftUI) |
|---|---|---|
| Design system | Material 3 + custom theme tokens | Custom tokens; follow Human Interface Guidelines |
| Previews | `@Preview` with multiple configs (dark, font scale, locales) | `#Preview` with variants |
| Navigation | Navigation Compose (type-safe routes) | `NavigationStack` with typed paths |
| Lists | `LazyColumn` with stable keys | `List`/`LazyVStack` with stable IDs |
| State hoisting | Stateless composables + state holders | Small views; state in models |
| Adaptive layouts | Window size classes; foldables/tablets | Size classes; iPad multitasking |
| Edge-to-edge | Required behaviour when targeting recent API levels — handle insets | Safe areas |

```text
🔴 Every screen handles loading, empty, error, offline, and permission-denied states
🔴 Support dark mode, large fonts (dynamic type / font scale), landscape where relevant, tablets/foldables per product policy
🟠 Stable keys in lists; avoid unnecessary recompositions/re-renders (Compose stability, SwiftUI identity)
🟠 Screenshot tests for key screens across themes and font sizes
```

---

## 7. Networking

```text
🔴 HTTPS only (Android Network Security Config; iOS App Transport Security on by default)
🔴 Timeouts (connect/read), retries with backoff only for idempotent calls or with idempotency keys
🔴 Send app version, platform, and build in headers (User-Agent/custom) for compatibility handling and support
🔴 Handle 401 (refresh token / re-auth), 426/upgrade-required or custom "min version" responses, 429 with Retry-After
🟠 Typed clients generated from OpenAPI (OpenAPI Generator, Apollo for GraphQL); kotlinx.serialization / Codable
🟠 Respect metered networks and Low Data Mode for large transfers; use background transfer APIs for big uploads
🟠 Compress payloads; paginate; cache with ETags
```

| Platform | Libraries |
|---|---|
| Android | OkHttp + Retrofit, Ktor client, kotlinx.serialization/Moshi |
| iOS | `URLSession` with async/await, (Alamofire optional) |
| KMP | Ktor client |
| Flutter | `dio`, `http` |
| React Native | `fetch` + TanStack Query, Axios |

### Certificate pinning — decide carefully

Pinning protects against some MITM scenarios but can brick apps if certificates rotate unexpectedly. If used: pin to **public keys (SPKI) of intermediate/root CAs**, include backup pins, have a remote-config escape hatch, and monitor. Many teams rely on platform TLS validation plus Certificate Transparency instead.

---

## 8. Offline-First Data and Sync

### 8.1 Pattern

```text
UI observes LOCAL database (single source of truth)
Repository: read from DB immediately → fetch from network → write to DB → UI updates automatically
Writes: write locally (pending state) → enqueue sync job → server confirms → reconcile
```

| Storage need | Android | iOS | Cross-platform |
|---|---|---|---|
| Relational / structured | Room | SwiftData, Core Data, GRDB | SQLDelight/Room KMP, drift (Flutter), op-sqlite/WatermelonDB (RN) |
| Key-value preferences | DataStore | `UserDefaults` (non-sensitive only) | MMKV, shared_preferences |
| Secrets/tokens | Android Keystore + encrypted storage | **Keychain** | flutter_secure_storage, expo-secure-store / react-native-keychain |
| Files/media | App-specific storage | App container / caches | — |

### 8.2 Sync rules

```text
🔴 Every mutation has a client-generated ID (UUID) and is idempotent server-side
🔴 Conflict strategy defined per entity: last-write-wins (with server timestamps), version checks (ETag/If-Match → 412),
    merge rules, or CRDTs for collaborative data
🔴 Sync jobs scheduled with platform schedulers (WorkManager / BGTaskScheduler) respecting battery and network constraints
🟠 Schema migrations for local DB versions tested (Room/SwiftData migrations) — users skip app versions
🟠 Show sync status and pending changes to users; never silently drop failed writes
🟠 Clear local personal data on logout/account deletion
```

---

## 9. Concurrency and Background Work

| Platform | Guidance |
|---|---|
| **Android** | Kotlin coroutines + Flow; `viewModelScope`/`lifecycleScope`; inject dispatchers; never block the main thread (ANR after ~5 s for input dispatch); **WorkManager** for deferrable guaranteed work; foreground services only for user-visible ongoing tasks with declared types and permissions |
| **iOS** | Swift concurrency (async/await, actors, `@MainActor`); Swift 6 strict concurrency checking for data-race safety; `BGTaskScheduler` (app refresh/processing tasks) and background `URLSession` transfers; no long work on the main actor |
| **Flutter** | `async`/`Future`; isolates for CPU-heavy work; `workmanager` plugin for background tasks |
| **React Native** | Keep JS thread free; offload heavy work to native modules / worklets; background tasks via platform-specific libraries |

```text
🔴 Cancel work when screens/scopes end (structured concurrency)
🔴 Respect OS background limits — the OS decides when background work runs; design for delays
🟠 Exact alarms, background location, and foreground services require strong justification and permissions on Android
```

---

## 10. Performance

### 10.1 Key metrics

| Metric | Target guidance | Tools |
|---|---|---|
| **Cold start** (time to first frame / fully drawn) | As fast as possible; many teams target < ~1–2 s on mid-range devices | Android: Macrobenchmark, `reportFullyDrawn`, Play Console vitals; iOS: Instruments (App Launch), MetricKit, Xcode Organizer |
| **Jank / frame drops** | Smooth at device refresh rate (60–120 Hz) | Android: JankStats, Perfetto, Compose tracing; iOS: Instruments (Animation Hitches), MetricKit hitch metrics |
| **ANRs (Android)** | Below Play's bad-behaviour threshold (user-perceived ANR rate) | Play Console Android vitals, Crashlytics/Sentry ANR reports |
| **Crash-free users/sessions** | Typically ≥ 99.5–99.9% depending on app category | Crashlytics, Sentry, Play vitals, Xcode Organizer |
| **App size** | Small download/install size (affects conversion in bandwidth-sensitive markets like India) | Play Console size reports, App Store Connect size reports |
| **Battery / network** | No wakelocks/polling abuse; batch network calls | Android Vitals (excessive wakeups), Battery Historian, Instruments Energy Log |
| **Memory** | No leaks; handle low-memory signals | LeakCanary, Android Studio Profiler, Instruments Leaks/Allocations |

### 10.2 Android performance tools and techniques

```text
- Baseline Profiles (and Startup Profiles) — ship AOT compilation hints for critical paths → faster startup and smoother first runs
- R8 full mode: shrinking, obfuscation, optimisation (keep rules for reflection/serialisation)
- App Startup library / lazy initialisation of SDKs (move third-party SDK init off the critical path)
- Macrobenchmark tests in CI for startup and scrolling
- Avoid main-thread I/O (StrictMode in debug builds)
```

See **35-android-studio-extensions.md** (Baseline Profiles section) for IDE tooling.

### 10.3 iOS performance techniques

```text
- Defer non-essential work after first frame; lazy-load SDKs
- Avoid heavy work in SwiftUI `body`; use Instruments SwiftUI templates to find excessive updates
- Image decoding off the main thread; downsample large images
- App thinning, asset catalogs, on-demand resources where appropriate
- MetricKit payloads for field launch time, hangs, and energy
```

---

## 11. Mobile Security (OWASP MASVS)

The **OWASP Mobile Application Security Verification Standard (MASVS)** and **Mobile Application Security Testing Guide (MASTG)** define controls and tests across categories:

| MASVS category | Key controls |
|---|---|
| **STORAGE** | Sensitive data only in Keychain/Keystore-backed storage; no secrets in logs, backups, screenshots, clipboard, or `UserDefaults`/`SharedPreferences` |
| **CRYPTO** | Platform crypto APIs; keys in hardware-backed stores; no hard-coded keys |
| **AUTH** | Server-side authentication/session management; biometrics bound to Keystore/Keychain keys (not just a UI gate) |
| **NETWORK** | TLS everywhere; no cleartext; correct certificate validation; pinning only with a rotation strategy |
| **PLATFORM** | Safe IPC (exported components, intents, URL schemes), WebView hardening, minimal permissions |
| **CODE** | Up-to-date dependencies, input validation, no debug code in release builds |
| **RESILIENCE** | Anti-tampering, obfuscation, root/jailbreak and debugger detection — defense in depth for high-risk apps (banking, payments), never a substitute for server checks |
| **PRIVACY** | Data minimisation, transparency, user control (MASVS-PRIVACY) |

### Key rules

```text
🔴 No secrets in the app (API keys that grant privileged access, signing keys, admin tokens) — anything shipped can be extracted
🔴 Tokens in Keychain / Keystore-backed encrypted storage; refresh tokens rotated
🔴 Validate everything server-side; the app is untrusted
🔴 Android: exported components declared explicitly (android:exported), PendingIntent immutability, no world-readable files
🔴 WebViews: disable JS/file access unless needed; never load untrusted content with JS bridges enabled
🟠 App integrity signals on the server: Play Integrity API (Android), App Attest / DeviceCheck (iOS) for high-risk flows
🟠 Disable screenshots/recents snapshots for highly sensitive screens (FLAG_SECURE / privacy overlays) where justified
🟠 Prevent sensitive data in backups (Android backup rules; iOS file protection classes / excluding from backup)
🟠 Code obfuscation (R8; Swift has limited obfuscation — rely on server-side controls)
```

Testing: MobSF for static/dynamic analysis, MASTG test cases, pentests for high-risk apps.

---

## 12. Privacy and Store Disclosures

| Obligation | Android | iOS |
|---|---|---|
| Disclose data collection | Play **Data safety** form | App Store **App Privacy** details |
| SDK data use | Audit every SDK (analytics, ads, crash reporting) | **Privacy manifests** for app and third-party SDKs; required-reason API declarations |
| Tracking consent | Comply with Play policies and local law | **ATT** prompt before tracking across apps/websites |
| Runtime permissions | Request in context, minimal set, handle denial; photo picker instead of broad media permissions | Purpose strings (`NS…UsageDescription`); limited photo library access |
| Children | Families policy; COPPA/DPDP children's data rules | Kids category rules |
| Legal | India DPDP Act consent and notices, GDPR where applicable (chapter 09 §15) | Same |

```text
🔴 Ask permissions only when the feature is used, with an explanation screen first
🔴 Keep disclosures accurate and updated when SDKs or data flows change (mismatches cause rejections and legal risk)
🟠 Consent management for analytics/marketing SDKs; respect opt-outs at SDK initialisation
```

---

## 13. Authentication

| Need | Recommendation |
|---|---|
| OAuth/OIDC login | Authorization Code + **PKCE** via the system browser (Android Custom Tabs, iOS `ASWebAuthenticationSession`) using **AppAuth** or the IdP's SDK — never embedded WebViews for third-party login |
| Passkeys | Android **Credential Manager** API; iOS **AuthenticationServices** (`ASAuthorizationPlatformPublicKeyCredentialProvider`) with associated domains |
| Biometrics | Use to unlock a Keystore/Keychain-protected credential (cryptographic binding), not as a boolean check |
| Session | Short-lived access tokens, rotating refresh tokens stored securely; handle revocation/logout everywhere |
| OTP | SMS Retriever / SMS User Consent API (Android), `oneTimeCode` text content type (iOS); never request SMS read permission for OTP |

---

## 14. Push Notifications

```text
Android: Firebase Cloud Messaging (FCM); notification channels; POST_NOTIFICATIONS runtime permission (Android 13+)
iOS:     APNs (token-based auth with .p8 key); request authorization in context; provisional authorization option
Cross-platform: FCM for both, or providers (OneSignal, Braze, Airship, Expo Notifications)
```

```text
🔴 Register/refresh device tokens on login and token rotation; delete on logout
🔴 No sensitive data in notification payloads (they appear on lock screens) — send IDs, fetch details in-app
🔴 Respect user preferences and quiet hours; separate transactional from marketing (consent)
🟠 Deep link from notification to the right screen; handle stale links gracefully
🟠 Measure delivery/open rates; handle invalid tokens from provider feedback
```

---

## 15. Deep Links and App Links

| Type | Android | iOS |
|---|---|---|
| Verified HTTPS links (preferred) | **Android App Links** with `assetlinks.json` at `/.well-known/` | **Universal Links** with `apple-app-site-association` at `/.well-known/` |
| Custom schemes | `myapp://` (not verified — can be hijacked) | Same caveat |
| Deferred deep links | Install referrer / attribution SDKs | Attribution SDKs, clipboard-free methods |

```text
🔴 Treat all deep link parameters as untrusted input (validate, authorise)
🔴 Use verified HTTPS links for anything sensitive (auth callbacks, payments)
🟠 Test links from email, SMS, browsers, QR codes, and when the app isn't installed (web fallback)
```

---

## 16. Accessibility

| Area | Android | iOS |
|---|---|---|
| Screen reader | TalkBack — `contentDescription`, Compose `semantics` | VoiceOver — `accessibilityLabel`, `.accessibilityElement(children:)` |
| Text scaling | Respect font scale (sp units); test at largest sizes | **Dynamic Type** (text styles); test accessibility sizes |
| Touch targets | ≥ 48×48 dp | ≥ 44×44 pt |
| Contrast & colour | WCAG 2.2 contrast ratios; don't rely on colour alone | Same; support Increase Contrast |
| Motion | Respect "remove animations" | Respect Reduce Motion |
| Focus order | Logical traversal order | Logical rotor/focus order |
| Testing | Accessibility Scanner, Compose UI test semantics, Espresso accessibility checks | Accessibility Inspector, XCUITest audits (`performAccessibilityAudit()`) |

Mobile apps are covered by accessibility laws in many markets; target WCAG 2.2 AA equivalents (chapter 12 §12).

---

## 17. Localisation

```text
- Externalise all strings (strings.xml / String Catalogs .xcstrings / ARB for Flutter / i18n libraries for RN)
- Plurals and gender via platform plural rules (ICU)
- Number, date, currency formatting via platform formatters with the user's locale (e.g. Indian digit grouping for INR)
- RTL support (Arabic/Urdu): mirrored layouts, start/end instead of left/right
- Fonts covering Indic scripts; test text expansion and truncation
- Per-app language preferences (Android 13+, iOS per-app language settings)
- Pseudo-localisation in debug builds
```

---

## 18. Testing

| Level | Android | iOS | Cross-platform |
|---|---|---|---|
| Unit | JUnit 5/4, Kotest, Turbine (Flow), MockK | **Swift Testing** (`@Test`, `#expect`), XCTest | Flutter `test`, Jest/Vitest (RN) |
| UI / component | Compose UI tests, Espresso, Robolectric | XCUITest, ViewInspector (SwiftUI unit) | Flutter widget tests, React Native Testing Library |
| Screenshot | Paparazzi, Roborazzi, Compose Preview Screenshot Testing | swift-snapshot-testing | golden tests (Flutter) |
| E2E | UI Automator, **Maestro**, Appium | XCUITest, Maestro, Appium | Maestro, Detox (RN), Patrol/integration_test (Flutter) |
| Performance | Macrobenchmark, Microbenchmark | XCTest performance metrics, Instruments | — |
| Device coverage | Firebase Test Lab, BrowserStack, AWS Device Farm, Sauce Labs, physical device lab | Same + Xcode Cloud | Same |

```text
Device matrix (example): low-end Android (2–4 GB RAM), mid-range Android (most common in your analytics), flagship Android,
oldest supported iOS device, current iPhone, a tablet/foldable if supported; test on slow/flaky networks and in airplane mode.
```

Pre-release testing tracks: Play **internal testing / closed testing** tracks, Apple **TestFlight**.

---

## 19. Build, Signing, and CI/CD

### 19.1 Signing

| Platform | Practice |
|---|---|
| Android | **Play App Signing**: Google holds the app signing key; you sign uploads with an **upload key** (stored in a secret manager/CI secret, backed up securely); register the upload key reset procedure |
| iOS | Distribution certificates and provisioning profiles; prefer **App Store Connect API keys** for CI; manage with Xcode automatic signing on CI, fastlane `match` (encrypted repo/storage), or Xcode Cloud |

```text
🔴 Signing keys/certs never in the repository; stored in CI secret stores/HSM-backed services with restricted access
🔴 Documented recovery procedures (lost upload key, expired certificates)
🟠 Separate bundle IDs/application IDs for dev/staging/prod builds (side-by-side installs)
```

### 19.2 Versioning

```text
Android: versionName (SemVer marketing version, e.g. 5.12.0), versionCode (monotonically increasing integer from CI)
iOS:     CFBundleShortVersionString (5.12.0), CFBundleVersion (build number, increasing per upload)
Tag releases in Git: mobile/v5.12.0 (+ build number in release notes)
```

### 19.3 CI/CD pipeline

```text
PR:      lint (ktlint/detekt, SwiftLint), unit tests, screenshot tests, build debug, static security scan
Main:    + UI tests on emulators/simulators or device farm, build release (R8), upload to internal track/TestFlight
Release: version bump, changelog/release notes, store metadata, staged rollout via API
```

| Tooling | Use |
|---|---|
| **fastlane** | Build, sign, screenshots, upload (both platforms) |
| Gradle Play Publisher | Publish to Play from Gradle |
| **Xcode Cloud** | Apple-hosted CI/CD with TestFlight integration |
| **EAS Build/Submit** (Expo) | React Native builds and store submission |
| Codemagic, Bitrise, GitHub Actions (macOS runners), GitLab CI | General mobile CI |

```ruby
# fastlane/Fastfile (excerpt)
platform :android do
  lane :internal do
    gradle(task: "bundle", build_type: "Release")
    upload_to_play_store(track: "internal", aab: lane_context[SharedValues::GRADLE_AAB_OUTPUT_PATH])
  end
end

platform :ios do
  lane :beta do
    app_store_connect_api_key(key_id: ENV["ASC_KEY_ID"], issuer_id: ENV["ASC_ISSUER_ID"], key_content: ENV["ASC_KEY_P8"])
    build_app(scheme: "ShopNow", export_method: "app-store")
    upload_to_testflight(skip_waiting_for_build_processing: true)
  end
end
```

---

## 20. Release Management

### 20.1 Release train

```text
Weekly or bi-weekly release train:
  Day 0  code freeze for train → release branch (chapter 07 §6.4)
  Day 0–2 QA on release candidate (internal track / TestFlight), regression + exploratory
  Day 2  submit for review
  Day 3+ staged rollout: Play 1% → 5% → 20% → 50% → 100% (halt on regressions); App Store phased release (7-day automatic)
Hotfix: fix on main → cherry-pick to release branch → expedited review if critical
```

### 20.2 Rollout gates

| Signal | Halt rollout if |
|---|---|
| Crash-free users | Drops below baseline by agreed threshold |
| ANR rate (Android) | Rises toward Play bad-behaviour threshold |
| Key funnel metrics | Login/checkout success rate drops |
| Backend errors from new version | Spike in 4xx/5xx by app version |
| Store reviews/support tickets | Spike related to the release |

### 20.3 OTA updates (React Native / Flutter code push)

Over-the-air JavaScript/asset updates (e.g. **Expo EAS Update**) can ship fixes without a store release **only within store rules** — no significant feature changes or changing the app's primary purpose without review, and no native code changes. Microsoft's App Center (and its CodePush service) was retired in March 2025; use maintained alternatives. Version OTA bundles against compatible native builds (runtime versions).

---

## 21. Remote Config, Feature Flags, and Forced Updates

| Capability | Use |
|---|---|
| Remote config (Firebase Remote Config, LaunchDarkly, ConfigCat, Statsig, Unleash) | Tune behaviour without a release |
| **Kill switches** | Disable a broken feature immediately |
| Gradual feature rollout | Percentage/cohort targeting |
| **Minimum supported version** | Server/config returns minimum version → app shows soft (recommended) or hard (required) update prompt; Android **In-App Updates** API (flexible/immediate) |
| Maintenance mode | Friendly message during backend maintenance |

```text
🔴 Ship every risky feature behind a remotely controlled flag with a safe default if config fails to load
🔴 Have a forced-update mechanism before you need it (security incidents, breaking backend changes)
🟠 Cache last known config; define defaults in the binary
🟠 Clean up stale flags (chapter 05 §18.2)
```

---

## 22. Observability

| Signal | Tools |
|---|---|
| Crashes & ANRs | Firebase Crashlytics, Sentry, Bugsnag, Embrace; Play Console vitals; Xcode Organizer |
| Performance | Firebase Performance Monitoring, Sentry Performance, MetricKit, Android vitals, OpenTelemetry mobile SDKs |
| Analytics | Privacy-respecting product analytics with consent |
| Logs | Remote logging with sampling and PII scrubbing (avoid shipping verbose logs) |
| Backend correlation | Send trace headers (`traceparent`) and app version with API calls |

```text
🔴 Upload mapping/dSYM files for every release (R8 mapping, iOS dSYMs) so stack traces are readable
🔴 Tag events with app version, build, OS version, device model
🔴 Dashboards per release: crash-free users, ANR rate, startup time, key funnels; alerts on regressions
```

---

## 23. Store Compliance Essentials

- [ ] Target API level (Android) and Xcode/SDK version (iOS) meet current deadlines
- [ ] Privacy disclosures (Data safety, App Privacy, privacy manifests) accurate, including SDKs
- [ ] Permissions minimal, justified, with purpose strings
- [ ] Account deletion available in-app (and via web for Play)
- [ ] Payments comply with platform rules for digital goods (in-app purchase/billing requirements and regional alternatives)
- [ ] Sign in with Apple considered where required
- [ ] Content ratings / age ratings completed
- [ ] Export compliance (encryption) answered correctly
- [ ] Support URL, privacy policy URL, contact details valid
- [ ] Test accounts provided to reviewers for login-gated apps
- [ ] Financial/loan app declarations (if applicable, including India-specific requirements)

---

## 24. In-App Purchases and Payments

| Scenario | Typical requirement (verify current store policies and regional rules) |
|---|---|
| Digital goods/subscriptions consumed in the app | Platform billing (Google Play Billing, Apple In-App Purchase / StoreKit 2), with regional exceptions and alternative billing programmes in some markets |
| Physical goods and real-world services (e-commerce, food delivery, rides) | Your own payment provider (cards, UPI, netbanking, wallets) is allowed |
| Reader apps / external links | Special entitlements and rules vary by platform and region |

```text
🔴 Validate purchases server-side (Google Play Developer API / App Store Server API + App Store Server Notifications V2; Real-time developer notifications for Play)
🔴 Entitlements are owned by your backend, keyed to the user account — not to local flags
🔴 Handle refunds, revocations, grace periods, billing retries, and subscription upgrades/downgrades
🟠 Use StoreKit 2 (iOS) and the latest Play Billing Library version required by Play
🟠 UPI intent/collect flows via PSP SDKs: verify final payment status server-side (webhook + status API), never trust the app callback alone
🟠 Test with sandbox accounts, license testers, and StoreKit configuration files
```

---

## 25. Mobile Backend Considerations

| Concern | Practice |
|---|---|
| **Version skew** | Old app versions call your APIs for months/years — evolve APIs additively, keep deprecated fields until usage drops (chapter 11 §11) |
| **Mobile BFF** | A backend-for-frontend aggregates calls and shapes payloads for small screens and slow networks |
| **Client identification** | Send `app version`, `build`, `platform`, `OS version` headers; log them; use them for compatibility decisions and minimum-version enforcement |
| **Payload efficiency** | Pagination, field selection, compression, image CDNs with device-appropriate sizes |
| **Idempotency** | Mobile networks drop responses — clients retry; mutations need idempotency keys (chapter 11 §7) |
| **Push fan-out** | Queue-based notification service; token cleanup from provider feedback |
| **Feature flags** | Server-evaluated flags or SDKs with offline defaults (§21) |
| **Rate limits** | Per user/device; protect OTP and login endpoints from abuse |
| **Observability** | API metrics segmented by app version and platform — spot a bad release quickly |

```http
GET /v1/orders?limit=20
User-Agent: ShopNow/5.12.0 (Android 16; Pixel 9; build 51200)
X-App-Version: 5.12.0
X-App-Build: 51200
X-Platform: android
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
```

```json
// Example config endpoint response consumed at app start
{
  "minSupportedVersion": { "android": "5.6.0", "ios": "5.6.0" },
  "recommendedVersion": { "android": "5.12.0", "ios": "5.12.0" },
  "flags": { "checkout-payment-retry": true, "new-search": false },
  "maintenance": { "active": false, "message": null }
}
```

---

## 26. Scenario Playbooks

### 26.1 New consumer app (India-first, low-end devices)
Native Android first (Kotlin/Compose) or Flutter/RN for both platforms; aggressive app size budget; offline-first caching; UPI intent payments via PSP SDKs; Hindi + regional language support; test on 2–4 GB RAM devices and 3G/4G throttling; Play staged rollouts; Crashlytics + Android vitals monitoring.

### 26.2 Banking / fintech app
Native apps with KMP-shared logic; MASVS L2 + resilience controls; Play Integrity / App Attest checked server-side; device binding; biometric-bound keys; screen capture protection on sensitive screens; regulator-driven requirements (e.g. RBI guidelines in India); pentests every major release; forced update mechanism.

### 26.3 Existing web company adding mobile
React Native with Expo sharing TypeScript models and API clients with the web app; BFF for mobile-specific endpoints; EAS Build/Submit/Update; native modules only where needed.

### 26.4 Enterprise internal app
Managed distribution (Managed Google Play, Apple Business Manager / custom apps); MDM/EMM policies; SSO with enterprise IdP; app config via managed configurations.

---

## 27. Checklists

### Architecture & code
- [ ] Layered architecture with UDF; DB as single source of truth for offline data
- [ ] Modularised by feature; version catalog/convention plugins (Android); Swift packages (iOS)
- [ ] Coroutines/Swift concurrency structured and cancellable; no main-thread I/O
- [ ] Network client with timeouts, auth refresh, min-version handling, trace headers
- [ ] Secure storage for tokens; no secrets in binary

### Quality & performance
- [ ] Unit + UI + screenshot tests in CI; E2E for critical journeys on real/emulated devices
- [ ] Baseline Profiles (Android); startup and jank measured with benchmarks
- [ ] Crash-free and ANR targets defined; alerts configured
- [ ] App size budget tracked per release
- [ ] Accessibility: TalkBack/VoiceOver pass, large fonts, contrast, touch targets

### Release
- [ ] Store requirements verified (target API, Xcode version, privacy disclosures)
- [ ] Signing keys secured; CI signing automated
- [ ] Release notes, store metadata, screenshots updated
- [ ] Staged/phased rollout with halt criteria
- [ ] Mapping files/dSYMs uploaded
- [ ] Kill switches and forced-update path verified
- [ ] Backend compatible with all supported app versions

---

## 28. References

### Android
- Android Developers: https://developer.android.com/
- Guide to app architecture: https://developer.android.com/topic/architecture
- Target API level requirements (Google Play): https://developer.android.com/google/play/requirements/target-sdk
- Play Console Help — target API policy: https://support.google.com/googleplay/android-developer/answer/11926878
- Jetpack Compose: https://developer.android.com/compose
- Baseline Profiles: https://developer.android.com/topic/performance/baselineprofiles/overview
- Android vitals: https://developer.android.com/topic/performance/vitals
- Credential Manager (passkeys): https://developer.android.com/identity/sign-in/credential-manager
- Play Integrity API: https://developer.android.com/google/play/integrity
- Now in Android (reference app): https://github.com/android/nowinandroid

### Apple
- Apple Developer documentation: https://developer.apple.com/documentation/
- Upcoming requirements (Xcode/SDK minimums): https://developer.apple.com/news/upcoming-requirements/
- App Store Review Guidelines: https://developer.apple.com/app-store/review/guidelines/
- Privacy manifest files: https://developer.apple.com/documentation/bundleresources/privacy-manifest-files
- Human Interface Guidelines: https://developer.apple.com/design/human-interface-guidelines/
- Swift concurrency: https://docs.swift.org/swift-book/documentation/the-swift-programming-language/concurrency/
- Swift Testing: https://developer.apple.com/xcode/swift-testing/
- App Attest: https://developer.apple.com/documentation/devicecheck/establishing-your-app-s-integrity

### Cross-platform
- Kotlin Multiplatform: https://kotlinlang.org/docs/multiplatform.html
- Compose Multiplatform: https://www.jetbrains.com/compose-multiplatform/
- Flutter: https://docs.flutter.dev/
- React Native: https://reactnative.dev/ · Expo: https://docs.expo.dev/

### Security and tooling
- OWASP MASVS: https://mas.owasp.org/MASVS/ · MASTG: https://mas.owasp.org/MASTG/
- MobSF: https://github.com/MobSF/Mobile-Security-Framework-MobSF
- AppAuth: https://appauth.io/
- fastlane: https://docs.fastlane.tools/
- Maestro: https://maestro.mobile.dev/

---

**Previous:** [13 — Backend Engineering](./13-backend-engineering.md) · **Next:** [15 — Desktop Engineering](./15-desktop-engineering.md)