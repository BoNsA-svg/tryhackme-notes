# Re-Test Outcomes and Evidence

## Outcome Model

Labels vary between organizations. Define them in the report and apply them consistently.

| Outcome | Criteria | Reporting implication |
|---|---|---|
| Fixed / Pass | Original proof and reasonable same-root-cause variants fail; root cause is removed; no material regression is observed | Close after recording positive verification evidence |
| Not Fixed / Fail | Original proof still succeeds, a variant proves the same root cause remains, or remediation was not deployed consistently | Keep open and update remediation guidance |
| Partially Fixed | Exposure is reduced but residual risk remains, only some instances are fixed, or the root cause is incompletely addressed | Describe residual conditions and reassess remaining risk |
| Mitigated / Compensating Control | Root cause remains, but an approved alternative control reduces exposure | Record control, residual risk, dependencies, owner, and review period |
| Risk Accepted | Authorized client owner formally accepts the unresolved risk | Do not describe as fixed or remediated |
| Unable to Retest | Access, stability, deployment, evidence, or scope prevents a valid conclusion | No assurance; explain what is required to complete verification |
| Not Applicable | The affected feature or asset was formally retired and exposure no longer exists | Verify retirement and absence of reachable equivalent paths |

## Decision Rules

A finding is not Fixed merely because:

- the exact string from the report is blocked;
- a scanner no longer detects it;
- the ticket is closed;
- the version banner changed;
- a WAF returns an error;
- one cluster node was patched;
- a client says the issue was fixed.

Fixed requires behavioral evidence that the documented weakness and its root cause no longer produce the security impact within the authorized scope.

## Risk Acceptance vs Compensating Control

**Risk Accepted**

- no corrective control is asserted as eliminating or reducing the finding;
- the responsible authority accepts the exposure in writing;
- the original finding remains open from a technical perspective.

**Compensating Control**

- the root cause may remain;
- a different control reduces likelihood or impact;
- effectiveness, ownership, monitoring, and durability are verified;
- residual risk is explicitly recorded.

Do not merge these outcomes.

## Evidence Standard

Every finding needs evidence, including findings that pass.

Capture:

- finding ID;
- re-test date and tester;
- target, environment, and instance;
- version/build where relevant;
- identity and role used;
- remediation description;
- original proof result;
- variants attempted and rationale;
- relevant request/response, command/output, or configuration evidence;
- regression result;
- cleanup;
- final outcome and reviewer.

## Evidence Organization

~~~text
retests/
├── F-01/
│   ├── original-evidence/
│   ├── retest-evidence/
│   └── notes.md
├── F-02/
│   ├── original-evidence/
│   ├── retest-evidence/
│   └── notes.md
└── evidence-index.md
~~~

Suggested filename:

~~~text
2026-09-25_F-01_production_original-poc-blocked.txt
~~~

Use sanitized full request/response pairs when they establish the result. A status code or screenshot without context is rarely sufficient.

## Comparable Evidence

For a credible comparison, preserve:

| Original | Re-test |
|---|---|
| Target and environment | Same authorized target or documented replacement |
| Preconditions and role | Equivalent current prerequisites |
| Request, command, or proof | Replayed safely |
| Observed vulnerable result | Current blocked or successful result |
| Date and version | New date and current version |
| Impact evidence | Evidence showing whether impact remains |

If the environment changed, explain how comparability was established.

## Failed-Finding Detail

For each failed or partial result, document:

1. original finding;
2. claimed remediation;
3. verification performed;
4. observed residual weakness;
5. supporting evidence;
6. current impact and residual conditions;
7. updated remediation;
8. required next verification.

## Fixed-Finding Detail

For each pass, retain enough evidence to show:

- correct target and deployment;
- original behavior no longer succeeds;
- reasonable variants also fail;
- root-cause correction or effective control;
- no material regression;
- all authorized instances were covered.

## New Observation Record

Keep new, out-of-scope observations separate:

| Field | Value |
|---|---|
| Date/time | |
| Observed location | |
| Minimal observation | |
| Why it may matter | |
| Testing intentionally not performed | |
| Client notified | |
| Authorization requested | |

This creates a clear boundary between observation and authorized verification.
