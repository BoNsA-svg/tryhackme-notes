# 1. Penetration Testing Process

> **Status:** Deep-complete (lesson text) — read in full 2026-11-09, 15 / 15 sections
> **Outline metadata:** Fundamental, 15 sections, 16 questions
> **Coverage checklist:** concepts ☑ · workflow ☑ · tools/commands ☑ · pitfalls ☑ · fixes/defences ☑ · engagement use ☑ · CTF use ☑
> **Named gaps (see `../GAPS.md`):** assessment questions deliberately not opened/submitted (at least 4 seen, in sections 7, 9, 11); my own lab/worked notes are blank — I completed this module ~4 weeks before this note system existed.
> **Account note:** HTB shows this module as already completed by me (Tactical badge, ~4 weeks ago) while the path overview shows 0% — both as displayed.

The stages of a professional test from first client contact to final report, and what a tester is responsible for at each stage, including the contracting and pre-engagement decisions that make testing legal and useful.

**Key concepts to own:** engagement stages; rules of engagement; scope; reporting purpose.

## Deep summary — section by section (original words)

### 1. Introduction to the Penetration Tester Path
- The path is a staged route to a Junior Penetration Tester role in four tiers (Fundamental, Easy, Medium, Hard) that build on each other; later modules assume earlier ones.
- Order of learning: background → each engagement phase → exploitation, privesc and pivoting → reporting → whole-network practice labs and an exam-style assessment.
- Decision: finish the path as a generalist foundation, or branch into a specialist track afterwards.
- Pitfall: rushing it as a checklist and skipping fundamentals, then hitting modules you lack prerequisites for.
- Use: engagement, with lab/CTF practice as preparation.
- *Site discrepancy noted: this section's own table breaks the path into 36 modules; the path preview shows 28. Unresolved — see GAPS.md.*

### 2. Academy Modules Layout
- Each module is self-contained: sections are theory (reading only) or interactive (reading + lab tasks). A lab belongs to its module; progress in one module doesn't depend on another.
- Lab sessions are time-limited and reset when the section changes — plan work inside the session (typically ~2 hours; challenge sessions shorter). Save/download anything you need before it ends.
- Lab types: Linux shell, desktop (RDP), challenge, AttackBox (SSH / noVNC). Don't confuse them.
- Some interactive tasks require independent research beyond the section text — that's expected, not a gap in the lesson.
- Question tiers (Easy/Medium/Hard) signal how much automation vs custom method is expected.
- Pitfall: assuming labs persist across sections, or that your notes survive a lab reset.
- Use: engagement and CTF/lab practice.

### 3. Academy Exercises & Questions
- Exercises exist to reinforce a concept, not as a CTF points contest. Questions after a section are usually flags produced by applying what the section taught — often by running the tool it covered on a lab target (an IP).
- Workflow: read theory → identify the tool/method → apply it on the target → capture the flag. Later sections can reuse tools from earlier ones, and a question may need flags from several earlier tasks.
- Success = producing the flag by applying the method. Failure = guessing or looking up an answer without understanding the technique.
- Use: engagement and CTF.

### 4. Penetration Testing Overview
- Pentesting = authorized, adversarial simulation. Distinguish: a vulnerability **scan** finds possible issues; an **assessment** identifies and classifies them; a **penetration test** exploits them to show real impact — across technology, people, and process.
- Decisions: engagement type (external, internal, web app, network, wireless, social engineering, physical) and knowledge level (Black / White / Gray Box), matched to the client's goal.
- Methodologies/standards: PTES, OSSTMM, OWASP Testing Guide, NIST SP 800-115, ISSAF.
- Success = realistic demonstration of exploitability tied to business impact. Failure = a scan report with no exploitation and no proof of what matters.
- Pitfalls: confusing scanning with testing; no agreed objective; hunting one bug instead of measuring overall posture.
- Use: engagement (primarily).

