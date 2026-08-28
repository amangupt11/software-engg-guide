# Manual Package-Rename Guide — `com.pinaak` → `in.urbancruise`

Step-by-step, tailored to the **actual current state** of this repo (React Native 0.86.2, Kotlin Android, iOS xcodeproj not workspace, `@react-native-firebase` present).

> **What this guide changes**
> - Android `applicationId` + `namespace`: `com.pinaak` → `in.urbancruise`
> - Android Java-source folder: `.../java/com/pinaak/` → `.../java/in/urbancruise/`
> - iOS `PRODUCT_BUNDLE_IDENTIFIER`: `org.reactjs.native.example.pinaak` (placeholder) → `in.urbancruise`
> - iOS display name (`CFBundleDisplayName`): `pinaak` → `Urban Cruise`
>
> **What this guide keeps unchanged (deliberately)**
> - The internal project name `pinaak` — used by `settings.gradle`, `app.json`, `package.json`, the iOS folder `ios/pinaak/`, and the `ios/pinaak.xcodeproj`. Changing these too is a bigger, riskier diff — covered in the **Optional** section at the end.
> - Android `app_name` in `strings.xml` is already `"Urban Cruise"`, so no change there.

---

## 0. Pre-flight

Do all of these before touching a single file.

1. **Commit or stash any pending work.** You want a clean tree so the rename diff is inspectable.
   ```bash
   git status                     # must be clean
   git checkout -b rename/in.urbancruise
   ```
2. **Close Xcode and Android Studio.** Both cache absolute paths inside `.xcworkspace/xcuserdata`, `DerivedData`, and `.idea/`. Editing gradle/pbxproj files while they're open causes corruption.
3. **Uninstall the app** from any physical device / simulator / emulator you've previously installed it on. Android and iOS treat the new package/bundle as a *different app*; the old install stays and eats disk.
4. **Stop the Metro bundler** (`Ctrl+C` in whichever terminal is running `npm start`).
5. Note down the current values so you can grep for stragglers later:
   - Android package: `com.pinaak`
   - iOS bundle: `org.reactjs.native.example.pinaak`

---

## 1. JavaScript / TypeScript layer

The JS side barely couples to the package name — but there's one file to touch and one to verify.

### 1.1 (Optional but recommended) Rebrand `displayName`

**File:** `app.json`

```diff
 {
   "name": "pinaak",
-  "displayName": "pinaak"
+  "displayName": "Urban Cruise"
 }
```

> Leave `"name": "pinaak"` alone for now. `index.js` uses `AppRegistry.registerComponent(appName, () => App)` where `appName` comes from this field, and the native side registers the same string via `MainActivity.getMainComponentName()`. Changing it here means we must change it there too (see Optional section).

### 1.2 Verify nothing else references the package literal

```bash
grep -RIn "com.pinaak" src/ App.tsx index.js __tests__/  # should return nothing
grep -RIn "org.reactjs.native.example" src/ App.tsx      # should return nothing
```

Nothing app-code depends on the package name today. Good.

---

## 2. Android — the multi-file dance

The trickiest platform. Six touchpoints: two config files, two Kotlin files, and a folder rename.

### 2.1 Update `applicationId` and `namespace`

**File:** `android/app/build.gradle`

```diff
 android {
-    namespace "com.pinaak"
+    namespace "in.urbancruise"
     defaultConfig {
-        applicationId "com.pinaak"
+        applicationId "in.urbancruise"
         ...
     }
 }
```

Both must change. `namespace` controls the R-class package (compile-time); `applicationId` is the store-facing identity (install-time).

### 2.2 Rename the Kotlin source folder

Current structure:
```
android/app/src/main/java/com/pinaak/
    ├── MainActivity.kt
    └── MainApplication.kt
```

Target structure:
```
android/app/src/main/java/in/urbancruise/
    ├── MainActivity.kt
    └── MainApplication.kt
```

From the repo root:

```bash
cd android/app/src/main/java
mkdir -p in/urbancruise
git mv com/pinaak/MainActivity.kt   in/urbancruise/MainActivity.kt
git mv com/pinaak/MainApplication.kt in/urbancruise/MainApplication.kt
rmdir com/pinaak com                    # remove now-empty old dirs
cd -                                     # back to repo root
```

> Using `git mv` (instead of a bare `mv`) preserves file history — reviewers see a rename, not a delete+add.

### 2.3 Update the `package` declaration inside the Kotlin files

