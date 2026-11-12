# HTB Academy — Network Enumeration with Nmap

> Source-organized study note for the Hack The Box Academy Penetration Tester path.

## Core Principle

Enumeration is not a race to run every scanner. The objective is to identify every meaningful way a target can be reached, questioned, or influenced. Tools accelerate collection, but accurate conclusions require understanding the protocol, validating responses manually, and noticing what automation misses.

## Nmap Capabilities

- Host discovery
- TCP and UDP port scanning
- Service and version detection
- Operating-system fingerprinting
- Nmap Scripting Engine (NSE) interaction
- Firewall and IDS/IPS behavior analysis

General syntax:

```bash
nmap <scan-types> <options> <target>
```

## Host Discovery

Always save results for comparison, evidence, and reporting.

```bash
# Subnet discovery
sudo nmap -sn 10.129.2.0/24 -oA discovery

# Target list
sudo nmap -sn -iL hosts.txt -oA discovery-list

# Single target with reason and packet trace
sudo nmap -sn -PE --reason --packet-trace <TARGET> -oA host

# Force ICMP rather than local ARP discovery
sudo nmap -sn -PE --disable-arp-ping <TARGET>
```

On a local Ethernet segment, Nmap commonly identifies hosts through ARP before ICMP. A missing response never proves that a host is offline; filtering and discovery method matter.

## Port States

| State | Meaning |
|---|---|
| `open` | A service accepted the probe or connection. |
| `closed` | The host responded, but no service is listening; TCP commonly returns RST. |
| `filtered` | Filtering prevents Nmap from deciding whether the port is open or closed. |
| `unfiltered` | ACK scan can reach the port but cannot determine open versus closed. |
| `open|filtered` | No response distinguishes an open UDP-style result from filtering. |
| `closed|filtered` | Idle scan cannot distinguish the two states. |

## TCP Scanning

```bash
# SYN scan; raw packets, normally requires privileges
sudo nmap -sS -p- <TARGET> -oA tcp-syn

# Full TCP connect scan
nmap -sT -p 443 <TARGET>

# Specific, ranged, top, and fast port sets
nmap -p 22,80,443 <TARGET>
nmap -p 1-1000 <TARGET>
nmap --top-ports 100 <TARGET>
nmap -F <TARGET>

# Explain packet-level behavior
sudo nmap -p 21 --packet-trace -Pn -n --disable-arp-ping <TARGET>
```

A SYN scan interprets SYN-ACK as open, RST as closed, and silence or certain ICMP errors as filtered. A connect scan completes the operating system's TCP connection and is easier to log.

## UDP Scanning

```bash
sudo nmap -sU --top-ports 100 <TARGET> -oA udp-top100
sudo nmap -sU -sV -p 53,67,68,69,123,161 <TARGET> -oA udp-services
```

UDP is slower and often ambiguous. An ICMP port-unreachable response indicates closed; a UDP response indicates open; silence commonly becomes `open|filtered`. Version detection can help provoke a protocol-specific response.

## Service Enumeration and Banner Grabbing

Use a fast scan first, then run detailed probes only against discovered ports while an all-port scan continues when scope permits.

```bash
sudo nmap -p- <TARGET> -oA all-ports
sudo nmap -sC -sV -p <PORTS> <TARGET> -oA services
sudo nmap -sV --stats-every 5s -p- <TARGET>

# Manual validation
nc -nv <TARGET> <PORT>
curl -i http://<TARGET>:<PORT>/
openssl s_client -connect <TARGET>:<PORT>

# Observe traffic when a service responds slowly or unexpectedly
sudo tcpdump -i <INTERFACE> host <TARGET>
```

Version strings are evidence, not guarantees. Confirm them through service behavior, banners, protocol responses, and—when needed—packet capture.

## Saving and Reporting Results

```bash
# Normal, grepable, and XML output together
sudo nmap -p- <TARGET> -oA target

# Convert XML to a readable HTML report
xsltproc target.xml -o target.html
```

| Option | Output |
|---|---|
| `-oN` | Normal text (`.nmap`) |
| `-oG` | Grepable text (`.gnmap`) |
| `-oX` | XML (`.xml`) |
| `-oA` | All three formats |

## Nmap Scripting Engine

```bash
# Default scripts
sudo nmap -sC -p <PORTS> <TARGET>

# Named scripts
sudo nmap -p 25 --script banner,smtp-commands <TARGET>

# Category — confirm authorization and impact first
sudo nmap -sV -p <PORTS> --script vuln <TARGET>
```

NSE categories include `auth`, `broadcast`, `brute`, `default`, `discovery`, `dos`, `exploit`, `external`, `fuzzer`, `intrusive`, `malware`, `safe`, `version`, and `vuln`. Categories such as `dos`, `exploit`, `intrusive`, `brute`, and `fuzzer` require explicit scope and impact review.

## Performance and Accuracy

```bash
nmap -F --initial-rtt-timeout 50ms --max-rtt-timeout 100ms <RANGE>
nmap -F --max-retries 2 <RANGE>
nmap -F --min-rate 300 <RANGE>
nmap -F -T4 <RANGE>
```

Faster settings trade completeness and stealth for speed. Short timeouts and few retries can miss hosts or ports. High rates and aggressive timing can trigger controls or distort results. Compare optimized scans with a slower baseline.

## Firewall and IDS/IPS Awareness

Firewalls filter packets; IDS detects suspicious patterns; IPS can actively block traffic. Different probe types may reveal different filtering behavior.

```bash
# SYN versus ACK comparison
sudo nmap -sS -p <PORTS> --reason <TARGET>
sudo nmap -sA -p <PORTS> --reason <TARGET>

# Decoy scan — only when explicitly authorized
sudo nmap -sS -D RND:5 -p <PORTS> <TARGET>

# Test whether source-port rules affect a probe
sudo nmap -sS --source-port 53 -p <PORTS> <TARGET>
```

ACK scans help map filtering rules but do not determine whether a service is open. Decoys do not make a scan invisible; traffic still comes from the real tester and may create alerts. Evasion testing must be explicitly included in the Rules of Engagement.

## Operational Workflow

1. Confirm scope, permitted rates, and whether IDS/IPS testing is authorized.
2. Discover hosts with methods appropriate to the network position.
3. Run an initial targeted scan and save it.
4. Scan all TCP ports and relevant UDP ports.
5. Detect services and versions on confirmed ports.
6. Validate important findings manually.
7. Use carefully selected NSE scripts.
8. Compare results, record uncertainty, and re-enumerate when evidence conflicts.

## Sources

- HTB Academy: Network Enumeration with Nmap
- Nmap Reference Guide and NSE documentation linked throughout the Academy material

> Use only against systems you own or have explicit authorization to assess.
