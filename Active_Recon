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
