# 🐧 Linux PrivEsc Cheat Sheet

## First Commands

```bash
id
whoami
uname -a
cat /etc/os-release
sudo -l
```

## Users / Groups / Credentials

```bash
cat /etc/passwd
cat /etc/group
find /home -maxdepth 3 -type f 2>/dev/null
find / -name '*.conf' -o -name '*.ini' -o -name '*.env' 2>/dev/null
```

Search for secrets:

```bash
grep -RniE 'pass(word)?|secret|token|key' /home /var/www 2>/dev/null
```

## SSH Keys

```bash
find /home /root -type f \( -name 'id_rsa' -o -name 'id_ed25519' -o -name '*.pem' \) 2>/dev/null
find /home /root -type f -name authorized_keys 2>/dev/null
```

If an authorised lab yields a private key:

```bash
chmod 600 <KEY>
ssh -i <KEY> <USER>@<TARGET_IP>
```

A writable `authorized_keys` can create persistence; never modify it unless explicitly permitted, and document any change.

## SUID / SGID

```bash
find / -perm -4000 -type f 2>/dev/null
find / -perm -2000 -type f 2>/dev/null
```

Check unusual binaries against known privilege-escalation techniques.

## Capabilities

```bash
getcap -r / 2>/dev/null
```

## Cron / Scheduled Jobs

```bash
cat /etc/crontab
ls -la /etc/cron.*
systemctl list-timers --all
```

Look for writable scripts or directories executed by root.

## Writable Files / PATH Issues

```bash
find / -writable -type f 2>/dev/null | head
find / -writable -type d 2>/dev/null | head
printf '%s\n' "$PATH"
```

## Processes / Services

```bash
ps aux
ss -lntup
systemctl --type=service --state=running
```

Internal-only services can be valuable.

## Environment / History / Keys

```bash
env
history
find /home -name '.bash_history' -o -name 'id_rsa' -o -name '*.pem' 2>/dev/null
```

## Quick Decision Tree

```text
sudo -l interesting? → investigate first
SUID/capability binary? → check abuse path
Writable root cron/service? → inspect execution path
Credentials found? → reuse/test locally
Interesting local service? → inspect/port-forward
Old/vulnerable software? → verify exact version/preconditions
Kernel exploit? → last resort after safer paths
```

## Automated Enumeration

If allowed and available:

```bash
./linpeas.sh
```

Automated tools are noisy and can produce false positives. Use their output to prioritize, then verify findings manually. Treat kernel exploits as a last resort: confirm the exact kernel, architecture, prerequisites, stability risk, and engagement permission before use.

Detailed notes: [Linux Privilege Escalation](../knowledge-base/privilege-escalation/linux-privilege-escalation.md)
