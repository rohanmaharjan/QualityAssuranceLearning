# Test Cases & Test Scenarios

This chapter goes deeper into how QA engineers actually **write, structure, organize, and manage** test scenarios and test cases in real projects — beyond just knowing the definitions.

---

# 1. Test Scenario vs Test Case — Quick Recap

| Test Scenario | Test Case |
|---|---|
| High-level "what to test" | Detailed "how to test" |
| One line, broad | Multiple steps, specific |
| Derived from requirements/use cases | Derived from test scenarios |
| Example: "Verify login functionality" | Example: "Verify login with valid email and password" |

> **Relationship:** One test scenario usually expands into multiple test cases.

Test Scenario: Verify login functionality
│
├── Test Case 1: Login with valid credentials
├── Test Case 2: Login with invalid password
├── Test Case 3: Login with unregistered email
├── Test Case 4: Login with empty fields
└── Test Case 5: Login with SQL injection in email field


---

# 2. How to Derive Test Scenarios from Requirements

Deriving good scenarios is a skill — it comes from carefully reading requirements and asking "what needs to be true for this to work correctly?"

### Example Requirement
> "Users should be able to apply a discount coupon at checkout. The coupon must be valid, unexpired, and the order total must meet the minimum requirement."

### Scenarios derived from this one requirement:
Verify coupon applies successfully when all conditions are met
Verify coupon is rejected when expired
Verify coupon is rejected when invalid/incorrect code
Verify coupon is rejected when order total is below minimum
Verify behavior when coupon is applied twice
Verify behavior when item is removed after coupon is applied
Verify UI shows updated total after coupon is applied
Verify coupon field accepts/rejects proper input formats

### A Simple Method: "Requirement → Question → Scenario"

For every requirement, ask:
- What is the **happy path**? (main working scenario)
- What are the **negative cases**? (things that should fail)
- What are the **boundary/edge cases**?
- What are the **security-related** concerns?
- What are the **UI/UX** concerns?
- What happens if a **dependency fails** (network, database, third-party service)?

This question-based approach is how experienced QA engineers avoid missing scenarios, instead of relying only on memory.

---

# 3. Components of a Well-Written Test Case

A good test case should be **clear, specific, and repeatable** — meaning any tester (not just the person who wrote it) can execute it and get a consistent result.

| Field | Purpose |
|---|---|
| **Test Case ID** | Unique identifier (e.g., TC_LOGIN_001) |
| **Test Case Title** | Short, clear description of what's being tested |
| **Module/Feature** | Which part of the application this belongs to |
| **Preconditions** | What must be true before executing the test |
| **Test Steps** | Numbered, exact steps to follow |
| **Test Data** | Specific input values used |
| **Expected Result** | What should happen if the software works correctly |
| **Actual Result** | What actually happened (filled in during execution) |
| **Status** | Pass / Fail / Blocked / Not Executed |
| **Priority** | How important this test case is (High/Medium/Low) |
| **Severity** | (if failed) How serious the resulting defect is |
| **Author / Date** | Who wrote it and when |

### Example — Well-Structured Test Case

Test Case ID: TC_LOGIN_005
Title: Verify login fails with correct email and wrong password
Module: Authentication
Priority: High

Precondition:

User has a registered account (email: test@example.com)
Application is accessible

Test Data:
Email: test@example.com
Password: WrongPass123

Steps:

Navigate to the login page
Enter the registered email
Enter an incorrect password
Click the "Login" button

Expected Result:

Login should fail
Error message "Invalid email or password" should be displayed
User should remain on the login page
No session/token should be created

Actual Result: [Filled during execution]
Status: [Pass/Fail]


### Why "Invalid email or password" (not "Wrong password") matters
Notice the expected result doesn't say "wrong password" — a well-designed system shows a **generic error message** so it doesn't reveal whether the email exists in the system. This is a small but important security-related detail a good tester should catch and write into the expected result.

---

# 4. Characteristics of a Good Test Case

| Characteristic | Meaning |
|---|---|
| **Clear** | Anyone reading it understands exactly what to do |
| **Concise** | No unnecessary steps or information |
| **Reusable** | Can be run repeatedly with consistent results |
| **Independent** | Doesn't rely on another test case's leftover state (when possible) |
| **Traceable** | Linked back to a specific requirement |
| **Accurate** | Expected result matches the actual requirement, not assumptions |

### Common Mistakes to Avoid

❌ Vague steps: "Test the login page"
✅ Specific steps: "Enter 'test@gmail.com' in email field, click Login"

❌ Missing test data: "Enter a valid password"
✅ Exact test data: "Enter password: Test@1234"

❌ Vague expected result: "It should work"
✅ Specific expected result: "User is redirected to /dashboard and welcome message displays username"


---

# 5. Positive, Negative, and Destructive Test Cases

### Positive Test Cases
Verify the system behaves correctly with **valid** input — the "happy path."
Valid email + correct password → successful login
Valid coupon + order above minimum → discount applied

### Negative Test Cases
Verify the system correctly **rejects** invalid input or handles errors gracefully.
Invalid email format → validation error shown
Expired coupon → rejected with clear message
Empty required field → form doesn't submit

### Destructive Test Cases
Intentionally try to **break** the system — often overlaps with security and error-guessing mindset.
SQL injection in input fields
Uploading a corrupted/oversized file
Rapid double-clicking the submit button
Manipulating API requests directly (bypassing UI validation)

---

# 6. Test Data Design

Test data is the actual input values used while executing test cases. Poor test data leads to false confidence — tests that "pass" but don't actually verify anything meaningful.

### Categories of Test Data

