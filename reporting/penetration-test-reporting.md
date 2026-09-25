# Penetration-Test Reporting Methodology

## Purpose

A penetration-test report explains:

- what was authorized and tested;
- what was found and how it was verified;
- what the findings mean to the organization;
- what must be fixed first and why;
- what was not tested or could not be completed;
- what artifacts remain and what must be cleaned up.

The report is not a tool-output dump. It is a defensible account of the assessment and an actionable remediation plan.

## Audiences

| Audience | Needs | Primary sections |
|---|---|---|
| Business leadership | Exposure, business impact, priorities, required decisions | Executive summary |
| Security and risk | Risk ranking, attack chains, themes, ownership, coverage | Findings overview and recommendations |
| Developers and administrators | Root cause, affected locations, proof, exact fixes, retest criteria | Detailed findings |
| Auditors and future testers | Scope, methodology, limitations, evidence trail | Appendices |

Write each section for its reader. Keep technical depth in findings, not in the executive summary.

## Recommended Report Structure

1. Cover page and confidentiality marking
2. Document control and contacts
3. Scope, objectives, assumptions, and limitations
4. Executive summary
5. Findings and strategic recommendations
6. Findings summary table
7. Detailed vulnerability write-ups
8. Methodology
9. Assessment coverage and limitations
10. Assessment artifacts and cleanup
11. Retest results, when applicable
12. Supporting appendices

## End-to-End Workflow

### 1. Reconcile Scope

Compare actual coverage with the Rules of Engagement.

Record:

- tested, partially tested, untested, and out-of-scope assets;
- approved changes made during the engagement;
- blocked tests and their cause;
- time, access, stability, or data-handling constraints;
- exclusions that limit assurance.

Never imply complete coverage when only part of the scope was assessed.

### 2. Normalize Evidence

For every candidate finding, preserve:

- target and exact affected component;
- date and time;
- prerequisites and access level;
- request, response, command, or configuration evidence;
- observed result;
- minimal proof of impact;
- cleanup action;
- evidence location.

Redact secrets and unrelated personal data. A screenshot may support a finding, but reproducible text evidence should carry the technical claim.

### 3. Validate Findings

Before reporting an issue:

- reproduce it;
- rule out scanner false positives;
- confirm the affected asset is in scope;
- establish the root cause;
- use the least destructive proof necessary;
- distinguish observed facts from inferred impact;
- document prerequisites and limitations.

### 4. Rate Risk

Use the client’s approved risk matrix. If a public system such as CVSS is used, record its version, vector, and score. CVSS measures vulnerability severity; business priority may also depend on asset criticality, exposure, exploitability, regulatory impact, and compensating controls.

Rate each finding on its own. Explain attack chains separately so individually moderate issues are not silently inflated.

### 5. Write Detailed Findings

Use the finding-writing standard and keep a golden thread:

~~~text
Evidence → Root cause → Realistic impact → Root-cause fix → Retest condition
~~~

A reader should not need to guess where the issue exists, how it was verified, or what “fixed” means.

### 6. Identify Themes and Chains

After drafting individual findings, look across them for:

- repeated authorization failures;
- inconsistent input handling;
- weak identity controls;
- excessive privileges;
- exposed secrets;
- missing segmentation;
- monitoring gaps;
- combinations that create a higher-impact path.

These observations belong in findings and recommendations, not by changing isolated risk scores without explanation.

### 7. Write the Executive Summary Last

Summarize the finished body of evidence. Cover:

- assessment objective and scope;
- overall posture;
- most important business risks;
- material limitations;
- immediate and strategic actions.

Avoid payloads, endpoint lists, acronyms without explanation, and unsupported statements such as “the application is secure.”

### 8. Perform QA

Complete technical, risk, editorial, privacy, and delivery checks. A second reviewer should be able to reproduce the conclusions from the report and evidence.

### 9. Deliver Securely

Use the delivery mechanism agreed in the Rules of Engagement. Confirm:

- the recipient list;
- encryption and access controls;
- retention and deletion requirements;
- critical-finding escalation;
- version and checksum if required.

Do not send an unencrypted report containing sensitive evidence through an unapproved channel.

### 10. Retest

For each remediated finding:

- repeat the original proof;
- test reasonable bypasses and adjacent paths;
- verify the root cause, not only the original payload;
- record the date, build/version, evidence, and result.

Suggested states: Fixed, Partially Fixed, Not Fixed, Unable to Retest, Risk Accepted.

## Core Writing Rules

- Use clear, objective language.
- Describe completed testing in past tense.
- Avoid first-person phrasing.
- Separate observation from inference.
- Name the affected endpoint, parameter, host, role, or control.
- Explain client-specific impact.
- Put the root-cause remediation first.
- Label defense-in-depth measures as additional controls.
- Mask secrets and minimize personal data.
- Use terminology, spelling, severity labels, and formatting consistently.

## Common Failure Modes

| Failure | Why it weakens the report |
|---|---|
| Pasting scanner output | It does not establish validity, impact, or remediation |
| Generic impact | It fails to connect the issue to the client |
| Generic remediation | The owner cannot implement or validate the fix |
| Screenshot-only evidence | It may omit the request, response, context, and reproducibility |
| Inflated severity | It damages trust and prioritization |
| Missing limitations | It creates false assurance |
| Writing the summary first | It encourages conclusions before analysis is complete |
| Exposing secrets | The report becomes an additional security risk |

## Definition of Done

A report is ready when every material claim is supported, every finding is reproducible, every recommendation addresses a root cause, coverage is transparent, sensitive data is controlled, and both technical and business readers know what to do next.
