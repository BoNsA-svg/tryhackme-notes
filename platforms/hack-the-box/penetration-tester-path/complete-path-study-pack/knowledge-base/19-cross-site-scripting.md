# 19. Cross-Site Scripting (XSS)

> **Status:** Deep-complete (lesson text) — read in full 2026-11-09/10, 10 / 10 sections
> **Outline metadata:** Easy, Tier 2, 10 sections, 7 interactive exercises, 1 assessment (as displayed), est. 6 hours
> **Coverage checklist:** concepts ☑ · workflow ☑ · tools/commands ☑ · pitfalls ☑ · fixes/defences ☑ · engagement use ☑ · CTF use ☑
> **Named gaps (see `../GAPS.md`):** 8 question entries deliberately not attempted (§2 ×1, §3 ×1, §4 ×1, §5 ×2, §7 ×1, §8 ×1, §10 ×1 — the skills assessment) and no target spawned. Hint buttons were present in most exercise sections and deliberately not clicked.
> **Access note:** initially locked; free "Unlock Module" used (read access only, as disclosed in V9). No PRO feature clicked.
> **Account note:** this module was **not** already completed by me (reader showed 0%) — the notes run ahead of my hands-on here too.

Finding and demonstrating reflected, stored, and DOM-based XSS, and understanding what session theft / account action impact it proves.

**Key concepts to own:** reflected vs stored vs DOM; payload placement; context and encoding; impact demonstration; output encoding fixes.

## Deep summary — section by section (original words)

**1. Intro to XSS** — XSS = unsanitized user input rendered as JavaScript in other users' browsers. Purely client-side (the server isn't directly harmed), but so common it's rated medium risk — low impact × high probability. Three types: **stored, reflected, DOM**. Real-world weight: the Samy worm (stored, MySpace), the 2014 TweetDeck self-retweeting tweet, an Apache XSS used to steal passwords. *Use: both.*

**2. Stored XSS** — The most critical type: the payload is **saved in the database** and executes for every visitor until someone cleans the DB. Test loop: submit benign input → check rendering → submit test payload → confirm execution → refresh to confirm persistence. Confirm via page source (Ctrl+U) that the payload sits unsanitized. No pop-up ≠ not vulnerable — some browsers block alert in some contexts; fall back to other proof payloads and check the source. *(1 Q — gap.)* *Use: both.*

**3. Reflected XSS** — Input is echoed back unfiltered (classically in error/confirmation messages) but **not stored** — it fires only when a victim loads the crafted URL. Workflow: find an echoed input → confirm the payload is in the response source → check the method in DevTools (GET = the payload travels in a shareable link; POST can't just be sent as a URL). The victim must be lured — that's the delivery problem. *(1 Q — gap.)* *Use: both.*

**4. DOM XSS** — Fully client-side: page JavaScript takes a **source** (e.g. a URL parameter, often after `#`) and writes it into a **sink** (document.write, innerHTML, outerHTML, jQuery add/append/after…) without sanitizing. Tells: no HTTP request in the Network tab; the input is absent from the raw page source (Ctrl+U) but present in the **rendered DOM** (Inspector). Payload mechanics matter: `<script>` doesn't execute inside innerHTML — use an event-handler payload (e.g. an image tag with onerror) instead. *(1 Q — gap.)* *Use: both.*

**5. XSS Discovery** — Three routes: scanners (Nessus, Burp Pro, ZAP; open-source XSStrike, XSSer — passive scans review code for DOM sinks, active scans fire payloads), manual payload lists (PayloadAllTheThings, Payload-Box — any page input can be a vector, not just form fields), and **code review (source → sink tracing) — the most reliable**. Universal rule: **a payload visible in source is not proof — only execution is.** Verify every tool finding manually. *(2 Q — gap.)* *Use: both.*

**6. Defacing** — Stored XSS weaponized cosmetically: rewrite background, title, and body text so every visitor sees the attacker's page (the 2018 NHS defacement is the cited case — reputational and even market impact). Build it incrementally — add one element at a time and test; minify big payloads. Remediation means cleaning the stored payload from the DB, not just patching the input. *Use: both.*

**7. Phishing** — Inject a fake login form into a page victims already trust; credentials post to the attacker's server → account takeover. Craft matters: pick a payload that actually works in that page's context, strip leftover original elements (remove by id, comment out the rest) so nothing betrays the fake, and use a **PHP logging server, not bare netcat** — netcat mangles the HTTP exchange and the victim sees an error. Legitimate use: authorized phishing simulations to measure awareness; report the root cause (missing output encoding), not just the phish. *(1 Q — gap.)* *Use: both.*

**8. Session Hijacking** — Steal the session cookie, replay it (DevTools Storage → add cookie → refresh), you're the victim — no password needed. **Blind XSS** is the hard variant: the payload fires on a page you can't see (admin panels, contact/review/ticket forms, even the User-Agent header). Find it by remote-script beaconing: inject a script tag pointing at your server with a per-field URL — the request that arrives names the vulnerable field. Exfiltrate quietly (an Image().src beacon beats a visible document.location redirect). Format rules still apply per field (an email field must look like an email). Defence: **HttpOnly** (JS can't read the cookie) + Secure flags + server-side session validation; treat blind XSS as high severity. *(1 Q — gap.)* *Use: both.*

**9. XSS Prevention** — Defence in depth across three layers. **Front-end:** validate input, sanitize with DOMPurify, avoid dangerous sinks (innerHTML, document.write, the jQuery HTML-insertion family), never drop raw input into script/style/attribute/comment contexts. **Back-end (the vital layer — front-end checks are bypassed by sending requests directly):** validate (filter_var), sanitize, and **encode output** (htmlspecialchars/htmlentities) so input renders as text. **Server config:** HTTPS everywhere, Content-Security-Policy (script-src 'self'), X-Content-Type-Options: nosniff, HttpOnly + Secure cookies, WAF as a backstop. *Use: engagement (developer-facing).*

**10. Skills Assessment (scenario)** — A blog app: find the input field, get a payload that *executes* (not just reflects), then session-hijack the reviewing admin's cookie. The trap is stopping at a payload that appears in source without executing. *(1 Q — gap; scenario text only.)* *Use: engagement.*

## My worked notes (yours, as you do the labs)

- *(To fill in fresh — this module is not started on my account. Practice the three types separately (§§2–4) until you can state source, sink, and delivery for each without thinking; §§7–8 are the impact demos worth doing carefully.)*