| Category | Example (for a "quantity" field, 1–10 allowed) |
|---|---|
| Valid data | 5 |
| Invalid data | -1, 0, 11, "abc" |
| Boundary data | 1, 10, 0, 11 |
| Null/empty data | (blank field) |
| Special characters | `@#$%^&*`, emojis, unicode |
| Large data | 10,000-character string in a name field |
| Security payloads | `' OR '1'='1`, `<script>alert(1)</script>` |

### Where Test Data Comes From
- Manually created by the tester
- Extracted/copied from production (masked for privacy)
- Generated using tools/scripts (for large volumes)
- Boundary Value Analysis / Equivalence Partitioning outputs (from Step 4)

---

# 7. Test Case Prioritization

Not all test cases are equally important, especially with limited time. Prioritization ensures the most critical functionality is tested first.

| Priority | Meaning | Example |
|---|---|---|
| **High** | Core functionality, must-work features | Login, checkout, payment |
| **Medium** | Important but not critical | Sorting products, filters |
| **Low** | Minor/cosmetic features | Tooltip text, minor UI spacing |

### Risk-Based Prioritization
QA often prioritizes based on:
- **Business impact** — how much revenue/users affected if this breaks?
- **Usage frequency** — how often do users interact with this feature?
- **Complexity** — more complex logic = higher chance of defects
- **Change frequency** — recently changed code is more likely to have new bugs

---

# 8. Requirement Traceability Matrix (RTM)

A document that **maps** each requirement to its corresponding test case(s), ensuring nothing is missed and every requirement has test coverage.

### Example RTM

| Requirement ID | Requirement Description | Test Case ID(s) | Status |
|---|---|---|---|
| REQ_001 | User can log in with valid credentials | TC_LOGIN_001 | Pass |
| REQ_002 | User account locks after 5 failed attempts | TC_LOGIN_010, TC_LOGIN_011 | Pass |
| REQ_003 | User can reset password via email | TC_PWD_001 to TC_PWD_006 | In Progress |
| REQ_004 | Coupon applies only if order > $100 | TC_COUPON_003 | Fail |

### Why RTM Matters
- Confirms **every requirement has at least one test case** (no coverage gaps)
- Helps quickly identify the **impact of a requirement change** — which test cases need updating
- Useful during audits or client reviews to prove testing coverage

---

# 9. Test Case Review Process

Before test cases are executed, they're often reviewed by peers or leads to catch mistakes early — this is a form of **static testing** (reviewing without executing).

### What Reviewers Check
- Are all requirements covered?
- Are steps clear and unambiguous?
- Is test data appropriate?
- Are expected results accurate and specific?
- Are negative/edge cases included, not just happy path?
- Any duplicate or redundant test cases?

### Review Outcomes

Approved → Ready for execution
Needs Rework → Sent back with comments
Rejected/Not Needed → Removed from the suite (e.g., duplicate)


---

# 10. Organizing Test Cases into a Test Suite

A **Test Suite** is a collection of test cases grouped together, usually by feature, module, or testing type.

### Example Test Suite Structure

Test Suite: E-commerce Regression Suite
│
├── Login Module
│ ├── TC_LOGIN_001 – Valid login
│ ├── TC_LOGIN_002 – Invalid password
│ └── TC_LOGIN_003 – Account lock after 5 attempts
│
├── Cart Module
│ ├── TC_CART_001 – Add product to cart
│ └── TC_CART_002 – Remove product from cart
│
└── Checkout Module
├── TC_CHECKOUT_001 – Apply valid coupon
└── TC_CHECKOUT_002 – Payment failure handling


Suites are often further grouped by purpose:

Smoke Suite → Small set of critical tests, run on every build
Regression Suite → Larger set, run before releases
Sanity Suite → Focused tests for a specific fix
Full Suite → Complete test case set for major releases


---

# 11. Writing Test Cases from a User Story (Agile Context)

In Agile teams, requirements often come as **user stories** rather than formal specification documents.

### Example User Story
> **As a** registered user
> **I want to** reset my password
> **So that** I can regain access to my account if I forget it

### Acceptance Criteria (usually provided with the story)
User can request a password reset using their registered email
Reset link is valid for 30 minutes
Reset link can only be used once
Password must meet complexity requirements
User receives confirmation after successful reset

### Test Cases derived from Acceptance Criteria

TC01 → Request reset with valid registered email
TC02 → Request reset with unregistered email
TC03 → Use reset link within 30 minutes → success
TC04 → Use reset link after 30 minutes → expired error
TC05 → Use reset link twice → second attempt fails
TC06 → Set new password not meeting complexity rules → rejected
TC07 → Successfully reset password → confirmation shown/sent


> Notice how **acceptance criteria map almost directly to test cases** — this is one of the most useful skills in Agile QA work.

---

## Key Takeaways

- A **test scenario** is the "what," a **test case** is the detailed "how" — one scenario often produces many test cases.
- Deriving scenarios is a skill: for every requirement, systematically check happy path, negative cases, boundaries, security, UI/UX, and failure dependencies.
- A well-written test case is clear, specific, reusable, independent, and traceable — vague steps or vague expected results reduce its value.
- Test cases fall into **positive**, **negative**, and **destructive** categories — all three are needed for real coverage.
- Test data should be deliberately designed (valid, invalid, boundary, null, special characters, security payloads) — not randomly picked.
- Prioritize test cases by **business impact, usage frequency, complexity, and change frequency** when time is limited.
- An **RTM** ensures every requirement has traceable test coverage and helps manage the impact of requirement changes.
- Test cases go through a **review process** before execution to catch gaps early (a form of static testing).
- Test cases are grouped into **test suites** (smoke, sanity, regression, full) for organized execution.
- In Agile, **acceptance criteria** in user stories map almost directly to test cases — a key practical skill.