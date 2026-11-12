# 17. SQL Injection Fundamentals

> **Status:** Deep-complete (lesson text) — read in full 2026-11-09, 17 / 17 sections
> **Outline metadata:** Medium, Tier 0, 17 sections, 12 interactive exercises, 1 assessment (as displayed), est. 8 hours
> **Coverage checklist:** concepts ☑ · workflow ☑ · tools/commands ☑ · pitfalls ☑ · fixes/defences ☑ · engagement use ☑ · CTF use ☑
> **Named gaps (see `../GAPS.md`):** 14 question entries deliberately not attempted (§4–§7 ×1 each, §9–§15 ×1 each, §17 ×3 — the skills assessment) and no target spawned. Example credential/hash values shown in lesson outputs were never copied — the questions asking for them are hands-on work.
> **Access note:** initially locked; free "Unlock Module" used (read access only, as disclosed in V9). No PRO feature or Hint clicked.
> **Account note:** this module was **not** already completed by me (reader showed 0%) — the notes run ahead of my hands-on here too.

How SQL injection happens, how to detect it manually, and how to extract data through union, error, and blind techniques — understanding the queries, not just running a tool.

**Key concepts to own:** in-band / union injection; error-based; blind boolean and time-based; authentication bypass (concept); input handling fixes.

## Deep summary — section by section (original words)

**1. Introduction** — Apps assemble queries from code + user input; a quote can close the input's string early and everything after it is parsed as SQL. Injection impact ladder: data breach → login bypass → admin access → file read/write → backdoor → full takeover. Over-privileged DB accounts magnify every step. Focus here: relational (MySQL); NoSQL differs. *Use: both.*

**2. Intro to Databases** — A DBMS provides CRUD with concurrency control, consistency, security, backups, ACID transactions. Architecture: client apps → middleware (drivers/libraries) → DBMS; app and DB may share a host or split for scale. Vocabulary: database, table, row, column, cell, key. *Use: both (foundation).*

**3. Types of Databases** — Relational: fixed schema, tables joined by primary/foreign keys, SQL. NoSQL: flexible schema in four models — key-value (Redis), document (MongoDB), wide-column (Cassandra), graph (Neo4j). Don't treat NoSQL as SQL — its injection looks different (covered later in the path). *Use: both (foundation).*

