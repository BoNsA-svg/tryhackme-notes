# AI Supply-Chain Attack Vectors — Quick Reference

> Never load or install an untrusted artifact on your workstation. Quarantine, hash, inspect statically, then use an authorized disposable sandbox.

## Fast Triage

~~~bash
ls -lh suspicious-artifact
file suspicious-artifact
sha256sum suspicious-artifact
~~~

For suspected pickle:

~~~bash
python3 -m pickletools suspicious.pkl > suspicious.pickletools.txt
less suspicious.pickletools.txt
~~~

Never run `pickle.load()` on an untrusted file.

## Pickle Indicators

Review context around `GLOBAL`, `STACK_GLOBAL`, `REDUCE`, `NEWOBJ`, and `BUILD`.

High-risk references:

- `os.system`, `os.popen`;
- `subprocess`;
- `socket`;
- `eval`, `exec`;
- shell/download utilities;
- external URLs or encoded commands.

Opcodes alone are not proof. Identify the resolved callable and arguments.

## Vector Matrix

| Vector | Trigger | Primary review |
|---|---|---|
| Pickle/deserialization | Load/import | Disassembly and loader review |
| Custom layer/code | Load/inference | Architecture and repository code |
| Weight backdoor | Trigger input | Provenance and behavioral testing |
| Dependency confusion | Resolution/install | Index configuration and lock data |
| Typosquatting | Selection/install | Exact name and publisher |
| Repository compromise | Download/update | Identity, history, signatures, diffs |
| Prompt-template compromise | Template update | Version, hash, diff, regression |
| Silent API update | Provider change | Version IDs and behavior baseline |
| API-key compromise | Credential use | Secret audit and provider logs |

## Dependency Review

- Exact spelling and publisher
- Correct registry
- Private/public source separation
- Pinned version and hash
- Lock file
- Transitive inventory
- Install scripts
- VCS/URL dependencies
- Version jumps
- Restricted build egress

A CVE scanner does not reliably detect malicious packages.

## Repository Review

- Publisher verified independently
- Namespace checked for homoglyphs
- Immutable revision pinned
- Ownership changes reviewed
- File history inspected
- Signatures/hashes/attestations checked
- Custom code reviewed
- Model, adapter, tokenizer, and converter lineage recorded
- Platform warnings reviewed

## SafeTensors Rule

~~~text
SafeTensors reduces serialization-code risk.
It does not validate weights, architecture code, tokenizers,
adapters, dependencies, converters, or publishers.
~~~

## Sandbox Minimum

- Disposable
- No secrets
- No host mounts
- Non-root
- Read-only artifact
- Network blocked/simulated
- Resource limits
- Process/file/DNS/socket monitoring
- Destroy after use

## Hosted API Checks

- Immutable version where possible
- Returned version logged
- Regression set
- Alias/fallback review
- Prompt templates pinned
- Keys scoped and rotated
- Spend/rate alerts
- Retention understood
- Provider changes monitored

## Investigation Flow

~~~text
Quarantine → preserve hashes → inspect without execution
→ verify source/dependencies/prompts → sandbox if authorized
→ correlate telemetry → reconstruct timeline
→ contain/rotate/rebuild → retest
~~~

## Corrections to Remember

- Not every `.pt` or `.bin` has identical loading risk.
- PyTorch risk depends on version and loader configuration.
- SafeTensors does not sanitize architecture code.
- Low download count is not proof of malice.
- A leaked key does not automatically expose historical prompts.
- Code execution is not “medium” merely because it occurs at inference.
