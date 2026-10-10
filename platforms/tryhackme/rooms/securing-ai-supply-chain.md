# TryHackMe Room Notes — Securing the AI Supply Chain

## Room Purpose

This room develops layered defenses for AI supply chains: safe serialization, integrity and provenance verification, model scanning, architecture inspection, dependency governance, SBOM generation, sandbox telemetry, and hosted-provider assurance.

## Core Workflow

~~~text
Quarantine → verify source → verify integrity/provenance
→ scan artifact and architecture → audit dependencies
→ sandbox behavior → approve immutable release → monitor
~~~

## Safe Serialization

SafeTensors stores tensor data without pickle-style object reconstruction. It meaningfully reduces load-time code-execution risk.

It does not prove that weights, publisher, adapters, tokenizer, architecture code, converter, dependencies, or behavior are trustworthy.

### Restricted PyTorch Loading

~~~python
state = torch.load("candidate.pt", weights_only=True)
~~~

PyTorch 2.6 uses `weights_only=True` by default when `pickle_module` is not supplied. Pin the runtime and set the option explicitly when supporting multiple versions.

A failure under restricted loading is not permission to retry unrestricted. Investigate the checkpoint and require approval.

## Safe Conversion

Converting a pickle requires reading it, so conversion itself is risky.

1. Quarantine and hash.
2. Verify source and signature.
3. Inspect statically.
4. Try restricted loading.
5. If unavoidable, use a disposable network-isolated sandbox.
6. extract only tensor state.
7. save to a data-only format.
8. hash, sign, scan, and test the result.

Avoid absolute statements such as “SafeTensors loading is always safe”; parser and runtime security still matter.

## Integrity and Provenance

~~~bash
sha256sum model.safetensors
~~~

A matching checksum proves byte equality, not publisher identity. The expected checksum must arrive through a trusted independent channel.

Use signatures, attestations, immutable revisions, transparency logs, and controlled publishing for stronger provenance.

## Model Cards

Review owner, version, license, intended use, data, metrics, limitations, framework, dependencies, hashes, lineage, and disclosure contact.

A sparse card raises risk; a detailed card is still only a claim until validated.

## Adapters and Transformations

LoRA adapters, merges, conversions, quantizations, pruning, distillation, and fine-tunes are independent supply-chain artifacts. Each transformation creates a new artifact requiring hashes, provenance, scanning, evaluation, and approval.

## Static Scanning

### Fickling

~~~bash
fickling suspicious.pkl
fickling --check-safety -p suspicious.pkl
~~~

### ModelScan

~~~bash
modelscan -p suspicious-model
~~~

Output and CLI behavior vary by tool version. Pin versions and interpret actual findings rather than relying only on severity labels.

No scanner guarantees safety.

## Architecture Inspection

Inspect layer types, Lambda/custom layers, custom operators, remote code, tokenizers, preprocessors, and entrypoints.

SafeTensors cannot contain or convert an entire Keras architecture. If executable architecture code is loaded separately, it remains a separate risk.

A Lambda layer is suspicious in an untrusted artifact but may be legitimate; review its implementation and provenance.

## Telemetry and Sandboxing

Monitor imports, process creation, file access, network activity, child processes, credential access, persistence, resource use, and returned object type.

Python audit hooks can provide observability. They are not a security sandbox and malicious Python code may bypass hooks installed at runtime. Real containment requires OS-level isolation.

## Dependency Security

Use exact pins, hashes, lock files, controlled registries, restricted build egress, and publisher verification.

Critical correction: pip does not guarantee that a private `index-url` takes priority over a public `extra-index-url`. Candidates are evaluated together. A private-first configuration alone does not eliminate dependency confusion.

Prefer a controlled proxy with allowlisting or explicit package-source mapping.

### pip-audit

~~~bash
pip-audit -r requirements.txt
~~~

This identifies known vulnerabilities. It is not a malware, provenance, or typosquatting detector.

## SBOM and ML-BOM

~~~bash
syft ./project --exclude './venv/**' -o cyclonedx-json > sbom.json
~~~

SPDX and CycloneDX are both broadly useful. Supplement the software SBOM with model lineage, adapters, datasets, prompts, transformations, hashes, evaluations, and licenses.

Licensing consequences are context-dependent; avoid blanket claims that one dependency automatically forces the entire project to become open source.

## Hosted APIs

Replace file-level controls with:

- provider due diligence;
- approved data classifications;
- version pinning where supported;
- scoped API keys;
- behavior baselines;
- prompt governance;
- aggregator/fallback review;
- human review for high-impact decisions;
- change monitoring and exit planning.

Behavior monitoring is analogous to integrity monitoring only at a conceptual level; it cannot cryptographically prove the underlying model is unchanged.

## Promotion Gate

Suggested registry states:

~~~text
quarantined → under-review → approved/conditional
                         ↘ rejected
approved → deprecated/revoked
~~~

Production should load only immutable artifacts explicitly promoted to an approved state.

## Related Permanent Notes

- [Securing the AI Supply Chain](../../../knowledge-base/ai-security/securing-ai-supply-chain.md)
- [Hardening Methodology](../../../pentesting-methodology/ai-security/ai-supply-chain-hardening.md)
- [Hardening Cheat Sheet](../../../cheatsheets/securing-ai-supply-chain.md)
- [Understanding AI Supply Chains](../../../knowledge-base/ai-security/ai-supply-chains.md)
- [Attack Vectors](../../../knowledge-base/ai-security/ai-supply-chain-attack-vectors.md)

## Room Summary

AI supply-chain security is a promotion and governance process, not a one-time scan. Safe formats remove one attack class; trusted provenance, architecture inspection, dependency control, sandboxing, behavioral evaluation, immutable promotion, monitoring, and revocation cover the rest.
