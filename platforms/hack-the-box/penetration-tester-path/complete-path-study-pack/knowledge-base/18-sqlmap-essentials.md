# 18. SQLMap Essentials

> **Status:** Deep-complete (lesson text) — read in full 2026-11-09, 11 / 11 sections
> **Outline metadata:** Easy, Tier 2, 11 sections, 7 interactive exercises, 1 assessment (as displayed), est. 4 hours
> **Coverage checklist:** concepts ☑ · workflow ☑ · tools/commands ☑ · pitfalls ☑ · fixes/defences ☑ · engagement use ☑ · CTF use ☑
> **Named gaps (see `../GAPS.md`):** 17 question entries deliberately not attempted (§1 ×1, §4 ×3, §6 ×3, §7 ×1, §8 ×2, §9 ×4, §10 ×2, §11 ×1 — the skills assessment) and no target spawned. Hint buttons were present in most exercise sections and deliberately not clicked. Example credential/token values in lesson outputs were never copied — the questions (mostly "contents of flag table X") are hands-on work.
> **Access note:** initially locked; free "Unlock Module" used (read access only, as disclosed in V9). No PRO feature clicked.
> **Account note:** this module was **not** already completed by me (reader showed 0%) — the notes run ahead of my hands-on here too.

Automating SQL injection detection and database enumeration with SQLMap once you understand the underlying technique, and controlling how aggressive the tool is.

**Key concepts to own:** pointing sqlmap at a request; level / risk; database and table enumeration; dumping safely in a lab; tamper concepts.

## Deep summary — section by section (original words)

**1. SQLMap Overview** — sqlmap automates the manual work of Module 17: probe parameters, identify technique + DBMS, extract data. Know the **technique letters (BEUSTQ)**: **B** boolean blind (slow — ~7–8 requests per character), **E** error-based (fast), **U** union (fastest — whole table in one request), **S** stacked queries, **T** time-based blind (slowest; when nothing else shows), **Q** inline queries; plus out-of-band DNS exfiltration as the fallback. Pick by speed/visibility, and verify findings manually — automation proposes, you confirm. *(1 Q — gap.)* *Use: both.*

**2. Getting Started with SQLMap** — `-h` basic help, `-hh` the full set (the wiki is the manual). First run: `sqlmap -u "<url>?id=1" --batch` — `--batch` accepts defaults so runs are repeatable; results and session data land in `~/.sqlmap/output/<host>/`. Remember the session file remembers earlier answers — a stale session can confuse a re-run. "All tested parameters do not appear to be injectable" means check the setup before concluding anything. *Use: both.*

**3. SQLMap Output Description** — Read the log as a story: stability checks → is the parameter dynamic → heuristic probes ("might be injectable" is a hint, **not proof**) → DBMS fingerprint → technique tests → "**parameter is vulnerable**" is the confirmation line. Traps: answering "skip other DBMSes" too early (some engines mimic others — Firebird types can look like MySQL), and reflective values creating false positives. Record the injection-point summary for the report. *Use: both.*

**4. Running SQLMap on an HTTP Request** — The make-or-break step: feed sqlmap the **exact** request. Fastest route: DevTools "copy as cURL", swap `curl` → `sqlmap`. GET via `-u`; POST via `--data`; target one parameter with `-p` or mark it with `*`; long/complex requests from a file with `-r` (Burp capture). Don't forget `--cookie`, custom headers (`-H`), `--random-agent` (the default sqlmap user-agent announces the tool), `--mobile`, `--method`; JSON/XML bodies are supported. Missing cookies/headers or testing the wrong parameter is why most "not injectable" results happen. *(3 Q — gap.)* *Use: both.*

**5. Handling SQLMap Errors** — Debug with evidence, not guesses: `--parse-errors` surfaces the DB's own error messages, `-t <file>` saves all traffic, `-v 6` prints full requests + errors, and `--proxy` routes through Burp so you can inspect and replay the failing request by hand. Keep the traffic log with the scan. *Use: both.*

