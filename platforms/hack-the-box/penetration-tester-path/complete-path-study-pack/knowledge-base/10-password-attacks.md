# 10. Password Attacks

> **Status:** Deep-complete (lesson text) — read in full 2026-11-09, 26 / 26 sections
> **Outline metadata:** Medium, Tier 0, 26 sections, 22 interactive exercises, 0 assessments (as displayed), est. 3 days
> **Coverage checklist:** concepts ☑ · workflow ☑ · tools/commands ☑ · pitfalls ☑ · fixes/defences ☑ · engagement use ☑ · CTF use ☑
> **Named gaps (see `../GAPS.md`):** 63 question entries deliberately not attempted (spread across 21 sections — counts per section in the summaries) and no target spawned; three questions require downloading attached files (a ZIP in §6, a ZIP in §16, a pcap in §18) — nothing was downloaded, per the standing rules; the §26 skills assessment is scenario-text only. The lessons print credentials inside some exercise questions — none copied here or anywhere.
> **Access note:** initially locked; free "Unlock Module" used (read access only, as disclosed in V9). No PRO feature clicked.
> **Account note:** this module was **not** already completed by me — the notes run ahead of my hands-on here too.

How passwords are stored and attacked: capturing hashes, cracking them offline, spotting weak or default credentials, and reusing credentials (including pass-the-hash style techniques) where cracking fails.

**Key concepts to own:** hash types and storage; wordlists and rules; offline cracking workflow; credential reuse; default credentials.

## Deep summary — section by section (original words)

### Foundations & cracking (§1–7)

**1. Introduction** — Authentication (proving who you are: know / have / are / somewhere-you-are factors) vs authorization (what you may do). Passwords remain the primary corporate authenticator, and the human math is bad: short passwords from small alphabets fall fast, weak passwords and reuse are widespread, and few people change a password after a breach. Personal rule from the module: check your own passwords against breach data (HaveIBeenPwned); a breached password is a compromised password. *Use: both.*

**2. Introduction to Password Cracking** — A hash is a fixed-size one-way output; systems store the hash, and cracking = finding an input that produces the same hash. Techniques: **rainbow tables** (precomputed guesses — defeated cheaply by **salting**, since even a small salt explodes the table size), **brute force** (guaranteed, too slow for long passwords), **dictionary** (real-world lists like rockyou, ~14M entries — far faster). Speed depends enormously on hash type: fast hashes (MD5) crack orders of magnitude faster than slow ones (DCC2). Workflow: identify the type → pick the technique → run. Defence: slow, salted hashing. *(1 Q — gap.)* *Use: both.*

**3. Introduction to John The Ripper** — JtR (jumbo build) and its three modes: **single crack** (candidates derived from account/GECOS fields + rules — catches name-based passwords), **wordlist** (each entry, optionally rule-mangled), **incremental** (Markov-model systematic guessing). Identify the format first (`hashid -j`); hundreds are supported. File passwords become crackable via the `*2john` converter family. Run: hash to a file → `john --wordlist=… [--rules] [--format=…]` → `john --show`. *(2 Q — gap.)* *Use: both.*

**4. Introduction to Hashcat** — GPU-accelerated cracking. You must supply the **hash mode** (`-m`, identify via help/example-hashes/`hashid -m`) and **attack mode** (`-a`): 0 = dictionary, 3 = mask. Rules (`-r`, e.g. best64) mutate wordlist entries; masks test a pattern with charset placeholders (`?l ?u ?d ?s ?a`, custom sets `-1`–`-4`). Choose dictionary+rules when the structure is unknown, a mask when part of the pattern is known — and **estimate the runtime before starting a wide mask**. *(3 Q — gap.)* *Use: both.*

**5. Writing Custom Wordlists and Rules** — Custom beats generic: users build passwords from personal/org patterns, and complexity policies push them into predictable shapes (capital first letter, appended year/month, trailing symbol). Rules expand one base word into its likely variants; generate candidates to a file (`--stdout`) to inspect before cracking. OSINT seeds the base list — e.g. **CeWL** scrapes a target's own site vocabulary. Pitfall: every rule × every word explodes; stay targeted and in scope. *(1 Q — gap.)* *Use: both.*

