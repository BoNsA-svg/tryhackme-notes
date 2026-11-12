# 14. Using Web Proxies

> **Status:** Deep-complete (lesson text) — read in full 2026-11-09, 15 / 15 sections
> **Outline metadata:** Easy, Tier 2, 15 sections, 8 interactive exercises, 1 assessment (as displayed), est. 8 hours
> **Coverage checklist:** concepts ☑ · workflow ☑ · tools/commands ☑ · pitfalls ☑ · fixes/defences ☑ · engagement use ☑ · CTF use ☑
> **Named gaps (see `../GAPS.md`):** 11 question entries deliberately not attempted (§4 ×1, §7 ×1, §8 ×1, §9 ×1, §10 ×1, §11 ×1, §13 ×1, §15 ×4 — the skills assessment) and no target spawned; §8's question needs an attached file — the download link was not clicked (no-download rule); §6's two inline exercises have no submit control and its "show the answer" disclosure was not clicked.
> **Access note:** initially locked; free "Unlock Module" used (read access only, as disclosed in V9). No PRO feature clicked.
> **Account note:** this module was **not** already completed by me (reader showed 0%) — the notes run ahead of my hands-on here too.

Intercepting and modifying web traffic with Burp Suite and OWASP ZAP: proxy setup, intercept, repeater-style manual testing, and using the proxy as the center of web testing.

**Key concepts to own:** intercepting proxy workflow; scope; repeating and modifying requests; encoding / decoding; comparing responses.

## Deep summary — section by section (original words)

**1. Intro to Web Proxies** — A web proxy sits between browser/app and server as a man-in-the-middle for HTTP(S) — unlike a sniffer, it sees *only* web traffic, but in full request/response detail. The two tools: **Burp Suite** (built-in browser; Pro adds the active scanner, unthrottled Intruder, some extensions; Community is enough for most work) and **OWASP ZAP** (free, open source, no paid tier, no throttling). Learn both; the proxy is one instrument among scanners, fuzzers, crackers, sniffers. *Use: both.*

**2. Setting Up** — Both tools run on all major OSes and come preinstalled on PwnBox/Parrot/Kali; launch via installer, command (`burpsuite`, `zaproxy`) or `java -jar`. Burp Community can't save projects — use a temporary project for short work; ZAP can skip session persistence. *Use: both.*

**3. Proxy Setup** — Everything hinges on routing the browser through the listener (default **127.0.0.1:8080**): use the tool's pre-configured browser (proxy + CA cert already set), or configure Firefox manually (FoxyProxy helps) and **import the proxy's CA certificate** so HTTPS interception doesn't trigger warnings. Failure modes: port 8080 already in use (proxy won't start), missing CA cert (browser warnings). Install the cert once per browser profile. *Use: both.*

**4. Intercepting Web Requests** — Interception pauses a request *before* the server sees it: edit any parameter, then Forward or Drop. That's the foundation for injection and logic testing. Discipline: the browser sends many requests — **verify the intercepted one is the action you're testing** before editing. ZAP: break button / Ctrl+B, HUD Step (examine each) vs Continue. *(1 Q — gap.)* *Use: both.*

