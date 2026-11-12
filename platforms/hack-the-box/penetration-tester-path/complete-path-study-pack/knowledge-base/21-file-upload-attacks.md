# 21. File Upload Attacks

> **Status:** Deep-complete (lesson text) — read in full 2026-11-10, 11 / 11 sections
> **Outline metadata:** Medium, Tier 2, 11 sections, 8 interactive exercises, 1 assessment (as displayed), est. 8 hours
> **Coverage checklist:** concepts ☑ · workflow ☑ · tools/commands ☑ · pitfalls ☑ · fixes/defences ☑ · engagement use ☑ · CTF use ☑
> **Named gaps (see `../GAPS.md`):** 8 question entries deliberately not attempted (§2 ×1, §3 ×1, §4 ×1, §5 ×1, §6 ×1, §7 ×1, §8 ×2, §11 ×1 — the skills assessment) and no target spawned. Hint buttons were present in most exercise sections and deliberately not clicked.
> **Access note:** initially locked; free "Unlock Module" used (read access only, as disclosed in V9). No PRO feature clicked.
> **Account note:** this module was **not** already completed by me (reader showed 0%) — the notes run ahead of my hands-on here too.

Testing upload functions: what filtering is attempted, how it fails (extensions, content types, magic bytes, storage location), and what safe upload design looks like.

**Key concepts to own:** extension and MIME checks; double extensions / bypass classes; where uploads land; execution conditions; safe design.

## Deep summary — section by section (original words)

**1. Intro to File Upload Attacks** — An upload form that stores files where they can be requested is a code-placement primitive: worst case (unauthenticated, unvalidated) it's straight to a shell. Secondary payoffs even when code won't run: stored XSS/XXE, DoS, overwriting configs, abusing bundled vulnerable libraries. "File upload" is a *family* of bypass cases, not one bug — and an allowed file type never means "safe." *Use: both.*

**2. Absent Validation** — No checks at all: fingerprint the stack first (index extension, Wappalyzer, Intruder with an extensions wordlist) so your test script matches the server's language, upload a minimal echo test, request it under the uploads path, and **confirm execution** before escalating — stored-but-not-executed is a different (weaker) finding. *(1 Q — gap.)* *Use: both.*

**3. Upload Exploitation** — Two shell styles: a **web shell** (browser-driven; public ones like phpbash, SecLists' Web-Shells collection, or a tiny custom one passing a parameter to system()) vs a **reverse shell** (msfvenom php/reverse_php or the classic PHP reverse shell with IP/port edited in) — reverse is generally preferred: more interactive, more reliable. Set the listener before triggering; the classic failure is forgetting to set your own callback address/port in the file. *(1 Q — gap.)* *Use: both.*

**4. Client-Side Validation** — Checks that live only in the browser (JS handler, `accept` attribute) are advisory — the server never sees them enforced. Two bypasses: rewrite the upload in Burp (filename + content) after the browser approves it, or strip the handler/attribute in the Inspector and upload directly. If it still fails, a back-end check also exists — move down the ladder. *(1 Q — gap.)* *Use: both.*

**5. Blacklist Filters** — A list of *blocked* extensions is inherently incomplete and often case-sensitive. Fuzz the extension space (Burp Intruder + PayloadsAllTheThings/SecLists extension lists), sort responses by length to spot the distinct "uploaded" response — alternate executable extensions (e.g. .phtml) and mixed-case variants on case-insensitive filesystems are the usual survivors. Fix direction: whitelists, not longer blacklists. *(1 Q — gap.)* *Use: both.*

**6. Whitelist Filters** — Stronger, but weak implementations fail three ways: (1) an **unanchored regex** — the allowed extension just has to appear *somewhere*, so `shell.jpg.php` passes; (2) **server misconfiguration** — an Apache FilesMatch without an end anchor can treat `shell.php.jpg` as PHP; (3) **character injection** — `%20`, `%0a`, `%00` (PHP ≤5.x), slashes, ADS streams on Windows — truncating or reinterpreting the name. Method: double extensions first, then fuzz generated filename permutations in Intruder and confirm which actually execute. Fix: anchor the pattern to the end, reject odd characters, configure handlers precisely. *(1 Q — gap.)* *Use: both.*

**7. Type Filters** — Two content checks, both beatable: the **Content-Type header** is client-supplied — fuzz it (SecLists content-type list; mind that a multipart request carries *two* Content-Type headers — change the file's, not the request's); **magic bytes** — prepend a GIF signature (GIF8/GIF87a/GIF89a — printable ASCII) so `mime_content_type()` sees an image while the .php extension keeps it executable. Real defence is parsing/re-encoding the image, which destroys embedded code. Filters layer — combine bypasses. *(1 Q — gap.)* *Use: both.*

**8. Limited File Uploads** — When only "safe" types are allowed, the file *content* is still hostile: **HTML/SVG uploads** → stored XSS (SVG is XML — scripts and external entities ride along: XXE file reads incl. php://filter source theft, SSRF; the same XML trick lives inside Office/PDF files); **image metadata** (exiftool-injected comment/artist fields executing where displayed); **resource attacks** — decompression bombs, pixel-flood images claiming gigapixel dimensions; **traversal in the filename** itself. Defence: serve uploads from a separate domain with safe headers, strip metadata, disable external entities, cap size/dimensions. *(2 Q — gap.)* *Use: both.*

**9. Other Upload Attacks** — The **filename is an attack surface**: command injection if the app shells out to move it, XSS if it's reflected, SQLi if it's stored in a query. The uploads directory is often undisclosed — find it by fuzzing and by forcing verbose errors. Windows quirks: reserved names (CON, NUL, COM1…) and 8.3 short names (`WEB~1.CON` for web.conf) can overwrite or confuse. Rename server-side; never pass a filename to a shell or query unsafely. *Use: both.*

**10. Preventing File Upload Vulnerabilities** — Layered, back-end enforced: whitelist (end-anchored) **plus** dangerous-extension blacklist as a net; content validation (parse/re-encode images); **hide the uploads directory** and serve files through a controlled download script (authorization, path validation, 403 on direct access, Content-Disposition: attachment + nosniff); randomized stored names; separate server/container + open_basedir to cap blast radius; php.ini `disable_functions` (exec/system/passthru…); size limits; malware scanning; WAF as a secondary layer. This checklist doubles as the engagement report's action list. *Use: engagement.*

**11. Skills Assessment (scenario)** — An early-stage e-commerce app, any upload form in scope: probe which validation layers exist, combine bypasses, reach RCE — **and** produce the findings + mitigations write-up. The assessment grades the report habit, not just the shell. *(1 Q — gap; scenario text only.)* *Use: both.*

## My worked notes (yours, as you do the labs)

- *(To fill in fresh — this module is not started on my account. Work the bypass ladder in order (§§4–7) on the exercises — most real targets need two bypasses stacked, so practice combining them, and always confirm execution, not just acceptance.)*
