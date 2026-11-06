# Agentic Security Monitoring

## Core Principle

A valid credential and a successful tool call do not prove that an action was intended. Agent monitoring must reconstruct causality and authority:

```text
Trigger → Delegation → Tool call → Control/approval → Result → External effect
```

The trustworthy record must be append-only or tamper-resistant and outside the agent's ability to rewrite.

## Minimum Telemetry Model

Every event should support correlation across:

- time: timestamp and ordered sequence;
- execution: session, turn, trace, span, call, and parent IDs;
- identity: user, agent, delegated agent, service identity, and approver;
- action: tool, sanitized parameters, target, and destination;
- origin: message, schedule, memory, retrieved content, or agent handoff;
- policy: control name, policy version, decision, reason, and modifications;
- outcome: success/failure, affected resource, data volume, and external effect;
- integrity: immutable reference and payload/content hash.

Log credential names or service identities, never credential secrets.

## Content Lineage

Indirect content can become instruction. Preserve lineage for:

- retrieved documents and webpages;
- emails and tickets;
- API/MCP/A2A responses;
- OCR, screenshots, audio, and transcripts;
- memory reads and writes.

For transformations, record source hash, output hash or protected output, parser/model version, recipient agent, and downstream trace ID.

## Baselines

Build baselines per agent, role, tenant, environment, and credential:

| Dimension | Examples |
|---|---|
| Tools | expected tool set and first-seen tools |
| Identities | expected service accounts and privilege scopes |
| Frequency | calls per hour/day/week |
| Sequence | normal tool-call chains |
| Parameters | date ranges, target systems, data volume |
| Destinations | approved internal/external recipients |
| State | expected schedules and memory topics |
| Resources | token, latency, compute, and cost ranges |

Use both rolling statistics and a frozen trusted baseline. Rolling-only baselines can normalize slow drift.

## Authorization Correlation

A sensitive action is authorized only if the request, approval, and execution agree on:

- requester and beneficiary;
- resource, operation, and privilege level;
- parameters and destination;
- time window;
- sequence: approval precedes action.

Workflow state such as “closed” does not prove authorization.

## Detection Patterns

High-value detections include:

- first-seen tool, credential, destination, or agent-to-agent edge;
- sensitive tool used outside normal sequence;
- escalating parameter scope or export range;
- repeated `not_evaluated` controls;
- schedule execution without an attributable creation;
- memory mutation followed by gradual behavior change;
- result without call, call without result, or broken delegation lineage;
- cross-tenant or cross-environment actions;
- credential used outside its agent runtime;
- unusual data volume, frequency, cost, or fan-out.

Absence can be a detection signal. Expected-record invariants should be monitored explicitly.

## Controls

- Guardrails should execute outside the model and log allow, deny, modify, or not-evaluated.
- Human checkpoints should protect high-impact actions, log request and decision, and avoid approval fatigue.
- Kill switches must be independent of agent credentials, immediate, tested, owned, and auditable.
- Credential revocation, egress blocking, queue suspension, and memory isolation should be available containment actions.

## Monitoring Design Rule

Write expected behavior as a machine-testable policy:

```text
<agent> may call <tool> with <identity>
against <target> for <task/tenant>
within <rate, parameter, and time bounds>
after <required approval/control>
and must emit <required records>.
```

## Related Material

- [Agentic AI Fundamentals](agentic-ai-fundamentals.md)
- [Agentic Threat Modelling](agentic-threat-modelling.md)
- [Investigation Methodology](../../pentesting-methodology/ai-security/agentic-security-monitoring.md)
- [Cheat Sheet](../../cheatsheets/agentic-security-monitoring.md)
