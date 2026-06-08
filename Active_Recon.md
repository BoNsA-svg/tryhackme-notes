# Active Reconnaissance Quick Reference

> **Core Principle:** Active reconnaissance involves directly interacting with a target system to identify live hosts, open ports, running services, and exposed technologies. Unlike passive recon, these techniques generate detectable traffic and may trigger alerts from firewalls, WAFs, IDS/IPS, or EDR solutions.

---

# 1. Web Browser & Developer Tools

The web browser is one of the most powerful active reconnaissance tools because its traffic naturally resembles legitimate user activity.

## DevTools Shortcuts

| Platform | Shortcut |
|-----------|-----------|
| Windows / Linux | `Ctrl + Shift + I` |
| macOS | `Option + Command + I` |

## Recon-Critical DevTools Tabs

### Network

Displays real-time requests and responses.

Look for:

- `Server`
- `X-Powered-By`
- `Content-Security-Policy`
- Response status codes
- Cookies

### Sources

Inspect loaded:

- JavaScript files
- CSS files
- HTML documents

Look for:

- Hardcoded API endpoints
- Internal URLs
- Directory structures
- Developer comments

### Application (Storage)

Inspect:

- Cookies
- Local Storage
- Session Storage

Potential findings:

- Session tokens
- Authentication data
- Tracking identifiers

### Security

Review:

- SSL/TLS certificate details
- Certificate chain
- Subject Alternative Names (SANs)

Potential findings:

- Undocumented subdomains
- Alternate hostnames

### Console

Execute JavaScript directly within the page context for testing and inspection.

## Useful Recon Extensions

| Extension | Purpose |
|------------|----------|
| Wappalyzer | Technology fingerprinting |
| BuiltWith | Technology fingerprinting |
| FoxyProxy | Proxy switching for Burp Suite or OWASP ZAP |
| User-Agent Switcher | Browser/device emulation |

---

# 2. Host & Path Discovery

## Ping (ICMP Host Discovery)

Uses:

- ICMP Echo Request (Type 8)
- ICMP Echo Reply (Type 0)

### Basic Usage

#### Linux / macOS

```bash
ping -c 5 <IP_or_Domain>
```

#### Windows

```cmd
ping -n 5 <IP_or_Domain>
```

#### Force IPv4 or IPv6

```bash
ping -4 <IP>
ping -6 <IPv6>
```

## TTL-Based OS Fingerprinting

The Time To Live (TTL) value can provide clues about the target operating system.

| Operating System | Default TTL |
|------------------|------------|
| Linux / Unix | 64 |
| Windows | 128 |

Example:

```text
ttl=58
```

Likely indicates:

- Linux host
- Approximately 6 network hops away

---

## Ping Troubleshooting Matrix

| Result | Likely Meaning | Recommended Action |
|----------|---------------|-------------------|
| Fast replies / 0% loss | Host is alive | Proceed with service enumeration |
| Destination Host Unreachable | No route or host offline | Verify IP and network path |
| 100% packet loss | ICMP blocked by firewall | Use TCP or UDP discovery |
| High latency / packet loss | Congestion or filtering | Run traceroute |

---

## Traceroute (Path Mapping)

Traceroute increments TTL values to identify each hop between your system and the target.

Each router returns:

```text
ICMP Time Exceeded
```

when the TTL reaches zero.

### Linux / macOS

```bash
traceroute <IP>
```

### Windows

```cmd
tracert <IP>
```

### TCP Traceroute

Useful when UDP traffic is filtered.

```bash
traceroute -T <IP>
```

### ICMP Traceroute

```bash
traceroute -I <IP>
```

### Continuous Monitoring

```bash
mtr <IP>
```

Combines:

- Ping statistics
- Traceroute functionality

### Note

```text
*
```

typically indicates a router that suppresses ICMP responses.

---

# 3. Banner Grabbing & Netcat Operations

Banner grabbing involves connecting to services and reading identifying information exposed by the server.

This information often includes:

- Service name
- Software version
- Protocol details

---

## Telnet (Legacy TCP Client)

> Telnet is unencrypted and should only be used for plaintext services.

### Syntax

```bash
telnet <IP> <PORT>
```

