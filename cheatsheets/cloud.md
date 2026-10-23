# Cloud Pentesting Cheat Sheet

> Follow the [full methodology](../pentesting-methodology/cloud/cloud-penetration-testing.md).

## Context

- [ ] Provider and testing policy
- [ ] Account/subscription/project and regions
- [ ] Principal, MFA, roles, and session
- [ ] Production and cost restrictions

## Passive Exposure Discovery

```text
DNS records pointing to cloud-provider domains
Storage URLs in HTML, JavaScript, CSS, and public documents
Search-engine results for company terms on provider storage domains
Public repositories and infrastructure-as-code
Company abbreviations used in bucket/container names
```

Discovery is not authorization. Confirm ownership and scope before listing, downloading, or testing permissions. If a secret or private key is publicly exposed, preserve minimal evidence and follow the engagement's incident-notification process.

## Workflow

1. Identity and organization context
2. Resource/control-plane inventory
3. Public exposure and network paths
4. IAM/resource-policy analysis
5. Secrets, storage, snapshots, logs, and metadata
6. Privilege-path validation
7. Minimal proof and cleanup

| Principal | Action | Resource | Condition | Effective result | Risk |
|---|---|---|---|---|---|
|  |  |  |  |  |  |

Avoid adjacent tenants, bulk data, expensive resources, persistence, and organization-wide changes.

Related knowledge: [Domain, Cloud, and Staff OSINT](../knowledge-base/reconnaissance/domain-cloud-staff-osint.md)

