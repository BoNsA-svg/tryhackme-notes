# 13. Active Directory Enumeration & Attacks

> **Status:** Deep-complete (lesson text) — read in full 2026-11-09, 36 / 36 sections
> **Outline metadata:** Medium, Tier 2, 36 sections, 29 interactive exercises, 2 assessments (as displayed), est. 7 days
> **Coverage checklist:** concepts ☑ · workflow ☑ · tools/commands ☑ · pitfalls ☑ · fixes/defences ☑ · engagement use ☑ · CTF use ☑
> **Named gaps (see `../GAPS.md`):** 75 question entries deliberately not attempted (per-section counts in the summaries; the two skills assessments alone carry 20) and no target spawned. The lessons and question prompts print example credentials/hashes — none copied here or anywhere.
> **Access note:** initially locked; free "Unlock Module" used (read access only, as disclosed in V9; the unlock toast appeared with a delay). No PRO feature clicked.
> **Account note:** this module was **not** already completed by me (reader showed 0%) — the notes run ahead of my hands-on here too. This is the largest module in the path.

How Active Directory is structured, how to enumerate users, groups, computers, policies and trust, and the common AD misconfigurations and attack paths (Kerberos abuse, delegation issues, ACL mistakes) testers look for.

**Key concepts to own:** AD structure: domains, OUs, GPOs; LDAP / Kerberos basics; BloodHound graph thinking; Kerberoasting / AS-REP roasting; ACL and delegation abuse.

## The module's arc (read this first)

External recon → initial domain enumeration → **foothold** (LLMNR/NBT-NS poisoning) → **password policy, then spraying done responsibly** → enumerate the security controls on your foothold → **credentialed enumeration** (Linux and Windows, BloodHound) → living off the land → **Kerberoasting** → **ACL abuse** → **DCSync** → privileged access and the double-hop problem → current vulnerabilities (NoPac, PrintNightmare, PetitPotam) → misc misconfigurations → **domain trusts** (child→parent, cross-forest) → hardening and auditing → two full skills assessments. The scenario frame: authorized internal pentests for the fictional Inlanefreight domains — scope is written down (§3) and staying inside it is part of the exercise.

**Tool-per-phase habit (§2):** keep an inventory — recon (Kerbrute, enum4linux-ng, ldapsearch/windapsearch), capture (Responder/Inveigh), enumeration (PowerView, BloodHound/SharpHound, CrackMapExec/NetExec, Snaffler), Kerberos abuse (Rubeus, GetUserSPNs.py, GetNPUsers.py), ACL/relay/coercion (Impacket suite, ntlmrelayx, PetitPotam), escalation (Mimikatz, secretsdump), audit (PingCastle, Group3r, ADRecon). Know which are noisy before you run them.

## Deep summary — section by section (original words)

**1. Introduction** — AD centralizes identity, authentication and authorization for Windows enterprises, so one AD weakness compounds into domain-wide impact. The module's path: recon → foothold → credentialed enumeration → lateral movement → privilege escalation → domain compromise → defences. Lab attack hosts: a Windows box and a Parrot Linux box. *Use: engagement.*

**2. Tools of the Trade** — The per-phase tool table (see frame above). Failure mode: forcing a Windows-only tool onto a Linux host, or running a tool without knowing what it touches on the domain. *Use: both.*

**3. Scenario** — Two authorized internal pentests (one from an external breach position, one from inside) against named fictional domains/subnet; phishing, other domains/IPs and the real company's public website (beyond passive recon) are explicitly out of scope. Lesson: confirm scope in writing; never confuse the lab's fictional domain with a real one. *Use: engagement.*

**4. External Recon and Enumeration Principles** — Passive first: WHOIS, DNS, certificates, search engines, archives reveal infrastructure and people without touching the target. Record sources; distrust single-source findings; go active only when authorized. *(1 Q — gap.)* *Use: both.*

**5. Initial Enumeration of the Domain** — First active look from a network position: discover hosts and services, identify DCs and key servers (incl. SQL Server), map the domain's shape. Correlate service versions with known issues before acting; not every open port is equally interesting. *(2 Q — gap.)* *Use: both.*

**6. LLMNR/NBT-NS Poisoning — from Linux** — When DNS fails, hosts broadcast LLMNR/NBT-NS name queries; answering them (**Responder**) makes clients send you their **NTLMv2 hashes** — crack offline (Hashcat), reuse the credentials. Defence: disable LLMNR/NBT-NS; enforce SMB signing. Never poison a production network without authorization. *(3 Q — gap.)* *Use: both.*

