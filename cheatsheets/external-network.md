# External Network Pentesting Cheat Sheet

> Use only against explicitly authorized targets. Follow the [full methodology](../pentesting-methodology/network/external-network-penetration-testing.md).

## Scope

- [ ] IPs/domains and ownership confirmed
- [ ] Source IP, window, and rate approved
- [ ] TCP/UDP and credential-testing rules recorded
- [ ] Fragile/prohibited services identified

## Workflow

1. Passive ownership and exposure research
2. Host discovery
3. Full authorized TCP and targeted UDP coverage
4. Service-specific enumeration
5. Authentication/configuration testing
6. Manual vulnerability validation
7. Minimal impact proof and cleanup

## Inventory

| Target | Port/protocol | Service | Version confidence | Auth | Priority | Status |
|---|---|---|---|---|---|---|
|  |  |  |  |  |  |  |

## Checks

- DNS, mail, VPN, SSH/RDP, file transfer, databases, and management
- TLS/protocol configuration and cleartext alternatives
- Default/shared credentials only when authorized
- Origin exposure behind CDN/WAF
- Unsupported software with verified preconditions
- Rate limiting, MFA, lockout, and information disclosure

## Evidence

Record UTC time, source IP, target, command/configuration, raw response or packet, expected/actual result, impact, and cleanup.

