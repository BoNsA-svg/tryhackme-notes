# Agentic Security Monitoring

> Source: TryHackMe — Agentic Security Monitoring  
> Scope: defensive monitoring and authorized investigation of agentic systems.

## Purpose

Agent actions can be harmful even when every tool call succeeds and uses a valid account. Monitoring must therefore answer what started an action, what the agent did, which identity gave it authority, what checked it, and what effect occurred.

## Incident Lifecycle

| Stage | Question | Expected record |
|---|---|---|
| Start | What caused the work? | message, queue post, schedule creation/fire, or retrieved-content record |
| Act | What tool was called and with which authority? | tool call with parameters and credential identity |
| Check | Did a person or policy inspect the risky action? | approval decision or guardrail decision |
| Reach | What changed, left, or was returned? | tool result and destination/effect evidence |

A schedule should leave a creation record and a record every time it fires. A risky call should have a control or approval record.

## Agent Audit Logging

A complete chain should preserve:

- timestamp, session, turn, actor, and immutable event reference;
- initiating user, agent, schedule, or external content;
- delegation and queue handoffs;
- tool name, sanitized parameters, target, and call ID;
- service identity or credential name used by the tool;
- approval and guardrail decisions;
- result, destination, status, and affected resource;
- memory reads and writes;
- content or payload hash where raw content cannot be retained safely.

Do not log secrets, access tokens, passwords, connection strings, or unnecessary personal data.

### TryTrainMe Event Types

| Event | Meaning |
|---|---|
| `message` | request arrives or reply leaves |
| `queue.post` | task/result handed to another agent |
| `queue.read` | receiving agent collects it |
| `tool.call` | tool, parameters, and credential used |
| `tool.result` | outcome of the call |
| `schedule.create` | schedule created and attributed |
| `trigger.fire` | schedule ran |
| `approval.decision` | human approval or refusal |
| `guardrail` | automated control decision |

For media conversion, record the source, output text or its hash, and the downstream recipient. Without lineage, hidden instructions can enter another agent without an investigable trail.

## Behavioural Baselines

Baseline each agent by:

1. expected tools;
2. expected service identities;
3. daily/weekly call volume;
4. normal tool sequences;
5. target systems, tenants, destinations, and parameter ranges;
6. schedules, memory use, cost, and resource consumption.

For a new agent, define the expected baseline before deployment and compare the first weeks against it.

Red flags include first-seen tools or credentials, volume outside the normal range, unusual parameter values, unexpected destinations, and missing preceding records. A deviation is a lead, not proof of compromise.

## Authorization Test

An unusual action is authorized only when:

1. a service request asks for the same action;
2. an approval record names that request and action;
3. approval occurs before execution.

A closed request is not proof of authorization. Compare the request subject, target, account, access level, approver, timestamps, and sequence numbers.

## Drift and Missing Records

Rolling averages can move with gradual compromise. Compare behavior to both:

- a recent rolling baseline; and
- a fixed trusted reference period.

Investigate memory entries that could explain drift, who wrote them, and whether a person approved them. Session-only monitoring misses instructions that persist and activate later.

Missing evidence is itself evidence. Examples:

- `trigger.fire` without `schedule.create`;
- risky `tool.call` without approval or guardrail;
- delegation without queue lineage;
- conversion without output/recipient lineage;
- result without a matching call.

## Control Decisions

| Decision | Meaning |
|---|---|
| `allow` | evaluated and permitted |
| `deny` | evaluated and blocked |
| `modify` | allowed with safer parameters |
| `not_evaluated` | control existed but was disabled |

`not_evaluated` is not equivalent to a safe allow.

## Checkpoints and Kill Switches

A human checkpoint holds one risky call until a person decides. It should log both the request for approval and the answer. Excessive prompts create approval fatigue.

A kill switch stops the agent, sits outside the agent's own authority, works immediately even when other controls fail, revokes relevant credentials, has a named operator, and logs who used it, when, against which agent, and what access was removed.

## Practical Incident Result

In the room investigation:

- `docs-agent` used a new media/transcript-reading capability for call `C-2210`; request `R-5512` and a prior approval matched, so the action was authorized.
- `log-agent` ran three scheduled exports with expanding ranges from 7 to 90 days.
- The guardrails were `not_evaluated`, no request or approval authorized the exports, and no `schedule.create` record existed.
- Therefore the exports were unauthorized and schedule ownership was untraceable.

## Related Notes

- [Agentic Fundamentals](agentic-fundamentals.md)
- [Agentic Threat Modelling](agentic-threat-modelling.md)
- [Reusable Knowledge](../../../knowledge-base/ai-security/agentic-security-monitoring.md)
- [Investigation Methodology](../../../pentesting-methodology/ai-security/agentic-security-monitoring.md)
- [Cheat Sheet](../../../cheatsheets/agentic-security-monitoring.md)