**7. LLMNR/NBT-NS Poisoning — from Windows** — Same capture from a Windows foothold with **Inveigh** (PowerShell) when no Linux attack host is available; identical crack-and-reuse tail. Extra risk: running it from a monitored host. *(1 Q — gap.)* *Use: both.*

**8. Password Spraying Overview** — Theory: a few common passwords × many accounts stays under per-account lockout thresholds. Slow, logged, and agreed with the client in advance — a lockout storm is a client incident, not a trophy. *Use: both.*

**9. Enumerating & Retrieving Password Policies** — Read the policy **before** spraying: min length, complexity, lockout threshold and duration define how many attempts are safe and how long to wait between rounds (PowerView, LDAP, CrackMapExec). Fine-grained policies can differ per group — don't assume the default applies to everyone. *(2 Q — gap.)* *Use: both.*

**10. Password Spraying — Making a Target User List** — Valid names first (SMB null sessions, LDAP anonymous bind, **Kerbrute userenum** — fast and doesn't generate logon-failure events — or credentials in hand). Filter by lockout risk (badpwdcount, last-bad-attempt time) and **log every attempt**: account, DC, time, password tried; tell the client immediately if a lockout happens. *(1 Q — gap.)* *Use: both.*

**11. Internal Password Spraying — from Linux** — rpcclient / Kerbrute passwordspray / CrackMapExec; the same spray with `--local-auth` tests **local admin password reuse** across hosts. Pitfall: forgetting `--local-auth` can hammer (and lock) the built-in domain administrator. Defence: **LAPS** — unique local admin password per host kills reuse. *(1 Q — gap.)* *Use: both.*

**12. Internal Password Spraying — from Windows** — **DomainPasswordSpray** is the safer tool: it builds its own user list, reads the lockout policy itself, and excludes accounts within one attempt of lockout. Detection side: bursts of 4625 (logon failure) / 4771 (Kerberos pre-auth failure) events. Defence: MFA, separate admin accounts, passphrase hygiene. *(1 Q — gap.)* *Use: both.*

**13. Enumerating Security Controls** — After the foothold, gauge the terrain before choosing tools: Defender status (`Get-MpComputerStatus`), effective AppLocker policy, PowerShell language mode (Constrained Language?), and **who can read LAPS passwords** (LAPSToolkit). Controls vary per host because policy application is uneven — the classic gap: AppLocker blocking 64-bit PowerShell but not the 32-bit path or the ISE; domain-joiners holding extended rights that include reading LAPS passwords. *Use: engagement.*

**14. Credentialed Enumeration — from Linux** — Any valid credential (password, NTLM hash, or SYSTEM on a joined host) unlocks deep enumeration: users/groups/computers, logged-on users, shares — CrackMapExec/NetExec (`--users --groups --loggedon-users --shares`, spider), SMBMap, rpcclient, windapsearch — then ingest into **BloodHound** (bloodhound-python + Neo4j) and query for paths. Tool notes: psexec.py drops a binary+service (loud, SYSTEM); wmiexec.py is quieter but runs as the user. Watch default-readable shares (SYSVOL, NETLOGON) and who's logged on where — a DA session on a workstation is a path. *(2 Q — gap.)* *Use: both.*

**15. Credentialed Enumeration — from Windows** — **PowerView** for native in-memory enumeration (Get-DomainUser/Computer/Group/Controller, Test-AdminAccess, SPN hunting), **SharpHound** collection → BloodHound paths, **Snaffler** to hunt credentials in share content (its classic find: a web config with a database connection string). Corroborate BloodHound paths with direct checks before acting; data you never query is worthless. *(4 Q — gap.)* *Use: both.*

**16. Living Off the Land** — No internet, no uploads: native tools map host and domain — `net`, dsquery (with LDAP filters incl. userAccountControl bit-matching), WMIC, PowerShell. Quieter, **not unlogged**: PowerShell Operational logging (4104) and session events still record you, even on a downgraded PowerShell. Quirk: EDR often watches `net.exe` by name; `net1` does the same job without the string trigger. *(3 Q — gap.)* *Use: both.*

**17. Kerberoasting — from Linux** — Any domain user can request a service ticket (TGS) for accounts with **SPNs**; the ticket is encrypted with the service account's NTLM hash, so crack it offline: **GetUserSPNs.py** `-request` → Hashcat **mode 13100**. RC4 tickets (type 23) crack far faster than AES (type 18). A retrieved ticket proves nothing until the password cracks. Defence: long service passwords, (g)MSA with auto-rotation, audit RC4. *(2 Q — gap.)* *Use: both.*

**18. Kerberoasting — from Windows** — Same attack natively: setspn to enumerate, PowerView `Get-DomainSPNTicket` or **Rubeus kerberoast** to request, Mimikatz to export from memory; Hashcat 13100 (RC4) / 19700 (AES). Detection: "Audit Kerberos Service Ticket Operations" — a burst of 4769s (10–20 TGS requests per account quickly) is the tell. Note: Rubeus `/tgtdeleg` fails against Server 2019 DCs. *(2 Q — gap.)* *Use: both.*

**19. ACL Abuse Primer** — Every AD object carries an ACL of ACEs: the **DACL** grants/denies, the **SACL** audits. One wrong ACE can hand over a user (force password change), a group (add member), an object (write) or the domain itself (replication rights) — read ACEs as **chains toward privilege**, not a flat list. Some rights are easy to miss (e.g. ReadGMSAPassword). Changing passwords/memberships is disruptive — agree it with the client. *(2 Q — gap.)* *Use: both.*

**20. ACL Enumeration** — Don't dump the whole domain's ACLs (huge, 1–2 hrs in big environments): query by the **SID of the account you control**, resolve GUIDs to readable right names (`Get-DomainObjectACL -ResolveGUIDs`), and trace the chain; confirm with BloodHound's outbound-control view. Learn the manual method as a tooling fallback; document the chain. *(5 Q — gap.)* *Use: engagement.*

**21. ACL Abuse Tactics** — The worked chain: reset user A's password → add A to a nested help-desk group → use the group's rights to set a **temporary SPN** on a privileged account → Kerberoast *that* account (targeted Kerberoasting) — takeover without ever touching the target's password. **Cleanup order matters**: remove the SPN *before* the group membership (afterwards you no longer hold the right to remove it), restore everything. Detection: event 5136 (directory object modified) with Advanced Audit Policy. *(1 Q — gap.)* *Use: engagement.*

**22. DCSync** — Hold the **DS-Replication-Get-Changes-All** extended right and you can impersonate a DC and pull the domain's credential database via the replication protocol: **secretsdump.py** (`-just-dc-ntlm`, per-user, history, status options) or Mimikatz `lsadump::dcsync`. Yields every NTLM hash + Kerberos keys, incl. **krbtgt** (→ Golden Ticket), and exposes accounts with **reversible encryption** (cleartext!). Defence: replication rights only for DA/EA/DC computer accounts; audit them; disable reversible encryption. Use only with explicit client agreement. *(3 Q — gap.)* *Use: both.*

**23. Privileged Access** — Credentials become movement via three doors: RDP, WinRM/PSRemoting, SQL Server admin — enumerate who holds them per host first (BloodHound edges CanRDP / CanPSRemote / SQLAdmin; PowerView local-group queries), then connect (Enter-PSSession, evil-winrm, PowerUpSQL/mssqlclient). Watch the escalation gifts: SQL service accounts often hold SeImpersonatePrivilege → SYSTEM. RDP access ≠ admin rights. Log every host you touch. *(3 Q — gap.)* *Use: both.*

**24. Kerberos "Double Hop" Problem** — WinRM uses a network logon: your credentials are **not cached** on the first host, so the second hop (e.g. to the DC) fails with what looks like a permissions error; RDP's interactive logon caches them. Fixes: pass a PSCredential explicitly per command, or register a PSSession configuration with `-RunAsCredential` (needs an elevated PowerShell — not available from an evil-winrm shell). Plan the second hop before you rely on WinRM. *Use: engagement.*

**25. Bleeding Edge Vulnerabilities** — Three fast user→DA paths, each with real breakage risk (client communication is part of the section): **NoPac** (create a machine account, spoof its name as a DC, get a SYSTEM ticket, DCSync — blocked if MachineAccountQuota is 0), **PrintNightmare** (Spooler RPC loads your DLL — can crash the spooler), **PetitPotam** (coerce a DC to authenticate to AD CS, relay for a certificate → TGT; needs AD CS web enrollment). Defences: patch; quota 0; disable the spooler where possible; EPA + SSL on enrollment; disable NTLM where feasible. *(2 Q — gap.)* *Use: engagement.*

**26. Miscellaneous Misconfigurations** — The grab-bag that rewards broad AD knowledge: Exchange privileged groups, the Printer Bug (spooler coercion), MS14-068, LDAP sniffing, DNS records as intel (adidnsdump), **passwords in description fields**, PASSWD_NOTREQD accounts, **SYSVOL/GPP passwords** (patched in 2014 — old entries were never cleaned up), **AS-REP roasting** (accounts with pre-auth disabled: GetNPUsers.py / Rubeus asreproast), GPO abuse (group3r, SharpGPOAbuse). *(2 Q — gap.)* *Use: both.*

**27. Domain Trusts Primer** — Trusts link authentication between domains: parent-child, tree-root, cross-link, external, forest, bastion types; transitive or not; one-way or two-way. A weaker trusted domain is an attack path into the stronger one. **External trusts apply SID filtering; intra-forest trusts don't** — that difference decides which attacks cross. Enumerate and document every trust's direction and type before testing (Get-DomainTrust, netdom, BloodHound trust map). *(3 Q — gap.)* *Use: engagement.*

**28. Attacking Domain Trusts — Child → Parent (Windows)** — **SID-history (ExtraSids) attack**: compromise the child domain, DCSync its krbtgt, then forge a Golden Ticket whose *sidHistory* includes the parent's **Enterprise Admins SID**. Inside a forest there's no SID filtering, so the parent honors it — child compromise = forest compromise. Every input (SIDs, krbtgt hash) must be exact; the ticket's named user need not exist. Forest-wide impact → explicit client agreement. *(3 Q — gap.)* *Use: engagement.*

**29. Attacking Domain Trusts — Child → Parent (Linux)** — The Impacket version of the same attack: secretsdump for krbtgt, lookupsid.py for SIDs, **ticketer.py** to forge the ticket (ccache; mind KRB5CCNAME), psexec.py to use it, or **raiseChild.py** to automate the chain. Understand every step before running an autopwn-style script. *(1 Q — gap.)* *Use: engagement.*

**30. Attacking Domain Trusts — Cross-Forest (Windows)** — Across a **bidirectional forest trust**: Kerberoast the other forest's SPNs (Rubeus `/domain`), check for shared/reused admin passwords, and enumerate **foreign group membership** — only Domain Local groups can hold foreign principals, and a foreign member of a privileged group is a bridge. SID filtering limits what crosses; unique admin passwords per forest are the defence. *(1 Q — gap.)* *Use: engagement.*

**31. Attacking Domain Trusts — Cross-Forest (Linux)** — GetUserSPNs.py with `-target-domain` for cross-forest Kerberoasting; run **bloodhound-python against both domains** and check "Users with Foreign Domain Group Membership" — collecting only one side misses the link. Practical note: the collector needs the DC's DNS *hostname* (add it to resolv.conf), not just an IP. *(3 Q — gap.)* *Use: engagement.*

**32. Hardening Active Directory** — The defensive counterpart: annual audits (naming, OUs, GPOs, FSMO, apps, hosts, trusts, privileged users), then the hardening stack — LAPS, gMSA, Protected Users (careful: can break legitimate auth), MachineAccountQuota 0, SMB/LDAP signing, remove RC4 from Kerberos encryption types, disable the RID-500 admin in favor of a LAPS-managed account — mapped to ATT&CK per technique. Run BloodHound/PingCastle/Grouper regularly; test backups/DR; quarterly assessments where budget allows. Baseline before buying detection. *Use: engagement.*

**33. Additional AD Auditing Techniques** — Turn enumeration into client-fundable evidence: **AD Explorer** snapshots (before/after), **PingCastle** healthcheck (may need a system-date adjustment to start), **Group3r** for GPOs, **ADRecon** for the full dataset + Excel report (needs the GroupPolicy module). Reports with prioritization beat raw tool output. *(1 Q — gap.)* *Use: engagement.*

**34. Skills Assessment Part I (scenario)** — External start: a password-protected **web shell** in an upload directory → enumerate AD → Kerberoast → find credentials → escalate → flag on the DC. The 8 questions are milestones of **one chain**, not independent puzzles — record the full path, not just flags. *(8 Q — gap; scenario text only.)* *Use: engagement.*

**35. Skills Assessment Part II (scenario)** — Internal start from a client-provided Linux VM, full scope ahead of a merger: capture a foothold hash → crack → config/SQL credentials → ACL abuse (a user with GenericAll over Domain Admins) → DC → the **krbtgt hash** as proof. Report every flaw found, not only the ones on the DC path. *(12 Q — gap; scenario text only.)* *Use: engagement.*

**36. Beyond this Module** — AD continues into cross-domain trusts, persistence and C2, and transfers to hybrid/cloud. Backfill: Introduction to Active Directory; deepen: AD BloodHound / LDAP / PowerView modules, Hashcat module; practice machines and Pro Labs named; community canon (SpecterOps research, "Six Degrees of Domain Admin", ADSecurity, DFIR Report, ATT&CK). AD changes with each new vulnerability — treat this module as a foundation, not a finish. *Use: both.*

## My worked notes (yours, as you do the labs)

- *(To fill in fresh — this module is not started on my account, and at 29 interactive sections + 2 assessments it is the biggest hands-on block in the path. Suggested order when working it: §§6–12 foothold+spray, §§14–18 enumeration+Kerberoast, §§19–22 ACL+DCSync, §§27–31 trusts, then the two assessments back to back.)*
