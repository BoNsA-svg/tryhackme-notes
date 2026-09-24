# Mobile Application Pentesting Cheat Sheet

> Follow the [full methodology](../pentesting-methodology/mobile/mobile-application-penetration-testing.md).

## Record

- Build/version and package/bundle ID
- Device/emulator, OS, rooted/jailbroken state
- Backend environment, accounts, roles, and proxy setup

## Static

- Manifest/plist, permissions, exported components, entitlements
- Secrets, endpoints, schemes/deep links, WebViews
- Cryptography, storage, signing, libraries, source maps

## Dynamic

- API traffic and certificate validation
- Files, databases, preferences, Keychain/Keystore, logs, cache
- IPC/intents, deep links, notifications, clipboard, screenshots
- Biometrics, logout, backup, backgrounding, compromised-device behavior

## Backend

- Object/function/property authorization
- Token lifecycle, rate limits, mass assignment, business logic
- Compare User A, User B, and privileged roles

## Cleanup

Remove proxy certificates, profiles, test files, hooks, accounts, and instrumentation.