**6. Cracking Protected Files** — Encrypted documents, PDFs and SSH keys found on a host. Find candidates by extension or by key header (`BEGIN … PRIVATE KEY`); test whether an SSH key is passphrase-protected by deriving its public key (a prompt means protected). Extract a crackable hash with the matching converter (ssh2john, office2john, pdf2john, zip2john…) → crack with a wordlist → use the recovered password. Unknown extension ≠ unprotected — research it first. *(1 Q, requires a download — gap.)* *Use: both.*

**7. Cracking Protected Archives** — ZIP / 7z / RAR and BitLocker volumes: same pattern (extract hash → crack → open/mount). BitLocker specifics: AES, unlocked by password/PIN/48-digit recovery key; `bitlocker2john` extracts, and the **password** hash form cracks with hashcat `-m 22100` — don't confuse it with the recovery-key hash form. Mount decrypted volumes with dislocker on Linux/macOS. *(2 Q — gap.)* *Use: both.*

### Remote attacks (§8–9)

**8. Network Services** — Online brute force against WinRM (5985/5986), SSH, RDP (3389), SMB (445): enumerate service + users first, then attack with **Hydra** (`-L/-P`, per-service modules; note old Hydra fails on SMBv3) or **NetExec** (`(Pwn3d!)` marks an admin-grade login), then connect with the native client (evil-winrm, xfreerdp, smbclient, Metasploit's smb_login). The governing risk is **account lockout** — online attacks are loud by nature. Defence: lockout policy, MFA on remote access, no defaults. *(4 Q — gap.)* *Use: both.*

**9. Spraying, Stuffing, and Defaults** — The low-noise trio. **Spraying**: one common password × many accounts (stays under per-account lockout thresholds) — NetExec, Kerbrute for AD user validation. **Stuffing**: replay breached username:password pairs, betting on reuse — Hydra `-C`. **Defaults**: factory credentials on routers/firewalls/databases, checked against vendor docs / default-credential lists — firmware versions matter. Defence: change defaults at deployment; MFA makes one leaked password insufficient. *(1 Q — gap.)* *Use: both.*

### Extracting from Windows (§10–15)

**10. Windows Authentication Process** — The logon chain: credential provider → **WinLogon** → **LSASS** (protected process; enforces policy, authenticates, audits) → an authentication package → local **SAM** or a domain controller's **NTDS.dit**. SAM is SYSKEY-encrypted on disk and locked while Windows runs — you cannot just read it live. Credential Manager stores secrets in the Vault/Credentials folders. Defence anchor points: protect LSASS (Credential Guard), monitor hive access. *Use: both.*

**11. Attacking SAM, SYSTEM, and SECURITY** — The three hives together unlock local credentials: SAM (LM/NT hashes), SYSTEM (the boot key that decrypts SAM), SECURITY (LSA secrets incl. cached domain logons for offline use). Needs admin. Workflow: `reg save` the hives → move them over a share (Impacket smbserver) → **secretsdump** → crack the NT hash (hashcat `-m 1000`) or reuse it; cached domain credentials are DCC2 (`-m 2100`, PBKDF2 — ~800× slower, rarely worth a casual crack). NetExec can dump remotely (`--sam` / `--lsa` with local auth). DPAPI is the encryption layer over many of these secrets (Impacket dpapi, mimikatz, DonPAPI). *(3 Q — gap.)* *Use: both.*

**12. Attacking LSASS** — LSASS caches logon material in memory: dump the process (Task Manager "Create dump file", or the comsvcs MiniDump route — noisy and commonly flagged by AV) → parse offline with **pypykatz** → NT/SHA1 hashes, Kerberos data, DPAPI master keys, and plaintext only where WDIGEST still provides it (older Windows). Crack NT hashes with a dictionary. Defence: LSASS as protected process (PPL) + Credential Guard + access auditing. *(2 Q — gap.)* *Use: both.*

**13. Attacking Windows Credential Manager** — The Vault stores web/Windows/generic credentials (saved by "Remember me" and apps), AES-encrypted with keys themselves protected by DPAPI; Credential Guard instead isolates secrets in a virtualization enclave. Enumerate with `cmdkey /list` or the stored-credential UI; reuse directly via `runas /savecred`, or decrypt offline once the DPAPI master key is available (mimikatz vault/dpapi, SharpDPAPI, LaZagne, DonPAPI). Note: a saved credential outlives its purpose — old logins still work. *(1 Q — gap.)* *Use: both.*

**14. Attacking Active Directory and NTDS.dit** — Domain logons validate against a DC, so the sequence is: build usernames (naming conventions, emails, **username-anarchy**) → validate them (**Kerbrute** userenum — carefully; validation failures log Event 4776 and spraying can hang/lock) → if a DC is compromised, take **NTDS.dit** (every domain hash) via a volume shadow copy (`vssadmin`) together with the SYSTEM hive → **secretsdump** `-ntds` → crack (`-m 1000`) or pass-the-hash. *(4 Q — gap.)* *Use: both.*

**15. Credential Hunting in Windows** — Think like an admin about where passwords end up: config files, scripts, browser stores, password-manager exports, Group Policy/SYSVOL, unattend.xml, AD description fields, shared drives. Start broad (keyword + pattern search, e.g. `findstr` for password-related terms) across the file types that actually occur — including non-English keywords — then use **LaZagne**'s modules (browsers, chats, mail, sysadmin, wifi…) for application stores. *(5 Q — gap.)* *Use: both.*

### Linux & network (§16–19)

**16. Linux Authentication Process** — PAM is the auth framework. `/etc/passwd` is world-readable (user, UID/GID, home, shell; the password field is normally `x`), `/etc/shadow` is root-only and holds `$<id>$<salt>$<hash>` plus aging fields (`!`/`*` = no login; empty = no password). Algorithm ids include MD5, Blowfish, SHA-256, SHA-512, Yescrypt (current Debian default). `/etc/security/opasswd` keeps old hashes. A writable `/etc/passwd` would be catastrophic (delete root's field → passwordless root). Combine with `unshadow` and crack (sha512crypt = hashcat `-m 1800`); don't expect MD5-era speeds from modern hashes. *(2 Q, one requires a download — gap.)* *Use: both.*

**17. Credential Hunting in Linux** — Four hunting grounds: **files** (configs, databases, notes, scripts, cron, logs — grep broadly: password variants, pwd, token, key), **history** (bash history/config can hold plaintext), **memory** (tools like mimipenguin extract keys/passwords from processes), **key-rings & browsers** (GNOME Keyring / KDE Wallet; Firefox's logins.json + key4.db via Firefox Decrypt). Start in $HOME; **LaZagne** (Linux) automates much of it. Also hunt world-writable files while you're there. *(1 Q — gap.)* *Use: both.*

**18. Credential Hunting in Network Traffic** — TLS hides most modern traffic, but plaintext protocols persist on legacy/misconfigured systems (HTTP, FTP, SNMPv2, POP3/IMAP/SMTP without TLS, LDAP, VNC…). In **Wireshark**: filter to the protocol/port, follow the stream, search for credential strings. **Pcredz** automates extraction from a pcap or live capture: mail/FTP logins, SNMP strings, HTTP Basic/NTLM, NTLMv1/v2 hashes from SMB/LDAP/MSSQL/HTTP, Kerberos AS-REQ pre-auth hashes (crackable offline). Defence: kill plaintext protocols, enforce TLS, monitor for cleartext credentials. *(4 Q, one requires a download — gap.)* *Use: both.*

**19. Credential Hunting in Network Shares** — Shares accumulate secrets in configs, scripts and documents. Search strategically: IT/script shares over photo shares; keywords incl. local-language variants; short generic filenames are suspicious. Scale: manual browsing (smbclient/smbmap, PowerShell Select-String) → dedicated hunters — **Snaffler** (domain-joined), **PowerHuntShares** (HTML report), **MANSPIDER** (from Linux), NetExec `--spider --content --pattern`. *(2 Q — gap.)* *Use: both.*

### Lateral movement with stolen material (§20–23)

**20. Pass the Hash (PtH)** — Authenticate with the **NTLM hash itself** — no plaintext needed; the hash stays valid until the password changes. Sources: SAM dumps, NTDS.dit, LSASS. Tools by service: mimikatz `sekurlsa::pth`, Impacket psexec/wmiexec/atexec/smbexec, NetExec `smb -H <hash>` (marks local-admin hosts), evil-winrm `-H`, xfreerdp `/pth:` (needs Restricted Admin Mode). Constraint: UAC token filtering means, for local accounts, effectively only the built-in RID-500 administrator administers remotely by default — and **LAPS** randomizing local admin passwords blunts reuse. Use the NT hash, not LM. *(7 items — gap.)* *Use: both.*

**21. Pass the Ticket (PtT) from Windows** — Kerberos in brief: prove your hash once → get a **TGT** from the KDC → exchange it for service tickets (TGS) that services accept without a password. Tickets sit in LSASS cache: export/import with mimikatz (`sekurlsa::tickets /export` → .kirbi, `kerberos::ptt`) or **Rubeus** (dump, `ptt`, `asktgt`, `createnetonly`); a hash/key can be turned straight into a TGT ("overpass-the-hash"). Check the ticket's user and expiry before trusting it; format mistakes break the import. *(4 items — gap.)* *Use: both.*

**22. Pass the Ticket (PtT) from Linux** — Domain-joined Linux uses Kerberos too: tickets as **ccache** files (usually /tmp, selected by KRB5CCNAME) and long-term keys in **keytab** files (hunt cron/scripts that `kinit` with one). Reuse: copy a ccache and point KRB5CCNAME at it (back up your own first), or `kinit -k -t <keytab>`; keytab hashes can be extracted for offline cracking. Impacket tools take `-k -no-pass`; evil-winrm needs a Kerberos config. If the DC isn't directly reachable, tunnel (Chisel + Proxychains). Principal names are case-sensitive — a classic failure. *(10 items — gap.)* *Use: both.*

**23. Pass the Certificate** — PKINIT lets a **certificate** (smart-card style) authenticate to Kerberos instead of a hash. Two attack shapes: **ESC8** — relay an NTLM authentication to an AD CS **HTTP** enrollment endpoint so it issues a certificate *for the victim*, then use it to request a TGT (gettgtpkinit); **Shadow Credentials** — if you can write a user's msDS-KeyCredentialLink attribute, you can attach your own key and mint credentials for them (pywhisker). Certificate logons are common in smart-card environments, not an edge case. Defence: HTTPS + protections on enrollment endpoints, monitor KeyCredentialLink changes, disable the Spooler where unneeded. *(2 Q — gap.)* *Use: both.*

### Management & assessment (§24–26)

**24. Password Policies** — Policy design per NIST SP800-63B / CIS / PCI DSS. Modern guidance: change on suspected breach, **not** on a fixed rotation schedule — forced rotation trains users to increment a trailing digit and still "comply". A blacklist (company name, months, seasons, common words) does more than length rules. Enforce via AD Group Policy, communicate it, audit it. Passphrase-style passwords are strong but OSINT can expose the personal facts they're built from. *Use: both.*

**25. Password Managers** — The practical fix for ~100 accounts: one strong master password → a master key → an encrypted vault (providers claim zero-knowledge). Cloud (sync, extensions) vs local/self-hosted (KeePass etc.) is the main fork. A manager alone isn't enough — MFA on the vault; complements/alternatives: FIDO2 keys, TOTP, IP restrictions, device compliance, passwordless. *Use: both.*

**26. Skills Assessment (scenario)** — End-to-end chain against a fictional company network: an externally exposed DMZ host, then private internal hosts (jump, file, DC) reachable only by **pivoting through the DMZ host**; the roadmap is the credential-theft shuffle — foothold → local admin → lateral movement → domain credentials. The stated defences mirror the module: segmentation, MFA everywhere, LAPS, restricted admin rights. *(1 Q — gap; read as scenario text only.)* *Use: engagement.*

## My worked notes (yours, as you do the labs)

- *(To fill in fresh — this module is not started on my account, and with 22 interactive sections it is the most hands-on module so far. Priorities when working it: §3–4 cracking fluency, §11–14 extraction chain, §20–22 reuse techniques.)*