**6. Attack Tuning** — Payload = **vector** (the SQL) wrapped in **prefix/suffix** (the boundary fitting the target's quoting); `--prefix`/`--suffix` override for odd quoting. `--level 1–5` / `--risk 1–3` widen the payload set (defaults ≈72 per parameter; max = thousands) — raise one step at a time, and know that **OR-based payloads need a risk raise because they can hit UPDATE/DELETE statements** — get approval before high-risk runs on production. Response matching: `--code`, `--titles`, `--string`, `--text-only`; restrict techniques with `--technique=BEU`; UNION tuning: `--union-cols` (exact count — determine it, don't guess), `--union-char`, `--union-from`. Watch payloads with `-v 3`. *(3 Q — gap.)* *Use: both.*

**7. Database Enumeration** — After detection: `--banner --current-user --current-db --is-dba --hostname --passwords`, then `--tables -D <db>`, `--dump -T <table>` (CSV into the output dir), `-C` columns, `--start/--stop` row range, `--where` row filter, `--dump-all`, `--exclude-sysdbs`. In-band returns data directly; blind extracts row by row — don't waste hours blind when in-band works. Dump only what scope requires; dumped data is credential-bearing evidence — protect it and report what was *reachable*, not just what you took. DB root ≠ OS root. *(1 Q — gap.)* *Use: both.*

**8. Advanced Database Enumeration** — `--schema` maps every table's structure; `--search` finds tables/columns by name pattern (hunt for user/pass-like names). sqlmap offers **automatic dictionary cracking** when dumps contain hashes (built-in ~1.4M wordlist, many hash types) — uncracked hashes go to a dedicated cracker (Module 10), not endless retries. `--all --batch` pulls everything accessible: powerful, potentially very long — think about time and scope first. *(2 Q — gap.)* *Use: both.*

**9. Bypassing Web Application Protections** — Match the bypass to the protection: `--csrf-token` (fresh token per request; auto-detects csrf/xsrf/token params), `--randomize` (unique values per request), `--eval` (compute a parameter in Python before each request). Identity/network: `--proxy` / `--proxy-file` (rotation), `--tor --check-tor`, `--random-agent`. WAF: sqlmap probes for one and identYwaf names it; `--skip-waf` skips the check. **Tamper scripts** (`--tamper`, `--list-tampers`) rewrite payloads — space2comment, randomcase, between, equaltolike, percentage — chainable, order matters. Also `--chunked` (split keywords across chunks) and HTTP parameter pollution. Document which protection fell and how; a bypass is itself a finding. *(4 Q — gap.)* *Use: both.*

**10. OS Exploitation** — From injection to host: check `--is-dba` first (non-DBA file reads fail *silently* — "no data retrieved"). `--file-read` pulls a file locally; `--file-write` + `--file-dest` uploads (e.g. a shell into the webroot — gated by secure_file_priv + OS write permission, same gates as Module 17 §15); `--os-shell` for an interactive shell — if the default technique returns nothing, force one (`--technique=E`) and answer the language prompt. Verify a written file is actually served before trusting it. Clean up every file/shell you create; host access via SQLi is a high-severity finding. *(2 Q — gap.)* *Use: both.*

**11. Skills Assessment (scenario)** — End-to-end against a protected app: build the exact request (§4) → debug (§5) → tune (§6) → enumerate (§§7–8) → bypass (§9) → OS if needed (§10). Skipping request setup and jumping to enumeration is the failure mode the assessment is built to catch. *(1 Q — gap; scenario text only.)* *Use: engagement.*

## My worked notes (yours, as you do the labs)

- *(To fill in fresh — this module is not started on my account. The exercises are a case series (Cases #1–#11 plus the final) — work them in order; each isolates one skill from §§4–10. Keep the request files and output dirs as your evidence trail.)*
