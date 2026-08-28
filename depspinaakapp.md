# Pinaak Mobile Application

A production-grade React Native CLI application built with modern architecture and best practices.

## Tech Stack

- React Native 0.86
- React 19
- TypeScript
- Redux Toolkit
- TanStack Query
- React Navigation 7
- Firebase
- MMKV
- React Hook Form
- Axios

---

# Project Goals

- Production Ready
- Enterprise Architecture
- Scalable
- Secure
- Maintainable
- Modular
- High Performance
- Google Play Compliant
- App Store Compliant

---

# Dependency Guide

## Navigation

### @react-navigation/native

Purpose

Root navigation library.

Used For

- Navigation Container
- Deep Linking
- Theme

---

### @react-navigation/native-stack

Purpose

Native Stack Navigation.

Used For

- Login
- Splash
- Booking Flow
- Settings
- Profile

---

### @react-navigation/bottom-tabs

Purpose

Bottom Navigation.

Used For

- Customer Bottom Tabs
- Vendor Bottom Tabs
- Driver Bottom Tabs

---

### @react-navigation/material-top-tabs

Purpose

Top Tabs.

Used For

- Booking Tabs
- Active / Completed Trips
- Driver Requests

---

## State Management

### @reduxjs/toolkit

Purpose

Global State.

Used For

- User
- Auth
- Theme
- Language
- Settings

---

### react-redux

Purpose

Connect React with Redux.

---

### redux-persist

Purpose

Persist Redux Store.

Storage

MMKV

---

## Server State

### @tanstack/react-query

Purpose

API Cache.

Used For

- Bookings
- Trips
- Vendor Requests
- Notifications

Features

- Cache
- Retry
- Refetch
- Background Sync

---

## Network

### axios

Purpose

HTTP Client.

Used For

- API Requests
- Token Refresh
- Request Interceptors
- Response Interceptors

---

## Forms

### react-hook-form

Purpose

Forms.

Used For

- Login
- Registration
- Profile
- Booking
- Vendor

---

### zod

Purpose

Validation.

Used For

- Form Validation
- API Validation

---

## Storage

### react-native-mmkv

Purpose

Fast Local Storage.

Store

- Theme
- Language
- User Role
- Filters
- Redux Persist

Never Store

- Password
- JWT
- Refresh Token

---

### react-native-keychain

Purpose

Secure Storage.

Store

- JWT
- Refresh Token
- Biometric Secret

---

## Firebase

### @react-native-firebase/app

Purpose

Firebase Initialization.

---

### @react-native-firebase/messaging

Purpose

Push Notifications.

Used For

- Booking Updates
- Driver Assigned
- Trip Started
- Trip Completed

---

### @react-native-firebase/crashlytics

Purpose

Crash Reporting.

Used For

- Crash Monitoring
- Error Reporting

---

## UI

### @gorhom/bottom-sheet

Purpose

Bottom Sheets.

Used For

- Filters
- Actions
- Vehicle Selection

---

### react-native-reanimated

Purpose

Animations.

Used For

- Screen Transition
- Bottom Sheet
- Swipe

---

### react-native-gesture-handler

Purpose

Gesture Support.

Used For

- Swipe
- Drag
- Bottom Sheet

---

### react-native-safe-area-context

Purpose

Safe Area Support.

---

### react-native-screens

Purpose

Native Screen Optimization.

---

### react-native-svg

Purpose

SVG Support.

Used For

- Icons
- Illustrations

---

### react-native-pager-view

Purpose

Pager View.

Used For

- Top Tabs

---

### @shopify/flash-list

Purpose

High Performance List.

Used For

- Booking List
- Driver List
- Notifications
- Trips

---

## Utilities

### react-native-config

Purpose

Environment Variables.

Used For

- API URL
- Google Keys
- Razorpay Key

---

### react-native-device-info

Purpose

Device Information.

Used For

- Version
- Device Model
- OS

---

### @react-native-community/netinfo

Purpose

Network Detection.

Used For

- Offline Mode
- Retry

---

## Documents

### react-native-image-picker

Purpose

Image Selection.

Used For

- Profile Photo
- Vehicle Images
- Documents

---

### react-native-pdf

Purpose

PDF Viewer.

Used For

- Invoice
- Booking PDF

---

### react-native-file-viewer

Purpose

Open Files.

Used For

- PDF
- Documents

---

# Security Rules

JWT → Keychain

Refresh Token → Keychain

Theme → MMKV

Language → MMKV

Filters → MMKV

Profile Cache → MMKV

Never store passwords.

Never store JWT inside MMKV.

Always use HTTPS in production.

---

# Architecture

Presentation

↓

Navigation

↓

Screens

↓

Redux + React Query

↓

Services

↓

Axios

↓

Backend API

---

# Code Quality

- ESLint
- Prettier
- Husky
- Commitlint
- lint-staged

---

# Build

Development

npm run android

Production

Generate Android App Bundle (.aab)

Generate iOS Archive

---

# Future Packages

These packages will be installed only when needed.

- react-native-maps
- react-native-permissions
- Razorpay
- Firebase Analytics
- Firebase App Check
- Firebase Performance
- Lottie
