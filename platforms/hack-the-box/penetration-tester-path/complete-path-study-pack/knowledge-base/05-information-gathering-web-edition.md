# 5. Information Gathering — Web Edition

> **Status:** Deep-complete (lesson text) — read in full 2026-11-09, 19 / 19 sections
> **Outline metadata:** Easy, Tier 2, 19 sections, 9 interactive exercises, 1 assessment (as displayed), est. 8 hours
> **Coverage checklist:** concepts ☑ · workflow ☑ · tools/commands ☑ · pitfalls ☑ · fixes/defences ☑ · engagement use ☑ · CTF use ☑
> **Named gaps (see `../GAPS.md`):** 30 question entries (33 counting parts) deliberately not attempted across 9 sections (§3 ×2, §5 ×3, §7 ×1, §8 ×3, §9 ×5, §11 ×3, §15 ×1, §17 ×7, §19 ×5) and no target spawned; the skills assessment (§19) is scenario-text only; my worked notes are blank — I completed this module ~21 hours before this note system existed.
> **Account note:** HTB shows this module as already completed by me (badge: "Information is not knowledge, or is it?", earned ~21h before the read). Its completion page shows the Penetration Tester path at 25.5%.

Web reconnaissance, passive and active: DNS, crawling, archives, headers and technology fingerprinting to map a web target before attacking it.

**Key concepts to own:** passive vs active recon; subdomain enumeration; virtual hosts; crawling; tech fingerprinting.

## Deep summary — section by section (original words)

**1. Introduction** — Web recon is the opening phase of a web engagement: passive, active and semi-passive collection. A good map — host data, ownership, subdomains, tech, archived paths — focuses later testing and keeps early noise down. Treat the site as an entity, gather public data first, then narrow onto the application. *Use: engagement (habits transfer to CTF).*

**2. WHOIS** — The WHOIS protocol (RFC 3912, port 43, plain text) and the registry chain (IANA → regional registries → registrars). A query returns ownership/lifecycle facts: registrant, creation/expiry, name servers, registrar, status. Pitfalls: formats vary by registry; data goes stale or is privacy-masked (GDPR). Cross-check and take only what's needed. *Use: both.*

**3. Utilizing WHOIS** — Applied in three scenarios: IP-range ownership, domain ownership, and networks belonging to an organization. Registry data + the ASN tells you which network an IP belongs to; ARIN is a strong ASN starting point; the five RIRs are ARIN, RIPE, APNIC, LACNIC, AFRINIC. Cross-check registries — never conclude from one. *(2 Q — gap.)* *Use: both.*

**4. DNS** — Domain anatomy (subdomain / second-level / TLD), zones, records A, AAAA, NS, MX, CNAME, PTR, TXT, SOA, and DNSSEC. Resolution runs client → resolver → root → TLD → authoritative; the hosts file is a local shortcut. Mapping the records = mapping the infrastructure. Verify against more than one resolver; watch caching. Tools: dig, host, nslookup. *Use: both.*

**5. Digging DNS** — Hands-on `dig`: one record type at a time from a chosen resolver — `+short` for clean output, `-t A|MX|NS|CNAME|TXT|ANY`, `@resolver` to choose the server, `+trace` / `+norecurse` to control recursion, plus reverse lookups. Pitfalls: one resolver only, skipping reverse. Also dnsenum. *(3 Q — gap.)* *Use: both.*

**6. Subdomains** — Separate records hosting separate services (www, mail, api, admin, dev, internal…), each possibly a different version or a misconfiguration — undiscovered subdomains are hidden attack surface. Enumerate with active *and* passive methods, filter carefully (response-based filtering can hide real ones, and ordinary-looking pages get overlooked), validate what's live. *Use: both.*

**7. Subdomain Bruteforcing** — Guess names from a wordlist and resolve each: wordlist quality decides coverage. Four steps: choose the list (general / targeted / custom) → iterate and query → DNS lookup → filter + validate. Handle wildcard DNS or results flood. Tools: dnsenum (`-r` recursive), fierce, dnsrecon, amass, assetfinder, puredns. *(1 Q — gap.)* *Use: both.*

**8. DNS Zone Transfers** — AXFR is legitimate zone replication; if a server allows it to anyone, one query dumps the whole zone (subdomains, IPs, NS records). Flow: request → SOA → records → completion → acknowledgement. Workflow: list **all** name servers and try `dig axfr` against **each** — testing only one is the classic miss. Defence: `allow-transfer` restricted to authorized secondaries. *(3 Q — gap.)* *Use: engagement (appears in CTFs too).*

