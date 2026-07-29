# Linux Privilege Escalation — Enumeration Notes

## Overview

Privilege escalation is a **process, not a single exploit**. There is **no universal technique** that works on every Linux system. Success depends on the target's configuration, including:

* Kernel version
* Installed software
* Available programming languages
* User permissions
* Scheduled tasks
* Network configuration
* Misconfigurations
* Credentials and passwords

The primary goal is to move from a **low-privileged account** to a **higher-privileged account (usually root)**.

---

# What is Privilege Escalation?

Privilege escalation is the process of exploiting:

* Vulnerabilities
* Misconfigurations
* Design flaws
* Weak permissions

to gain access to resources or privileges that should normally be restricted.

### Typical Progression

```
No Access
      │
      ▼
Initial Foothold (Low-Privilege User)
      │
      ▼
Enumeration
      │
      ▼
Identify Weaknesses
      │
      ▼
Exploit
      │
      ▼
Root Shell
```

---

# Why Privilege Escalation Matters

During real penetration tests, attackers rarely obtain administrator access immediately.

Instead, they usually begin with a standard user account and then escalate privileges to:

* Reset passwords
* Bypass access controls
* Read protected files
* Modify configurations
* Install persistence
* Create or modify privileged users
* Execute administrative commands

---

# Learning Objectives

After this section you should understand how to:

* Enumerate Linux systems manually
* Gather operating system information
* Identify users and permissions
* Discover hidden files
* Enumerate network information
* Find privilege escalation opportunities

---

# Enumeration

## Most Important Phase

Enumeration is arguably the **most important step** in Linux privilege escalation.

Before exploiting anything, understand:

* Operating system
* Kernel version
* Running services
* Installed applications
* Users
* Groups
* Scheduled jobs
* Network interfaces
* File permissions

Without enumeration you're simply guessing.

---

# Operating System Enumeration

---

## hostname

Displays the machine hostname.

```bash
hostname
```

Example:

```
home
```

Useful because hostnames sometimes reveal system roles.

Example:

```
SQL-PROD-01
WEB-SERVER
DC01
MAILSERVER
```

---

## uname

Shows kernel information.

```bash
uname -a
```

Example

```
Linux home 6.8.0-41-generic
```

Important fields

* OS
* Hostname
* Kernel Version

Kernel version is especially valuable when researching privilege escalation vulnerabilities.

---

## /proc/version

Displays kernel build information.

```bash
cat /proc/version
```

Example

```
Linux version 6.8.0-41-generic
```

Also reveals:

* Build machine
* Compiler version
* Linker version

Useful for identifying the exact kernel build.

---

## /etc/issue

Displays operating system version.

```bash
cat /etc/issue
```

Example

```
Ubuntu 22.04.1 LTS
```

Can help identify:

* Distribution
* Version
* Release

---

# Process Enumeration

## ps

Lists running processes.

Basic:

```bash
ps
```

---

### ps aux

Shows:

* All users
* Background processes
* User running each process

```bash
ps aux
```

Useful for spotting:

* Root-owned services
* Suspicious processes
* Software versions

---

### ps axjf

Shows process tree.

```bash
ps axjf
```

Useful for understanding:

* Parent/child relationships
* Services spawned by other services

---

# Cron Jobs

Cron schedules commands to execute automatically.

Common uses:

* Backups
* Maintenance
* Log cleanup

View system cron jobs:

```bash
cat /etc/crontab
```

Example

```
30 2 * * 1 root /home/ubuntu/clear-mail.sh
```

Meaning:

| Field  | Description      |
| ------ | ---------------- |
| 30     | Minute           |
| 2      | Hour             |
| *      | Day              |
| *      | Month            |
| 1      | Monday           |
| root   | Runs as root     |
| Script | Command executed |

If a root cron job executes a writable script, it may become a privilege escalation vector.

Also inspect:

```
/etc/cron.d/
/var/spool/cron/
```

---

# Installed Packages

List installed software:

```bash
dpkg -l
```

Useful for:

* Identifying vulnerable software
* Version comparisons
* Searching exploit databases

---

# User Enumeration

---

## id

Displays user identity.

```bash
id
```

Example

```
uid=1001(john)
gid=1001(john)
groups=100(users)
```

You can also inspect another user:

```bash
id matt
```

Useful for discovering:

* UID
* GID
* Group memberships
* sudo membership