**File:** `android/app/src/main/java/in/urbancruise/MainActivity.kt`
```diff
-package com.pinaak
+package in.urbancruise

 import com.facebook.react.ReactActivity
 ...
```

**File:** `android/app/src/main/java/in/urbancruise/MainApplication.kt`
```diff
-package com.pinaak
+package in.urbancruise

 import android.app.Application
 ...
```

### 2.4 Verify `AndroidManifest.xml`

**File:** `android/app/src/main/AndroidManifest.xml`

Modern AGP builds (this project uses one) put the package in `build.gradle`'s `namespace` — the manifest **should not** have a `package="..."` attribute on `<manifest>`. Confirm:

```bash
grep -n 'package=' android/app/src/main/AndroidManifest.xml
```

The only matches should be `<package android:name="com.whatsapp" />` style entries inside `<queries>` — those are **third-party** packages the app can query via `canOpenURL`, **not** your own package. Leave them alone.

If (unexpectedly) you find `package="com.pinaak"` on the `<manifest>` root element, delete just that attribute — the namespace in gradle wins.

### 2.5 Sanity-check nothing else in `android/` references the old package

```bash
grep -RIn "com\\.pinaak" android/ \
    --exclude-dir=build \
    --exclude-dir=.gradle \
    --exclude-dir=.cxx
```

Expected output: **nothing.** If anything shows up (e.g. in a proguard file, a `strings.xml`, or `link-assets-manifest.json`), review and update it. In a stock RN project there won't be anything.

### 2.6 Clean the Android build cache

```bash
cd android
./gradlew clean
cd ..
```

This wipes the previously-generated R classes and BuildConfig that hard-coded `com.pinaak`.

---

## 3. iOS — Xcode's project file

Two things to change: the bundle identifier (in `project.pbxproj`) and the display name (in `Info.plist`).

### 3.1 Update `PRODUCT_BUNDLE_IDENTIFIER`

**File:** `ios/pinaak.xcodeproj/project.pbxproj`

There are **two** occurrences — one under the `Debug` build configuration, one under `Release`. Both must change.

```diff
-PRODUCT_BUNDLE_IDENTIFIER = "org.reactjs.native.example.$(PRODUCT_NAME:rfc1034identifier)";
+PRODUCT_BUNDLE_IDENTIFIER = "in.urbancruise";
```

Two acceptable ways to do this:

