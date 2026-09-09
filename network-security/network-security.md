# Network Security: Protocol Attacks & Defenses

## Overview

This note summarizes the core concepts of network protocol vulnerabilities, focusing on the mechanics and mitigations for three foundational attack vectors: **Sniffing**, **Man-in-the-Middle (MITM)**, and **Password Attacks**. Identifying these flaws allows penetration testers to demonstrate impact, while enabling defenders to enforce fundamental security controls.

---

##  Key Takeaways & The Modern Landscape

* **Cleartext Insecurity:** Protocols transferring data without encryption are inherently vulnerable to sniffing and MITM manipulation.
* **The Solution:** Transition to encrypted alternatives (e.g., HTTPS, SSH, SFTP).
* **Persistent Risks:** While the modern internet heavily defaults to TLS encryption, cleartext protocols are still frequently encountered in:
* Legacy or unpatched systems.
* Internal corporate networks lacking internal encryption models.
* Resource-constrained IoT and embedded systems.
* Misconfigured services where TLS is available but not enforced.
* Development and staging environments.


* **Authentication Weaknesses:** Even with robust encryption, password-based endpoints remain high-value targets. Security architectures should prioritize rate limiting, multi-factor authentication (MFA), or passwordless configurations (passkeys/certificates).

---

##  Protocol & Port Reference

| Protocol | TCP Port | Application(s) | Data Security |
| --- | --- | --- | --- |
| **FTP** | 21 | File Transfer | Cleartext |
| **FTPS** | 990 | File Transfer | Encrypted (implicit TLS) |
| **HTTP** | 80 | Worldwide Web | Cleartext |
| **HTTPS** | 443 | Worldwide Web | Encrypted (implicit TLS) |
| **IMAP** | 143 | Email (MDA) | Cleartext |
| **IMAPS** | 993 | Email (MDA) | Encrypted (implicit TLS) |
| **POP3** | 110 | Email (MDA) | Cleartext |
| **POP3S** | 995 | Email (MDA) | Encrypted (implicit TLS) |
| **SFTP** | 22 | File Transfer | Encrypted (SSH) |
| **SMTP** | 25 | Email (MTA) | Cleartext |
| **SMTP Submission** | 587 | Email (MTA, client submission) | STARTTLS* |
| **SMTPS** | 465 | Email (MTA) | Encrypted (implicit TLS) |
| **SSH** | 22 | Remote Access and File Transfer | Encrypted |
| **Telnet** | 23 | Remote Access | Cleartext |

> ***Note on STARTTLS:** The connection begins unencrypted on the standard port and upgrades to TLS after the client issues the `STARTTLS` command. This differs from implicit TLS, where encryption begins immediately upon connection. It is common on port 587 but can also apply to ports 25, 110, and 143.

---

##  Tooling Reference

### Hydra Quick Reference

Hydra is a fast and flexible network logon cracker supporting numerous protocols.

```bash
# General Syntax Example
hydra -L users.txt -P passwords.txt -t 4 10.10.10.10 ssh

```

| Option | Explanation |
| --- | --- |
| `-l username` | Provide a single login name |
| `-L users.txt` | Provide a file containing usernames |
| `-p password` | Provide a single password to try |
| `-P wordlist.txt` | Specify the password list to use |
| `server service` | Set the target server address and service to attack (e.g., `10.10.10.10 ssh`) |
| `-s PORT` | Use in case of a non-default service port number |
| `-V` or `-vV` | Show the username and password combinations being tried (Verbose) |
| `-t n` | Number of parallel connections (threads) |
| `-w n` | Wait time between connections in seconds |
| `-f` | Stop execution after the first valid credential pair is found |
| `-d` | Display debugging output |

### Additional Security Tools

* **Wireshark / tcpdump:** Network packet capture and deep-packet analysis.
* **Bettercap:** MITM attacks, ARP spoofing, and network reconnaissance.
* **mitmproxy:** Interactive HTTPS proxy for inspecting and modifying traffic.
* **testssl.sh:** Command-line tool to check TLS/SSL configurations and vulnerabilities.
* **Nmap:** Port scanning, service detection, and script-based enumeration (NSE).
* **CrackMapExec / NetExec:** Automated password spraying and lateral movement across Windows/Active Directory environments.
* **Hashcat / John the Ripper:** High-performance offline password hash cracking.
* **Burp Suite:** Web application security testing and web traffic interception.

---

##  Defensive Checklist

* [ ] **Enforce Strong Ciphers:** Verify all services use TLS 1.2 or TLS 1.3 with secure cipher suites.
* [ ] **Disable Cleartext Protocols:** Deprecate Telnet, FTP, and HTTP, or isolate them to legacy segments if decommissioning is impossible.
* [ ] **Harden SSH:** Require key-based authentication and explicitly set `PasswordAuthentication no`.
* [ ] **Password Hygiene:** Enforce robust password complexity policies along with breached password detection.
* [ ] **Rate Limiting:** Implement account lockout policies or strict rate limiting on all public-facing authentication endpoints.
* [ ] **Deploy MFA:** Enable multi-factor authentication for all enterprise and sensitive administrative systems.
* [ ] **Network Segmentation:** Limit the broadcast domain and blast radius of sniffing/spoofing attacks using VLANs and zero-trust policies.
* [ ] **Strict Certificate Validation:** Ensure clients properly validate upstream certificates to prevent downstream MITM injection.
* [ ] **Implement HSTS:** Force HTTP Strict Transport Security (HSTS) headers on web apps to completely mitigate SSL stripping techniques.
* [ ] **Continuous Monitoring:** Maintain granular logging to detect authentication anomalies and brute-force indicators early.
