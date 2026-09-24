# Internal Network Pentesting Cheat Sheet

> Follow the [full methodology](../pentesting-methodology/network/internal-network-penetration-testing.md).

## Starting Context

- [ ] Segment, routes, VPN/jump host, and supplied identity recorded
- [ ] Lockout/spraying and lateral-movement permissions confirmed
- [ ] Critical and prohibited assets identified

## Workflow

1. Local network and identity context
2. Controlled host/service discovery
3. Shares, management, databases, backups, and deployment systems
4. Credential and trust analysis
5. Segmentation validation
6. Approved lateral movement
7. Host/AD privilege paths
8. Evidence and cleanup

## Tracking

| Host | Service | Access | Credential source | Trust/pivot | Priority | Status |
|---|---|---|---|---|---|---|
|  |  |  |  |  |  |  |

Never spray without lockout awareness. Confirm destination scope before every pivot. Avoid persistence and bulk collection.