**5. Intercepting Responses** — Catch the response before rendering and change what the browser enforces: enable a disabled field, unhide hidden form fields, reveal HTML comments. Powerful for *exploring* behavior — but a client-side change is not a server-side finding; confirm real weaknesses server-side, and note the change vanishes on refresh (that's what §6 fixes). Burp: response-intercept rules / "Unhide hidden form fields"; ZAP HUD: Show/Enable, Comments view. *Use: both.*

**6. Automatic Modification** — Match-and-replace **rules** rewrite matching requests/responses on every pass — no per-request interception, and changes survive refreshes. Define: match (string or regex), replacement, target (request header/body, response body), scope. Burp: HTTP match and replace rules; ZAP: Replacer (Ctrl+R). Pitfalls: regex anchors (matching a whole header line), rules too broad (breaking app content) or too narrow (never firing). Test with interception on to confirm the rule fires. *(Two inline exercises — no questions; "show answer" not clicked.)* *Use: both.*

**7. Repeating Requests** — The workhorse loop: pull a request from **Proxy History** into **Repeater** (Burp, Ctrl+R) or ZAP's Request Editor, edit, send, compare — far faster than intercepting per test; "Change request method" flips GET/POST. Keep the original for comparison. Remember POST bodies are URL-encoded — edits must respect encoding. (ZAP shows only the final request; Burp keeps original + edit.) *(1 Q — gap.)* *Use: both.*

**8. Encoding/Decoding** — Values arrive encoded (Base64, URL, HTML, hex…): decode to read/change, then **re-encode with the same method** before sending, or the server misparses (raw spaces, `&`, `#` break requests). Burp: Repeater Convert Selection / Ctrl+U, Decoder tab, Inspector. ZAP: Encoder/Decoder/Hash dialog (Ctrl+E). Gotcha: ZAP auto-URL-encodes in the background — know what your tool already did. *(1 Q, needs an attached file not downloaded — gap.)* *Use: both.*

**9. Proxying Tools** — Route CLI tools and thick clients through the proxy so their traffic lands in your history — the way to *verify what an automated tool actually sends*. Point the tool at `http://127.0.0.1:8080` (its own proxy setting, Metasploit's PROXIES, or proxychains with an `http 127.0.0.1 8080` line for tools without one). Proxying slows tools down — do it while investigating, not for bulk runs. *(1 Q — gap.)* *Use: both.*

**10. Burp Intruder** — Burp's fuzzer: mark a payload position (§) in a request, choose payloads (Simple List, Runtime file for big lists, Character Substitution) + SecLists wordlists, add processing rules and Grep-Match/Grep-Extract, run, sort by status/length. **Community Intruder is throttled to ~1 req/s** — fine for short jobs and login spraying, hopeless for big wordlists; use Pro or a CLI fuzzer there. Confirm every hit manually in the browser before reporting. *(1 Q — gap.)* *Use: both.*

**11. ZAP Fuzzer** — The unthrottled free alternative: same workflow (Attack > Fuzz, mark location, payloads/processors, threads, depth- vs breadth-first), with built-in payload sets (incl. dirbuster lists, Numberzz) and processors (encode/decode, Base64, hashing, prefix/postfix). Read results by status, **body size and response time (RTT)** — size differences spot distinct pages, RTT spots time-based issues. *(1 Q — gap.)* *Use: both.*

**12. Burp Scanner** — Pro-only: crawl → passive analysis of captured traffic → active audit with a check preset; triage by **severity × confidence** on the Dashboard. Rules: define scope and exclude sensitive items *before* scanning; verify critical findings manually; and the professional line — **scanner output is supplementary data for the report you write, never a tool report handed to the client as the deliverable**. *Use: engagement.*

**13. ZAP Scanner** — The free equivalent: **Spider** (Ajax Spider for JS-heavy apps) builds the site map → passive alerts accumulate → **Active Scan** attacks pages/parameters → alerts grouped by risk with remediation. Don't active-scan before the tree is populated; don't report informational alerts as findings; verify high-risk alerts manually. *(1 Q — gap.)* *Use: both.*

**14. Extensions** — Community plug-ins add decoding/hashing/beautification/checks: Burp's **BApp Store** (some Pro-only; e.g. Decoder Improved), ZAP's **Marketplace** add-ons in Release/Beta/Alpha tiers (FuzzDB sets add fuzzing wordlists). Keep a short trusted list per tool; read docs first; test new/alpha add-ons on a throwaway target, never first-run one mid-assessment. *Use: both.*

**15. Skills Assessment (scenario)** — Four situations, each matched to a *different* proxy feature: enable a disabled control (response interception), decode a multiply-encoded cookie (Decoder), handle the short hash that falls out (cracking), and proxy an external tool (proxychains/Metasploit). The lesson: pick the feature that fits instead of forcing one workflow everywhere. *(4 Q — gap; scenario text only.)* *Use: engagement.*

## My worked notes (yours, as you do the labs)

- *(To fill in fresh — this module is not started on my account. It's short and foundational for the whole web block that follows: get Burp and ZAP both configured (listener + CA cert) and run §§4–11 hands-on before moving to Ffuf.)*
