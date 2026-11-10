# Cheatsheet
Generic, standard penetration-testing reference — scan it in the moment. Always adapt to the exact target and stay inside scope / the lab rules.

## D1. Methodology at a glance


Recon → Enumerate → Assess → Exploit (prove it) → Post-ex (in scope) → Document. When stuck: enumerate more, assume less.

## D2. Enumeration order (any host)


☐  Ports and versions (full scan, then targeted service scans on what is open).

☐  For each service: what is it, what version, does it allow anonymous / default access, what does it leak?

☐  Web: fingerprint tech, crawl, fuzz content, check vhosts, note every parameter and form.

☐  File services: list shares and permissions before downloading anything.

☐  Save raw output per host — future-you needs it at reporting time.

## D3. Linux privilege escalation checklist


☐  whoami / id / groups; sudo -l; find SUID binaries; check capabilities.

☐  Cron jobs and writable scripts / paths they run; writable service configs.

☐  Interesting files: configs, history, keys, backups — in home dirs and web roots.

☐  Running processes and internal ports not visible from outside.

☐  Kernel and key software versions vs known issues — after misconfigurations, not before.

## D4. Windows privilege escalation checklist


☐  whoami /priv and group memberships; systeminfo and patch level.

☐  Services: permissions, unquoted paths, binaries you can replace (concepts — verify before acting).

☐  Scheduled tasks, startup items, always-install-elevated style misconfigurations.

☐  Stored credentials: config files, saved creds, unattended install files.

☐  Internal ports and locally-running services.

## D5. Web testing loop (per application)

1. Map it: crawl, fuzz, vhosts, parameters, roles / accounts available.
1. Test access control first (IDOR, verb tampering, forced browsing) — often higher impact than exotic bugs.
1. Test inputs: SQLi, XSS, command injection, file inclusion — one parameter at a time, note what reflects / errors / delays.
1. Test uploads and file handling separately.
1. Re-test key findings in a clean session before writing them up.

## D6. Active Directory quick list


☐  Enumerate: users, groups, computers, SPNs, GPOs, shares, sessions — graph it (BloodHound thinking).

☐  Look for: weak / reused passwords, Kerberoastable service accounts, AS-REP roastable users, delegation and ACL mistakes.

☐  Check what your current user can actually write to or control — paths beat single tricks.

☐  Document the path (A can → B can → domain) — that chain is the finding.

## D7. Pivoting reminder

- One foothold = a new network position, not the end. Re-run discovery from the foothold's view.
- Forwarding types: local (reach their service via your port), remote (give them a path back via the foothold), dynamic / SOCKS (route many tools through it).
- Label every tunnel: what it forwards, from where, and tear it down / record it when done.

## D8. Standard tool roles (what each is for)

- Nmap — discovery and service enumeration.
- Ffuf / Gobuster — web content, parameter and vhost discovery.
- Burp Suite / ZAP — intercepting, repeating and modifying web requests.
- SQLMap — automating an SQL injection you already understand manually.
- Hashcat / John — offline password cracking; Hydra / Medusa — online guessing, used carefully.
- Metasploit — framework for known exploits and session handling.
- LinPEAS / WinPEAS — privilege-escalation enumeration on a host you already access.
- BloodHound — graphing Active Directory relationships and attack paths.
- Chisel / SSH forwarding / Ligolo — tunneling and pivoting.
- Searchsploit / vendor advisories — matching an exact version to known issues.

End of V1. Next step: Bonz logs into HTB Academy himself; deep summaries are then added module by module in Part C, in the same four-part structure, and this file becomes V2, V3, … until all 28 modules are done — at which point Parts A, B and D are the finished engagement / CTF / cheatsheet set and Part C is the post-certificate knowledge base.

## D9. Nmap quick reference (added V4, from Module 3)

