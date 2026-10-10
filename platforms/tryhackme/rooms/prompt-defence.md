# TryHackMe Room Notes — Prompt Defence

## Room Purpose

This room explains how to reduce prompt-injection and jailbreaking risk through layered controls. The key lesson is that prompt-level defenses are not absolute, so secure systems combine model-facing guardrails with deterministic access control, output validation, least privilege, monitoring, and incident response.

## Learning Outcomes

- Explain why no single prompt or classifier guarantees immunity.
- Harden system instructions without trusting them as a security boundary.
- Apply input and output guardrails.
- protect retrieval and tool-use paths.
- Limit blast radius through least privilege.
- Monitor, investigate, and retest prompt-security controls.

## Core Principle

```text
Assume manipulation is possible.
Make success less likely.
Make successful manipulation less powerful.
Detect it quickly.
Recover safely.
```

## Defence-in-Depth Layers

| Layer | Purpose |
|---|---|
| System-prompt hardening | Clarify task, scope, and refusal behavior |
| Input guardrails | Detect obvious and semantic attacks before inference |
| Retrieved-content controls | Treat documents, email, search, and RAG chunks as untrusted |
| Deployment controls | Restrict data, tools, credentials, and actions |
| Output validation | Prevent model output becoming executable downstream input |
| Monitoring | Detect repeated attacks, abuse, leakage, and unusual actions |
| Continuous testing | Measure regression after every material change |

## System-Prompt Hardening

Useful patterns include:

- tight task scope;
- clear instruction hierarchy;
- explicit handling of override attempts;
- restrictions on conflicting roleplay;
- separation of system, user, and retrieved content;
- safe refusal and redirection behavior.

Limits:

- a prompt is still natural-language guidance;
- delimiters help interpretation but do not enforce authorization;
- prompt text can leak;
- secrets must never be stored inside it;
- “ignore attacks” sentences are not a complete defense.

Use provider role fields rather than concatenating user input into the system message.

## Input Guardrails

### Blocklists

Blocklists are fast and inexpensive, but they mainly catch known strings. Paraphrasing, encoding, character substitution, multilingual prompts, and indirect injection can evade them.

### Semantic Classifiers

A classifier can detect intent beyond exact words, but it remains fallible. Adversarial input can target the classifier, and aggressive settings can block legitimate cybersecurity, medical, or research requests.

A practical cascade is:

1. cheap format and pattern checks;
2. normalization and limits;
3. semantic classification;
4. enhanced review for uncertain or high-risk cases.

## Input and Output Coverage

- **Input guardrails** inspect content before it reaches the model.
- **Output guardrails** inspect generated content before it reaches users or downstream systems.
- **Context guardrails** must also inspect retrieved documents, emails, RAG chunks, search results, and tool output.

Filtering only the user's direct message leaves indirect prompt-injection paths exposed.

## Least Privilege

Prompt injection becomes dangerous when the model has broad capability. Limit:

- retrieved data to the active user's permissions;
- tools to the minimum required set;
- credentials to short-lived, scoped tokens;
- write operations separately from read operations;
- destinations and parameters through allowlists;
- high-impact actions through explicit approval;
- code and file operations through sandboxing.

Authorization must be enforced by trusted application code when the action is executed.

## Improper Output Handling

Model output must never be trusted merely because the model generated it.

| Destination | Secure treatment |
|---|---|
| Browser | Encode/sanitize to prevent XSS |
| Database | Use parameterized queries |
| Shell/interpreter | Avoid direct execution; sandbox and allowlist |
| Function/tool | Validate strict schema and permissions |
| Email/message | Verify recipient and require confirmation |

Output filtering is defense in depth. The receiving system remains responsible for safe handling.

## Monitoring and Rate Limiting

Useful telemetry includes:

- model and prompt version;
- guardrail decisions;
- retrieved source IDs;
- tool requests and approval;
- authorization failures;
- output-validation failures;
- token and latency anomalies.

Watch for repeated reformulations, encoded inputs, rapid refusal probing, abnormal tool sequences, unusual data volume, and cross-tenant access attempts.

## Corrections and Nuance

- Model behavior is probabilistic, but access control, authorization, and validation can be deterministic.
- A system prompt is not a security boundary.
- Delimiters are helpful signals, not hard isolation.
- Blocklists have value as a cheap first layer, not as the primary defense.
- Published bypass percentages are specific to particular products, configurations, datasets, and dates; they are not universal.
- Latency thresholds vary by application and should be measured rather than treated as fixed.
- The correct OWASP category for unsafe downstream use of model output is **LLM05:2025 Improper Output Handling**.

## Safe Practical Testing

Use:

- synthetic secrets;
- canary values;
- mock tools;
- controlled identities;
- isolated tenants;
- repeated trials;
- documented stop conditions.

Do not cause real data disclosure, code execution, financial changes, or messages to external recipients.

## Related Permanent Notes

- [Prompt Defence](../../../knowledge-base/ai-security/prompt-defence.md)
- [Prompt Defence Implementation Methodology](../../../pentesting-methodology/ai-security/prompt-defence-implementation.md)
- [Prompt Defence Cheat Sheet](../../../cheatsheets/prompt-defence.md)
- [Prompt Injection](../../../knowledge-base/ai-security/prompt-injection.md)
- [Jailbreaking](../../../knowledge-base/ai-security/jailbreaking.md)

## Room Summary

Prompt security is an architectural problem, not merely a wording problem. Hardened prompts and guardrails reduce attack probability; least privilege, external authorization, strict output handling, approval gates, and monitoring determine whether a bypass becomes a security incident.
