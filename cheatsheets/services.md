# 🔌 Service Enumeration Quick Reference

> Fast first-pass reference for authorized CTFs and labs. Find the port, run the basic checks, then pivot based on evidence.

## Port → Service → First Action

| Port | Service | First action |
|---:|---|---|
| 21 | FTP | Check anonymous access and files |
| 22 | SSH | Record version; test discovered credentials/keys |
| 25 | SMTP | Identify mail server and possible users |
| 53 | DNS | Query records and test zone transfer |
| 80/443 | HTTP(S) | Fingerprint, source, robots, dirs, vhosts |
| 88 | Kerberos | Identify AD domain/DC and enumerate known usernames |
| 111 | RPCbind | Enumerate RPC/NFS services |
| 139/445 | SMB | Enumerate shares, access, users, files |
| 389/636 | LDAP(S) | Identify directory/domain and permitted queries |
| 2049 | NFS | List exports and inspect mount permissions |
| 3306 | MySQL | Identify version; test authorized/discovered creds |
| 5432 | PostgreSQL | Identify version; test authorized/discovered creds |
| 1433 | MSSQL | Identify version; test authorized/discovered creds |
| 3389 | RDP | Record domain/hostname; use valid discovered creds |
| 5985/5986 | WinRM | Valid Windows creds may provide remote shell |
| 3000/5000/8000/8080/8443 | Alternate web | Treat each as a separate web application/API |

---

## FTP — 21

```bash
nmap -sV -p21 $IP
ftp $IP
```

Try anonymous access when appropriate:

```text
Username: anonymous
Password: anonymous
```

Look for downloadable files, backups, configs, credentials, and writable directories.

---

## SSH — 22

```bash
nmap -sV -p22 $IP
ssh <USER>@$IP
ssh -i <KEY> <USER>@$IP
```

SSH usually becomes valuable after finding a username, password, or private key elsewhere.

---

## SMTP — 25

```bash
nmap -sV -p25 $IP
nc $IP 25
```

Useful information may include hostname, mail software, domains, and—depending on configuration—valid user clues.

---

## DNS — 53

```bash
nmap -sU -sV -p53 $IP
dig @${IP} <DOMAIN> ANY
dig @${IP} <DOMAIN> AXFR
```

Useful SRV records for AD:

```bash
dig @${IP} _ldap._tcp.dc._msdcs.<DOMAIN> SRV
dig @${IP} _kerberos._tcp.<DOMAIN> SRV
```

If names are discovered, add required lab hostnames to `/etc/hosts` when DNS resolution is unavailable.

---

## HTTP / HTTPS — 80, 443, 3000, 5000, 8000, 8080, 8443

```bash
curl -i http://$IP/
curl -k -i https://$IP/
whatweb http://$IP/
```

Always check:

```text
/
/robots.txt
/sitemap.xml
page source
JavaScript
cookies
headers
login forms
API endpoints
virtual hosts
```

Directory discovery:

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://$IP/FUZZ
```

Virtual hosts:

```bash
ffuf -w <WORDLIST> -u http://$IP/ -H 'Host: FUZZ.<DOMAIN>' -fs <BASELINE_SIZE>
```

→ [Web Cheat Sheet](web.md)

---

## SMB — 139 / 445

```bash
nmap -sV -p139,445 $IP
smbclient -L //$IP -N
smbmap -H $IP
```

With credentials:

```bash
smbclient -L //$IP -U '<USER>%<PASSWORD>'
smbmap -H $IP -u '<USER>' -p '<PASSWORD>'
```

Check shares for configs, scripts, backups, documents, usernames, passwords, and keys.

If the host appears domain-joined, pivot to the [Active Directory Cheat Sheet](active-directory.md).

---

## Kerberos — 88

First identify:

```text
Domain
Domain Controller
DNS server
Potential usernames
```

Then use the AD workflow rather than treating Kerberos as an isolated service.

→ [Active Directory Cheat Sheet](active-directory.md)

---

## LDAP / LDAPS — 389 / 636

```bash
nmap -sV -p389,636 $IP
ldapsearch -x -H ldap://$IP -s base
```

If anonymous or credentialed queries are allowed, enumerate the directory within the competition scope.

→ [Active Directory Cheat Sheet](active-directory.md)

---

## RPC / NFS — 111 / 2049

```bash
rpcinfo -p $IP
showmount -e $IP
```

If an export is accessible:

```bash
mkdir -p /tmp/nfs
sudo mount -t nfs $IP:/<EXPORT> /tmp/nfs -o nolock
ls -la /tmp/nfs
```

Look for SSH keys, configs, backups, credentials, scripts, and permission mistakes.

---

## MySQL — 3306

```bash
nmap -sV -p3306 $IP
mysql -h $IP -u <USER> -p
```

After legitimate access:

```sql
SHOW DATABASES;
USE <database>;
SHOW TABLES;
```

Look for application users, password hashes, tokens, configuration data, and clues to other services.

---

## PostgreSQL — 5432

```bash
nmap -sV -p5432 $IP
psql -h $IP -U <USER> -d <DATABASE>
```

Useful commands after access:

```text
\l
\dt
\d <table>
```

---

## MSSQL — 1433

```bash
nmap -sV -p1433 $IP
```

With discovered credentials, enumerate databases and permissions using your available MSSQL client tooling. Record whether the account has elevated database/server roles before attempting anything else.

---

## RDP — 3389

```bash
nmap -sV -p3389 $IP
xfreerdp /v:$IP /u:<USER> /p:<PASSWORD>
```

Use when you already have valid credentials and GUI access is useful.

---

## WinRM — 5985 / 5986

With valid credentials:

```bash
evil-winrm -i $IP -u <USER> -p '<PASSWORD>'
```

A successful WinRM login gives a useful Windows foothold.

→ [Windows PrivEsc](windows-privesc.md)

---

## Unknown Port

```bash
nmap -Pn -sC -sV -p<PORT> $IP
nc -nv $IP <PORT>
curl -i http://$IP:<PORT>/
curl -k -i https://$IP:<PORT>/
```

Ask:

1. What protocol is speaking?
2. What product/version is exposed?
3. Does it require authentication?
4. Is there documentation or a known default path?
5. Does information from another service unlock this one?

## Competition Rule

**Enumerate the service before attacking it. Versions, banners, files, usernames, hostnames, and credentials should drive the next step.**
