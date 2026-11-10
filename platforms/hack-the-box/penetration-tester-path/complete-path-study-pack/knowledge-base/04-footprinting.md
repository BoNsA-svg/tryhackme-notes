# 4. Footprinting

> **Status:** Deep-complete (lesson text) — read in full 2026-11-09, 21 / 21 sections
> **Outline metadata:** Medium, Tier 2, 21 sections, 14 interactive exercises, 0 assessments (as displayed), est. 2 days
> **Coverage checklist:** concepts ☑ · workflow ☑ · tools/commands ☑ · pitfalls ☑ · fixes/defences ☑ · engagement use ☑ · CTF use ☑
> **Named gaps (see `../GAPS.md`):** 35 exercise questions deliberately not attempted and no target spawned in any of the 14 interactive sections (§6 ×2, §7 ×6, §8 ×2, §9 ×4, §10 ×2, §11 ×6, §12 ×3, §13 ×2, §14 ×2, §15 ×1, §16 ×2, §19–21 ×1 each); the three skills labs (§19–21) are scenario-text only; my worked notes are blank — I completed this module before this note system existed.
> **Account note:** HTB shows this module as already completed by me (badge: "You need to trace before you can hunt"). Its completion page shows the Penetration Tester path at 21.8%.

Systematically extracting information from common infrastructure services (DNS, mail, file sharing, databases, and similar) to build a picture of a target before exploiting anything.

**Key concepts to own:** DNS enumeration; SMB / NFS / FTP / SMTP / SNMP basics; database service enumeration; banner and version analysis.

## Deep summary — section by section (original words)

### Method first (§1–5)

**1. Enumeration Principles** — Enumeration = systematic info gathering, split passive (public sources, no contact) vs active (direct interaction). What you gather before scanning decides what you test; unverified info wastes effort or misleads. Rule: multi-source verification, and document where every fact came from. *Use: both.*

**2. Enumeration Methodology** — Three stages, in order: (1) infrastructure (domains, cloud, people), (2) host-based (per-service), (3) remote management protocols. Manual work for accuracy + tools for speed; Nmap `-sV -sC --script` plus per-service tools. Pitfall: skipping the infrastructure stage or trusting automation alone. *Use: both.*

**3. Domain Information** — WHOIS, DNS records (A, AAAA, MX, NS, TXT, SOA), subdomains, web footprinting. Sources: `whois`, `dig` (incl. trying AXFR), crt.sh certificate transparency (certs leak forgotten subdomains), Google dorks, Wayback Machine, subdomain brute force / sublist3r / amass. Output: a full subdomain + IP inventory, verified across sources. *Use: both.*

**4. Cloud Resources** — Check all three major clouds (AWS, GCP, Azure): storage names are predictable, so build candidate bucket/blob names from company/domain words and read the responses — public S3 buckets, Azure blobs and GCP buckets expose files, backups, sometimes credentials. Pitfall: checking one provider only, assuming cloud is private by default. Defence: make buckets private, use provider hardening checklists. *Use: both.*

**5. Staff** — People enumeration from public sources (LinkedIn, socials, job boards, company pages): profiles reveal tech stack and projects, and names let you deduce the email/username format (first.last, flast, …) to build a user list for later phases. Tools: theHarvester, crosslinked, Hunter-style lookups. Scope discipline: public data only, inside the agreed scope, source recorded per name. *Use: engagement (CTF as recon practice).*

### Host-based — one service at a time (§6–16)

*Pattern for every service below: Nmap version + service scripts first → native client → note the version for known issues → the dangerous setting is the finding.*

**6. FTP (21)** — Modes active/passive; the dangerous settings are anonymous login and especially anonymous **write**. Workflow: `nmap -sV -sC -p 21 --script ftp-*` (ftp-anon, ftp-syst, ftp-brute) → `ftp <ip>` anonymous → list/download → note writable dirs → version check. Defence: disable anonymous, prefer SFTP. *(2 Q — gap.)*

**7. SMB (139/445)** — Shares, permissions, users; dangerous: guest access, null sessions, SMBv1. Workflow: `nmap --script smb-*` (enum-shares, enum-users, enum-sessions, os-discovery, security-mode) → rpcclient for users/groups → `smbclient -L` for shares → smbmap for permissions → read exposed shares; enum4linux / CrackMapExec (NetExec) as alternates. Remember share permissions ≠ NTFS permissions — check both. Defence: no guest/null sessions, SMB signing, v2/v3 only. *(6 Q — gap.)*

**8. NFS (2049, +111)** — Exports say what's shared with whom. Workflow: `nmap --script nfs-*` → `showmount -e <ip>` → `mount -t nfs <ip>:/share /mnt` → inspect. The critical misconfiguration is **no_root_squash**: a client root user keeps root rights on the share and can create root-owned files. Defence: restrict exports per-host, root_squash on, NFSv4 + Kerberos. *(2 Q — gap.)*

**9. DNS (53)** — Records A/AAAA/CNAME/MX/NS/TXT/SOA/PTR; try a zone transfer (`dig -t AXFR`), brute-force subdomains (dnsenum, dnsrecon, fierce, sublist3r), check recursion and version disclosure (`dig CHAOS TXT version.bind`). Dangerous: AXFR allowed, open recursion, version leaked. Defence: restrict transfers, no public recursion, hide version. *(4 Q — gap.)*

