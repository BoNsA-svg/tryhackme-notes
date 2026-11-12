# TryHackMe Room Notes — Understanding AI Supply Chains

## Room Purpose

This room introduces the trust relationships behind AI systems: models, data, frameworks, dependencies, build infrastructure, repositories, and hosted providers. It explains how attackers compromise upstream components that organizations already trust.

## Learning Objectives

- Understand traditional and AI-specific supply chains.
- Identify the components of an AI/ML deployment.
- Compare downloaded-model and hosted-API risk.
- Recognize model, dependency, data, and infrastructure attacks.
- Review a public model repository before approval.
- Understand why scanners and safe formats are only individual controls.

## Core Idea

```text
Your application inherits risk from everything used to build,
transform, distribute, load, and operate its AI capability.
```

Supply-chain attacks scale because compromising one trusted upstream component can affect many downstream consumers.

## Traditional vs AI Supply Chains

Traditional software relies on packages, build systems, registries, and maintainers. AI adds:

- model weights;
- model architectures and custom layers;
- tokenizers and preprocessing code;
- training and evaluation data;
- adapters such as LoRA;
- model conversion and quantization;
- model registries;
- hosted inference providers.

Log4Shell is best understood as a vulnerable dependency with enormous downstream reach, while SolarWinds is a classic deliberate supply-chain compromise. Both demonstrate inherited upstream risk, but they are not the same incident type.

## AI Supply-Chain Components

| Component | Examples |
|---|---|
| Models | Base weights, architecture, tokenizer, adapters |
| Datasets | Training, evaluation, and retrieval corpora |
| Frameworks | PyTorch, TensorFlow, scikit-learn |
| Dependencies | NumPy, Pillow, tokenizers, transitive packages |
| Infrastructure | CI/CD, registries, artifact storage, build runners |
| Providers | Hosted models, aggregators, API gateways |

## Transfer Learning

Transfer learning and fine-tuning reduce cost, but they inherit the base model's provenance and behavior. Fine-tuning must not be assumed to remove a backdoor; survival depends on the attack and training process.

Adapters add another artifact that must be assessed independently:

```text
Base model + adapter + tokenizer + inference code
```

## Downloaded Models

A downloaded model crosses the organization's trust boundary and runs on local infrastructure.

Risks include:

- unsafe deserialization;
- remote custom code;
- malicious dependencies;
- compromised converters;
- poisoned weights;
- hidden trigger behavior;
- credential and network exposure during loading.

### File-Format Nuance

- Pickle and joblib can execute code during deserialization.
- Many legacy PyTorch checkpoint paths involve pickle and require careful safe-loading controls.
- SafeTensors is designed as a non-executable tensor format.
- Keras/HDF5 models may involve custom objects or unsafe loading paths.
- ONNX and GGUF are not pickle formats, but provenance, parser vulnerabilities, and malicious behavior still matter.

SafeTensors eliminates the pickle-style serialization path; it does not prove that the model, adapter, tokenizer, dependencies, or behavior are safe.

## Hosted APIs

Hosted APIs avoid local model deserialization but introduce different dependencies:

- opaque training and fine-tuning;
- model changes behind an endpoint;
- provider or aggregator compromise;
- API-key theft;
- shared infrastructure;
- data retention and privacy;
- regional and compliance constraints;
- provider availability.

The supply chain still exists; the customer has less direct visibility into it.

## Four Attack Layers

### Model Layer

- serialization payload;
- malicious custom architecture;
- weight-level trigger or backdoor;
- malicious adapter;
- compromised tokenizer or preprocessing.

### Dependency Layer

- dependency confusion;
- typosquatting;
- compromised package publisher;
- malicious or vulnerable transitive package;
- install-time execution.

### Data Layer

- data or label poisoning;
- trigger insertion;
- evaluation manipulation;
- compromised public dataset.

Claims that a particular poisoning percentage always succeeds should not be generalized beyond the exact study conditions.

### Infrastructure Layer

- stolen publisher credentials;
- CI/CD workflow manipulation;
- registry compromise;
- artifact replacement;
- unsigned releases;
- malicious mirrors and converters.

## Incidents Covered

- PyTorch `torchtriton` dependency confusion, 2022.
- Exposed Hugging Face API tokens, reported in 2023.
- Malicious pickle-based Hugging Face models, reported in 2024.
- Ultralytics build and publishing compromise, 2024.
- `@solana/web3.js` maintainer-account compromise, 2024.
- Malformed model artifacts designed to evade scanning, reported in 2025.

Exact counts, losses, affected versions, and timelines should be checked against the original incident reports before reuse.

## Model Repository Review

Check:

1. publisher identity and account history;
2. immutable repository revision;
3. file types and loading method;
4. custom or remote code;
5. dependency names and versions;
6. model card, training data, limitations, and license;
7. security scan results;
8. community warnings;
9. hashes, signatures, and attestations;
10. adapter, tokenizer, conversion, and quantization lineage.

A verified organization, high download count, or clean automated scan is supporting evidence—not approval by itself.

## Safe Evaluation Workflow

```text
Inventory → verify publisher → pin revision → hash artifacts
→ inspect without loading → scan code/dependencies
→ execute only in isolated sandbox → monitor behavior
→ document decision → continuously reassess
```

The sandbox should contain no secrets, have restricted networking and host access, use a non-privileged identity, impose resource limits, and be destroyed after testing.

## Key Takeaways

- Model files may be capable of code execution depending on format and loading path.
- Model, dependency, data, and infrastructure attacks need different controls.
- Transitive dependencies expand the attack surface.
- Automated scanning is necessary but incomplete.
- Compromised trusted identities can defeat reputation-based checks.
- Every conversion, merge, fine-tune, adapter, and quantization creates new provenance requirements.
- Downloaded and API-hosted models have different—not absent—supply chains.

## Related Permanent Notes

- [Understanding AI Supply Chains](../../../knowledge-base/ai-security/ai-supply-chains.md)
- [AI Supply-Chain Assessment Methodology](../../../pentesting-methodology/ai-security/ai-supply-chain-assessment.md)
- [AI Supply-Chain Cheat Sheet](../../../cheatsheets/ai-supply-chain.md)
- [AI System Reconnaissance](../../../knowledge-base/ai-security/ai-system-reconnaissance.md)
- [AI Threat Modelling](../../../knowledge-base/ai-security/ai-threat-modelling.md)
