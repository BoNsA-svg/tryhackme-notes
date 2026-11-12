# 26. Windows Privilege Escalation

> **Status:** Deep-complete (lesson text) — read in full 2026-11-10, 33 / 33 sections
> **Outline metadata:** Medium, Tier 2, 33 sections, 26 interactive exercises, 2 assessments (as displayed), est. 4 days
> **Coverage checklist:** concepts ☑ · workflow ☑ · tools/commands ☑ · pitfalls ☑ · fixes/defences ☑ · engagement use ☑ · CTF use ☑
> **Named gaps (see `../GAPS.md`):** 47 question entries deliberately not attempted (46 questions + 1 optional exercise in §26, incl. the two skills assessments §§32–33) and no target spawned. 10 Hint buttons and §26's "Reveal Answer" disclosure were present and deliberately not clicked. Many question prompts supply lab RDP login details — none were copied anywhere (credential material, per the standing rule).
> **Access note:** initially locked; free "Unlock Module" used (read access only, as disclosed in V9). No PRO feature clicked.
> **Account note:** this module was **not** already completed by me (reader showed 0%) — the notes run ahead of my hands-on here too.

Raising privileges on Windows by enumeration: who you are, patch level, installed software, services, scheduled tasks, stored credentials, and common misconfigurations on modern and legacy systems.

**Key concepts to own:** situational awareness first; unquoted service paths / weak permissions (concepts); token privileges; stored creds hunting; patch comparison.

**Goal state:** Local Administrators membership or a SYSTEM token — and sometimes a better-placed *other user* is the real prize (their session, their saved credentials). Same discipline as Linux: enumerate first, tools second, exploits matched to build.

## Deep summary — by category (original words)

### Lay of the land (§§1–5)

**1. Introduction** — Why escalate: gold-image testing, local resources, a domain foothold from a domain-joined SYSTEM box, credential harvesting for lateral movement. Vectors: token privileges, group privileges, UAC, weak permissions, kernel patch gaps, credential theft, traffic capture. Managed workstations may have no internet/USB and a tight firewall — manual skill and locally staged tools (C:\Tools) are the fallback, not the exception. *Use: both.*

**2. Useful Tools** — The automation bench: Seatbelt, winPEAS, PowerUp, SharpUp, JAWS, SessionGopher, Watson, LaZagne, WES-NG, Sysinternals. Each covers a different layer (misconfigs, services, password stores, patch gaps, ACLs). In client environments prefer **source-compiled** builds — stock binaries get flagged — and treat every tool finding as a lead to verify by hand; tools are noisy and can crash services. *Use: both.*

**3. Situational Awareness** — Orient before acting: ipconfig /all, arp -a, route print (dual-homed hosts, DC addresses, traces of admin RDP/WinRM sessions = lateral-movement leads). Check the defences **first** — Get-MpComputerStatus, Get-AppLockerPolicy — because they decide which tools/PoCs can even run; know AppLocker's default rules (Program Files/Windows/signed apps allowed for Everyone) so a block doesn't surprise you. *(2 Q — gap.)* *Use: both.*

**4. Initial Enumeration** — The repeatable checklist: systeminfo, tasklist /svc, wmic qfe / Get-HotFix, installed products, netstat -ano, quser, whoami /priv + /groups, net user / localgroup / administrators, net accounts. Rank targets (SYSTEM > built-in admin > local-admin member > domain user in local admins) and let the inventory choose the vector. Re-run after every privilege gain. *(5 Q — gap.)* *Use: both.*

**5. Communication with Processes** — Tokens, loopback services, and **named pipes** are privilege paths: services bound to localhost are *assumed* safe and often aren't (vendor admin interfaces with defaults), and a pipe with a weak DACL accepts your commands. Enumerate with netstat -ano, PipeList, `gci \\.\pipe\`, AccessChk; check which account each service runs as. Beware masqueraded pipe names in detection — and in your own OPSEC. *(2 Q — gap.)* *Use: both.*

### User privileges (§§6–9)

**6. Privileges Overview** — Privileges ≠ access rights: a token carries User SID + Group SIDs + privileges; an object's DACL decides access. Privileges can sit **disabled** in the token and must be enabled before use; admin logons may hand you a *filtered* token. Detection note: event 4672 logs special-privilege logons. *(Theory.)* *Use: both.*

**7. SeImpersonate / SeAssignPrimaryToken** — Service accounts (SQL, web, Jenkins) commonly hold SeImpersonate → a **Potato** tool tricks a SYSTEM process into authenticating to you, and you run as SYSTEM. Version discipline: JuicyPotato dies on Server 2019 / Win10 1809+ — PrintSpoofer or RoguePotato there. Fix: strip the privilege from service accounts that don't need it. *(1 Q — gap.)* *Use: both.*

**8. SeDebugPrivilege** — Debug rights = read other processes' memory: ProcDump LSASS → Mimikatz for an admin's NTLM hash, or spawn a child with a privileged parent's token. Usually admins/developers only — which is exactly why it's valuable. Watch for misleading PoC output (an "error" may mean success). Fix: restrict "Debug programs", monitor LSASS access. *(1 Q — gap.)* *Use: both.*

