Here is the structured, step-by-step documentation formatted directly for a GitHub repository (`README.md` or a security study wiki).

---

# Mobile Application Penetration Testing Fundamentals

A comprehensive guide to mobile application security assessments, static and dynamic analysis methodologies, OWASP Mobile Top 10 mapping, and practical CTF analysis workflows for Android and iOS targets.

---

## 📌 Learning Objectives & Prerequisites

### Objectives

* Understand the core concepts of mobile application penetration testing from an offensive/red team perspective.
* Deconstruct application packages across both Android and iOS platforms.
* Execute static and dynamic analysis methodologies to uncover vulnerabilities.
* Map technical findings to the **OWASP Mobile Top 10** framework.

### Prerequisites

* **Linux Fundamentals**
* **Web Fundamentals** (HTTP/HTTPS communications)
* **Burp Suite Basics** (Conceptual proxy knowledge)

---

## 🏗️ Phase 1: Application Package Architecture

Mobile applications are compiled packages containing executable code, asset stores, and manifest definitions isolated by OS-level sandboxing.

### Package Components

| Component | Android (`.apk`) | iOS (`.ipa`) | Security Significance |
| --- | --- | --- | --- |
| **Manifest / Config** | `AndroidManifest.xml` | `Info.plist` | Dictates permissions, exposed components, and OS flags. |
| **Compiled Code** | DEX / Dalvik Bytecode | Mach-O Binary | Contains core business logic and potential hardcoded secrets. |
| **Assets & Resources** | `/assets`, `/res` | Bundled Plists, Assets | Stores strings, databases, configurations, and internal media. |

### The Mobile Sandbox Model

* Operating systems isolate applications in dedicated containers.
* Direct inter-process communication (IPC) and file read access across app containers are blocked by default.
* Misconfigurations inside manifests can weaken sandbox boundaries, expanding the target attack surface.

---

## 🔄 Phase 2: Penetration Testing Methodology

```
+------------------+     +------------------+     +------------------+     +------------------+
| 1. Reconnaissance| --> |2. Static Analysis| --> |3. Dynamic Analysis| --> |   4. Reporting   |
+------------------+     +------------------+     +------------------+     +------------------+

```

### 1. Reconnaissance

* Identify target application functionality, architecture, and user roles.
* Sourced backend API endpoints and obtain target application packages (`.apk` / `.ipa`).

### 2. Static Analysis (At-Rest Examination)

* Decompile and unpack binary packages without execution.
* Inspect manifest/configuration files for exposed flags and over-privileged permissions.
* Scan codebase and assets for hardcoded secrets, API tokens, and credentials.

### 3. Dynamic Analysis (Runtime Examination)

* Intercept and manipulate HTTPS client-server communications.
* Monitor runtime data storage, IPC, and system log streams.
* Execute runtime instrumentation to bypass client-side security controls.

### 4. Reporting

* Categorize findings based on severity (**Critical**, **High**, **Medium**, **Low**).
* Provide exact steps to reproduce, impact assessments, and remediation guidance.

---

## 🔍 Phase 3: Static Analysis Deep-Dive

### Key Target Files & Misconfigurations

* **Android (`AndroidManifest.xml`)**:
* `android:debuggable="true"`: Allows runtime debugging and memory dumping.
* `android:exported="true"`: Exposes activities/services to unauthorized third-party apps on the device.
* Over-requested permissions (`CAMERA`, `READ_CONTACTS`, `ACCESS_FINE_LOCATION`).


* **iOS (`Info.plist`)**:
* `NSAllowsArbitraryLoads = true`: Disables App Transport Security (ATS), allowing cleartext HTTP communications.



### Primary Static Tooling

* **Decompilers**: `JADX`, `Apktool`
* **Automated Scanner**: **MobSF (Mobile Security Framework)**

---

## ⚡ Phase 4: Dynamic Analysis Techniques

### 1. Traffic Interception & SSL Pinning Bypass

