# HTB Junior Penetration Tester — Private Notes

Penetration Tester Job Role Path (CPTS path) · 28 modules · 495 sections

> **PRIVATE REPO ONLY.** These are personal study notes, summarized in original
> words. They contain no Hack The Box lesson text, no assessment answers, and
> no flags. Do not make this repository public and do not share it.

## How to use this repo

| File | What it is for |
|---|---|
| `engagement-ready.md` | Professional workflow, scope/rules checklists, finding & report templates — for authorized assessments |
| `ctf-ready.md` | Time-boxed box workflow, stuck-reset, per-box notes template — for CTFs and practice boxes |
| `knowledge-base/` | One file per module (28) — the full long-term reference to use after the certificate |
| `cheatsheet.md` | Quick-scan reference: enumeration, web loop, privesc, AD, pivoting, tool roles |
| `PROGRESS.md` | Module-by-module tracker — what is outline-only, in progress, or deep-complete |
| `GAPS.md` | Every known information gap, explicitly listed. If it is not in here or marked complete in `PROGRESS.md`, it is not done |

## The no-gaps rule

A module is only marked **deep-complete** in `PROGRESS.md` when every section of
that module on HTB Academy has been accounted for and its coverage checklist
(concepts, workflow, tools/commands, pitfalls, fixes/defences, engagement use,
CTF use) is fully ticked in its knowledge-base file. Outline-only is a stated
state, never a silent one.

## Versions

- **V1 (2026-11-09):** Built from the public HTB Academy path outline. All 28
  modules present at outline level; Engagement / CTF / Cheatsheet complete as
  generic references. Deep summaries: 0 of 28 modules.
