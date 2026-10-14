# Web Enumeration Cheat Sheet

> Combined THM and HTB quick reference for authorized testing.

## Baseline

```bash
export TARGET="192.0.2.10"
export DOMAIN="example.test"

curl -iskL "http://$TARGET/"
curl -IL "https://$DOMAIN"
whatweb "http://$TARGET/"
```

Check every web port separately:

```text
80  443  3000  5000  8000  8080  8443
```

## Manual Checks

```bash
curl -s "http://$TARGET/robots.txt"
curl -s "http://$TARGET/sitemap.xml"
curl -s "http://$TARGET/favicon.ico" | md5sum
```

Browser:

```text
Ctrl+U -> source
Inspector -> elements and hidden fields
Debugger -> JavaScript flow
Network -> API requests and responses
```

Look for comments, credentials, usernames, hidden fields, API routes, JavaScript endpoints, source maps, internal paths, and configuration objects.

## Directory and File Discovery

Gobuster:

```bash
gobuster dir -u "http://$TARGET/" \
  -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

ffuf:

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt \
  -u "http://$TARGET/FUZZ"
```

Extensions:

```bash
ffuf -w WORDLIST -u "http://$TARGET/FUZZ" \
  -e .php,.txt,.bak,.old,.zip
```

## Subdomains and Virtual Hosts

```bash
gobuster dns -d "$DOMAIN" \
  -w /usr/share/SecLists/Discovery/DNS/namelist.txt
```

```bash
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  -H "Host: FUZZ.$DOMAIN" \
  -u "http://$TARGET/" \
  -fs BASELINE_SIZE
```

Add confirmed local names only in authorized lab environments:

```bash
echo "$TARGET app.$DOMAIN" | sudo tee -a /etc/hosts
```

## Status Codes

| Code | Meaning | Action |
|---|---|---|
| `200` | Accessible | Inspect content and behavior |
| `301/302` | Redirect | Follow and record destination |
| `401` | Authentication required | Record scheme and login surface |
| `403` | Resource exists but denied | Record path; test only permitted bypass hypotheses |
| `404` | Not found | Compare with baseline/custom error page |

## Fingerprinting

```bash
whatweb "$TARGET"
whatweb --no-errors 10.10.10.0/24
```

Also inspect:

- HTTP headers and cookies
- TLS certificate CN and SAN values
- Default pages and favicons
- CMS, plugins, themes, and exposed installers
- Framework-specific files and routes

## Record Per Application

```text
URL / IP / hostname / port:
Status and redirects:
Title and purpose:
Server / framework / CMS:
Directories and files:
Subdomains / vhosts:
Authentication / roles:
Interesting parameters / APIs:
Evidence:
Next hypothesis:
```

## Decision Rule

```text
Automated discovery -> Manual validation -> Confirm technology/state
                    -> Form testable hypothesis -> Preserve evidence
```

Detailed notes: [Web Enumeration](../knowledge-base/web-security/web-enumeration.md) · [Full Web Methodology](../pentesting-methodology/web/web-application-penetration-testing.md)