---

## env

Displays environment variables.

```bash
env
```

Useful variables:

```
PATH
HOME
USER
SHELL
SSH_CLIENT
```

The PATH variable may expose interpreters such as:

* Python
* Perl
* Ruby
* GCC

These can sometimes aid privilege escalation.

---

## history

Displays previous shell commands.

```bash
history
```

May reveal:

* Passwords
* Usernames
* Administrative commands
* Interesting files

---

## sudo -l

Lists commands the current user may execute with sudo.

```bash
sudo -l
```

Always one of the first commands to run.

Misconfigured sudo permissions are among the most common privilege escalation vectors.

---

# /etc/passwd

List users.

```bash
cat /etc/passwd
```

Extract usernames only:

```bash
cat /etc/passwd | cut -d ":" -f1
```

Find normal users:

```bash
cat /etc/passwd | grep /home
```

Useful for identifying:

* Human users
* Service accounts
* Potential password reuse targets

---

# Network Enumeration

---

## ifconfig

Displays interfaces.

```bash
ifconfig
```

Modern equivalent:

```bash
ip addr
```

Look for:

* Multiple interfaces
* Docker bridges
* Internal networks
* VPN adapters

These may indicate pivoting opportunities.

---

# netstat

Displays network connections.

---

### All connections

```bash
netstat -a
```

---

### Listening TCP ports

```bash
netstat -lt
```

Shows services waiting for incoming connections.

---

### Network statistics

```bash
netstat -s
```

Useful for troubleshooting.

---

### Active connections

```bash
netstat -tp
```

Displays:

* PID
* Program
* Connection state

---

### Listening services with PIDs

```bash
netstat -tpln
```

Very useful during enumeration.

Shows:

* Listening ports
* Program names
* Process IDs

---

### Interface statistics

```bash
netstat -i
```

---

### Common command

```bash
netstat -ano
```

Options:

* **a** → All sockets
* **n** → Numeric addresses
* **o** → Timers

---

Modern replacement:

```bash
ss -tpl
```

---

# File Enumeration

---

## ls -la

Always include hidden files.

```bash
ls -la
```

Example:

```
.secret.txt
```

Hidden files often contain:

* Credentials
* Keys
* Configuration
* Notes

---

# find

One of the most important Linux enumeration commands.

---

## Find a file

```bash
find /home -name flag1.txt
```

---

## Find directories

```bash
find / -type d -name config
```

---

## World writable files

```bash
find / -type f -perm 0777
```

---

## Executable files

```bash
find / -perm -a=x
```

---

## Files owned by a user

```bash
find /home -user frank
```

---

## Recently modified

```bash
find / -mtime -10
```

Modified within 10 days.

---

## Recently accessed

```bash
find / -atime -10
```

---

## Changed within one hour

```bash
find / -cmin -60
```

---

## Accessed within one hour

```bash
find / -amin -60
```

---

## Large files

```bash
find / -size +100M
```

Useful for locating:

* Archives
* Databases
* Backups

---

## Ignore permission errors

```bash
find / ... 2>/dev/null
```

Redirects errors to `/dev/null`.

---

# World-Writable Directories

Examples:

```bash
find / -writable -type d 2>/dev/null
```

```bash
find / -perm -222 -type d 2>/dev/null
```

```bash
find / -perm -o w -type d 2>/dev/null
```

These locations may allow:

* File replacement
* Script modification
* Privilege escalation

---

# World-Executable Directories

```bash
find / -perm -o x -type d 2>/dev/null
```

---

# Find Development Tools

Search for installed interpreters:

```bash
find / -name python*
```

```bash
find / -name perl*
```

```bash
find / -name gcc*
```

Knowing available languages helps determine which exploitation techniques are possible.

---

# Wildcard Searches

Example:

```bash
find / -name pass*.txt
```

Finds:

```
pass.txt
password.txt
passwords.txt
```

---

# SUID Enumeration

One of the most important enumeration commands.

```bash
find / -perm -u=s -type f 2>/dev/null
```

SUID allows a program to execute with the privileges of its owner instead of the current user.

Misconfigured SUID binaries are a frequent privilege escalation vector.

---

# Essential Enumeration Checklist

## System

```bash
hostname
uname -a
cat /proc/version
cat /etc/issue
dpkg -l
```

---

## Processes

