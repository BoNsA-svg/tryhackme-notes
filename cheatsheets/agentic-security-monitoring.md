# Agentic Security Monitoring Cheat Sheet

## Incident Chain

```text
START            ACT               CHECK              REACH
trigger/input → tool + identity → control/approval → result/effect
```

## Five Questions

```text
Who acted?
What tool and parameters?
Which credential supplied authority?
What started and approved it?
What changed, left, or was returned?
```

## Required Records

```text
[ ] timestamp + ordered sequence
[ ] session/turn/trace/call IDs
[ ] user + agent + delegated agent
[ ] tool + sanitized parameters + target
[ ] service identity/credential name
[ ] queue/delegation lineage
[ ] schedule create + trigger fire
[ ] memory read/write + writer
[ ] approval request + decision
[ ] guardrail name + decision
[ ] result + destination/effect
[ ] content/payload hash
```

Never log passwords, tokens, connection strings, or unnecessary personal data.

## Baseline

```text
Tools | Credentials | Frequency | Sequences
Targets | Parameters | Destinations | Tenants
Schedules | Memory | Cost/resource use
```

Compare against both a rolling baseline and a frozen trusted baseline.

## Red Flags

```text
First-seen tool / credential / destination / agent edge
Sensitive tool outside normal sequence
Growing export range or data volume
Cross-tenant/environment target
Unexpected schedule or memory mutation
Repeated not_evaluated controls
Missing call/result/approval/creation/lineage record
Credential used outside agent runtime
```

## Authorization Test

An unusual action is authorized only if:

```text
Request matches action
+ Approval names request/action
+ Approval occurs before action
= Authorized
```

Match beneficiary, resource, operation, privilege, parameters, destination, and time. “Closed” is not authorization.

## Guardrail Decisions

| Decision | Interpretation |
|---|---|
| allow | evaluated and permitted |
| deny | evaluated and blocked |
| modify | executed with safer parameters |
| not_evaluated | control disabled; call was not checked |

## Missing-Record Checks

```text
trigger.fire        ↔ schedule.create
risky tool.call     ↔ approval/guardrail
queue.read/post     ↔ delegation chain
tool.result         ↔ tool.call
converted content   ↔ source + output hash + recipient
memory effect       ↔ memory write + writer + approval
```

## Investigation Loop

```text
Preserve → Timeline → Trigger → Delegations
→ Calls/identities → Controls/approvals → Results/effects
→ Baseline → Missing records → Contain → Report
```

## Containment

```text
Kill switch | Revoke credentials | Block egress
Suspend queues/schedules | Isolate memory | Preserve evidence
```

A kill switch must sit outside the agent's authority and log operator, time, target agent, reason, and revoked access.

## Related Notes

[Knowledge](../knowledge-base/ai-security/agentic-security-monitoring.md) · [Methodology](../pentesting-methodology/ai-security/agentic-security-monitoring.md) · [Threat Modelling](agentic-threat-modelling.md)
