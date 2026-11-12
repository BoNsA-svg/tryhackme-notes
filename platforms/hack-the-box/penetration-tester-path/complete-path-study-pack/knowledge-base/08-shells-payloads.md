# 8. Shells & Payloads

> **Status:** Deep-complete (lesson text) — read in full 2026-11-09, 17 / 17 sections
> **Outline metadata:** Medium, Tier 0, 17 sections, 10 interactive exercises, 0 assessments (as displayed; page header says "19 Interactive Questions" but 26 question entries were encountered — see gaps), est. 2 days
> **Coverage checklist:** concepts ☑ · workflow ☑ · tools/commands ☑ · pitfalls ☑ · fixes/defences ☑ · engagement use ☑ · CTF use ☑
> **Named gaps (see `../GAPS.md`):** 26 question entries deliberately not attempted (§3 ×2, §4 ×2, §5 ×2, §7 ×2, §9 ×3, §10 ×2, §13 ×2, §14 ×2, §15 ×2, §16 ×7) and no target spawned; the §16 Live Engagement capstone is scenario-text only; display count (19) vs encountered (26) unreconciled.
> **Access note (transparency):** this module was initially **locked** on the account; the reader clicked its free "Unlock Module" button to gain read access. The unlock cost nothing, marked nothing complete, submitted nothing, and changed no progress or settings — but it was an account action beyond pure navigation, and it is recorded here rather than hidden. Future modules may need the same.
> **Account note:** this module was **not** already completed by me (no badge, no progress shown) — the notes run ahead of my hands-on here too.

How reverse and bind shells and payloads work on Windows and Linux, how to catch them, stabilize them, and upgrade a dumb shell into a usable terminal.

**Key concepts to own:** reverse vs bind shells; listeners; shell stabilization; TTY upgrade; payload types.

## Deep summary — section by section (original words)

**1. Shells & Payloads (concepts)** — A shell takes typed instructions and returns text output; "catching a shell" = exploiting a weakness so you can run commands on the target. It's a *foothold*, not the goal — it opens privilege escalation, pivoting, file transfers, persistence. A payload is the code that exploits the weakness and delivers the shell (the word also means packet data / the instruction part of code / malware itself). Chain: find weakness → deliver payload → land shell → work. *Use: both.*

**2. Engagement Preparation (framing)** — The module is structured as a junior tester's graded preparation (fictional CAT5 Security): bind/reverse shells, payload basics, Windows shell, Linux shell, web shell, spotting shells/payloads, then a final end-to-end challenge (§16). Work the skills in order; the capstone assumes all of them. *Use: engagement framing / CTF training.*

**3. Anatomy of a Shell** — Three parts: the **terminal emulator** (displays text), the **command language interpreter** (Bash, PowerShell, Zsh, cmd — turns input into OS calls), and the OS. The emulator is not the language. Identify your interpreter before running anything (`ps`, `env`/`$SHELL`) — behaviour differs between shells. In ATT&CK terms a shell is a "Command and Scripting Interpreter" execution technique. *(2 Q — gap.)* *Use: both.*

**4. Bind Shells** — The target listens; you connect in. Needs the listener already running on the target, and inbound firewall/NAT usually block it — which is why this section mainly builds the contrast with reverse shells. A plain netcat pipe carries messages, not a shell: the **FIFO trick** (`mkfifo` + `cat` + `/bin/bash -i`) routes input and output through the connection to make it a real shell. Test firewall behaviour early. *(2 Q — gap.)* *Use: both.*

**5. Reverse Shells** — You listen; the target connects *out* — outbound is often overlooked, and a common port like 443 improves the odds. Workflow: listener on your side (`nc -lvnp`) → trigger a one-liner on the target that connects back and hands I/O to a shell. Use the Reverse Shell Cheat Sheet as a base but **customize** — defenders signature the stock commands, and DPI can spot a cleartext shell even on 443. Expect endpoint protection to react. *(2 Q — gap.)* *Use: both.*

**6. Introduction to Payloads** — Read a one-liner line by line before you run it: Bash = FIFO + shell + redirects + connection; PowerShell = TCPClient + stream + byte buffer + read loop + Invoke-Expression + send-back. Understanding each part is how you adapt a payload past a block — and how a defender spots it. Choose the form that fits the target's OS, interpreters and languages. A copied command that breaks silently is the failure mode. *Use: both.*

**7. Automating Payloads & Delivery with Metasploit** — The framework automates the repetitive parts: identify the service → `search` a matching module → `use` → `set` options (target, share, credentials, LHOST) → `exploit` → Meterpreter session (the walkthrough lands SYSTEM via an SMB module). Your job is choosing the right module and understanding what it will do — running a destructive module blind is the pitfall. (Full treatment in Module 9.) *(2 Q — gap.)* *Use: both.*