```bash
ps aux
ps axjf
```

---

## Scheduled Tasks

```bash
cat /etc/crontab
ls /etc/cron.d
```

---

## Users

```bash
id
env
history
sudo -l
cat /etc/passwd
```

---

## Networking

```bash
ifconfig
ip addr
netstat -tpln
netstat -ano
ss -tpl
```

---

## Files

```bash
ls -la
find / -perm -u=s -type f 2>/dev/null
find / -writable -type d 2>/dev/null
find / -size +100M
find / -name pass*.txt
```

---

# Key Takeaways

* Privilege escalation depends on **careful enumeration**, not luck.
* Gather as much information as possible before attempting exploitation.
* Focus on the operating system, kernel, users, processes, scheduled tasks, networking, and file permissions.
* Commands like `find`, `ps`, `netstat`/`ss`, `sudo -l`, and `id` are essential tools for discovering potential privilege escalation paths.
* Enumeration provides the context needed to identify and exploit misconfigurations that lead to higher privileges.

---

# Linux Privilege Escalation: Common Misconfigurations (Defensive Study Notes)

## Overview

Privilege escalation often results from **misconfigurations**, not software vulnerabilities. The goal is to enumerate the system thoroughly and identify places where privileged processes, binaries, or services can be influenced by a low-privileged user.

> **Key mindset:** Enumerate first, exploit second.

---

# 1. Sudo Misconfigurations

## What is sudo?

`sudo` allows authorized users to execute specific commands with elevated privileges without granting full root access.

Check your permissions:

```bash
sudo -l
```

Example:

```text
User labuser may run the following commands:

(ALL) NOPASSWD: /bin/cat
```

This tells us:

* User can execute `/bin/cat`
* Runs as root
* No password required (`NOPASSWD`)

---

## Enumeration Checklist

```bash
sudo -l
```

Look for:

* NOPASSWD entries
* Wildcards
* Dangerous binaries
* Environment variables (LD_PRELOAD, SETENV)

---

## GTFOBins

The first place to check after finding sudo permissions is:

**GTFOBins**

