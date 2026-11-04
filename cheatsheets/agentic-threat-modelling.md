# Agentic Threat-Modelling Cheat Sheet

## Core Chain

```text
Influence → Trust boundary → Agent → Authority → Effect → Impact
```

## Inventory

```text
[ ] Assets and sensitive data
[ ] Users and direct inputs
[ ] Retrieved/indirect content
[ ] Orchestrator and specialist agents
[ ] Tools and side effects
[ ] Credentials and permissions
[ ] Memory and persistent state
[ ] MCP servers and external APIs
[ ] A2A agents and Agent Cards
[ ] Outputs, egress, and real-world effects
```

## Five Boundary Questions

1. What crosses the boundary?
2. Who controls it?
3. Why is it trusted?
4. What authority becomes reachable?
5. What happens if that trust is wrong?

## Attack Surface

```text
Direct input   Indirect content   Memory
Tools          Credentials        Delegation
MCP            A2A                Output/egress
```

An attack surface is reachable functionality. A vulnerability is the missing control that makes abuse possible.

## Attack-Path Template

```text
Entry point:
Trust boundary:
Influenced component:
Authority/tool/credential:
Action or effect:
Organizational impact:
Controls present:
Missing control:
```

## STRIDE

| Category | Agentic question |
|---|---|
| Spoofing | Can an identity, agent, server, or result be impersonated? |
| Tampering | Can instructions, memory, parameters, or state be changed improperly? |
| Repudiation | Can user → agent → credential → action be attributed? |
| Information Disclosure | Can context, tools, memory, or egress expose data? |
| Denial of Service | Can loops, fan-out, cost, or tools exhaust resources? |
| Elevation of Privilege | Can an agent reach authority outside its role/task? |

## Three Views

```text
STRIDE                      → What security property failed?
OWASP Agentic guidance      → How did agentic design enable it?
OWASP Agentic Top 10        → Which recurring risk pattern is it?
```

## Priority Factors

```text
Reachability   Authority   Blast radius
Detectability  Reproducibility
Persistence    Reversibility   Sensitivity
```

Prioritize the full path, not the phrase “prompt injection.”

## Break the Chain

```text
Source controls
  → data/instruction separation
  → provenance and validation
  → policy-enforced authorization
  → scoped credentials
  → parameter and target constraints
  → human approval
  → egress restrictions
  → monitoring and kill switch
```

## Expected-Behavior Rule

```text
<agent> may call <tool>
with <identity>
against <target/resource>
for <task/tenant>
within <rate/cost/time>
after <approval>
and must log <events>.
```

## Finding Format

```text
Attack path
Agentic weakness
STRIDE impact
Agentic/OWASP category
Business impact
Evidence-based priority
Chain-breaking remediation
Monitoring and validation
```

Detailed notes: [Agentic Threat Modelling](../knowledge-base/ai-security/agentic-threat-modelling.md) · [Methodology](../pentesting-methodology/ai-security/agentic-threat-modelling.md) · [Agentic Fundamentals](agentic-ai-security.md)