### Example: HTTP Banner

```bash
telnet 10.145.191.225 80
```

Then type:

```http
GET / HTTP/1.1
Host: target

```

Press **Enter twice** to submit the request.

---

## Netcat (`nc`)

A versatile TCP/UDP networking utility.

### Client Mode

Connect to a remote service.

```bash
nc <IP> <PORT>
```

Example:

```bash
nc 10.145.191.225 21
```

Some services (such as FTP and SMTP) immediately display banners after connection.

---

### Listening Mode

Start a listener.

```bash
nc -vnlp <PORT>
```

Example:

```bash
nc -vnlp 1234
```

### IPv6 Listener

```bash
nc -6 -lp <PORT>
```

---

## Common Netcat Flags

| Flag | Description |
|--------|-------------|
| `-l` | Listen mode |
| `-p` | Specify port |
| `-n` | Disable DNS resolution |
| `-v` | Verbose output |
| `-vv` | Extra verbose output |
| `-k` | Keep listening after disconnect |

---

## TLS / HTTPS Banner Inspection

Traditional `telnet` and many `nc` implementations cannot negotiate TLS.

### HTTP Headers

```bash
curl -I https://<IP>
```

### SSL/TLS Inspection

```bash
openssl s_client -connect <IP>:<PORT>
```

### Netcat with SSL Support

```bash
ncat --ssl <IP> <PORT>
```

---

# Quick Reference Command Dashboard

```bash
# =========================
# HOST DISCOVERY & PATHS
# =========================

ping -c 5 10.145.191.225
ping -n 5 10.145.191.225

traceroute 10.145.191.225
traceroute -T 10.145.191.225
tracert 10.145.191.225

mtr 10.145.191.225

# =========================
# BANNER GRABBING
# =========================

nc 10.145.191.225 80

nc -vnlp 1234

curl -I http://10.145.191.225

openssl s_client -connect 10.145.191.225:443

ncat --ssl 10.145.191.225 443
```

---

# Key Takeaways

- Active reconnaissance generates traffic that may be logged or detected.
- Browser Developer Tools often reveal valuable application intelligence.
- `ping`, `traceroute`, and `mtr` help identify live hosts and network paths.
- Banner grabbing can expose service versions useful for vulnerability assessment.
- TLS-enabled services require tools such as `openssl s_client` or `ncat --ssl`.
- ICMP filtering does not necessarily mean a host is offline.
  
---

# Nmap Host Discovery: Protocol Reference Note

## Overview

Host discovery (or "ping scanning") is the foundational phase of network reconnaissance. Its primary goal is to determine which target hosts are active while minimizing unnecessary traffic. Any valid response from a target indicates that the host is online and responsive.

---

##  Nmap Command-Line Options Reference

When conducting host discovery exclusively, append the **`-sn`** flag. Omitting `-sn` instructs Nmap to automatically transition into its default port-scanning routines against any discovered live hosts.

| Scan Type & Command | Purpose / Protocol Mechanism | When to Use It | When **NOT** to Use It |
| --- | --- | --- | --- |
| **ARP Scan**<br>

<br>`sudo nmap -PR -sn 10.200.6.0/24` | Uses Address Resolution Protocol requests. Local devices must respond to ARP to communicate on Layer 2. | **Local Subnets:** Always use this when scanning your immediate network segment. It is the fastest, most accurate method and completely bypasses host-based firewalls. | **External Networks:** Do not use this when scanning targets past a router or over the internet. ARP packets cannot cross router boundaries. |
| **ICMP Echo Scan**<br>

<br>`sudo nmap -PE -sn 10.200.6.0/24` | Sends a standard ICMP Type 8 (Echo Request), expecting a Type 0 (Echo Reply) back. | **Internal/Legacy Networks:** Ideal for scanning internal environments or corporate networks where ICMP is explicitly permitted for troubleshooting. | **Internet-Facing Targets:** Do not rely on this for external infrastructure. Modern edge firewalls and cloud providers (like AWS) almost universally drop ICMP Echo requests by default. |
| **ICMP Timestamp Scan**<br>