[https://gtfobins.github.io/](https://gtfobins.github.io/)

Search the binary you're allowed to execute and look for:

* Sudo
* SUID
* Capabilities

---

## Leveraging Program Functionality

Sometimes a binary isn't directly exploitable.

Instead:

* Read its help page

```bash
binary -h
binary --help
man binary
```

Look for options that:

* Read files
* Load configuration files
* Execute plugins/modules
* Spawn shells

Example:

Apache allows alternate configuration files using:

```
-f
```

A privileged configuration mistake can sometimes expose sensitive files such as `/etc/shadow`.

---

# LD_PRELOAD Misconfiguration

Sometimes sudo preserves the environment:

Example:

```text
env_keep+=LD_PRELOAD
```

If present:

A shared library can be loaded **before** the target program starts.

Potential outcome:

* Custom code executes with elevated privileges.

Typical process:

1. Check sudo configuration
2. Create shared library
3. Compile as `.so`
4. Launch sudo binary using LD_PRELOAD

Modern systems rarely expose this configuration, but it's worth checking.

---

# 2. SUID (Set User ID)

## What is SUID?

Normally:

```
Program runs with YOUR permissions
```

With SUID:

```
Program runs with OWNER'S permissions
```

If owned by root:

```
Runs as root
```

---

## Finding SUID Files

```bash
find / -type f -perm -04000 -ls 2>/dev/null
```

Example output:

```text
/usr/bin/passwd
/usr/bin/sudo
/usr/bin/chsh
/bin/mount
/bin/nano
```

---

## Why It Matters

Some binaries become dangerous when SUID is enabled.

Always compare findings with GTFOBins.

Common examples:

* vim
* nano
* find
* bash
* cp
* less
* more
* tar

---

## Typical Enumeration

```bash
find / -perm -4000 2>/dev/null
```

Then search GTFOBins.

---

## Possible Outcomes

Misconfigured SUID binaries may allow:

* Reading protected files
* Modifying protected files
* Creating privileged users
* Spawning root shells

---

# 3. PATH Hijacking

## What is PATH?

Linux searches directories listed in `$PATH` for executables.

View it:

```bash
echo $PATH
```

Example:

```
/usr/local/bin
/usr/bin
/bin
```

---

## Why It Matters

Poorly written privileged programs may execute commands like:

```c
system("backup");
```

instead of

```c
system("/usr/bin/backup");
```

Linux searches PATH.

If an attacker controls a writable PATH directory:

* Fake executable runs instead.

---

## Enumeration

Check PATH:

```bash
echo $PATH
```

Find writable directories:

```bash
find / -type d -writable 2>/dev/null
```

Questions to ask:

* Is a writable directory inside PATH?
* Can PATH be modified?
* Does a privileged binary call commands without full paths?

---

# 4. Linux Capabilities

## What Are Capabilities?

Capabilities split root privileges into smaller permissions.

Instead of full root:

Examples include:

* Network administration
* Raw sockets
* Changing user IDs

---

## Enumerate Capabilities

```bash
getcap -r / 2>/dev/null
```

Example:

```text
vim = cap_setuid+ep
ping = cap_net_raw+ep
```

---

## Important Capability

```
cap_setuid
```

Allows a program to change its UID.

Misconfigured binaries with this capability may be abused if the application supports scripting or code execution.

---

## Why Capabilities Matter

Capabilities are **not visible** during SUID enumeration.

Always run:

```bash
getcap -r /
```

---

# 5. Cron Jobs

## What is Cron?

Cron automatically runs scheduled tasks.

Many execute as root.

---

## Enumeration

Check:

```bash
cat /etc/crontab
```

Also inspect:

```
/etc/cron.d/
/etc/cron.hourly/
/etc/cron.daily/
/etc/cron.weekly/
/etc/cron.monthly/
/var/spool/cron/crontabs/
```

---

## What to Look For

* Writable scripts
* Missing scripts
* Incorrect permissions
* Relative paths
* PATH manipulation
* Programs using wildcards

---

## Example

```
* * * * * root backup.sh
```

Questions:

* Can I edit `backup.sh`?
* Does it run as root?
* Is the file missing?
* Does cron search PATH?

---

## Common Weaknesses

* World-writable scripts
* Deleted scripts still referenced
* PATH hijacking
* Weak file permissions

---

# 6. NFS Misconfiguration

## What is NFS?

Network File System allows remote directories to be shared across machines.

Configuration:

```bash
cat /etc/exports
```

---

## Dangerous Option

```
no_root_squash
```

Normally:

```
Remote root → nfsnobody
```

With:

```
no_root_squash
```

Remote root remains root.

---

## Enumeration

On target:

```bash
cat /etc/exports
```

From attacker machine:

```bash
showmount -e TARGET_IP
```

Look for:

* Writable shares
* `no_root_squash`

---

## Why It Matters

If both conditions exist:

* Writable share
* no_root_squash

Root-owned files created remotely remain root-owned on the target.

This can allow creation of privileged executables or other dangerous artifacts.

---

# Privilege Escalation Workflow

```
Initial Shell
      │
      ▼
System Enumeration
      │
      ▼
Check sudo permissions
      │
      ▼
Find SUID binaries
      │
      ▼
Enumerate capabilities
      │
      ▼
Inspect cron jobs
      │
      ▼
Check PATH
      │
      ▼
Inspect NFS exports
      │
      ▼
Compare findings with GTFOBins
      │
      ▼
Identify misconfiguration
      │
      ▼
Gain elevated privileges
```

---

# Core Enumeration Commands

## Sudo

```bash
sudo -l
```

## SUID

```bash
find / -perm -4000 2>/dev/null
```

## Capabilities

```bash
getcap -r / 2>/dev/null
```

## PATH

```bash
echo $PATH
```

## Writable directories

```bash
find / -type d -writable 2>/dev/null
```

## Cron

```bash
cat /etc/crontab
```

```bash
ls -la /etc/cron*
```

## NFS

```bash
cat /etc/exports
```

```bash
showmount -e TARGET_IP
```

---

# Key Takeaways

* Always start with **enumeration** before attempting privilege escalation.
* Use `sudo -l` to identify allowed privileged commands.
* Compare SUID binaries and capabilities against **GTFOBins** for known behaviors.
* Check whether privileged programs rely on the **PATH** variable.
* Inspect **cron jobs** for writable scripts, missing files, or weak configurations.
* Review **NFS exports** for dangerous options such as `no_root_squash`.
* Most successful privilege escalations come from **small configuration mistakes**, not dramatic software exploits. Careful enumeration is usually the deciding factor.
