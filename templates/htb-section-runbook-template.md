# HTB Section Engagement Runbook Template

> Use only against systems explicitly authorized by Hack The Box or by the assessment's written rules of engagement.

## Section information

- **Module:** `<MODULE_NAME>`
- **Section:** `<SECTION_NUMBER> - <SECTION_TITLE>`
- **Environment:** `<PWNBOX | VPN WORKSTATION | WINDOWS TARGET | LINUX TARGET>`
- **Objective:** `<WHAT THIS SECTION TEACHES OR ACCOMPLISHES>`

## Why this matters during an engagement

Explain the operational problem this technique solves, when it is useful, and what prerequisite condition makes it appropriate.

## Data flow or action direction

```text
<SOURCE_HOST> --<PROTOCOL OR ACTION>--> <DESTINATION_HOST>
```

| Role | System | Responsibility |
|---|---|---|
| Source | `<HOST>` | Holds or sends the original data |
| Destination | `<HOST>` | Receives or processes the data |
| Listener | `<HOST OR N/A>` | Waits for the connection when required |

## Prerequisites

- Written authorization and target scope confirmed
- Required credentials or session access available
- Necessary tool or interpreter present
- Required port reachable
- Source and destination paths writable
- Sensitive-data handling requirements understood

## Values to collect before starting

| Placeholder | How to obtain it | Example |
|---|---|---|
| `TARGET_IP` | HTB target panel or approved scope | `10.129.x.x` |
| `PWNBOX_IP` | `ip -br addr show tun0` | `10.10.14.x` |
| `TARGET_USER` | Lab instructions | `htb-student` |
| `SOURCE_FILE` | File selected for the exercise | `/tmp/test.txt` |
| `DESTINATION_FILE` | Approved destination path | `/tmp/received.txt` |
| `PORT` | Section instructions or approved listener | `8000` |

## Quick decision table

| Situation | Use | Reason | Avoid when |
|---|---|---|---|
| `<CONDITION>` | `<METHOD>` | `<WHY>` | `<LIMITATION>` |

## Step-by-step procedure

### Step 1: prepare the source

State which host runs the command:

```bash
<COMMAND>
```

Expected result:

```text
<EXPECTED_OUTPUT OR FILE>
```

### Step 2: prepare the receiver or listener

```bash
<COMMAND>
```

Explain every important option:

| Option | Meaning |
|---|---|
| `<OPTION>` | `<EXPLANATION>` |

### Step 3: perform the action

```bash
<COMMAND>
```

Clearly identify:

- Which host runs it
- Which direction the data moves
- Which placeholders must be replaced
- Whether the command creates, overwrites, or deletes anything

### Step 4: verify success

```bash
<VERIFICATION_COMMAND>
```

Verification must check the real outcome, not only the command's exit status. When transferring files, check the filename, size, content or hash as appropriate.

## Hands-on lab workflow

Use harmless dummy content unless the exercise explicitly requires another file.

```bash
echo "HTB authorized practice data" > section-test.txt
```

Document the complete workflow from preparation through verification so it can be repeated during a later engagement.

## Troubleshooting

| Symptom | Likely cause | Check | Resolution |
|---|---|---|---|
| `<ERROR>` | `<CAUSE>` | `<DIAGNOSTIC_COMMAND>` | `<FIX>` |

Do not skip errors silently. Record what failed and which change fixed it.

## Security and operational notes

- Identify whether the protocol encrypts content in transit.
- Identify whether authentication is present.
- State what endpoint, network or command telemetry may be generated.
- Avoid real PII, financial data and unnecessary client secrets.
- Prefer dummy data for control testing.
- Explain any risk of overwriting files, executing uploaded content or exposing a listener.
- Keep credentials and encryption passwords out of commands and notes where practical.

## Cleanup

List the exact cleanup commands, but target only artifacts created by the exercise:

```bash
<STOP_LISTENER_COMMAND>
<REMOVE_SPECIFIC_TEST_FILE_COMMAND>
```

Never use broad destructive paths or wildcards for engagement cleanup.

## Engagement quick reference

```bash
# 1. Prepare
<COMMAND>

# 2. Listen or serve
<COMMAND>

# 3. Transfer or execute
<COMMAND>

# 4. Verify
<COMMAND>

# 5. Clean up
<COMMAND>
```

## Key takeaways

- `<TAKEAWAY_ONE>`
- `<TAKEAWAY_TWO>`
- `<TAKEAWAY_THREE>`

