# 22. Command Injections

> **Status:** Deep-complete (lesson text) — read in full 2026-11-10, 12 / 12 sections
> **Outline metadata:** Medium, Tier 2, 12 sections, 9 interactive exercises, 1 assessment (as displayed), est. 6 hours
> **Coverage checklist:** concepts ☑ · workflow ☑ · tools/commands ☑ · pitfalls ☑ · fixes/defences ☑ · engagement use ☑ · CTF use ☑
> **Named gaps (see `../GAPS.md`):** 9 question entries deliberately not attempted (§2 ×1, §3 ×1, §4 ×1, §5 ×1, §6 ×1, §7 ×1, §8 ×1, §9 ×1, §12 ×1 — the skills assessment) and no target spawned. Hint buttons were present in most exercise sections and deliberately not clicked. The assessment prompt's lab login details were never copied — they're credential material.
> **Access note:** initially locked; free "Unlock Module" used (read access only, as disclosed in V9). No PRO feature clicked.
> **Account note:** this module was **not** already completed by me (reader showed 0%) — the notes run ahead of my hands-on here too.

Finding command injection where user input reaches a system shell, confirming it safely, and working around simple filters; plus how proper input handling prevents it.

**Key concepts to own:** injection points; blind command injection; filter bypass classes; output vs no-output confirmation.

## Deep summary — section by section (original words)

**1. Intro to Command Injections** — User input flowing into a back-end system command (PHP exec/system/shell_exec/passthru/popen; Node child_process) means the attacker appends their own command — full host compromise, potentially the network beyond it. OWASP ranks injection #3. It isn't web-only: local binaries and thick clients inject too. The tell is output that differs from the intended command's result. *Use: both.*

**2. Detection** — The operator set, and how each changes what runs and what's shown: `;` (both, in order), newline, `&` (background), `|` (pipe), `&&` (second only if first **succeeds**), `||` (second only if first **fails**), backticks / `$()` (sub-shell — output substituted into the first command). Platform note: `;` fails in Windows CMD but works in PowerShell. Method: baseline the normal output, then try operators one at a time and compare. A validation error is itself a signal — something noticed your input. *(1 Q — gap.)* *Use: both.*

**3. Injecting Commands** — Front-end-only validation again: the browser refuses the payload and *no request is even sent* (check the Network tab — that's the proof the check never reaches the server). Route through Burp/ZAP, send to Repeater, URL-encode (Ctrl+U), resend — both commands' output in one response. *(1 Q — gap.)* *Use: both.*

**4. Other Injection Operators** — `&&`/`||` semantics in practice: for an OR chain to show your output, the first command must be made to fail (e.g. empty input). Read missing output through exit codes (0 success / 1 failure) instead of guessing. Wider lesson: the same special characters mean different things across injection families (SQL, LDAP, XPath, traversal, headers…) — classify the injection before choosing characters. *(1 Q — gap.)* *Use: both.*

**5. Identifying Filters** — When a working payload suddenly draws a generic "invalid input" error, a blacklist or WAF fired. **Decompose**: resend the payload piece by piece — operator alone, space alone, command word alone — until the denial isolates exactly what's blocked. A block page naming your IP/request hints at a WAF rather than app code. The output of this section is a *map* of blocked characters/words — every later bypass is chosen from it. *(1 Q — gap.)* *Use: both.*

**6. Bypassing Space Filters** — Spaces are commonly blacklisted; three replacements: **tab** (`%09`), **`${IFS}`** (its default value is space+tab), and **brace expansion** (`{ls,-la}` expands to the spaced form). Use the newline operator (rarely blocked, works on Linux and Windows) as your injection operator. Try each in turn; remember to URL-encode the tab/newline. *(1 Q — gap.)* *Use: both.*

**7. Bypassing Other Blacklisted Characters** — Need a character you can't type (e.g. `/` for paths)? **Slice it out of an environment variable** by position — `$PATH`/`$HOME` contain slashes; other variables hold other characters (enumerate with printenv; Windows: CMD substring syntax or PowerShell indexing). Or **shift characters**: `tr` with a range one step off in the ASCII table generates the target character indirectly. Pitfall: offsets are system-specific — derive them from *this* target's printenv, never copy from another box. *(1 Q — gap.)* *Use: both.*

**8. Bypassing Blacklisted Commands** — The filter substring-matches command *words*; insert characters the shell discards: quotes (Linux + Windows — matching type, even count), backslash or `$@` (Linux), caret (Windows). Verify first that the word itself is what's blocked (§5's map) before obfuscating blindly. *(1 Q — gap.)* *Use: both.*

**9. Advanced Command Obfuscation** — Three escalations: **case** (a case-sensitive blacklist misses mixed case; Windows commands are case-insensitive anyway; on Linux lowercase programmatically with tr or parameter expansion), **reversal** (send the command backwards; `rev` or PowerShell index reversal restores it at execution), **encoding** (base64/hex hides *every* filtered character; decode in a sub-shell — `base64 -d`, openssl, xxd — using a here-string to avoid a possibly-blocked pipe). Discipline: verify the decoded result locally first, and keep earlier-filtered characters out of the wrapper too. *(1 Q — gap.)* *Use: both.*

**10. Evasion Tools** — When manual tricks run out: **Bashfuscator** (Linux — generate with a named technique/layer count, deterministic mode for repeatable tests) and **Invoke-DOSfuscation** (Windows — interactive; test its output with pwsh, which PwnBox ships). Always run generated output locally before sending it — never fire a payload you don't understand at a target. *Use: both.*

**11. Command Injection Prevention** — Layered: (1) **avoid system commands** — use a built-in/library function instead (a socket check instead of shelling to ping); (2) **validate format** server-side (FILTER_VALIDATE_IP, strict regex, a validation library); (3) **sanitize** to an allowlist of characters for that field; escaping alone (escapeshellcmd) is explicitly *not* sufficient. (4) Harden: WAF (mod_security), least-privilege web user, disable_functions, open_basedir, reject double-encoded/non-ASCII URLs, patch libraries. Verify the fix by attacking it with §§2–9. *Use: engagement.*

**12. Skills Assessment (scenario)** — A file-manager app met mid-pentest: authenticate, find the command execution feature, detect the injection, map the filters, stack the bypasses. The assessment is the whole module in one target — one bypass rarely suffices against layered filters. *(1 Q — gap; scenario text only; its lab login details were not copied.)* *Use: both.*

## My worked notes (yours, as you do the labs)

- *(To fill in fresh — this module is not started on my account. The exercises form a ladder (§§2–9 each add one filter and its bypass) — do them in order without skipping; the skill is the §5 filter-mapping step, everything after is choosing from the map.)*