### 5. Laws and Regulations
- Testing without authorization is illegal. Key frameworks named: US CFAA, UK Computer Misuse Act, EU rules including GDPR, US CISA — plus local law wherever the test touches.
- Authorization must be written, in scope, and documented; exceeding it can be a criminal offence. This is why contracts, NDAs and Rules of Engagement exist.
- Workflow: written authorization + contract → confirm jurisdiction and in/out scope → define data handling → confirm who must be notified before starting.
- Pitfalls: assuming authorization in one country covers another; failing to protect sensitive data found during a test.
- Practice: authorization in writing first, know the relevant jurisdictions, professional code of conduct, treat everything discovered as confidential.
- Use: engagement only.

### 6. Penetration Testing Process
- A repeatable five-phase process: pre-engagement → information gathering → vulnerability assessment → exploitation → post-exploitation + reporting, with proof-of-concept and post-engagement closeout part of it.
- The phases **loop**: new information found later can send you back to an earlier phase. It is not a one-way checklist.
- Pitfalls: exploiting before you understand what you found; poor notes that leave findings unprovable.
- Practice: follow a recognized methodology (see §4), keep organized notes and an evidence record at every phase.
- Use: engagement (primarily).

### 7. Pre-Engagement
- Before any testing the engagement is defined: scope, targets, rules, contacts, legal basis, and the deliverable.
- Contents: scoping checklist, Rules of Engagement, contract, NDA/PSA, IP ranges and domain lists, excluded systems and dates, time windows, sensitive-data handling, contact and **emergency contact** lists, communication channels, retest windows, network diagrams, required credentials.
- Success = a signed, detailed plan everyone agrees on. Failure = vague scope, legal risk, undefined target.
- Pitfalls: forgetting emergency contacts; missing retest terms; no written authorization.
- Use: engagement only.
- *Gap: this section ends with an assessment question (Question 1, +20 cubes) behind a "Go to Questions" control — not opened or submitted, by rule.*

### 8. Information Gathering
- **Passive** gathering first (no contact with the target): domains, people, technologies, DNS records, public sources — Whois/registries, search engines, OSINT. **Active** gathering second (direct probing): hosts, ports, services, versions — noisier, detectable if careless.
- Workflow: confirm scope → passive research → active enumeration (network and web).
- Pitfalls: probing out-of-scope systems; confusing recon with a vulnerability scan; trusting automated output only and missing shadow IT / third-party assets.
- Practice: use multiple sources, validate findings, organize results before assessment.
- Use: engagement and CTF.

### 9. Vulnerability Assessment
- Systematic identification and **classification** of vulnerabilities — distinct from scanning (possible issues) and pentesting (exploiting them). Risk is reasoned as impact × likelihood.
- Workflow: gather data (scans, manual review, version/config checks) → confirm which findings are real → classify (e.g. Access Control, Information Exposure) → score (CVSS, with CVE/CWE) → prioritize remediation by business impact.
- Tools: Nmap, Nessus/OpenVAS, Nuclei and similar scanners.
- Success = a verified, prioritized list where severity reflects realistic impact. Failure = a raw scanner dump that treats every score as truth.
- Pitfalls: scan result ≠ finding; ignoring false positives/negatives; missing config and logic issues scanners can't see.
- Use: engagement and CTF.
- *Gap: ends with an assessment question (Question 1, +20 cubes) — not opened or submitted, by rule.*

### 10. Exploitation
- Using a **confirmed** vulnerability to gain access, methodically: RCE, injection, file inclusion, credential attacks, and related classes — not random attempts.
- Workflow: confirm manually → choose the least destructive approach that proves impact → run exploit/payload → take access → move to privesc/persistence.
- Tools: Metasploit + msfvenom; shells via nc / socat / bash; Burp Suite, SQLMap; Hydra for credential attacks.
- Success = controlled, in-scope access with evidence. Failure = untested/destructive exploits, out-of-scope attacks, downtime on production.
- Pitfalls: not understanding your payload; mishandled/timing-out shell sessions; careless credential reuse.
- Practice: test payloads in a lab first, snapshot/backup where possible, log everything for the report.
- Use: engagement and CTF.

