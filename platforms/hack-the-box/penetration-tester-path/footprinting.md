# HTB Academy — Footprinting

> Organized source note for the Hack The Box Academy Penetration Tester path.

## Core Idea

Footprinting builds a map of the target before exploitation. The objective is not merely to discover open ports, but to understand infrastructure, exposed services, software behavior, trust relationships, users, privileges, and configuration choices. Every confirmed fact should produce a more precise question.

## Six-Layer Enumeration Model

| Layer | Focus | Typical questions |
|---|---|---|
| 1. Internet presence | Domains, subdomains, netblocks, cloud assets, public records | What belongs to the organization and is in scope? |
| 2. Gateway | Firewalls, VPNs, WAFs, mail gateways, proxies | Which controls shape traffic and visibility? |
| 3. Accessible services | Ports, protocols, products, versions, configuration | What can be interacted with remotely? |
| 4. Processes | Application workflows and service purpose | What business or technical function does the service perform? |
| 5. Privileges | Users, roles, ACLs, trusts, authentication | What can each identity access or change? |
| 6. OS setup | Operating system, patching, local configuration | What host-level paths become relevant after access? |

Enumeration is iterative. New data should send the tester back to earlier layers to expand or correct the asset map.

## Domain and Internet Presence

Useful passive and low-impact sources include:

- WHOIS/RDAP and registrar data
- Public DNS records and authoritative name servers
- Certificate Transparency logs
- Search engines and cached results
- Shodan-style third-party scan data
- Public repositories and source code
- Company websites, documents, and metadata
- Job advertisements and employee profiles

```bash
whois <DOMAIN>
dig <DOMAIN> A
dig <DOMAIN> MX
dig <DOMAIN> TXT
dig <DOMAIN> NS
curl -s 'https://crt.sh/?q=%25.<DOMAIN>&output=json'
```

Correlate domains, hostnames, IP addresses, autonomous systems, mail providers, certificate names, and cloud endpoints. Validate ownership and scope before active probing.

## Cloud Resources

Cloud providers secure their underlying platforms, while customers remain responsible for resource configuration. Publicly reachable object storage, snapshots, artifacts, and static websites can expose documents, source code, credentials, or internal naming.

Discovery signals include:

- DNS records pointing to AWS, Azure, or GCP
- URLs in HTML, JavaScript, CSS, and document metadata
- Search queries containing provider domains and company terms
- Public bucket indexes and third-party bucket-search services
- Organization abbreviations and product names used in storage names

Do not assume a discoverable bucket is authorized for modification. Confirm ownership and scope, minimize data access, and report exposed secrets immediately under the Rules of Engagement.

## Staff and Technology Intelligence

Employee profiles, job postings, conference material, and public repositories can reveal:

- Programming languages and frameworks
- Databases, CI/CD systems, source-control platforms, and cloud providers
- Security products and operational processes
- Email patterns, usernames, project names, and internal terminology
- Hardcoded tokens, keys, configuration files, and development endpoints

Treat this information as leads. Verify technologies through independent evidence instead of assuming every advertised skill is deployed.

## Service Footprinting Workflow

1. Identify the protocol and expected ports.
2. Run targeted version and default-script scans.
3. Interact with the native client or a simple protocol client.
4. Check anonymous/guest access only when permitted.
5. Enumerate configuration, capabilities, shares, records, schemas, or banners.
6. Record credentials or secrets without over-collecting protected data.
7. Connect findings across services and update the attack-path hypothesis.

## FTP and TFTP

FTP uses a control channel on TCP/21 and separate data connections. Active and passive modes differ in which side initiates the data channel. FTP is cleartext unless protected by TLS. TFTP uses UDP, has no built-in authentication, and does not support directory listings.

```bash
nmap -sV -sC -p21 <TARGET>
ftp <TARGET>
```

Check banners, anonymous access, directory listings, readable files, writable locations, passive-mode behavior, and exposed configuration. Avoid uploads unless explicitly authorized.

## SMB and Samba

SMB exposes files, printers, named pipes, and other shared resources. Modern SMB commonly uses TCP/445; older NetBIOS-based communication also uses 137–139. Samba provides SMB services on Unix-like systems.

```bash
nmap -sV -sC -p139,445 <TARGET>
smbclient -L //<TARGET> -N
smbmap -H <TARGET>
rpcclient -U '' -N <TARGET>
```

Record dialects, signing requirements, domain/workgroup, guest access, shares, permissions, users, groups, policies, and readable files.

## NFS

NFS exposes remote filesystems. Authorization frequently relies on client addresses and UID/GID mappings rather than strong user authentication.

```bash
rpcinfo -p <TARGET>
showmount -e <TARGET>
sudo mount -t nfs <TARGET>:/<EXPORT> /mnt/nfs -o nolock
```