**Discovery:** `-sn` ping sweep · `-PE` ICMP echo · `-PS` TCP SYN ping · `-PA` TCP ACK ping · `-PU` UDP ping · ARP locally · `-Pn` skip discovery (marks everything up — use carefully) · `-sL` list only
**Scan types:** `-sS` SYN (fast, root) · `-sT` connect (logged) · `-sU` UDP (slow — don't skip) · `-sA` ACK (maps firewall rules) · `-sN/-sF/-sX` NULL/FIN/Xmas · `-sI <zombie>` idle
**Ports:** `-p-` all · `-p 22,80,443` list · `--top-ports <N>`
**Detection:** `-sV` service/version (tune: `--version-intensity 0-9`) · `-O` OS · `-A` aggressive bundle · `-sC` default scripts · `--script <cat|name>` NSE (categories: safe, default, discovery, vuln, intrusive — intrusive can DoS, lab first)
**Output — always save:** `-oA <name>` (all formats) · `-oN/-oX/-oG` individually · `--resume` an interrupted scan · `-iL <list>` targets from file
**Timing/performance:** start `-T4` (T0 paranoid → T5 insane) · `--max-retries 1-2` · `--host-timeout` · `--scan-delay` / `--min-rate` / `--max-rate` tuned to the network
**Evasion (authorized tests only):** map filtering first with `-sA`/`-sF` + `--reason` · `-f`/`--mtu` fragment · `-D RND:10` decoys · `-g 53` source port · `--data-length` pad · slow timing T0/T1
**Standard starter (lab/CTF):** `nmap -sV -sC -p- -T4 -oA scan <target>` — then a targeted follow-up per open port.

## D10. Per-service enumeration quick table (added V5, from Module 4 — Footprinting)

Pattern for every service: Nmap `-sV -sC --script <svc>-*` first → native client → version → known issues. The "dangerous setting" is usually the finding.

| Service | Port | First checks | Dangerous setting |
|---|---|---|---|
| FTP | 21 | anonymous login, list files, writable dirs | anonymous write |
| SMB | 139/445 | shares + permissions (smbclient/smbmap), users (rpcclient) | guest/null sessions, SMBv1 |
| NFS | 2049 | `showmount -e`, mount, inspect | `no_root_squash` |
| DNS | 53 | records, try AXFR, subdomain brute force | zone transfer allowed, open recursion |
| SMTP | 25 | VRFY/EXPN/RCPT user enum, open-relay test | open relay |
| IMAP/POP3 | 143/993, 110/995 | TLS connect, LOGIN → LIST → SELECT → FETCH | plaintext auth, creds shared with SMTP |
| SNMP | 161/udp | community strings (public/private + brute), `snmpwalk` | default strings, write access, v1/v2c |
| MySQL | 3306 | databases/users, FILE privilege (`load_file`) | weak root, anonymous users, broad FILE |
| MSSQL | 1433 | databases/roles, `xp_cmdshell` check (mssqlclient.py) | xp_cmdshell on, weak `sa` |
| Oracle TNS | 1521 | SID guess (ODAT), default accounts, sqlplus | default accounts, listener w/o password |
| IPMI/BMC | 623/udp | version, cipher-zero, RAKP hash dump → Hashcat 7300 | vendor defaults, BMC on general network |
| SSH/Rsync | 22/873 | ssh-audit, `rsync --list-only` | SSH-1, anonymous rsync, R-services trust (`+`) |
| RDP/WinRM/WMI | 3389/5985-6/135 | NLA check, evil-winrm, wmiexec with creds/hash | no NLA/MFA, exposed beyond mgmt hosts |

## D11. Web recon sequence (added V6, from Module 5)

Run in this order; each step feeds the next. Verify across sources — automation proposes, you confirm.

1. **WHOIS** — ownership, registrar, dates, name servers; ASN for IP ranges (ARIN first).
2. **DNS records** — `dig` A/AAAA/MX/NS/CNAME/TXT/SOA per resolver; reverse lookups too.
3. **Zone transfer** — try `dig axfr` against **every** name server, not just one.
4. **Subdomains, passive** — CT logs (crt.sh / Censys → `jq` dedupe), Wayback Machine, dorks.
5. **Subdomains, active** — brute force (dnsenum / amass / puredns) with a tuned wordlist; filter wildcards.
6. **Virtual hosts** — `gobuster vhost --append-domain` / ffuf on the Host header; add hits to the hosts file.
7. **Fingerprint** — `curl -I`, WhatWeb, Wappalyzer, wafw00f: server, CMS, framework, versions, WAF.
8. **Known files** — `/robots.txt` (a map, not a lock), `/.well-known/` (security.txt, openid-configuration).
9. **Crawl** — Burp/ZAP/Scrapy in scope; hunt backups (.bak/.old), configs, logs; correlate, don't dump raw URLs.
10. **Dorks + archives** — `site:` `filetype:` `inurl:` `intitle:` + GHDB patterns; Wayback for removed pages (target never touched).
11. **Automate, then verify** — FinalRecon / Recon-ng / SpiderFoot for coverage; manual confirmation before anything is reported.

## D12. Vulnerability assessment discipline (added V7, from Module 6)

- **Inventory first** — you can't assess what you haven't listed (hosts, cloud, SaaS, OT, apps).
- **Scan both ways** — unauthenticated *and* credentialed; confirm the credentialed login actually succeeded in the output.
- **Safe checks on**; agree exclusions (printers, legacy, HA systems) with the client first; **never DoS checks unless explicitly requested**.
- **Validate everything** — a scanner hit is a lead; static (version) checks lie more than dynamic ones. False positives get suppressed by rule, not ignored.
- **Score honestly** — CVSS Base + Temporal + Environmental; never quote the raw scanner number as the severity.
- **The scanner PDF is never the deliverable.** Report = Executive Summary → Methodology → Scope/Duration → Findings (name, CVE, CVSS, description, PoC, affected systems, remediation). Scan output rides along as an appendix.
- Nessus UI: port 8834 · OpenVAS/GSA UI: port 8080 · OpenVAS safe default config: "Full and fast".

## D13. File transfer quick picks (added V8, from Module 7)

Always: checksum after transfer (md5sum / Get-FileHash). HTTP(S) egress is the most likely to be open.

- **Windows download:** `Invoke-WebRequest` / `Net.WebClient` · SMB via `impacket-smbserver` + `net use` · FTP via `pyftpdlib` · `certutil` / `bitsadmin` as fallbacks. Watch the 8,191-char cmd limit — base64 + decode beats long one-liners.
- **Linux download:** `wget` / `curl -o` · pure-Bash `/dev/tcp` when nothing else exists · `scp` when SSH is open.
- **Uploads:** base64 paste for small files · `curl -T` / POST to your own Nginx PUT receiver or upload server · WinRM `Copy-Item -ToSession` · RDP shared folder.
- **No tools? Use an interpreter:** Python `urlretrieve`, PHP `file_get_contents`, Ruby `Net::HTTP`, Perl `LWP::Simple` — check what's installed first.
- **Sensitive data:** encrypt before it moves — `openssl enc -aes256 -pbkdf2` or AES script on Windows; unique password per client; dummy data for egress tests.
- **Quiet options:** LOLBins/GTFOBins (LOLBAS project) and a browser-matching User-Agent — defenders baseline command lines and UAs, so pick deliberately.

## D14. Shells quick reference (added V9, from Module 8)

- **Bind vs reverse:** bind = target listens, you connect (firewalls/NAT usually block it). Reverse = you listen (`nc -lvnp <port>`), target connects out — default choice; a common outbound port (443) helps, but DPI can still spot cleartext.
- **Read the payload before running it.** Bash one-liner = FIFO + shell + redirects + connection. PowerShell = TCPClient + stream + read loop + Invoke-Expression. If you can't explain each part, don't run it.
- **Stock commands get signatured** — customize cheat-sheet one-liners; edit stock web shells (Laudanum/Antak): your address in, comments/art out, rename, set its password.
- **TTY upgrade ladder:** `/bin/sh -i` → Python `pty.spawn` (then Ctrl-Z, `stty raw -echo; fg`, `export TERM=xterm`) → Perl/Ruby/Lua → awk/find/vim exec. Then immediately: `whoami`, `id`, `sudo -l`, `ls -la`.
- **MSFvenom:** staged = small loader + download (fragile on bad networks); stageless = all-in-one. Match OS + architecture exactly; encode for signature AV; listener set *before* delivery.
- **Web shell = foothold only.** Upgrade to a reverse shell fast, note its path + hashes for the report, delete it when done.
- **Defender tells to remember:** odd-port heartbeats (4444 = Meterpreter default), shell/PowerShell logging, upload monitoring — vary ports and keep activity explainable.

## D15. Metasploit workflow (added V10, from Module 9)

- **Core flow:** `search <term>` (filters: `cve:` `type:` `platform:`) → `info` (read it — a failed exploit does NOT prove the target is safe) → `use <module>` → `show options` / `show targets` → `set RHOSTS/LHOST/payload/...` (`setg` = global) → `run` / `exploit`.
- **Payloads:** Singles (all-in-one) · Stagers (small loader) · Stages (Meterpreter — in-memory, the default goal). Match architecture exactly; keep it small.
- **Sessions:** Ctrl+Z backgrounds · `sessions` lists · `sessions -i <id>` re-enters · `exploit -j` runs as a job · `jobs` / `kill` manage. Background, don't Ctrl+C.
- **Meterpreter first moves:** `sysinfo`, `getuid` → `ps` + `migrate <stable pid>` → `background` → `use post/multi/recon/local_exploit_suggester` → escalate → only then harvest (`hashdump` etc.).
- **Database = engagement memory:** `msfdb run` → one `workspace` per project → `db_nmap` / `db_import` → `hosts` / `services` / `creds` / `loot` → `db_export` to back up.
- **MSFvenom + handler:** `msfvenom -p <payload> LHOST=.. LPORT=.. -f <fmt>` (-e encoder, -b badchars, -x template) → `use exploit/multi/handler` with the same payload → deliver. Encoding is plumbing, not stealth — test against detection first.

## D16. Password attacks quick reference (added V11, from Module 10)

**Identify first:** `hashid` (add `-m` for hashcat mode, `-j` for John format). Wrong mode = wasted hours.
**Hashcat modes worth memorizing:** NTLM `-m 1000` · DCC2 (cached domain) `-m 2100` (~800× slower — think first) · sha512crypt `-m 1800` · BitLocker password hash `-m 22100` · MD5 `-m 0`.
**Attacks:** `-a 0` dictionary (+ `-r best64.rule`) · `-a 3` mask (`?l ?u ?d ?s ?a`, custom `-1`–`-4`) — estimate mask runtime *before* launching. John: `--wordlist` / `--rules` / `--incremental` / single-crack for name-based passwords; `unshadow` combines passwd+shadow.
**Protected file?** Convert, then crack: `ssh2john · office2john · pdf2john · zip2john · bitlocker2john · keepass2john`.
**Custom list:** CeWL the target's own site → base words → rules. Complexity policies make passwords predictable (Capital + word + year + symbol).
**Online (loud — lockout risk):** Hydra per-service; NetExec (`(Pwn3d!)` = admin-grade). Spray = 1 password × many users; stuffing = breached pairs (Hydra `-C`); defaults = check vendor docs per firmware.
**Extraction order (Windows):** SAM+SYSTEM+SECURITY hives → secretsdump · LSASS dump → pypykatz · Credential Manager (`cmdkey /list`, DPAPI) · DC: shadow-copy NTDS.dit + SYSTEM → secretsdump `-ntds`.
**Don't crack what you can reuse:** PtH (NT hash — mimikatz/Impacket/NetExec `-H`/evil-winrm `-H`; RID-500 caveat, LAPS blunts it) · PtT (mimikatz/Rubeus .kirbi; from Linux: ccache + KRB5CCNAME, keytabs) · PtC (PKINIT; ESC8 relay, Shadow Credentials).
**Hunt before you crack:** configs/scripts/history/shares/browsers (LaZagne, Snaffler, MANSPIDER), plaintext protocols in traffic (Pcredz). Search `passw|pwd|token|key` + local-language variants.

## D17. Common services attack map (added V12, from Module 11)

**Frame every attack:** Source → Process → Privileges → Destination. **Check misconfigurations before CVEs** (defaults, anonymous access, over-broad rights, debug/error leaks).
- **FTP (21):** `ftp-anon` NSE · anonymous login (write access = critical) · Medusa brute · bounce scan (`nmap -b`) mostly historical.
- **SMB (445/139):** null session first (`smbclient -N -L`, rpcclient) · enum4linux-ng / smbmap · CrackMapExec/NetExec spray (lockout risk) · Impacket exec trio · Responder → NetNTLMv2 (`-m 5600`) crack or **relay** (ntlmrelayx). Defence: no SMBv1, signing on, no null sessions.
- **SQL (1433/3306):** privileges decide everything. MSSQL: `xp_cmdshell` (off by default) · `xp_dirtree` to your SMB listener = service-account hash theft · impersonation (`EXECUTE AS`) · linked servers. MySQL: `INTO OUTFILE` (secure_file_priv), `LOAD_FILE`, UDFs.
- **RDP (3389):** spray > brute (Crowbar/Hydra) · `tscon` session hijack (pre-2019) · PtH needs Restricted Admin Mode (off by default).
- **DNS (53):** `dig AXFR` · subdomain enum (Subfinder passive, Sublist3r brute) · **dangling CNAME = takeover** (verify via provider error) · spoofing needs MITM.
- **Mail (25/587, 143/993, 110/995):** MX first · VRFY/EXPN/RCPT user enum (smtp-user-enum) · cloud: O365spray/MailSniper · open relay test (smtp-open-relay, swaks).
- **CVE names to recognize:** SMBGhost CVE-2020-0796 (SMBv3 compression RCE) · BlueKeep CVE-2019-0708 (wormable RDP) · OpenSMTPD CVE-2020-7247 (sender-field RCE) · CoreFTP CVE-2022-22836 (PUT traversal write).

## D18. Pivoting decision guide (added V13, from Module 12)

**First, always:** enumerate the foothold's NICs, IPs, routes, gateways (`ipconfig`/`ifconfig`, `ip route`). Second NIC or tunnel interface = another network. Draw the map.
**Pick by what you have:**
- SSH creds on the pivot → `ssh -D 9050` (dynamic SOCKS) + proxychains · one service only → `ssh -L` · target can't reach you → `ssh -R` (payload calls the *pivot's internal IP*, not yours).
- Meterpreter session → `autoroute -s <net>` + `auxiliary/server/socks_proxy`, or `portfwd` for single ports.
- Windows pivot, nothing to upload → `netsh interface portproxy` (built-in) · `plink -D` + Proxifier for RDP apps.
- No SSH, one binary allowed → **Chisel** (reverse mode: server `--reverse` on you, client `R:socks` from pivot; mind glibc/build).
- Only a web server → Rpivot (server on you, client calls back). Only DNS out → dnscat2 (slow, keep it short). Only ICMP out → ptunnel-ng (root both ends, slow).
- RDP session only → SocksOverRDP (register the DLL first) = SOCKS inside the RDP channel.
- Simple relay for one shell → socat `TCP4-LISTEN:port,fork TCP4:dest:port` on the pivot.
**Proxychains rules:** TCP-connect scans only (`nmap -Pn -sT`); ping sweeps lie on Windows (ICMP blocked) — verify "empty network" another way.
**Hygiene:** forward only what you need, note every route/credential as found, tear tunnels down when done. Double pivot = chain the same pattern hop by hop.

