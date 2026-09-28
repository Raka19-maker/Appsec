# Web and API Assessment: OWASP Juice Shop

## Target / lab
- **Application:** OWASP Juice Shop
- **Type:** Web application with REST API (Angular front end, Node.js back end)
- **Setup:** Docker, run locally (`docker run -p 3000:3000 bkimminich/juice-shop`)

## Scope
- **In scope:** `http://localhost:3000` web UI and `/api` and `/rest` endpoints
- **Out of scope:** Host machine and Docker environment
- **Approach:** Manual testing aligned with OWASP Top 10 and OWASP API Security Top 10
- **Tools:** Burp Suite, Postman, OWASP ZAP

## Summary of findings

| ID | Finding | Severity | Status |
|----|---------|----------|--------|
| F-01 | Login bypass via SQL injection | 🔴 Critical | Validated |
| F-02 | Mass assignment: register as admin | 🔴 Critical | Validated |
| F-03 | BOLA / IDOR: view other users' baskets | 🟠 High | Validated |
| F-04 | DOM XSS in search | 🟡 Medium | Validated |

## Findings

### F-01: Login bypass via SQL injection
- **Severity:** 🔴 Critical
- **Location:** `POST /rest/user/login` (`email` field)
- **Description:** The email value is placed directly into a SQL query.
- **Impact:** Anyone can log in as the administrator without a password.

**Validation steps**
1. Enter `admin@juice-sh.op'--` as the email and any password.
2. Submit the login form.
3. The session is logged in as the admin user.

**Remediation guidance**
- Use parameterised queries or the ORM's safe query methods.
- Add rate limiting and alerting on failed logins.

### F-02: Mass assignment on user registration
- **Severity:** 🔴 Critical
- **Location:** `POST /api/Users`
- **Description:** The API accepts any field in the request body, including `role`.
- **Impact:** A new user can give themselves administrator rights.

**Validation steps**
1. Register a new user and intercept the request in Burp.
2. Add `"role": "admin"` to the JSON body.
3. Log in as the new user; the admin section is accessible.

**Remediation guidance**
- Bind request bodies to an allow-list of fields (a DTO); never write `role` from client input.
- Reference: OWASP API Security Top 10, API3 Broken Object Property Level Authorization.

### F-03: BOLA on basket endpoint
- **Severity:** 🟠 High
- **Location:** `GET /rest/basket/{id}`
- **Description:** The API returns any basket by ID without checking it belongs to the caller.
- **Impact:** Any user can read other customers' baskets.

**Validation steps**
1. Log in and view your basket; note the request `GET /rest/basket/1`.
2. In Postman or Repeater, change the ID to `2`.
3. Another user's basket contents are returned.

**Remediation guidance**
- Check on the server that the basket belongs to the authenticated user before returning it.
- Reference: OWASP API Security Top 10, API1 Broken Object Level Authorization.

### F-04: DOM XSS in search
- **Severity:** 🟡 Medium
- **Location:** Search bar (`/#/search?q=`)
- **Description:** The search term is written into the page without sanitisation.
- **Impact:** A crafted link runs script in the victim's browser.

**Validation steps**
1. Search for `<iframe src="javascript:alert('xss')">`.
2. The alert fires.

**Remediation guidance**
- Don't bypass Angular's built-in sanitisation (`bypassSecurityTrustHtml`) for user input.
- Add a Content Security Policy.

## Key takeaways
- Covered both the web UI and the underlying REST API, showing how API-level flaws (BOLA, mass assignment) lead to account takeover and data exposure.
