# Web Application Assessment: PortSwigger Web Security Academy

## Target / lab
- **Application:** PortSwigger Web Security Academy labs
- **Type:** Web application
- **Setup:** PortSwigger hosted labs

## Scope
- **In scope:** Lab instances for SQL injection, cross-site scripting, access control and SSRF
- **Out of scope:** PortSwigger infrastructure outside the lab instance
- **Approach:** Manual testing aligned with OWASP Top 10
- **Tools:** Burp Suite (Proxy, Repeater, Intruder)

## Summary of findings

| ID | Finding | Severity | Status |
|----|---------|----------|--------|
| F-01 | SQL injection (UNION-based) in product category filter | 🔴 Critical | Validated |
| F-02 | Stored XSS in blog comments | 🟠 High | Validated |
| F-03 | IDOR: user ID controlled by request parameter | 🟠 High | Validated |
| F-04 | SSRF via stock check feature | 🟠 High | Validated |

## Findings

### F-01: SQL injection in product category filter
- **Severity:** 🔴 Critical
- **Location:** `GET /filter?category=`
- **Description:** The `category` parameter is concatenated into a SQL query without sanitisation.
- **Impact:** An attacker can read any table in the database, including usernames and passwords.

**Validation steps**
1. Intercept the category request in Burp and send it to Repeater.
2. Find the column count with `' ORDER BY 1--`, `' ORDER BY 2--` until an error appears.
3. Send `' UNION SELECT username, password FROM users--`.
4. Usernames and passwords appear in the product list.

**Remediation guidance**
- Use parameterised queries (prepared statements) for all database access.
- Run the application with a least-privilege database account.
- Reference: OWASP SQL Injection Prevention Cheat Sheet.

### F-02: Stored XSS in blog comments
- **Severity:** 🟠 High
- **Location:** `POST /post/comment` (comment field)
- **Description:** Comment text is stored and rendered without output encoding.
- **Impact:** Script runs in every visitor's browser, allowing session theft or actions as the victim.

**Validation steps**
1. Post a comment containing `<script>alert(document.domain)</script>`.
2. Reload the blog post.
3. The alert fires for every user who views the post.

**Remediation guidance**
- HTML-encode user input on output, using the template engine's auto-escaping.
- Add a Content Security Policy that blocks inline scripts.
- Reference: OWASP XSS Prevention Cheat Sheet.

### F-03: IDOR on account page
- **Severity:** 🟠 High
- **Location:** `GET /my-account?id=`
- **Description:** The server trusts the `id` parameter instead of the logged-in session.
- **Impact:** Any logged-in user can view other users' account data, including API keys.

**Validation steps**
1. Log in as `wiener` and open My Account.
2. In Repeater, change `id=wiener` to `id=carlos`.
3. Carlos's account details and API key are returned.

**Remediation guidance**
- Take the user identity from the server-side session, not from request parameters.
- Enforce an ownership check on every object access.
- Reference: OWASP Authorization Cheat Sheet.

### F-04: SSRF via stock check
- **Severity:** 🟠 High
- **Location:** `POST /product/stock` (`stockApi` parameter)
- **Description:** The server fetches whatever URL is supplied in `stockApi`.
- **Impact:** An attacker can reach internal-only services such as an admin panel.

**Validation steps**
1. Intercept the stock check request.
2. Change `stockApi` to `http://localhost/admin`.
3. The internal admin page is returned, and users can be deleted through it.

**Remediation guidance**
- Allow-list the exact hosts the feature may call; do not accept full URLs from the client.
- Block requests to internal and loopback addresses at the network layer.
- Reference: OWASP SSRF Prevention Cheat Sheet.

## Key takeaways
- Showed a manual, Burp-driven workflow across the most common web vulnerability classes, with a clear fix for each.
