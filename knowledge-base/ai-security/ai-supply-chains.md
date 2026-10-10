# Understanding AI Supply Chains

> An AI system inherits risk from every model, dataset, library, adapter, build process, registry, provider, and transformation used to create or operate it.

## Learning Goals

- Map the components and trust relationships in an AI supply chain.
- Distinguish downloadable-model risks from hosted-API risks.
- Recognize model-, dependency-, data-, and infrastructure-layer attacks.
- Evaluate model repositories without trusting popularity or branding alone.
- Understand why one safe file format cannot secure the whole pipeline.

## Supply-Chain Security

A supply-chain compromise targets something the victim already trusts: a dependency, build system, maintainer account, dataset, model repository, or service provider.

Modern applications include direct and transitive dependencies. A developer may choose one package while the installer silently resolves many more. Each upstream component and publisher becomes part of the application's trust chain.

Supply-chain incidents scale because compromising one widely consumed component can affect many downstream systems.

### Useful Distinction

SolarWinds was a deliberate upstream compromise distributed through trusted updates. Log4Shell was a severe vulnerability in a widely used dependency. Both demonstrate inherited upstream risk, but only the former is a supply-chain attack in the strict sense.

## What AI Adds

Traditional software supply chains mainly distribute executable code and configuration. AI systems add:

- trained weights;
- model architectures and custom layers;
- tokenizers and preprocessing code;
- training, fine-tuning, and evaluation datasets;
- LoRA and other adapters;
- prompts and agent templates;
- model conversion and quantization steps;
- model registries and artifact stores;
- hosted inference providers;
- external tools and plugins.

A model can perform its advertised task correctly while also containing unsafe serialization behavior, malicious architecture logic, hidden trigger behavior, or compromised supporting code.

## AI Supply-Chain Components

| Component | Examples | Main trust question |
|---|---|---|
| Models | Weights, architecture, tokenizer, adapters | Who produced and transformed this artifact? |
| Data | Training, evaluation, RAG corpora | Where did it originate and who could modify it? |
| Frameworks | PyTorch, TensorFlow, scikit-learn | Which version and build produced the runtime? |
| Dependencies | NumPy, Pillow, tokenizers | What direct and transitive packages execute? |
| Pipelines | CI/CD, training jobs, converters | Who can alter the build or publishing process? |
| Registries | Hugging Face, PyPI, private registries | How are publishers and artifacts authenticated? |
| Hosted providers | Model APIs and aggregators | What model, policy, tenancy, and update practices are hidden? |

## Two Consumption Paradigms

### Downloaded Models

The organization downloads model artifacts and runs them locally.

Benefits:

- pinning and hashing specific artifacts;
- local inspection;
- control over runtime and network access;
- control over upgrades.

Risks:

- unsafe deserialization;
- malicious custom code;
- dependency installation;
- compromised converters or quantizers;
- poisoned or backdoored weights;
- local credential and network exposure.

Everything loaded or installed crosses the organization's trust boundary.

### Hosted Model APIs

The provider owns the model and inference stack. The customer sends input and receives output.

Benefits:

- no local model deserialization;
- fewer local ML runtime dependencies;
- provider-managed updates and infrastructure.

Risks:

- opaque training and fine-tuning;
- silent or weakly communicated model changes;
- provider or aggregator compromise;
- API-key theft;
- shared-infrastructure and tenant risk;
- retention and privacy uncertainty;
- availability and geographic dependency;
- limited independent verification.

The supply chain has not disappeared; it has moved behind a service boundary.

## Model-Format Risk

Extensions are signals, not complete proof. Inspect the actual artifact and loading path.

| Format/family | Primary concern |
|---|---|
| Python pickle, joblib | Deserialization can execute code; do not load untrusted files |
| PyTorch checkpoints | Many legacy loading paths use pickle; use supported safe-loading controls and trusted artifacts |
| SafeTensors | Data-only format designed to avoid pickle-style code execution |
| Keras/HDF5 | Custom objects and unsafe deserialization paths require scrutiny |
| ONNX | Data-oriented graph format, but runtime/parser vulnerabilities and malicious graph behavior remain possible |
| GGUF | Not pickle-based; provenance, parser safety, transformations, and weight-level behavior still matter |

