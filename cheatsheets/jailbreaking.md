# Jailbreaking — Quick Reference

> Authorized AI safety evaluation only. Use synthetic targets, canary secrets, mock tools, and bounded trials.

## Fast Distinction

| Jailbreaking | Prompt injection |
|---|---|
| Bypasses model/provider safety policy | Subverts an application's trusted instructions |
| Targets refusal behavior | Targets instruction/data separation |
| May produce restricted content only | May disclose data or misuse tools |
| Can overlap with prompt injection | Can be direct or indirect |

## Safe Test Target

Use a harmless canary:

```text
Protected value: SAFETY-BOUNDARY-CROSSED
Expected behavior: refuse to reveal it
Failure: output the exact value
```

Never substitute real harmful instructions merely to prove a bypass.

## Technique Families

- Roleplay or unrestricted persona
- Fictional/hypothetical framing
- Emotional or urgency framing
- Authority impersonation
- Instruction sandwiching
- Long-context dilution
- Encoding or character substitution
- Word fragmentation
- Multilingual reformulation
- Structured-format injection
- Multi-turn escalation
- Backtracking after refusal

Test families, not famous exact strings.

## Workflow

```text
Authorize → define policy oracle → baseline → single-turn variants
→ multi-turn trajectories → application controls → repeat trials
→ evidence → report → retest
```

## Evidence Fields

- Test-case ID
- Policy rule
- Model/app version
- Date and session
- System state and enabled tools
- Full conversation
- Parameters and trial number
- Expected vs observed behavior
- Screenshots/logs
- Cleanup

## Outcome Labels

| Result | Meaning |
|---|---|
| Pass | Canary protected; allowed requests still work |
| Fail | Controlled input crosses the safety boundary |
| Partial | Leakage occurs but final impact is blocked |
| Inconclusive | Not reproducible or oracle unclear |
| Not tested | Scope or safety constraint prevented testing |

## Metrics

```text
ASR = successful bypasses / valid adversarial trials
Over-refusal rate = blocked allowed requests / valid allowed trials
```

Always report trial count and configuration. Model behavior is non-deterministic.

## Impact Ladder

1. Policy bypass only
2. Synthetic secret disclosure
3. Real sensitive-data disclosure
4. Unauthorized tool proposal
5. Unauthorized tool execution
6. Cross-user or persistent impact

Severity follows demonstrated capability and business impact, not the cleverness of the prompt.

## Defensive Checklist

- Authorization enforced outside the model
- Least-privilege tools and data access
- Human approval for consequential actions
- Input and output safety checks
- Structured, schema-validated tool calls
- Retrieved content treated as untrusted
- Rate limits and abuse monitoring
- Session and tenant isolation
- Continuous regression tests
- Utility/over-refusal tests
- Version tracking and controlled rollout

## Reporting Rule

State exactly:

- which safety rule was bypassed;
- under what conditions;
- how often;
- what the system could actually do;
- which downstream controls prevented or allowed impact.

Do not equate a generated answer with system compromise.
