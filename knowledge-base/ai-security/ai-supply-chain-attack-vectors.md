# AI Supply-Chain Attack Vectors

> AI supply-chain attacks target artifacts, dependencies, repositories, build systems, prompt assets, and providers that an AI application already trusts.

## Learning Goals

- Explain unsafe model deserialization without executing a payload.
- Distinguish serialization, architecture, and weight-level attacks.
- Identify dependency confusion and typosquatting.
- Evaluate model-repository trust signals.
- Recognize hosted-API and prompt-template supply-chain attacks.
- Safely investigate combined attack chains.

## Unsafe Model Deserialization

Serialization converts an in-memory object into stored data. Deserialization reconstructs it later.

Python's `pickle` format can reconstruct complex objects by importing globals and calling functions. Its `__reduce__` mechanism may specify a callable and arguments that execute during unpickling. Untrusted pickle data must therefore be treated as executable code.

Potential impact includes code execution, credential theft, persistence, reconnaissance, cryptomining, and lateral movement.

Do not create or execute a live payload merely to prove the risk. Static inspection and sandbox telemetry provide safer evidence.

### PyTorch Nuance

PyTorch checkpoints have historically used pickle-backed mechanisms. Exposure depends on the PyTorch version, checkpoint contents, loader configuration, custom classes, and whether restricted weight-only loading is active.

Modern safe-loading controls reduce risk but do not establish trust. Never assume every `.pt`, `.pth`, or `.bin` file has identical behavior.

## Safe Pickle Inspection

Do not use `pickle.load()`, `joblib.load()`, or an unrestricted model loader during initial analysis.

~~~bash
ls -lh suspicious.pkl
file suspicious.pkl
sha256sum suspicious.pkl
python3 -m pickletools suspicious.pkl > suspicious.pickletools.txt
less suspicious.pickletools.txt
~~~

Review the complete disassembly for:

- `os.system`, `os.popen`;
- `subprocess`;
- `socket`;
- `eval`, `exec`, or `compile`;
- shell or download utilities;
- external URLs, domains, IP addresses, or encoded blobs;
- unexpected import paths.

Opcodes such as `GLOBAL`, `STACK_GLOBAL`, and `REDUCE` are not automatically malicious. Legitimate pickles use them too. Risk depends on the resolved callable, arguments, and stack context.

Static disassembly is not proof of safety. Malformed artifacts, wrappers, parser differences, and hidden behavioral logic can complicate analysis.

## Architecture-Level Attacks

Executable behavior may reside in:

- Keras Lambda or custom layers;
- custom PyTorch classes;
- remote repository Python files;
- tokenizers and preprocessors;
- ONNX custom operators;
- code required by `trust_remote_code=True`;
- container startup scripts.

### SafeTensors Correction

SafeTensors stores tensor data and avoids pickle-style executable serialization. It does not store or sanitize an entire executable architecture.

If a deployment separately loads malicious architecture or tokenizer code, replacing weight storage with SafeTensors does not neutralize that code. Severity must follow actual prerequisites and impact; arbitrary code execution is not automatically “medium” because it occurs at inference.

## Weight-Level Attacks

Fine-tuning or poisoning can encode trigger-dependent behavior in weights while ordinary evaluation remains acceptable.

Possible outcomes include targeted misclassification, misinformation, policy bypass, hidden trigger behavior, decision bias, or memorized-data leakage.

Static malware scanning will not reliably detect a behavioral backdoor. Evaluation requires provenance, independent datasets, trigger analysis, behavioral testing, and runtime monitoring.

## GGUF and Local LLMs

GGUF is a data-oriented local-LLM format and is not Python pickle. Risks remain:

- malicious or backdoored weights;
- untrusted quantization or conversion;
- tampered metadata;
- runtime/parser vulnerabilities;
- substituted files;
- compromised tokenizer or chat template;
- fake repository provenance.

Download count and upload date are weak contextual signals. Prefer an independently verified publisher, immutable revision, published hash, reproducible conversion, and self-produced quantization where practical.

## Dependency Confusion

Dependency confusion occurs when an internal package name also exists publicly. A resolver searching multiple sources may select the attacker's package if it is the preferred candidate.

Risk factors:

- `--extra-index-url` with private packages;
- internal names not reserved publicly;
- unusually high public versions;
- unpinned dependencies;
- missing hashes;
- build systems with open registry access;
- installation with powerful credentials.

Safer patterns include a controlled index or proxy, explicit source mapping, pinned versions and hashes, lock files, restricted build egress, trusted publishing, and short-lived identities.

## Typosquatting

Typosquatting uses similar names:

- swapped, missing, or repeated letters;
- removed hyphens;
- Unicode homoglyphs;
- added words such as `official`, `release`, or `v2`;
- fake organization namespaces.

Verify the publisher independently and inspect names character by character. Training examples should not be treated as claims that a package currently exists.

## Repository Manipulation

Attackers may create fake organizations, copy documentation, steal write tokens, replace files, move tags, or add malicious adapters, tokenizers, conversions, and dependencies.

| Signal | Value | Limitation |
|---|---|---|
| Verified publisher | Stronger identity evidence | Account or token can still be compromised |
| Download history | Indicates adoption | Popularity is not security |
| Detailed model card | Supports review | Content may be false or copied |
| Security scan | Finds known patterns | Parser evasion and backdoors remain |
| SafeTensors | Removes pickle-style risk | Does not validate weights or code |
| Established history | Provides context | Legitimate owners can be compromised |

There is no universal rule that a low download count proves malice.

## Hosted-API Attack Vectors

### Silent Model or Policy Updates

A provider may change models, routing, or policies behind an endpoint.

Controls:

- pin immutable versions where available;
- record response model/version identifiers;
- maintain behavioral regression tests;
- review alias upgrades;
- monitor performance and output drift.

### API-Key Compromise

A stolen key may enable unauthorized use or cost abuse. Data exposure depends on the provider's account and retention capabilities; a leaked key does not automatically expose all historical requests.

Use scoped secrets, managers, rotation, revocation, per-environment keys, and usage alerts.

### Prompt-Template Compromise

Prompts, agent policies, workflow files, evaluation sets, and tool descriptions are supply-chain artifacts.

Store reviewed copies internally, pin hashes and versions, require code review, disable unreviewed remote updates, and record the deployed template version.

### Upstream Model or Data Poisoning

Hosted customers cannot statically inspect provider weights. Use provider assurance, independent evaluation, high-impact human review, behavioral monitoring, contractual change controls, and an exit plan.

## Chained Attacks

A realistic campaign may combine:

1. fake or compromised repository identity;
2. unsafe model artifact;
3. confused or typosquatted dependency;
4. compromised prompt template;
5. stolen registry or API credential;
6. command-and-control or exfiltration.

Investigate the complete trust, execution, fallback, and persistence chain instead of stopping at the first indicator.

## Related Notes

- [Understanding AI Supply Chains](ai-supply-chains.md)
- [Attack-Vector Testing Methodology](../../pentesting-methodology/ai-security/ai-supply-chain-attack-vector-testing.md)
- [Attack Vectors Cheat Sheet](../../cheatsheets/ai-supply-chain-attack-vectors.md)
- [AI Supply-Chain Assessment](../../pentesting-methodology/ai-security/ai-supply-chain-assessment.md)
