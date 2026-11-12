# Prompt Defence — Quick Reference

> Assume the model can be manipulated. Reduce likelihood, contain impact, detect abuse, and recover safely.

## Layered Architecture

```text
User / external content
        ↓
Normalize + input guardrails
        ↓
Scoped prompt + least data
        ↓
Model
        ↓
Output validation
        ↓
Tool broker + authorization
        ↓
Approval + execution
        ↓
Logging + monitoring
```

## Trust Rules

- System prompts guide behavior; they are not secret stores.
- User input is untrusted.
- RAG documents and tool output are untrusted.
- Model output is untrusted.
- Authorization decisions belong outside the model.
- The downstream component validates before use.

## System Prompt Checklist

- Narrow role and task scope
- Native message roles used
- User/retrieved content kept outside privileged messages
- External content explicitly treated as data
- Clear refusal and redirection behavior
- No secrets or credentials
- Prompt version recorded
- Bypass and over-refusal tested

## Input and Context Controls

- Size, file-type, and rate limits
- Unicode normalization
- Cheap pattern checks
- Semantic injection/policy classifier
- PII and secret detection
- Scan RAG chunks, documents, emails, and tool output
- Preserve provenance
- Quarantine uncertain content
- Retrieve only user-authorized records

## Tool Safety

- Allowlisted tools
- Strict typed schemas
- Reject unknown fields
- Validate targets and arguments
- Recheck identity and authorization
- Short-lived scoped credentials
- Resource and transaction limits
- Sandbox code/file operations
- Human approval for consequential actions
- Log proposal, approval, execution, and result

## Output Safety by Sink

| Sink | Control |
|---|---|
| HTML | Encode and sanitize |
| SQL | Parameterized query |
| Shell/code | Do not directly execute; sandbox and allowlist |
| API/tool | Strict schema and authorization |
| Email/message | Verify recipient and require preview/approval |
| Logs/UI | Redact secrets and render safely |
| JSON | Strict parser; reject extras |

## Testing Matrix

- Direct override
- Indirect injection
- Roleplay/jailbreak
- Encoding and homoglyphs
- Multilingual variants
- Long-context dilution
- Multi-turn escalation
- System-prompt extraction
- Data exfiltration
- Unauthorized tool call
- Cross-tenant access
- Unsafe output to each downstream sink

Use synthetic canaries and mock side effects.

## Metrics

```text
ASR = successful attacks / adversarial trials
Over-refusal = blocked legitimate tasks / legitimate trials
Containment = blocked unauthorized actions / attempted actions
```

Always record model, prompt, guardrail, and application versions.

## Common Mistakes

- Secrets in prompts
- Long prompt treated as a complete fix
- Exact-string blocklist as primary defense
- Filtering users but not retrieved content
- Model allowed to approve its own actions
- RAG broader than user permissions
- Direct execution of model output
- No approval for high-impact actions
- No utility testing
- No retest after configuration changes

## Incident Response

```text
Contain capability → preserve transcript and logs → identify failed layer
→ check data/actions → rotate secrets → fix deterministic controls
→ update probabilistic controls → add regression test → retest
```
