# PINAAK

> Production-grade React Native Application

---

# Project Information

| Item | Value |
|------|-------|
| Project | PINAAK |
| React Native | 0.86.2 |
| React | 19.2.3 |
| Language | TypeScript |
| Platform | Android + iOS |
| Architecture | Component Based |
| State Management | Redux Toolkit |
| Navigation | bottomTab + stack + drawer |

---

# Philosophy

This project follows these principles:

- Keep dependencies minimal
- Write clean and maintainable code
- Build reusable components
- Separate UI from Business Logic
- Follow scalable folder structure
- Security first
- Easy future upgrades
- Easy onboarding for new developers

---

# Folder Structure

```
src/
│
├── api/
├── assets/
│   ├── fonts/
│   ├── icons/
│   └── images/
│
├── components/
│
├── config/
│
├── constants/
│
├── hooks/
│
├── navigation/
│
├── screens/
│
├── services/
│
├── store/
│
├── theme/
│
├── types/
│
├── utils/
│
└── App.tsx
```

---

# Development Rules

## 1. Never Put Everything Inside Screens

Bad

```
HomeScreen
    API
    Storage
    Validation
    Business Logic
```

Good

```
HomeScreen
      ↓
Service
      ↓
API
```

---

## 2. Reusable Components

Create reusable components.

Example

```
Button
Input
Header
Loader
Card
Avatar
Modal
```

Avoid duplicate code.

---

## 3. Business Logic

Never write business logic inside UI.

UI should only display data.

---

## 4. API

Never call APIs directly from Screens.

Instead

```
Screen

↓

Service

↓

Axios

↓

Backend
```

---

## 5. Constants

Never hardcode

Bad

```ts
color: "#FF9900"
```

Good

```ts
color: COLORS.primary
```

---

## 6. Theme

Everything should come from theme.

```
theme/
    colors.ts
    fonts.ts
    spacing.ts
    radius.ts
```

---

## 7. Images

```
assets/images/
```

---

## 8. Icons

```
assets/icons/
```

Prefer SVG whenever possible.

---

## 9. Fonts

```
assets/fonts/
```

---

## 10. Environment

Never hardcode

- API URL
- Keys
- Version

Use

```
.env
```

or

```
config/

dev.ts
staging.ts
production.ts
```

---

# Naming Convention

## Components

```
UserCard.tsx
LoginButton.tsx
```

PascalCase

---

## Screens

```
HomeScreen.tsx
LoginScreen.tsx
```

---

## Hooks

```
useAuth.ts
useTheme.ts
```

---

## Services

```
authService.ts
userService.ts
```

---

## Utils

```
formatDate.ts
capitalize.ts
```

---

## Types

```
User.ts
Product.ts
```

---

# State Management

Prefer

- Zustand

Large projects

- Redux Toolkit

Avoid mixing multiple state libraries.

---

# Navigation

Keep navigation separate.

```
navigation/

RootNavigator.tsx

AuthNavigator.tsx

MainNavigator.tsx
```

---

# Security Checklist

Never

❌ Store passwords

❌ Store API Secret

❌ Store Private Keys

Use

- Secure Storage
- HTTPS
- Token Refresh
- Session Timeout

---

# Performance Checklist

Always

✓ React.memo

✓ useMemo

✓ useCallback

✓ FlatList

✓ Lazy Loading

✓ Image Optimization

Avoid

❌ Large Components

❌ Unnecessary Renders

❌ Huge Images

---

# Error Handling

Handle

- Network Error
- Timeout
- Unauthorized
- Offline
- Invalid Response
- Unknown Error

Never allow app crashes.

---

# Logging

Development

```
console.log()
```

Production

Use

- Crashlytics
- Sentry

Remove debug logs before release.

---

# Dependency Rules

Before installing any package ask

1. Do I really need it?

2. Is it actively maintained?

3. Compatible with current React Native?

4. Supports New Architecture?

5. Is there a better alternative?

Never install unnecessary libraries.

---

# Git Rules

Never Commit

```
node_modules/

android/build/

ios/build/

.env

*.keystore
```

---

# Build Checklist

Android

- App Icon
- Splash Screen
- Release Keystore
- ProGuard
- Version Code
- Version Name

iOS

- App Icon
- Launch Screen
- Provisioning Profile
- Distribution Certificate

