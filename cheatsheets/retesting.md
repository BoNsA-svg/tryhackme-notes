# Penetration-Test Re-Testing Cheat Sheet

## First Decision

| Ask | Re-test | Reassessment |
|---|---|---|
| Question | Are known findings fixed? | What vulnerabilities exist now? |
| Scope | Named findings | Full attack surface |
| Change | Targeted remediation | Rewrite, migration, redesign, M&A |
| Output | Addendum/status report | New pentest report |

If the environment substantially changed, recommend reassessment.

## Before Testing

- [ ] New written authorization, SOW, and Rules of Engagement
- [ ] Exact findings, assets, environments, and instances
- [ ] Original report, PoCs, root causes, and evidence
- [ ] Claimed remediation and deployment/build details
- [ ] Credentials and roles
- [ ] Stop conditions and prohibited actions
- [ ] New-observation escalation process
- [ ] Approved outcome labels and delivery format

## Per-Finding Loop

~~~text
Baseline → Confirm target → Understand fix → Replay PoC
→ Test same-root-cause variants → Check regression/instances
→ Classify → Capture evidence → Clean up
~~~

## Outcome Quick Reference

- **Fixed:** root cause removed; PoC and variants fail; no material regression.
- **Not Fixed:** PoC or a variant proves the same weakness remains.
- **Partially Fixed:** exposure reduced, but residual weakness remains.
- **Mitigated:** compensating control reduces risk; root cause remains.
- **Risk Accepted:** client accepts unresolved risk in writing.
- **Unable to Retest:** no valid conclusion was possible.
- **Not Applicable:** retired feature/asset was verified absent.

## Verification by Fix Type

**Code**
- Original PoC
- Equivalent syntax/encoding
- Authorized diff review
- Root-cause construction removed

**Patch**
- Correct target/version
- Correct advisory/CVE
- PoC after patch
- Every scoped node

**Configuration**
- Effective runtime value
- Reload/restart confirmed
- Direct path checked
- Consistency across systems

**Architecture**
- Original attack path remapped
- Direct and routed access checked
- Segmentation/trust boundaries verified
- Reassessment considered

## Do Not Issue a Pass Because

- Ticket says closed
- Exact payload is blocked
- Scanner is clean
- Banner changed
- WAF returned an error
- One instance was patched
- Staging is fixed
- Compensating control exists

## Scope Boundary

Variants of the documented root cause in the authorized component may be verification. A new endpoint or attack path is a new observation unless explicitly authorized.

When new surface appears:

1. Stop.
2. Record minimally.
3. Notify.
4. Separate the memo.
5. Get written authorization.

## Evidence Minimum

- Finding ID and re-test date
- Target/environment/instance
- Version/build and identity/role
- Claimed remediation
- Original PoC result
- Variants and rationale
- Full relevant request/response or command/output
- Regression result
- Cleanup
- Outcome and reviewer

## Report Skeleton

1. Re-test scope and limitations
2. Summary and outcome counts
3. Findings status table
4. Failed/partial finding details
5. New observations reference
6. Cleanup
7. Evidence appendix

Always state: **This re-test is not a full reassessment.**
