markdown
# Web & HTTP Fundamentals

This chapter covers the underlying web technologies every QA engineer should understand before moving into API testing, since API testing (Step 10) builds directly on these concepts.

---

# 1. How the Web Works (Client-Server Model)

Client (Browser/App) ──── Request ────> Server
Client (Browser/App) <─── Response ──── Server


- **Client:** Sends requests (browser, mobile app, Postman, automation script)
- **Server:** Processes requests and sends back responses
- **Communication protocol:** HTTP/HTTPS

### Example Flow — Logging into a Website
User enters email/password and clicks "Login"
Browser sends an HTTP POST request to the server with credentials
Server validates credentials against the database
Server sends back an HTTP response (success + session token, or error)
Browser processes the response and redirects/shows error accordingly

---

# 2. What is HTTP?

**HTTP (HyperText Transfer Protocol)** is the protocol used for communication between clients and servers on the web.

**HTTPS** is the secure version of HTTP — it encrypts data in transit using SSL/TLS, so sensitive data (passwords, tokens) can't be easily intercepted.

HTTP → Not encrypted (avoid for sensitive data)
HTTPS → Encrypted (standard for modern web apps)


> As a tester, if you ever see a login page or payment page still using plain `http://` instead of `https://`, that's a **security defect** worth flagging immediately.

---

# 3. HTTP Request Structure

An HTTP request has four main parts:
Request Line → Method + URL + HTTP Version
Headers → Metadata about the request
(Optional) Body → Data being sent (for POST/PUT/PATCH)

### Example Request

```http
POST /api/login HTTP/1.1
Host: example.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...

{
  "email": "test@example.com",
  "password": "Test@1234"
}
```

---

# 4. HTTP Methods (Verbs)

| Method | Purpose | Example Use Case |
|---|---|---|
| **GET** | Retrieve data | Fetch a list of products |
| **POST** | Create new data | Submit a login form, create a new user |
| **PUT** | Update/replace an entire resource | Replace a user's full profile |
| **PATCH** | Update part of a resource | Update just the user's email |
| **DELETE** | Remove a resource | Delete a product from the cart |

### Example — Same Resource, Different Methods

GET /api/users/101 → Fetch user with ID 101
PUT /api/users/101 → Replace user 101's entire data
PATCH /api/users/101 → Update only specific fields of user 101
DELETE /api/users/101 → Delete user 101


### Important Distinction: PUT vs PATCH

PUT request body:
{
"name": "John",
"email": "john@example.com",
"phone": "9800000000"
}
→ Replaces the ENTIRE user object. If "phone" is missing, it may get erased.

PATCH request body:
{
"email": "newemail@example.com"
}
→ Updates ONLY the email field, leaves everything else unchanged.


> This distinction is a common interview question — testers should verify that PUT truly replaces data (and check what happens to fields not included in the request) versus PATCH only modifying specified fields.

---

# 5. HTTP Status Codes

Status codes tell the client the result of their request. They're grouped into five categories.

1xx → Informational
2xx → Success
3xx → Redirection
4xx → Client Error
5xx → Server Error


## Most Important Status Codes for QA

| Code | Meaning | Example |
|---|---|---|
| **200 OK** | Request succeeded | Successful GET request |
| **201 Created** | Resource successfully created | New user registered |
| **204 No Content** | Success, but no data returned | Successful DELETE request |
| **301 Moved Permanently** | Resource permanently moved to new URL | Old URL redirects to new one |
| **400 Bad Request** | Invalid request from client | Missing required field |
| **401 Unauthorized** | Authentication required/failed | Invalid or missing login token |
| **403 Forbidden** | Authenticated but not allowed | Regular user trying to access admin panel |
| **404 Not Found** | Resource doesn't exist | Requesting a deleted product |
| **405 Method Not Allowed** | Wrong HTTP method used | Sending GET where only POST is allowed |
| **409 Conflict** | Request conflicts with current state | Trying to register an already-used email |
| **422 Unprocessable Entity** | Request understood but has validation errors | Invalid email format submitted |
| **429 Too Many Requests** | Rate limit exceeded | Too many login attempts in a short time |
| **500 Internal Server Error** | Generic server-side failure | Unhandled exception in backend code |
| **502 Bad Gateway** | Invalid response from upstream server | Server acting as a proxy got a bad response |
| **503 Service Unavailable** | Server temporarily unavailable | Server under maintenance or overloaded |

### Why This Matters for Testing

A defect isn't just "the app doesn't work" — as a tester, you should notice **wrong status codes** too:

❌ Bug: Login with wrong password returns 200 OK with an error message in the body
✅ Correct: Login with wrong password should return 401 Unauthorized


Returning the wrong status code (even if the message is correct) is itself a valid defect, especially for API testing — automated systems and other integrations rely on status codes, not just message text.

---

# 6. HTTP Headers

Headers carry additional metadata about the request or response.

## Common Request Headers

| Header | Purpose |
|---|---|
| `Content-Type` | Format of the request body (e.g., `application/json`) |
| `Authorization` | Credentials/token for authentication |
| `Accept` | What response format the client expects |
| `User-Agent` | Info about the client (browser/device) |
| `Cookie` | Sends stored cookies back to the server |

## Common Response Headers

| Header | Purpose |
|---|---|
| `Content-Type` | Format of the response body |
| `Set-Cookie` | Server sets a cookie on the client |
| `Cache-Control` | Caching rules for the response |
| `Content-Length` | Size of the response body |

### Example