<br>`sudo nmap -PP -sn 10.200.6.0/24` | Sends an ICMP Type 13 request, asking the target for its current system time. | **Firewall Evasion:** Use this when a target blocks standard ICMP Echo (`-PE`) requests, as network administrators often forget to block alternate ICMP types. | **Strictly Patched Networks:** Avoid as your *only* discovery method on highly secured systems, as modern operating systems frequently disable or restrict timestamp responses. |
| **ICMP Address Mask Scan**<br>

<br>`sudo nmap -PM -sn 10.200.6.0/24` | Sends an ICMP Type 17 request to determine the target's subnet mask. | **Legacy Architecture:** Use this when hunting for older, unpatched systems or niche embedded devices within an enterprise network. | **Modern Systems:** Do not use on up-to-date networks. Most modern operating systems (Windows Vista/7/10/11, modern Linux kernels) reject or completely ignore Address Mask queries. |
| **TCP SYN Ping Scan**<br>

<br>`sudo nmap -PS22,80,443 -sn 10.200.6.0/30` | Sends an empty TCP packet with the `SYN` flag set to common ports. A response (`SYN/ACK` or `RST`) proves the host is live. | **Filtering Evasion & Remote Targets:** Use this to scan through standard stateful firewalls. It is highly effective for public-facing internet infrastructure when targeting common web ports (`80`, `443`). | **Silent/Stealth Operations:** Do not use if you are trying to avoid detection by an Intrusion Detection System (IDS), as half-open connection attempts to administrative ports like `22` or `443` easily trigger alerts. |
| **TCP ACK Ping Scan**<br>

<br>`sudo nmap -PA22,80,443 -sn 10.200.6.0/30` | Sends a spoofed TCP packet with the `ACK` flag set, mimicking an established connection. | **Stateless Firewall Evasion:** Use this to bypass older or poorly configured stateless firewalls that automatically allow packets marked as part of an active TCP session. | **Stateful Firewalls (SPI):** Do not use against modern stateful firewalls. A stateful firewall tracks active connections; it will instantly recognize the unsolicited `ACK` packet as invalid and silently drop it. |
| **UDP Ping Scan**<br>

<br>`sudo nmap -PU53,161,162 -sn 10.200.6.0/30` | Sends UDP packets to target ports. An active host responds with an ICMP "Port Unreachable" error. | **Bypassing TCP-Only Filters:** Use this when targets are heavily guarded by firewalls that filter all incoming TCP traffic but leave common UDP ports (like DNS `53` or SNMP `161`) accessible. | **High-Traffic/Fast Scans:** Do not use when speed is your top priority. UDP host discovery is inherently slower because it relies on ICMP error generation rates, which operating systems aggressively rate-limit. |

---

##  DNS Resolution & Behavioral Modifiers

By default, Nmap performs reverse-DNS lookups on all discovered active hosts to provide hostname context. You can modify this behavior using the following flags:

* **`-n` (No DNS Lookup):** Disables reverse-DNS translation entirely.
* *When to use:* Use this to drastically speed up scanning times, especially when enumerating large subnets or dealing with slow network DNS servers.
* *When NOT to use:* Do not use if you specifically need corporate hostnames (e.g., `HR-PC.local`) to help identify high-value targets during internal penetration testing.


* **`-R` (Reverse-DNS Lookup All):** Forces Nmap to attempt reverse-DNS resolution on *every single target IP address* specified in the range.
* *When to use:* Use when scanning highly restricted networks where hosts are completely silent, but the local DNS server still holds active pointer (PTR) records for them.
* *When NOT to use:* Do not use on massive subnets, as querying thousands of inactive IPs will cause severe scan delays.


* **`-sn` (Host Discovery Only):** Disables port scanning completely after host verification is complete.

---

##  Strategic Deployment Blueprint

* **On Local Subnets:** Always default to an **ARP Scan (`-PR`)**. It is native to layer-2 operations, incredibly fast, and cannot be blocked by host-based firewalls (like Windows Defender) on the same network segment.
* **Across Routers/Firewalls:** Combine **TCP SYN (`-PS`)** and **TCP ACK (`-PA`)** pings targeting common web and administrative ports (such as `80`, `443`, `22`). This dual approach maximizes the chances of penetrating stateful packet inspections and access control lists (ACLs).
