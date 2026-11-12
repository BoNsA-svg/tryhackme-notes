# Domain, Cloud, and Staff OSINT

> **Methodology context:** Apply through the [Reconnaissance Methodology](../../pentesting-methodology/reconnaissance/reconnaissance-methodology.md).

## Purpose

Public infrastructure and human-generated information can reveal the technologies, naming conventions, hosting providers, cloud resources, and identities that shape an organization's attack surface. The goal is to build an evidence-backed asset map—not to treat every search result as owned, current, or in scope.

## Domain Intelligence

Collect and correlate:

- WHOIS/RDAP ownership and registrar data
- Authoritative name servers and DNS records
- Certificate Transparency names
- IP addresses, netblocks, and autonomous systems
- Mail providers and SPF/DMARC records
- Historical DNS and cached content
- Third-party service banners

```bash
whois <DOMAIN>
dig <DOMAIN> A
dig <DOMAIN> AAAA
dig <DOMAIN> NS
dig <DOMAIN> MX
dig <DOMAIN> TXT
curl -s 'https://crt.sh/?q=%25.<DOMAIN>&output=json'
```

Certificate names and DNS answers are leads. CDNs, shared hosting, acquisitions, and stale records can produce false ownership assumptions.

## Cloud-Resource Discovery

Public cloud exposure commonly appears through:

- Provider hostnames in DNS
- Static-content URLs in page source and JavaScript
- Object-storage links in indexed documents
- Public bucket or container listings
- Repository configuration and infrastructure-as-code
- Company abbreviations embedded in resource names

Provider patterns include:

| Provider | Common public-storage clues |
|---|---|
| AWS | `s3.amazonaws.com`, regional S3 endpoints, S3 website endpoints |
| Azure | `blob.core.windows.net`, storage-account subdomains |
| GCP | `storage.googleapis.com`, Google Cloud Storage URLs |

Search examples:

```text
site:amazonaws.com "<COMPANY>"
site:blob.core.windows.net "<COMPANY>"
site:storage.googleapis.com "<COMPANY>"
```

Discovery is not authorization. Confirm ownership and Rules of Engagement before listing, downloading, or testing permissions. If exposed credentials or private keys are encountered, stop unnecessary access, preserve minimal evidence, and follow the incident-notification procedure.

## Staff and Job-Posting Intelligence

Focus on technical roles and public work history to infer:

- Languages and frameworks
- Databases and message brokers
- Cloud platforms and container orchestration
- CI/CD and source-control products
- Security tools and governance processes
- Project names, naming schemes, email formats, and likely usernames

Job requirements indicate organizational demand, not guaranteed deployment. Raise confidence only when multiple independent sources agree.

## Public Repository Review

Look for:

- Configuration files and environment examples
- Hardcoded secrets, tokens, or keys
- Internal hostnames, domains, and API paths
- Cloud resource names
- Dependency versions and deployment manifests
- Commit history that may retain deleted secrets

Never use discovered credentials outside explicit authorization. Report active secrets through the engagement's escalation procedure.

## Collection Table

| Finding | Source | Timestamp | Ownership confidence | Scope status | Technical implication | Validation |
|---|---|---|---|---|---|---|
|  |  |  |  |  |  |  |

## Defensive Lessons

- Continuously inventory internet-facing and cloud assets.
- Monitor CT logs and DNS changes.
- Prevent public storage by policy and test access controls.
- Scan repositories and commit history for secrets.
- Remove secrets from code and rotate any exposed credential.
- Review job posts and public profiles for unnecessary operational detail.
- Treat cloud object storage and development artifacts as production-sensitive data.

## Source Contributions

- Hack The Box Academy — Footprinting

> Use public information responsibly and keep all active validation within written scope.