### 11. Post-Exploitation
- Everything after the first foothold: what the access actually reaches — credentials, sensitive data, and movement toward a business objective, within scope.
- Workflow on the host: enumerate → escalate privileges → persistence + credential gathering → collect data → pivot.
- Tools: LinPEAS / WinPEAS, Mimikatz-class credential tools, CrackMapExec/NetExec, Meterpreter post-exploitation (hashdump, keylogging, credential reuse).
- Pitfalls: destructive privesc or broken services losing the foothold; unsecured persistence; forgetting clean-up after pivoting; reusing credentials out of scope.
- Practice: enumerate before escalating, document every step, keep access controlled and removable.
- Use: engagement and CTF.
- *Gap: ends with two assessment questions (+20 cubes each) — not opened or submitted, by rule.*

### 12. Lateral Movement
- Moving system-to-system inside a network using stolen credentials, trust relationships, and the normal admin/auth protocols that tie systems together — abusing legitimate mechanisms rather than always finding a new vulnerability.
- Workflow: enumerate trust and admin access → reuse credentials (pass-the-hash / pass-the-ticket concepts) → cross via SMB, RDP, WinRM, SSH, WMI.
- Tools: PsExec, WinRM, CrackMapExec/NetExec, WMI, Impacket, BloodHound (AD path mapping), Responder (AD contexts).
- Pitfalls: account lockouts from aggressive spraying; pushing credentials onto critical systems in business hours; missing an AD trust path.
- Practice: map trust before acting, lowest-impact method, record every hop.
- Use: engagement and CTF (especially AD labs).

### 13. Proof-of-Concept
- A PoC is real, reproducible, ideally non-destructive evidence that a vulnerability is exploitable — it turns a claim into something a client can check, including the impact, without causing harm.
- Workflow: pick the safest demonstration (command output, screenshots, logs, video) → record exact reproduction steps → validate the finding is real before writing it up.
- Pitfalls: exfiltrating sensitive data as "proof"; a PoC the client can't reproduce; confusing a PoC with a full weaponized exploit.
- Practice: clean minimal reproduction, full sequence in the report, remediation advice alongside.
- Use: engagement (primarily).

### 14. Post-Engagement
- Closing a test safely so the client is left no worse off: remove everything you created (accounts, backdoors, persistence, credentials/C2), secure then delete client data appropriately, deliver the report, walk it through, arrange retesting.
- Report contents: executive summary, findings, evidence, remediation guidance.
- Success = clear close-out, a client who understands the findings, no leftover access. Failure = leftover backdoors/credentials, exposed client data, no follow-up.
- Use: engagement only.

### 15. Practice
- Theory without hands-on practice doesn't hold. Technical skill is half the job; communication (writing and presenting) is the other half.
- Practice ladder: modules → retired machines → active machines → Pro Labs / Endgames. Per-module loop: read → do exercises → redo while taking notes → write both technical and non-technical documentation.
- Starting regimen suggested in the module: 2 modules, 3 retired machines (2 easy, 1 medium), 5 active machines (2 easy, 2 medium, 1 hard), 1 Pro Lab/Endgame. Compare against write-ups / walkthroughs (e.g. IppSec) to find your gaps — then fill them.
- Pitfalls: practising only what you're already good at; skipping documentation; being unable to prove on paper what you could do in a lab.
- Practice: proofread docs with technical and non-technical readers; rehearse kickoffs and report walkthroughs.
- Use: engagement and CTF.

## My worked notes (yours, as you do the labs)

- *(Blank — module completed ~4 weeks before this note system. Add anything you remember doing differently, or re-do one section's exercise and record your path here.)*
