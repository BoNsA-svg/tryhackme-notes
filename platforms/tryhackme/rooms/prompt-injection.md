# TryHackMe — Prompt Injection

> Clean room-specific study note. Reusable material lives in [Prompt Injection](../../../knowledge-base/ai-security/prompt-injection.md), the [testing methodology](../../../pentesting-methodology/ai-security/prompt-injection-testing.md), and the [cheat sheet](../../../cheatsheets/prompt-injection.md).

## Learning Objectives

- Understand how LLM applications assemble context.
- Explain the root cause of prompt injection.
- Distinguish direct and indirect injection.
- Recognize common technique families and ingestion surfaces.
- Connect manipulated model behavior to real application impact.
- Apply the concepts safely in an authorized simulation.

## Inside an LLM Application

The model may receive system or developer instructions, user messages, conversation history, retrieved documents, tool results, and memory.

Providers serialize these sources with role fields and model-specific templates. This helps the model prioritize instructions, but it does not create an unbreakable security compartment.

The correct security assumption is:

> Role hierarchy guides model behavior; application code enforces authorization.

## Chat Templates and Hierarchy

ChatML-style markers are used by some model families, while other models use different templates. Their exact tokens are implementation-specific.

OpenAI's published chain of command broadly prioritizes:

~~~text
Platform → Developer → User → Tool
~~~

Untrusted documents, quoted text, files, and tool outputs should be interpreted as data. Because adherence is probabilistic, sensitive permissions cannot depend solely on that behavior.

## What Prompt Injection Is

Prompt injection occurs when untrusted input changes an LLM application's behavior in an unintended way.

The root cause is not merely “one stream of text.” It is the combination of:

- natural-language instruction following;
- mixed-trust context;
- imperfect instruction/data separation;
- applications trusting generated outputs or tool choices;
- missing deterministic authorization around sensitive operations.

OWASP identifies Prompt Injection as LLM01:2025.

## Direct Injection

The attacker controls the normal input channel.

Technique families include:

- task substitution;
- paraphrased overrides;
- authority impersonation;
- fake conversation history;
- instructions in markup or comments;
- multi-turn shaping;
- payload splitting;
- requests to repeat, transform, or encode hidden context.

A phrase blocklist is insufficient because the same intent can be expressed in many ways.

## Indirect Injection

The instruction is embedded in content the application later processes:

- web pages and search results;
- email and attachments;
- PDFs and documents;
- calendar events;
- RAG knowledge sources;
- source repositories and issues;
- tool/API outputs;
- images and multimodal content;
- memory shared across sessions.

A benign user request such as “summarize my Wednesday meetings” can activate an instruction hidden in a calendar event.

## Prompt Injection vs Related Terms

| Term | Distinction |
|---|---|
| Jailbreaking | Prompt injection aimed at bypassing safety restrictions |
| System-prompt leakage | Disclosure outcome that injection may cause |
| Data poisoning | Corruption of training, retrieval, or memory data |
| Improper output handling | Unsafe downstream use of model output |
| Excessive agency | Excessive model tools or autonomy that amplify impact |

## Why Impact Depends on the Application

A text-only weather bot may suffer output steering or reputation damage. An assistant connected to customer files, email, databases, code execution, or financial tools can create confidentiality, integrity, and authorization failures.

A chatbot agreeing to sell a vehicle for one dollar illustrates manipulated output and weak business integration. It does not mean generated text automatically forms a legally binding transaction.

## Technique Examples

### Paraphrased Override

Replacing recognizable words with synonyms can bypass exact-string filters while preserving the same intent.

### Format-Based Injection

Instructions may be hidden in HTML comments, JSON, YAML, document metadata, or code comments. Test what the model receives, not only what a human interface displays.

### Simulated Dialogue

A fabricated prior conversation can imply that restrictions were already lifted.

### Multi-Turn Shaping

Earlier messages establish behavior that activates later. Conversation history and memory are part of the attack surface.

### Payload Splitting

Separate benign-looking fragments become an instruction only after the application combines messages, fields, chunks, or documents.

## Safe Testing

Use:

- synthetic accounts and data;
- unique canary secrets;
- mock tools and destinations;
- reversible actions;
- repeated trials;
- exact model and prompt versions.

Test the model's response and the surrounding control plane. If the model proposes an unauthorized database action but the database gateway rejects it, the manipulation exists but the security impact is contained.

## Defenses

- Treat external content as untrusted.
- Preserve content provenance.
- Separate content and instructions where supported.
- Enforce authorization outside the model.
- Apply least privilege to tools.
- Validate tool parameters and output formats.
- Require human approval for sensitive actions.
- Restrict network egress.
- Keep secrets out of prompts.
- Monitor context, retrieval, tools, approvals, and policy decisions.
- Test direct, indirect, multi-turn, multimodal, and persistent cases.

No single prompt, delimiter, classifier, or filter eliminates prompt injection.

## Reporting

Report the complete path:

~~~text
Source → Ingestion → Model manipulation → Missing control → Impact
~~~

Record trial counts because model behavior is non-deterministic. Do not score a finding solely because the model printed an alarming sentence.

## Key Takeaways

- Prompt injection is an application-level security problem.
- Direct attacks arrive through user input.
- Indirect attacks arrive through external content.
- System prompts and role markers help but are not access controls.
- Risk grows with sensitive context, tools, autonomy, and user trust.
- Deterministic authorization and least privilege contain the blast radius.
