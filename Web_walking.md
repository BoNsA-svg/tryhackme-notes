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
