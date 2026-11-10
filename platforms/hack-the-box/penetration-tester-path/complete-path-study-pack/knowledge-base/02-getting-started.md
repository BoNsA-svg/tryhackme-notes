# 2. Getting Started

> **Status:** Deep-complete (lesson text) — read in full 2026-11-09, 23 / 23 sections
> **Outline metadata:** Fundamental, Tier 0, 23 sections, 11 interactive exercises, 0 assessments (as displayed), est. 8 hours
> **Coverage checklist:** concepts ☑ · workflow ☑ · tools/commands ☑ · pitfalls ☑ · fixes/defences ☑ · engagement use ☑ · CTF use ☑
> **Named gaps (see `../GAPS.md`):** 14 exercise questions deliberately not attempted and no lab target spawned (§6 ×1, §7 ×3, §8 ×1, §9 ×1, §11 ×2, §15–18 ×1 each, §23 ×2); the §23 Knowledge Check box was not attacked; my worked notes are blank — I completed this module ~2 weeks before this note system existed.
> **Account note:** HTB shows this module as already completed by me ("Your first battle" badge, ~2 weeks ago).

Orientation to penetration testing and to working in HTB labs: how boxes, VPN access, and the lab environment fit together, and the basic workflow for attacking a first machine.

**Key concepts to own:** lab workflow; VPN / attack box basics; first-box methodology.

## Deep summary — section by section (original words)

### Setup and mindset

**1. Infosec Overview** — Infosec = protecting information across OS, network, web, physical and mobile/cloud domains. Hacking framed as experimentation and problem-solving; pentesting adds professional structure. Use a methodology (PTES, OSSTMM, OWASP, NIST SP 800-115, ISSAF) instead of ad-hoc attempts; security is an ongoing process. Pitfall: thinking that running attack tools alone makes you a tester. *Use: engagement, CTF as preparation.*

