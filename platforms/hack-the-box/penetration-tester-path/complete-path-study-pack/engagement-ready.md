# Engagement Ready
Use this part when the target is a client / lab you are formally authorized to test. The difference from a CTF is scope, evidence, and reporting — not just getting a shell.

## A1. Before you touch anything


☐  Written authorization and scope confirmed: which IPs / domains / apps are in, which are out.

☐  Rules of engagement agreed: testing window, rate limits, out-of-scope techniques, emergency contact / stop procedure.

☐  Objectives agreed: what counts as success (access, data, a specific system), and what must NOT be disrupted.

☐  Your notes folder is set up before starting: one folder per host, timestamps on, screenshots named consistently.

## A2. The engagement workflow

1. Pre-engagement — scope, rules, objectives, reporting format agreed (see A1).
1. Recon / information gathering — passive first, then active. Record sources for every fact.
1. Enumeration — hosts, ports, services, versions, users, shares, web content. Enumeration drives everything after it.
1. Vulnerability assessment — separate 'interesting' from 'actually exploitable in scope'. Note false positives.
1. Exploitation — the minimum action that proves impact. Prefer proof over damage; avoid persistence / destructive payloads unless explicitly agreed.
1. Post-exploitation (only if in scope) — what the access actually reaches: data, other users, other hosts. Document impact, not trophies.
1. Pivoting / lateral movement (only if in scope) — treat every new network segment as a fresh enumeration problem.
1. Documentation & reporting — findings written so a non-hacker can understand impact and a sysadmin can reproduce and fix them.
1. Clean-up & debrief — remove anything you placed on target systems if required, hand over evidence, walk the client through fixes.

## A3. Finding note template (copy per finding)

- Title / host / service / date-time found:
- What it is, in plain language:
- Evidence (command summary in your own words, screenshot filename, output excerpt you generated):
- Impact — what an attacker gains from this, specifically:
- Severity reasoning (not just a scanner score):
- Fix / recommendation:
- Retest result:

## A4. Hard lines

- No authorization = no testing. A similar-looking IP is not in scope until confirmed.
- Stop and contact the client if you hit real personal data, a system outside scope, or signs someone else is already inside.
- Never leave a test account, shell, or file behind without recording it for clean-up.