- **V2 (2026-11-09):** Module 1 (Penetration Testing Process) deep-complete — all 15 sections read in the logged-in account and summarized. Deep summaries: 1 of 28 modules, 15 of 495 sections.
- **V3 (2026-11-09):** Module 2 (Getting Started) deep-complete — all 23 sections read and summarized. Deep summaries: 2 of 28 modules, 38 of 495 sections.
- **V4 (2026-11-09):** Module 3 (Network Enumeration with Nmap) deep-complete — all 12 sections read and summarized; cheatsheet gained a dedicated Nmap quick reference (D9). Deep summaries: 3 of 28 modules, 50 of 495 sections.
- **V5 (2026-11-09):** Module 4 (Footprinting) deep-complete — all 21 sections read and summarized; cheatsheet gained a per-service enumeration quick table (D10). Deep summaries: 4 of 28 modules, 71 of 495 sections.
- **V6 (2026-11-09):** Module 5 (Information Gathering — Web Edition) deep-complete — all 19 sections read and summarized; cheatsheet gained the web recon sequence (D11). Deep summaries: 5 of 28 modules, 90 of 495 sections.
- **V7 (2026-11-09):** Module 6 (Vulnerability Assessment) deep-complete — all 17 sections read and summarized; cheatsheet gained VA discipline notes (D12). Deep summaries: 6 of 28 modules, 107 of 495 sections.
- **V8 (2026-11-09):** Module 7 (File Transfers) deep-complete — all 10 sections read and summarized; cheatsheet gained file-transfer quick picks (D13). Deep summaries: 7 of 28 modules, 117 of 495 sections.
- **V9 (2026-11-09):** Module 8 (Shells & Payloads) deep-complete — all 17 sections read and summarized; cheatsheet gained the shells quick reference (D14). Deep summaries: 8 of 28 modules, 134 of 495 sections.
- **V10 (2026-11-09):** Module 9 (Using the Metasploit Framework) deep-complete — all 15 sections read and summarized; cheatsheet gained the Metasploit workflow (D15). Deep summaries: 9 of 28 modules, 149 of 495 sections.
- **V11 (2026-11-09):** Module 10 (Password Attacks) deep-complete — all 26 sections read and summarized; cheatsheet gained the password-attacks quick reference (D16). Deep summaries: 10 of 28 modules, 175 of 495 sections.
- **V12 (2026-11-09):** Module 11 (Attacking Common Services) deep-complete — all 19 sections read and summarized; cheatsheet gained the common-services attack map (D17). Deep summaries: 11 of 28 modules, 194 of 495 sections.
- **V13 (2026-11-09):** Module 12 (Pivoting, Tunneling, and Port Forwarding) deep-complete — all 18 sections read and summarized; cheatsheet gained the pivoting decision guide (D18). Deep summaries: 12 of 28 modules, 212 of 495 sections.
- **V14 (2026-11-09):** Module 13 (Active Directory Enumeration & Attacks) deep-complete — all 36 sections read and summarized; cheatsheet gained the AD attack path (D19). Deep summaries: 13 of 28 modules, 248 of 495 sections — past the halfway mark.
- **V15 (2026-11-09):** Module 14 (Using Web Proxies) deep-complete — all 15 sections read and summarized; cheatsheet gained the web-proxy workflow (D20). Deep summaries: 14 of 28 modules, 263 of 495 sections.
- **V16 (2026-11-09):** Module 15 (Attacking Web Applications with Ffuf) deep-complete — all 13 sections read and summarized; cheatsheet gained the ffuf playbook (D21). Deep summaries: 15 of 28 modules, 276 of 495 sections.
- **V17 (2026-11-09):** Module 16 (Login Brute Forcing) deep-complete — all 13 sections read and summarized; cheatsheet gained the brute-forcing quick reference (D22). Deep summaries: 16 of 28 modules, 289 of 495 sections.
- **V18 (2026-11-09):** Module 17 (SQL Injection Fundamentals) deep-complete — all 17 sections read and summarized; cheatsheet gained the SQL injection workflow (D23). Deep summaries: 17 of 28 modules, 306 of 495 sections.
- **V19 (2026-11-09):** Module 18 (SQLMap Essentials) deep-complete — all 11 sections read and summarized; cheatsheet gained the SQLMap quick reference (D24). Deep summaries: 18 of 28 modules, 317 of 495 sections.
- **V20 (2026-11-10):** Module 19 (Cross-Site Scripting (XSS)) deep-complete — all 10 sections read and summarized; cheatsheet gained the XSS workflow (D25). Deep summaries: 19 of 28 modules, 327 of 495 sections.
- **V21 (2026-11-10):** Module 20 (File Inclusion) deep-complete — all 11 sections read and summarized; cheatsheet gained the file-inclusion playbook (D26). Deep summaries: 20 of 28 modules, 338 of 495 sections.
- **V22 (2026-11-10):** Module 21 (File Upload Attacks) deep-complete — all 11 sections read and summarized; cheatsheet gained the upload bypass ladder (D27). Deep summaries: 21 of 28 modules, 349 of 495 sections.
- **V23 (2026-11-10):** Module 22 (Command Injections) deep-complete — all 12 sections read and summarized; cheatsheet gained command-injection operators + filter evasion (D28). Deep summaries: 22 of 28 modules, 361 of 495 sections.
- **V24 (2026-11-10):** Module 23 (Web Attacks) deep-complete — all 18 sections read and summarized; cheatsheet gained verb tampering · IDOR · XXE (D29). Deep summaries: 23 of 28 modules, 379 of 495 sections.
- **V25 (2026-11-10):** Module 24 (Attacking Common Applications) deep-complete — all 33 sections read and summarized; cheatsheet gained the common-applications attack map (D30). Deep summaries: 24 of 28 modules, 412 of 495 sections.
- **V26 (2026-11-10):** Module 25 (Linux Privilege Escalation) deep-complete — all 28 sections read and summarized; cheatsheet gained the Linux privesc checklist (D31). Deep summaries: 25 of 28 modules, 440 of 495 sections.
- **V27 (2026-11-10):** Module 26 (Windows Privilege Escalation) deep-complete — all 33 sections read and summarized; cheatsheet gained the Windows privesc checklist (D32). Deep summaries: 26 of 28 modules, 473 of 495 sections.
- **V28 (2026-11-10):** Module 27 (Documentation & Reporting) deep-complete — all 8 sections read and summarized; cheatsheet gained the reporting & notes template (D33). Deep summaries: 27 of 28 modules, 481 of 495 sections.
- **V29 (2026-11-10) — FINAL:** Module 28 (Attacking Enterprise Networks) deep-complete — all 14 sections read and summarized; cheatsheet gained the capstone engagement playbook (D34). **The full path is now deep-complete at lesson-text level: 28 of 28 modules, 495 of 495 sections.** What remains open is only my own hands-on: the exercise/assessment questions, targets, and worked notes throughout (see GAPS.md) — those are mine to do, module by module.
