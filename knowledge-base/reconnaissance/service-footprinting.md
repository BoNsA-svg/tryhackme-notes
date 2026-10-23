# Service Footprinting

> **Methodology context:** Apply through the [Reconnaissance Methodology](../../pentesting-methodology/reconnaissance/reconnaissance-methodology.md).

## Principle

A port number suggests a protocol; it does not prove one. Service footprinting combines Nmap, banners, native clients, configuration knowledge, and manual interaction to determine what a service actually exposes and why it matters.

## Repeatable Workflow

1. Confirm TCP/UDP transport and service identity.
2. Record product, version, hostname/domain, and encryption.
3. Run safe, service-specific discovery.
4. Test anonymous or guest behavior only where permitted.
5. Use discovered credentials with the native client.
6. Enumerate permissions before attempting changes.
7. Correlate files, users, names, and credentials with other services.
8. Preserve minimal evidence and avoid unnecessary data access.

## Protocol Matrix

| Service | Common ports | High-value observations |
|---|---:|---|
| FTP/TFTP | 21/TCP, 69/UDP | Anonymous access, readable/writable files, cleartext auth |
| SMB | 139/445 TCP | Domain/workgroup, dialect, signing, shares, ACLs, users |
| NFS/RPC | 111/2049 | Exports, client restrictions, UID/GID mapping, `no_root_squash` |
| DNS | 53 TCP/UDP | Records, nameservers, recursion, version, zone transfers |
| IMAP/POP3 | 143/993, 110/995 | Banner, TLS, auth methods, folders/mailboxes |
| SNMP | 161 UDP | Version, community/auth model, OIDs, device/process/network data |
| MySQL | 3306 TCP | Version, auth, databases, grants, application data |
| MSSQL | 1433 TCP/1434 UDP | Instance names, version, roles, databases, linked servers |
| Oracle TNS | 1521 TCP | Listener version, SIDs/services, management exposure |
| IPMI | 623 UDP | BMC version, cipher suites, authentication weaknesses |
| SSH | 22 TCP | Version, algorithms, keys, auth methods |
| Rsync | 873 TCP | Modules, anonymous listing, read/write controls |
| R-services | 512–514 TCP | Legacy trust relationships and cleartext access |
| RDP | 3389 TCP/UDP | NLA, TLS, certificate, hostname/domain |
| WinRM | 5985/5986 TCP | HTTP(S), auth schemes, credentialed management |

## Configuration-Driven Reasoning

Understand the server-side controls that shape exposure:

- FTP: anonymous login, write permissions, chroot, passive port ranges.
- SMB: share ACLs, guest mapping, signing, dialect support.
- NFS: export host ranges, `rw`, `root_squash`, UID/GID behavior.
- DNS: recursion, version disclosure, transfer ACLs.
- Mail: plaintext auth, TLS, mailbox permissions.
- SNMP: version and community strings versus SNMPv3.
- Databases: remote binding, anonymous/default accounts, grants, file privileges.
- Remote management: permitted users, trust files, NLA/TLS, management listeners.

## Validation Standard

For each finding, capture:

- The exact command and raw response
- Whether authentication was absent, guest, or credentialed
- The effective permissions—not merely the login result
- Any uncertainty caused by filtering, proxies, or deceptive banners
- The smallest artifact necessary to demonstrate impact

## Defensive Baseline

- Disable unused services and legacy protocols.
- Restrict management interfaces to dedicated networks.
- Require modern encryption and strong authentication.
- Remove anonymous/guest access unless explicitly required.
- Enforce least privilege on shares, exports, databases, and remote management.
- Prevent zone transfers and SNMP access from untrusted networks.
- Require SMB signing where appropriate and retire SMBv1.
- Monitor enumeration patterns and failed authentication.

## Related Notes

- [Nmap Network Enumeration](nmap-network-enumeration.md)
- [Network Security](../network-security/network-security.md)
- [Service Enumeration Cheat Sheet](../../cheatsheets/services.md)

## Source Contributions

- Hack The Box Academy — Footprinting

> Use only in authorized labs, CTFs, and explicitly scoped assessments.
