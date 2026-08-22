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

# Nmap Port Scanning: Core Mechanics & Timing Controls

## Overview

Once live hosts have been discovered, the next phase of reconnaissance is port scanning. This phase identifies open ports, determines what protocols are active (TCP or UDP), and maps out the target’s attack surface.

---

## 🛠️ Core Port Scan Types

The three fundamental scan types each interact with the target operating system's network stack differently to determine if a port is `open`, `closed`, or `filtered`.

| Scan Type & Command | Protocol Mechanism | When to Use It | When **NOT** to Use It |
| --- | --- | --- | --- |
| **TCP Connect Scan**<br>

<br>`nmap -sT 10.144.176.188` | Completes the full **TCP Three-Way Handshake** (`SYN` ➡️ `SYN/ACK` ⬅️ `ACK`). It relies on the underlying operating system's standard network API (`connect`). | **Non-Root Users:** Use this when you do not have administrative/root privileges on your attack machine, as it does not require raw packet crafting permissions. | **Stealth Operations:** Do not use if you want to remain covert. Because it completes the full connection, the target service logs the connection request, leaving a distinct digital footprint. |
| **TCP SYN Scan**<br>

<br>`sudo nmap -sS 10.144.176.188` | Often called a **"Half-Open" or Stealth Scan**. It sends a `SYN` packet and waits for a `SYN/ACK`. If received, Nmap immediately tears down the connection with a `RST` packet *before* the handshake finishes. | **Default TCP Scans:** This is the industry standard for fast, high-performance scanning. It is less likely to be logged by target applications because the connection is never fully established. | **No Root/Sudo Privileges:** Do not use if you cannot run commands as root, as Nmap requires raw socket access to craft these custom TCP headers. |
| **UDP Scan**<br>

<br>`sudo nmap -sU 10.144.176.188` | Sends raw UDP packets to target ports. If no response is received, the port is marked `open|filtered`. If it receives an ICMP Type 3 Code 3 error, the port is `closed`. | **Service Hunting:** Use this when explicitly auditing for common UDP services like DNS (`53`), SNMP (`161`), DHCP (`67/68`), or OpenVPN (`1194`). | **High-Speed Scans:** Do not use if you are in a rush. UDP scanning is inherently slow because it must wait for timeouts, and modern OS kernels strictly limit how fast they send ICMP error messages. |

---

## ⚙️ Port Selection Modifiers

By default, Nmap scans the top 1,000 most common ports. Use these modifiers to change the scope of your scan:

* **`-p-` (All Ports):** Scans the entire valid port range from **1 to 65535**. Use this to find stealthy services running on non-standard ports.
* **`-p1-1023` (Privileged Ports):** Scans the well-known system ports reserved for critical services (like SSH, HTTP, FTP).
* **`-F` (Fast Mode):** Scans only the top 100 most common ports, drastically slashing scan times.
* **`-r` (Consecutive Order):** Scans ports sequentially from lowest to highest, instead of randomizing the order (Nmap's default behavior to evade basic threshold detection).

---

## ⏱️ Timing & Performance Optimization

Controlling how fast Nmap transmits packets is essential for dodging security controls and managing network bandwidth.

### Timing Templates (`-T<0-5>`)

Nmap provides six predefined timing profiles. Higher numbers sacrifice stealth for speed:

* **`-T0` (Paranoid) & `-T1` (Sneaky):** Extremely slow. Used to completely evade Intrusion Detection Systems (IDS) by waiting up to 5 minutes between individual packets.
* **`-T2` (Polite):** Slows down the scan to consume less bandwidth and prevent crashing fragile or legacy systems.
* **`-T3` (Normal):** The default behavior. Balances speed and accuracy based on network responsiveness.
* **`-T4` (Aggressive):** Speeds up the scan significantly. Recommended for modern, reliable broadband or local lab environments.
* **`-T5` (Insane):** Maximum speed. Sends packets aggressively. Only use this on high-speed corporate networks where you don't care about making noise or dropping packets due to congestion.

### Granular Performance Controls

For fine-tuned control over your scanning signature, use these advanced options:

* **`--max-rate 50`**: Restricts Nmap from sending more than 50 packets per second. This is an excellent way to manually stay under the radar of automated firewall blocking rules.
* **`--min-rate 15`**: Guarantees Nmap sends at least 15 packets per second, ensuring your scan finishes within a predictable timeframe.
* **`--min-parallelism 100`**: Forces Nmap to run at least 100 probes in parallel. This is useful when optimizing performance across highly stable, fast local networks.

---

# Nmap Advanced Port Scanning & Evasion Techniques

## Overview

Advanced port scanning techniques manipulate TCP headers in non-standard ways to probe ports. These methods are primarily used to bypass stateless firewalls, evade Intrusion Detection Systems (IDS), or map out firewall filtering rules.

According to RFC 793, any packet sent to a **closed** port must prompt a `RST` response, while packets sent to an **open** port with unexpected flag combinations should be silently dropped. This behavior forms the foundation of Null, FIN, and Xmas scans.

---

##  Advanced & Inverse TCP Scan Types

These scans rely on setting specific TCP flags to observe how the target responds, allowing you to infer port states.

| Scan Type & Command | Flag Configurations | When to Use It | When **NOT** to Use It |
| --- | --- | --- | --- |
| **TCP Null Scan**<br>

<br>`sudo nmap -sN 10.144.173.120` | Sets **no flags** at all (all bits in the TCP flag byte are 0). | **Bypassing Stateless Firewalls:** Useful for sneaking past basic filters looking for standard handshake attempts (`SYN`). | **Windows Environments:** Do not use against Windows-based systems. Microsoft's TCP/IP stack ignores RFC 793 and responds with a `RST` packet regardless of whether the port is open or closed, breaking the logic of the scan. |
| **TCP FIN Scan**<br>

<br>`sudo nmap -sF 10.144.173.120` | Sets only the **`FIN` bit** (used normally to gracefully close a connection). | **IDS Evasion:** Good for slipping past older intrusion detection systems configured to flag standard `SYN` sweeps. | **Windows Targets:** Like the Null scan, Windows targets will incorrectly return a `RST` for both open and closed states. |
| **TCP Xmas Scan**<br>

<br>`sudo nmap -sX 10.144.173.120` | Sets the **`FIN`, `PSH`, and `URG` flags** simultaneously, lighting up the packet "like a Christmas tree." | **Defeating Linux/Unix Default Rules:** Effective against standard Linux/BSD/Unix systems to check for unfiltered, open ports without establishing a formal connection state. | **Modern Next-Gen Firewalls (NGFW):** Avoid when stealth is paramount. The unusual combination of `FIN+PSH+URG` is highly anomalous and instantly triggers alerts on modern firewalls. |
| **TCP Maimon Scan**<br>

<br>`sudo nmap -sM 10.144.173.120` | Sets the **`FIN` and `ACK` flags**. | **Probing Derived BSD Stacks:** Named after its discoverer, Uriel Maimon. It is tailored to find open ports on older, BSD-derived systems that treat this flag pattern differently than standard modern OS kernels. | **Modern Linux/Windows Networks:** Most modern operating systems drop or block this identically across open and closed ports, yielding uniform `RST` packets that provide no data. |
| **TCP ACK Scan**<br>

<br>`sudo nmap -sA 10.144.173.120` | Sets only the **`ACK` flag**. It does *not* determine if a port is open or closed. | **Mapping Firewall Rulesets:** Use this explicitly to map out firewall rules. If Nmap receives a `RST`, the port is **unfiltered**. If no response comes back (or an ICMP error is returned), the port is **filtered**. | **Service Enumeration:** Do not use if you are trying to find open applications, as the results only display `unfiltered` or `filtered`. |
| **TCP Window Scan**<br>

<br>`sudo nmap -sW 10.144.173.120` | Identical to an ACK scan, but it inspects the **TCP Window field** of the returning `RST` packet. | **Exploiting Specific OS Handshaking:** On certain operating systems, a closed port returns a Window size of `0`, while an open port returns a positive Window size, revealing open ports using only `ACK` packets. | **General Targeting:** Do not rely on this globally; it yields false positives or entirely hidden results on systems that do not differentiate Window sizes for error handling. |
| **Custom TCP Scan**<br>

<br>`sudo nmap --scanflags [FLAGS] 10.144.173.120` | Allows manual specification of any combination of TCP flags: `URG`, `ACK`, `PSH`, `RST`, `SYN`, `FIN`. | **Advanced Firewall Evading:** Use this if you have manually discovered a unique loophole or blindspot in a specific network security control or proprietary firewall. | **Standard Recon:** Avoid for basic operations due to the unnecessary complexity over built-in flags. |

---

##  IDS/IPS Evasion & Spoofing Modifiers

These options obscure your identity, mask the source of the traffic, or manipulate the packet structure to slide through network security appliances.

* **Spoofed Source IP (`-S SPOOFED_IP`):** Replaces your real IP with a fake one in the IP header.
* *Context:* The target replies directly to the spoofed IP. You will not see the responses unless you have a separate packet capture (`tcpdump`/`Wireshark`) running on a position where you can sniff the return traffic.


* **Spoofed MAC Address (`--spoof-mac SPOOFED_MAC`):** Changes your Layer 2 hardware address. Useful for bypassing router/switch Access Control Lists (ACLs) or MAC filtering on wireless/local networks.
* **Decoy Scan (`-D DECOY_IP1,DECOY_IP2,ME`):** Blends your real scanning traffic with packets sent from fake decoy IP addresses. The target's logs will see dozens of systems scanning them at once, hiding your true IP address in the noise.
* **Idle (Zombie) Scan (`-sI ZOMBIE_IP`):** An advanced, completely blind scan. It maps out open ports on a target without ever sending a packet from your actual IP. Instead, it monitors changes in the IP ID (`IPID`) fragment identification field of an idle third-party host ("the zombie").
* **Packet Fragmentation (`-f` or `-ff`):** Splits the TCP header across multiple tiny 8-byte (`-f`) or 16-byte (`-ff`) IP fragments. This splits the flag indicators up, causing older packet filters and deep packet inspection systems to miss the malicious flag combinations completely.

### Auxiliary Traffic Controls

* **`--source-port PORT_NUM`:** Forces Nmap to send probes from a specific port (e.g., DNS port `53` or HTTP port `80`). Many poorly configured firewalls explicitly trust all incoming traffic originating from standard service ports.
* **`--data-length NUM`:** Appends random, meaningless data to the packet payload to reach the specified byte size. This alters the predictable packet signature of Nmap, throwing off signature-based IDS alerts.

---

##  Investigation & Troubleshooting Flags

When experimenting with advanced or custom flags, standard output may become ambiguous. Use these switches to understand Nmap's decision-making process:

* **`--reason`:** Displays the explicit reason why Nmap classified a port into a specific state. For instance, it will tell you if it marked a port `open\|filtered` because of a "no-response" or `unfiltered` because it received a "reset (RST)".
* **`-v` / `-vv` (Verbose / Very Verbose):** Prints out open ports instantly as they are discovered rather than waiting for the entire scan execution cycle to finish.
* **`-d` / `-dd` (Debugging / Deep Debugging):** Displays low-level packet-handling data, raw timeouts, and system socket errors. Essential for troubleshooting when your custom flags or spoofing chains break network routes.

---

# Nmap Service, OS, and Script Enumeration

## Overview

After mapping out open ports, the next step is **fingerprinting**. This involves identifying the exact service versions running on those ports, detecting the host operating system, executing automated vulnerability scripts, and saving the gathered intelligence for documentation and report writing.

---

##  Service Version & OS Detection

Standard port scans only guess a service based on its port number (e.g., assuming port 22 is SSH). Fingerprinting forces Nmap to interact with the port using banners and protocol probes to determine exactly what is running.

| Option | Purpose / Protocol Mechanism | When to Use It | When **NOT** to Use It |
| --- | --- | --- | --- |
| **`-sV`** | **Service Version Detection:** Interrogates open ports with specific probes to determine service names and exact version numbers. | **Vulnerability Research:** Crucial for looking up public exploits (CVEs). Knowing a target runs `Apache httpd 2.4.41` rather than just "HTTP" lets you pinpoint exact security flaws. | **Ultra-Stealth/Fast Operations:** Do not use if speed is critical. It establishes connections and exchanges data with every open port, making it slower and easier for network monitors to spot. |
| **`-sV --version-light`** | **Lightweight Probes:** Restricts Nmap to only its most common service probes (Intensity Level 2). | **Time-Constrained Scans:** Great when you need version info quickly across a larger pool of open ports without waiting for exhaustive protocol checks. | **Obscured Services:** Avoid if you suspect the target is running services on non-standard ports, as light probes might fail to identify them. |
| **`-sV --version-all`** | **Exhaustive Probes:** Forces Nmap to test every single available version probe (Intensity Level 9) against the open ports. | **Hardened/Custom Targets:** Use this when standard version scanning returns an "unknown service" or when a target is heavily modified to hide its identity. | **Standard Audits:** Do not use unless necessary, as it adds a massive amount of overhead traffic and can significantly drag down scan completion times. |
| **`-O`** | **OS Detection:** Analyzes low-level characteristics of the TCP/IP stack (like TTL values and TCP window sizes) within returning packets to guess the operating system. | **Exploit Tailoring:** Essential for deciding whether to weaponize an attack path for Windows or Linux architectures. | **Firewalled/Filtered Environments:** Do not rely heavily on this if the target is behind a proxy or strict firewalls, as security appliances often mask or sanitize stack signatures. |
| **`--traceroute`** | **Network Path Tracking:** Traces the network hop sequence to the target system concurrently with the scan. | **Network Mapping:** Excellent for understanding the path your packets take and identifying intermediary security layers or routers. | **Local Subnets:** Completely redundant when scanning devices on your immediate layer-2 local network segment. |

---

##  Automating Enumeration with Nmap Scripting Engine (NSE)

The Nmap Scripting Engine (NSE) allows you to run automated scripts written in Lua to conduct advanced vulnerability assessments, service discovery, and credential brute-forcing.

* **`--script=default` or `-sC`:** Executes Nmap’s curated set of default safe scripts. These perform basic vulnerability checks, grab detailed banner data, and enumerate open configurations without crashing the target.
* **`--script=SCRIPTS`:** Runs a user-specified script or category of scripts (e.g., `--script=vuln`, `--script=http-enum`, or a custom local script file).
* **`-A` (Aggressive Scan Mode):** A comprehensive, high-utility macro flag. Specifying **`-A`** instructs Nmap to automatically enable four heavy-hitting options at once:
* Service Version Detection (`-sV`)
* Operating System Detection (`-O`)
* Default NSE Scripts (`-sC`)
* Traceroute (`--traceroute`)
* *Note:* While highly effective for a quick, complete overview of a single target machine, it is incredibly noisy and easily detected by defenders.



---

##  Structuring & Saving Scan Outputs

Maintaining a record of your scan results is vital for evidence keeping and passing data into other security tools. Nmap allows you to output your findings in distinct formats using the options below:

```text
                  ┌───►  -oN  (Normal: Clean terminal-style format)
                  ├───►  -oG  (Grepable: Single-line format for regex/command line filtering)
┌──────────────┐  │
│ Nmap Output  ├──┼───►  -oX  (XML: Interoperable format for Metasploit, Burp, or reporting engines)
└──────────────┘  │
                  └───►  -oA  (All formats: Generates all three styles at the same time)

```

* **`-oN` (Normal Output):** Saves the findings in the clean, plain-text layout you see in your command-line terminal. Ideal for quick manual reading.
* **`-oG` (Grepable Output):** Formats all target info and open ports onto a single line per host. This is specifically designed for command-line parsing using tools like `grep`, `awk`, or `cut`.
* **`-oX` (XML Output):** Structures your data into formal XML. This is the preferred format for programmatically importing your scan data straight into automated toolsets like the Metasploit Framework, Burp Suite, or custom parsing scripts.
* **`-oA` (All Formats):** A master backup switch. It saves your current scan into three separate files simultaneously using your designated base name (e.g., `scan_results.nmap`, `scan_results.gnmap`, and `scan_results.xml`), ensuring you have the data structured for any future workflow.

---

Awesome job cracking that order! It's definitely a great pattern to document. Let's create a clean, professional cheat sheet for your reference notes so you can quickly review this scanning methodology during your upcoming labs and projects.

---

# Nmap Scanning Methodology: Stealth & Evasion

When executing an engagement against a secured network perimeter, running a blind or overly aggressive scan will instantly trigger security alerts. A comprehensive, professional workflow prioritizes configuration, masking, and rule analysis before diving into heavy port scanning.

##  The Correct Logical Sequence

### 1️ Configure packet fragmentation using `-f` option

* **Purpose:** Network Layer Evasion
* **Why it's first:** Before a single packet ever hits the target infrastructure, you must configure your local tool parameters. Splitting the IP header across several smaller packets (fragmentation) forces simple firewalls and Intrusion Detection Systems (IDS) to process them individually, often allowing them to slip past unexamined.
* **Nmap Implementation:** `nmap -f [Target]`

### 2️ Launch decoy scan to obscure the true source

* **Purpose:** Identity Obfuscation
* **Why it's second:** With your packet structures modified for evasion, you next layer your source masking. By configuring a pool of active decoy IP addresses, your actual scanning traffic blends into a crowd of spoofed addresses. If security analysts notice the scan, they cannot easily determine which IP is the real attacker.
* **Nmap Implementation:** `nmap -D RND:10,ME [Target]` *(Generates 10 random decoys and blends your IP in)*

### 3️ Execute ACK scan to map firewall rules

* **Purpose:** Defensive Perimeter Mapping
* **Why it's third:** Now that your stealth and masking parameters are live, you probe the perimeter's defenses. An ACK scan doesn't look for open ports; it sends a TCP packet with only the ACK flag set. Stateful firewalls drop these, while stateless firewalls or open systems respond with a RST packet—allowing you to map exactly which ports are *filtered* vs. *unfiltered*.
* **Nmap Implementation:** `nmap -sA [Target]`

### 4️ Perform initial TCP SYN scan to identify open ports

* **Purpose:** Half-Open Target Enumeration
* **Why it's last:** Now that you have modified your packets, masked your identity, and mapped the firewall rules standing in your way, you can safely launch the core stealth scan. The SYN scan (half-open scan) targets the unfiltered channels to determine exactly which operational services are open and listening without completing the noisy 3-way handshake.
* **Nmap Implementation:** `nmap -sS [Target]`

---

##  Quick-Reference Command Blueprint

When you are combining these exact steps into a unified, weaponized command line on your terminal, it looks like this:

```bash
sudo nmap -sS -sA -f -D RND:5 [Target_IP]

```

* **`-sS`** ➡️ Tells Nmap to perform the target SYN stealth scan.
* **`-sA`** ➡️ Evaluates the firewall state tracking alongside the scan.
* **`-f`** ➡️ Fragments the outgoing packets.
* **`-D RND:5`** ➡️ Wraps the entire operation inside 5 random decoy addresses.

---

# Windows & Active Directory Credential Harvesting — Engagement Note

## 1. Executive Summary

Credential harvesting is a high-value post-compromise activity in Windows environments. Once an operator obtains local Administrator or SYSTEM-level access, multiple credential stores may become accessible, including:

* **LSASS memory**
* **SAM + SYSTEM registry hives**
* **LSA Secrets**
* **DPAPI-protected credentials**
* **Cached domain credentials (MSCache v2 / DCC2)**
* **NTDS.dit on Domain Controllers**

During an assessment, these stores can provide password hashes, Kerberos material, cached credentials, service-account credentials, and—in some configurations—plaintext credentials.

The assessment path demonstrated in the supplied material is:

**Local Administrator → credential stores → cached domain credentials → crackable DCC2 credential → domain account → Domain Administrator → NTDS.dit → NTLM hash → Pass-the-Hash → SYSTEM on DC**

The important engagement takeaway is that **no software vulnerability or privilege-escalation exploit was required**. Existing credential exposure and excessive credential reuse were sufficient to progress toward domain compromise.

---

# 2. Credential Store Matrix

| Credential Store      | Primary Contents                                                               | Typical Access Level                                     | Assessment Value                           |
| --------------------- | ------------------------------------------------------------------------------ | -------------------------------------------------------- | ------------------------------------------ |
| **LSASS**             | NTLM/SHA1 material, Kerberos tickets, sometimes plaintext credentials          | Administrator/SYSTEM                                     | Current interactive-session credentials    |
| **SAM**               | Local-account password hashes                                                  | SYSTEM / administrative access                           | Local account compromise                   |
| **SYSTEM hive**       | BootKey material used to decrypt SAM                                           | SYSTEM / administrative access                           | Required alongside SAM                     |
| **LSA Secrets**       | Cached credentials, service/scheduled-task secrets and other protected secrets | SYSTEM                                                   | Service/domain credential recovery         |
| **DPAPI**             | Application/user secrets, saved credentials                                    | User context and/or access to appropriate DPAPI material | Recovery of stored application credentials |
| **MSCache v2 / DCC2** | Cached domain-logon secrets                                                    | Local administrative access                              | Offline password cracking                  |
| **NTDS.dit**          | Domain accounts, NTLM hashes and Kerberos key material                         | Domain replication privileges / DC-level access          | Domain-wide credential compromise          |

---

# 3. Initial Access & Privilege Context

The assessment begins with **local Administrator access to a domain-joined workstation (WRK)**.

This distinction is important:

> Local Administrator access does **not** automatically mean Domain Administrator access.

Instead, the workstation becomes a credential-collection point. The objective is to determine whether users with higher privileges have authenticated to the workstation previously or whether privileged credentials have been stored there.

### Assessment questions

During an engagement, establish:

1. What local accounts exist?
2. Which users have authenticated to the workstation?
3. Are privileged/domain-admin accounts present?
4. Are service credentials stored locally?
5. Are cached domain credentials available?
6. Can stored credentials be recovered?
7. Can recovered credentials authenticate elsewhere?

---

# 4. LSASS Memory

## Purpose

**LSASS (`lsass.exe`)** is responsible for Windows authentication and security-policy enforcement.

The supplied material identifies LSASS as potentially containing:

* NTLM hashes
* LM hashes
* Kerberos TGTs
* Kerberos service tickets
* Plaintext credentials in some configurations
* Credential-manager material

Because LSASS contains live authentication material, it is one of the highest-value credential stores on a compromised Windows host.

## Mimikatz assessment

The supplied workflow uses:

```text
privilege::debug
sekurlsa::logonpasswords
```

`privilege::debug` enables the privilege required for accessing protected process memory, while `sekurlsa::logonpasswords` enumerates credential material associated with current logon sessions.

### Engagement interpretation

A successful LSASS extraction should be reviewed for:

* Domain usernames
* Local usernames
* NTLM hashes
* Kerberos material
* Plaintext passwords
* Credential Manager entries

### Important limitation

**LSASS is a live credential source.**

If a privileged user previously authenticated but no longer has an active session, their credentials may no longer be present in LSASS.

Therefore:

> **Absence of a domain-admin credential in LSASS does not establish that the workstation is free of privileged credentials.**

Move to persistent credential stores such as cached credentials, DPAPI, LSA Secrets, and registry hives.

---

# 5. SAM + SYSTEM

## SAM

The **Security Account Manager (SAM)** contains password hashes for local Windows accounts.

Typical targets include:

* Local Administrator
* Local users
* Other locally created accounts

The SAM itself is protected, so the **SYSTEM hive** is required because it contains the BootKey material used to decrypt the SAM database.

Relevant locations:

```text
%SystemRoot%\System32\config\SAM
%SystemRoot%\System32\config\SYSTEM
```

## Assessment workflow

The supplied material demonstrates exporting both hives:

```powershell
reg save HKLM\SAM C:\Users\Administrator\Desktop\SAM
reg save HKLM\SYSTEM C:\Users\Administrator\Desktop\SYSTEM
```

Mimikatz can then process the exported material:

```text
lsadump::sam /sam:"C:\Users\Administrator\Desktop\SAM" /system:"C:\Users\Administrator\Desktop\SYSTEM"
```

### Expected result

The important artifact is the **NTLM hash associated with local accounts**.

### Engagement significance

Recovered local NTLM hashes may support:

* Offline password analysis
* Password-reuse investigation
* Authentication testing where authorized
* Identification of weak/common local administrator passwords

A particularly important finding is **shared local administrator credentials across multiple systems**, because compromise of one workstation can then facilitate movement to others.

---

# 6. LSA Secrets

LSA Secrets are stored beneath:

```text
HKLM\SECURITY\Policy\Secrets
```

The supplied material identifies potential contents including:

* Cached domain credentials
* Service-account credentials
* Scheduled-task credentials
* Other protected authentication material

LSA Secrets generally require elevated privileges, with **SYSTEM-level access** being particularly important for extraction.

### Mimikatz workflow

The supplied workflow demonstrates elevating to SYSTEM:

```text
privilege::debug
token::elevate
lsadump::cache
```

The result can include **MSCache v2 / DCC2** values associated with domain users who have logged on to the workstation.

### Engagement interpretation

A workstation may therefore contain evidence of privileged users even when those users have no current interactive session.

This is a major distinction:

**LSASS = live credentials**

**Cached credentials = credentials retained for offline authentication**

---

# 7. DPAPI

## Purpose

Windows **Data Protection API (DPAPI)** protects user-specific secrets.

The supplied material identifies examples including:

* RDP credentials
* Browser/application credentials
* Wi-Fi credentials
* Windows Credential Manager data

DPAPI protection is tied to the user's cryptographic context and, depending on the credential, may require the appropriate user context/master keys to decrypt.

## Mimikatz

The supplied workflow uses:

```text
vault::list
vault::cred /export
```

The assessment demonstrated that stored credentials could include entries such as:

```text
TRYHACKME\svc-app
ElonTusk
```

### Important assessment distinction

Being **Local Administrator** does not necessarily mean every DPAPI secret can immediately be decrypted.

The supplied example specifically demonstrates that a credential belonging to another user/service account may remain protected because the corresponding DPAPI material is tied to that user's context.

Therefore, document separately:

* **Credential discovered**
* **Credential successfully decrypted**
* **Credential identified but still protected**

Do not treat these as equivalent findings.

---

# 8. MSCache v2 / DCC2

One of the most valuable findings in the supplied attack path is **cached domain credentials**.

MSCache v2 exists so domain users can authenticate when a workstation cannot contact a Domain Controller.

Example format:

```text
$DCC2$10240#username#hash
```

### Critical distinction

DCC2 hashes are **not equivalent to NTLM hashes for Pass-the-Hash purposes**.

They are designed for offline password verification and therefore are primarily useful for:

> **Offline password cracking**

rather than direct PtH authentication.

---

# 9. Offline Cracking Assessment

The supplied workflow uses John the Ripper with the MSCache2 format:

```bash
john --format=mscash2 dc2_hash.txt \
    --wordlist=/usr/share/wordlists/rockyou.txt
```

The assessment objective is to determine whether the cached domain credential can be recovered using an authorized password dictionary.

### Risk interpretation

A cracked DCC2 credential becomes significantly more valuable when the account:

* Has access to additional systems
* Has administrative privileges
* Has excessive permissions
* Reuses the password elsewhere
* Is a privileged/domain-admin account

This is where credential harvesting can become **credential-based lateral movement**.

---

# 10. Domain Controller Credential Extraction

Once an account possessing appropriate domain privileges is obtained, the assessment can move from workstation credential stores to the **Domain Controller**.

The supplied material demonstrates the Impacket `secretsdump.py` workflow with:

```text
-just-dc
```

This performs domain credential extraction through the **DRSUAPI** mechanism rather than dumping the workstation's local SAM/LSA data.

The resulting material can include:

* Domain usernames
* RIDs
* NTLM hashes
* Kerberos keys

### NTDS.dit significance

`NTDS.dit` is the Active Directory database on a Domain Controller.

It represents the authentication database for the domain and therefore has substantially greater impact than a workstation's SAM database.

Compromise of domain credential material can expose authentication material for:

* Domain users
* Service accounts
* Computer accounts
* Privileged administrators

---

# 11. Pass-the-Hash

A recovered **NTLM hash** can potentially be used for authentication without recovering the plaintext password, where the target protocol and configuration permit it.

The supplied workflow demonstrates using an administrative NTLM hash with Impacket `psexec`.

The resulting session demonstrates:

```text
whoami
nt authority\system
```

### Engagement significance

This establishes the critical distinction between:

**Password compromise**

and

**Credential-material compromise**

An attacker does not necessarily need the plaintext password if a reusable authentication secret such as an NTLM hash is available.

---

# 12. Attack Chain

The complete chain demonstrated by the supplied material can be represented as:

```text
Local Administrator
        │
        ▼
   Credential Stores
        │
        ├── LSASS
        │
        ├── SAM + SYSTEM
        │
        ├── LSA Secrets
        │
        └── DPAPI
        │
        ▼
Cached Domain Credentials
        │
        ▼
       DCC2
        │
        ▼
 Offline Password Cracking
        │
        ▼
Higher-Privilege Domain Account
        │
        ▼
Domain Controller
        │
        ▼
NTDS / Domain Credential Material
        │
        ▼
NTLM Hash
        │
        ▼
 Pass-the-Hash
        │
        ▼
SYSTEM on Domain Controller
        │
        ▼
Domain Compromise
```

---

# 13. Findings to Capture During an Engagement

For each recovered credential or hash, maintain an evidence table similar to:

| Host | Account       | Credential Type | Source             | Privilege       | Reusable?            | Impact      |
| ---- | ------------- | --------------- | ------------------ | --------------- | -------------------- | ----------- |
| WRK  | Administrator | NTLM            | SAM                | Local Admin     | Potentially          | High        |
| WRK  | svc-app       | Password        | DPAPI/LSASS        | Service account | Validate             | High        |
| WRK  | Domain user   | DCC2            | Cached credentials | Domain user     | Crack offline        | Medium/High |
| DC   | Administrator | NTLM            | NTDS               | Domain Admin    | Yes, where supported | Critical    |

Avoid storing recovered plaintext credentials in the report unless necessary. Use controlled evidence storage and redact secrets from the final client-facing report.

---

# 14. Key Engagement Lessons

### 1. Local Administrator is a credential-collection position

Local Administrator access should immediately trigger an assessment of credential exposure.

### 2. LSASS is only one source

Failure to obtain credentials from LSASS does **not** mean the host contains no valuable credentials.

Check:

```text
LSASS
SAM + SYSTEM
LSA Secrets
DPAPI
Cached credentials
```

### 3. Cached credentials can bridge privilege boundaries

A privileged user may have authenticated to a workstation previously, leaving cached authentication material even after their interactive session has ended.

### 4. DCC2 requires cracking

DCC2 is fundamentally different from an NTLM hash for authentication purposes. Its principal assessment value is determining whether the underlying password can be recovered offline.

### 5. NTDS compromise changes the scope

Once domain credential material is obtained from a DC, the assessment has moved from **host compromise** toward **domain compromise**.

### 6. Credential reuse is the multiplier

The most important question after recovering any credential is:

> **Where else does this credential work?**

A low-privilege credential can become high impact if it is reused on privileged systems.

---

# 15. Defensive Recommendations

The demonstrated attack chain also gives a clear remediation roadmap.

### Credential protection

* Deploy **LSASS protections** appropriate to the Windows version and environment.
* Minimize unnecessary administrative access.
* Use separate administrative accounts rather than privileged accounts for routine workstation activity.
* Prevent privileged accounts from interactively logging onto ordinary workstations where possible.

### Local administrator security

* Eliminate shared local Administrator passwords.
* Deploy **Windows LAPS / Microsoft LAPS** to provide unique, managed local administrator credentials.
* Rotate local administrative credentials regularly.

### Service accounts

* Avoid storing static service-account passwords where possible.
* Prefer **gMSAs** where appropriate.
* Review scheduled tasks and services for embedded credentials.
* Reduce service-account privileges.

### Domain administration

* Restrict Domain Admin logons to dedicated administrative systems.
* Apply a tiered administrative model.
* Monitor privileged authentication to workstations.
* Protect Domain Controllers as a separate security tier.

### Credential monitoring

Monitor for suspicious activity involving:

* LSASS access
* SAM/SYSTEM hive extraction
* Remote Registry activity
* DRSUAPI/DCSync behavior
* Unusual administrative SMB activity
* Remote service creation
* Pass-the-Hash patterns
* Credential-dumping tools

---

# 16. Engagement Conclusion

The assessment demonstrates that **credential exposure can provide a complete path from workstation compromise to domain compromise without exploiting a software vulnerability**.

The critical sequence was:

> **Local Administrator → credential extraction → cached domain credential → password recovery → privileged domain access → NTDS credential extraction → NTLM reuse → SYSTEM on the Domain Controller**

From a penetration-testing perspective, the most significant finding is therefore not simply that credential-dumping tools work. The deeper issue is that **privileged authentication material was accessible from a workstation that had already been compromised**.

The primary remediation objective should be to break this chain at multiple points: **reduce privileged logons to workstations, protect credential material, eliminate password reuse, secure service accounts, deploy unique local administrator credentials, and tightly restrict access to Domain Controllers and domain credential replication mechanisms.**