* Route app network traffic through intercepting proxies (e.g., Burp Suite).
* **SSL Pinning**: A security control forcing apps to validate server certificates against a hardcoded store.
* **Bypass Tooling**: Use **Objection** or **Frida** to hook network libraries and disable validation at runtime.

### 2. Runtime Instrumentation with Frida

* Attach hooks to running process memory.
* Modify method return values, bypass biometric authentication, or extract active memory keys.

### 3. System Logging Inspection

* Monitor real-time system logs (`logcat` on Android / system logs on iOS).
* Identify sensitive data leaked into production build logs (**OWASP M9**).

---

## 🛡️ OWASP Mobile Top 10 Reference Mapping

* **M1: Improper Credential Usage**: Hardcoded API keys, DB credentials, or tokens in source code/assets.
* **M3: Insecure Authentication**: Missing server-side re-verification, easily bypassed local authentication.
* **M5: Insecure Communication**: Usage of cleartext HTTP or disabled TLS/ATS checks.
* **M7: Insufficient Binary Protections**: Missing code obfuscation, root/jailbreak detection, or tamper-resistance mechanisms.
* **M8: Security Misconfiguration**: Exported internal components, over-requested permissions, debug flags enabled.
* **M9: Insecure Data Storage**: Sensitive data saved in unencrypted local databases, plists, or system logs.

---

## 🚩 Practical CTF Walkthrough: Analyzing LeakyPackage

### Overview

* **Target Company**: Helix Solutions
* **Environment**: Local MobSF Instance (`http://<MACHINE_IP>:8000`)
* **Default Credentials**: `mobsf` / `mobsf`

---

### Part 1: Android Analysis (`LeakyPackage.apk`)

#### Step 1: Automated Scan & Initial Security Score

1. Upload `LeakyPackage.apk` to MobSF.
2. Review the static analysis dashboard:
* **Security Score**: `40/100` (High Risk).
* **Package Name**: `com.tryhackme.leakypackage`.
* **Exported Activities**: `1 / 2`.



#### Step 2: Permission Inspection

Navigate to **Permissions**. Identify over-requested, high-risk permissions:

* `ACCESS_FINE_LOCATION`
* `CAMERA`
* `READ_CONTACTS`
* `READ_EXTERNAL_STORAGE`
* `RECORD_AUDIO`

> **Finding**: Excessive permission requests broaden the attack surface if compromised (**OWASP M8**).

#### Step 3: Manifest Analysis (`AndroidManifest.xml`)

Navigate to **Security Analysis** > **Manifest Analysis**:

1. `android:debuggable="true"` (Line 13) — Allows runtime debugging.
2. `com.tryhackme.leakypackage.AdminPanelActivity` with `android:exported="true"` (Line 20) — Component accessible by external applications without explicit permission requirements.

#### Step 4: Source Code Extraction

Navigate to **Security Analysis** > **Code Analysis**:

* **File 1 (`HelixConfig.java`)**: Hardcoded API Key identified on line 5 (**OWASP M1**).
* **File 2 (`DatabaseHelper.java`)**: Plaintext production database host, database name, and root password exposed on lines 5–7 (**OWASP M1**).

---

### Part 2: iOS Analysis (`LeakyPackage.ipa`)

#### Step 5: Upload & Assets Inspection

1. Upload `LeakyPackage.ipa` to MobSF.
2. Review basic metadata:
* **App Identifier**: `com.tryhackme.LeakyPackage`.
* **Security Score**: `45/100`.



#### Step 6: App Transport Security (ATS) Misconfiguration

1. Open **Decompiled Assets** > **View Info.plist**.
2. Locate the `NSAppTransportSecurity` block:
```xml
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <true/>
</dict>

```


3. Verify under **Security Analysis** > **Transport Security**.

> **Finding**: ATS disabled completely, enabling unencrypted HTTP communication (**OWASP M5 / M8**).

#### Step 7: Sensitive Data in Property Lists

1. Navigate to **Reconnaissance** > **Files** (or inspect `LeakyPackage.app/internal_config.plist`).
2. Extract plaintext configuration values and internal secrets embedded directly inside bundled property lists (**OWASP M9**).

---
