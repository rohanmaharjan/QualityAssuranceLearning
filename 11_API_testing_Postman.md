markdown
# API Testing + Postman

This chapter builds directly on Web & HTTP Fundamentals (Step 9) and covers how QA engineers test APIs in practice, using Postman as the primary tool.

---

# 1. What is API Testing?

API Testing verifies that an Application Programming Interface (API) meets expectations for functionality, reliability, performance, and security — **without going through the UI**.

UI Testing: User → Browser → UI → API → Database
API Testing: Tester/Tool → API → Database (UI bypassed entirely)


## Why Test APIs Directly (Not Just Through UI)?
Faster than UI testing (no page loading, rendering, clicking)
More stable (not affected by UI changes/CSS/layout issues)
Tests business logic directly at the source
Can test edge cases hard to trigger through UI (malformed data, specific headers)
Enables early testing — API may be ready before UI is built
Easier to automate and integrate into CI/CD pipelines

---

# 2. Types of API Testing

| Type | Focus |
|---|---|
| **Functional Testing** | Does the API return correct data/behavior for given input? |
| **Validation Testing** | Does the API reject invalid/malformed input correctly? |
| **Negative Testing** | How does the API handle unexpected/incorrect requests? |
| **Load/Performance Testing** | How does the API perform under heavy traffic? |
| **Security Testing** | Is the API protected against unauthorized access, injection, etc.? |
| **Integration Testing** | Does the API work correctly with other connected services? |
| **Contract Testing** | Does the API response match the agreed schema/structure? |

---

# 3. What is Postman?

Postman is the most widely used tool for manually testing, exploring, and automating API requests — without writing code.

