# Getting Started: Privilege Escalation

> Source: HTB Academy - Penetration Tester Path  
> Purpose: Organized source notes for authorized labs and assessments

## Core Idea

Initial access usually runs as a low-privileged account. Privilege escalation identifies local vulnerabilities, exposed credentials, and misconfigurations that allow movement to another user, `root`, Administrator, or `SYSTEM`.

The process is evidence-driven:

```text
Confirm context -> Enumerate -> Prioritize -> Validate safely -> Verify privileges
```

## Enumeration Resources

- HackTricks Linux and Windows privilege-escalation checklists
- PayloadsAllTheThings Linux and Windows methodologies
- GTFOBins for Unix binaries with security-relevant functionality
- LOLBAS for Windows-native binaries and scripts

Automated tools include:

| Platform | Examples |
|---|---|
| Linux | LinPEAS, LinEnum, linuxprivchecker |
| Windows | WinPEAS, Seatbelt, JAWS |

Automated enumeration can execute many commands, generate telemetry, and trigger endpoint controls. Use it only when permitted, understand what it runs, and manually verify every finding.

## High-Value Areas

### Kernel and Operating-System Vulnerabilities

Record the exact OS and kernel/build version, then research applicable advisories and prerequisites. Kernel exploits can crash or destabilize production systems; test them in a matching lab and obtain explicit approval before any production validation.

### Installed Software

Linux package inventory:

```bash
dpkg -l
```

On Windows, inspect installed applications, service binaries, and `C:\Program Files`. Confirm exact versions and configurations before associating software with a vulnerability.

### User Privileges

Linux sudo rights:

```bash
sudo -l
```

Windows token privileges:

```cmd
whoami /priv
whoami /groups
```

Prioritize unusual delegated rights, passwordless commands, privileged tokens, and writable privileged execution paths.

### Scheduled Execution

Linux locations include:

```text
/etc/crontab
/etc/cron.d/
/var/spool/cron/crontabs/
```

Windows scheduled tasks can be inspected with:

```cmd
schtasks /query /fo LIST /v
```

Look for privileged jobs that execute writable scripts, binaries, or paths. Do not modify production tasks without explicit authorization and a rollback plan.

### Exposed Credentials

Review authorized readable locations such as configuration files, application logs, shell history, PowerShell history, unattended-installation files, and service configuration. Test password reuse only within scope and record where each credential originated.

### SSH Keys

Private keys may exist under:

```text
/home/USER/.ssh/id_rsa
/root/.ssh/id_rsa
```

Restrict a copied private key before use:

```bash
chmod 600 id_rsa
ssh -i id_rsa USER@TARGET
```

Write access to a controlled user's `authorized_keys` can provide SSH access, but adding a key is persistence and therefore requires explicit authorization. Record and remove any test key during cleanup.

## Prioritization

1. Existing delegated privileges and credentials
2. Writable privileged services, tasks, scripts, and paths
3. SUID/SGID, capabilities, token privileges, and service permissions
4. Local-only services and configuration secrets
5. Vulnerable installed software
6. Kernel exploits as a high-risk last resort

## Promoted References

- [Linux Privilege Escalation Knowledge](../../../../knowledge-base/privilege-escalation/linux-privilege-escalation.md)
- [Windows Privilege Escalation Knowledge](../../../../knowledge-base/privilege-escalation/windows-privilege-escalation.md)
- [Linux PrivEsc Cheat Sheet](../../../../cheatsheets/linux-privesc.md)
- [Windows PrivEsc Cheat Sheet](../../../../cheatsheets/windows-privesc.md)

> Use these techniques only in systems you own or have explicit authorization to test.