Review export options such as `rw`, `no_root_squash`, and broad host ranges. Compare numeric ownership with local identities and unmount cleanly after testing.

## DNS

DNS maps names, addresses, mail routes, service records, and administrative authority. Misconfiguration can expose complete zone contents or internal naming.

```bash
dig @<DNS_SERVER> <DOMAIN> NS
dig @<DNS_SERVER> <DOMAIN> MX
dig @<DNS_SERVER> <DOMAIN> TXT
dig @<DNS_SERVER> <DOMAIN> ANY
dig AXFR @<DNS_SERVER> <DOMAIN>
```

Query specific record types, test zone transfer only against authorized domains, enumerate likely subdomains, and distinguish authoritative answers from cached or recursive responses.

## IMAP and POP3

IMAP manages server-side mailboxes; POP3 primarily retrieves messages. Cleartext and TLS-wrapped variants commonly use 143/993 and 110/995.

```bash
nmap -sV -sC -p110,143,993,995 <TARGET>
openssl s_client -connect <TARGET>:993
openssl s_client -connect <TARGET>:995
```

Identify server software, authentication methods, TLS configuration, mailbox capabilities, and—only with valid authorized credentials—accessible folders and messages.

## SNMP

SNMP exposes structured management data through OIDs and MIBs. SNMPv1/v2c rely on community strings; SNMPv3 adds stronger authentication and encryption.

```bash
nmap -sU -sV -p161 --script snmp-info <TARGET>
snmpwalk -v2c -c <COMMUNITY> <TARGET>
onesixtyone -c <COMMUNITY_LIST> <TARGET>
```

Look for device details, interfaces, routes, running processes, installed software, usernames, and configuration mistakes. Community-string testing must respect lockout, rate, and scope constraints.

## MySQL and MSSQL

Database exposure can disclose product versions, authentication behavior, schemas, users, application data, and server privileges.

```bash
nmap -sV -sC -p3306 <TARGET>
mysql -h <TARGET> -u <USER> -p

nmap -sV -sC -p1433 <TARGET>
impacket-mssqlclient <DOMAIN>/<USER>:<PASSWORD>@<TARGET>
```

After valid access, enumerate databases, tables, users, grants, server roles, linked servers, and configuration. Do not dump entire databases when a minimal proof demonstrates impact.

## Oracle TNS

Oracle TNS commonly listens on TCP/1521 and routes clients to database services identified by service names or SIDs.

```bash
nmap -sV -p1521 --script oracle-tns-version <TARGET>
tnscmd10g version -h <TARGET>
odat all -s <TARGET>
```

Record listener version, services/SIDs, authentication posture, and exposed management features. Old Oracle deployments may retain weak defaults, but credentials must only be tested within scope.

## IPMI

IPMI/BMC interfaces provide out-of-band hardware management and commonly use UDP/623. They are high impact because they can operate independently of the host OS.

```bash
nmap -sU -sV -p623 <TARGET>
```

Identify the IPMI version and cipher/authentication configuration. Hash-retrieval or credential validation requires explicit approval because BMC compromise can grant durable hardware-level control.

## Linux Remote Management

- SSH: TCP/22; inspect algorithms, host keys, authentication methods, and banners.
- Rsync: TCP/873; enumerate modules and access controls.
- R-services: legacy ports 512–514; inspect trusted-host relationships such as `.rhosts` and `hosts.equiv`.

```bash
ssh-audit <TARGET>
rsync --list-only rsync://<TARGET>/
nmap -sV -p512,513,514,873 <TARGET>
```

Legacy trust files and unauthenticated rsync modules can expose data or enable access, but writes should not be attempted without explicit permission.

## Windows Remote Management

- RDP: TCP/UDP 3389; record NLA, TLS, certificate, host, and domain information.
- WinRM: HTTP 5985 and HTTPS 5986; valid credentials may provide remote PowerShell access.
- WMI: management interfaces typically reached through RPC/DCOM or WinRM; useful after credentialed access.

```bash
nmap -sV -p3389,5985,5986 <TARGET>
xfreerdp /v:<TARGET> /u:<USER> /p:<PASSWORD>
evil-winrm -i <TARGET> -u <USER> -p '<PASSWORD>'
```

Use remote-management clients only with authorized credentials and preserve evidence of the account context and permissions.

## Evidence Checklist

- Asset ownership and scope
- Source and timestamp
- Port, protocol, service, version, and confidence
- Authentication methods and encryption
- Anonymous or guest access
- Shares, exports, zones, modules, schemas, or management surfaces
- Users, domains, hostnames, paths, and trust relationships
- Minimal evidence of exposed sensitive data
- Defensive controls and any uncertainty

## Sources

- HTB Academy: Footprinting

> All techniques are limited to authorized assessments, labs, and CTF environments.
