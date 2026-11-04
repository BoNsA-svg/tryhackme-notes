# TryHackMe — Agentic Threat Modelling

> Source-organized room note for the TryHackMe Agentic AI Security Path.

## Purpose

Agentic threat modelling examines how attacker-controlled influence can enter an agent estate, cross trust boundaries, reach tools or delegated authority, propagate across agents and protocols, and create real-world effects.

The model is only one component. A useful threat model covers the complete system: people, inputs, orchestrators, specialist agents, memory, tools, identities, credentials, data stores, MCP servers, A2A agents, external services, and outputs.

## Agent Estate Inventory

Start with each agent's role and effective capability.

| Agent type | Example role | Typical capability | Relative concern |
|---|---|---|---|
| Orchestrator | Delegates work | Queue or hand off tasks | Propagation and authority routing |
| Documentation | Retrieves knowledge | Read services/runbooks | Sensitive retrieval and indirect input |
| Service desk | Processes requests | Read/update tickets | Untrusted ticket content plus write access |
| Access | Manages permissions | Check/grant access | Identity and privilege changes |
| Deployment | Manages releases | Inspect/deploy | Production code and availability |
| Mail | Reads/sends email | Read/send messages | Data ingress and external egress |
| Telemetry | Queries logs/metrics | Search monitoring data | Sensitive operational information |
| Media | Transcribes/extracts | Process untrusted media | Hidden or indirect instructions |

Capability alone does not determine authority. Every tool executes under a credential, so the effective principal is the combination of agent, tool, identity, permissions, and approval policy.

## Scoped Identities

Separate identities reduce blast radius:

```text
docs-agent   → docs-read-svc
deploy-agent → deploy-prod-svc
mail-agent   → mail-send-svc
```

A shared administrative credential turns compromise of a low-risk agent into compromise of every permission held by the shared account. Apply Least Agency by giving each agent only the identity and permissions required for its role.

## External Connections

MCP servers, external APIs, retrieved knowledge, and A2A agents create trust boundaries. Review:

- What information leaves the organization
- What content, capability, or action returns
- Which credential authenticates the connection
- What authority is delegated
- Whether capability metadata and results are trusted
- Whether external effects require approval

## Four Building Blocks

### Assets

Identify what the organization must protect: identities, credentials, employee data, internal knowledge, tickets, deployment systems, source code, production environments, memory, telemetry, and audit logs.

### Components

Inventory interfaces, orchestrators, agents, tools, memory stores, databases, MCP clients/servers, APIs, and external A2A agents.

### Data flows

Trace each request and response through the estate:

```text
Employee → Orchestrator → Specialist agent → Tool/data store → Result
```

At each step record who controls the data, whether it may contain instructions, how it is authenticated, and whether it can influence later action.

### Trust boundaries

A trust boundary exists where data moves between components with different security assumptions:

```text
UNTRUSTED                         TRUSTED
External webpage ───────────────→ Documentation agent
Support ticket ─────────────────→ Service-desk agent
External MCP result ────────────→ Orchestrator context
```

The central principle is:

```text
Data can become influence.
Influence can reach authority.
Authority can create effects.
```

## Five Boundary Questions

For every meaningful connection ask:

1. What crosses the boundary?
2. Who controls it?
3. Why does the receiver trust it?
4. What authority becomes reachable?
5. What happens if the trust assumption is wrong?

## Agentic Attack Surface

| Surface | Example abuse |
|---|---|
| Direct input | User manipulates agent instructions |
| Indirect input | Malicious content in email, webpage, document, ticket, API response, or transcript |
| Memory | Influence persists into later sessions |
| Tools | Agent invokes database, deployment, email, or other side-effect capability |
| Credentials | Permissions exceed the current task |
| Delegation | Influence or authority passes to another agent |
| MCP | External server changes available capabilities or returns malicious content |
| A2A | External agent influences reasoning or receives delegated authority |
| Output/effect | Data leaves, messages are sent, access changes, or systems are modified |

An attack surface is not automatically a vulnerability. The vulnerability is the missing or ineffective control that makes abuse possible.