**10. SMTP (25/587/465)** — Mail flow HELO/EHLO → MAIL FROM → RCPT TO → DATA. User enumeration via VRFY / EXPN / RCPT TO responses (`smtp-user-enum -M VRFY|RCPT|EXPN`), Nmap `smtp-*` scripts incl. smtp-open-relay; connect manually with telnet/nc. MX records locate the server (check for external MX too). Dangerous: open relay (send mail as the domain), user enumeration. Defence: disable VRFY/EXPN, authenticated relaying only, SPF/DKIM/DMARC. *(2 Q — gap.)*

**11. IMAP / POP3 (143/993, 110/995)** — IMAP keeps mail server-side with folders; POP3 downloads (often deletes). Mail credentials usually match SMTP, so one confirmed account opens the mailbox — where password resets and sensitive data sit. Workflow: Nmap capability scripts → TLS connect (`openssl s_client -connect <ip>:993` or `curl imaps://<ip>/INBOX`) → LOGIN → LIST → SELECT → FETCH. Dangerous: plaintext auth without TLS, passwords written to logs. Defence: TLS required, no plaintext auth, MFA, never log passwords. *(6 Q — gap.)*

**12. SNMP (161/udp)** — Monitoring protocol; the MIB/OID tree holds system data, processes, software, routing — and community strings (`public`/`private`) are its passwords, sent in cleartext on v1/v2c (v3 adds auth + privacy). Workflow: `nmap -sU -p 161 --script snmp-*` → brute strings (onesixtyone, braa, snmp-brute) → `snmpwalk -v2c -c <string> <ip>` / snmp-check. Dangerous: default strings, write access enabled. Defence: SNMPv3, change defaults, restrict to management hosts, read-only. *(3 Q — gap.)*

**13. MySQL (3306)** — System databases: mysql, information_schema, performance_schema, sys. Workflow: `nmap --script mysql-*` (info, users, empty-password, variables, audit) → `mysql -u <user> -p -h <ip>` → `show databases; use <db>; show tables;` → read data; check **FILE privilege** (`load_file` / `into outfile`) — it turns database access into file read/write. Dangerous: weak root password, anonymous users, no bind-address restriction, broad FILE grants. Defence: bind localhost, strong passwords, no anonymous users, least privilege. *(2 Q — gap.)*

**14. MSSQL (1433/1434)** — System databases: master, model, msdb, tempdb, resource. It runs as a service account, so database access can become host control. Workflow: `nmap --script ms-sql-*` (info, empty-password, config, xp-cmdshell) → connect (Impacket `mssqlclient.py`, SSMS, mssql-cli) → enumerate databases/roles → check **xp_cmdshell** (OS command execution — should be disabled). Auth: Windows vs SQL auth (watch the `sa` account). Defence: Windows auth preferred, strong sa, xp_cmdshell off, encrypted connections. *(2 Q — gap.)*

**15. Oracle TNS (1521)** — The listener exposes the database's SID/service name. Workflow: `nmap --script oracle-tns-version` → guess the SID (ODAT sidguesser, oracle-sid-brute) → test default accounts (ODAT passwordguesser and its other modules) → `sqlplus user/pass@<ip>:1521/<SID>` → query. Dangerous: default accounts enabled, listener with no password, weak SYS/SYSTEM. Defence: change defaults, disable unused accounts, restrict and encrypt the listener. *(1 Q — gap.)*

**16. IPMI (623/udp)** — Out-of-band hardware management (the BMC: iLO, iDRAC, generic IPMI) — power and console control independent of the OS, so it's *more* powerful than an OS login. Two named flaws: IPMI 2.0's RAKP flaw leaks a salted password hash (dump with `nmap --script ipmi-dumphashes` or Metasploit, crack offline with Hashcat mode 7300), and cipher-zero authentication bypass. Workflow: detect (`ipmi-version`, `ipmi-cipher-zero`) → try vendor defaults → dump + crack hashes (ipmitool for interaction). Defence: change defaults, isolate the BMC on a management network, strong passwords, disable cipher zero. *(2 Q — gap.)*

### Remote management (§17–18)

**17. Linux Remote Management** — SSH (22): check version and config (ssh-audit) — SSH-1 is deprecated, prefer key auth + MFA. Rsync (873): `rsync --list-only` can list/download shares; anonymous rsync is a finding. Legacy R-services (rexec 512, rlogin 513, rsh 514, plus rstat/ruptime/rwho) trust by host via /etc/hosts.equiv and ~/.rhosts — a `+` entry trusts *everyone*, and a trusted host can be impersonated. Defence: remove R-services, restrict rsync. *Use: both.*

**18. Windows Remote Management** — RDP (3389, check NLA — rdp-sec-check), WinRM (5985/5986 — evil-winrm), WMI (135 — Impacket wmiexec.py), xfreerdp for RDP sessions. All three turn valid credentials *or a hash* into remote command execution, i.e. lateral-movement paths. Defence: NLA + MFA enforced, restrict to management hosts, disable what's unused. *Use: both.*

### Skills labs (§19–21, scenario text only)

**19. Easy** — Chain infrastructure + host enumeration: staff/forum OSINT (SSH keys discussed publicly, credentials found by teammates) feeds the host stage. Lesson: do the OSINT first; one technique alone fails.
**20. Medium** — A multi-service box (SSH/SCP, HTTPS behind a reverse proxy, a mail web app, NFS, a web portal): enumerate *everything* first, then work each service, then use the weakest. Pitfall: missing the hidden NFS or the proxied web service.
**21. Hard** — A mail + management server that is *also* the backup server for internal accounts: the credentials live in its secondary (backup) role. Lesson: check every role a server has, not just its main one.

## My worked notes (yours, as you do the labs)

- *(Blank — module completed before this note system. The per-service exercises in §6–16 and the three skills labs are the hands-on pieces worth re-doing and recording here, service by service.)*