**9. Virtual Hosts** — One IP, many sites, switched by the Host header (name-, IP-, port-based). Fuzzing the header finds sites linked nowhere. Workflow: hosts-file entries for candidates → `gobuster vhost` (`--append-domain`, `-t` threads, `-k` ignore TLS errors, `-o` output) / Feroxbuster / ffuf → watch for responses differing from the default site. Pitfalls: missing hosts entries; mistaking the baseline response for a hit. *(5 Q — gap.)* *Use: both.*

**10. Certificate Transparency Logs** — Public append-only ledgers of issued TLS certificates. They exist to catch rogue certs and hold CAs accountable — and passively reveal subdomains, including from old/expired certs that brute force would never guess. Workflow: query crt.sh / Censys → parse JSON (`curl` + `jq`) → dedupe → validate liveness. Noise and stale entries are the pitfalls. *Use: both.*

**11. Fingerprinting** — Profile the stack: server software/version, CMS, frameworks, WAF presence — from headers, banners, probing and page content. Versions map to known issues. Workflow: fetch headers → detect server/CMS → confirm WAF → targeted checks. Tools: `curl -I`, Wappalyzer, BuiltWith, WhatWeb, Nmap, Netcraft, wafw00f, Nikto. Pitfalls: single-signal trust; WAF masking the real server; generic headers hiding the stack. Defence: minimize banner/header disclosure. *(3 Q — gap.)* *Use: both.*

**12. Crawling** — Systematic browsing from a seed URL (breadth-first = level by level; depth-first = one branch to the end) extracting links, comments, metadata and sensitive files — watch for backups (.bak/.old), web.config, settings.php, error/access logs. Tools: Burp Spider, OWASP ZAP, Scrapy, Apache Nutch. Scope discipline is the professional rule; JS-rendered links and deep branches are the easy misses. *Use: both.*

**13. robots.txt** — Root-level file (Robots Exclusion Standard) whose directives (Disallow, Allow, Crawl-delay, Sitemap) are guidance, **not access control** — so its Disallow list doubles as a map of what the owner considers sensitive. Fetch it, parse it, probe the listed paths. Watch for crawler traps/honeypots in it. Defence: never put sensitive paths in robots.txt; enforce access server-side. *Use: both.*

**14. .Well-Known URIs** — RFC 8615's `/.well-known/` holds site-wide metadata (paths registered with IANA): security.txt (RFC 9116), change-password, openid-configuration, assetlinks.json, mta-sts.txt (RFC 8461). The OpenID config is the rich one — it reveals auth endpoints, the JWKS URI, scopes, response types and algorithms. `curl` the known paths and parse what returns; don't assume every site implements them. *Use: both, mainly engagement.*

**15. Creepy Crawlies** — Automated crawlers in practice (Scrapy + the ReconSpider script, Burp Spider, ZAP, Nutch): walk a site and dump *structured* results — emails, links, external/JS files, form fields, media, comments — turning a raw crawl into an inventory of leads. Practice: stay in scope, throttle, respect the rules; aggressive crawling = load on the target. *(1 Q — gap.)* *Use: both.*

**16. Search Engine Discovery** — Google dorking: operators that surface indexed content never meant to be found — `site:`, `inurl:`, `filetype:`, `intitle:`, `intext:`, `cache:`, `link:`, `related:`, `info:`, `allintitle:`, `allinurl:` — combined to expose files, login pages and data; the Google Hacking Database holds proven query patterns. Limit: only what's indexed exists here. Defence: noindex where appropriate; never publish sensitive files. *Use: both.*

**17. Web Archives** — The Wayback Machine's snapshots reveal removed pages, old endpoints and changed content — and it's stealthy, because the live target is never touched. Workflow: look up the domain → browse captures by date → compare versions. Pitfall: assuming the archive is complete or current. Lesson for defenders: anything once public may persist — clean up old exposures. *(7 Q — gap.)* *Use: both.*

**18. Automating Recon** — Frameworks chain the checks into one run (efficiency, scale, consistency, coverage, workflow integration): FinalRecon (`--url --headers --sslinfo --whois --crawl --dns --sub --dir --wayback --ps --full`), Recon-ng, theHarvester, SpiderFoot, the OSINT Framework as a map. The rule that survives automation: **verify findings manually** — blind trust in tool output is the failure mode, and automated runs are noisy. *Use: both.*

**19. Skills Assessment** — Capstone chaining the whole module against a lab target: WHOIS → robots.txt → subdomain brute force → crawling → analysis, adding discovered subdomains to the hosts file as you go. It tests the chain, not a single tool; skipping steps is the failure mode. *(5 question entries — gap; read as scenario text only.)* *Use: engagement.*

## My worked notes (yours, as you do the labs)

- *(Blank — module completed ~21 hours before this note system. The interactive sections and the §19 capstone are the hands-on pieces worth re-doing and recording here.)*