## Attack Path

An attack path explains how influence becomes organizational impact.

| Element | Question |
|---|---|
| Entry point | Where did attacker-controlled influence enter? |
| Trust boundary | Which assumption or security boundary was crossed? |
| Influenced component | Which agent or subsystem processed it? |
| Authority | Which tool, credential, permission, or delegation enabled action? |
| Effect | What action or state change occurred? |
| Impact | What was exposed, modified, disrupted, or gained? |

Example:

```text
Malicious webpage
  → external-web/internal-agent boundary
  → docs-agent interprets data as instruction
  → directory-read authority
  → delegation to mail-agent
  → external email
  → confidential employee data disclosure
```

Naming only “prompt injection” is insufficient. Defenders need the complete route, authority, impact, and interruption points.

## STRIDE

STRIDE identifies the security property that failed.

| Category | Agentic example |
|---|---|
| Spoofing | External service or agent impersonates a trusted identity |
| Tampering | Agent modifies records or state without authorization |
| Repudiation | Action cannot be attributed to user, agent, credential, or delegation |
| Information Disclosure | Tool, response, or external action exposes sensitive data |
| Denial of Service | Agent enters a loop or repeatedly invokes expensive operations |
| Elevation of Privilege | Agent reaches tools or authority beyond its intended scope |

One attack path may map to several STRIDE categories.

## Three Complementary Views

| Framework | Main question |
|---|---|
| STRIDE | What security property failed? |
| OWASP Agentic AI Threats & Mitigations | Which agentic capability or trust assumption enabled it? |
| OWASP Agentic Top 10 | Which recurring risk pattern should be prioritized? |

Relevant agentic areas include goals/instructions, memory, tools, credentials, delegation, inter-agent communication, external systems, and real-world effects. Do not force a multi-stage path into one exclusive label.

## Prioritization

Assess at least:

- **Reachability:** Can an attacker reach the entry point easily?
- **Authority:** Which permissions become available?
- **Blast radius:** How many agents, systems, users, or records could be affected?
- **Detectability:** Would the action look legitimate or trigger monitoring?
- **Reproducibility:** Can the path be triggered consistently?

Also consider sensitivity, reversibility, required preconditions, human approval, and persistence.

## Finding Format

| Field | Record |
|---|---|
| Attack path | Entry → boundary → component → authority → effect |
| Agentic weakness | Missing separation, excessive authority, unsafe delegation, etc. |
| STRIDE impact | One or more affected security properties |
| Agentic category | Relevant OWASP agentic area or Top 10 pattern |
| Business impact | Confidentiality, integrity, availability, financial, safety, or compliance effect |
| Priority | Reachability, authority, blast radius, detectability, reproducibility |
| Controls | Preventive, detective, and containment breakpoints |

## Breaking the Chain

Layer controls at multiple stages:

```text
Input isolation
  → context/instruction separation
  → provenance and content validation
  → policy-enforced tool authorization
  → scoped credentials
  → human approval
  → egress controls
  → monitoring and response
```

A control is most valuable when it prevents attacker influence from reaching authority or prevents authority from producing an unsafe effect.

## Expected Behavior

Threat modelling should define observable expectations:

- Which agent may call which tool
- Which identity it must use
- Which targets and data classes are allowed
- When approval is mandatory
- Maximum loops, fan-out, cost, and volume
- Permitted egress destinations
- Required audit events

These expectations become test cases and monitoring rules.

## Related Notes

- [Agentic AI Fundamentals](agentic-fundamentals.md)
- [Agentic Threat Modelling Knowledge](../../../knowledge-base/ai-security/agentic-threat-modelling.md)
- [Agentic Threat-Modelling Methodology](../../../pentesting-methodology/ai-security/agentic-threat-modelling.md)
- [Agentic Threat-Modelling Cheat Sheet](../../../cheatsheets/agentic-threat-modelling.md)

## Source

- TryHackMe — Agentic Threat Modelling

> Use only in systems you own or are explicitly authorized to assess.
