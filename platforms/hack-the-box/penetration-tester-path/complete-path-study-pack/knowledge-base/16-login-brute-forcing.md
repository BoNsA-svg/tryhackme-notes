# 16. Login Brute Forcing

> **Status:** Deep-complete (lesson text) — read in full 2026-11-09, 13 / 13 sections
> **Outline metadata:** Easy, Tier 2, 13 sections, 8 interactive exercises, 1 assessment (as displayed), est. 6 hours
> **Coverage checklist:** concepts ☑ · workflow ☑ · tools/commands ☑ · pitfalls ☑ · fixes/defences ☑ · engagement use ☑ · CTF use ☑
> **Named gaps (see `../GAPS.md`):** 11 question entries deliberately not attempted (§3 ×1, §4 ×1, §7 ×1, §8 ×1, §10 ×2, §11 ×1, §12 ×2, §13 ×2 — the two-part skills assessment) and no target spawned. §12's provided wordlist links (usernames.txt / passwords.txt) were deliberately not clicked or downloaded.
> **Access note:** initially locked; free "Unlock Module" used (read access only, as disclosed in V9). No PRO feature clicked.
> **Account note:** this module was **not** already completed by me (reader showed 0%) — the notes run ahead of my hands-on here too.

Brute-forcing login services and web forms responsibly and effectively, choosing wordlists well, and recognizing lockout and rate-limit risks.

**Key concepts to own:** online vs offline attacks; targeting SSH / FTP / web forms; username generation; lockout awareness.

## Deep summary — section by section (original words)

**1. Introduction** — Brute forcing = systematic trial-and-error over combinations until one works; viability depends on password complexity, your compute, and the target's defences. The attack family: simple brute force, dictionary, hybrid, credential stuffing, spraying, rainbow tables, reverse brute force, distributed. Use it when other avenues are exhausted, policies are weak, or you're targeting specific accounts — not as a reflex. *Use: both.*

**2. Password Security Fundamentals** — Strength grows **exponentially** with length and charset (6 lowercase chars ≈ hundreds of millions of combos; 8 chars ≈ hundreds of billions). Judge a target by length/complexity/uniqueness/randomness plus its policy (NIST: 12+ characters, passphrases over complexity theater, no reuse). The soft spots: short/common/personal passwords, reuse, predictable patterns — and **default credentials** shipped on devices and services, which are a known, documented risk class. Defence: long unique random passwords, defaults changed at deployment. *Use: both.*

**3. Brute Force Attacks** — The math discipline: combos = charset^length; hardware speed decides whether the space is searchable at all. **Compute the space and estimate the time before you start** — if pure brute force is infeasible, switch to a smarter attack instead of burning hours. (The exercise is a 4-digit PIN against an app endpoint via a small script.) *(1 Q — gap.)* *Use: both.*

**4. Dictionary Attacks** — People pick memorable passwords, so a curated list finds most weak ones far faster than exhaustive search. Choose or build the list, **tailor it to the audience/system** (company, industry, hobby terms), iterate, and if it's not in the list, change approach — a list is a hypothesis, not an exhaustive space. Record which list you used. *(1 Q — gap.)* *Use: both.*

**5. Hybrid Attacks** — Forced rotation breeds predictable mutations (base word + number/symbol, year bumps); hybrid attacks mutate dictionary words to match — dictionary speed, wider coverage, much smaller space than pure brute force. Filter candidate lists to the target's policy with `grep -E` (length + required classes) so you don't waste attempts on impossible candidates. Credential stuffing (breached pairs replayed elsewhere) rides the same reuse problem; the defence is unique passwords + MFA + managers. *Use: both.*

**6. Hydra** — The fast parallel network login cracker: many protocols (ssh, ftp, http-get/http-post-form, smtp, pop3, mysql, rdp, vnc…). Core flags: `-l/-L` login(s), `-p/-P` password(s), `-t` tasks, `-f` stop on first success, `-s` port, `-M` target list, `-x` generate passwords (min:max:charset). Verify target and service before a big run; respect rate limits and lockouts; some services need extra options. *Use: both.*

**7. Basic HTTP Authentication** — Basic Auth mechanics: server replies 401 + `WWW-Authenticate`; browser sends `user:pass` Base64 in the Authorization header — a reversible encoding, so guessing is a straight brute-force problem: `hydra -l <user> -P <list> <ip> http-get -s <port>`. Check the actual port first; a 401 isn't proof a guess failed if the service is unreachable. Defence: long unique passwords, MFA — and TLS, since Basic Auth exposes credentials in transit without it. *(1 Q — gap.)* *Use: both.*

**8. Login Forms** — Hydra's `http-post-form`: `"path:params:condition"` with `^USER^`/`^PASS^` placeholders and an **F=** (failure text) or **S=** (success marker — a 302, a dashboard title) condition. The recon is the skill: inspect the form first (dev tools/proxy) for the POST path, field names, hidden fields and CSRF tokens — guessing field names is the classic failure, and a condition string that matches both outcomes finds nothing true. Prefer S= when it's unambiguous. *(1 Q — gap.)* *Use: both.*

**9. Medusa** — The parallel, modular alternative: `medusa -h/-H -u/-U -p/-P -M <module> -m <opts> -t <tasks> -f/-F` across many service modules (FTP, HTTP, IMAP, MySQL, POP3, RDP, SSHv2, Telnet, VNC, Web Form). The `-e n` (empty password) / `-e s` (password = username) checks test the weakest defaults. Rule of thumb from the module: Hydra for web forms, Medusa for parallel multi-service runs — and keep task counts within what the target tolerates. *Use: both.*

**10. Web Services** — Chained service cracking: crack SSH with Medusa, log in, **enumerate what else the host listens on** (`netstat -tulpn`, nmap localhost), then crack the second service (FTP) the same way — the username often falls out of context (e.g. a home directory name). Don't stop at the first cracked service; don't assume the same password works twice. Defence: unique passwords per service, key-based SSH, kill cleartext FTP. *(2 Q — gap.)* *Use: engagement.*

**11. Custom Wordlists** — Generic lists rarely contain *this* target's credentials; tailored ones do. Build from sourced OSINT: **Username Anarchy** generates username variants (initials, separators, leetspeak, interests); **CUPP** (`cupp -i`) profiles a person into password candidates; filter both to the target's policy with `grep -E`, then run Hydra. Quality of the source data decides success — guesswork about someone's life is not OSINT. Engagement rule: build from the client's own data, with permission; doing this to a real person without authorization is out of scope and unlawful. *(1 Q — gap.)* *Use: engagement.*

**12. Skills Assessment Part 1 (scenario)** — Chained: brute-force a Basic Auth login with the provided lists; the output (a username) is Part 2's input. Treat chained assessments as one workflow and preserve each step's output. *(2 Q — gap; scenario text only. The provided wordlist download links were not clicked.)* *Use: engagement.*

**13. Skills Assessment Part 2 (scenario)** — Use Part 1's username to brute the target login, identify the (different) FTP user, retrieve the file. Watch the trap: the login username and the FTP username are separate values. Document the chain so every step traces to the next. *(2 Q — gap; scenario text only.)* *Use: engagement.*

## My worked notes (yours, as you do the labs)

- *(To fill in fresh — this module is not started on my account. Work §§7–8 and §10 hands-on (Hydra Basic Auth, http-post-form conditions, Medusa service chaining), and build one CUPP/Username Anarchy list in §11 before attempting the two-part assessment.)*
