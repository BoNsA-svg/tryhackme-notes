# 9. Using the Metasploit Framework

> **Status:** Deep-complete (lesson text) — read in full 2026-11-09, 15 / 15 sections
> **Outline metadata:** Easy, Tier 0, 15 sections, 4 interactive exercises, 0 assessments (as displayed), est. 5 hours
> **Coverage checklist:** concepts ☑ · workflow ☑ · tools/commands ☑ · pitfalls ☑ · fixes/defences ☑ · engagement use ☑ · CTF use ☑
> **Named gaps (see `../GAPS.md`):** 9 question entries deliberately not attempted (§2 ×2, §4 ×1, §6 ×1, §10 ×3, §11 ×2) and no target spawned. The lessons print example credential material (hashes/secrets in the Meterpreter walkthrough, an API-key placeholder in the evasion section) — none of it was copied into these notes or anywhere else, by the standing rule; nothing in the summaries depends on those values.
> **Access note:** this module was also initially locked; the reader used its free "Unlock Module" button (as disclosed in V9), which granted read access only. No PRO feature was clicked.
> **Account note:** this module was **not** already completed by me — the notes run ahead of my hands-on here too.

Working in Metasploit: finding and configuring exploit modules, payloads, sessions, and basic post-exploitation, and knowing when the framework helps and when manual work is better.

**Key concepts to own:** msfconsole workflow; search / use / set / run; sessions; meterpreter basics; auxiliary modules.

## Deep summary — section by section (original words)

**1. Preface** — The tool-discipline argument: automation saves time and teaches, but creates comfort zones and tunnel vision ("the tool can't do it" ≠ "it can't be done"). Credibility comes from results, not tools. Rule: know the tool inside out, audit it, stay able to work by hand. *Use: both.*

**2. Introduction to Metasploit** — A Ruby-based, modular platform for writing, testing and executing exploit code, with modules for enumeration, attack and evasion under `/usr/share/metasploit-framework` (modules: auxiliary, encoders, evasion, exploits, nops, payloads, post). Its strength: matching known exploits to vulnerable service versions in a few commands, then switching between sessions. It's a Swiss-army knife for common unpatched flaws — not a substitute for manual skill. Pro = GUI/task chains/reporting; Framework = console. Keep it updated; read a module before running it. *(2 Q — gap.)* *Use: both.*

**3. Introduction to MSFconsole** — The console is the main interface (tab completion, external commands allowed; the banner reports version + module counts). Keep the framework current (`apt update && apt install metasploit-framework`) or new modules are missing. Engagement structure inside the console mirrors the process: enumeration → preparation → exploitation → privilege escalation → post-exploitation. Enumerate first; let service versions drive module choice. *Use: both.*

**4. Modules** — Prepared single-purpose scripts, organized by type/OS/service/name. An exploit module is a tested PoC — but **a failed exploit does not prove the target is safe**; modules often need tailoring. Workflow: `search` (filters: cve, author, platform, type) → `info` (read it) → `use` → `options` → `set` (host, credentials) → `run`; `setg` makes a value global across modules. Audit the module's code before running it. *(1 Q — gap.)* *Use: both.*

