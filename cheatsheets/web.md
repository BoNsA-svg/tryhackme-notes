# 🌐 Web Competition Cheat Sheet

## 1. Baseline

```bash
curl -i http://<TARGET>/
whatweb http://<TARGET>/
nikto -h http://<TARGET>/
```

Check first:

```text
robots.txt
sitemap.xml
page source
JavaScript files
cookies/local storage
HTTP headers
login/register/reset flows
API endpoints
```

## 2. Content Discovery

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt \
-u http://<TARGET>/FUZZ
```

Extensions:

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt \
-u http://<TARGET>/FUZZ -e .php,.txt,.html,.bak,.old,.zip,.json
```

Gobuster alternative:

```bash
gobuster dir -u http://<TARGET> \
-w /usr/share/seclists/Discovery/Web-Content/common.txt \
-x php,txt,html,bak
```

## 3. Vhosts / Subdomains

```bash
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
-H 'Host: FUZZ.<DOMAIN>' -u http://<IP>/ -fs <BASELINE_SIZE>
```

```bash
gobuster dns -d <DOMAIN> \
-w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

## 4. Parameter / Input Checklist

For every input, ask:

```text
Can I change object IDs?
Can I access another user's data?
Can I inject SQL/commands/templates?
Can I control a file path or filename?
Can I upload a file?
Can I influence redirects/URLs?
Can I alter role/admin fields?
Can I bypass authentication logic?
```

## 5. High-Value Files

```text
.env
.git/
config.php
web.config
backup.zip
*.bak
*.old
*.swp
package.json
requirements.txt
Dockerfile
docker-compose.yml
```

## 6. When You Find a Technology

```text
Identify exact product/framework/version
        ↓
Confirm from more than one signal
        ↓
Search for known issues/CVEs
        ↓
Verify preconditions
        ↓
Test carefully
```

Do not jump from a banner directly to exploitation without confirming the version and conditions.

## 7. Stuck Checklist

- Try the app as both authenticated and unauthenticated.
- Compare responses for different users/IDs.
- Inspect every JavaScript file.
- Check alternate methods: GET/POST/PUT/PATCH/DELETE.
- Check alternate content types: form, JSON, XML.
- Inspect redirects and status-code differences.
- Check vhosts, backups, source control, and API documentation.

Detailed notes: [Web Walking](../web-security/web-walking.md) · [Vulnerability Knowledge](../web-security/vulnerability-knowledge.md)
