**Wireless Security Fundamentals & Reference Notes**

---

### **1. Wi-Fi Architecture & Standards**

#### **Core Components & Identifiers**

* **Access Point (AP):** Hardware bridge connecting wireless clients to a wired network segment.
* **Client:** End-host (equipped with a wireless NIC) transmitting/receiving RF frames.
* **SSID (Service Set Identifier):** Human-readable network name broadcast in beacon frames.
* **BSSID (Basic Service Set Identifier):** Unique Layer-2 identifier of an AP interface (typically its MAC address).

#### **Frequency Spectrum Comparison**

| Parameter | 2.4 GHz Band | 5 GHz Band |
| --- | --- | --- |
| **Max Range (Indoor)** | ~45 meters (better physical penetration) | ~15 meters (high attenuation through solid obstacles) |
| **Max Data Rate** | Up to ~600 Mbps | Up to ~1300 Mbps (higher throughput) |
| **Interference Level** | High (Crowded: Bluetooth, Microwaves, 802.11 b/g/n) | Low (Less congested, non-overlapping channels) |

#### **IEEE 802.11 Standards Progression**

* **802.11b / 802.11a / 802.11g:** Legacy standards (11–54 Mbps).
* **802.11n (Wi-Fi 4):** Introduced MIMO; supports 2.4 GHz and 5 GHz.
* **802.11ac (Wi-Fi 5):** Operates on 5 GHz band; high-density enterprise standard.
* **802.11ax (Wi-Fi 6):** Operates on 2.4/5/6 GHz; uses OFDMA for efficiency in high-density environments.

---

### **2. Association & Authentication Workflows**

#### **Wi-Fi Connection Sequence**

`Scanning / Probing` $\rightarrow$ `Open Authentication Handshake` $\rightarrow$ `Association Request/Response` $\rightarrow$ `Key Exchange / 802.1X Authentication` $\rightarrow$ `Encrypted Data Flow`

* **Open System Authentication:** AP acknowledges client capability without verifying identity. Used prior to actual key negotiation across both Personal and Enterprise profiles.
* **WPA2-Personal (PSK):** Following association, client and AP perform a **4-Way Handshake** to prove knowledge of the Pre-Shared Key without transmitting it directly.
* **WPA2-Enterprise (802.1X):** Post-association, EAP frames are passed to a centralized **RADIUS** server for multi-factor/credential-based identity verification before port state transitions to authorized.

---

### **3. Wi-Fi Security Protocols & Vulnerabilities**

#### **Protocol Security Analysis**

* **WEP (Wired Equivalent Privacy):** **Deprecated.** Uses RC4 with small, static Initialization Vectors (IVs). Cryptographically broken; vulnerable to statistical IV collection and recovery of the key within minutes.
* **WPA (Wi-Fi Protected Access):** **Deprecated.** Temporary fix using TKIP. Vulnerable to cryptographic degradation.
* **WPA2:** Employs **AES-CCMP** for confidentiality and integrity.
* *Vulnerability:* Susceptible to offline dictionary attacks if an attacker captures the initial 4-Way Handshake.


* **WPA3:** Replaces the 4-Way Handshake with **SAE (Simultaneous Authentication of Equals)** (a Diffie-Hellman derivative). Protects against offline dictionary attacks and enforces forward secrecy.

#### **Primary Attack Vectors**

* **Rogue AP:** Unauthorized physical AP plugged into an internal switch port bypassing perimeter access controls.
* **Evil Twin:** Unauthorized AP broadcasting a target network's SSID to trick clients into connecting, enabling Man-in-the-Middle (MitM) traffic interception or credential harvesting.
* **Deauthentication Attack:** Injecting spoofed, unencrypted 802.11 Management Frames (Deauth/Disassociation) to force clients off the network or force a reconnect to capture handshakes.
* **WPS Pin Attacks:** Exploiting the 8-digit PIN design flaw in WPS to recover the WPA/WPA2 passphrase via brute force.

---

### **4. Bluetooth & Short-Range RF Protocols**