---

# Before Every Commit

✔ App builds

✔ No warnings

✔ No console.log

✔ ESLint passes

✔ TypeScript passes

✔ Code formatted

---

# Before Release

✔ Test Android

✔ Test iPhone

✔ Test Tablet

✔ Test Offline

✔ Test Slow Internet

✔ Test Login

✔ Test Logout

✔ Test Crash Recovery

✔ Version Updated

✔ Release Notes

---

# Future Packages

Navigation

```
@react-navigation/native
```

Animation

```
react-native-reanimated
```

Gesture

```
react-native-gesture-handler
```

API

```
axios
```

Storage

```
@react-native-async-storage/async-storage
```

Forms

```
react-hook-form
```

Validation

```
zod
```

State

```
zustand
```

---

# Things To Remember

- Keep code simple.
- Write reusable code.
- Avoid duplicate code.
- Prefer composition over inheritance.
- Never over-engineer.
- Keep dependencies minimal.
- Test before every release.
- Document important decisions.
- Always think about future maintainability.

---

# Personal Project Checklist

- [ ] Project Created
- [ ] Folder Structure Ready
- [ ] App Icon Added
- [ ] Splash Screen Added
- [ ] Theme Setup
- [ ] Navigation Setup
- [ ] State Management
- [ ] API Layer
- [ ] Authentication
- [ ] Secure Storage
- [ ] Environment Variables
- [ ] Error Handling
- [ ] Logging
- [ ] Crash Reporting
- [ ] Push Notifications
- [ ] Deep Linking
- [ ] Release Build
- [ ] Play Store Upload
- [ ] App Store Upload

---

# Goal

Build an application that is

- Reliable
- Secure
- Scalable
- Maintainable
- Easy to Upgrade
- Production Ready

---

**Last Updated:** YYYY-MM-DD

# React Native Production Checklist

> This document serves as a long-term guide for building and maintaining a production-grade React Native application. Follow these principles to ensure reliability, scalability, maintainability, performance, and security.

---

# 1. Start with the Latest Stable Versions

Always use:

- Latest stable React Native
- Latest stable React
- Latest stable Android SDK
- Latest stable Xcode
- Latest stable Node.js LTS
- TypeScript

### Avoid

- Beta versions
- Release candidates (RC)
- Experimental packages in production

### Why?

- Better security
- Long-term support
- Easier upgrades
- Community support

---

# 2. Keep Dependencies Minimal

Every dependency increases:

- Bugs
- Security risks
- APK/IPA size
- Maintenance effort
- Upgrade complexity

### Before Installing Any Package

Ask yourself:

- Can React Native already do this?
- Is the package actively maintained?
- Is it compatible with the current React Native version?
- Does it support the New Architecture (Fabric/TurboModules)?
- Does it have a large community?

### Goal

A production application should use only the dependencies it truly needs.

---

# 3. Use TypeScript from Day One

Avoid JavaScript for medium or large projects.

### Benefits

- Compile-time error checking
- Safer refactoring
- Better IntelliSense
- Fewer runtime crashes
- Easier collaboration

---

# 4. Follow a Scalable Folder Structure

Example:

```
src/
│
├── api/
├── assets/
├── components/
├── config/
├── constants/
├── hooks/
├── navigation/
├── screens/
├── services/
├── store/
├── theme/
├── types/
├── utils/
└── App.tsx
```

### Avoid

Putting everything inside one folder.

---

# 5. Separate Business Logic from UI

❌ Bad

```
Screen
 ├── API
 ├── Storage
 ├── Validation
 └── UI
```

✅ Good

```
Screen
      ↓
Service
      ↓
API
```

### Rule

Screens should display data, not contain business logic.

---

# 6. Centralize Configuration

Never hardcode:

- API URLs
- API Keys
- Feature Flags
- App Version
- Environment Values

Use:

```
config/
    dev.ts
    staging.ts
    production.ts
```

or

```
.env
```

---

# 7. Use Proper Navigation

Prefer React Navigation.

Structure:

```
navigation/

RootNavigator.tsx
AuthNavigator.tsx
MainNavigator.tsx
```

Avoid one massive navigator file.

---

# 8. Plan State Management

Avoid excessive prop drilling.

Choose one solution:

