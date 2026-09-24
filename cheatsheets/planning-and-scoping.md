# Planning and Scoping Cheat Sheet

> Complete before active testing. This is not a replacement for legal review or a signed Rules of Engagement document.

## Engagement Definition

~~~text
Business concern:
Critical assets:
Threat scenario:
Starting position:
Success condition:
Safety boundary:
Required deliverables:
~~~

## Classify the Test

- Type: external / internal / web/API / wireless / cloud / mobile / physical/social
- Knowledge: known / partially known / unknown environment
- Starting position: external / internal / supplied account / assumed breach

## Scope Register

| Asset | Owner | Environment | In/Out | Restrictions | Priority |
|---|---|---|---|---|---|
|  |  |  |  |  |  |

Include IPs, domains, URLs, APIs, mobile builds, cloud IDs/regions, SSIDs/sites, accounts/roles, and third parties.

## Legal Gate

- [ ] NDA
- [ ] MSA
- [ ] SOW
- [ ] Authorization letter
- [ ] DPA where personal-data processing applies
- [ ] Signer has authority over every target

## RoE Gate

- [ ] Start/end date, time zone, and testing windows
- [ ] Blackout, peak, batch, backup, and maintenance periods
- [ ] Primary and emergency contacts
- [ ] Routine, urgent, and critical escalation channels
- [ ] Source IPs, test accounts, and access method
- [ ] Scan rates and automated tools
- [ ] Credential testing and lockout rules
- [ ] Exploitation, privilege escalation, lateral movement, and persistence rules
- [ ] Data access/modification/exfiltration limits
- [ ] DoS/load/stress rules
- [ ] Stop and restart authority

## Evidence Handling

- [ ] Collect minimum proof
- [ ] Encrypt at rest and in transit
- [ ] Restrict access
- [ ] Use approved transfer channel
- [ ] Record retention/destruction deadline
- [ ] Define accidental sensitive-data procedure

## Cloud

- [ ] Account/project/subscription and regions
- [ ] Customer ownership verified
- [ ] Provider policy reviewed immediately before testing
- [ ] Shared infrastructure and other tenants excluded

## Deliverables

- [ ] Status-update frequency
- [ ] Critical-finding notification
- [ ] Draft and factual review
- [ ] Final report deadline
- [ ] Retest scope and window

## Stop Conditions

Stop and escalate for instability, unexpected sensitive data, real compromise indicators, out-of-scope access, third-party impact, or a client cease-testing request.

## Final Question

~~~text
Can every planned action be traced to:
written authorization + an in-scope target + an RoE permission?
~~~

If not, do not perform it.

## Related Notes

- [Full Planning and Scoping Methodology](../pentesting-methodology/foundations/engagement-planning-and-scoping.md)
- [Threat Modelling Cheat Sheet](threat-modelling-for-pentesters.md)

