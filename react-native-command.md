# React Native Community CLI - Command Notes

> Project: Pinaak
> Framework: React Native Community CLI
> React Native: 0.86.2

---

# 1. Environment Verification

## Check React Native Environment

```bash
npx react-native doctor
```

### Purpose

- Verifies development environment
- Checks Node.js
- Checks Java (JDK)
- Checks Android SDK
- Checks Android Studio
- Checks ADB
- Detects missing dependencies

### When to use

- First project setup
- After installing Android Studio
- After updating SDK/JDK
- When build fails

---

## Show Environment Information

```bash
npx react-native info
```

### Purpose

Displays:

- React Native version
- React version
- Node version
- npm version
- Java version
- Android SDK
- Hermes status
- New Architecture status

### When to use

- Debugging
- Bug reports
- Sharing environment details

---

# 2. Metro Bundler

## Start Metro

```bash
npm start
```

or

```bash
npx react-native start
```

### Purpose

Starts the Metro Bundler.

### When to use

Every time before running the app.

---

## Start Metro with Clean Cache

```bash
npm start -- --reset-cache
```

or

```bash
npx react-native start --reset-cache
```

### Purpose

Clears Metro cache.

### Use only when

- Strange errors
- Module not found
- Old bundle cached
- React Native upgrade
- Dependency issues

Do NOT use every day.

---

# 3. Android Development

## Run Android App

```bash
npm run android
```

or

```bash
npx react-native run-android
```

### Purpose

- Builds the Android app
- Installs APK
- Launches application

### When to use

Daily development.

---

## Check Connected Devices

```bash
adb devices
```

Expected

```text
List of devices attached

emulator-5554    device
```

---

## List Android Emulators

```bash
emulator -list-avds
```

Example

```text
Pixel_9_API_35
```

---

## Start Emulator

```bash
emulator -avd Pixel_9_API_35
```

Replace with your emulator name.

---

# 4. Package Management

## Install Dependencies (Recommended)

```bash
npm ci
```

### Purpose

- Clean installation
- Uses package-lock.json
- Reproducible builds

### Use

- First setup
- CI/CD
- Fresh clone

---

## Install New Package

```bash
npm install <package-name>
```

Example

```bash
npm install axios
```

Use only after researching compatibility.

---

## Remove Package

```bash
npm uninstall <package-name>
```

Example

```bash
npm uninstall axios
```

---

## Update package-lock

```bash
npm install
```

Only when intentionally changing dependencies.

---

# 5. Code Quality

## Lint

```bash
npm run lint
```

Checks

- JavaScript
- TypeScript
- Coding style
- Best practices

Run before every commit.

---

## Tests

```bash
npm test
```

Runs Jest tests.

---

# 6. Android Build

## Clean Gradle

```bash
cd android
gradlew clean
cd ..
```

Use when Gradle build is corrupted.

---

## Clean Metro Cache

```bash
npm start -- --reset-cache
```

---

# 7. Git Commands

## Clone Repository

```bash
git clone <repository-url>
```

---

## Check Status

```bash
git status
```

---

## Create Feature Branch

```bash
git checkout -b feature/login
```

---

## Pull Latest Changes

```bash
git pull
```

---

## Commit

```bash
git commit -m "feat: add login screen"
```

---

# 8. Useful Windows Commands

## Check Java

```powershell
java --version
```

---

## Check Java Compiler

```powershell
javac --version
```

---

## Check JAVA_HOME

```powershell
$env:JAVA_HOME
```

---

## Check Android SDK

```powershell
$env:ANDROID_HOME
```

---

## Check SDK Root

```powershell
$env:ANDROID_SDK_ROOT
```

---

## Check Node

```powershell
node -v
```

---

## Check npm

```powershell
npm -v
```

---

## Check ADB

```powershell
adb --version
```

---

## Check Emulator

```powershell
emulator -version
```

---

# 9. Daily Development Workflow

```text
1. Open Android Studio

↓

2. Start Emulator

↓

3. adb devices

↓

4. npm start

↓

5. Open Second Terminal

↓

6. npm run android

↓

7. Start Coding 🚀
```

---

# 10. Troubleshooting Workflow

```text
Build Error

↓

npx react-native doctor

↓

adb devices

↓

npm start -- --reset-cache

↓

gradlew clean

↓

npm run android
```

---

# 11. Command Summary

| Command | Purpose |
|----------|---------|
| `npx react-native doctor` | Verify development environment |
| `npx react-native info` | Show React Native environment info |
| `npm start` | Start Metro Bundler |
| `npm start -- --reset-cache` | Start Metro and clear cache |
| `npm run android` | Build and run Android app |
| `adb devices` | List connected devices |
| `emulator -list-avds` | List Android emulators |
| `emulator -avd <name>` | Start Android emulator |
| `npm ci` | Install dependencies from lockfile |
| `npm install <package>` | Install a new package |
| `npm uninstall <package>` | Remove a package |
| `npm run lint` | Run ESLint |
| `npm test` | Run Jest tests |
| `gradlew clean` | Clean Android Gradle build |

---

# Best Practices

✅ Use `npm ci` for fresh installations.

✅ Start Metro before running the app.

✅ Keep the emulator running while developing.

✅ Run `npm run lint` before committing.

✅ Run `npm test` before opening a Pull Request.

✅ Use `--reset-cache` only when needed.

✅ Use `npx react-native doctor` after environment changes.

✅ Keep React Native, React, and CLI versions aligned.