**8. Crafting Payloads with MSFvenom** — Build payloads as *files* when the network route isn't available — delivery by email, link, social engineering, USB. Key choice: **staged** (small loader pulls the rest — fragile if the second stage can't download) vs **stageless** (everything at once — better on poor networks). Build for the exact OS/architecture (`-f exe`/`elf`, LHOST/LPORT), encode against signature AV, set your listener, plan the delivery. *Use: both.*

**9. Infiltrating Windows** — Windows dominates business endpoints, so its famous flaw classes (SMB, Print Spooler, RDP, DNS, auth weaknesses) are the biggest foothold source. Workflow: fingerprint first (TTL, `nmap -O/-A`, banners) → match a known exploit (the walkthrough checks MS17-010 with a scanner module, then runs the psexec variant with a reverse Meterpreter payload) → SYSTEM shell. Choose cmd (simple, text) vs PowerShell (objects, scriptable) deliberately. Pitfall: assuming an old famous exploit works on a patched host. *(3 Q — gap.)* *Use: both.*

**10. Infiltrating Unix/Linux** — Most of the web runs on Unix-like systems, so a vulnerable web app is usually the route to the OS behind it. Workflow: scan (services + app version) → find a matching exploit (sometimes a community module you copy into Metasploit's tree) → run it → Meterpreter session → **upgrade the limited shell to interactive** (TTY). Version-match the exploit to the application or it fails. *(2 Q — gap.)* *Use: both.*

**11. Spawning Interactive Shells** — A raw shell can't run interactive commands (sudo!), so upgrade to a TTY. Ladder: `/bin/sh -i` → an interpreter (Python `pty.spawn`, Perl, Ruby, Lua) → a system tool that can exec (awk, find -exec, vim). Then immediately check your position: `ls -la`, `sudo -l` — a NOPASSWD sudo entry may itself be the next step. Don't rely on Python alone; know several routes. *Use: both.*

**12. Introduction to Web Shells** — Browser-based command access on a web server, placed via upload flaws, SQLi, file inclusion or command injection (PHP/JSP/ASP.NET; WAR deployment on Java). Treat it strictly as the *initial* foothold: web shells are unstable and get removed — upgrade to a reverse shell quickly, clean up, document the entry point. *Use: both.*

**13. Laudanum** — A ready-made web-shell collection in many languages (`/usr/share/laudanum`). Stock files are signatured by AV, so **edit before use**: set your address, strip identifying comments/art. Workflow: copy in the target's language → customize → upload via the app's upload function → browse to the uploaded path (confirm the location first) → run commands. *(2 Q — gap.)* *Use: both.*

**14. Antak Webshell** — An ASP.NET shell (from Nishang) presenting a PowerShell-like interface: each command runs as a new process, can run scripts in memory, encodes commands. Prepare it like Laudanum: copy → **rename** → set its protective credentials → strip art/comments → upload → log in → run commands → use it to call back to your listener. Pitfalls: stock filename or default credentials left in place; AV flagging on upload. *(2 Q — gap.)* *Use: both.*

**15. PHP Web Shells** — PHP is the most common server language, so a PHP shell fits most web targets. Where the app only allows images, intercept the upload in **Burp Suite** and change the file's Content-Type to an allowed image type to pass the filter, then browse to the file and run commands. Record the file's hashes and location for the report; move to a reverse shell quickly and delete the payload afterwards. *(2 Q — gap.)* *Use: both.*

**16. The Live Engagement (capstone)** — End-to-end scenario (text only): from a foothold host *inside* the target network, choose recon → exploit → payload → shell for three hosts (Windows, Linux, web app) and gather what's asked. Attacking from the wrong network and skipping per-host validation are the failure modes. *(7 Q — gap.)* *Use: engagement.*

**17. Detection & Prevention** — The defender's mirror: shells/payloads leave traces in uploads, unusual commands and abnormal sessions. Detection: log shell/PowerShell use and app logs, NetFlow baselines (top talkers, heartbeats on odd ports — 4444 is Meterpreter's default), DPI, endpoint AV/firewall/patching, ATT&CK as the shared vocabulary. Prevention is defence in depth: sandbox exposed apps, least privilege, segmentation + DMZ, NAT/firewall so inbound can't reach internal hosts, STIG hardening. Pitfalls: disabled AV/firewall; exceptions outside change management. *Use: engagement (shapes how you operate).*

## My worked notes (yours, as you do the labs)

- *(To fill in fresh — this module is not started on my account. The interactive sections (§3–5, §7, §9–10, §13–15) and the §16 Live Engagement capstone are the hands-on core; record per-target which shell route actually worked.)*
