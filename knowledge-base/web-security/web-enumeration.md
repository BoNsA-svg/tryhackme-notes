# Web Enumeration

> Unified from TryHackMe and Hack The Box web-enumeration material.

## Objective

Web enumeration maps an application's reachable surface before vulnerability testing. It combines manual inspection, automated content discovery, DNS and virtual-host discovery, technology fingerprinting, and evidence collection.

Web services often use ports `80` and `443`, but alternate ports such as `3000`, `5000`, `8000`, `8080`, and `8443` must be treated as separate applications until proven otherwise.

## Enumeration Workflow

1. Confirm every HTTP(S) service and create a URL inventory.
2. Establish a baseline response for each application.
3. Inspect headers, redirects, cookies, and TLS certificates.
4. Review `robots.txt`, `sitemap.xml`, favicon, HTML source, JavaScript, and network requests.
5. Fingerprint the server, framework, CMS, and client libraries.
6. Discover directories, files, extensions, subdomains, and virtual hosts.
7. Inspect authentication flows, roles, forms, parameters, and APIs.
8. Validate discoveries manually and record evidence.

## Manual Discovery

### `robots.txt`

```bash
curl -s http://TARGET/robots.txt
```

Disallowed paths may reveal administrative panels, upload directories, private areas, or deprecated routes. The file controls crawler behavior; it does not enforce access control.

### `sitemap.xml`

```bash
curl -s http://TARGET/sitemap.xml
```

Sitemaps can reveal deep routes, older pages, and content that is difficult to reach through normal navigation.

### Headers and Redirects

```bash
curl -IL https://TARGET
curl -iskL https://TARGET/
```

Review `Server`, `X-Powered-By`, `Location`, `WWW-Authenticate`, cookies, caching behavior, content types, and missing or inconsistent security headers. Treat disclosed versions as hypotheses until corroborated.

### Favicon Fingerprinting

```bash
curl -s https://TARGET/favicon.ico | md5sum
```

A default favicon can identify a framework or product. Compare the hash with a trusted favicon reference, then confirm using other signals.

### Source and Browser Tools

- View source: `Ctrl+U`
- Inspector: examine rendered elements and hidden fields
- Debugger: trace client-side JavaScript
- Network panel: review API calls, parameters, methods, and responses

Look for comments, usernames, test credentials, internal paths, API routes, configuration objects, source maps, and referenced scripts.

## Technology Fingerprinting

```bash
whatweb TARGET
whatweb --no-errors 10.10.10.0/24
```

WhatWeb can identify web servers, frameworks, CMS products, libraries, titles, email addresses, and selected headers. EyeWitness can add screenshots and visual triage across many targets.

TLS certificates may expose organization names, email addresses, common names, and Subject Alternative Names. Confirm that any newly discovered hostname is in scope before testing it.

## Directory and File Discovery

### Gobuster

```bash
gobuster dir -u http://TARGET/ \
  -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

### ffuf

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt \
  -u http://TARGET/FUZZ
```

Useful extensions can be added when the technology suggests them:

```bash
ffuf -w WORDLIST -u http://TARGET/FUZZ -e .php,.txt,.bak,.old,.zip
```

Establish a nonexistent-path baseline before filtering by response size or status. Custom error pages can make every candidate look valid.

### Status-Code Meaning

| Code | Interpretation |
|---|---|
| `200` | Accessible resource |
| `301` / `302` | Valid redirect; inspect its destination |
| `401` | Resource requires authentication |
| `403` | Resource likely exists but access is denied |
| `404` | Not found, subject to custom error behavior |

`401`, `403`, and redirects are findings worth recording; enumeration is not limited to `200` responses.

## Subdomain and Virtual-Host Discovery

DNS-backed subdomains:

```bash
gobuster dns -d example.test \
  -w /usr/share/SecLists/Discovery/DNS/namelist.txt
```

Virtual hosts when names may not exist in public DNS:

```bash
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  -H 'Host: FUZZ.example.test' \
  -u http://TARGET/ \
  -fs BASELINE_SIZE
```

Subdomains and virtual hosts may expose admin panels, APIs, development environments, and customer portals. Verify each result manually.

## Interpreting Findings

- A discovered CMS is a starting point: check configuration, plugins, themes, exposed installers, and authentication state.
- Exact version matches do not prove vulnerability; verify prerequisites and environmental conditions.
- An unfinished installer, debug page, backup file, or exposed configuration may provide a simpler path than a public exploit.
- Automated tools locate candidates. Manual review determines meaning and impact.

## Evidence to Preserve

- IP, hostname, port, protocol, and URL
- HTTP method, status, redirect chain, and response size
- Page title and application purpose
- Server, framework, CMS, and version evidence
- Directories, files, subdomains, and virtual hosts
- Authentication entry points and observed roles
- Certificate names and metadata
- Commands, timestamps, screenshots, and response excerpts
- Follow-up hypothesis and validation result

## Source Records

- [TryHackMe Junior Penetration Testing](../../platforms/tryhackme/learning-paths/junior-penetration-testing.md)
- [HTB Getting Started: Web Enumeration](../../platforms/hack-the-box/penetration-tester-path/getting-started-web-enumeration.md)

## Related References

- [Web Application Pentesting Methodology](../../pentesting-methodology/web/web-application-penetration-testing.md)
- [Web Enumeration Cheat Sheet](../../cheatsheets/web-enumeration.md)
- [Complete Web Cheat Sheet](../../cheatsheets/web.md)