**2. Pentest Distro** — Purpose-built distros bundle tools so setup and drivers just work: Kali (dedicated pentest distro), Parrot OS (broader, privacy/crypto/coding), BlackArch (Arch-based, very large toolset via pacman), PwnBox (HTB's in-browser attack box). Pick one main distro, keep it updated (`apt update && apt upgrade -y`), configure tools. Pitfalls: distro-hopping, unmaintained installs. *Use: both.*

**3. Staying Organized** — A per-client folder structure from step one (project → client → assessment → evidence / credentials / screenshots / scans, one folder per host) plus Markdown notes, checklists, report templates and a findings database — so evidence can be found, checked and cited later. Tools named: Cherrytree, VS Code, Notion, GitBook, etc. Pitfall: syncing client-sensitive data to cloud services. *Use: engagement and CTF.* (This repo is exactly this idea, applied to HTB.)

**4. Connecting Using VPN** — HTB labs are reached over OpenVPN through a tunnel interface (tun0); one VPN region at a time, matched to the machine's region. Lab targets use private ranges, Starting Point uses public ones. Workflow: download the .ovpn pack → `sudo openvpn user.ovpn` → verify tun0 exists (`ifconfig`) and routes point through it (`netstat -rn`) → only then attack. Success signal: the OpenVPN initialization-complete message plus visible tun0 routes. Pitfalls: nested VPNs, wrong region, confusing public/private ranges. *Use: CTF/lab.*

### Pentesting basics

**5. Common Terms** — TCP (reliable/ordered) vs UDP (fast/connectionless); services are identified by port (well-known 1–1023: FTP 21, SSH 22, DNS 53, HTTP 80, HTTPS 443, SMB 445, RDP 3389); shell types (bind / reverse / web); common web servers (Apache, IIS, Nginx); the OWASP Top 10 (2021) categories as a checklist for web targets. Decide shell type by network egress — a reverse shell connects back out, bypassing inbound filtering. Pitfall: judging a service/risk from a port number or version string alone. *Use: both.*

**6. Basic Tools** — One tool per phase: Nmap (enumeration), Wappalyzer / WhatWeb (stack ID), Burp Suite (proxy), ffuf / Gobuster (fuzzing/content discovery), netcat, bash, Metasploit (shells/exploitation). Starter commands: `nmap -sV -sC -p- <ip>`, `ffuf -w <wordlist> -u http://<ip>/FUZZ`, `nc -lvnp 4444`, bash reverse shell via `/dev/tcp`. Workflow: scan → actually read the output → choose the next tool from what the target runs. *Use: both.* (Exercise not attempted — gap.)

**7. Service Scanning** — Nmap scan types trade stealth for completeness: connect `-sT` (reliable, logged), SYN `-sS` (stealthier), UDP `-sU` (slow), NULL/FIN/Xmas `-sN/-sF/-sX`, aggressive `-A`, OS `-O`, versions `-sV`, scripts `-sC`, full range `-p-`, timing `T0–T5`, saved output `-oN/-oX/-oG`. Workflow: host discovery → ports → services/versions → OS/scripts → save everything. Pitfalls: treating top-1000 as complete; fastest timing on fragile networks; unsaved output. *Use: both.* (3 questions not attempted — gap.)

**8. Web Enumeration** — Footprint the web stack (`whatweb`), check robots.txt and sitemap.xml, read source/headers for framework clues, brute-force directories/subdomains (Gobuster, ffuf), then look up the exact version found. Commands: `gobuster dir -u http://<ip> -w <wordlist>`, `ffuf -u http://<ip>/FUZZ -w <wordlist>`, curl for manual requests. Pitfalls: one wordlist only; ignoring status codes/sizes; stopping at the first 404. *Use: both.* (1 question — gap.)

**9. Public Exploits** — For a known version, search public exploit collections: `searchsploit <term>`, filter by type (`-t webapps`), copy one locally (`-m <id>`), keep the database updated (`-u`). Always **read exploit code before running it** (it can be backdoored or hardcoded to another target), adapt IP/port/paths, test in a lab first. Failure = wrong-version exploit or a crashed target. *Use: both.* (1 question — gap.)

**10. Types of Shells** — Bind (target listens — often blocked), reverse (target connects back — preferred through firewalls/NAT), web shell (code via an uploaded file). Workflow: pick type by egress → start the listener first (`nc -lvnp 4444` or socat) → trigger payload → upgrade to a full TTY (`python3 -c 'import pty; pty.spawn("/bin/bash")'`, then `stty raw -echo; fg`). Pitfalls: listener started after the payload; never stabilizing the TTY. *Use: both.*

**11. Privilege Escalation** — Enumerate first (`sudo -l`, `id`, `uname -a`, `ps aux`; LinEnum/LinPEAS, WinPEAS for automation; GTFOBins as reference). Linux paths: SUID, sudo misconfigurations, capabilities, cron, PATH issues, writable files, Docker group, kernel (last, carefully). Windows paths: services, scheduled tasks, unquoted service paths, tokens, UAC issues. Pitfall: blind kernel exploits breaking the box. *Use: both.* (2 questions — gap.)

**12. Transferring Files** — Serve from your machine and pull from the target is usually simplest: `python3 -m http.server 8000` → `wget` / `curl -O`; also scp, base64, netcat/socat on Linux; on Windows: `certutil -urlcache -f <url> <file>`, bitsadmin, PowerShell `Invoke-WebRequest`, SMB shares. Verify integrity with a checksum, keep more than one method ready, clean up uploads afterwards. *Use: both.*

### HTB platform

**13. Starting Out** — Machines: retired (write-ups available — attempt first, compare after) vs active (no write-ups); difficulties Very Easy→Insane; pick at your level and respect community norms (no spoilers). *Use: CTF/lab.*

**14. Navigating HTB** — Platform areas have different jobs: Starting Point (guided), Machines, Challenges, Academy (learning), Labs/Pro Labs/Endgames (bigger simulations); ranks, achievements, teams, leaderboards. Basic loop: choose → spawn (get IP) → VPN in → attack → submit user/root flags → keep notes. Pitfalls: wrong IP, no VPN, missed session limits. *Use: CTF/lab.*

### First box — the "Nibbles" walkthrough (§15–19), method level

**15. Enumeration** — The whole lesson is: enumerate fully. Full-port scan (`nmap -sV -sC -p- <ip>`) found SSH + HTTP; directory brute-forcing (`gobuster dir` with a standard wordlist) exposed a hidden directory naming a blog app; version ID then pointed to public exploits. Failure mode: a default top-port scan + default page and stopping there.

**16. Web Footprinting** — Confirm the app/version (`whatweb`, gobuster, curl on source), find the admin login, and let the site's own content suggest credentials (weak/default admin password) instead of blind brute force. Defensive takeaway to report: change defaults, patch the CMS.

**17. Initial Foothold** — With admin access, abuse the app's upload feature to place a web/reverse shell (watch extension filters), listener already running, then visit the uploaded file to trigger the callback and stabilize the shell (Python TTY upgrade). Metasploit is the alternate route (§19). Defences: validate upload types, store uploads outside the web root, least-privilege web user.

**18. Privilege Escalation** — Enumerate sudo (`sudo -l`): a root-run script invoked another program by relative path, and that file was replaceable — so root executed attacker-controlled code. Lesson class: privileged scripts must use absolute paths, sudo rights must be least-privilege, root-run binaries must not be writable.

**19. Alternate Method — Metasploit** — Same foothold via the framework's matching public-exploit module for that blog software: search, select, set RHOSTS/RPORT/TARGETURI/USERNAME/PASSWORD/LHOST/LPORT, run, get a session. Pitfalls: wrong TARGETURI, missing LHOST/LPORT, not checking version compatibility. Defences as §17.

### Problem solving and next steps

**20. Common Pitfalls** — Rushing/skipping enumeration, wrong payload or target IP, not reading error output, trusting exploits without version checks, rabbit-holing, no notes, asking for help too early/late, VPN misconfiguration, wrong listener address. Antidote: methodical enumeration, read what the target says, document, rule ideas out before switching.

**21. Getting Help** — Self-help order: module docs / man pages → targeted search → forums and Discord → write-ups and reference collections (HackTricks, PayloadAllTheThings, walkthroughs). Ask with context: what you tried, the output, the target in general terms. Never leak flags/answers or spoil machines.

**22. Next Steps** — Continue the Penetration Tester path in order (each later module deepens one phase), keep practising boxes and Pro Labs alongside, CPTS as the certification goal. Pitfall: jumping to very hard boxes or studying with no hands-on practice.

**23. Knowledge Check** — Unguided capstone applying the whole module: quick scan → full-port scan → detailed enumeration of every port → web footprinting → public-exploit search or manual exploitation → foothold two ways (manual / Metasploit) → TTY upgrade → manual + automated filesystem enumeration (LinEnum/LinPEAS) → **two** distinct privesc paths to root. Organize discovered data before deciding how to escalate. *Gap: read as lesson text only — its box was not spawned or attacked, and its 2 questions were not attempted.*

## My worked notes (yours, as you do the labs)

- *(Blank — module completed ~2 weeks before this note system. The Nibbles walkthrough and Knowledge Check are the two pieces worth re-doing hands-on and recording here in your own words.)*
