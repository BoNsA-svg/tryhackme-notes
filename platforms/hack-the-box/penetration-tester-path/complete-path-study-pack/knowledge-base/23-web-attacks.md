# 23. Web Attacks

> **Status:** Deep-complete (lesson text) — read in full 2026-11-10, 18 / 18 sections
> **Outline metadata:** Medium, Tier 2, 18 sections, 10 interactive exercises, 1 assessment (as displayed), est. 2 days
> **Coverage checklist:** concepts ☑ · workflow ☑ · tools/commands ☑ · pitfalls ☑ · fixes/defences ☑ · engagement use ☑ · CTF use ☑
> **Named gaps (see `../GAPS.md`):** 10 question entries deliberately not attempted (§3 ×1, §4 ×1, §8 ×1, §9 ×1, §10 ×1, §11 ×1, §14 ×1, §15 ×1, §16 ×1, §18 ×1 — the skills assessment) and no target spawned. Hint buttons were present in §§3, 4, 8, 9, 10, 11, 14, 15 and deliberately not clicked. The assessment prompt's lab login details were never copied — they're credential material.
> **Access note:** initially locked; free "Unlock Module" used (read access only, as disclosed in V9). No PRO feature clicked.
> **Account note:** this module was **not** already completed by me (reader showed 0%) — the notes run ahead of my hands-on here too.

Three common web flaws in one module: HTTP verb tampering, IDOR (insecure direct object references / broken access control), and XXE (XML external entity) — how to spot, prove, and explain each.

**Key concepts to own:** verb tampering; IDOR testing pattern; XXE basics; access-control testing mindset.

## Deep summary — section by section (original words)

### HTTP Verb Tampering (§§1–5)

**1. Introduction to Web Attacks** — The module's three classes: verb tampering, IDOR, XXE. Web apps (even internal or API-only ones) are a primary route into an organization, so these "web" flaws are network-compromise findings, not a separate discipline. *Use: both.*

**2. Intro to HTTP Verb Tampering** — Apps are designed around GET/POST, but clients can send any of the nine verbs. If auth or filters are scoped to specific methods, an uncovered verb (HEAD, PUT, DELETE…) reaches the function unchecked. Two flavours: **config** (server auth rule lists verbs) and **coding** (a filter validates one method's parameters while the sink reads a combined source that also takes the other method). Enumerate accepted verbs with OPTIONS, then map what each function/filter actually covers. *Use: both.*

**3. Bypassing Basic Authentication** — Config flavour in practice: Basic Auth covers GET/POST only; **HEAD** behaves like GET but returns no body — and still executes the action (e.g. a reset) without a credential prompt. A 401 on one method proves nothing about the others. *(1 Q — gap.)* *Use: both.*

**4. Bypassing Security Filters** — Coding flavour: the filter inspects only one method's parameters, so changing the request method in Burp walks a blocked filename (carrying a command injection) straight past it. Lesson: test every function with **every method the server accepts**, not just the one the UI uses. *(1 Q — gap.)* *Use: both.*

**5. Verb Tampering Prevention** — Config: use the "except" forms (Apache LimitExcept, Tomcat http-method-omission) so auth covers all verbs except the intended ones; disable HEAD unless needed. Code: one consistent parameter source per function, and security checks that cover **all** request sources. A config fix doesn't cure a coding flaw and vice versa. *Use: engagement.*

### IDOR (§§6–12)

**6. Intro to IDOR** — A direct object reference (file, record, account) the user can change, where the server **doesn't verify ownership**. The reference isn't the bug — the missing back-end access check is. UI hiding is not access control. Impact runs disclosure → modification → privilege escalation; hard to detect automatically, so it ships to production often. *Use: both.*

**7. Identifying IDORs** — References hide in URL parameters, headers, cookies, and front-end JS — including admin calls merely *disabled* for lower roles, which can be invoked directly. Encoded references decode/re-encode; hashed references may still be computable if the front-end shows the scheme. Method: register **two accounts** and replay one user's calls as the other — the cross-account comparison is the proof. *Use: both.*

**8. Mass IDOR Enumeration** — Predictable ids (documents named by user id + date) invite bulk theft: test one id change, find the stable link pattern, then automate (Intruder/ZAP Fuzzer, or a fetch–regex–loop bash script). Watch for a hidden filter parameter that actually gates the data, and don't trust the first page view — links can differ when the visible list doesn't. *(1 Q — gap.)* *Use: both.*

**9. Bypassing Encoded References** — A hash isn't reversible, but it doesn't need to be: the front-end JS revealed the scheme (hash of the base64-encoded user id), so references for *any* user are reproducible and enumerable exactly like clear-text ids. Rule: **never compute security references on the front end**; a reference with no computable scheme (random, server-issued) is what actually resists this. *(1 Q — gap.)* *Use: both.*

**10. IDOR in Insecure APIs** — Function-level IDOR: act as other users, not just read. The exercise API's only real authorization was a **client-side role cookie**; its user-id/identifier matching checks rejected the direct attacks — but the read path (GET another user's details) leaked the identifier those attacks needed. Lesson: when the write path resists, pivot to the read path; partial checks create a false sense of security. *(1 Q — gap.)* *Use: both.*