## D19. AD attack path (added V14, from Module 13)

**Chain:** passive recon → domain enum → foothold (**Responder/Inveigh** LLMNR poisoning → crack NTLMv2) → **policy first, then spray** (Kerbrute userenum doesn't log failures; DomainPasswordSpray self-limits; log every attempt; `--local-auth` for local-admin reuse — LAPS kills it) → check the foothold's controls (Defender, AppLocker gaps, LAPS readers) → **credentialed enum** (NetExec/PowerView + **BloodHound** — query it, don't just collect; Snaffler for creds in shares) → **Kerberoast** (GetUserSPNs/Rubeus → hashcat **13100** RC4 / 19700 AES; watch for 4769 bursts) → **AS-REP roast** pre-auth-disabled accounts → **ACL chains** (query by *your* SID, resolve GUIDs; targeted Kerberoast via a temporary SPN — clean up SPN first, membership second) → **DCSync** (needs replication rights; krbtgt = Golden Ticket) → lateral via CanRDP/CanPSRemote/SQLAdmin edges (mind the **double-hop**: WinRM doesn't cache creds — PSCredential or RunAs session config) → **trusts** (child→parent: ExtraSids Golden Ticket, no SID filtering in-forest; cross-forest: Kerberoast `-target-domain`, foreign group members; collect BloodHound on *both* sides).
**Fast lanes (use with care):** NoPac · PrintNightmare · PetitPotam → cert → TGT. **Also check:** GPP/SYSVOL passwords, description-field passwords, PASSWD_NOTREQD, GPO abuse, Exchange privileged groups.
**Hardening anchors:** LAPS + gMSA · MachineAccountQuota 0 · disable LLMNR/NBT-NS · SMB/LDAP signing · no RC4 · Protected Users · audit replication rights · PingCastle/Group3r/ADRecon regularly.

## D20. Web proxy workflow (added V15, from Module 14)

**Setup once:** listener 127.0.0.1:8080 · browser through it (pre-configured browser, or Firefox + FoxyProxy) · **CA cert imported** or HTTPS breaks with warnings.
**Loop:** browse the app normally → work from **Proxy History** → **Repeater** for systematic edits (keep the original; respect URL-encoding in POST bodies) → Intercept only when you must catch a live action — and confirm it's the right request before editing.
**Responses:** intercept to enable/unhide fields and read comments — exploration only; a client-side change is *not* a finding until confirmed server-side. Make it persistent with **match-and-replace rules** (test the rule fires; beware regex anchors and over-broad matches).
**Encoding:** decode to read, **re-encode with the same method** to send (Burp Decoder/Ctrl+U · ZAP Ctrl+E). Know what your tool auto-encodes.
**Fuzzing:** Burp Intruder (§ positions, Grep-Match, sort by status/length) — **Community throttled ~1 req/s**: short jobs only. ZAP Fuzzer unthrottled; read body size + RTT, not just status. CLI fuzzers for big lists (next module).
**Other tools through the proxy:** tool's own proxy setting, Metasploit PROXIES, or proxychains `http 127.0.0.1 8080` — to *see what the tool sends*; slow, so investigate-then-unproxy.
**Scanners:** Spider/crawl → passive → active (Burp Scanner is Pro-only; ZAP's is free). Triage severity × confidence, verify highs manually, never hand a raw scanner report to a client.

## D21. Ffuf playbook (added V16, from Module 15)

**Pattern:** `ffuf -w <list>:FUZZ -u <url-with-FUZZ>` · `-ic` skips wordlist comments · `-t` threads (be gentle on remote targets — you can DoS) · verify every hit manually.
- **Directories:** FUZZ in the path position (SecLists directory-list-2.3; size = time decision). Empty page ≠ missing directory.
- **Pages:** fingerprint the extension first (`.FUZZ` on index.*; the list already has the dot) → then filenames with that extension (`FUZZ_1.FUZZ_2`). Headers lie; fuzzing doesn't.
- **Recursive:** `-recursion -recursion-depth 1 -e .php -v` — cap depth, then hand-pick branches.
- **Subdomains vs vhosts:** subdomain fuzzing = public DNS only (`FUZZ.domain`). **Vhost fuzzing** = `-H 'Host: FUZZ.domain'` against the IP; everything returns 200, so **judge by response size**, not status.
- **Filtering:** measure the default size first, then `-fs <size>` (matchers `-mc/-ml/-mr/-ms/-mw`, filters `-fc/-fl/-fr/-fs`). Size filtering can hide a same-size real hit — spot-check.
- **Parameters:** GET `?FUZZ=value` (burp-parameter-names list) · POST `-X POST -d 'FUZZ=key'` + form content-type. "Invalid value" = the parameter exists.
- **Values:** custom lists (e.g. `seq` ranges) in the value position; start plausible, widen, record the list.
- **Order of work:** vhosts → extensions → pages → parameters → values. Each list feeds the next.
- **Name won't resolve?** `/etc/hosts` entry; log it and clean it up after the engagement.

## D22. Login brute-forcing quick reference (added V17, from Module 16)

**Order of attack:** defaults → tailored dictionary → hybrid (mutated dictionary) → pure brute force only if the math says the space is searchable. Compute charset^length and estimate time *before* starting. Online = lockout/rate-limit risk always (see D16).
**Hydra:** `-l/-L` user(s) · `-p/-P` pass(es) · `-t` tasks · `-f` stop on first hit · `-s` port · `-M` target list · `-x min:max:charset` generate.
- Basic Auth: `hydra -l <user> -P <list> <ip> http-get -s <port>`
- Forms: `http-post-form "path:params:condition"` — `^USER^`/`^PASS^`, `F=` failure text or `S=` success marker (302/dashboard). **Inspect the form first** (path, field names, hidden fields, CSRF) — guessing fields is the #1 failure.
**Medusa:** `-M <module>` (ssh, ftp, http, mysql, rdp, vnc…) · `-e n` empty password · `-e s` password = username. Hydra for web forms, Medusa for parallel multi-service.
**After one service falls:** enumerate the host (`netstat -tulpn`, nmap localhost) — a second service usually exists; username often leaks from context (home dirs). Don't assume password reuse.
**Custom lists:** Username Anarchy (username variants) + CUPP `-i` (person profile) → filter to the target policy with `grep -E` → Hydra. Client data only, with permission.
**Defence line:** 12+ char unique passphrases, defaults changed, MFA, key-based SSH, no cleartext FTP, TLS for Basic Auth.

## D23. SQL injection workflow (added V18, from Module 17)

**Detect:** `'` → error or behavior change = injectable. Silence ≠ safe (blind: boolean differences, `SLEEP(5)` timing).
**Bypass login:** always-true condition in the WHERE; mind precedence (**AND outranks OR**); comment out the tail — `-- ` (or `--+` in URL), `#` → `%23` in URLs, `/* */`. Close any wrapping parenthesis first.
**Union workflow:** ① error confirms → ② count columns: `ORDER BY n` until error (count = n−1), cross-check `UNION SELECT 1,2,3…` (ERROR 1222 = wrong count; pad with NULL) → ③ map **visible** columns with distinct values → ④ payload in a visible slot (`@@version`, `user()`, `database()`).
**Enumerate:** `information_schema` — SCHEMATA → TABLES → COLUMNS (filter it; skip default MySQL schemas) → UNION-select the target columns.
**Files (needs FILE privilege):** read `LOAD_FILE(path)` via a visible column (NULL = privilege/OS-permission missing). Write `SELECT … INTO OUTFILE` — gated by `secure_file_priv` (NULL = disabled); find webroot via configs/errors; probe file first; **delete any shell afterwards**.
**MySQL CLI hygiene:** `-p` prompts (never inline passwords — shell history), least-privilege user, `;` ends statements.
**Fix to recommend:** parameterized queries first; then strict validation, least-privilege DB accounts, no error/version leakage; WAF is a layer, not the fix.

## D24. SQLMap quick reference (added V19, from Module 18)

**Techniques (BEUSTQ):** B boolean blind (slow) · E error-based (fast) · U union (fastest) · S stacked · T time-based (last resort) · Q inline. Restrict with `--technique=BEU`.
**Setup is everything:** exact request or nothing — DevTools copy-as-cURL → swap to sqlmap, or `-r request.txt` from Burp. `-u` GET · `--data` POST · `-p` one parameter (or mark `*`) · `--cookie` / `-H` headers · `--random-agent` (default UA announces the tool) · `--batch` for repeatable runs. Output: `~/.sqlmap/output/<host>/`.
**Read the log:** heuristic "might be injectable" ≠ proof; "parameter is vulnerable" = confirmed. Don't skip other DBMSes too early.
**Debug:** `--parse-errors` · `-t traffic.txt` · `-v 6` · `--proxy http://127.0.0.1:8080` (Burp).
**Tune:** `--level 1–5` / `--risk 1–3` — raise one step at a time; OR payloads (risk-gated) can hit UPDATE/DELETE — approval before high-risk on production. `--prefix/--suffix` for odd quoting · `--string/--text-only/--code` matching · `--union-cols` exact count.
**Enumerate:** `--banner --current-user --current-db --is-dba --passwords` → `--tables -D` → `--dump -T` (`-C` columns, `--where`, `--start/--stop`) · `--schema` map · `--search` by name pattern · auto hash-cracking on dumps.
**Protections:** `--csrf-token` · `--randomize` · `--eval` computed params · `--tamper` (space2comment, randomcase, between…) — order matters · `--chunked` · `--proxy-file`/`--tor`.
**OS:** `--is-dba` first (non-DBA reads fail silently) · `--file-read` · `--file-write` + `--file-dest` (secure_file_priv gates) · `--os-shell` (force `--technique=E` if silent). Remove everything you plant.

## D25. XSS workflow (added V20, from Module 19)

**Classify first:** Stored (saved in DB, fires for every visitor — most critical) · Reflected (echoed, not stored — victim must load your crafted URL; GET only for links) · DOM (never reaches the server — source → sink in page JS; check the *rendered DOM*, not Ctrl+U source).
**Prove it:** payload in source ≠ vulnerability. **Only execution counts.** No pop-up ≠ safe — check the source, try another proof payload.
**DOM mechanics:** `<script>` doesn't run inside innerHTML — use event-handler payloads (img onerror style). Dangerous sinks: innerHTML, outerHTML, document.write, jQuery html/append/after family.
**Discovery:** scanners (Burp Pro, ZAP, XSStrike) to scale → manual payload lists (PayloadAllTheThings) → code review (source→sink) is most reliable. Verify everything manually.
**Impact demos:** deface (stored; clean the DB to remediate) · fake login form (strip original elements; log with a PHP server, not netcat) · session hijack (steal cookie → replay via DevTools Storage).
**Blind XSS:** fires where you can't see (admin views, tickets, User-Agent). Beacon with a remote script tag, per-field URL — the callback names the field. Exfil via Image().src, not a visible redirect.
**Fix to recommend:** back-end validation + output encoding (htmlspecialchars) + DOMPurify front-end; avoid dangerous sinks; CSP script-src 'self', nosniff, **HttpOnly + Secure** cookies (HttpOnly is what stops cookie theft).

## D26. File inclusion playbook (added V21, from Module 20)

**Find it:** a parameter choosing a file (`?page=` `?language=`) → try an absolute path, then `../` — find the *minimal* traversal depth; errors leak the exact include string. Remember second-order: payloads stored in the DB (usernames, avatar paths) fire later.
**Bypass the filter in front of you:** non-recursive strip → `....//` / `..././` · blocked chars → `%2e%2e%2f` / double-encode · prefix check → approved prefix + traverse back up · appended extension → php://filter (below). Null byte & truncation are historical (pre-5.3 PHP) — recognize, don't rely.
**Read source:** `php://filter/read=convert.base64-encode/resource=<file>` → `base64 -d` locally (whole string!). Fuzz for PHP pages first (check 403s), follow file references to the config.
**Escalate to RCE — pick by what the target allows:**
- Wrappers (need `allow_url_include` — verify in php.ini via LFI): `data://text/plain;base64,…` · `php://input` (POST body) · `expect://`
- RFI: host the shell — HTTP (80/443), FTP, SMB/UNC on Windows; test with a localhost URL first.
- Upload + LFI: GIF-magic-bytes polyglot, find stored path, include it. zip:// / phar:// when enabled.
- Log poisoning: shell into User-Agent → include access.log (Nginx often readable, Apache usually not); session file `sess_<PHPSESSID>` (re-poison per command); also /proc/self/environ, SSH/FTP/mail logs.
**Automate:** ffuf the parameter, LFI-Jhaddix wordlist, fuzz webroot + log/config paths; read apache2.conf/envvars for exact paths.
**Fix to recommend:** IDs→files server-side map (input never reaches include) · basename() · allow_url_include/fopen Off · open_basedir/webroot lock · WAF tuned report-only first.

## D27. File upload bypass ladder (added V22, from Module 21)

**Probe order:** fingerprint the stack → minimal echo test → **confirm execution** (accepted ≠ executed).
1. **No validation:** upload the shell in the server's language; request it under the uploads path.
2. **Client-side only:** rewrite in Burp after approval, or strip the JS handler/`accept` in the Inspector.
3. **Blacklist:** fuzz extensions (Intruder + SecLists/PAT lists), sort by response length — try alternates (.phtml) and mixed case.
4. **Whitelist:** unanchored regex → `shell.jpg.php` · server FilesMatch quirk → `shell.php.jpg` · character injection (`%20 %0a %00`, ADS on Windows) → fuzz generated permutations.
5. **Type checks:** Content-Type is yours — fuzz it (change the *file's* header in multipart) · magic bytes → prepend `GIF89a`.
**Shells:** web shell for a quick proof; reverse shell (msfvenom php/reverse_php, listener first, IP/port set) for real work.
**"Safe" types still bite:** SVG/HTML → stored XSS + XXE (also inside Office/PDF XML) · metadata payloads (exiftool) · zip/pixel bombs · traversal in the filename. The filename itself: command/SQL/XSS injection surface.
**Fix to recommend:** end-anchored whitelist + blacklist net, parse/re-encode content, hidden uploads dir served via a controlled download script, random names, separate host/container, disable_functions, size caps.

## D28. Command injection: operators + filter evasion (added V23, from Module 22)

**Operators:** `;` both in order · newline · `&` background · `|` pipe · `&&` second only if first succeeds · `||` second only if first **fails** (break the first command to get output) · backticks/`$()` sub-shell substitution. `;` fails in Windows CMD, works in PowerShell. Read silence via exit codes.
**Front-end check?** No request in the Network tab = it never reaches the server → Burp Repeater, URL-encode (Ctrl+U), resend.
**Map the filter first:** resend the payload piece by piece (operator / space / word) until the denial names the blocked part. Every bypass is chosen from that map.
**Space blocked:** tab `%09` · `${IFS}` · brace expansion `{ls,-la}` — with the newline operator.
**Character blocked (e.g. `/`):** slice it from an env var by position (`$PATH`, `$HOME` — offsets from *this* box's printenv) · or `tr` ASCII shifting.
**Command word blocked:** insert shell-ignored characters — quotes (pairs, even count), `\` / `$@` (Linux), `^` (Windows).
**Still blocked:** mixed case · reversed string + `rev` · base64/hex + decode in a sub-shell (here-string avoids a blocked pipe) · Bashfuscator / Invoke-DOSfuscation. Verify every obfuscated payload locally first.
**Fix to recommend:** don't shell out (library function instead) → validate format → allowlist-sanitize (escaping alone insufficient) → WAF, least-privilege web user, disable_functions, open_basedir.

## D29. Verb tampering · IDOR · XXE (added V24, from Module 23)

**Verb tampering:** OPTIONS first — list accepted verbs. Auth/filter scoped to GET/POST? Try HEAD (executes like GET, no body), PUT, DELETE. Config flavour (verb-listed auth rules → use LimitExcept-style fixes) vs coding flavour (filter checks one method, sink reads both → change the method in Burp).
**IDOR:** the reference isn't the bug — the missing server-side ownership check is. Test with **two accounts**: replay A's calls as B. References hide in params, headers, cookies, front-end JS (incl. "disabled" admin calls). Encoded? decode/change/re-encode. Hashed? read the front-end for the scheme and recompute it. Predictable ids → mass-enumerate (Intruder or fetch-regex-loop script). Write path blocked? Pivot to the read path — leaked identifiers unlock the write path. Client-side role cookies are not authorization.
**XXE:** find XML input → internal entity test → SYSTEM entity file read → source via `php://filter` base64 wrapper. Content breaks XML? Parameter-entity CDATA assembly with a hosted DTD. Nothing reflected? Error-based (undefined entity leaks content in the error). Fully blind? OOB — parser fetches your listener URL with base64 content as parameter/subdomain; XXEinjector automates.
**Fixes to recommend:** auth + filters cover **all** verbs · server-side RBAC keyed to the session (UUIDs supplement, never replace, access checks; never compute references client-side) · current XML libraries, DTDs/external entities off, errors hidden, prefer JSON.

## D30. Common applications attack map (added V25, from Module 24)

**Method (every product):** discover → fingerprint version → known issues for that exact version → abuse built-in features → clean up + document. Triage at scale: Nmap web ports → EyeWitness/Aquatone screenshots over the XML → validate scanner output by hand.
**WordPress:** plugins > core > themes for vulns · child theme can mask the real theme · users from login error differences · brute force via XML-RPC (WPScan) · admin → Theme Editor shell. **Joomla:** droopescan · admin → template error-page code. **Drupal:** PHP Filter module (pre-8) or backdoored module upload · Drupalgeddon 1/2/3.
**Tomcat:** version from Server header/error pages · manager brute force → WAR/JSP shell → undeploy · Ghostcat CVE-2020-1938 (AJP 8009 file read) · CGI CVE-2019-0232 on Windows (PATH unset — full paths, URL-encoded).
**Jenkins:** 8080 · Script Console Groovy = RCE at the service account (often SYSTEM on Windows). **Splunk:** 8000/8089 · weak/no auth → custom app upload → service-privilege shell. **PRTG:** CVE-2018-9276 blind command injection in notification parameters. **GitLab:** /help for version (post-login) · open registration + sign-up-form user enumeration · spray carefully (lockout) · CVE-2021-22205 ExifTool RCE. **ColdFusion:** /CFIDE/administrator, ports 8500/5500 · traversal CVE-2010-2861, upload RCE CVE-2009-2265.
**Cross-cutting:** thick clients and service-connecting apps hide credentials in binaries (Process Monitor for temp drops, GDB/PEDA for ELF strings, dnSpy for .NET connection strings) — recovered passwords are lateral-movement keys. IIS tilde: 8.3 short names leak hidden files. LDAP injection: wildcards in both fields. Mass assignment: add the unshown privileged parameter (Burp intercept). Shellshock still lives on EOL CGI.
**Hardening to recommend:** living inventory · defaults changed/disabled, custom admin names, admin MFA · no external admin logins without need · disable editors/deploy features · patch fast · backups + monitoring · beware trial→free downgrades that drop authentication.

## D31. Linux privilege escalation checklist (added V26, from Module 25)

**Enumerate first (manual before LinPEAS):** whoami · id · sudo -l (read it all) · uname -a + os-release (note EOL) · $PATH/env · users/groups/homes · .ssh + known_hosts · bash histories · configs & web roots (wp-config pattern) · services **and who runs them** · packages/versions vs known issues · cron (crontab, /etc/cron.d) · unmounted filesystems (lsblk/fstab) · GTFObins-check every interesting binary.
**Permissions:** SUID/SGID finds (`-perm -4000` / `-6000`) → odd ones + GTFObins · sudo rights → binary options that exec (tcpdump postrotate pattern; AppArmor may block) · groups: lxd/docker ≈ root, disk → debugfs, adm → logs · capabilities: getcap -r / → cap_dac_override-style file writes.
**Environment:** writable dir early in PATH or `.` in PATH → plant a same-named script · wildcard in a privileged command (tar checkpoint) → filenames become options · restricted shell → substitution/backticks/chaining/functions.
**Services:** world-writable script run by root cron (prove it with pspy; back up + restore) · vulnerable root service version (Screen 4.5.0) · logrotten trio (writable log + root logrotate + vulnerable version) · NFS no_root_squash → planted setuid · tcpdump → cleartext creds · root tmux on a shared socket.
**Containers:** LXD privileged container + host / mounted · Docker host-root mount or writable docker.sock · Kubelet 10250 anonymous → pod tokens → host-mount pod.
**Internals:** sudo env_keep LD_PRELOAD → malicious .so · setuid binary with writable RUNPATH → replacement .so (exact exported symbol) · Python: writable installed module / sys.path shadowing / SETENV PYTHONPATH.
**Version exploits (match exactly, crash risk):** Baron Samedit CVE-2021-3156 · sudo -u#-1 CVE-2019-14287 · PwnKit CVE-2021-4034 · Dirty Pipe CVE-2022-0847 (5.8–5.17; clean up any setuid shell) · Dirty COW · Netfilter CVE-2021-22555 / -2022-25636 / -2023-32233.
**Hardening to recommend:** auto-patching · absolute paths in cron/sudo · no creds in world-readable files · privileged-group discipline · Lynis/baseline audits alongside (never instead of) testing.

## D32. Windows privilege escalation checklist (added V27, from Module 26)

**Orient:** ipconfig /all · arp -a · route print · Defender (Get-MpComputerStatus) + AppLocker policy **before** running tools. **Enumerate:** systeminfo · tasklist /svc · Get-HotFix/wmic qfe · netstat -ano · quser · whoami /priv + /groups · net localgroup administrators. Tools: winPEAS, Seatbelt, SharpUp/PowerUp, WES-NG — source-compiled on client boxes; verify every finding by hand. Re-run after **every** privilege gain.
**Token privileges:** SeImpersonate (service accounts) → Potato family — JuicyPotato dead on 2019/1809+, use PrintSpoofer/RoguePotato · SeDebug → ProcDump LSASS → Mimikatz · SeTakeOwnership → takeown + icacls, then **revert**.
**Groups:** Backup Operators → robocopy /B reads through deny ACEs; shadow copy → NTDS offline · Event Log Readers → 4688 command lines leak passwords · DnsAdmins → ServerLevelPluginDll = SYSTEM on the DC (permission + cleanup) · Hyper-V Admins ≈ Domain Admins if DCs are VMs · Print Operators → SeLoadDriver → vulnerable signed driver · Server Operators → sc config binPath on a LocalSystem service (1053 can still mean it ran).
**OS attacks:** UAC bypass via auto-elevator DLL search order (not a security boundary; "Always notify" resists) · weak service/registry/autorun ACLs (SharpUp, AccessChk; back up + restore binaries) · patch gaps → HiveNightmare CVE-2021-36934, PrintNightmare, build-matched kernel exploits only · SYSTEM third-party services with local flaws · loopback services + named-pipe DACLs.
**Credentials:** findstr "password" · unattend.xml · PowerShell history · web.config · DPAPI stores (owner + machine bound) · cmdkey/runas /savecred · browsers/KeePass (13400)/LaZagne/SessionGopher · AutoLogon registry · PuTTY/Wi-Fi keys · shares/Sticky Notes/restic backups/mRemoteNG on pillage.
**Restricted/misc:** Citrix breakout via dialogs, shortcut targets, script types · .lnk UNC plants → Responder NetNTLMv2 (-m 5600; SCF dead on 2019) · AlwaysInstallElevated (both keys) → MSI as SYSTEM · writable scheduled-task scripts · LOLBins (certutil).
**Hardening to recommend:** gold images + WSUS/GPO vs CIS/STIG · Credential Guard · least-privilege groups (audit stale membership) · strip unneeded token privileges · EOL systems segmented with a retirement plan.

## D33. Reporting & notes template (added V28, from Module 27)

**Notes structure (mirror this tree):** Admin · Deliverables · Evidence · Findings · Scans · Notes · OSINT · Wireless · Logging · Misc · Retest. Note sections: attack path, credentials, findings, research, AD/OSINT, admin/scoping, activity log, payload log. Tmux logging ON from the first command; one note per finding; log every artifact left behind and every account/system change (written approval first).
**Evidence hygiene:** solid-bar redaction, never blur/pixelate (reversible) · terminal output > screenshots · paste as plain text · <SNIP> gaps · trim hash middles · no unneeded client PII.
**Finding fields:** description/platforms · impact · affected systems · recommendation (exact settings/paths, test-first warning, interim workaround) · references (vendor-agnostic, accessible) · steps to reproduce — one figure per step, full config, narrative between figures · optional CVE / OWASP / MITRE / CVSS.
**Report anatomy:** exec summary (non-technical, ~1.5–2 pp, no vendor pitches) · attack-chain narrative · findings by impact · short/medium/long-term recommendations · static appendices (scope, methodology, severity, bios) + dynamic (attempts/payloads, compromised creds, config changes, extra scope, info gathering, DPAT).
**Process:** report written DURING the test from a fresh blank template (never a previous client's file) · immediate notification for can't-wait findings (exposed unauth RCE, default creds) · QA by a non-author (technical + styling passes) · Draft → review meeting → FINAL → archive per retention (≥ until retest) · a vuln assessment is not a pentest — never write it as one.

## D34. Capstone engagement playbook (added V29, from Module 28) — the whole path in order

**Pre-engagement:** signed RoE in hand · scope + out-of-scope pinned (no phishing/physical/DoS, no changes without written consent) · two contacts per side · heavy scans out-of-hours · start-of-testing email · VM + notes tree + report template ready before scanning.
**External:** top-1000 → full-port version/script scan → dig axfr (+ passive fallback) → ffuf vhosts with an invalid-host baseline · probe every service, log dead ends as findings (VRFY on, rpcbind exposed = Low) · web apps via EyeWitness, one flaw class each is enough: IDOR · verb tampering + upload → RCE · weak WP admin → theme editor · SQLi → sqlmap · blind XSS → cookie · SSRF → local files · open GitLab · XXE · filtered command injection (newline/quoting bypasses) on an internal-IP host = pivot.
**Foothold:** Socat reverse shell → full-tty upgrade → stabilize · local enum (groups! adm → /var/log auth trails) → persistence save point: backup route + sudo -l GTFOBins key read → root SSH.
**Internal:** pivot (ssh -D/ProxyChains or autoroute) · ping sweep · dead ends logged (null session, Tomcat) · NFS won't ride a proxy — mount from the pivot host · app admin (xp_cmdshell / shell upload) → PrintSpoofer → SYSTEM → reg save hives → secretsdump → PtH-validate → first domain cred.
**Lateral:** BloodHound paths first · re-enumerate shares as EVERY new identity (spider_plus) · SYSVOL + scripts + description fields · Kerberoast all, spray within lockout policy · WinRM/RDP via ssh -L · unattend.xml · watched-folder SYSTEM tasks · pillage (LSA, browsers, password DBs).
**Domain:** GenericWrite → fake SPN → targeted Kerberoast → crack → GenericAll over a DCSync-holding group → secretsdump -just-dc-ntlm · remove the fake SPN · DPAT + PingCastle · test segmentation with a double pivot · never open/remove PII.
**Closeout:** end-of-testing email (no surprise findings) · findings risk-ordered · attack-path-of-least-resistance written out · TOTAL cleanup itemized in appendices · logs retained · archive + lessons learned + retest agreed.
