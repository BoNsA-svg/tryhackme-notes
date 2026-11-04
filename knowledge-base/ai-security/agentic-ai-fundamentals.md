# Agentic AI Fundamentals

## Definition

An agentic AI application gives a model enough architecture to select and execute actions toward a goal. The model may plan, maintain state, call tools, delegate to other agents, and use results to choose subsequent steps.

Agentic behavior is a spectrum. A system can permit autonomous tool selection while still requiring human approval for consequential execution.

## Architecture Spectrum

| System | Who chooses the route? | External effects | Security character |
|---|---|---|---|
| LLM response | Model generates text | None by default | Primarily information and content risk |
| Deterministic workflow | Application code | Predefined actions | Easier to test and constrain |
| Agentic workflow | Model | Dynamically selected actions | Larger probabilistic control surface |
| Multi-agent system | Orchestrator and workers | Distributed actions | Identity and delegation complexity |

Use the least autonomous architecture that meets the business requirement.

## Agent Loop

```text
Perception → Reasoning → Decision → Action → Observation → Updated state
```

Each loop can accumulate state and authority. Security controls must evaluate every action, not merely the user's original request.

## Tools as Capability Boundaries

A tool schema tells the model how to request an action. The execution layer must independently enforce:

1. Caller identity
2. User and agent authorization
3. Input constraints
4. Target/resource scope
5. Side-effect policy
6. Confirmation requirements
7. Rate, cost, and time limits
8. Output filtering and provenance
9. Audit logging

A system prompt such as “do not delete files” is not an authorization control. The delete capability should be absent, narrowly scoped, or guarded outside the model.

## Agent-Computer Interface Security

Model-facing tools should be designed as carefully as human-facing interfaces.

| Design property | Security value |
|---|---|
| Precise name and description | Reduces wrong-tool selection |
| Strict schema and enums | Limits malformed or adversarial arguments |
| Narrow operation | Shrinks blast radius |
| Explicit side-effect label | Supports approval policy |
| Idempotency key | Limits duplicate execution |
| Structured error | Reduces unsafe recovery guesses |
| Bounded result | Prevents context flooding and data overexposure |

## Multi-Agent Identity and Delegation

Multi-agent architecture can reduce privilege concentration if each worker has a distinct identity and only the tools required for its role. It becomes dangerous when credentials or permissions silently flow through the delegation chain.

Track:

- Initiating user
- Orchestrator identity
- Worker identity
- Delegation reason
- Granted scope and expiration
- Credential used
- Tool and resource touched
- Result and subsequent decision

Do not assume a child agent should inherit its parent's authority.

## MCP Trust Model

MCP separates the host, client, and server. Security review should cover:

- Server provenance and ownership
- Transport security and authentication
- Tool/resource/prompt discovery changes
- Per-tool authorization
- Schema validation
- Prompt or content injection through tool metadata and results
- Data returned to the model
- Local process permissions
- Server updates and supply-chain risk

Discovery means “available,” not “approved.”

## A2A Trust Model

External agents introduce an independently controlled reasoning system. Review:

- Agent Card provenance and integrity
- Allowlisting and onboarding
- Authentication and audience restriction
- Claimed versus verified capabilities
- Data minimization
- Task and result schemas
- Delegated authority
- Result validation
- Revocation and incident response

Treat an external agent's output as untrusted data, even when authenticated.

## Least Agency

Least Agency asks two questions:

1. Is autonomy actually required here?
2. If it is, what is the minimum identity, permission, duration, data, and action set required?

Implementation patterns:

- Replace autonomous decisions with deterministic routing when rules are stable.
- Give each agent a unique workload identity.
- Issue just-in-time, short-lived credentials.
- Separate read, propose, approve, and execute capabilities.
- Require human approval for irreversible, financial, external-communication, privilege, or high-volume actions.
- Constrain targets with allowlists and policy-as-code.
- Apply budgets, quotas, rate limits, and maximum loop counts.
- Revoke permissions and clear temporary state after completion.

## Memory and State

Memory can preserve poisoned instructions, sensitive data, obsolete decisions, or excessive permissions across sessions. Define:

- What may be written
- Which identity owns it
- Who can read or modify it
- Provenance and timestamp
- Retention and deletion
- Integrity validation
- Separation between tenants and tasks

Persistent memory should not silently become trusted instruction.

## Failure Modes

- Incorrect or malicious tool selection
- Parameter manipulation
- Excessive permission or standing credentials
- Confused-deputy behavior
- Unauthorized delegation
- Credential inheritance
- Delegation loops
- Duplicate or repeated side effects
- Tool-result prompt injection
- Persistent memory poisoning
- External-agent impersonation
- Loss of audit attribution

## Defensive Principle

```text
Model proposes.
Policy authorizes.
Application executes.
Telemetry records.
```

The model may help choose an action, but deterministic controls should decide whether that action is allowed.

## Source Contributions

- TryHackMe — Agentic Fundamentals

## Related References

- [Agentic AI Security Review](../../pentesting-methodology/ai-security/agentic-ai-security-review.md)
- [Agentic AI Security Cheat Sheet](../../cheatsheets/agentic-ai-security.md)

> Agentic security testing requires explicit authorization and strict handling of credentials, tools, memory, and external systems.
