# Web Application Pentesting Cheat Sheet

> Fast execution reference for authorized testing. Follow the full [Web Application Penetration Testing Methodology](../pentesting-methodology/web/web-application-penetration-testing.md) for context, safety, evidence, and reporting.

## 0. Scope Gate

- [ ] Target, environment, and ownership confirmed
- [ ] Dates, source IPs, and scan rate approved
- [ ] Allowed and forbidden techniques recorded
- [ ] Test accounts and roles available
- [ ] Data-handling, stop conditions, and emergency contact recorded

```bash
export TARGET="192.0.2.10"
export HOST="app.example.test"
export DOMAIN="example.test"
export RATE="100"       # Choose from the RoE; do not copy a lab rate blindly
export WORDLIST="/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt"
```

## 1. Inventory and Priorities

| ID | URL / host | Purpose | Auth / roles | Sensitive assets | Scope | Priority |
|---|---|---|---|---|---|---|
| APP-01 |  |  |  |  |  |  |

- Identify assets and trust boundaries.
- Write the highest-risk attack paths.
- Test business-critical paths before low-value surface area.

## 2. Discover and Baseline

```bash
nmap -Pn -sV --open -p- --min-rate "$RATE" "$TARGET" -oA web-ports
curl -iskL --max-redirs 5 "https://$HOST/"
curl -isk "https://$HOST/robots.txt"
curl -isk "https://$HOST/sitemap.xml"
whatweb -a 3 "https://$HOST/"
```

Record per application:

- URL, IP, port, environment, title, status, redirects
- Headers, cookies, TLS, response length/hash
- Random nonexistent-path response for soft-404 comparison
- Technology/version evidence from multiple signals
- Login, registration, reset, API docs, admin, uploads, and errors

## 3. Browser + Burp Mapping

Use:

- **Proxy / HTTP history** — capture normal traffic
- **Target / Site map** — inventory endpoints
- **Repeater** — change one variable at a time
- **Comparer** — compare users, roles, and object responses
- **Decoder** — inspect tokens and encodings
- **Intruder** — small authorized test sets only

Walk the application as anonymous, User A, User B, and a privileged user.

| Method | Path | Role | Inputs | Object | State change | Notes |
|---|---|---|---|---|---|---|
|  |  |  |  |  |  |  |

## 4. Content and Virtual Hosts

```bash
ffuf -u "https://$HOST/FUZZ" -w "$WORDLIST" -ac
ffuf -u "https://$HOST/FUZZ" -w "$WORDLIST" -e .php,.asp,.aspx,.jsp,.json,.txt,.bak,.zip -ac
ffuf -u "https://$TARGET/" -H "Host: FUZZ.$DOMAIN" \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -ac
gobuster dir -u "https://$HOST/" -w "$WORDLIST" -k
```

- Filter wildcard/soft-404 responses by status, size, words, lines, and redirects.
- Keep `200`, `30x`, `401`, and `403` leads.
- Inspect HTML, JavaScript, source maps, comments, API schemas, and archived URLs.
- Treat each distinct virtual host as a separate application.

## 5. Authentication and Sessions

- [ ] Registration and account enumeration
- [ ] Login rate limits, lockout, monitoring, and alternate endpoints
- [ ] MFA enrollment, recovery, replay, bypass, and downgrade
- [ ] Reset token entropy, expiry, single use, destination, and invalidation
- [ ] Session rotation after login/privilege change
- [ ] Logout, idle/absolute timeout, concurrency, and replay
- [ ] `Secure`, `HttpOnly`, `SameSite`, domain/path, lifetime
- [ ] CSRF on every state-changing request
- [ ] OAuth/OIDC/SAML/JWT validation and privilege mapping

## 6. Authorization Matrix

For each request, vary one dimension at a time:

```text
Identity → Object → Action → State/Tenant
```

| Role | Object | Read | Create | Update | Delete | Privileged action |
|---|---|---:|---:|---:|---:|---:|
| Anonymous |  |  |  |  |  |  |
| User A: own |  |  |  |  |  |  |
| User A: User B's |  |  |  |  |  |  |
| Privileged |  |  |  |  |  |  |

Check IDs in paths, queries, bodies, nested JSON, headers, and cookies. Test horizontal, vertical, tenant, workflow-state, bulk, search, and export access.

## 7. Input and Injection

- [ ] SQL/NoSQL/LDAP/XPath injection
- [ ] Reflected, stored, and DOM XSS
- [ ] OS command injection
- [ ] Server-side template injection
- [ ] Path traversal and file inclusion
- [ ] SSRF and server-side URL fetching
- [ ] XXE
- [ ] Unsafe deserialization / prototype pollution
- [ ] Header injection / request desynchronization indicators
- [ ] Open redirect
- [ ] CSV, email, PDF, Markdown, and other downstream sinks

Start with harmless differential probes. Confirm reproducibility and impact; do not equate an error with exploitation.

## 8. Files, APIs, and Browser Controls

### File handling

- Extension, MIME, magic bytes, content, filename, traversal, overwrite
- Storage path, direct access, authorization, caching, and execution
- Image/document/archive processing and predictable downloads

### APIs

- BOLA/IDOR, function/property authorization
- Mass assignment and excessive data exposure
- UI/API authentication differences
- Methods, versions, batch operations, pagination, and rate limits
- GraphQL introspection, depth, complexity, object authorization
- WebSocket origin, authentication, authorization, and token expiry

### Browser/client

- DOM sources/sinks, CSP, clickjacking, CORS
- Browser storage, source maps, service workers, secrets
- `postMessage` origin/message validation
- Client-only controls with no server enforcement

## 9. Business Logic and Race Tests

- [ ] Skip, reorder, replay, or repeat workflow steps
- [ ] Modify price, quantity, currency, discount, shipping, or limits
- [ ] Try zero, negative, maximum, duplicate, and stale values
- [ ] Reuse one-time coupons, links, approvals, refunds, or credits
- [ ] Send concurrent requests against balances, stock, quotas, and one-time actions
- [ ] Combine individually allowed actions into a forbidden outcome

## 10. Components and CVEs

Before using a public exploit:

- [ ] Product and exact version corroborated
- [ ] Primary advisory read
- [ ] Platform, module, auth, and configuration preconditions met
- [ ] Least-destructive proof chosen
- [ ] Stop condition and cleanup recorded

## 11. Evidence Template

```text
Finding ID:
UTC timestamp:
Target / environment:
Identity / role / tenant:
Preconditions:
Sanitized request or command:
Relevant response:
Expected result:
Actual result:
Technical impact:
Business impact:
Cleanup performed:
```

Use synthetic data where possible. Preserve raw requests/responses and record blocked or untested coverage.

## 12. When Stuck

1. Did I find every host, port, vhost, API version, and role?
2. Did I compare anonymous, two peers, and a privileged account?
3. Did I inspect JavaScript, source maps, schemas, archives, and browser traffic?
4. Did I test every state-changing workflow and trust boundary?
5. Did I vary identity, object, action, state, sequence, and timing separately?
6. Am I following the highest-risk business path or merely the easiest lead?

## Related Notes

- [Full Web Application Pentesting Methodology](../pentesting-methodology/web/web-application-penetration-testing.md)
- [Web Walking](../web-security/web-walking.md)
- [Vulnerability Knowledge](../web-security/vulnerability-knowledge.md)
- [Threat Modelling for Pentesters](../pentesting-methodology/foundations/threat-modelling-for-pentesters.md)
- [Reconnaissance](../pentesting-methodology/reconnaissance.md)
