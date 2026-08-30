markdown
# Security Testing Basics

Security Testing verifies that an application protects data, resources, and users from unauthorized access, vulnerabilities, and malicious attacks. This chapter builds on security concepts scattered across earlier steps (SQL injection in Step 11, auth in Step 10) and organizes them into a structured foundation.

---

# 1. What is Security Testing?

Security Testing is a type of non-functional testing focused on uncovering vulnerabilities, threats, and risks in an application that could allow unauthorized access, data breaches, or other malicious harm.

> Functional testing asks "does the login work?" — Security testing asks **"can someone break in, bypass, or abuse this login?"**

## Why QA Should Care About Security (Not Just Dedicated Security Teams)
QA is often the first line of defense — finding basic vulnerabilities before release
Many security issues (broken auth, exposed data, injection) are catchable with
the same "what can go wrong?" mindset already used in functional testing (Step 1)
Security bugs are typically far more costly than functional bugs if they reach production
Not every company has a dedicated security team — QA often fills this gap

---

# 2. Core Security Principles (CIA Triad)

| Principle | Meaning |
|---|---|
| **Confidentiality** | Data is only accessible to authorized users |
| **Integrity** | Data cannot be altered by unauthorized parties |
| **Availability** | Systems remain accessible to legitimate users when needed |

### Example Violations

Confidentiality violation: An attacker accesses another user's private order history
Integrity violation: An attacker modifies the price of an item during checkout
Availability violation: A DDoS attack takes the application offline


---

# 3. Authentication vs Authorization (Critical Distinction)

Already introduced in Step 10 — expanded here since it's central to security testing.

| Authentication | Authorization |
|---|---|
| "Who are you?" (verifying identity) | "What are you allowed to do?" (verifying permissions) |
| Login with email/password | Checking if a logged-in user can access admin features |
| Failure → 401 Unauthorized | Failure → 403 Forbidden |

### Example Test Cases

Authentication tests:

Login with valid credentials → success
Login with invalid credentials → rejected
Access a protected endpoint without a token → 401
Access a protected endpoint with an expired token → 401

Authorization tests:

Regular user tries to access admin panel (with a VALID token) → 403
User A tries to view User B's private data using User A's own valid token → 403
User attempts to escalate their own role via a hidden/unvalidated API field → should be rejected

---

# 4. OWASP Top 10 (Industry-Standard Vulnerability List)

OWASP (Open Web Application Security Project) publishes a regularly updated list of the most critical web application security risks. QA engineers should be familiar with these at a conceptual level.

## 1. Broken Access Control
Users can act outside their intended permissions.

Example: Changing a URL from /orders/101 to /orders/102 lets a user
view another user's order, even though they're not logged in as that user.

Test: Try accessing resources by ID that don't belong to the current user.


## 2. Cryptographic Failures
Sensitive data isn't properly protected (in transit or at rest).

Example: Passwords stored in plain text in the database instead of hashed.
Example: Login form submitted over HTTP instead of HTTPS.

Test: Check if sensitive data is encrypted in transit (HTTPS) and verify
(via database inspection, Step 11) that passwords are hashed, not stored in plain text.


## 3. Injection
Untrusted data sent to an interpreter as part of a command or query (covered in depth in Step 11 for SQL Injection).

Types: SQL Injection, Command Injection, LDAP Injection

Test: Try inputs like ' OR '1'='1 or ; DROP TABLE users; in every input field.


## 4. Insecure Design
Security flaws baked into the application's fundamental design, not just implementation bugs.

Example: A password reset flow that doesn't expire reset links, or
allows unlimited reset attempts without rate limiting.


## 5. Security Misconfiguration
Default, incomplete, or overly permissive configurations.

Example: Admin panel accessible without authentication because it was
never properly locked down. Default admin credentials left unchanged.
Detailed error messages exposing server/stack trace information to users.

Test: Check error pages for leaked technical details (stack traces,
database structure, server versions).


## 6. Vulnerable and Outdated Components
Using libraries/frameworks with known vulnerabilities.

Example: An application using an old version of a library with a
publicly known security flaw that hasn't been patched.

QA relevance: Not usually directly tested by QA, but worth flagging
if outdated dependency versions are visible/known.


## 7. Identification and Authentication Failures
Weaknesses in login, session management, and credential handling.

Example: No account lockout after repeated failed login attempts
(allows brute-force password guessing).
Example: Session tokens don't expire, or remain valid after logout.

Test: Attempt many rapid failed logins — is there a lockout/rate limit?
Test: Log out, then try reusing the old session token — is it rejected?


