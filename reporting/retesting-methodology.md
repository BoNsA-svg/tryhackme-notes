# Penetration-Test Re-Testing Methodology

## Purpose

A re-test answers a narrow question:

> Were the findings from the original assessment actually remediated?

It is also called remediation verification or remediation validation. It compares current behavior with the original finding, proof of concept, root cause, and remediation guidance.

A re-test is not a clean bill of health for the entire environment. Its conclusion applies only to the findings and assets explicitly included in the re-test scope.

## Re-Test or Full Reassessment?

| Dimension | Re-test | Full reassessment |
|---|---|---|
| Primary question | Are the known findings fixed? | What vulnerabilities exist now? |
| Scope | Named findings and authorized variants | Full defined attack surface |
| Baseline | Original report and evidence | Current environment treated as a new assessment |
| Trigger | Client reports remediation | Major rewrite, migration, acquisition, redesign, or elapsed time |
| Discovery | Limited to verification | Full discovery and testing |
| Deliverable | Addendum, status update, or re-test report | New pentest report |

Ask what changed since the original assessment. If the architecture or application was substantially rebuilt, recommend a reassessment rather than presenting a narrow re-test as comprehensive assurance.

## Authorization and Pre-Engagement

Treat the re-test as a new authorized engagement, even when performed by the original team.

Confirm:

- new authorization, Statement of Work, and Rules of Engagement;
- exact findings, assets, environments, and instances;
- testing dates and permitted hours;
- production or staging status;
- credentials and roles;
- prohibited techniques and stop conditions;
- data-handling and evidence-delivery requirements;
- expected status labels and deliverable;
- escalation contacts;
- how new observations must be reported.

Original authorization should not be assumed to remain valid.

## Scope Precision

These are different scopes:

- re-test finding F-01;
- re-test the login form;
- re-test all authentication controls.

The authorization must say which one applies.

Variant testing is appropriate when it verifies whether the same documented root cause remains in the authorized component. Testing a newly noticed endpoint, different application area, or new attack path may be new-surface testing and requires written authorization.

## Inputs Required

Before testing, obtain:

- final original report;
- original evidence and proof-of-concept notes;
- affected assets and instances;
- remediation description;
- deployment date and build/version;
- changed files or commit diff, when review is authorized;
- compensating-control documentation;
- formal risk-acceptance records;
- known environmental differences;
- client ticket references.

## Per-Finding Workflow

### 1. Reconstruct the Baseline

Record:

- original finding ID and severity;
- affected component;
- prerequisites;
- original proof;
- root cause;
- original impact;
- recommended remediation.

### 2. Confirm the Current Environment

Verify the authorized target, environment, instance, version/build, access level, and deployment state. A patch in staging does not establish that production is fixed.

### 3. Understand the Claimed Fix

Classify the remediation:

- code correction;
- configuration change;
- vendor patch or upgrade;
- architectural change;
- compensating control;
- risk acceptance.

Do not rely only on the closed ticket or stated version.

### 4. Repeat the Original Proof

Use the same safe proof and record the current result. If it still works, the finding fails.

### 5. Test Root-Cause Variants

If the original proof is blocked, test reasonable variants that exercise the same root cause and remain within scope.

Examples include:

- alternative syntax or encoding;
- equivalent parameter placement;
- case and delimiter changes;
- direct versus expected routed access;
- unauthenticated and authorized low-privilege states;
- all explicitly scoped cluster nodes or instances.

The goal is not unlimited discovery. It is to distinguish a root-cause fix from a signature or symptom block.

### 6. Check for Regression

Confirm that the remediation:

- did not break legitimate behavior;
- did not expose debug data or secrets;
- did not create an obvious bypass in the same control;
- applies consistently to the authorized instances;
- preserves required security controls.

### 7. Assign the Outcome

Use the approved terminology and support it with evidence. See [Re-Test Outcomes and Evidence](retest-outcomes-and-evidence.md).

### 8. Escalate New Observations

If new attack surface is noticed:

1. stop before expanding testing;
2. preserve a minimal, non-invasive observation;
3. notify the client through the agreed channel;
4. document it separately;
5. obtain written authorization before further testing.

### 9. Clean Up

Remove test accounts, data, files, sessions, configuration changes, and other artifacts when authorized. Record anything that remains and assign an owner.

## Verification by Remediation Type

### Code Fix

- repeat the original proof;
- test reasonable same-root-cause variants;
- review the authorized code diff where available;
- verify the dangerous construction was replaced, not merely filtered;
- verify related behavior in the explicitly scoped component.

### Vendor Patch

- verify the target instance and installed version;
- confirm the release addresses the reported vulnerability;
- repeat the original proof;
- check every authorized node or instance;
- do not treat banner/version change alone as proof.

### Configuration Change

- verify the effective runtime configuration;
- test the original access path;
- check restart/reload requirements;
- verify consistency across authorized systems;
- ensure the setting cannot be bypassed through the same scoped interface.

### Architectural Change

- remap the original attack path;
- verify direct and intended routed access;
- test segmentation or trust-boundary enforcement;
- confirm controls apply to every authorized route;
- recommend reassessment when the change materially expands the attack surface.

### Compensating Control

Confirm that the control is deployed, effective, monitored, durable, and formally accepted. Record the root cause as unresolved unless it was actually removed.

## Common False-Pass Patterns

- only replaying the exact original payload;
- trusting a closed ticket;
- checking version banners without retesting behavior;
- testing staging instead of the authorized production target;
- checking one node in a cluster;
- accepting a denylist as a root-cause correction;
- overlooking direct access behind a new proxy or WAF;
- treating a compensating control as full remediation;
- claiming application-wide assurance from a few retested findings;
- allowing new-surface investigation to exceed authorization.

## Completion Criteria

A re-test is complete when every in-scope finding has:

- a verified target and baseline;
- recorded remediation type;
- original-PoC result;
- appropriate root-cause variant testing;
- regression observations;
- a justified outcome;
- comparable current evidence;
- cleanup status;
- a clear next action.
