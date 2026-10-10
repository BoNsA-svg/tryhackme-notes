# Jailbreaking AI Models

> Jailbreaking is the attempt to bypass a model's safety policy or refusal behavior. It is related to prompt injection, but the security question is different.

## Learning Goals

- Explain why aligned models refuse some requests.
- Distinguish jailbreaking from prompt injection.
- Recognize common single-turn and multi-turn jailbreak patterns.
- Evaluate jailbreak resistance safely and reproducibly.
- Separate a model-policy bypass from an application-impact finding.

## Jailbreaking vs Prompt Injection

| Concept | Primary target | Typical objective | Example impact |
|---|---|---|---|
| Jailbreaking | Model or provider safety policy | Make the model produce content it should refuse | Restricted content generation |
| Prompt injection | An LLM application's instruction/data boundary | Make untrusted input override application instructions | Data disclosure, tool misuse, unauthorized actions |

The categories can overlap. A user may inject instructions into an application specifically to bypass its safety policy. Classify the finding by the boundary crossed and the demonstrated impact.

A jailbreak does not automatically grant access to data, tools, or systems. Those consequences exist only when the surrounding application exposes such capabilities.

## What Creates the “Jail”?

Modern AI systems can combine several safety layers:

- supervised safety fine-tuning;
- preference optimization such as RLHF or RLAIF;
- constitutional or rule-guided training;
- system and developer instructions;
- input and output classifiers;
- application policy engines;
- tool permissions and approval gates;
- rate limits, monitoring, and abuse response.

Model refusals are learned behavior, but deployed systems may also use deterministic controls outside the model. Therefore, “the model is only probabilistic” is an incomplete description of the whole system.

## The Helpfulness–Safety Trade-Off

A system that refuses every request is safe but useless. A system that complies with every request is useful but unsafe. Evaluation must measure both:

- **under-refusal:** unsafe requests are answered;
- **over-refusal:** legitimate requests are blocked;
- **consistency:** equivalent requests receive equivalent treatment;
- **robustness:** minor wording or formatting changes do not reverse the decision.

## Common Jailbreak Families

| Family | How it pressures the model | Safe evaluation approach |
|---|---|---|
| Persona or roleplay | Assigns a character supposedly exempt from policy | Ask the persona to reveal a synthetic canary |
| Fictional framing | Recasts a restricted request as a story or simulation | Use harmless prohibited test content |
| Emotional framing | Uses urgency, grief, authority, or sympathy | Keep the target output non-sensitive |
| Obfuscation | Encodes, fragments, translates, or substitutes characters | Transform a harmless canary request |
| Instruction sandwiching | Hides the restricted request among benign tasks | Mix ordinary tasks with one controlled boundary test |
| Authority impersonation | Claims developer, auditor, or emergency authority | Verify the model does not trust claims without real authorization |
| Context dilution | Uses long text to weaken earlier restrictions | Test whether policy remains stable in long contexts |
| Multi-turn escalation | Gradually moves from benign discussion to a disallowed endpoint | Evaluate the complete trajectory, not isolated turns |
| Refusal adaptation | Rephrases after each refusal to locate weak decisions | Record each branch and stop at the agreed limit |

The effectiveness of each family varies by model, version, system prompt, decoding settings, safety stack, language, and conversation history. Success rates from one experiment should not be treated as universal.

## Multi-Turn Jailbreaking

Multi-turn attacks exploit conversational state. A common trajectory is:

1. establish an apparently legitimate context;
2. obtain harmless background information;
3. narrow toward a policy boundary;
4. refer to the model's previous answers as authority;
5. reframe or backtrack after a refusal;
6. request the final restricted output.

Security controls should assess the intent and trajectory of the conversation, not only the latest message. A prior model response must never become authorization for a later action.

## DAN as a Historical Case

“DAN” (“Do Anything Now”) was a family of early roleplay jailbreaks that asked a model to adopt an unrestricted persona. Community variants added dual responses, fictional penalties, token systems, and reminders to remain in character.

Its lasting importance is methodological:

- jailbreaks evolve through iterative testing;
- public prompts become obsolete when models and mitigations change;
- role consistency can conflict with safety behavior;
- defenses must address technique families rather than exact strings.

A historical DAN prompt is not a reliable modern test and should not be treated as a universal bypass.

## Impact Model

Rate impact according to what was actually demonstrated:

| Result | Meaning |
|---|---|
| Policy bypass only | Model produced content outside the defined safety policy |
| Sensitive disclosure | Model exposed confidential prompt, user, or organizational data |
| Unauthorized tool proposal | Model attempted an action but the application blocked execution |
| Unauthorized tool execution | A real side effect occurred without valid authorization |
| Cross-user or persistent impact | The bypass affected other users, stored state, or future sessions |

The last three usually represent application-security failures in addition to model-safety failures.

## Defensive Principles

- Treat model output as untrusted.
- Enforce authorization in code and at every tool boundary.
- Minimize tool permissions and data access.
- Require confirmation for consequential actions.
- Apply input and output safety controls.
- Separate retrieved content from instructions.
- Use structured tool calls with strict schemas.
- Monitor repeated refusals, rapid rephrasing, encoding, and long escalation chains.
- Continuously evaluate after model, prompt, policy, or tool changes.
- Preserve utility tests so stricter defenses do not make the system unusable.

No prompt, classifier, or blocklist is a complete defense by itself.

## Related Notes

- [Prompt Injection](prompt-injection.md)
- [LLM Pentesting](llm-pentesting.md)
- [Jailbreaking Evaluation Methodology](../../pentesting-methodology/ai-security/jailbreaking-evaluation.md)
- [Jailbreaking Cheat Sheet](../../cheatsheets/jailbreaking.md)
- [Prompt Defence](prompt-defence.md)
- [Prompt Defence Implementation Methodology](../../pentesting-methodology/ai-security/prompt-defence-implementation.md)

## Responsible Use

Perform jailbreak evaluation only on systems you own or are explicitly authorized to test. Use synthetic data, mock tools, canary secrets, controlled accounts, and documented stop conditions.
