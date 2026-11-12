# Penetration Testing Foundations and Process

> Unified from TryHackMe and Hack The Box learning material.

## What a Penetration Test Is

A penetration test is an authorized, evidence-driven assessment that identifies weaknesses, validates realistic attack paths, explains business impact, and recommends remediation. It is not simply a race to obtain administrator access or capture a flag.

The process is structured but iterative. New information discovered during exploitation or post-exploitation can require more enumeration, research, or validation.

## Unified Engagement Lifecycle

| Phase | Purpose | Typical outputs |
|---|---|---|
| Pre-engagement | Establish authorization, objectives, scope, safety, and communications | Signed scope, RoE, test plan, contacts |
| Information gathering | Map assets, identities, technologies, and exposed services | Asset and service inventory |
| Vulnerability assessment | Form and prioritize testable weakness hypotheses | Candidate findings and attack paths |
| Exploitation | Safely validate selected weaknesses | Reproducible proof and evidence |
| Post-exploitation | Measure the impact of obtained access | Privilege, data, and trust analysis |
| Lateral movement | Test reachable authorized systems and relationships | Documented attack-path transitions |
| Proof of concept | Package minimal, repeatable evidence | Reproduction steps and artifacts |
| Post-engagement | Report, clean up, brief stakeholders, and support retesting | Final report and remediation plan |

TryHackMe commonly summarizes the technical core as information gathering, enumeration/scanning, exploitation, privilege escalation, and post-exploitation. HTB presents a more engagement-complete eight-stage model. These are compatible: the shorter model describes the technical attack flow, while the longer model includes the contractual, evidentiary, and client-delivery phases.

## Analysis During Assessment

| Analysis type | Security use |
|---|---|
| Descriptive | Record what exists: ports, services, versions, applications, and behavior |
| Diagnostic | Explain why an observation occurred and identify its cause |
| Predictive | Estimate which attack paths or weaknesses are likely to succeed |
| Prescriptive | Recommend actions that reduce risk or eliminate the weakness |

Keep observations separate from assumptions. For example, an open TCP port `2121` is evidence. Guessing that it is FTP is a hypothesis. Connecting safely, reading a banner, or using a tuned probe validates or disproves that hypothesis.

## Testing Perspectives

| Model | Knowledge provided | Main implication |
|---|---|---|
| Black box | Little or no internal knowledge | More time required for discovery and enumeration |
| Gray box | Partial knowledge or standard user access | Efficient, realistic testing of exposed and authenticated paths |
| White box | Extensive architecture, code, or privileged knowledge | Deep coverage of logic, implementation, and controls |

## Rules That Govern Every Phase

- Written authorization and scope override technical opportunity.
- Rules of engagement define permitted tools, rates, techniques, hours, and stop conditions.
- A newly discovered system is not automatically authorized for testing.
- Use the least destructive proof that demonstrates the issue.
- Preserve timestamps, commands, outputs, screenshots, and affected assets.
- Record negative, blocked, and untested results as well as successful ones.
- Remove approved test artifacts and verify cleanup.
- A real penetration test prioritizes coverage, evidence quality, and client risk—not CTF speed.

## From Observation to Validated Finding

```text
Observe -> Ask precise questions -> Form hypothesis -> Research -> Test safely
       -> Reproduce -> Measure impact -> Preserve evidence -> Recommend remediation
```

If the evidence does not support a useful hypothesis, return to information gathering. Enumeration and vulnerability assessment naturally overlap.

## Vulnerability Research

After identifying a product and version:

1. Confirm the fingerprint through more than one signal when possible.
2. Search vendor advisories, CVE/NVD records, Exploit-DB, Packet Storm, Vulners, and reputable research.
3. Read the vulnerability conditions, affected versions, prerequisites, and impact.
4. Review proof-of-concept code before execution.
5. Adapt tests to the observed environment instead of assuming default configuration.
6. Validate with the least risky technique permitted by the RoE.

Scanner and CVE matches are leads, not proof. Configuration, patches, mitigations, architecture, and prerequisites determine whether a weakness is actually exploitable.

## Source Records

- [TryHackMe Junior Penetration Testing](../../platforms/tryhackme/learning-paths/junior-penetration-testing.md)
- [HTB Penetration Testing Process](../../platforms/hack-the-box/penetration-tester-path/penetration-testing-process.md)
- [HTB Pre-Engagement, Information Gathering, and Vulnerability Assessment](../../platforms/hack-the-box/penetration-tester-path/pre-engagement-information-gathering-vulnerability-assessment.md)

## Related References

- [Planning and Scoping Methodology](../../pentesting-methodology/foundations/engagement-planning-and-scoping.md)
- [Reconnaissance Methodology](../../pentesting-methodology/reconnaissance/reconnaissance-methodology.md)
- [Penetration Testing Process Cheat Sheet](../../cheatsheets/pentest-process.md)
