# TryHackMe — Agentic Fundamentals

> Source-organized room note for the TryHackMe Agentic AI Security Path.

## What Makes a System Agentic?

An agentic system can perceive context, reason about a goal, choose a next step, and act with a meaningful degree of autonomy. The distinction is architectural: the same model can power either a fixed workflow or an agent.

| Architecture | Decision path |
|---|---|
| Plain LLM | Produces a response from its context |
| Fixed workflow | Engineering code decides the route, tool, or specialist |
| Agentic system | The model decides which available action, tool, or agent to use |

Removing hardcoded routing increases flexibility, but it also moves an authorization-sensitive decision from deterministic application logic into probabilistic model reasoning.

## Core Capabilities

- **Planning and reasoning:** decomposes a goal and determines the order of work.
- **Memory and statefulness:** retains conversation state and may read/write persistent memory.
- **Action and tool use:** changes or queries external systems rather than only describing an action.

Together these capabilities create a loop:

```text
Perceive → Reason → Decide → Act → Observe result → Repeat
```

## ReAct and Environmental Grounding

Reasoning alone may generate an unverified answer. Action alone lacks the context needed to choose safely. ReAct-style systems interleave reasoning and action so tool results become fresh evidence for the next decision.

```text
Goal
  ↓
Reason about next step
  ↓
Request a tool action
  ↓
Application executes it
  ↓
Tool result enters context
  ↓
Reason again
```

Tool feedback improves grounding, but a successful tool response proves only what the tool returned—not that the requested action was authorized or safe.

## Tool Anatomy

A model-facing tool normally defines:

- A stable name
- A precise description and usage boundary
- A structured input schema
- Required parameters
- An application-side implementation
- A structured result returned to the model

Example definition:

```json
{
  "name": "check_role",
  "description": "Look up an employee's role and permitted scopes.",
  "input_schema": {
    "type": "object",
    "properties": {
      "employee_id": { "type": "string" }
    },
    "required": ["employee_id"]
  }
}
```

The model emits a structured tool request; the application—not the model—must authenticate, authorize, validate, execute, and return the result.

## Agent-Computer Interface

Agent-Computer Interface (ACI) applies interface-design discipline to tools intended for models. Security-relevant qualities include:

- Unambiguous names and descriptions
- Narrow parameters and strict schemas
- Explicit side-effect and confirmation semantics
- Machine-readable errors
- Bounded outputs
- Safe defaults and idempotency where possible

Poorly designed tools increase wrong-tool selection, malformed arguments, unintended side effects, and recovery failures.

## Multi-Agent Systems

A common pattern is **orchestrator-workers**:

```text
User → Orchestrator → Specialist agent(s) → Orchestrator → Final response
```

The orchestrator decomposes work, delegates subtasks, and synthesizes results. A sub-agent can be represented to the orchestrator as another tool or handoff.

### Benefits

- Smaller credential and capability scope per agent
- Less reasoning contamination between independent subtasks
- Specialist prompts and tools
- Parallel execution

### Costs and risks

- Higher token and infrastructure cost
- More coordination states and trust boundaries
- Harder tracing and debugging
- Delegation loops or conflicting results
- Identity, credential, and permission propagation
- Greater difficulty attributing an action to the responsible agent

## Agent Communication

### Model Context Protocol (MCP)

MCP standardizes how an AI application discovers and connects to external tools, resources, and prompts.

| Role | Responsibility |
|---|---|
| Host | The AI application coordinating model interactions |
| Client | Maintains a connection to one MCP server |
| Server | Exposes tools, resources, or prompts |

MCP standardizes connectivity; it does not remove the need for authentication, authorization, provenance checks, input validation, output validation, or least privilege.

### Agent-to-Agent (A2A)

A2A supports communication with independently operated agents. A typical interaction includes:

1. Discover the external agent through its Agent Card.
2. Determine and complete the required authentication.
3. Send a task or message in the advertised format.
4. Optionally receive streamed progress or results.

Agent Cards describe claimed capabilities and authentication requirements. They are metadata, not proof that the agent is trustworthy.

### MCP versus A2A

| Standard | Primary relationship |
|---|---|
| MCP | Agent/application to tools, data, prompts, and resources |
| A2A | Independent agent to independent agent |

A single architecture can use MCP for tools and A2A for external agents.

## Least Agency

Least Agency combines least privilege with minimization of autonomy. Do not introduce autonomous behavior where a deterministic workflow is sufficient, and do not give an agent more authority than its current task requires.

Practical controls:

- Time-boxed access rather than standing access
- Separate credentials for each agent
- Tool-specific permissions instead of broad service accounts
- Explicit tracking of delegated and inherited authority
- Human approval for high-impact or irreversible actions
- Read-only operations by default
- Scope, cost, and rate limits
- Revocation and session termination

## Security Boundaries

Every transition should be treated as a trust boundary:

- User input → orchestrator
- Orchestrator → worker agent
- Agent → tool
- MCP client → MCP server
- Local agent → external A2A agent
- Short-term context → persistent memory
- Tool result → subsequent reasoning

## Key Takeaways

- Agentic behavior comes from architecture, not merely from model capability.
- Tools convert uncertain model decisions into real-world effects.
- Environmental feedback improves grounding but does not enforce authorization.
- Multi-agent specialization reduces some blast radius while adding coordination risk.
- MCP and A2A solve different communication problems and can coexist.
- Least Agency is the default security principle for autonomy, credentials, delegation, and tools.

## Related Knowledge

- [Agentic AI Fundamentals](../../../knowledge-base/ai-security/agentic-ai-fundamentals.md)
- [Agentic AI Security Review](../../../pentesting-methodology/ai-security/agentic-ai-security-review.md)
- [Agentic AI Security Cheat Sheet](../../../cheatsheets/agentic-ai-security.md)

## Source

- TryHackMe — Agentic Fundamentals

> Use agentic testing techniques only in systems you own or are explicitly authorized to assess.
