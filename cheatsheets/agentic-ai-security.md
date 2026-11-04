# Agentic AI Security Cheat Sheet

## Identify the Architecture

```text
Plain LLM        → responds only
Fixed workflow   → code selects routes/actions
Agent            → model selects actions/tools
Multi-agent      → orchestrator delegates to workers
MCP              → host/client/server tool and data connection
A2A              → independent agent communication
```

## Core Loop

```text
Perceive → Reason → Decide → Act → Observe → Repeat
```

Every repetition must remain inside identity, permission, cost, time, and action limits.

## Inventory

```text
[ ] Agents and orchestrators
[ ] Models and system instructions
[ ] Tools and side effects
[ ] MCP hosts, clients, and servers
[ ] External A2A agents and Agent Cards
[ ] Credentials and workload identities
[ ] Short-term and persistent memory
[ ] Human approvals and stop controls
[ ] Logs, traces, budgets, and rate limits
```

## Least Agency

```text
[ ] Is autonomy necessary?
[ ] Unique identity per agent
[ ] One task-scoped credential per agent
[ ] Short-lived access
[ ] Tool-specific permissions
[ ] Read-only by default
[ ] No silent child-agent inheritance
[ ] Human approval for consequential actions
[ ] Target allowlists
[ ] Loop, fan-out, cost, rate, and time limits
[ ] Revoke access after completion
```

## Tool Review

```text
[ ] Clear name and description
[ ] Strict schema, types, enums, and required fields
[ ] Side effects explicitly identified
[ ] Authorization enforced outside the model
[ ] Resource and target scope enforced
[ ] Confirmation for irreversible operations
[ ] Idempotency and duplicate prevention
[ ] Structured errors
[ ] Bounded, labeled, untrusted results
[ ] Complete audit event
```

## Delegation Review

```text
[ ] Allowlisted worker
[ ] Minimum context passed
[ ] User authorization preserved
[ ] Delegation reason logged
[ ] Worker cannot exceed parent grant
[ ] Loop and fan-out protection
[ ] Results validated before reuse
[ ] Action attributed to responsible identity
```

## MCP Review

```text
[ ] Server provenance verified
[ ] Authenticated transport
[ ] Discovered tools/resources/prompts reviewed
[ ] Per-tool authorization
[ ] Metadata and results treated as untrusted
[ ] Local server process least privilege
[ ] Updates and supply chain controlled
[ ] Sensitive output filtered
```

## A2A Review

```text
[ ] Agent Card integrity/provenance
[ ] External agent allowlisted
[ ] Authentication and audience restricted
[ ] Claimed capabilities verified
[ ] Minimum data shared
[ ] Delegated authority bounded
[ ] Response validated
[ ] Revocation and incident path
```

## Memory Review

```text
[ ] Tenant and task isolation
[ ] Write permission controlled
[ ] Source and timestamp retained
[ ] Stored content cannot silently become instruction
[ ] Sensitive data filtered
[ ] Integrity checked
[ ] Retention and deletion enforced
```

## High-Risk Approval Gates

Require deterministic policy and usually human approval for:

- Deletion or irreversible changes
- Financial transactions
- External messages or publication
- Privilege or identity changes
- Deployment and code execution
- Bulk data access or export
- New persistent memory or credentials

## Secure Control Split

```text
Model proposes → Policy authorizes → Application executes → Telemetry records
```

Detailed notes: [Agentic AI Fundamentals](../knowledge-base/ai-security/agentic-ai-fundamentals.md) · [Review Methodology](../pentesting-methodology/ai-security/agentic-ai-security-review.md)
