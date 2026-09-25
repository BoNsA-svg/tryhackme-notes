# Evidence and Appendices

## Evidence Principles

Evidence must be sufficient, minimal, reproducible, and protected.

- Capture only what proves the finding.
- Prefer sanitized text for requests, responses, and commands.
- Use screenshots to add visual context, not as the sole proof.
- Record timestamps, target, identity/role, and relevant versions.
- Mask passwords, tokens, session identifiers, personal data, and unrelated records.
- Keep originals in the approved evidence store when redaction changes presentation.
- Follow the Rules of Engagement for encryption, retention, access, and destruction.

## Evidence Naming

Use a stable pattern:

~~~text
FINDING-ID_asset_step_description.ext
PT-003_api-brightcart-thm_02_unauthorized-response.txt
~~~

Keep finding identifiers consistent across the report, evidence directory, retest notes, and tracking system.

## Assessment Scope Appendix

Record actual coverage, not only the original scope.

| Asset or range | Intended coverage | Actual status | Notes |
|---|---|---|---|
| Example target | Full | Tested | Completed as planned |
| Example target | Full | Partial | One role unavailable |
| Example target | Full | Untested | Stability restriction |
| Third-party service | None | Out of scope | Not owned by client |

Include:

- approved scope changes;
- exclusions;
- blocked or incomplete testing;
- test accounts and roles used;
- important environment constraints;
- assurance limitations;
- recommended follow-up.

## Assessment Artifacts Appendix

Track every meaningful change introduced during testing.

| Artifact | Location | Purpose | Cleanup status | Owner/action |
|---|---|---|---|---|
| Test account | Identity store | Authorization testing | Removed | Verified |
| Uploaded file | Web root | Upload validation | Removal pending | Client to remove |
| Created record | Application DB | Workflow proof | Removed | Verified |
| Configuration change | Test environment | Controlled validation | Reverted | Verified |

Potential artifacts include accounts, files, web shells, scheduled tasks, registry keys, cloud resources, firewall changes, test data, emails, payloads, and persistence mechanisms.

## Cleanup Rules

- Prefer reversible, minimally invasive changes.
- Record an artifact when it is created, not at the end.
- Attempt cleanup within authorization.
- Verify removal.
- Clearly assign unresolved cleanup.
- Escalate any artifact that could create continuing access or harm.

The appendices form the audit trail for coverage, cleanup, retesting, and future assessments.
