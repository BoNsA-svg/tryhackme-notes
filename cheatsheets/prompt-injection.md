# Prompt Injection Cheat Sheet

> Authorized testing only. Use synthetic data, canary secrets, mock tools, and reversible actions.

## Definitions

| Term | Fast definition |
|---|---|
| Prompt injection | Untrusted input changes intended LLM application behavior |
| Direct injection | Attacker submits instructions through normal input |
| Indirect injection | Instructions arrive through documents, web, email, RAG, or tools |
| Jailbreak | Injection used to bypass model safety restrictions |
| Prompt leakage | System/developer instructions are disclosed |
| Excessive agency | Model has more tools or authority than necessary |

## Context Sources

- Platform/system/developer instructions
- User input
- Conversation history
- RAG documents
- Emails, files, and websites
- Tool/API outputs
- Persistent memory
- Multimodal content

## Risk Path

~~~text
Attacker control → Context ingestion → Model manipulation
→ Missing authorization/validation → Data, decision, or action impact
~~~

## Test Order

1. Confirm scope, side effects, and stop conditions.
2. Map every context and ingestion source.
3. Map tools, permissions, tenants, and approval.
4. Record normal task and refusal baselines.
5. Test direct categories.
6. Test each indirect ingestion surface.
7. Test multi-turn and memory persistence.
8. Verify downstream authorization.
9. Test output/rendering channels.
10. Repeat material cases and record success rate.
11. Clean up and report.

## Direct Test Categories

- Task substitution
- Paraphrased override
- Authority impersonation
- Fake dialogue/history
- Markup/comment/structured-data instruction
- Multi-turn shaping
- Split instruction
- Protected-context transformation
- Cross-language or encoded equivalent, if authorized

Use a benign canary outcome first.

## Indirect Surfaces

- RAG documents and vector stores
- Email and attachments
- Calendar entries
- Websites and search results
- PDFs and office files
- Repositories, issues, and READMEs
- Tool and API responses
- Images and metadata
- Shared or persistent memory

## Impact Levels

- Output steering only
- Sensitive-data disclosure
- Trusted-decision manipulation
- Unauthorized tool request, blocked
- Unauthorized action executed
- Persistent or cross-user compromise

Severity depends on capability, not how dramatic the prompt sounds.

## Non-Negotiable Controls

1. System prompts are not access-control boundaries.
2. Treat retrieved content and tool output as untrusted data.
3. Enforce user, tenant, object, operation, and parameter authorization in code.
4. Give tools least-privileged, short-lived credentials.
5. Require explicit approval for sensitive or irreversible actions.
6. Never put secrets in prompts.
7. Validate tool arguments and output schemas.
8. Restrict network egress and external destinations.
9. Log context provenance, retrieval, tool requests, authorization, and approvals.
10. Assume filters and delimiters can fail.

## Evidence

- App/model/prompt/guardrail versions
- Exact sanitized input and output
- Context source and provenance
- Retrieved document IDs
- Tool call and arguments
- Authorization and approval result
- Trial count and success rate
- Side effect or blocked effect
- Cleanup

## Finding Formula

> Attacker-controlled **[source]** entered **[context]**, altered **[behavior]**, and caused/attempted **[impact]** because **[deterministic control]** was missing or bypassed.

## Retest

- Original test
- Paraphrased variants
- Indirect variants
- Multi-turn/persistence
- Cross-user/tenant isolation
- Downstream authorization
- Human approval
- Logging and cleanup
- Normal workflow regression

## Related

- [Full Prompt Injection Note](../knowledge-base/ai-security/prompt-injection.md)
- [Testing Methodology](../pentesting-methodology/ai-security/prompt-injection-testing.md)
- [AI Threat Modelling Cheat Sheet](ai-threat-modelling.md)