**9. SeTakeOwnershipPrivilege** — WRITE_OWNER over anything: enable it, takeown the target, icacls yourself access, read, then **revert ownership and ACLs** — taking ownership is destructive and auditable. Backup/VSS-style service accounts are the usual holders. *(1 Q — gap.)* *Use: both.*

### Group privileges (§§10–15)

**10. Built-in Groups + Backup Operators** — The module's group map: Backup Operators, Event Log Readers, DnsAdmins, Hyper-V Administrators, Print Operators, Server Operators. Backup Operators in focus: SeBackupPrivilege reads *through* deny ACEs (robocopy /B needs no extra tooling) and a shadow copy yields the whole NTDS offline (diskshadow → reg save → secretsdump). Stale membership is the root cause — audit it. *(1 Q — gap.)* *Use: both.*

**11. Event Log Readers** — 4688 process-creation auditing logs **full command lines**, and passwords ride command lines: membership lets you harvest credentials from the Security log (wevtutil / Get-WinEvent) with no exploit at all. PowerShell script-block logging leaks the same way. Fix: never pass secrets as parameters; centralize and restrict logs. *(1 Q — gap.)* *Use: both.*

**12. DnsAdmins** — Members can set the DNS service's **ServerLevelPluginDll**; DNS runs as SYSTEM, so your DLL loads as SYSTEM on the DC at next restart (dnscmd to configure; clean the registry value after). Restarting DNS on a DC is destructive — explicit permission only. Same group can also plant WPAD records. *(1 Q — gap.)* *Use: both.*

**13. Hyper-V Administrators** — Full Hyper-V control: if DCs are virtualized, treat this group as **Domain Admins** — clone a live DC, mount its disk offline, extract NTDS.dit. The lesson's hard-link vector (CVE-2018-0952 / -2019-0841) is patched — build-check before relying on it. *(Theory.)* *Use: both.*

**14. Print Operators** — The group carries **SeLoadDriverPrivilege**: register and load a signed-but-vulnerable driver (the Capcom pattern) → kernel execution as SYSTEM. Version-dependent (HKCU driver loading died at Win10 1803; HKLM only after). Remove the registry key in cleanup. *(1 Q — gap.)* *Use: both.*

**15. Server Operators** — SeBackup/SeRestore **plus control of local services**: find a LocalSystem service the group fully controls, `sc config` its binPath to your command (e.g. add yourself to Administrators), restart it. Trap: error 1053 on start can still mean your command *ran* — verify membership before declaring failure. *(1 Q — gap.)* *Use: both.*

### Attacking the OS (§§16–20)

**16. User Account Control** — UAC is a convenience, **not a security boundary**: admin logons get split tokens, and bypasses ride auto-elevating binaries' DLL search order (a writable folder in the path — e.g. under the user profile — is enough). Check EnableLUA/ConsentPromptBehaviorAdmin and the build, then match a UACMe technique; "Always notify" is the level with few bypasses. *(1 Q — gap.)* *Use: both.*

**17. Weak Permissions** — Writable service binaries, editable service configs (binPath/ImagePath), weak registry and autorun ACLs: SharpUp/AccessChk enumerate them; back up the original binary, swap or repoint, restart, **restore**. Installers pointing services into user-writable folders are common. UsoSvc (CVE-2019-1322) is the named example. *(1 Q — gap.)* *Use: both.*

**18. Kernel Exploits** — Patch level → build-matched exploit: the catalogue runs MS08-067/MS17-010 through the ALPC 0-day, **HiveNightmare CVE-2021-36934** (readable SAM hives), and **PrintNightmare CVE-2021-1675/-34527**. Sherlock/WES-NG/Metasploit do the matching. Kernel exploits can crash the host — tested, build-exact PoCs only, and patch lag itself is the reportable finding. *(1 Q — gap.)* *Use: both.*

