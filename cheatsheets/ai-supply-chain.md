# AI Supply Chain — Quick Review

> Do not load an unknown model on your normal workstation. Inspect first; execute only inside an isolated disposable environment.

## Supply-Chain Map

```text
Publisher → repository/provider → artifact/API
→ dependencies/data → transform/train/build
→ registry → deployment → runtime
```

## Four Attack Layers

| Layer | Examples |
|---|---|
| Model | Unsafe deserialization, custom code, backdoor, malicious adapter/tokenizer |
| Dependency | Confusion, typosquatting, compromised update, vulnerable transitive package |
| Data | Poisoning, label manipulation, contamination, unauthorized dataset changes |
| Infrastructure | Stolen token, CI/CD compromise, registry replacement, malicious mirror |

## Download vs API

| Downloaded model | Hosted API |
|---|---|
| Artifact crosses your boundary | Provider pipeline stays opaque |
| Serialization and dependency risk | Provider, key, tenancy, and silent-update risk |
| You can hash and sandbox it | You depend on provider version controls |
| You own runtime security | Provider owns inference infrastructure |

## Repository Triage

- Publisher verified independently
- Established history and ownership
- Exact immutable revision recorded
- Model card, limitations, license, and data documented
- Files and loading path inspected
- No unexplained custom code
- No unnecessary `trust_remote_code`
- Dependencies pinned and reviewed
- Hashes/signatures/attestations verified
- Security scan reviewed
- Community warnings checked
- Adapters, tokenizer, and conversions traced

Badges, downloads, and clean scans are signals—not proof.

## Format Signals

| Format | Fast rule |
|---|---|
| Pickle/joblib | Never load untrusted |
| PyTorch checkpoint | Assume unsafe legacy loading until verified |
| SafeTensors | Avoids pickle-style execution; still verify weights and provenance |
| Keras/HDF5 | Inspect custom objects and deserialization path |
| ONNX | Data-oriented; keep runtime patched and inspect graph/provenance |
| GGUF | Not pickle; still verify source, converter, parser, and behavior |

## Static Checks

- Identify actual file type
- Hash every artifact
- Safely list archive contents
- Inspect configs/tokenizers/code
- Review unsafe serialization
- Compare with upstream revision
- Scan binaries and dependencies
- Verify signatures and provenance

## Sandbox Rules

- Disposable VM/container
- No secrets
- No host mounts
- Non-root
- Network denied or tightly controlled
- Read-only model input
- Resource limits
- Process/file/network monitoring
- Destroy after testing

## Dependency Checks

- Direct + transitive inventory
- Lock files and hashes
- Correct registry and publisher
- Install scripts reviewed
- Confusion/typosquat check
- CVE and malware scanning
- Trusted publishing
- Remove unused packages

## Data Checks

- Source, owner, license, consent
- Immutable version and hash
- Modification permissions
- Collection and labeling process
- Contamination and anomaly testing
- Independent evaluation set
- Sensitive-data review
- Transformation lineage

## Hosted-API Checks

- Exact provider/model/endpoint
- Version pinning and change notice
- Data retention/training use
- Tenant and regional controls
- Aggregator/fallback routing
- Key scope, rotation, and spend limits
- Audit and incident terms
- Exit and outage plan

## Approval Outcomes

```text
Approved | Conditional | Rejected | Quarantined | Deprecated
```

Record owner, hashes/version, restrictions, expiry, and retest triggers.

## Never Assume

- Fine-tuning removed a backdoor
- SafeTensors made the model trustworthy
- Quantization preserved provenance
- A verified account cannot be compromised
- A clean scanner result means safe
- An API alias always serves the same model
