---
title: Planning and Scoping
platform: TryHackMe
content_type: room
status: completed
topics:
  - penetration-testing
  - pre-engagement
  - scope
  - rules-of-engagement
  - legal
  - compliance
tags:
  - tryhackme
  - methodology
  - planning
---

# Planning and Scoping

## Learning Outcomes

- Explain penetration testing and distinguish it from vulnerability assessment and red teaming.
- Classify testing by target, knowledge level, and starting position.
- Define allowlisted and blocklisted scope.
- Recognize core legal documents and why written authorization matters.
- Build Rules of Engagement covering timing, communication, safety, data, and escalation.
- Identify how compliance and cloud-provider policies influence an engagement.
- Translate a client request into an actionable test plan.

## 1. What a Penetration Test Is

A penetration test is an authorized, controlled simulation of cyber attacks against systems, networks, applications, people, or facilities. It aims to:

- Identify weaknesses
- Validate realistic impact
- Evaluate defensive controls
- Produce actionable remediation guidance

The client’s core questions are:

~~~text
Where are we exposed?
How bad could it get?
What should we do about it?
~~~

Authorization distinguishes professional testing from criminal activity. Simulation means impact is demonstrated without causing unnecessary lasting harm.

## 2. Penetration Testing Lifecycle

PTES describes seven phases:

1. Pre-engagement interactions
2. Intelligence gathering
3. Threat modelling
4. Vulnerability analysis
5. Exploitation
6. Post-exploitation
7. Reporting

Professional testing contains far more planning, analysis, evidence handling, and communication than the public image of “hacking.”

## 3. Penetration Test vs. Vulnerability Assessment

| Characteristic | Vulnerability assessment | Penetration test |
|---|---|---|
| Coverage | Broad | Focused and deep |
| Automation | Scanner-heavy | Predominantly human-led |
| Primary result | Potential weaknesses | Validated attack paths and impact |
| Chaining | Limited | Findings may be chained |
| Frequency | Often frequent | Periodic or after major change |

Neither replaces the other.

## 4. Test Types

- Network: infrastructure, hosts, services, and segmentation
- Web/API: identity, authorization, inputs, sessions, and workflows
- Wireless: access points, authentication, protocols, and isolation
- Cloud: IAM, control plane, workloads, storage, and networks
- Mobile: Android/iOS client plus backend APIs
- Physical: facilities, locks, badges, surveillance, and procedures

## 5. Knowledge Approaches

| Modern term | Traditional term | Tester receives |
|---|---|---|
| Known environment | White box | Extensive documentation, credentials, and possibly source |
| Partially known environment | Gray box | Selected access and partial documentation |
| Unknown environment | Black box | Minimal initial information |

Known-environment testing is appropriate when the client wants maximum coverage rather than maximum attacker realism.

## 6. External vs. Internal

- External asks whether an outside attacker can gain entry.
- Internal assumes a foothold or insider position and asks how far access can spread.

## 7. Pentest vs. Red Team

| Penetration test | Red team |
|---|---|
| Defined technical scope | Organization-wide objective |
| Usually days to weeks | Often weeks to months |
| Finds many weaknesses | Pursues selected realistic objectives |
| Usually known to stakeholders | Detection team may be kept unaware |
| Primary output is vulnerability and impact assurance | Primary output includes detection and response effectiveness |

## 8. Scope

Scope defines what is authorized and prohibited.

Include:

- Networks and addresses
- Domains and applications
- APIs and mobile backends
- Cloud accounts/regions/services
- Wireless networks and locations
- Test users and roles
- Explicitly excluded third parties and fragile systems

Avoid:

- Scope so broad that testing becomes superficial
- Scope so narrow that critical paths are excluded
- Undocumented scope creep

## 9. Legal Documents

| Document | Role |
|---|---|
| NDA | Confidentiality |
| MSA | Overall contractual relationship |
| SOW | Engagement-specific scope, schedule, deliverables, and cost |
| Authorization letter | Explicit permission to conduct testing |
| DPA | Personal-data processing requirements where applicable |

An authorization letter must be signed by someone who has authority over the assets. A client cannot authorize testing against a supplier’s systems.

## 10. Rules of Engagement

The RoE specifies:

- Dates, time zone, windows, and blackout periods
- Contacts and communication channels
- Routine, urgent, and critical escalation
- Permitted/prohibited tools and techniques
- Credential and lockout controls
- Data minimization, encryption, transfer, retention, and destruction
- Stop conditions and restart authorization
- Status and final deliverable schedule

### BrightCart example

| Component | Decision |
|---|---|
| Peak restriction | No disruptive testing weekday 6–10 PM ET |
| Batch restriction | Do not interrupt the nightly 2 AM payment job |
| Critical escalation | Call the VP of Engineering immediately |
| Status | Daily update through the primary contact |
| Prohibited | DoS/DDoS |
| Third party | ShipFast is out of scope |

## 11. Compliance

| Framework | Relevance |
|---|---|
| PCI DSS | Testing of payment environments, accepted methodology, remediation, and retesting |
| HIPAA | Security evaluation and careful ePHI handling; proposed requirements must not be treated as final law |
| GDPR | Security evaluation, minimization, and processor/data-handling obligations |
| SOC 2 | Testing may support assurance over Trust Services Criteria |
| ISO 27001 | Testing supports vulnerability management and secure development controls |

Verify current standards before every engagement.

## 12. BrightCart Engagement Analysis

### Recommended approach

Known-environment testing because BrightCart offers diagrams, architecture, AWS configuration, and role-based accounts.

### In scope

- Customer web application
- Payment REST API
- Internal employee portal
- iOS and Android applications
- AWS production environment in the authorized region/account
- Corporate internal network and approved Wi-Fi

### Out of scope

- ShipFast infrastructure
- Disaster-recovery environment under change freeze
- Denial-of-service testing

### Core test types

- Web/API
- External and internal network
- Cloud
- Mobile
- Wireless, if formally included in the final scope

### Primary business objective

Determine whether an attacker starting from the corporate network or a compromised employee identity can reach the payment database.

### Required preconditions

- NDA before technical information is shared
- Existing MSA confirmed
- Engagement-specific SOW
- Signed authorization letter
- Applicable privacy/data-processing terms
- Final RoE and contacts

### Scheduling

Avoid the client’s peak hours and nightly payment-processing window. Work backward from the audit date to allow technical testing, report review, remediation, and retesting.

## Key Takeaways

1. Authorization must be explicit, asset-specific, and written.
2. Scope and RoE are different: scope says what; RoE says how.
3. Ownership determines who can authorize testing.
4. Knowledge level and starting position are separate design choices.
5. Compliance affects coverage and evidence but does not replace risk-based testing.
6. Critical findings and accidental disruption need immediate escalation paths.
7. A professional test is successful when it answers the client’s risk question safely and reproducibly.

## Related Cyber Notes

- [Engagement Planning and Scoping Methodology](../../../pentesting-methodology/foundations/engagement-planning-and-scoping.md)
- [Threat Modelling](../../../pentesting-methodology/foundations/threat-modelling-for-pentesters.md)
- [Planning and Scoping Cheat Sheet](../../../cheatsheets/planning-and-scoping.md)