#### **Classic Bluetooth vs. Bluetooth Low Energy (BLE)**

| Feature | Classic Bluetooth | Bluetooth Low Energy (BLE) |
| --- | --- | --- |
| **Primary Use Case** | Continuous streaming (Audio, Data) | Intermittent low-bandwidth data (IoT, Wearables) |
| **Pairing Mechanism** | Secure Simple Pairing (SSP) via ECDH | Legacy Pairing or LE Secure Connections |
| **Default Security Risk** | Unencrypted/PIN fallback mode | "Just Works" mode (unauthenticated, vulnerable to MitM) |

#### **Bluetooth Attacks**

* **Bluejacking:** Sending unsolicited messages/data to nearby discoverable devices.
* **Bluesnarfing:** Unauthorized exfiltration of sensitive files or contact lists via open connections.
* **Bluebugging:** Exploiting firmware flaws to establish an AT command shell and take control of device functions (e.g., initiating calls, listening).

---

### **5. RFID, NFC, and Emerging IoT Wireless**

#### **RFID vs. NFC**

* **RFID:** Asymmetric systems (Reader $\rightarrow$ Tag). Ranges up to 100m+ (Active) or several meters (Passive). Operates across LF, HF, and UHF frequencies.
* **NFC:** Subset of HF RFID (13.56 MHz). Bidirectional peer-to-peer/card-emulation operating within $\approx 5\text{ cm}$.

#### **RFID/NFC Attack Profiles**

* **Cloning:** Exfiltrating raw UID/sector data from static cards onto writable transponders.
* **Relay Attack:** Extending actual physical range by proxying RF communications real-time between an isolated target card and a remote reader over a secondary network layer.
* **Skimming:** Uncontained passive collection of transmitted card data in public spaces.

#### **IoT Protocol Vulnerabilities**

```
           +-----------------------------------------------------------+
           |                Emerging IoT Protocols                     |
           +-----------------+---------------------+-------------------+
                             |                     |
                             v                     v
                 +-----------------------+  +-----------------------+
                 |    Zigbee (802.15.4)  |  |    Z-Wave             |
                 +-----------------------+  +-----------------------+
                 | Mesh topology; default|  | Uses S2 (Diffie-      |
                 | fallback transport    |  | Hellman); vulnerable  |
                 | keys expose joining   |  | to S0 downgrade       |
                 | device traffic.       |  | attacks (Z-Shave).    |
                 +-----------------------+  +-----------------------+

```

* **LoRaWAN:** Uses static session keys under Activation by Personalization (ABP); vulnerable to replay attacks if frame counter resets occur. Over-the-Air Activation (OTAA) is preferred.
* **Cellular IoT (LTE-M / NB-IoT):** Strong air-interface encryption, but weak host device security (exposed management endpoints, unpatched firmware, sleeping-mode DoS).
* **Infrared (IR):** Line-of-sight protocol lacking Layer-3/Layer-2 authentication; vulnerable to deterministic command replay attacks.

---

### **6. Defense & Hardening Matrix**

| Vulnerability / Attack Vector | Remediation Strategy |
| --- | --- |
| **Offline Passphrase Cracking** | Enforce WPA3-SAE, or use complex 20+ character passphrases on WPA2-Personal. |
| **Rogue Access Points / Evil Twins** | Deploy WIPS (Wireless Intrusion Prevention Systems) and enforce 802.1X Network Access Control (NAC). |
| **Management Frame Spoofing** | Enable **802.11w** Protected Management Frames (PMF). |
| **Zigbee Key Sniffing** | Enforce unique per-device Install Codes to derive transport keys; eliminate default global keys. |
| **Z-Wave S0 Downgrade** | Disable legacy S0 fallback support on the controller; require explicit user confirmation for unencrypted pairings. |
| **RFID/NFC Skimming** | Mandate Faraday shielding sleeves for transport tokens; implement AES-encrypted card storage (e.g., MIFARE DESFire). |
| **Network Pivoting via IoT** | Enforce strict enterprise Network Segmentation (VLANs) isolating all wireless IoT infrastructure from core server networks. |
