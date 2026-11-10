# 15. Attacking Web Applications with Ffuf

> **Status:** Deep-complete (lesson text) — read in full 2026-11-09, 13 / 13 sections
> **Outline metadata:** Easy, Tier 0, 13 sections, 8 interactive exercises, 1 assessment (as displayed), est. 5 hours
> **Coverage checklist:** concepts ☑ · workflow ☑ · tools/commands ☑ · pitfalls ☑ · fixes/defences ☑ · engagement use ☑ · CTF use ☑
> **Named gaps (see `../GAPS.md`):** 12 question entries deliberately not attempted (§3 ×1, §4 ×1, §5 ×1, §7 ×1, §9 ×1, §10 ×1, §12 ×1, §13 ×5 — the skills assessment) and no target spawned. Display note: §2's text refers to "the exercise in the question below" but no question box was displayed in that section in either view — recorded as displayed (0 questions), not assumed.
> **Access note:** initially locked; free "Unlock Module" used (read access only, as disclosed in V9). No PRO feature clicked.
> **Account note:** this module was **not** already completed by me (reader showed 0%) — the notes run ahead of my hands-on here too.

Fuzzing web applications with Ffuf to find hidden directories, files, parameters, and virtual hosts that are not linked anywhere.

**Key concepts to own:** content discovery; parameter fuzzing; vhost fuzzing; filtering by size / words / status; wordlist choice.

## Deep summary — section by section (original words)

**1. Introduction** — The module teaches one tool deeply: **ffuf**, among the most common and reliable web fuzzers. Fuzzing output is enumeration, not a result — a 200 on a fuzzed path means "this exists; go examine it manually." *Use: both.*

**2. Web Fuzzing** — Fuzzing = sending many inputs from a prepared wordlist and studying which responses differ from the baseline (e.g. 200 on a real path vs 404 on an invented one). Wordlist craft is half the skill: match the list to the target type (SecLists directory-list-2.3 in several sizes), weigh size against runtime, and use `-ic` to skip wordlist comment lines so they don't pollute results. Establish the baseline response first. *Use: both.*

**3. Directory Fuzzing** — The core pattern: `ffuf -w <wordlist>:FUZZ -u http://<target>/FUZZ` — the FUZZ keyword marks where each word lands. Speed is extreme (~90k URLs in seconds in the example), which cuts both ways: **keep thread counts (`-t`) reasonable on remote targets — you can DoS the service or break your connection.** Verify hits by visiting them; an existing directory may serve an empty page — empty ≠ missing. *(1 Q — gap.)* *Use: both.*

**4. Page Fuzzing** — Directories hold pages with extensions, and the **extension fingerprints the technology**. Don't trust server headers — fuzz it: `.FUZZ` on a known file (index.*) with a common-extensions list, then fuzz filenames with the found extension (two keywords: `FUZZ_1.FUZZ_2`). Watch the mechanics: the extensions list already contains the dot, so adding one breaks the pattern; and one extension returning 200 doesn't prove others don't exist. *(1 Q — gap.)* *Use: both.*

**5. Recursive Fuzzing** — `-recursion -recursion-depth 1` re-scans under every discovered directory automatically; add `-e .php` for the known extension and `-v` so full URLs make the tree readable (without it you can't tell which page lives under which directory; note directories also appear extensionless, doubling output). The request count explodes with depth — **cap the depth**, then hand-pick interesting branches for directed scans. *(1 Q — gap.)* *Use: both.*

**6. DNS Records** — Why a lab name won't resolve: browsers check `/etc/hosts` first, then public DNS; a name in neither is unreachable *by name* even though the site answers by IP. Fix: add the mapping (`echo "<IP> <name>" >> /etc/hosts`) — and don't forget the port when you switch from IP to name. Record every hosts entry you add during an engagement and clean up after. Also: a recursive scan finding nothing may mean the content lives on a *subdomain*, not that nothing exists. *Use: both.*

**7. Sub-domain Fuzzing** — Fuzz `FUZZ.<domain>` against public DNS to find subdomains that resolve to a working IP. Wordlist size is a choice (the 5k top-1-million list for a quick pass, bigger for coverage). Interpret zero hits correctly: on a lab name with no public records, zero hits means "none public" — it says nothing about hidden hosts (that's §8). *(1 Q — gap.)* *Use: both.*

**8. Vhost Fuzzing** — One IP can serve many sites (**virtual hosts**), with or without DNS records. Fuzz the **Host header**: `ffuf -H 'Host: FUZZ.<domain>'` against the IP — no hosts entry needed. Every word returns 200 (the URL doesn't change), so **the signal is response size**: a different size = a different vhost serving different content. Judging by status code alone concludes "nothing here" wrongly. A found vhost may still need a hosts entry to browse normally. *Use: both.*

**9. Filtering Results** — Defaults hide 404s; the noise that remains is same-y 200s. Measure the **default response size**, then filter it out (`-fs <size>`; siblings: matchers `-mc -ml -mr -ms -mw`, filters `-fc -fl -fr -fs`) so only the unusual responses stay. Filter on a *measured* value, never a guess — and know the risk: size filtering also hides any real hit that happens to share that size. Re-verify survivors by visiting them (incl. cross-checking: a page on one vhost should 404 on another). *(1 Q — gap.)* *Use: both.*

**10. Parameter Fuzzing — GET** — Pages that gate content behind a parameter (`?key=value`) often use **unpublished** parameters — less tested, prime hunting. Fuzz the parameter-name position (`?FUZZ=value`) with a parameter-names list (burp-parameter-names), filter the default size, then visit with the found parameter and test it (injection, logic). A found parameter may be deprecated — check it actually does something. *(1 Q — gap.)* *Use: both.*

**11. Parameter Fuzzing — POST** — POST parameters live in the body: `ffuf -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded'` (the content type matters — wrong type, misleading results). Verify hits with curl. A parameter that *rejects* your value ("invalid") still **exists** — that's a confirmed finding and the doorway to value fuzzing. *Use: both.*

**12. Value Fuzzing** — Once a parameter exists and validates its value, brute the value: build a custom wordlist (e.g. sequential IDs with a shell loop, `seq 1 1000`), fuzz the value position, submit the winner (curl). Start with a plausible range and widen; don't assume values are sequential — they may be random or hashed, in which case this approach fails honestly. Record the wordlist used. *(1 Q — gap.)* *Use: both.*

**13. Skills Assessment (scenario)** — From only an IP: vhost/subdomain fuzzing → **extension fuzzing before page fuzzing** → find the access-gated page → fuzz its parameters → fuzz values → retrieve the data. The questions must be answered *in dependency order* — each list (vhosts, extensions, parameters) feeds the next step; of several parameters found, only one returns data. Keep every intermediate list for the report. *(5 Q — gap; scenario text only.)* *Use: engagement.*

## My worked notes (yours, as you do the labs)

- *(To fill in fresh — this module is not started on my account. It's a single-tool module: work §§3–5 and §§7–12 hands-on until the FUZZ-position + filter reflex is automatic, then run the §13 chain end to end.)*
