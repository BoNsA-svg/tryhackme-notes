# TryHackMe — Re-Testing

> Source-specific study note. Reusable guidance lives in the [Re-Testing Methodology](../../../reporting/retesting-methodology.md), with a [quick reference](../../../cheatsheets/retesting.md) and [report template](../../../templates/retest-report-template.md).

## Learning Objectives

- Distinguish a scoped re-test from a full reassessment.
- Classify Fixed, Not Fixed, Partial, Mitigated, Risk Accepted, and Unable to Retest outcomes.
- Detect symptom patches by testing justified variants.
- Collect comparable evidence for both successful and failed remediation.
- Maintain clear authorization boundaries.

## Re-Test vs Reassessment

A re-test verifies only findings from the original report. It uses the original evidence and root cause as a baseline and normally produces an addendum or status report.

A reassessment evaluates the current attack surface as a new engagement and produces a new pentest report. It is appropriate after a major rewrite, migration, redesign, or other substantial change.

Ask what changed. A rebuilt application should not receive application-wide assurance from replaying a handful of old findings.

## Separate Authorization

Re-testing happens after the original engagement and remediation. Confirm fresh written authorization, exact findings and assets, environments, dates, credentials, prohibited actions, evidence requirements, and escalation contacts.

Scope language matters:

- “Re-test F-01” is narrow.
- “Re-test the login form” covers a component.
- “Re-test authentication” may cover multiple surfaces.

Clarify ambiguity before testing.

## Remediation Window

The remediation window is the agreed time for the client to implement fixes before re-testing. Its duration depends on severity, complexity, business constraints, and contract. Define it in the original Statement of Work when possible rather than assuming a universal timeframe.

## Outcome Classification

| Outcome | Meaning |
|---|---|
| Fixed / Pass | Root cause removed; original proof and reasonable variants fail |
| Not Fixed / Fail | Original proof or a variant shows the same weakness remains |
| Partially Fixed | Exposure reduced but residual weakness remains |
| Mitigated | Approved compensating control reduces risk without removing root cause |
| Risk Accepted | Client formally accepts the unresolved risk |
| Unable to Retest | Access or scope prevented a reliable conclusion |

Terminology varies. Criteria and supporting evidence matter more than the label.

## Root Cause vs Symptom

A blocked payload does not prove remediation. The sequence is:

1. Review the original finding and root cause.
2. Confirm the current authorized target.
3. Repeat the original proof.
4. If blocked, test reasonable variants exercising the same root cause.
5. Check all explicitly scoped instances.
6. Check for regression.
7. classify the result.

For SQL injection, blocking one string is a symptom patch. Parameterized queries correct the underlying confusion between SQL syntax and user-controlled data.

## Boundaries for Variant Testing

Testing alternate syntax or encoding against the same authorized input may establish whether the original root cause remains.

A newly noticed endpoint, unrelated form, or new attack path is not automatically part of the re-test. Record it minimally, notify the client separately, and obtain written authorization before further testing.

## Verification by Remediation Type

- **Vendor patch:** confirm the actual instance and release, then repeat the proof. A version banner alone is insufficient.
- **Code fix:** replay the proof, test justified variants, and review the authorized diff when available.
- **Configuration:** verify the effective runtime setting and every scoped instance.
- **Architecture:** remap the original path, including direct access and trust boundaries. Consider a reassessment if the change is extensive.
- **Compensating control:** verify its effectiveness and document residual risk; do not call the root cause fixed.

## Evidence

Evidence is required for every finding, including passes.

Organize it by finding ID and preserve:

- original and re-test dates;
- target, environment, and instance;
- identity and role;
- version/build;
- remediation claim;
- original proof result;
- variants tested;
- full relevant request/response or tool output;
- cleanup and final outcome.

Comparable evidence allows a reviewer to see the vulnerable baseline and the current result.

## Re-Test Report

The deliverable supplements rather than replaces the original report. Include:

1. re-test authorization, scope, and limitations;
2. dates and outcome totals;
3. a status row for every in-scope finding;
4. details for failed, partial, or mitigated findings;
5. cleanup and evidence appendix;
6. a clear statement that this was not a full reassessment.

New out-of-scope observations belong in a separate dated memo.

## Key Takeaway

Re-testing is comparison, not discovery. A competent re-test verifies the root cause, not only the original payload, while remaining strictly inside the newly authorized scope.
