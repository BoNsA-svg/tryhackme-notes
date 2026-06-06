Here is your clean, high-visibility quick-reference sheet for passive reconnaissance. I've cleaned up the formatting and organized the commands so you can easily copy-paste or glance at them while you're in the zone.

---

## Passive Reconnaissance Overview

> **Core Principle:** Gathering intelligence without direct interaction with the target. It is the stealthiest phase of recon, triggering no alerts and carrying minimal legal risk when used ethically.

### Key Tools & Techniques

* **WHOIS:** Uncovers domain registration details (registrar, creation/expiry dates, name servers). *Note: Most personal details are now redacted for privacy.*
* **DNS Lookups:** Queries public resolvers (like `1.1.1.1`) for specific record types:
* `A` / `AAAA`: IPv4 and IPv6 addresses.
* `MX`: Mail servers.
* `TXT`: Used for SPF, DMARC, and ownership verification.


* **Subdomain Enumeration:** * **DNSDumpster:** Excellent for DNS aggregation and visual graphing.
* **crt.sh:** Searches Certificate Transparency (CT) logs. This is the **most effective passive method** for discovering subdomains via public SSL/TLS certificates.


* **Exposed Services:** * **Shodan.io:** Used to find device banners, open ports, and hosting information without scanning the target yourself.

---

## Command Quick-Reference

Use this table for a quick syntax check. **`dig`** is the modern, recommended tool, while **`nslookup`** is legacy but still useful.

| Purpose | Command-line Example |
| --- | --- |
| **Lookup WHOIS record** | `whois tryhackme.com` |
| **Lookup DNS A records (Recommended)** | `dig tryhackme.com A` |
| **Lookup DNS MX records @ specific server (Rec.)** | `dig @1.1.1.1 tryhackme.com MX` |
| **Lookup DNS TXT records (Recommended)** | `dig tryhackme.com TXT` |
| **Lookup DNS A records (Legacy)** | `nslookup -type=A tryhackme.com` |
| **Lookup DNS MX records @ specific server (Legacy)** | `nslookup -type=MX tryhackme.com 1.1.1.1` |
| **Lookup DNS TXT records (Legacy)** | `nslookup -type=TXT tryhackme.com` |
| **Passive subdomain discovery (Browser)** | Visit `https://crt.sh` and search `%.tryhackme.com` |

---

## Pro Tips for Attickers & Defenders

### For the Attacker / Researcher

* **Privacy First:** Use DNS over HTTPS (DoH) or DNS over TLS (DoT) resolvers (like `1.1.1.1`) to keep your own reconnaissance queries private from local snooping.
* **Expect Fluidity:** Results change over time. IPs rotate (especially with anycast networks like Cloudflare), subdomains drift, and privacy redactions continue to tighten.

### For the Defender

* **Monitor Your Footprint:** Don't let attackers see it first. Set up Shodan/Censys alerts for your IP spaces.
* **Watch the Logs:** Monitor CT logs for any new certificates issued under your domains.
* **Track DNS:** Actively track DNS changes to prevent **subdomain takeover** risks (where a DNS record points to a dead cloud service that an attacker can reclaim).
