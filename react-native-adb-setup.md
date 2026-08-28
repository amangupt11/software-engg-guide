# React Native - Wireless Android Debugging Setup

## Purpose

This document is a reusable setup/reference guide for developing a React Native Android application using a **physical device ** as the development and debugging device.

The phone is connected to the Windows development machine using **Android Debug Bridge (ADB) over Wi-Fi**, so a USB cable is not required for normal development after the initial setup.

---

## 1. Setup Overview

The development environment looks like this:

```text
┌──────────────────────────────┐
│ Windows Development PC       │
│                              │
│ React Native project         │
│ Node.js / npm                │
│ Android SDK / ADB            │
│ Android Studio               │
│ Metro bundler                │
└──────────────┬───────────────┘
               │
               │ Wi-Fi / ADB
               │
               ▼
┌──────────────────────────────┐
│         │
│                              │
│ physical Android device      │
│ React Native debug app       │
└──────────────────────────────┘
```

### Confirmed working device connection

The following connection was successfully established:

```text
Phone IP:             192.168.1.63
ADB pairing port:     43033
ADB connection port:  39497
ADB device:           192.168.1.63:39497
ADB status:            device
```

> Important: IP addresses and ports can change. The values above are the values from the successful setup and should be treated as an example/reference, not permanent values.

---

# 2. Requirements

You need:

- Windows PC
- Android Studio
- Android SDK
- Android Platform Tools (`adb`)
- Node.js
- npm
- React Native project
- PC and phone connected to the same Wi-Fi network

---

# 3. Verify ADB on Windows

Open PowerShell and run:

```powershell
adb --version
```

You should get an Android Debug Bridge version.

If Windows says:

```text
adb : The term 'adb' is not recognized...
```

then Android Platform Tools are either not installed or `adb` is not in the Windows `PATH`.

After fixing the PATH, open a new PowerShell window and run:

```powershell
adb --version
```

---

# 4. Enable Developer Options on Samsung Galaxy M17

On the phone:

1. Open **Settings**
2. Open **About phone**
3. Open **Software information**
4. Find **Build number**
5. Tap **Build number** approximately 7 times
6. Enter your phone PIN/password if requested
7. Android should report that Developer options have been enabled

Then return to:

```text
Settings
  → Developer options
```

---

# 5. Enable Wireless Debugging

On the Galaxy M17:

```text
Settings
  → Developer options
  → Wireless debugging
```

Turn **Wireless debugging** ON.

Make sure the phone and Windows PC are connected to the same local Wi-Fi network.

---

# 6. Pair the Phone with ADB

In the phone's:

```text
Developer options
  → Wireless debugging
```

choose:

```text
Pair device with pairing code
```

The phone will show information similar to:

```text
IP address & Port
192.168.1.63:43033

Wi-Fi pairing code
636325
```

The actual IP address, port, and pairing code will be different each time.

In PowerShell:

```powershell
adb pair <PHONE_IP>:<PAIRING_PORT>
```

Example:

```powershell
adb pair 192.168.1.63:43033
```

ADB will ask:

```text
Enter pairing code:
```

Enter the six-digit code shown on the phone.

A successful pairing looks like:

```text
Successfully paired to 192.168.1.63:43033
```

---

# 7. Connect to the Phone

After pairing, return to the main:

```text
Developer options
  → Wireless debugging
```

Look for the phone's normal:

```text
IP address & Port
```

This is the **connection port**.

It is usually different from the pairing port.

Example:

```text
192.168.1.63:39497
```

Run:

```powershell
adb connect 192.168.1.63:39497
```

Successful output:

```text
connected to 192.168.1.63:39497
```

---

# 8. Verify the ADB Device

Run:

```powershell
adb devices
```

Expected result:

```text
List of devices attached
192.168.1.63:39497    device
```

The important part is:

```text
device
```

This means ADB can communicate with the physical device phone.

---

# 9. Important: Pair Port vs Connection Port

There are two different ports involved in wireless debugging.

## Pairing

```powershell
adb pair <IP>:<PAIRING_PORT>
```

Example:

```powershell
adb pair 192.168.1.63:43033
```

## Connecting

```powershell
adb connect <IP>:<CONNECTION_PORT>
```

Example:

```powershell
adb connect 192.168.1.63:39497
```

### Do not assume they are the same.

For example, this was the successful setup:

```text
Pairing:
192.168.1.63:43033

Connection:
192.168.1.63:39497
```

---

# 10. Run the React Native Application

Go to the React Native project:

```powershell
cd C:\Users\ADMIN\Desktop\pinaak
```

Verify the phone:

```powershell
adb devices
```

Then run:

```powershell
npx react-native run-android
```

Depending on the project configuration, this may also work:

```powershell
npm run android
```

React Native should build the Android application and install it on the connected Galaxy M17 5G.

---

# 11. Start Metro Manually

If you want to run Metro separately, open one PowerShell terminal:

```powershell
npx react-native start
```

Leave that terminal running.

Then open a second PowerShell terminal:

```powershell
npx react-native run-android
```

This is often useful during development because Metro stays visible in its own terminal.

---

# 12. React Native Metro Connection Over ADB

For reliable development, especially when Metro is running on the Windows PC, you can forward the Metro port through ADB.

Run:

```powershell
adb reverse tcp:8081 tcp:8081
```

If more than one Android device is connected, specify the device:

```powershell
adb -s 192.168.1.63:39497 reverse tcp:8081 tcp:8081
```

Then start Metro:

```powershell
npx react-native start
```

And in another terminal:

```powershell
npx react-native run-android
```

### Why `adb reverse`?

React Native's Metro development server normally uses port:

```text
8081
```

`adb reverse` allows the Android device to access the development server running on the PC through the ADB connection.

---

# 13. Recommended Daily Development Workflow

Once the phone has already been paired, the normal workflow is:

## Step 1 — Check Wireless Debugging

On the phone:

```text
Settings
  → Developer options
  → Wireless debugging
```

Make sure it is ON.

## Step 2 — Connect ADB

Run:

```powershell
adb connect <PHONE_IP>:<CONNECTION_PORT>
```

Example:

```powershell
adb connect 192.168.1.63:39497
```

## Step 3 — Verify

```powershell
adb devices
```

Expected:

```text
192.168.1.63:39497    device
```

## Step 4 — Forward Metro

```powershell
adb -s 192.168.1.63:39497 reverse tcp:8081 tcp:8081
```

## Step 5 — Start Metro

```powershell
npx react-native start
```

## Step 6 — Run Android

In another terminal:

```powershell
npx react-native run-android
```

---

# 14. If `adb devices` Shows Nothing

Run:

```powershell
adb devices
```

If you get:

```text
List of devices attached
```

with no device below it, try:

```powershell
adb connect <PHONE_IP>:<CONNECTION_PORT>
```

Example:

```powershell
adb connect 192.168.1.63:39497
```

Then:

```powershell
adb devices
```

---

# 15. If ADB Says `offline`

Example:

```text
192.168.1.63:39497    offline
```

Try:

```powershell
adb disconnect 192.168.1.63:39497
```

Then:

```powershell
adb connect 192.168.1.63:39497
```

Then:

```powershell
adb devices
```

If necessary, restart ADB:

```powershell
adb kill-server
adb start-server
```

Then connect again:

```powershell
adb connect <PHONE_IP>:<CONNECTION_PORT>
```

---

# 16. If ADB Says `unauthorized`

Example:

```text
192.168.1.63:39497    unauthorized
```

Look at the phone.

Android may be asking you to authorize debugging.

Accept the authorization prompt.

If the prompt does not appear, disconnect/reconnect:

```powershell
adb disconnect
adb connect <PHONE_IP>:<CONNECTION_PORT>
```

Then check:

```powershell
adb devices
```

---

# 17. If `adb connect` Fails

Example:

```text
failed to connect to 192.168.1.63:39497
```

Check:

### A. Same Wi-Fi

The Windows PC and phone should normally be on the same Wi-Fi/local network.

### B. Wireless debugging is enabled

On the phone:

```text
Developer options
  → Wireless debugging
```

