# Getting Started: Web Enumeration

> Source: HTB Academy - Penetration Tester Path  
> Module: Getting Started  
> Purpose: Practical web-enumeration reference for authorized labs and assessments

## Core Idea

Web services commonly appear on TCP ports `80` and `443`, although they may run on any port. A single server can host several applications or virtual hosts, giving web services a large attack surface even when the rest of the host exposes very little.

Web enumeration aims to identify:

- Hidden files and directories
- Subdomains and virtual hosts
- Web servers, frameworks, and content-management systems
- Version information and configuration clues
- Administrative interfaces and unfinished installations
- Information exposed through certificates, `robots.txt`, and source code

Tool output is evidence, not the final conclusion. Every promising result should be inspected and validated manually.

## Recommended Workflow

1. Confirm the web service and inspect its headers.
2. Fingerprint the server and application technologies.
3. Review TLS certificate details when HTTPS is enabled.
4. Check `robots.txt` and the page source.
5. Enumerate hidden files and directories.
6. Enumerate subdomains or virtual hosts when a domain is available.
7. Visit and validate interesting results.
8. Record URLs, status codes, technologies, evidence, and follow-up actions.

## Directory and File Enumeration with Gobuster

Gobuster's `dir` mode tests candidate paths from a wordlist against a target web server.

```bash
gobuster dir \
  -u http://10.10.10.121/ \
  -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

| Option | Meaning |
|---|---|
| `dir` | Use directory and file enumeration mode |
| `-u` | Specify the target URL |
| `-w` | Specify the wordlist |

Example findings may include:

```text
/.hta           (Status: 403)
/.htpasswd      (Status: 403)
/.htaccess      (Status: 403)
/index.php      (Status: 200)
/server-status  (Status: 403)
/wordpress      (Status: 301)
```

### Interpreting Status Codes

| Code | Meaning | Pentesting interpretation |
|---|---|---|
| `200` | Successful request | The resource exists and is accessible |
| `301` / `302` | Redirect | Follow the redirect and inspect the destination |
| `401` | Authentication required | Identify the authentication method and permitted testing options |
| `403` | Forbidden | The resource probably exists, but direct access is denied |
| `404` | Not found | The path was not found; custom error behavior may require filtering |

A `403` is still useful because it confirms that the server recognizes the path. A `301` is not a failed result; it often identifies a valid directory or canonical URL.

If enumeration reveals an application such as WordPress, inspect its state before looking for exploits. An exposed installer, debug page, default configuration, or unfinished setup can be more important than a version banner.

## Installing SecLists

SecLists provides wordlists for web content, DNS, usernames, passwords, fuzzing, and other security testing tasks.

```bash
sudo apt update
sudo apt install seclists -y
```

Alternatively, clone the upstream repository:

```bash
git clone https://github.com/danielmiessler/SecLists
```

The packaged wordlists are commonly installed under `/usr/share/seclists/`. Paths can differ by distribution and installation method, so verify the location before scanning.

## DNS Subdomain Enumeration

Gobuster's `dns` mode tests candidate subdomain names and resolves those that exist.

```bash
gobuster dns \
  -d inlanefreight.com \
  -w /usr/share/SecLists/Discovery/DNS/namelist.txt
```

| Option | Meaning |
|---|---|
| `dns` | Use DNS enumeration mode |
| `-d` | Specify the target domain |
| `-w` | Specify the subdomain wordlist |

Subdomains can expose separate applications, administrative panels, APIs, development systems, or customer portals. Resolve and inspect each result rather than assuming every name hosts the same application.

## Banner Grabbing and HTTP Headers

Use `curl` to retrieve response headers and follow redirects:

```bash
curl -IL https://www.inlanefreight.com
```

Useful clues include:

- `Server`: web-server product and sometimes its version
- `X-Powered-By`: application language or framework
- `Location`: redirect destination
- `WWW-Authenticate`: authentication scheme
- `Content-Type`: response format and character encoding
- Framework- or CMS-specific headers and links

Headers may be missing, intentionally altered, or supplied by an intermediary. Treat them as leads that require corroboration.

## Technology Fingerprinting with WhatWeb

Scan one host:

```bash
whatweb 10.10.10.121
```

Scan a network range while suppressing connection errors:

```bash
whatweb --no-errors 10.10.10.0/24
```

WhatWeb can identify web-server software, operating-system hints, frameworks, content-management systems, JavaScript libraries, page titles, email addresses, and selected HTTP headers. Detected versions should be confirmed before mapping them to vulnerabilities.

## Visual Reconnaissance with EyeWitness

EyeWitness can screenshot multiple web applications, fingerprint them, and help identify login panels or default pages. It is useful for triaging many discovered web targets before examining high-value applications directly.

## TLS Certificate Inspection

HTTPS certificates may reveal:

- Organization and department names
- Email addresses
- Common names and Subject Alternative Names
- Additional internal or external hostnames
- Issuer and validity information

Certificate data can expand the target map, but newly discovered names must still be checked against the engagement scope before testing.

## Inspecting `robots.txt`

```bash
curl http://10.10.10.121/robots.txt
```

Example:

```text
User-agent: *
Disallow: /private
Disallow: /uploaded_files
```

`robots.txt` controls search-engine indexing; it is not an access-control mechanism. Disallowed paths may reveal administrative pages, private directories, upload locations, or deprecated content. Visit discovered paths and record their behavior.

## Reviewing Page Source

Press `Ctrl+U` in most browsers to view the original HTML source. Look for:

- Developer comments
- Test credentials or usernames
- Hidden form fields
- API routes and JavaScript endpoints
- Internal hostnames and file paths
- Debug information
- Referenced scripts, configuration objects, and source maps

Browser developer tools are also valuable because dynamically loaded content may not appear in the original HTML.

## Quick Command Reference

| Objective | Command |
|---|---|
| Enumerate directories | `gobuster dir -u http://TARGET/ -w /usr/share/seclists/Discovery/Web-Content/common.txt` |
| Enumerate subdomains | `gobuster dns -d DOMAIN -w /usr/share/SecLists/Discovery/DNS/namelist.txt` |
| Retrieve headers | `curl -IL https://TARGET` |
| Retrieve `robots.txt` | `curl http://TARGET/robots.txt` |
| Fingerprint one host | `whatweb TARGET` |
| Fingerprint a subnet | `whatweb --no-errors 10.10.10.0/24` |
| View browser source | `Ctrl+U` |

## What to Record

For each discovered application, record:

- IP address, hostname, port, protocol, and base URL
- HTTP status and redirect chain
- Page title and application purpose
- Server, framework, CMS, and suspected versions
- Discovered directories, files, subdomains, and virtual hosts
- Authentication entry points
- Certificate names and metadata
- Evidence supporting each observation
- Follow-up action and validation result

## Key Takeaways

- A single web service may expose multiple applications.
- Similarity to a known product or version is a clue, not proof of vulnerability.
- `200`, `301`, `401`, and `403` responses can all reveal useful resources.
- Headers, certificates, `robots.txt`, and source code are inexpensive information sources.
- Directory and subdomain enumeration should be followed by manual validation.
- Findings must remain inside the authorized scope, even when enumeration reveals additional systems.

> Use these techniques only in labs, CTFs, or systems for which you have explicit authorization.