## 8. Software and Data Integrity Failures
Code/infrastructure that doesn't verify the integrity of updates, critical data, or CI/CD pipelines.

Example: An application that auto-updates from an unverified source,
allowing malicious code injection.


## 9. Security Logging and Monitoring Failures
Insufficient logging/monitoring, delaying detection of an active breach.

Example: Repeated failed login attempts from the same IP aren't logged
or flagged, allowing a brute-force attack to go unnoticed for a long time.


## 10. Server-Side Request Forgery (SSRF)
An application fetches a remote resource without validating the user-supplied URL, potentially allowing access to internal systems.

Example: An "import image from URL" feature that lets an attacker supply
an internal server address instead, potentially exposing internal services.


---

# 5. Cross-Site Scripting (XSS)

A vulnerability where an attacker injects malicious scripts into content viewed by other users.

### Example Payload
```html
<script>alert('XSS')</script>
```

If a comment field doesn't sanitize input, and this gets stored and later rendered on the page, the script executes in **other users'** browsers when they view the comment — potentially stealing session cookies, redirecting users, or performing actions on their behalf.

### Types of XSS

| Type | Description |
|---|---|
| **Stored XSS** | Malicious script is saved (e.g., in a comment/profile) and served to other users later |
| **Reflected XSS** | Malicious script is immediately reflected back in the response (e.g., via a URL parameter) |
| **DOM-based XSS** | Vulnerability exists in client-side JavaScript that unsafely handles user input |

### Testing for XSS

Test inputs to try in text fields, search boxes, comments, profile names:

<script>alert(1)</script> <img src=x onerror=alert(1)> "><script>alert(1)</script>

Expected (secure) behavior: These should be displayed as literal text on the page,
NOT executed as actual JavaScript. If an alert box pops up, that's a real XSS vulnerability.


---

# 6. Cross-Site Request Forgery (CSRF)

Tricks a logged-in user's browser into performing an unwanted action on a site they're authenticated with, without their knowledge.

Example scenario:

User is logged into their banking site (has an active session)
User visits a malicious website in another tab
The malicious site silently submits a form to the bank's
"transfer money" endpoint, using the user's existing session cookie
If the bank doesn't verify the request's legitimacy (e.g., via a CSRF token),
the transfer happens without the user's knowledge or consent

### Testing for CSRF Protection
Check if state-changing requests (POST/PUT/DELETE) require a unique CSRF token
Try replaying/resubmitting a request without the expected CSRF token → should be rejected

---

# 7. Insecure Direct Object Reference (IDOR)

A specific, very common form of Broken Access Control — occurs when an application exposes direct references to internal objects (like database IDs) without properly checking if the requester is authorized to access them.

### Example

GET /api/invoices/1042 → returns User A's invoice (User A is logged in) ✓ correct
GET /api/invoices/1043 → User A changes the ID in the URL/request
and receives User B's invoice ✗ IDOR vulnerability