```http
Request Header:
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
Content-Type: application/json

Response Header:
Content-Type: application/json
Set-Cookie: session_id=abc123; HttpOnly; Secure
```

> `HttpOnly` and `Secure` flags on cookies are security-relevant — `HttpOnly` prevents JavaScript access (protects against XSS stealing cookies), `Secure` ensures the cookie is only sent over HTTPS. Testers should verify these flags are set correctly on sensitive cookies like session tokens.

---

# 7. Request Body Formats

| Format | Description | Example |
|---|---|---|
| **JSON** | Most common for modern APIs | `{"email": "test@example.com"}` |
| **XML** | Older format, still used in some systems | `<email>test@example.com</email>` |
| **Form-data** | Used for file uploads and traditional forms | `key=value` pairs, supports files |
| **x-www-form-urlencoded** | URL-encoded key-value pairs | `email=test%40example.com` |

---

# 8. Cookies vs Sessions vs Tokens (Authentication Concepts)

## Cookies
Small pieces of data stored in the browser, sent automatically with every request to the same domain.

Set-Cookie: session_id=abc123; Expires=Wed, 09 Sep 2026 10:00:00 GMT; HttpOnly; Secure


## Sessions
Server-side storage that tracks a logged-in user's state, usually linked to the client via a session cookie.
User logs in
Server creates a session, stores it server-side
Server sends session_id to client via cookie
Client sends session_id with every subsequent request
Server looks up the session to identify the user

## Tokens (e.g., JWT — JSON Web Token)
A self-contained piece of data that proves identity, often used in modern APIs instead of server-side sessions.

Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...


### Session-based vs Token-based Auth

| Session-based | Token-based (JWT) |
|---|---|
| Server stores session data | Server doesn't need to store anything (stateless) |
| Session ID sent via cookie | Token sent via `Authorization` header |
| Harder to scale across multiple servers | Easier to scale (stateless) |
| Common in traditional web apps | Common in modern APIs/SPAs/mobile apps |

> Understanding this distinction matters for API testing — testers need to know how to properly authenticate requests (cookie vs bearer token) when testing protected endpoints in Postman/automation.

---

# 9. URL Structure

https://api.example.com:443/v1/users?role=admin&active=true#profile
│ │ │ │ │ │
protocol domain port path query params fragment


| Part | Example | Purpose |
|---|---|---|
| **Protocol** | `https://` | Communication method |
| **Domain** | `api.example.com` | Server address |
| **Port** | `:443` | Specific port (often omitted, defaults apply) |
| **Path** | `/v1/users` | Specific resource/endpoint |
| **Query Parameters** | `?role=admin&active=true` | Additional filters/options |
| **Fragment** | `#profile` | Client-side reference (e.g., scroll to section) |

### Query Parameters — Testing Relevance

GET /api/products?category=shoes&sort=price&page=2

Test scenarios to consider:

Valid category → correct filtered results
Invalid/nonexistent category → empty results or error?
Missing parameter → does it use a default?
Multiple values → ?category=shoes&category=bags → how is this handled?
Special characters in parameter values → properly encoded?

---

# 10. REST API Basics (Foundation for Step 10)

**REST (Representational State Transfer)** is an architectural style for designing APIs, built entirely on HTTP concepts covered above.

## REST Principles
Resources are identified by URLs (e.g., /users, /users/101)
Standard HTTP methods represent actions (GET, POST, PUT, PATCH, DELETE)
Stateless — each request contains all information needed (no server-side session dependency)
Responses typically in JSON format

### Example REST Endpoints for a "Products" Resource

GET /api/products → Get all products
GET /api/products/45 → Get product with ID 45
POST /api/products → Create a new product
PUT /api/products/45 → Replace product 45 entirely
PATCH /api/products/45 → Update part of product 45
DELETE /api/products/45 → Delete product 45


---

# 11. Using Browser DevTools to Inspect HTTP Traffic

A practical QA skill — inspecting real requests/responses directly from the browser (Chrome DevTools → Network tab).

### What to check in DevTools
Request URL and method
Request headers and body (payload)
Response status code
Response headers and body
Timing (how long the request took)

### Practical Use Case for QA

Bug report says: "Clicking Save doesn't work."

Instead of guessing, open DevTools → Network tab → click Save →
observe: Did a request even fire? What status code came back?
What was in the response body? This often reveals the actual root cause
(e.g., a 400 error due to a missing required field) much faster than
guessing from the UI alone.


---

## Key Takeaways

- The web works on a **client-server model**, communicating via **HTTP/HTTPS**.
- **HTTP methods** (GET, POST, PUT, PATCH, DELETE) map to specific actions on resources — understanding PUT vs PATCH is a common interview point.
- **HTTP status codes** (2xx success, 4xx client error, 5xx server error) are critical to verify during testing — a wrong status code is a valid defect even if the message looks fine.
- **Headers** carry metadata like `Content-Type` and `Authorization`; security-relevant cookie flags (`HttpOnly`, `Secure`) should be verified by testers.
- **Cookies, sessions, and tokens (JWT)** are different ways to manage authentication state — token-based auth is more common in modern APIs.
- **URL structure** (protocol, domain, path, query parameters) matters for designing test cases around query parameter behavior.
- **REST APIs** are built entirely on these HTTP fundamentals — this chapter is the direct foundation for API Testing (Step 10).
- **Browser DevTools** (Network tab) is a practical, everyday tool for QA to inspect real HTTP traffic and diagnose bugs faster.