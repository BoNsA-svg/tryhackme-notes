# Securing the AI Supply Chain — Quick Reference

## Golden Rule

~~~text
Quarantine → verify provenance → inspect → sandbox-test
→ approve immutable artifact → monitor → reassess
~~~

## Safe Serialization

- Prefer SafeTensors/data-only formats.
- Reject untrusted pickle/joblib.
- Use `torch.load(..., weights_only=True)` for compatible legacy state dictionaries.
- Verify PyTorch version and loader behavior.
- Never disable restricted loading just to make an unknown checkpoint work.
- Convert only in an isolated environment.
- Treat converted output as a new artifact.

## Integrity vs Authenticity

| Control | Proves |
|---|---|
| SHA-256 comparison | File matches the expected bytes |
| Digital signature | Artifact was signed by the holder of a key |
| Attestation | Claims about how/where it was built |
| Model card | Documentation—not cryptographic proof |

Expected hashes must come through a trusted independent channel.

## Static Scanning

~~~bash
sha256sum candidate
python3 -m pickletools candidate.pkl > candidate.pickletools.txt
fickling --check-safety -p candidate.pkl
modelscan -p candidate
~~~

Pin tool versions and archive results. “No issue found” is not proof of safety.

## Architecture Review

- Layer types and names
- Lambda/custom layers
- Custom operators
- Remote code
- Tokenizer/preprocessor
- Adapters
- Container/notebook entrypoints
- External resources
- Comparison with known-good architecture

SafeTensors does not sanitize separately loaded code.

## Sandbox Minimum

- Disposable VM/container
- No secrets
- Non-root
- Read-only input
- No host mounts
- Network disabled/simulated
- Resource limits
- OS-level process/file/DNS/socket/syscall telemetry
- Destroy after testing

Python audit hooks provide telemetry, not containment.

## Dependency Controls

- One controlled proxy/index
- Explicit source policy
- Exact pins and hashes
- Enforced lock file
- Transitive inventory
- Build/install-script review
- CVE and malware checks
- Restricted build egress
- Trusted publishing
- Short-lived credentials

Do not assume private `index-url` beats public `extra-index-url`; pip selects candidates across locations.

## Useful Commands

~~~bash
pip-audit -r requirements.txt
pip-compile --generate-hashes
syft ./project --exclude './venv/**' -o cyclonedx-json > sbom.json
~~~

`pip-audit` finds known vulnerabilities, not all malicious packages.

## ML-BOM Fields

- Model/base/adapters/tokenizer
- Immutable IDs and hashes
- Signatures/attestations
- Datasets and transformations
- Framework/runtime
- Dependencies
- Prompt/policy versions
- Build/conversion/quantization lineage
- Evaluations and limitations
- License and approval owner

## Hosted Provider Gate

- Retention/training-use policy
- Versioned endpoints/change notice
- Tenant and regional controls
- Security evidence
- Incident contact
- Scoped/rotated keys
- Behavior baseline
- Prompt governance
- Aggregator/fallback review
- Exit plan

Behavior monitoring detects drift; it is not a cryptographic checksum.

## Promotion States

~~~text
quarantined → under-review → approved/conditional
                         ↘ rejected
approved → deprecated/revoked
~~~

## Retest Triggers

- Model, adapter, tokenizer, or prompt change
- Conversion, merge, fine-tune, or quantization
- Dependency update
- New scanner/advisory
- Publisher ownership/key change
- Provider alias change
- Runtime anomaly