### Testing for IDOR
Log in as User A, note an ID belonging to User A's own resource (e.g., order ID)
Log in as User B (or reuse User A's session), attempt to access
a resource ID known to belong to a DIFFERENT user
Expected: 403 Forbidden or 404 Not Found
If the data is actually returned: confirmed IDOR vulnerability

> IDOR is one of the most practical, easy-to-test vulnerabilities for QA — it doesn't require deep security expertise, just methodically trying to access other users' data by changing IDs.

---

# 8. Security Testing for Authentication Flows

Password Policy:

Are weak passwords (e.g., "12345678") rejected?
Is there a minimum length/complexity requirement enforced?

Account Lockout:

Does the account lock (or introduce delay) after repeated failed attempts?
Can the lockout be bypassed by changing IP address or user agent?

Password Reset:

Does the reset link expire after a reasonable time?
Can the reset link be used more than once?
Is the reset token sufficiently random/unguessable (not a predictable sequence)?

Session Management:

Does the session expire after a period of inactivity?
Is the session invalidated properly on logout (can't reuse the old token)?
Are session tokens transmitted securely (HTTPS only, HttpOnly/Secure cookie flags — Step 9)?

Multi-Factor Authentication (if applicable):

Can MFA be bypassed by directly calling an API endpoint?
Is the OTP/code time-limited and single-use?

---

# 9. Testing Input Validation and Sanitization

General inputs to try across ALL text fields (login, search, forms, comments):

SQL injection payloads: ' OR '1'='1, '; DROP TABLE users; --
XSS payloads: <script>alert(1)</script>
Extremely long strings (buffer/length handling)
Special characters: < > & " ' % ; ( )
Null bytes: %00
Unicode/emoji characters
Whitespace-only input
File upload: uploading a script file disguised with an image extension
(e.g., malicious.php.jpg) to test if the server properly validates file TYPE, not just extension

---

# 10. API Security Testing (Building on Step 10)
Verify all protected endpoints require valid authentication
Verify authorization is checked (not just authentication) — user can't access others' data
Verify rate limiting exists on sensitive endpoints (login, password reset, OTP verification)
Verify sensitive data (passwords, tokens, card numbers) never appears in API responses
Verify proper HTTP status codes are used (401 vs 403 — don't leak whether a resource exists)
Verify API doesn't expose verbose error messages/stack traces
Test with modified/tampered JWT tokens (e.g., altered payload) — should be rejected

### Example — Testing for Data Exposure

```json
// BAD: Password hash exposed in API response
{
  "id": 101,
  "email": "test@example.com",
  "password_hash": "$2b$10$abc123..."
}

// GOOD: Sensitive fields excluded entirely
{
  "id": 101,
  "email": "test@example.com"
}
```

---

# 11. HTTPS and Transport Security
Verify the entire application uses HTTPS, not just the login page
(mixed HTTP/HTTPS content can expose data on non-secure pages)
Verify HTTP requests are automatically redirected to HTTPS
Check for the presence of HSTS (HTTP Strict Transport Security) headers
Verify SSL/TLS certificate is valid and not expired

---

# 12. Basic Security Testing Tools (Awareness Level)

| Tool | Purpose |
|---|---|
| **OWASP ZAP** | Free, open-source tool for finding vulnerabilities (automated scanning + manual testing) |
| **Burp Suite** | Widely used tool for intercepting/manipulating requests, finding vulnerabilities |
| **Postman** | Can be used to manually test auth bypass, IDOR, and injection scenarios (Step 10) |
| **sqlmap** | Automated tool specifically for detecting/exploiting SQL injection |
| **Browser DevTools** | Inspecting cookies, headers, and requests manually (Step 9) |

> For QA at the "basics" level, manual testing with **Postman** (trying different tokens, IDs, and payloads) and basic **OWASP ZAP** scans often provide the most practical value without needing deep specialization.

---

# 13. Practical Security Test Checklist (Login Feature Example)

Bringing it all together with a concrete example, revisiting the login feature from Step 1 with a security lens:

✔ SQL injection payloads rejected in email/password fields
✔ XSS payloads in any text field are rendered as text, not executed
✔ Generic error message for wrong email/password (doesn't reveal which is wrong)
✔ Account locks/rate-limits after repeated failed attempts
✔ Password field is masked and not exposed in browser autocomplete inappropriately
✔ Session token is not exposed in the URL
✔ Session cookie has HttpOnly and Secure flags set
✔ Login only works over HTTPS
✔ Old session token becomes invalid after logout
✔ JWT token (if used) is properly validated — tampering with it is rejected
✔ Password reset link expires and can only be used once
✔ No sensitive data (password hash, tokens) appears in any API response


---

## Key Takeaways

- Security Testing verifies **Confidentiality, Integrity, and Availability (CIA Triad)** — not just "does it work," but "can it be broken into or abused?"
- **Authentication (who are you?)** vs **Authorization (what can you do?)** is a critical distinction — test both separately (401 vs 403).
- The **OWASP Top 10** is the industry-standard reference for common vulnerabilities: Broken Access Control, Injection, Security Misconfiguration, and others.
- **XSS** (injecting scripts that execute in other users' browsers) and **CSRF** (tricking a logged-in user's browser into unwanted actions) are common, testable web vulnerabilities.
- **IDOR** (accessing other users' data by changing an ID) is one of the most practical vulnerabilities for QA to test — no deep expertise required, just systematic ID manipulation.
- Authentication flows deserve focused security testing: password policy, account lockout, password reset expiry/reuse, and proper session invalidation on logout.
- **Input validation testing** (SQL injection, XSS payloads, long strings, special characters) should be applied broadly across all input fields, not just login.
- API security testing should verify both authentication AND authorization on every protected endpoint, and confirm sensitive data never leaks into responses.
- Basic tools like **OWASP ZAP**, **Burp Suite**, and even **Postman** (manually testing tampered tokens/IDs) provide practical value at the QA "basics" level.
- Security testing uses the same **"what can go wrong?"** mindset from Step 1 — just applied with an attacker's perspective instead of a typical user's.