**Option A — Open Xcode** (safer, no risk of malforming pbxproj):
1. `open ios/pinaak.xcodeproj` (do **not** try to open a workspace — this project doesn't have one; Podfile is present but CocoaPods generates `.xcworkspace` only after `pod install`).
2. Select the `pinaak` project in the navigator → target `pinaak` → **Signing & Capabilities** tab.
3. Change **Bundle Identifier** to `in.urbancruise`. Xcode applies to both Debug and Release automatically.
4. Save (`Cmd+S`) and quit Xcode.

**Option B — Edit `project.pbxproj` directly** (faster but pbxproj syntax is unforgiving):
```bash
sed -i.bak \
  's|PRODUCT_BUNDLE_IDENTIFIER = "org\.reactjs\.native\.example\.$(PRODUCT_NAME:rfc1034identifier)"|PRODUCT_BUNDLE_IDENTIFIER = "in.urbancruise"|g' \
  ios/pinaak.xcodeproj/project.pbxproj
rm ios/pinaak.xcodeproj/project.pbxproj.bak
```
Verify:
```bash
grep PRODUCT_BUNDLE_IDENTIFIER ios/pinaak.xcodeproj/project.pbxproj
# → both lines should now show "in.urbancruise"
```

### 3.2 Update `CFBundleDisplayName`

**File:** `ios/pinaak/Info.plist`

```diff
 <key>CFBundleDisplayName</key>
-<string>pinaak</string>
+<string>Urban Cruise</string>
```

> `CFBundleName` and `CFBundleIdentifier` are already set to `$(PRODUCT_NAME)` and `$(PRODUCT_BUNDLE_IDENTIFIER)` respectively — they pick up the pbxproj change automatically. Only `CFBundleDisplayName` is hard-coded.

### 3.3 Reinstall Pods

```bash
cd ios
rm -rf Pods build Podfile.lock
pod install
cd ..
```

Necessary because CocoaPods generates `Pods-pinaak-{acknowledgements,dummy,etc}` files that reference the previous target settings; a clean `pod install` re-derives them.

---

## 4. Firebase — regenerate config

This project depends on `@react-native-firebase/app`, `messaging`, and `crashlytics`. Firebase identifies apps by their platform-specific identifier.

**You must:**
1. Sign in to https://console.firebase.google.com → your Pinaak project.
2. **Project Settings → General → Your apps.**
3. Either:
   - **Add new apps** with `in.urbancruise` for Android and `in.urbancruise` for iOS (recommended if this is a real relaunch — cleanest audit trail), **or**
   - Delete the `com.pinaak` / `org.reactjs.native.example.pinaak` app entries and re-add.
4. Download the fresh config files and drop them at:
   - Android: `android/app/google-services.json`
   - iOS: `ios/pinaak/GoogleService-Info.plist` (drag into Xcode target so it gets copied into the app bundle)
5. If you use **Crashlytics dSYM upload** or **FCM APNs certificates**, re-configure those against the new bundle id in the console.

> Neither `google-services.json` nor `GoogleService-Info.plist` is currently committed to the repo, so Firebase presumably isn't wired up yet — but the moment someone tries to wire it, they must use the new identifier.

---

## 5. Clean everything and rebuild

Order matters. Cached artefacts from before the rename will crash the build in creative ways.

```bash
# 1. JS caches
rm -rf node_modules
npm install                              # or yarn — match the lockfile
watchman watch-del-all 2>/dev/null       # ok if watchman isn't installed
rm -rf $TMPDIR/metro-* $TMPDIR/haste-*   # macOS Metro cache

# 2. Android caches
cd android && ./gradlew clean && cd ..

# 3. iOS caches
cd ios
rm -rf build ~/Library/Developer/Xcode/DerivedData/pinaak-*
pod install
cd ..

# 4. Start Metro fresh
npm start -- --reset-cache               # leave running, open a new terminal for step 5
```

New terminal:

```bash
# Android
npm run android

# iOS (once Android confirmed working)
npm run ios
```

---

## 6. Verification checklist

Tick each after the rebuild:

- [ ] `grep -RIn "com\\.pinaak" android/ --exclude-dir=build --exclude-dir=.gradle` → empty.
- [ ] `grep -RIn "org\\.reactjs\\.native\\.example" ios/` → empty.
- [ ] App icon on Android launcher reads **Urban Cruise**.
- [ ] App icon on iOS home screen reads **Urban Cruise**.
- [ ] `adb shell pm list packages | grep urbancruise` → `package:in.urbancruise`.
- [ ] iOS Settings → General → iPhone Storage → app entry shows bundle `in.urbancruise` (or check via Xcode → Window → Devices and Simulators).
- [ ] `npm test` passes (`__tests__/App.test.tsx` doesn't touch package name — should be unaffected).
- [ ] Login → OTP → role selection → dashboard flow works (proves bootstrap orchestrator, Keychain, MMKV, and axios interceptors survived the rename).

---

## 7. Rollback (if things go wrong)

Because you branched in step 0.1:

```bash
git checkout main             # or whichever base you branched from
git branch -D rename/in.urbancruise
cd android && ./gradlew clean && cd ..
cd ios && rm -rf Pods build Podfile.lock && pod install && cd ..
```

Then reinstall the app on your device.

---

## Optional — full project rename (`pinaak` → `urbancruise`)

Only do this if you want the internal names consistent too. Higher risk because renaming the iOS `.xcodeproj` folder confuses many tools.

**Extra touchpoints:**

| File | Change |
|---|---|
| `package.json` | `"name": "pinaak"` → `"name": "urbancruise"` |
| `app.json` | `"name": "pinaak"` → `"name": "urbancruise"` |
| `android/settings.gradle` | `rootProject.name = 'pinaak'` → `= 'urbancruise'` |
| `android/app/src/main/java/in/urbancruise/MainActivity.kt` | `getMainComponentName()` returns `"pinaak"` → `"urbancruise"` **(must match `app.json` "name" exactly)** |
| `ios/pinaak/` folder | rename to `ios/urbancruise/` |
| `ios/pinaak.xcodeproj/` folder | rename to `ios/urbancruise.xcodeproj/` |
| `ios/Podfile` | `target 'pinaak' do` → `target 'urbancruise' do` |
| Inside `project.pbxproj` | every `pinaak` reference → `urbancruise` (Xcode's project navigator right-click → Rename does this correctly; sed will break it) |

> Recommendation: do the identifier rename (this main guide) first, verify it works end-to-end, ship or land it, **then** do the internal-name rename in a separate PR. Two small reviewable diffs beat one large scary one.
