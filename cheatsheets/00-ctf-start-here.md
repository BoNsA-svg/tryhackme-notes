# 🏁 CTF Start Here

> Fast decision guide for authorized CTFs, labs, and competitions.

## 1. First 5 Minutes

```bash
export IP=<TARGET_IP>
ping -c 2 $IP
nmap -Pn -p- --min-rate 3000 -T4 $IP -oA nmap/all-ports
```

Then scan discovered ports:

```bash
nmap -Pn -sC -sV -p<PORTS> $IP -oA nmap/services
```

Record immediately:

```text
IP:
Hostname/domain:
Open ports:
Web ports:
Interesting services:
Credentials/users found:
Potential attack paths:
```

## 2. Decide Where to Go

```text
Target
  ├─ Web ports? ─────────────→ web.md
  ├─ 88/389/445 + domain? ──→ active-directory.md
  ├─ Login/credentials? ─────→ passwords.md
  ├─ RCE / foothold? ────────→ shells.md
  ├─ Linux shell? ───────────→ linux-privesc.md
  └─ Windows shell? ─────────→ windows-privesc.md
```

## 3. Port → First Thought

| Port | Service | First checks |
|---:|---|---|
| 21 | FTP | anonymous login, files |
| 22 | SSH | usernames, keys, reused creds |
| 25 | SMTP | users, mail info |
| 53 | DNS | records, zone transfer |
| 80/443 | HTTP(S) | source, robots, dirs, vhosts, params |
| 88 | Kerberos | AD/domain, usernames |
| 111 | RPC | NFS/services |
| 139/445 | SMB | shares, users, files |
| 389/636 | LDAP | domain/directory enumeration |
| 2049 | NFS | exports, mount permissions |
| 3306 | MySQL | creds, remote access |
| 3389 | RDP | credentials |
| 5985/5986 | WinRM | credentials → shell |
| 8000/8080/3000 | Web | alternate apps/APIs |

## 4. Competition Loop

```text
Enumerate → Form hypothesis → Test → Record result → Pivot
```

When stuck, ask:

- Did I scan every TCP port?
- Did I inspect every web port separately?
- Did I check virtual hosts/subdomains?
- Did I read page source and JavaScript?
- Did I test discovered credentials everywhere reasonable?
- Did I inspect files, backups, configs, Git history, and shares?
- After foothold, did I enumerate locally before exploiting?

## 5. Useful Pages

- [Recon](recon.md)
- [Web](web.md)
- [Active Directory](active-directory.md)
- [Shells & Listeners](shells.md)
- [File Transfer](file-transfer.md)
- [Linux PrivEsc](linux-privesc.md)
- [Windows PrivEsc](windows-privesc.md)
- [Passwords](passwords.md)

## Golden Rule

**Do not spend 30 minutes attacking one idea without new evidence. Enumerate again and pivot.**
