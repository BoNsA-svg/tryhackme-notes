# Finding-Writing Standard

## Required Fields

| Field | Requirement |
|---|---|
| Identifier and title | Unique ID plus a concise title naming weakness and location |
| Risk rating | Client matrix or named scoring system, including version/vector when applicable |
| Affected assets | Exact hosts, URLs, endpoints, parameters, roles, or components |
| Summary | Plain-language statement of the weakness and outcome |
| Background | Enough context to understand the control failure |
| Preconditions | Authentication, role, access, timing, or user interaction required |
| Technical details | Reproducible steps and relevant request/response or command evidence |
| Impact | Realistic confidentiality, integrity, availability, financial, or business consequence |
| Likelihood | Exposure, attacker capability, reliability, and required conditions |
| Root cause | The underlying design, implementation, or configuration failure |
| Remediation | Root-cause correction first; defense in depth second |
| Retest criteria | Observable conditions that demonstrate resolution |
| References | Relevant vendor or standards guidance |
| Status | Confirmed, accepted, fixed, partial, not fixed, or unable to retest |

## Title Pattern

Use:

> [Access condition] [weakness] in [specific component]

Examples:

- Unauthenticated SQL Injection in Transaction Search
- Stored Cross-Site Scripting in Customer Feedback
- Standard Users Could Issue Refunds Through Missing Function-Level Authorization

Avoid vague titles such as “Input Validation Issue.”

## Golden Thread

Every section should support the next:

1. Evidence proves the behavior.
2. The behavior reveals a root cause.
3. The root cause enables a realistic impact.
4. The remediation removes that root cause.
5. Retest criteria prove the remediation works.

## Evidence Standard

A reproducible proof normally includes:

1. prerequisite state;
2. exact target;
3. sanitized request, command, or input;
4. relevant response or output;
5. observed security impact;
6. cleanup performed.

Do not include destructive reproduction steps when a safer proof establishes the issue.

## Impact Writing

Use observed facts first, then clearly marked reasonable consequences.

Weak:

> Attackers could steal everything.

Better:

> An unauthenticated attacker was able to bypass the login workflow and access the customer dashboard. If the database account used by the application can read additional customer tables, the same injection point may also expose customer records; that broader access was not attempted because bulk extraction was prohibited.

## Remediation Order

1. Root-cause fix
2. Required implementation detail
3. Defense-in-depth controls
4. Detection and monitoring improvements
5. Retest condition

For SQL injection, parameterized queries address the root cause. Input validation, least-privileged database permissions, and monitoring are additional controls; they are not substitutes for parameterization.

## Safer SQL Remediation Example

~~~csharp
const string query = "SELECT PasswordHash FROM Users WHERE Username = @username";

using var command = new SqlCommand(query, connection);
command.Parameters.Add("@username", SqlDbType.NVarChar, 100).Value = inputUsername;

var storedHash = command.ExecuteScalar() as string;
var authenticated = storedHash is not null &&
                    passwordHasher.Verify(storedHash, inputPassword);
~~~

This separates user input from SQL syntax and keeps password verification out of the query. Use the framework’s approved password-hashing API and migration guidance.

## Retest Criteria Example

The finding is fixed when:

- malicious and malformed values are treated only as data;
- valid authentication still works;
- unauthorized authentication cannot be reproduced;
- equivalent inputs at related login endpoints are not vulnerable;
- database errors and secrets are not exposed.

## Finding Lifecycle

| Status | Meaning |
|---|---|
| Draft | Not yet fully validated or reviewed |
| Confirmed | Reproduced and approved for reporting |
| Risk Accepted | Client formally accepts the residual risk |
| Fixed | Root cause could not be reproduced after remediation |
| Partially Fixed | Original path changed, but residual or equivalent exposure remains |
| Not Fixed | Original issue remains reproducible |
| Unable to Retest | Retest could not be completed; no assurance is implied |
