# 🪟 Active Directory Competition Cheat Sheet

## Identify the Domain / DC

```bash
nmap -Pn -sV -p53,88,135,139,389,445,464,636,3268,3269 <IP>
nslookup -type=SRV _ldap._tcp.dc._msdcs.<DOMAIN> <DNS_IP>
```

## SMB

```bash
smbclient -L //<IP> -N
smbmap -H <IP>
nxc smb <IP>
```

With credentials:

```bash
nxc smb <IP> -u '<USER>' -p '<PASS>'
smbclient //<IP>/<SHARE> -U '<DOMAIN>\<USER>%<PASS>'
```

## LDAP / Domain Information

```bash
ldapsearch -x -H ldap://<DC_IP> -s base namingcontexts
```

## Kerberos Username Enumeration

```bash
kerbrute userenum -d <DOMAIN> --dc <DC_IP> users.txt
```

## Password Policy

With valid credentials:

```bash
nxc smb <DC_IP> -u '<USER>' -p '<PASS>' --pass-pol
```

Before any password spraying, know the lockout policy.

## Credential Sources

Check:

```text
SMB shares
Git / Git history
Jenkins / CI logs
configuration files
scripts
internal documentation
service accounts
PowerShell history
```

## Credential Testing

```bash
nxc smb <DC_IP> -u users.txt -p '<PASSWORD>' --continue-on-success
```

Only use password spraying when competition rules permit and lockout risk is understood.

## Once You Have Credentials

```text
Validate SMB
Enumerate shares
Check WinRM
Check LDAP access
Look for additional users/groups
Look for service accounts
Map privilege relationships
Reuse discovered credentials carefully
```

WinRM:

```bash
evil-winrm -i <IP> -u '<USER>' -p '<PASS>'
```

## AD Ports to Remember

| Port | Meaning |
|---:|---|
| 53 | DNS |
| 88 | Kerberos |
| 135 | RPC |
| 389/636 | LDAP/LDAPS |
| 445 | SMB |
| 464 | Kerberos password |
| 3268/3269 | Global Catalog |
| 5985/5986 | WinRM |

## When Stuck

- Re-check SMB shares with every credential set.
- Search files for passwords/tokens/configs.
- Enumerate users and groups.
- Check whether a service account has more access than your current user.
- Look for multiple attack paths instead of forcing one technique.

Detailed notes: [Active Directory](../active-directory/active-directory.md)
