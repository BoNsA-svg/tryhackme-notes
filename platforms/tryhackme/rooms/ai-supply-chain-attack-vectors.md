# TryHackMe Room Notes — AI Supply-Chain Attack Vectors

## Room Purpose

This room examines unsafe model serialization, architecture code, weight-level backdoors, dependency confusion, typosquatting, repository manipulation, prompt-template compromise, and hosted-model API risks.

## Core Rule

~~~text
Untrusted pickle data must be treated as executable code.
~~~

Do not reproduce a live payload merely to prove the vulnerability. Preserve and inspect the artifact safely.

## Safe Pickle Investigation

~~~bash
ls -lh /path/to/model.pkl
file /path/to/model.pkl
sha256sum /path/to/model.pkl
python3 -m pickletools /path/to/model.pkl > model.pickletools.txt
less model.pickletools.txt
~~~

Review references to operating-system, process, network, dynamic-execution, download, and external-host functions.

`GLOBAL`, `STACK_GLOBAL`, and `REDUCE` occur in legitimate files. Context determines danger.

## PyTorch Nuance

The claim that `torch.load()` always executes arbitrary pickle code is too broad. Exposure varies by PyTorch version, loading options, custom classes, calling library, and checkpoint contents. Restricted loading lowers risk but does not prove provenance.

## Architecture and SafeTensors

Executable behavior can live in Keras custom layers, PyTorch classes, remote repository modules, tokenizers, preprocessors, custom operators, or containers.

SafeTensors stores tensor values. It does not convert or sanitize an entire Keras/Python architecture. Loading malicious code separately remains dangerous.

## Weight-Level and GGUF Risk

Backdoor behavior can be encoded into model weights and may survive conversion or quantization. GGUF is not pickle, but its publisher, source model, converter, tokenizer, runtime, hashes, and behavior still require verification.

## Dependency Confusion

Mixed public and private package sources can cause an unintended public package to win resolution.

Review:

- `--extra-index-url`;
- exposed internal names;
- unusual versions;
- missing pins and hashes;
- lock files;
- index mapping;
- build egress.

Do not publish test packages without explicit authorization.

## Typosquatting

Check swapped/repeated letters, missing hyphens, homoglyphs, added “official/release/v2” terms, and fake organization namespaces.

Training examples are illustrative; registry state changes.

## Repository Manipulation

A fake repository may look professional. A compromised legitimate repository is more dangerous because its reputation remains intact.

Check publisher identity, ownership changes, immutable revisions, history, signatures, custom code, dependency additions, adapters, tokenizers, conversions, scans, and warnings.

Downloads and badges are evidence, not guarantees.

## Chained Scenario

The TryTrainMe simulation combines:

1. deceptive repository identity;
2. unsafe pickle artifact;
3. dependency confusion;
4. outbound network activity as detection evidence.

Investigate the complete path:

~~~text
Trust → acquisition → installation → execution
→ persistence/access → detection → containment
~~~

## Hosted-API Vectors

- **Silent update:** record versions and run behavioral regression.
- **API-key compromise:** use scoped secrets, rotation, and usage alerts.
- **Prompt-template compromise:** store reviewed, pinned copies internally.
- **Upstream poisoning:** use provider assurance, independent testing, human review, monitoring, and an exit strategy.

A leaked API key does not automatically expose historical prompts; impact depends on provider capabilities and permissions.

## Corrected Vector Summary

| Vector | Trigger | Detection |
|---|---|---|
| Unsafe pickle | Load | Disassembly and sandbox telemetry |
| Custom architecture | Load/inference | Source and architecture review |
| Weight backdoor | Trigger | Provenance and behavioral testing |
| Dependency confusion | Resolution | Index/resolver review |
| Typosquatting | Install | Exact publisher/name verification |
| Repository compromise | Update | Identity, history, signatures, diffs |
| Prompt-template compromise | Refresh | Hash, diff, policy regression |
| Hosted-model change | Provider update | Version logging and drift tests |

Avoid fixed detection-difficulty ratings; difficulty depends on visibility, attacker sophistication, and implementation.

## Related Permanent Notes

- [AI Supply-Chain Attack Vectors](../../../knowledge-base/ai-security/ai-supply-chain-attack-vectors.md)
- [Attack-Vector Testing Methodology](../../../pentesting-methodology/ai-security/ai-supply-chain-attack-vector-testing.md)
- [Attack Vectors Cheat Sheet](../../../cheatsheets/ai-supply-chain-attack-vectors.md)
- [Understanding AI Supply Chains](../../../knowledge-base/ai-security/ai-supply-chains.md)
- [AI Supply-Chain Assessment](../../../pentesting-methodology/ai-security/ai-supply-chain-assessment.md)

## Summary

AI supply-chain attacks can combine deceptive trust signals, unsafe artifacts, package-resolution weaknesses, compromised credentials, and remote policy updates. Safe investigation avoids executing unknown content, validates each trust relationship, and reconstructs the complete attack chain.