**5. Targets** — An exploit is written for specific OS builds: the return address differs per version/service pack. `show targets` lists them; `set target` picks one; "Automatic" detects first but can be wrong — when it matters, identify the build deliberately (the module's comments explain how targets were derived; msfpescan finds return addresses in target binaries). Wrong target = crash or miss. *Use: both.*

**6. Payloads** — Three forms: **Singles** (self-contained, stable, large), **Stagers** (small loader that calls back), **Stages** (the advanced part pulled down by a stager). Meterpreter is the flagship stage: memory-resident, hard to detect, extensible. Workflow: `show payloads` → `set payload` (fit the architecture!) → set LHOST/LPORT → run → work the session (`getuid`, `shell`, and Meterpreter's extended functions). Keep payloads small and inconspicuous; a payload too large for the buffer fails outright. *(1 Q — gap.)* *Use: both.*

**7. Encoders** — Encoders adapt a payload to an architecture and strip **bad characters** that would break it (`msfvenom -e <encoder> -b '<badchars>' -i <iterations>`; `show encoders` lists options). They once doubled as AV evasion; modern detection has caught up — **encoding is plumbing, not stealth**. Check generated files against detection (msf-virustotal) and validate on the real target class. *Use: both.*

**8. Databases** — The PostgreSQL backend records the whole engagement: hosts, services, vulns, credentials, loot — importable, exportable, searchable. Workflow: `msfdb init/status/run` → `db_nmap` or `db_import` scan results → query with `hosts`, `services`, `creds`, `loot` → `db_export` backups. **One workspace per project** (`workspace` command); mixing projects and stale data are the pitfalls; a failed init usually means an outdated framework. *Use: engagement.*

**9. Plugins & Mixins** — The two extension mechanisms. **Plugins** bring third-party tools into the console with results flowing into the database (`load <plugin>`, read its help first; examples across recon/scanner/post tooling). **Mixins** are Ruby's shared-behaviour mechanism that makes the framework itself extensible. Install = place the .rb in the plugins folder with right permissions → load → verify. Verify a plugin's source before loading it. *Use: engagement.*

**10. Sessions & Jobs** — Multi-host management: background a session with **Ctrl+Z**, list with `sessions`, re-enter with `sessions -i <id>`; run tasks as background **jobs** (`exploit -j`), list with `jobs`, kill with `kill` / `jobs -K`. Jobs free ports an exploit is holding. Pitfalls: Ctrl+C killing the console instead of managing the job; losing track of which session belongs to which host; a session dying when its payload runtime fails. *(3 Q — gap.)* *Use: both.*

**11. Meterpreter** — The in-memory post-exploitation workflow: land the session → **migrate** to a stable process → run the local exploit suggester → escalate to SYSTEM → harvest credential material (hashdump, secret stores) for reuse. Verify each suggested escalation path against the actual host build before running it; handle harvested material carefully. (The lesson's example credential values are deliberately not reproduced here.) *(2 Q — gap.)* *Use: both.*

**12. Writing & Importing Modules** — When the framework lacks a module: find one in Metasploit format (e.g. via ExploitDB/`searchsploit`) → copy the .rb into the matching module folder with snake_case naming → `loadpath` / `reload_all` → `search` and `use` it. Writing your own = adapt a similar module as boilerplate: metadata → register options → exploit code (Ruby). A script that merely *looks* like a module but isn't framework-compatible is the trap; audit third-party modules before running them. *Use: both.*

**13. Introduction to MSFVenom** — The payload builder (successor to the old separate payload/encoder tools): build for the target OS/format (`-p <payload> -f exe|elf|aspx …`), deliver it where the victim will run it, catch the call-back with `exploit/multi/handler`, then escalate from the low-privilege session (local exploit suggester). Uploaded ≠ executed — test the whole delivery path end to end and have the escalation step ready. *Use: both.*

**14. Firewall and IDS/IPS Evasion** — Defence model first: endpoint protection per host + perimeter rules at the edge; detection by signature, anomaly-vs-baseline, or protocol behaviour, watched by a SOC. Evasion workflow: learn what the rules actually allow → hide the payload in a legitimate executable template (`msfvenom -x … -k`) → pack/archive → **test against a sandbox before deploying**, because a loud first try can burn the only chance. Meterpreter's encrypted tunnels help in transit. Pitfalls: trusting encoding alone; obvious NOP sleds; never testing. Defence answer: depth — signatures + behavioural baselines + a team that acts on alerts. *Use: both.*

**15. MSF6 Updates (August 2020)** — What the MSF6 jump changed: end-to-end encryption across Meterpreter, SMBv3 support, polymorphic shellcode generation, cleaner payload artifacts, and the Mimikatz extension replaced by **Kiwi**. The cost: MSF6 broke compatibility with MSF5 sessions and payloads — after upgrading, regenerate payloads and re-establish sessions, and never upgrade mid-engagement assuming old payloads still work. *Use: both.*

## My worked notes (yours, as you do the labs)

- *(To fill in fresh — this module is not started on my account. The interactive sections are §4, §6, §10 and §11; record a full search→use→set→run→session flow per target as you do them.)*