- Zustand (Recommended)
- Redux Toolkit
- Context API (Small Apps)

Do not mix multiple state management libraries unless necessary.

---

# 9. Build an API Layer

Never call APIs directly from Screens.

Architecture:

```
Screen
      ↓
Service
      ↓
Axios
      ↓
Backend
```

Example:

```
services/
    authService.ts
    userService.ts
```

---

# 10. Handle Errors Properly

Always handle:

- Network failures
- Server errors
- Timeouts
- Offline mode
- Invalid JSON
- Token expiration

Never allow the application to crash due to an API response.

---

# 11. Logging Strategy

Development

```
console.log()
```

Production

- Remove debug logs
- Use Crashlytics
- Use Sentry
- Record meaningful errors

---

# 12. Security

Never store:

- Passwords
- API Secrets
- Private Keys

Store sensitive data securely.

Follow:

- HTTPS only
- Certificate Pinning (High Security Apps)
- Input Validation
- Sanitize User Input
- Secure Token Storage

---

# 13. Authentication

Plan for:

- Access Token
- Refresh Token
- Auto Login
- Logout
- Token Refresh
- Session Expiration

---

# 14. Offline Support

Decide whether the app should:

- Work Offline
- Cache Responses
- Retry Requests
- Synchronize Later

---

# 15. Performance

Avoid:

- Unnecessary re-renders
- Large FlatLists without optimization
- Huge images
- Heavy calculations during rendering

Use:

- React.memo
- useMemo
- useCallback
- FlatList
- Image Optimization

---

# 16. Theme Support

Centralize:

```
theme/
    colors.ts
    fonts.ts
    spacing.ts
```

Support:

- Light Theme
- Dark Theme

---

# 17. Consistent Styling

Avoid magic numbers.

❌

```
padding: 17
```

✅

```
padding: spacing.md
```

---

# 18. Icons

Store icons in one place.

```
assets/icons/
```

Prefer:

- SVG

Instead of:

- PNG (when scalable graphics are appropriate)

---

# 19. Fonts

Store fonts in:

```
assets/fonts/
```

Use:

- Consistent font families
- Consistent font weights

---

# 20. Environment Separation

Maintain separate environments.

- Development
- Staging
- Production

Never connect Production apps to Development APIs.

---

# 21. Linting & Formatting

Always use:

- ESLint
- Prettier

Enable:

- Auto format on save

---

# 22. Git Workflow

Recommended `.gitignore`

```
node_modules/
android/.gradle/
android/build/
ios/build/
.env
*.keystore
```

Never commit:

- Secrets
- Generated files
- Build artifacts

---

# 23. Testing

Minimum testing:

- Unit Tests
- Android Testing
- iOS Testing
- Multiple Screen Sizes
- Different Android Versions
- Different iOS Versions

---

# 24. Build Configuration

Android

- Release Signing
- ProGuard / R8
- Version Code
- Version Name

iOS

- Release Configuration
- Provisioning Profile
- Distribution Certificate

---

# 25. CI/CD

Automate:

- Lint
- Tests
- Android Builds
- iOS Builds

Recommended Tools

- GitHub Actions
- Bitrise
- Codemagic
- Azure DevOps

---

# 26. Monitoring

Track:

- Crashes
- ANRs
- API Failures
- Startup Time
- Performance Metrics

Recommended

- Firebase Crashlytics
- Sentry

---

# 27. Dependency Management

Update dependencies carefully.

Always:

- Read changelogs
- Test upgrades
- Upgrade gradually
- Pin versions

Avoid upgrading everything at once.

---

# 28. Documentation

Maintain:

- README.md
- Setup Guide
- Architecture Overview
- Folder Structure
- API Documentation
- Release Process
- Coding Standards
- Troubleshooting Guide

Good documentation reduces onboarding time and future maintenance effort.

---

# Golden Rules

✅ Keep dependencies minimal

✅ Prefer TypeScript

✅ Separate UI from business logic

✅ Write reusable components

✅ Never hardcode configuration

✅ Handle every error gracefully

✅ Keep code clean and readable

✅ Optimize before releasing

✅ Test thoroughly

✅ Document important decisions

---

# Long-Term Goal

Build an application that is:

- Reliable
- Secure
- Scalable
- Maintainable
- Performant
- Easy to Upgrade
- Production Ready