Make sure it is ON.

### C. Connection port changed

The port displayed by Wireless debugging can change.

Check the phone again and use the currently displayed:

```text
IP address & Port
```

Then:

```powershell
adb connect <CURRENT_IP>:<CURRENT_PORT>
```

### D. Windows firewall/network isolation

Some Wi-Fi networks prevent devices from communicating with each other.

This can happen on:

- Guest Wi-Fi
- Corporate Wi-Fi
- Public Wi-Fi
- Networks with client isolation enabled

Try a normal private/home Wi-Fi network if possible.

---

# 18. If the Phone and PC Have Different IP Addresses

This is normal.

For example:

```text
PC:
192.168.1.20

Phone:
192.168.1.63
```

That is fine as long as they are on the same local network/subnet and can communicate.

You generally want something like:

```text
PC      192.168.1.x
Phone   192.168.1.x
```

---

# 19. Check Connected Devices

List devices:

```powershell
adb devices
```

Get the Android device model:

```powershell
adb shell getprop ro.product.model
```

Get the manufacturer:

```powershell
adb shell getprop ro.product.manufacturer
```

Get Android version:

```powershell
adb shell getprop ro.build.version.release
```

Get Android SDK/API level:

```powershell
adb shell getprop ro.build.version.sdk
```

---

# 20. Select a Specific Device

If multiple devices/emulators are connected:

```powershell
adb devices
```

Example:

```text
List of devices attached
192.168.1.63:39497    device
emulator-5554         device
```

Use:

```powershell
adb -s 192.168.1.63:39497 shell
```

For React Native:

```powershell
npx react-native run-android --deviceId 192.168.1.63:39497
```

If your React Native CLI version/project supports that option.

---

# 21. Useful ADB Commands

## Check devices

```powershell
adb devices
```

## Connect

```powershell
adb connect <IP>:<PORT>
```

## Disconnect one device

```powershell
adb disconnect <IP>:<PORT>
```

## Disconnect all network ADB connections

```powershell
adb disconnect
```

## Restart ADB

```powershell
adb kill-server
adb start-server
```

## Open Android shell

```powershell
adb shell
```

## Install an APK

```powershell
adb install path\to\app.apk
```

## View Android logs

```powershell
adb logcat
```

## Clear logcat

```powershell
adb logcat -c
```

## Forward Metro

```powershell
adb reverse tcp:8081 tcp:8081
```

## List reverse connections

```powershell
adb reverse --list
```

---

# 22. Useful React Native Commands

## Start Metro

```powershell
npx react-native start
```

## Run Android

```powershell
npx react-native run-android
```

## Reset Metro cache

```powershell
npx react-native start --reset-cache
```

## Run Android after resetting Metro cache

Terminal 1:

```powershell
npx react-native start --reset-cache
```

Terminal 2:

```powershell
npx react-native run-android
```

---

# 23. Common Complete Command Sequence

If the phone is already paired and its current connection address is:

```text
192.168.1.63:39497
```

run:

```powershell
adb connect 192.168.1.63:39497
```

Then:

```powershell
adb devices
```

Then:

```powershell
adb -s 192.168.1.63:39497 reverse tcp:8081 tcp:8081
```

Then:

```powershell
npx react-native start
```

Open another PowerShell:

```powershell
npx react-native run-android
```

---

# 24. Complete First-Time Setup Sequence

If the phone has never been paired:

### On phone

```text
Settings
→ About phone
→ Software information
→ Tap Build number 7 times
```

Then:

```text
Settings
→ Developer options
→ Wireless debugging
→ ON
→ Pair device with pairing code
```

### On PC

Use the pairing address shown on the phone:

```powershell
adb pair <PHONE_IP>:<PAIRING_PORT>
```

Enter the six-digit pairing code.

Then use the **main Wireless debugging IP address & Port**:

```powershell
adb connect <PHONE_IP>:<CONNECTION_PORT>
```

Verify:

```powershell
adb devices
```

Then:

```powershell
adb reverse tcp:8081 tcp:8081
```

Then:

```powershell
npx react-native start
```

And:

```powershell
npx react-native run-android
```

---

# 25. Important Notes About Pairing

Once the phone has successfully paired, you generally do not need to run:

```powershell
adb pair ...
```

every time.

For normal daily development, you usually only need:

```powershell
adb connect <PHONE_IP>:<CONNECTION_PORT>
```

However, if the pairing state is lost, Wireless debugging is reset, the device is forgotten, or Android requires a new pairing, repeat the pairing process.

---

# 26. Important Notes About IP Addresses and Ports

Do not permanently hard-code:

```text
192.168.1.63:39497
```

The phone's IP address and ADB connection port can change.

Always check:

```text
Settings
→ Developer options
→ Wireless debugging
```

for the current:

```text
IP address & Port
```

Use the currently displayed address.

---

# 27. Recommended Project Documentation

Keep this file in the React Native project's documentation folder, for example:

```text
pinaak/
├── android/
├── ios/
├── src/
├── package.json
├── README.md
└── docs/
    └── GALAXY-M17-WIRELESS-DEBUGGING.md
```

You can also keep a short section in the project's main `README.md`:

```markdown
## Android physical device Device Development

The project can be developed using a physical device  over ADB Wi-Fi.

See:

`docs/GALAXY-M17-WIRELESS-DEBUGGING.md`

Quick connection:

```powershell
adb connect <PHONE_IP>:<CONNECTION_PORT>
adb devices
adb reverse tcp:8081 tcp:8081
npx react-native start
npx react-native run-android
```
```

---

# 28. Troubleshooting Checklist

Before troubleshooting React Native itself, verify the lower-level connection first.

### Phone

- [ ] Developer options enabled
- [ ] Wireless debugging ON
- [ ] Phone connected to Wi-Fi
- [ ] PC connected to the same Wi-Fi
- [ ] Current IP address and connection port checked

### ADB

Run:

```powershell
adb devices
```

The device should show:

```text
device
```

Not:

```text
offline
```

Not:

```text
unauthorized
```

### Metro

Run:

```powershell
adb reverse tcp:8081 tcp:8081
```

Then:

```powershell
npx react-native start
```

### React Native

Finally:

```powershell
npx react-native run-android
```

---

# 29. Quick Reference Card

```text
PHONE
-----
Settings
→ Developer options
→ Wireless debugging
→ ON


PAIR (only when required)
-------------------------
adb pair <IP>:<PAIRING_PORT>


CONNECT
-------
adb connect <IP>:<CONNECTION_PORT>


VERIFY
------
adb devices


METRO
-----
adb reverse tcp:8081 tcp:8081


START METRO
-----------
npx react-native start


RUN APP
-------
npx react-native run-android
```

---

# 30. Successful Setup Record

The following was successfully tested:

```text
Device:


Connection:
ADB over Wi-Fi

Phone IP:
192.168.1.63

Pairing command:
adb pair 192.168.1.63:43033

Pairing result:
Successfully paired

Connection command:
adb connect 192.168.1.63:39497

Connection result:
connected to 192.168.1.63:39497

adb devices:
192.168.1.63:39497    device
```

This confirms that the physical device Galaxy M17 5G is usable as an Android development/debugging device from the Windows PC.

---

# 31. Final Daily Cheat Sheet

Use this whenever you start development:

```powershell
adb connect <CURRENT_PHONE_IP>:<CURRENT_CONNECTION_PORT>
adb devices
adb -s <CURRENT_PHONE_IP>:<CURRENT_CONNECTION_PORT> reverse tcp:8081 tcp:8081
npx react-native start
```

Then, in another terminal:

```powershell
npx react-native run-android
```

If the phone is already connected and `adb devices` shows `device`, you can skip the `adb connect` step.

---

## Key Rule to Remember

**`adb pair` is for pairing.**

**`adb connect` is for connecting.**

They may use different ports.

Example:

```powershell
adb pair 192.168.1.63:43033
adb connect 192.168.1.63:39497
```

Then:

```powershell
adb devices
```

If you see:

```text
192.168.1.63:39497    device
```

your Galaxy M17 is ready for React Native development.