**11. Chaining IDOR Vulnerabilities** — The chain: read another user (leaks identifier) → PUT changes with that identifier (email change → password reset to your address; or stored script in a profile field) → enumerate users, find the admin → set your own role to the admin role name (accepted: the value was valid and unchecked) → create/delete users. Client-controlled role values are the recurring root cause. *(1 Q — gap.)* *Use: both.*

**12. IDOR Prevention** — Server-side **RBAC keyed to the session token**: every object read/write checked against the requester's id and role map from the back end — never from a cookie or client field. Strong references (UUID v4 mapped server-side) raise the bar but are **not a substitute** for access control (and they make testing harder, which cuts both ways). Build access control into the core design, not as an afterthought. *Use: engagement.*

### XXE (§§13–17)

**13. Intro to XXE** — XML parsers resolve DTD-defined entities; an **external entity** (SYSTEM + file path/URL) makes the parser fetch server-side content when user-supplied XML is parsed (forms, SOAP APIs). Damage: file disclosure up to DoS; OWASP Top 10. XML is data markup, not display — don't let "we mostly use JSON" end the hunt; one XML endpoint is enough. *Use: both.*

**14. Local File Disclosure** — Test with an internal entity in a reflected element, then a SYSTEM entity for local files. Source code needs a **php://filter base64 wrapper** — raw PHP breaks the XML. Escalations exist but are conditional: shell via web-root write or the rarely-enabled expect filter; billion-laughs DoS (mostly mitigated in modern parsers). File disclosure ≠ code execution — report them separately. *(1 Q — gap.)* *Use: both.*

**15. Advanced File Disclosure** — Two refinements: **CDATA wrapping** for content that would break XML — assembled with **parameter entities** (DTD-only) around the file content, DTD hosted on your server; and **error-based** exfiltration — a deliberately undefined entity makes the parser's error message carry the file content when nothing is reflected. Error-based is unreliable for long/special-character files. *(1 Q — gap.)* *Use: both.*

**16. Blind Data Exfiltration** — Fully blind (no output, no errors): make the parser **call out** — the hosted DTD builds a URL on your listener with the base64-encoded file as a parameter (HTTP OOB) or as a subdomain (DNS OOB, captured with tcpdump). XXEinjector automates it from a saved request; remember it logs data base64-encoded rather than printing it. Needs a reachable listener and outbound access from the target. *(1 Q — gap.)* *Use: both.*

**17. XXE Prevention** — XXE is usually an **outdated library** problem, not a code problem: keep XML components current first. Then configure parsers safely: disable DTDs, external entities, parameter entities, XInclude, entity loops; hide runtime errors (starves error-based exfiltration). Prefer JSON APIs over XML/SOAP where possible; WAF is a layer, never the fix. *Use: engagement.*

**18. Skills Assessment (scenario)** — A social-network app combining all three classes: verb tampering, IDOR, and XXE chained into privilege escalation. The module's point lands here — the classes are rarely isolated in real apps; test each, then chain. *(1 Q — gap; scenario text only; its lab login details were not copied.)* *Use: both.*

## My worked notes (yours, as you do the labs)

- *(To fill in fresh — this module is not started on my account. Do the IDOR sections with two self-registered accounts ready (§7's method), and keep a listener you control running for §§15–16 — the XXE exfiltration exercises all depend on hosting a DTD.)*
