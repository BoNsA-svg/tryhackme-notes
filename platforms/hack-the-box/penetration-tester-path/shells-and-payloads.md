# HTB Academy: Shells & Payloads

> Engagement-ready study notes for authorized Hack The Box labs and assessments only. Every later section should add exact commands, direction, verification, troubleshooting, detection considerations, and cleanup when those elements are actually taught.

## Module information

- **HTB module:** Shells & Payloads
- **Penetration Tester path module:** 8
- **Total sections:** 17
- **Current detailed progress:** Section 1 / 17
- **Primary goal:** Understand how payloads produce shell access and how shells become the command-line foothold used for later assessment work.

---

## 1. Shells Jack Us In, Payloads Deliver Us Shells

### Section objective

Understand the difference between a **shell** and a **payload**, why obtaining a shell matters during a penetration test, and how the word *shell* changes slightly across computing, exploitation, and web contexts.

### Core mental model

```text
Enumerate target -> identify weakness -> deliver payload -> establish shell -> perform authorized follow-up work
```

The payload is what performs the intended action. A shell is the command interface obtained when that action successfully grants interactive or command-execution access.

### What is a shell?

A **shell** is a program that accepts instructions from a user and displays textual output from the operating system.

Common examples include:

| Operating environment | Shell examples |
|---|---|
| Linux and Unix-like systems | Bash, Zsh |
| Windows | Command Prompt (`cmd`), PowerShell |

In an ordinary administration context, the user opens a shell locally or through an approved remote-management service. In an exploitation context, a security tester may obtain access to a shell after successfully exploiting a vulnerability or bypassing a security restriction within the authorized scope.

Phrases such as these generally mean that command access was obtained:

- “I caught a shell.”
- “I popped a shell.”
- “I dropped into a shell.”
- “I’m in.”

These phrases do not describe the shell's privilege level, reliability, user identity, or scope. Those conditions still need to be established and documented.

### Why obtain a shell?

A shell exposes the operating system's command-line interface and can allow the tester to interact with system commands and the filesystem. Depending on authorization and privileges, it can support:

- Local system enumeration
- Privilege-escalation assessment
- File transfer
- Network pivoting
- Evidence collection and documentation
- Use of command-line assessment tools
- Persistence testing when explicitly authorized

A shell is normally a **foothold**, not the end of the assessment. The tester must determine what access was obtained, which actions are permitted next, and which actions would exceed the rules of engagement.

### Three shell perspectives

| Perspective | Meaning | Example |
|---|---|---|
| Computing | A text-based environment used to administer a computer and submit instructions | Bash, Zsh, `cmd`, PowerShell |
| Exploitation and security | Command access obtained after exploiting a weakness or bypassing a security control | Triggering a vulnerable Windows service and obtaining Command Prompt access |
| Web | A server-side script or application endpoint that accepts commands and interacts with the underlying host | An uploaded script that reads files or executes commands when requested through a browser |

### Web shell distinction

A web shell differs from an ordinary interactive shell:

- It commonly begins as a server-side script placed through an upload or application vulnerability.
- Commands may be submitted through an HTTP request or browser rather than an interactive terminal.
- Its capabilities depend on the web-server account and application environment.
- It may provide command execution, file access, or destructive capabilities.
- Uploading or executing one is permitted only when the lab or written assessment scope explicitly allows it.

### What is a payload?

The word **payload** has different meanings depending on context:

| Context | Meaning |
|---|---|
| Networking | The data carried inside a packet after protocol headers are excluded |
| Basic computing | The portion of an instruction that identifies the action to perform |
| Programming | Data carried or referenced by an instruction or data structure |
| Exploitation and security | Code designed to perform an action after a vulnerability is triggered, such as establishing authorized shell access in a lab |

Within this module, a payload is primarily the code or instruction set used to establish access or perform a post-exploitation action against an authorized vulnerable system.

### Shell versus payload

| Item | Question it answers |
|---|---|
| Payload | **What code or action is delivered to the target?** |
| Shell | **What command interface or access results from that action?** |

A payload does not always produce a shell. It may perform another action. Likewise, a shell is the resulting access interface, not the delivery code itself.

### Engagement checkpoint after obtaining a shell

Before taking further action, record and confirm:

| Check | Why it matters |
|---|---|
| Target identity | Confirms the shell belongs to the in-scope system |
| Operating system and shell type | Determines which syntax and tools are appropriate |
| Current user and privilege level | Establishes what the session can access |
| Access method | Documents the vulnerability, payload, and connection path |
| Session stability | Identifies whether the shell is interactive, limited, or likely to terminate |
| Allowed next steps | Prevents unauthorized persistence, pivoting, collection, or disruption |
| Evidence timestamp | Supports reproduction and final reporting |

### Operational boundaries

Obtaining shell access does not automatically authorize every possible follow-up action. Treat these as separate scope decisions:

- Privilege escalation
- Credential access
- Persistence
- Pivoting into another network
- Collection or exfiltration of client data
- Destructive commands or service interruption

Use dummy data where possible, minimize access to sensitive information, and preserve a tester activity log.

### Commands and hands-on exercise

Section 1 is conceptual and provides no target, payload command, listener command, or interactive exercise. Do not invent an exploitation workflow from this introduction. Later sections will add commands and labs when HTB introduces the relevant shell type, listener, payload, or delivery technique.

### Verification of understanding

You understand this section when you can explain:

1. A shell accepts commands and returns operating-system output.
2. A payload is the delivered code or action; it may produce a shell.
3. Shell access is a foothold that enables later authorized assessment activities.
4. Computing, exploitation, and web contexts use the word *shell* differently.
5. Obtaining access does not remove the rules-of-engagement boundary.

### Key takeaways

- Shells provide command-line access to the operating system.
- Payloads carry out actions and may deliver shell access.
- A shell is the beginning of post-exploitation work, not automatic permission for every action.
- Command-line proficiency makes assessment work faster and easier to automate.
- Later module sections will provide the concrete listeners, payloads, and shell-handling procedures.

---

## Related notes

- [Original complete-path Shells & Payloads summary](complete-path-study-pack/knowledge-base/08-shells-payloads.md)
- [HTB section engagement runbook template](../../../templates/htb-section-runbook-template.md)

