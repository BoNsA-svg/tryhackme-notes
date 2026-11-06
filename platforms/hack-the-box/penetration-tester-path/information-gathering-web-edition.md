# Information Gathering — Web Edition

> Source: Hack The Box Academy — Penetration Tester Path  
> Difficulty: Easy · Tier 2 · 19 sections  
> Scope: authorized web reconnaissance.

## Module Goals

Web reconnaissance builds an evidence-based inventory of a target's public presence before vulnerability analysis. The module combines passive and active techniques:

- WHOIS and registration intelligence;
- DNS records, subdomains, zone transfers, CT logs, and virtual hosts;
- HTTP and technology fingerprinting;
- crawling, `robots.txt`, sitemaps, and `.well-known` metadata;
- search-engine discovery and historical archives;
- repeatable automation.

## Passive vs Active Recon

| Type | Interaction | Examples |
|---|---|---|
| Passive | Uses third-party/public sources without touching target infrastructure directly | WHOIS, CT logs, search engines, archives |
| Active | Sends traffic to target-controlled systems | DNS queries, AXFR attempts, vhost brute forcing, fingerprinting, crawling |

Passive does not mean risk-free, and public data does not automatically make an action in scope. Rate limits and written authorization still govern active work.

## WHOIS

WHOIS/RDAP can expose:

- registrar and registry data;
- creation, update, and expiry dates;
- domain status protections;
- authoritative name servers;
- organization or privacy-service details;
- abuse contacts.

Treat registration age, privacy, registrar, and hosting as signals rather than conclusions. Historical WHOIS can reveal infrastructure and ownership changes.

```bash
whois DOMAIN
```

## DNS

Important records:

| Record | Purpose |
|---|---|
| A / AAAA | IPv4 / IPv6 address |
| CNAME | alias to another hostname |
| MX | mail exchanger |
| NS | authoritative name server |
| TXT | SPF, DMARC, verification, service clues |
| SOA | zone authority and timing information |
| SRV | service host and port |
| PTR | reverse mapping |

```bash
dig DOMAIN A
dig DOMAIN AAAA
dig DOMAIN MX
dig DOMAIN NS
dig DOMAIN TXT
dig DOMAIN SOA
dig +short DOMAIN
dig +trace DOMAIN
dig -x IP
dig @1.1.1.1 DOMAIN A
dig +noall +answer DOMAIN
```

## Subdomain Discovery

### Passive

- Certificate Transparency logs such as crt.sh;
- search engines and indexed URLs;
- passive DNS and OSINT sources;
- historical archives.

```bash
curl -s "https://crt.sh/?q=%25.DOMAIN&output=json" |
jq -r '.[].name_value' | sed 's/\*\.//g' | sort -u
```

### Active

Use general, targeted, and custom wordlists. Validate wildcard DNS before accepting results.

```bash
dnsenum --enum DOMAIN   -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -r
```

Potential tools: dnsenum, fierce, dnsrecon, Amass, assetfinder, puredns, ffuf, and gobuster.

### Zone Transfer

AXFR is intended for replication between authoritative servers. A misconfigured server may disclose the full zone.

```bash
dig DOMAIN NS +short
dig AXFR @NS_SERVER DOMAIN
```

A failed transfer is normal. A successful transfer requires careful evidence handling because it can reveal internal-looking names and IPs.

## Virtual Hosts

Subdomains are DNS names; virtual hosts are web-server routing identities. A vhost may exist even without public DNS.

```bash
gobuster vhost -u http://TARGET   -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt   --append-domain
```

Filter wildcard/default responses by status, length, words, lines, title, or body hash. Add confirmed names to a local hosts file only in authorized labs or when scope permits.

## Certificate Transparency

CT logs are append-only public records of issued TLS certificates. They help discover active, expired, historical, and hard-to-guess subdomains and may reveal naming conventions. Wildcards and duplicate SANs require normalization and DNS validation.

## Fingerprinting

Combine multiple weak signals:

- HTTP headers and redirects;
- cookies and error pages;
- HTML, script paths, metadata, generator tags, and API links;
- TLS certificate details;
- WAF behavior;
- automated signatures.

```bash
curl -I http://DOMAIN
curl -k -I https://DOMAIN
whatweb https://DOMAIN
wafw00f https://DOMAIN
nikto -h https://DOMAIN
```

Follow redirect chains deliberately and record each host. Tool findings are hypotheses until manually confirmed.

## Crawling and Discovery Files

Crawling follows known links; fuzzing guesses unknown paths. Collect:

- internal and external links;
- forms and parameters;
- JavaScript and external files;
- comments and metadata;
- emails;
- backup, configuration, log, and document references;
- images, audio, and video;
- response codes and content types.

```bash
curl -s https://DOMAIN/robots.txt
curl -s https://DOMAIN/sitemap.xml
curl -s https://DOMAIN/.well-known/security.txt
curl -s https://DOMAIN/.well-known/openid-configuration
```

`robots.txt` is crawler guidance, not access control. Disallowed paths are leads, not permission to bypass authorization.

The OpenID discovery document can reveal issuer, authorization/token/userinfo endpoints, JWKS URI, scopes, response types, and signing algorithms.

### ReconSpider

```bash
wget -O ReconSpider.zip <AUTHORIZED_SOURCE_URL>
unzip ReconSpider.zip
python3 ReconSpider.py http://DOMAIN
jq . results.json
```

Review emails, links, files, scripts, forms, media, and comments; do not treat automated output as verified.

## Search Engine Discovery

Useful operators:

```text
site:DOMAIN
site:DOMAIN inurl:login
site:DOMAIN (inurl:admin OR inurl:portal)
site:DOMAIN filetype:pdf
site:DOMAIN (filetype:xls OR filetype:docx)
site:DOMAIN (ext:conf OR ext:cnf)
site:DOMAIN filetype:sql
site:DOMAIN intitle:"index of"
site:DOMAIN "password reset"
```

Search results may be stale or out of scope. Validate ownership before interacting with discovered assets.

## Web Archives

The Wayback Machine can reveal old URLs, files, subdomains, parameters, technologies, staff references, and retired interfaces. Historical exposure remains useful as a lead but must be validated against the current system.

## Automation

Frameworks such as FinalRecon, Recon-ng, theHarvester, and SpiderFoot improve speed, consistency, and coverage. Automation does not replace analysis.

```bash
./finalrecon.py --headers --sslinfo --whois --url https://DOMAIN
./finalrecon.py --dns --sub --crawl --wayback --url https://DOMAIN
```

Tune threads, timeouts, wordlists, redirects, DNS servers, and output location to the Rules of Engagement. Preserve raw output and manually validate important findings.

## End-to-End Workflow

```text
Scope and identifiers
  → WHOIS/RDAP
  → DNS and CT logs
  → resolve and normalize subdomains
  → test authorized AXFR
  → identify live HTTP(S)
  → discover vhosts
  → fingerprint headers/TLS/WAF/technology
  → inspect robots, sitemap, and .well-known
  → crawl and analyze JS/files/forms
  → search engines and archives
  → automate for coverage
  → correlate, validate, prioritize, and report
```

## Key Lessons

- Recon is an iterative hypothesis-building process.
- Passive and active sources complement one another.
- DNS names, vhosts, and live applications are related but not identical inventories.
- Context turns isolated clues into attack paths.
- Automation gathers; the tester validates and interprets.
- Preserve commands, timestamps, sources, raw results, and confidence.

## Related Notes

- [Web Reconnaissance Knowledge](../../../knowledge-base/reconnaissance/web-reconnaissance.md)
- [Web Reconnaissance Methodology](../../../pentesting-methodology/reconnaissance/web-reconnaissance-methodology.md)
- [Web Reconnaissance Cheat Sheet](../../../cheatsheets/web-reconnaissance.md)