**19. Vulnerable Services** — Third-party services running as SYSTEM with local unauthenticated flaws (the lesson's Druva inSync case) give SYSTEM on an otherwise patched host. Enumerate installed software + listening ports, match versions to public flaws. Prefer a reverse shell over adding an admin user — quieter and less system-changing. *(1 Q — gap.)* *Use: both.*

**20. DLL Injection** — Code inside a trusted process's context: LoadLibrary via VirtualAllocEx/WriteProcessMemory/CreateRemoteThread, manual mapping, or reflective injection; discovery via ProcMon/Process Explorer (NAME NOT FOUND hijacks, proxying). Safe DLL Search Mode ordering decides which DLL wins. Defence: full paths, signed libraries, load monitoring. *(Theory.)* *Use: both.*

### Credential theft (§§21–23)

**21. Credential Hunting** — findstr for "password" across the filesystem; the usual graves: web.config, **unattend.xml** (should be deleted post-install, often isn't), PowerShell history (ConsoleHost_history.txt), Chrome's custom dictionary, and DPAPI-protected stores — decryptable only as the owning user *on that machine* (Import-Clixml pattern). *(2 Q — gap.)* *Use: both.*

**22. Other Files** — Shares and local oddments: Snaffler-style crawling, Sticky Notes' plum.sqlite, backups, virtual disks (guestmount). Some prizes (pagefile.sys, repair\sam, ntuser.dat, CCM logs) need elevation — **re-run the file hunt after every privilege gain**. Automation assists; manual search covers what scripts miss. *(1 Q — gap.)* *Use: both.*

**23. Further Credential Theft** — The saved-credential estate: cmdkey /list + runas /savecred, browsers (SharpChrome), KeePass databases (keepass2john → hashcat 13400; often one master password from everything), mail (MailSniper), LaZagne, SessionGopher, the Winlogon **AutoLogon** registry key, PuTTY sessions, and Wi-Fi keys via netsh. Collection is *detectable* (4688/16385) — defenders watch this; so should your report. *(4 Q — gap.)* *Use: both.*

**24. Citrix Breakout** — Locked-down published desktops (Citrix/PSM/kiosk) fall to their seams: a File→Open dialog (MS Paint + UNC paths), alternative file managers/registry editors, edited shortcut targets, script file types (.bat/.vbs/.ps1), and AlwaysInstallElevated once you have a prompt. Then it's ordinary privesc. Harden the published desktop, not just the perimeter. *(2 Q — gap.)* *Use: engagement.*

### Additional techniques (§§25–27)

**25. Interacting with Users** — Harvest from live user activity: passive capture (Npcap/tcpdump → net-creds), process command-line diffing, then active plants — **.lnk files** (SCF is dead on Server 2019) with UNC icons on a busy share trigger Responder/Inveigh NetNTLMv2 capture (hashcat -m 5600); allow a few minutes for a browse. *(1 Q — gap.)* *Use: both.*

**26. Pillaging** — Systematic post-exploitation harvest: installed apps → their stores. mRemoteNG's confCons.xml (decryptable; default master password), browser cookies → session hijack (SharpChromium; cookies live under the Network subfolder now), clipboard logging, and **restic** backup repos → restore → local hashes. The first host is a source, not the finish line. *(5 Q + 1 optional exercise — gap.)* *Use: both.*

**27. Miscellaneous Techniques** — The sweep-up list: LOLBins (certutil download/encode/decode, rundll32), **AlwaysInstallElevated** (needs *both* policy keys → MSI as SYSTEM), the certificate-dialog flaw CVE-2019-1388 (patched Nov 2019), writable scheduled-task scripts, secrets in user **description fields**, and mounted virtual disks → secretsdump on the hives. *(1 Q — gap.)* *Use: both.*

### End-of-life systems (§§28–30)

**28. Legacy Operating Systems** — EOL means unpatched forever; the skill is the *recommendation*: identify version + EOL date, establish the business reason it persists, rate the risk, propose segmentation/extended support/retirement. Mission-critical legacy boxes are fragile — agree handling with the client before touching them. *(Theory.)* *Use: engagement.*

**29. Windows Server 2008** — No Credential Guard/Device Guard/JEA/CFG: Sherlock/Exploit Suggester → matched exploits (MS10-092 schelevator pattern); mind architecture (migrate to a 64-bit process). EternalBlue only with explicit permission. Context decides the advice — a hospital's 2008 box gets segmentation, not a shrug. *(1 Q — gap.)* *Use: both.*

**30. Windows 7** — Same EOL logic on the desktop estate: systeminfo → updated Exploit Suggester DB → filter suggestions to *this* OS/arch (the list is full of DoS and mismatches), MS16-032 as the worked example. Hundreds of POS terminals change the remediation conversation. *(1 Q — gap.)* *Use: both.*

**31. Windows Hardening** — The mirror: trusted gold images (WDS/SCCM), WSUS patching, GPO-managed configuration against CIS/STIG baselines (Security Compliance Toolkit, STIG Viewer), Sysmon + network telemetry, Secure Boot + BitLocker, Credential/Device Guard, absolute paths in elevated tasks, no cleartext creds in readable places, history cleanup. Audits supplement pentests; neither replaces the other, and hardening must be tailored — never applied blind network-wide. *(Theory.)* *Use: engagement.*

**32–33. Skills Assessments I–II (scenarios)** — I: a non-domain server — command-injection foothold → found credentials → second flaw → SYSTEM, on a client whose patching is strong but configuration management is weak (the module's thesis in one box). II: a **gold-build review** — a standard Win10 image for 1,200+ employees, audited from a no-internet standard account: stage your own tools, enumerate completely, escalate, and report *every* vector so the baseline gets fixed. *(7 Q total — gap; scenario text only; prompts' login details not copied.)* *Use: both.*

## My worked notes (yours, as you do the labs)

- *(To fill in fresh — this module is not started on my account. Keep the Linux module's habit: an enumeration log you re-run after every privilege change — §§22/26 reward exactly that. The two assessments are the module in miniature: Part I for the exploit chain, Part II for gold-image auditing — do Part II's enumeration with tool output saved, since the deliverable there is the vector list, not just the shell.)*
