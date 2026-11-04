# Agentic Threat Modelling

## Core Model

Agentic threat modelling follows the movement of attacker influence through trust boundaries until it reaches authority and creates an effect.

```text
Influence → Trust boundary → Agent decision → Authority → Effect → Impact
```

This model complements traditional asset/component/data-flow analysis by explicitly mapping tools, credentials, delegation, memory, and external agent communication.

## Inventory Before Classification

Record:

- Assets and sensitive data
- Interfaces and direct inputs
- External and retrieved content
- Orchestrators and specialist agents
- Tools and side effects
- Credentials, roles, and approval gates
- Memory and persistent state
- MCP servers and external APIs
- A2A agents and Agent Cards
- Outputs, egress channels, and physical/real-world effects

## Authority Graph

An architecture diagram shows connections. An authority graph shows what each connection permits.

| From | To | Identity | Allowed action | Scope | Approval |
|---|---|---|---|---|---|
| User | Orchestrator | User session | Request task | Tenant | No |
| Agent | Tool | Workload identity | Read/update/deploy | Resource set | Varies |
| Orchestrator | Worker | Delegation grant | Specialist task | Task context | Varies |
| Agent | External system | API token | Query or action | Provider account | Varies |

Model the effective authority, not just the tool name. A harmless-looking read tool may use an identity capable of reading every tenant.

## Attack-Path Template

```text
Entry:
Trust boundary:
Influenced component:
Authority reached:
Action/effect:
Organizational impact:
Controls present:
Missing control:
```

## STRIDE Mapping

| STRIDE | Security question |
|---|---|
| Spoofing | Can a user, service, agent, tool, or result impersonate a trusted identity? |
| Tampering | Can inputs, memory, parameters, records, or state be modified improperly? |
| Repudiation | Can the initiating user, agent, credential, and action be reconstructed? |
| Information Disclosure | Can context, memory, tool results, or egress reveal protected data? |
| Denial of Service | Can loops, fan-out, tool cost, or resource exhaustion disrupt service? |
| Elevation of Privilege | Can influence reach authority outside the intended role or task? |

STRIDE describes impact. It does not replace analysis of how agentic architecture enabled the path.

## Agentic Enablers

Common enabling conditions include:

- Data and instructions share the same context
- External content is trusted because it was retrieved
- Persistent memory lacks provenance or integrity controls
- Tools have broad schemas or excessive side effects
- Authorization relies on model compliance
- Agents share credentials
- Child agents inherit parent authority
- Delegation lacks scope, depth, or fan-out limits
- MCP capabilities are trusted upon discovery
- A2A identity or capability claims are not verified
- Outputs can reach unrestricted external destinations

## Prioritization Model

Rate each threat using evidence rather than labels:

| Factor | Lower concern | Higher concern |
|---|---|---|
| Reachability | Admin-only/internal prerequisite | Public or routine input |
| Authority | Read-only narrow data | Privilege, deployment, finance, or external send |
| Blast radius | One record/task | Cross-tenant or organization-wide |
| Detectability | Strong policy and anomaly alerts | Looks like legitimate agent activity |
| Reproducibility | Unstable/specific conditions | Reliable and repeatable |
| Reversibility | Easy rollback | Irreversible or persistent |

Priority should reflect the complete path. A trivial injection into a powerless agent is different from the same technique reaching a production deployment credential.

## Control Breakpoints

Controls can interrupt different stages:

| Stage | Example controls |
|---|---|
| Entry | Authentication, source allowlists, content scanning |
| Boundary | Provenance labels, data/instruction separation, sandboxing |
| Decision | Constrained planning, deterministic routing, policy checks |
| Authority | Least Agency, scoped identities, just-in-time credentials |
| Action | Parameter validation, target allowlists, human approval |
| Egress/effect | DLP, destination restrictions, transaction limits |
| Detection | Trace correlation, behavioral baselines, anomaly alerts |

Defense in depth matters because no single classifier or prompt reliably blocks all malicious influence.

## Expected-Behavior Specification

Convert the threat model into enforceable and monitorable rules:

```text
<agent> may call <tool>
using <identity>
against <resources/targets>
for <task/tenant>
within <rate/cost/time limits>
with <approval condition>
and must emit <audit events>.
```

Unexpected tool-agent pairs, credential use, destinations, delegation depth, or data volumes become detection signals.

## Finding Standard

A useful finding contains:

1. Complete attack path
2. Trust boundary and untrusted influence
3. Authority that enabled action
4. Agentic weakness
5. STRIDE and agentic classification
6. Business impact
7. Evidence-based priority
8. Specific chain-breaking controls
9. Expected telemetry and validation test

## Related Notes

- [Agentic AI Fundamentals](agentic-ai-fundamentals.md)
- [Agentic Threat-Modelling Methodology](../../pentesting-methodology/ai-security/agentic-threat-modelling.md)
- [Agentic Threat-Modelling Cheat Sheet](../../cheatsheets/agentic-threat-modelling.md)

## Source Contributions

- TryHackMe — Agentic Threat Modelling

> Threat modelling and validation must remain within explicit authorization and data-handling boundaries.