**4. Intro to MySQL** — The `mysql` CLI: `-u` user, `-p` prompt for password (never type it inline — it lands in shell history), `-h` host, `-P` port (3306). Statements end with `;`, are case-insensitive (database names aren't). Core moves: SHOW DATABASES / CREATE DATABASE / USE / CREATE TABLE (INT, VARCHAR(n), DATETIME; AUTO_INCREMENT, NOT NULL, UNIQUE, PRIMARY KEY) / SHOW TABLES / DESCRIBE; SHOW GRANTS for privileges. Work as the least-privileged user that suffices — not root by default. *(1 Q — gap.)* *Use: both.*

**5. SQL Statements** — INSERT (single/multi-row, subset of columns), SELECT, UPDATE, DELETE, DROP (permanent, no confirmation), ALTER (add/rename/modify/drop columns). The discipline: **UPDATE/DELETE without WHERE hits every row**; constraints (NOT NULL, PK, UNIQUE) reject bad rows; back up before anything destructive. *(1 Q — gap.)* *Use: both.*

**6. Query Results** — ORDER BY (ASC default; multi-column = primary/secondary sort), LIMIT n and LIMIT offset,count (**offset starts at 0** — the classic off-by-one), WHERE (quote strings/dates, numbers bare), LIKE patterns (`%` any run, `_` exactly one char). These innocent clauses become payload machinery later. *(1 Q — gap.)* *Use: both.*

**7. SQL Operators** — AND/OR/NOT (`&&`/`||`/`!`); non-zero = true in MySQL. **Precedence: arithmetic → comparison → NOT → AND → OR** — AND outranks OR, which shapes every logic injection; use parentheses to force your grouping instead of hoping. *(1 Q — gap.)* *Use: both.*

**8. Intro to SQL Injections** — How the breakout works (input escapes its quotes, trailing query errors, comments hide the rest) and the taxonomy: **in-band** (union-based, error-based), **blind** (boolean-based, time-based), out-of-band. This module's focus: union-based. All user input is input — hidden fields included; a single sanitization step is not a defence. *Use: both.*

**9. Subverting Query Logic** — Login bypass: an unmatched quote breaks the query; an always-true condition makes WHERE true for every row, so the app logs in as the first account (usually admin). Placement respects precedence: with a known username attach the true-condition after AND; without one, inject the OR side in the password field too. Error silence ≠ not vulnerable — that's the blind case. Fix: parameterized queries, never string concatenation. *(1 Q — gap.)* *Use: both.*

**10. Using Comments** — Snip the query tail after your injection: `-- ` (needs the trailing space; `--+` in URLs), `#` (URL-encode as `%23` or it starts a fragment), `/* */` inline. Parentheses wrapping the conditions can break a naive payload — close the paren first, then comment the rest; the resulting syntax error pattern is itself a map of the query's shape. Defender's note: comment characters in input are a log red flag. *(1 Q — gap.)* *Use: both.*

**11. Union Clause** — UNION merges two SELECTs **only if both return the same column count** (else ERROR 1222 — which is itself useful feedback). Pad with NULL (fits any type) or literals. Defender's note: printing that error to users leaks the count. *(1 Q — gap.)* *Use: both.*

**12. Union Injection** — The workflow: (1) quote → error confirms injectability; (2) **count columns** with ORDER BY n until it errors (count = n−1), cross-check with UNION SELECT 1,2,3…; (3) **map which columns are visible** on the page with distinguishable values; (4) put the real payload (`@@version`, `user()`) in a visible slot. Injecting into a non-displayed column and concluding "doesn't work" is the classic false negative. *(1 Q — gap.)* *Use: both.*

**13. Database Enumeration** — Fingerprint properly (server banners are weak hints): `@@version`, `POW(1,1)`, `SLEEP(5)` when output is hidden. Then walk **information_schema**: SCHEMATA → TABLES → COLUMNS (skip the default MySQL schemas; filter — it's huge), `database()` for current context, dot notation to reach other databases, then UNION-select the target columns out. Defence: least privilege + restrict information_schema access + hide version/error detail. *(1 Q — gap.)* *Use: both.*

**14. Reading Files** — `LOAD_FILE()` through a visible UNION column reads server files **if** the DB user holds the FILE privilege and the OS user running MySQL can read the file (check via `USER()`/`CURRENT_USER()` and `information_schema.user_privileges`). Read configs and app source (view page source to see the PHP — browsers render it away). NULL result = missing privilege or OS permission, not a dead end for the injection itself. Defence: don't grant FILE casually; run the DB as a low-privilege OS user. *(1 Q — gap.)* *Use: both.*

**15. Writing Files** — `SELECT … INTO OUTFILE` writes content (a probe file first, then a shell into the webroot = code execution). Gates: FILE privilege + **`secure_file_priv`** (NULL = writes disabled; MySQL default points at its files dir; check `information_schema.global_variables`) + OS write permission. Find the webroot via server configs (LOAD_FILE), fuzzing, or errors. Remember junk UNION columns land in the file too — and **remove any shell you plant after the test**. Defence: lock secure_file_priv down, deny webroot writes, monitor for unexpected files. *(1 Q — gap.)* *Use: both.*

**16. Mitigating SQL Injection** — Layered defence, no single measure suffices: **parameterized queries** (placeholders + bind — data can never become code) as the primary fix; strict server-side validation (allowlist patterns); escaping (mysqli_real_escape_string) as a weaker layer; least-privilege DB accounts (a SELECT-only account on one table limits blast radius); WAF as monitoring/backup, never as the fix. Test your defences with the module's own techniques. *Use: engagement (developer-facing).*

**17. Skills Assessment (scenario)** — Black-box assessment of an HTTPS app with only an IP given: first make interception work — Burp's CA must be trusted (built-in Chromium, or install the PortSwigger CA in Firefox + FoxyProxy to 127.0.0.1:8080; PwnBox has it preinstalled) — then run the module workflow end to end (enumerate, inject, extract, escalate to RCE). Stay inside the contracted scope (SQLi) and report with §16 remediation. *(3 Q — gap; scenario text only; Hint not clicked.)* *Use: engagement.*

## My worked notes (yours, as you do the labs)

- *(To fill in fresh — this module is not started on my account. The MySQL sections (§§4–7) are quick wins; the real reps are §§9–15 — run the union workflow by hand until column-counting and visible-column mapping are reflexes before you let SQLMap do it in Module 18.)*
