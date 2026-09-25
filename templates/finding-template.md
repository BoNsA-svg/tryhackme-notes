# [F-ID] [Finding Title]

| Field | Value |
|---|---|
| Risk | [Critical / High / Medium / Low / Informational] |
| Score | [Method, version, vector, and score] |
| Status | [Confirmed / Risk Accepted / Fixed / Partial / Not Fixed / Unable to Retest] |
| Affected assets | [Exact hosts, URLs, endpoints, parameters, roles, or components] |
| Preconditions | [Authentication, access, interaction, timing] |
| CWE / reference | [If applicable] |

## Summary

[State the weakness, affected component, required access, and primary outcome in plain language.]

## Background

[Explain the security control and root cause at the level needed by the remediation owner.]

## Technical Details and Evidence

### Preconditions

[Required account, role, network position, configuration, or workflow state.]

### Reproduction

1. [Step]
2. [Step]
3. [Step]

~~~http
[Sanitized request or other evidence]
~~~

~~~http
[Relevant sanitized response or result]
~~~

### Observed Result

[State exactly what was demonstrated.]

## Impact

[Describe client-specific, realistic consequences. Separate demonstrated impact from additional plausible impact.]

## Likelihood

[Explain exposure, required capability, reliability, prerequisites, and compensating controls.]

## Root Cause

[Identify the design, implementation, or configuration failure.]

## Remediation

### Required Root-Cause Fix

[Primary correction.]

### Additional Defense in Depth

- [Optional control]
- [Optional monitoring or containment]

## Retest Criteria

- [Observable condition]
- [Negative/bypass test]
- [Regression or adjacent-path check]

## References

- [Vendor or standards guidance]

## Retest Record

| Date | Version/build | Tester | Result | Evidence |
|---|---|---|---|---|
|  |  |  |  |  |