## Core Postman Features
Send requests (GET, POST, PUT, PATCH, DELETE, etc.)
Organize requests into Collections
Use Environments for different configs (dev/staging/prod)
Write test scripts (JavaScript) to validate responses
Chain requests together (use one response's data in the next request)
Automate collection runs (Collection Runner, Newman CLI)
Generate API documentation
Mock servers for testing before backend is ready

---

# 4. Sending a Basic Request in Postman

### Example — GET Request

Method: GET
URL: https://api.example.com/v1/products

Headers:
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...


**Response:**
```json
{
  "status": "success",
  "data": [
    { "id": 1, "name": "Shoes", "price": 49.99 },
    { "id": 2, "name": "Bag", "price": 29.99 }
  ]
}
```

### Example — POST Request (Creating a Resource)

Method: POST
URL: https://api.example.com/v1/login
Headers:
Content-Type: application/json

Body (raw JSON):
{
"email": "test@example.com",
"password": "Test@1234"
}


**Expected Response:**
```json
{
  "status": "success",
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "user": {
    "id": 101,
    "email": "test@example.com"
  }
}
```
**Expected Status Code:** `200 OK`

---

# 5. Postman Collections

A **Collection** is a group of saved API requests, organized logically (often by module/feature), which can be run individually or all together.

### Example Collection Structure

Collection: E-commerce API Tests
│
├── Auth
│ ├── POST Login
│ ├── POST Register
│ └── POST Logout
│
├── Products
│ ├── GET All Products
│ ├── GET Product by ID
│ └── POST Create Product
│
└── Cart
├── POST Add to Cart
├── DELETE Remove from Cart
└── GET Cart Contents


---

# 6. Postman Environments and Variables

Environments let you switch between different configurations (dev, staging, production) without editing each request manually.

### Example Environment Variables

| Variable | Dev Value | Staging Value | Production Value |
|---|---|---|---|
| `base_url` | `https://dev-api.example.com` | `https://staging-api.example.com` | `https://api.example.com` |
| `auth_token` | (dev token) | (staging token) | (prod token) |

### Using Variables in Requests

URL: {{base_url}}/v1/products
Headers:
Authorization: Bearer {{auth_token}}


## Types of Variables in Postman

| Scope | Description |
|---|---|
| **Global** | Available across all collections/environments |
| **Environment** | Specific to a selected environment (dev/staging/prod) |
| **Collection** | Specific to a particular collection |
| **Local/Runtime** | Temporary, exists only during a single script run |

---

# 7. Writing Test Scripts in Postman (Assertions)

Postman uses JavaScript (via the `pm` object) in the **Tests** tab to validate responses automatically.

### Example — Basic Assertions

```javascript
// Verify status code is 200
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Verify response time is acceptable
pm.test("Response time is less than 500ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(500);
});

// Verify response body contains expected field
pm.test("Response has user email", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.user.email).to.eql("test@example.com");
});

// Verify token exists in response
pm.test("Token is present", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.token).to.not.be.undefined;
});
```

### Example — Negative Test Case Assertions

```javascript
// Testing login with wrong password — expecting 401
pm.test("Status code is 401 for invalid credentials", function () {
    pm.response.to.have.status(401);
});

pm.test("Error message is generic (doesn't reveal if email exists)", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.message).to.eql("Invalid email or password");
});
```

---

# 8. Chaining Requests (Using Data Between Requests)

A common real-world scenario: log in first, capture the auth token, then use it in subsequent requests automatically.

### Example — Capturing Token from Login Response

**In the Login request's Tests tab:**
```javascript
var jsonData = pm.response.json();
pm.environment.set("auth_token", jsonData.token);
```

**In subsequent requests, use:**

Authorization: Bearer {{auth_token}}


This way, the token is automatically captured and reused — no manual copy-pasting needed between requests.

---

# 9. Pre-request Scripts

Scripts that run **before** the request is sent — useful for setting up dynamic data.

### Example — Generating a Unique Email for Registration Testing

```javascript
var timestamp = Date.now();
pm.environment.set("random_email", "testuser" + timestamp + "@example.com");
```

**Used in the request body:**
```json
{
  "email": "{{random_email}}",
  "password": "Test@1234"
}
```

> This avoids "email already exists" errors when repeatedly testing registration.

---

# 10. Postman Collection Runner and Newman

## Collection Runner
Runs all requests in a collection sequentially (or in a defined order), useful for regression testing a whole set of API endpoints at once.

Collection Runner Flow:

Select collection
Select environment
Set number of iterations
Run
View pass/fail summary for each request's tests

## Newman (CLI Tool)
Newman is Postman's command-line companion, allowing collections to run **outside** the Postman UI — critical for CI/CD integration.

### Example Newman Command
```bash
newman run ecommerce-api-tests.postman_collection.json -e staging.postman_environment.json
```

**Example Output Summary:**

┌─────────────────────────┬──────────┬──────────┐
│ │ executed │ failed │
├─────────────────────────┼──────────┼──────────┤
│ requests │ 15 │ 0 │
│ assertions │ 42 │ 1 │
└─────────────────────────┴──────────┴──────────┘


> This connects directly to Step 17 (CI/CD) — Newman lets API test suites run automatically on every code push/deployment.

---

# 11. API Testing — Common Test Scenarios

### For a "Create User" API (`POST /api/users`)

Positive:

Valid data → 201 Created, user object returned

Negative:

Missing required field (e.g., no email) → 400 Bad Request
Invalid email format → 400/422 with clear error message
Duplicate email → 409 Conflict
Invalid data type (e.g., age as a string "abc") → 400 Bad Request

Security:

SQL injection payload in fields → should be rejected/sanitized
XSS payload in name field → should be sanitized before storage
Missing/invalid auth token (if endpoint is protected) → 401 Unauthorized
Unauthorized role trying to create admin user → 403 Forbidden

Boundary:

Name field with 1 character (minimum)
Name field at maximum allowed length
Name field exceeding maximum length → should be rejected

Response Structure:

Verify all expected fields are present in response
Verify field data types match schema (e.g., "id" is a number, not a string)
Verify no sensitive data leaked (e.g., password should NEVER appear in response)

---

# 12. Schema Validation

Verifying the response body's **structure** matches an agreed contract — important in team environments where frontend and backend are developed by different people/teams relying on a shared contract.

### Example — JSON Schema

```json
{
  "type": "object",
  "properties": {
    "id": { "type": "number" },
    "email": { "type": "string" },
    "created_at": { "type": "string" }
  },
  "required": ["id", "email", "created_at"]
}
```

### Postman Schema Validation Example
```javascript
const schema = {
  type: "object",
  properties: {
    id: { type: "number" },
    email: { type: "string" }
  },
  required: ["id", "email"]
};

pm.test("Response matches schema", function () {
    pm.response.to.have.jsonSchema(schema);
});
```

> Schema validation catches issues like a field unexpectedly changing type (e.g., `id` switching from a number to a string) — a subtle bug that's easy to miss with manual inspection but breaks integrations relying on the contract.

---

# 13. API Authentication Methods (Testing Perspective)

| Method | How it Works | Testing Focus |
|---|---|---|
| **Basic Auth** | Username:password encoded in header | Verify credentials required, test with wrong creds |
| **API Key** | Fixed key sent via header or query param | Verify invalid/missing key is rejected |
| **Bearer Token (JWT)** | Token sent via `Authorization` header | Verify expired/invalid/missing tokens are rejected |
| **OAuth 2.0** | Token obtained via authorization flow, often with refresh tokens | Verify token expiry, refresh flow, scope restrictions |

### Example — Testing Token Expiry

Test: Use an expired token to access a protected endpoint
Expected: 401 Unauthorized with a clear "token expired" message


### Example — Testing Authorization (Not Just Authentication)

Test: Regular user tries to access an admin-only endpoint using a VALID token
Expected: 403 Forbidden (authenticated, but not authorized)


> Distinguishing between **401 (not authenticated)** and **403 (authenticated but not permitted)** is a frequent interview question and a real testing gap in many applications.

---

# 14. Mock Servers

Postman allows creating **mock servers** that simulate API responses — useful when the real backend isn't ready yet, letting QA (and frontend developers) start testing/building against a predictable fake API early.

Frontend/QA → Mock Server (returns pre-defined sample responses) → No real backend needed yet


This supports **shift-left testing** — testing can begin before development is fully complete, aligning with the Agile principle of getting QA involved early (from Step 7).

---

# 15. Common Postman Interview/Practical Questions
What's the difference between PUT and PATCH? (covered in Step 9)
How do you pass data from one request to another in Postman?
How do you handle authentication in Postman (Bearer token setup)?
What's the difference between Postman Environment and Global variables?
How would you run a collection as part of a CI/CD pipeline?
How do you validate a JSON response's structure/schema?
How do you test an API that requires a token that expires quickly?

---

## Key Takeaways

- **API Testing** verifies functionality, validation, security, and performance directly at the API layer — faster and more stable than UI testing.
- **Postman** is the standard tool for manually building, organizing, and testing API requests, using **Collections** to group related requests.
- **Environments and variables** (`{{base_url}}`, `{{auth_token}}`) allow the same requests to run against dev/staging/production without manual edits.
- **Test scripts** (using the `pm` object) let testers assert status codes, response time, and response body content automatically.
- **Chaining requests** (capturing a token from login and reusing it) and **pre-request scripts** (generating dynamic test data) make test suites realistic and maintainable.
- **Collection Runner** and **Newman (CLI)** allow running full API test suites automatically — Newman is the bridge to CI/CD automation (Step 17).
- Good API test coverage includes **positive, negative, security, boundary, and response-structure** scenarios — not just "does it return 200."
- **Schema validation** ensures the response contract (field names/types) doesn't silently break integrations.
- Understanding **401 vs 403** and different auth methods (Basic, API Key, Bearer/JWT, OAuth 2.0) is essential for testing protected endpoints correctly.
- **Mock servers** support early, shift-left testing before the real backend is ready.