# Securing the AI Supply Chain

> Secure AI delivery requires provenance, safe formats, isolated inspection, dependency control, approval gates, runtime monitoring, and continuous reassessment. No single scanner or file format is sufficient.

## Learning Goals

- Reduce unsafe model-deserialization risk.
- Verify artifact integrity and publisher provenance.
- Scan model files and inspect architecture without executing them.
- Secure dependencies, build pipelines, registries, and prompt assets.
- Generate software and ML bills of materials.
- Govern hosted model providers and behavioral changes.

## 1. Prefer Non-Executable Model Formats

Python pickle is an object-serialization protocol whose reconstruction operations may invoke code. Avoid it for artifacts received across a trust boundary.

SafeTensors stores a validated metadata header and tensor data rather than arbitrary Python objects. This removes the pickle-style object-reconstruction path.

### What SafeTensors Does Not Prove

SafeTensors does not establish that:

- the weights are benign;
- the publisher is authentic;
- the file has not been substituted;
- the tokenizer or adapter is safe;
- architecture code is trustworthy;
- dependencies are uncompromised;
- the runtime parser has no vulnerabilities;
- model behavior contains no hidden trigger.

“Safe format” means a narrower serialization attack surface, not a trusted model.

## 2. Use Restricted PyTorch Loading

For a legacy checkpoint that must be inspected, supported PyTorch versions provide a restricted `weights_only` unpickler.

~~~python
import torch

state = torch.load("candidate.pt", weights_only=True)
~~~

PyTorch 2.6 changed the default to `weights_only=True` when the `pickle_module` argument is not supplied. Pin and verify the runtime version rather than assuming the default.

Restricted loading:

- rejects many arbitrary globals and callables;
- supports common tensor/state-dictionary cases;
- may fail on checkpoints containing custom objects;
- is not a provenance guarantee;
- is not a substitute for quarantine and static analysis.

Never switch to unrestricted loading merely because a checkpoint fails. Investigate why and require explicit approval.

## 3. Convert Legacy Artifacts Safely

Conversion requires reading the original artifact. Therefore, conversion can trigger the very issue it is intended to remove.

Safe workflow:

1. obtain the artifact in quarantine;
2. verify source, revision, hash, and signature;
3. inspect statically;
4. use restricted loading if compatible;
5. if unrestricted loading is unavoidable, use a disposable, network-isolated sandbox with no secrets;
6. extract only the required tensor state;
7. save in a non-executable format;
8. hash, sign, scan, and behavior-test the converted output;
9. destroy the conversion environment.

Do not describe `load_file()` as “always safe.” The format prevents embedded Python instructions, but secure operation still depends on trustworthy libraries, parsers, and system controls.

## 4. Verify Integrity and Provenance

### Hashes

~~~bash
sha256sum model.safetensors
~~~

A hash proves that two byte sequences match. It does not identify the publisher.

A checksum is useful only when the expected value arrives through an independently trusted channel. If an attacker can replace both the model and adjacent checksum file, the comparison proves nothing.

### Digital Signatures and Attestations

Prefer:

- signed releases or commits;
- artifact signatures;
- build attestations;
- immutable repository revisions;
- transparency logs;
- verifiable CI identity;
- documented conversion and quantization lineage.

Signatures establish control of a signing identity, so key protection, revocation, and publisher verification remain necessary.

## 5. Review Model Documentation

A model card should document:

- owner, organization, version, and license;
- intended and prohibited uses;
- base model and adapters;
- training and evaluation data;
- metrics and evaluation method;
- limitations, biases, and safety concerns;
- framework and dependency versions;
- artifact hashes and lineage;
- contact and disclosure process.

A sparse model card increases uncertainty but is not proof of malice. Detailed documentation may also be false; validate it against repository contents and external evidence.

## 6. Treat Every Transformation as a New Artifact

Reassess:

- LoRA and other adapters;
- merged models;
- converted formats;
- quantized outputs;
- pruned or distilled models;
- fine-tuned checkpoints;
- changed tokenizers or chat templates.

Record parent artifacts, tools, versions, commands, builder identity, environment, hashes, and evaluation results.

## 7. Model Acquisition Gate

~~~text
Quarantine → verify source → verify integrity/provenance
→ static scan → architecture review → dependency review
→ sandboxed behavior test → approve/reject → monitor
~~~

Suggested states:

- `quarantined`;
- `under-review`;
- `approved`;
- `conditionally-approved`;
- `rejected`;
- `deprecated`;
- `revoked`.

Production systems should only load immutable artifacts explicitly promoted to an approved state.

## 8. Static Model Scanning

### Fickling

Fickling analyzes pickle bytecode and may identify unsafe imports, calls, or dataflow without normal unpickling.

Common workflow:

~~~bash
fickling suspicious.pkl
fickling --check-safety -p suspicious.pkl
~~~

CLI flags and output vary by version. Pin the scanner version and archive the output. Silence or “no issue” means no known indicator was detected—not that the file is safe.

### ModelScan

ModelScan supports multiple ML artifact formats and reports suspicious operators.