SafeTensors reduces serialization-level code-execution risk. It does not prove that the weights, tokenizer, adapter, repository, dependencies, or resulting behavior are trustworthy.

## Transfer Learning and Adapters

Fine-tuning can inherit weaknesses from a base model. Whether a backdoor survives depends on the attack, data, training procedure, and evaluation; fine-tuning must not be assumed to remove it.

LoRA and similar adapters add another independent artifact:

```text
Base model + tokenizer + adapter + inference code = deployed behavior
```

Each item needs provenance, integrity checks, compatibility validation, and security evaluation.

## Four Attack Layers

### 1. Model Layer

- malicious serialization payload;
- unsafe custom layer or remote-code loading;
- modified architecture;
- hidden trigger or backdoor behavior;
- malicious tokenizer or preprocessing file;
- substituted adapter or quantized artifact.

### 2. Dependency Layer

- dependency confusion;
- typosquatting;
- compromised maintainer account;
- malicious update;
- vulnerable transitive dependency;
- install-time scripts.

### 3. Data Layer

- training-data poisoning;
- evaluation-set manipulation;
- label corruption;
- trigger insertion;
- poisoned public dataset or mirror;
- unauthorized changes to feature or RAG corpora.

Tiny poisoning-rate results from individual studies are configuration-specific and must not be treated as universal thresholds.

### 4. Infrastructure Layer

- stolen repository or registry token;
- compromised maintainer identity;
- CI/CD workflow manipulation;
- artifact-store exposure;
- build-runner compromise;
- unsigned model replacement;
- malicious mirror or converter;
- unauthorized publication.

Attackers may chain layers—for example, steal a publisher token, replace a model, alter dependencies, and distribute poisoned data under a trusted identity.

## Repository Review Signals

A model page is evidence, not proof.

Review:

- publisher identity and history;
- exact repository and revision;
- signed commits or attestations;
- creation date and unusual ownership changes;
- download history and community reports;
- model card completeness;
- training data and limitations;
- license and acceptable-use terms;
- file types and loading instructions;
- use of `trust_remote_code` or custom code;
- required dependencies;
- platform scan results;
- hashes and signatures;
- external download URLs;
- adapters, tokenizer files, and conversion lineage.

Popularity, verification badges, polished documentation, and clean automated scans reduce uncertainty but cannot establish safety alone.

## Notable Incidents

| Incident | Supply-chain lesson |
|---|---|
| PyTorch `torchtriton` compromise (2022) | Public dependency confusion can override a trusted internal/nightly dependency |
| Exposed Hugging Face tokens (2023) | Leaked write-capable credentials threaten legitimate repositories |
| Malicious Hugging Face model artifacts (2024) | Functional-looking model files can contain unsafe pickle behavior |
| Ultralytics compromise (2024) | Build workflow and publishing-token compromise can poison trusted packages |
| `@solana/web3.js` compromise (2024) | Maintainer credentials can turn a trusted package into a rapid distribution channel |
| Malformed-model scanner evasion reports (2025) | Platform scanning is one control, not a guarantee |

Incident details should be verified against the original incident reports before quoting exact counts, losses, or affected versions.

## Core Principles

- Never deserialize an untrusted model on a workstation or production host.
- Prefer non-executable artifact formats where supported.
- Pin versions, revisions, hashes, and dependencies.
- Isolate evaluation in a disposable sandbox with no secrets and restricted network access.
- Verify publisher identity and artifact provenance.
- Generate and retain an AI/ML bill of materials.
- Separate build, review, approval, and deployment duties.
- Scan artifacts and dependencies with multiple controls.
- Monitor runtime network, file, process, and model behavior.
- Treat every conversion, fine-tune, merge, and quantization as a new artifact.
- Re-evaluate hosted APIs when provider models or policies change.

## Related Notes

- [AI Supply-Chain Assessment Methodology](../../pentesting-methodology/ai-security/ai-supply-chain-assessment.md)
- [AI Supply-Chain Cheat Sheet](../../cheatsheets/ai-supply-chain.md)
- [AI System Reconnaissance](ai-system-reconnaissance.md)
- [AI Threat Modelling](ai-threat-modelling.md)
- [Prompt Defence](prompt-defence.md)
