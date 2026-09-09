# 🪟 Windows PrivEsc Cheat Sheet

## First Commands

```cmd
whoami
whoami /priv
whoami /groups
hostname
systeminfo
```

## Saved Credentials

```cmd
cmdkey /list
```

PowerShell history:

```powershell
type $Env:userprofile\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

Unattended files:

```text
C:\Unattend.xml
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Unattend\Unattend.xml
C:\Windows\System32\sysprep\sysprep.xml
```

## Services

```cmd
sc query
wmic service get name,displayname,pathname,startmode
```

Look for:

```text
Unquoted service paths
Writable service binaries
Writable service directories
Weak service permissions
Services running as SYSTEM
```

## Scheduled Tasks

```cmd
schtasks /query /fo LIST /v
```

Focus on tasks running as Administrator/SYSTEM whose executable or script is writable.

## Interesting Files / Configs

```cmd
dir C:\ /s /b | findstr /i "web.config unattend.xml sysprep.xml .config .ini"
```

Common IIS locations:

```text
C:\inetpub\wwwroot\web.config
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config
```

## Network / Processes

```cmd
tasklist
netstat -ano
ipconfig /all
route print
```

## Quick Decision Tree

```text
Interesting token privilege? → investigate token abuse
Saved credential? → test account context
Writable SYSTEM service/task? → high priority
Config/history password? → test/reuse
Vulnerable installed software? → verify version/preconditions
Internal service? → investigate locally
```

## Automated Enumeration

If allowed and available:

```powershell
.\winPEASx64.exe
```

Use automated output to prioritize and manually verify findings.

Detailed notes: [Windows Privilege Escalation](../privilege-escalation/windows-privilege-escalation.md)