~~~bash
modelscan -p suspicious-model
~~~

Treat severity labels as triage guidance. Review the actual operator, callable, architecture, and business context.

### Native Inspection

Use data-only parsers when possible:

- `pickletools` for pickle opcode review;
- `h5py` for HDF5 structure;
- safe archive listing;
- framework configuration inspection;
- custom-operator and remote-code review.

Use more than one scanner because parsers and detection rules differ.

## 9. Architecture Inspection

Enumerate:

- layer types and names;
- Lambda or custom layers;
- custom operators;
- preprocessing and postprocessing;
- tokenizer and chat-template behavior;
- remote repository code;
- external resources;
- model entrypoints.

A Lambda layer is not automatically malicious. Inspect its implementation and origin.

SafeTensors does not contain an entire Keras model architecture. If architecture code is stored or loaded separately, it needs its own review.

## 10. Runtime Telemetry and Sandboxing

Useful telemetry includes:

- file reads/writes;
- module imports;
- process creation;
- system calls;
- DNS and network connections;
- credential-store access;
- child processes;
- persistence attempts;
- returned object type;
- CPU, memory, and GPU anomalies.

Python audit hooks can add observability, but Python documentation explicitly states that hooks installed from Python are not a security sandbox and can be bypassed by malicious code. Use OS-level isolation: disposable VM/container, non-root identity, read-only mounts, disabled network, resource limits, seccomp or equivalent controls, and host telemetry.

## 11. Dependency Security

### Pin and Lock

Exact pins improve reproducibility:

~~~text
numpy==1.26.4
requests==2.32.3
~~~

Use hashes and lock files for direct and transitive dependencies. A lock file is useful only when installation enforces it.

### Avoid Dependency-Confusion Misconfiguration

pip does not give absolute priority to `index-url` over `extra-index-url`. Candidates from configured locations are considered together and the preferred candidate may win.

Therefore, this is not a complete defense:

~~~ini
[global]
index-url = https://private.example/simple/
extra-index-url = https://pypi.org/simple/
~~~

Prefer:

- one controlled repository/proxy;
- allowlisted upstream packages;
- explicit source mapping;
- internal package namespaces;
- pinned versions and hashes;
- restricted build egress;
- trusted publishing.

### Vulnerability and Malware Review

~~~bash
pip-audit -r requirements.txt
~~~

`pip-audit` identifies known vulnerabilities. It is not a malware scanner and does not prove a package is legitimate.

Review names, publishers, install scripts, source repositories, maintainers, version history, and resolver output separately.

## 12. SBOM and ML-BOM

Generate a standard SBOM for software components:

~~~bash
syft ./project --exclude './venv/**' -o cyclonedx-json > sbom.json
~~~

SPDX and CycloneDX both support security and license use cases; do not reduce the choice to “SPDX only for licenses” and “CycloneDX only for security.”

Supplement the software SBOM with AI-specific inventory:

- model and adapter identifiers;
- hashes and signatures;
- base-model lineage;
- framework/runtime;
- datasets and transformations;
- prompts and policies;
- evaluation results;
- license and usage restrictions;
- build/conversion/quantization provenance.

Licensing obligations depend on the specific licenses, linkage/distribution model, data rights, and jurisdiction. Avoid blanket claims that one copyleft dependency automatically forces an entire project to be open sourced.

## 13. Hosted Provider Security

Perform due diligence on:

- data use, retention, and deletion;
- training opt-out;
- versioned endpoints and deprecation policy;
- security certifications and independent assurance;
- incident notification and disclosure channel;
- regional processing and subprocessors;
- model cards and limitations;
- tenant isolation;
- business continuity and exit plan.

Certifications are evidence, not proof of secure behavior.

### Behavioral Monitoring

Maintain a versioned test suite measuring:

- task correctness;
- structured-output conformance;
- safety/refusal behavior;
- latency and error rate;
- bias and edge cases;
- security-critical decisions;
- tool-call behavior.

Behavior testing is not equivalent to a cryptographic checksum: results are probabilistic and cannot prove the underlying model is unchanged. It is a drift-detection control.

### Prompt Governance

Treat prompts, policies, tool definitions, and evaluation sets as code:

- keep reviewed copies internally;
- version and hash them;
- require change review;
- record deployed versions;
- prohibit unreviewed remote updates;
- run regression and adversarial tests.

## 14. Continuous Assurance

Reassess when:

- model, adapter, tokenizer, or prompt changes;
- a dependency or scanner changes;
- ownership or signing keys change;
- a model is converted, merged, or quantized;
- a provider alias changes;
- new intelligence identifies an affected component;
- runtime anomalies appear.

## Related Notes

- [Understanding AI Supply Chains](ai-supply-chains.md)
- [AI Supply-Chain Attack Vectors](ai-supply-chain-attack-vectors.md)
- [AI Supply-Chain Hardening Methodology](../../pentesting-methodology/ai-security/ai-supply-chain-hardening.md)
- [Securing AI Supply Chain Cheat Sheet](../../cheatsheets/securing-ai-supply-chain.md)
