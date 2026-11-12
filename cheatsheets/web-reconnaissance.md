# Web Reconnaissance Cheat Sheet

> Authorized targets only. Start passive, validate ownership, then increase interaction.

## Variables

```bash
export DOMAIN=example.com
export URL=https://example.com
export IP=192.0.2.10
mkdir -p recon/{whois,dns,ct,http,crawl,archive}
```

## WHOIS / Registration

```bash
whois "$DOMAIN" | tee recon/whois/whois.txt
```

Check registrar, dates, status locks, name servers, organization/privacy service, and abuse contact.

## DNS

```bash
for t in A AAAA CNAME MX NS TXT SOA; do
  dig "$DOMAIN" "$t" +noall +answer
done

dig +short "$DOMAIN"
dig +trace "$DOMAIN"
dig -x "$IP"
dig @1.1.1.1 "$DOMAIN" A
```

### Zone Transfer

```bash
dig "$DOMAIN" NS +short
dig AXFR @<NS_SERVER> "$DOMAIN"
```

## Certificate Transparency

```bash
curl -s "https://crt.sh/?q=%25.$DOMAIN&output=json" |
jq -r '.[].name_value' |
sed 's/\*\.//g' | sort -u |
tee recon/ct/hosts.txt
```

Resolve and validate; CT names can be historical.

## Subdomain Enumeration

```bash
dnsenum --enum "$DOMAIN"   -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -r

gobuster dns -d "$DOMAIN"   -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
```

Test a random label first to detect wildcard DNS.

## Live HTTP

```bash
curl -I "http://$DOMAIN"
curl -k -I "https://$DOMAIN"
whatweb "$URL"
```

Record status, title, length, redirect, canonical host, TLS name, and auth boundary.

## Virtual Hosts

```bash
gobuster vhost -u "http://$IP"   -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt   --domain "$DOMAIN" --append-domain
```

Alternative:

```bash
ffuf -u "http://$IP/" -H "Host: FUZZ.$DOMAIN"   -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt   -fs <DEFAULT_SIZE>
```

Compare against a random-host baseline; filter by status, size, words, title, redirect, or hash.

## Fingerprinting

```bash
curl -k -I "$URL"
whatweb "$URL"
wafw00f "$URL"
nikto -h "$URL"
```

Confirm with multiple signals. WAF discovery does not authorize evasion.

## Discovery Files

```bash
curl -s "$URL/robots.txt"
curl -s "$URL/sitemap.xml"
curl -s "$URL/.well-known/security.txt"
curl -s "$URL/.well-known/openid-configuration" | jq .
```

OpenID metadata: issuer, auth/token/userinfo endpoints, JWKS, scopes, response types, algorithms.

## Crawl

```bash
python3 ReconSpider.py "$URL"
jq . results.json
```

Review:

```text
links  JS  forms  parameters  APIs  comments
emails documents backups configs media external hosts
```

Crawling follows links; fuzzing guesses paths.

## Search Operators

```text
site:DOMAIN
site:DOMAIN (inurl:login OR inurl:admin)
site:DOMAIN filetype:pdf
site:DOMAIN (filetype:xls OR filetype:docx)
site:DOMAIN (ext:conf OR ext:cnf)
site:DOMAIN filetype:sql
site:DOMAIN intitle:"index of"
site:DOMAIN "password reset"
```

## Archives

```text
Wayback Machine:
- old hosts and paths
- retired APIs and parameters
- documents and backups
- technology and staff references
```

Historical does not mean currently owned or reachable—validate.

## Automation

```bash
./finalrecon.py --headers --sslinfo --whois --url "$URL"
./finalrecon.py --dns --sub --crawl --wayback --url "$URL"
```

Automation gathers; the tester validates.

## Workflow

```text
Scope → WHOIS → DNS/CT → Resolve → AXFR
→ Live HTTP → Vhosts → Fingerprint
→ robots/sitemap/.well-known → Crawl/JS/files
→ Search/archives → Automate → Correlate → Report
```

## Evidence

```text
[ ] source + timestamp
[ ] exact command/tool version
[ ] raw output
[ ] hostname/IP/port/protocol
[ ] redirect/canonical host
[ ] technology indicators + confidence
[ ] scope/ownership status
[ ] manual validation
```

Detailed notes: [Knowledge](../knowledge-base/reconnaissance/web-reconnaissance.md) · [Methodology](../pentesting-methodology/reconnaissance/web-reconnaissance-methodology.md) · [General Recon](recon.md)
