# Software Testing Notes

## What is Software Testing?

Software testing is the process of evaluating software to find defects and verify that it behaves according to specified requirements.

> Testing = checking whether the software does what it is supposed to do, and discovering what it does wrong.

---

## Verification vs Validation

### Verification
**"Are we building the product right?"**

Checks whether the software is being developed according to specifications/designs/requirements.

- Reviewing requirements
- Reviewing design documents
- Code reviews
- Documentation reviews

### Validation
**"Are we building the right product?"**

Checks whether the actual software satisfies the user's needs and requirements.

- Running login tests
- Testing checkout
- Testing registration
- Testing APIs

---

## QA vs Testing

### QA (focuses on preventing problems)
- Process-oriented
- Goal: prevent defects and improve the software development process
- Defining testing processes, establishing coding/testing standards, process improvement, reviewing development practices

### Testing (focuses on finding problems)
- Product-oriented
- Goal: find the defects in the actual software

---

## Q1: Login Page Testing Scenario

> Imagine you are given this application:
> A login page has:
> - Email field
> - Password field
> - Login button
> - "Forgot Password" link
>
> A developer tells you: *"The login feature is finished. Please test it."*
>
> As a QA engineer, what would you test?

**Answer:** There will always be a positive scenario (things that should work) and a negative scenario (things that should fail correctly).

### Positive Scenarios
- Valid email + correct password
- Valid credentials redirect to user's dashboard
- "Remember me" works if available

### Negative Scenarios
- Invalid email format
- Unregistered email
- Correct email + wrong password
- Empty email or password or both fields
- Extremely long email
- Password below minimum length or maximum length

### Security-related Scenarios
- Can someone login without a password?
- Can the password be exposed?
- Does it reveal if an email exists?
- How many failed attempts can be made?
- Can a user access another user's account?

### UI/UX Scenarios
- Login button visibility?
- Are error messages understandable?
- Does the page work on mobile?
- Does pressing Enter submit the form?
- Does "Forgot Password" work?

> **Important Note:**
> Don't only think: "What should I test?"
> Also think: "What can go wrong?"
>
> For example, password requirements = at least 8 characters
> Don't just test: 8 characters → works
> Also test: spaces, special characters, unicode, SQL injection, XSS payload, copy/paste, leading/trailing spaces

---

## What is a Defect/Bug?

A defect is a problem in the software where the actual behavior differs from the expected behavior or requirement.

**Example:**
Password must contain at least 8 characters.
You enter Password: `abcd`
The application allows you to log in. ❌

### Error vs Bug
- **Error:** A mistake made by a person (e.g., developer writes `>` instead of `>=`)
- **Defect/Bug:** The mistake in the software
- **Failure:** The software actually behaves incorrectly when the defect is executed

### Expected vs Actual Result
- **Requirement:** After successful login, the user should be redirected to `/dashboard`.
- **Test:** Enter valid credentials and click login button.
- **Expected:** User is redirected to the dashboard.
- **Actual:** User remains on the login page.

---

## Test Scenario vs Test Case

- **Test scenario** (What to test?): A high-level thing you want to test.
  - Example: *Test login functionality.*
- **Test case** (How exactly we test?): A specific set of steps, inputs, and expected results used to verify something.

**Example Test Case: Verify login with valid credentials**

**Precondition:** User has a registered account.

**Steps:**
1. Open login page.
2. Enter registered email.
3. Enter correct password.
4. Click Login.

**Expected:** User is redirected to dashboard.

### Test Scenario Example

Suppose we're testing an e-commerce website.

**Scenario:** Verify shopping cart functionality.

From one scenario, we can create multiple test cases:
- Add one product
- Add multiple products
- Remove product
- Increase quantity
- Decrease quantity
- Add out-of-stock product
- Cart persistence after refresh
- Cart persistence after logout/login
- Price calculation
- Tax calculation
- Discount calculation
- Empty cart
- Maximum quantity

---

## Functional vs Non-Functional Testing

### Functional Testing (What it does?)
Tests what the system does.
- Example: Does clicking Login authenticate the user?
- Login, Registration, Search, Checkout, Payment, Password reset

### Non-Functional Testing (How well it does?)
Tests how well the system performs.
- **Performance:** Can 10,000 users access the application simultaneously?
- **Security:** Can an unauthorized user access another user's account?
- **Usability:** Can users easily understand the interface?
- **Compatibility:** Does the application work on different browsers and devices?
- **Reliability:** Does the system remain stable for long periods?

---

## Black Box vs White Box vs Grey Box Testing

### Black Box Testing
You test the software without knowing its internal code/implementation.
- Example: Input → Login application → Output
- You don't care if it is developed using Py, JS, C++ — you only care about the behavior of the system.
- Manual functional testing is often black-box testing.

### White Box Testing
You have knowledge of the internal implementation/code.

```python
if password == stored_password:
    login()
```

---

- Developers commonly perform a lot of white-box testing through unit tests.

### Grey Box Testing
You have partial knowledge of the internal system (API architecture, database structure, authentication mechanism).

---

## Manual vs Automation Testing

### Manual Testing
A human performs the test.
- Open browser → Enter email → Enter password → Click login → Check results

### Automation Testing
A program performs the testing.
- Automation script → Open browser → Enter credentials → Click login → Verify dashboard

> **Note:** Automation is excellent for repetitive tasks, but it doesn't replace QA thinking.

---

## Smoke Testing

Imagine developers deploy a new build. Before spending 5 hours testing every feature, QA first asks:

**"Is this build stable enough to test?"**

That's where Smoke Testing comes in.

**Example flow:**
Can application open? → Can user login? → Can user search? → Can product be added to cart? → Can checkout open?

If the login itself is completely broken, there's no value in spending hours testing advanced features.

> **Smoke testing** is a set of basic tests performed to verify that the major functionality of a build is working and that the build is stable enough for further testing.

---

## Sanity Testing

Now imagine developers fix one specific bug:

**"Checkout discount calculation was incorrect."**

A new build is provided. You don't necessarily test the entire application from scratch — you check: *Does the particular change/fix work correctly?*

---

## Retesting vs Regression Testing

Suppose there's a bug: *Login fails with a valid password.* Developer fixes it.

### Retesting
You test that exact bug again.
- Previously failed test → Developer fixed bug → Run same test again
- **Purpose:** Verify that the specific defect has been fixed.

### Regression Testing
You test other existing functionality to ensure the fix didn't break anything else.
- **Purpose:** Did the fix break something else?

---

## Levels of Testing

### A) Unit Testing
Testing the smallest individual component of the software in isolation.

**Example:** A backend has:
- `validate_email()`
- `check_password()`
- `authenticate_user()`
- `create_session()`

A developer might test `validate_email()` in isolation — the database, frontend, and other components aren't necessarily involved.

### B) Integration Testing
Now we combine multiple components and check whether they work correctly together.

**Example:** For login
`Frontend → Login API → Authentication service → Database`

Even though each component works individually, they may not work together if they don't communicate correctly.

Flow: User enters username and password → frontend sends the request → API receives it → API queries database → database returns user → API responds → frontend redirects.

### C) System Testing
Now we test the complete integrated application as a whole.

**Example:** For an e-commerce website:
`Login → Search → Product → Cart → Checkout → Payment → Order → Email confirmation`

We're testing the complete system against its requirements.

### D) Acceptance Testing
Now we ask: **"Does this system satisfy the business/user requirements well enough to be accepted?"**

This is often associated with **UAT — User Acceptance Testing**.

**Example:** An e-commerce client requested: *"Users must be able to purchase products using credit cards."* The system may technically work, but the business/user needs to verify that it actually satisfies their requirements.

---

## Static vs Dynamic Testing

### Static Testing
Testing without executing the software. It involves reviewing requirements, design documents, test plans, source code, and documentation.

**Example:**
Requirement says: *"Password must contain at least 8 characters."*
During a requirement review, you notice another document says: *"Password must contain at least 6 characters."*
You can identify this inconsistency before the application is even built — this is called static testing.

**Static testing techniques:**
- **Review:** People examine a document/code to find issues.
- **Walkthrough:** The author walks others through the document/code.
- **Inspection:** A formal, structured review designed to identify defects.
- **Static analysis:** Tools analyze source code without executing it.

### Dynamic Testing
Involves executing the software and observing its behavior — like running the application.

**Example:** Open login page → Enter email → Enter password → Click login → Observe result

---

## Test Design Techniques

### A) Equivalence Partitioning
Suppose age must be between 18 and 60.

A beginner might test 18, 19, 20, 21, ... 60 — that's unnecessary. Instead, divide the possible inputs into groups/classes where the system should behave similarly.

- **Valid Partition:** 18–60
- **Invalid Partition:** <18 and >60

Instead of testing many values, we could select representatives like: below valid range, valid range, above valid range. We assume values in the same partition will behave similarly.

### B) Boundary Value Analysis
Specifically focuses on the edges.

**Example:** age = 18–60
The important values are: 17, 18, 19, 59, 60, 61

This is useful because programmers usually make mistakes around boundaries (e.g., `if age > 18:` instead of `if age >= 18:`).

### C) Decision Table Testing
Imagine a login system with these rules:
- Email must be registered.
- Password must be correct.
- Account must be active.

A user's login result depends on multiple conditions simultaneously — this is where Decision Table Testing becomes useful.

**We create conditions:**

| Condition | C1 |
|---|---|
| Registered email? | Yes/No |
| Correct password? | Yes/No |
| Account active? | Yes/No |

Then determine the expected result for each combination. It becomes especially useful when there are many combinations of conditions — used when requirements contain AND, OR, multiple conditions, discounts, permissions, authentication rules.

### D) State Transition Testing
Some systems don't simply have Input → Output. Instead, the system can exist in different states, and an action causes it to move from one state to another.

**Example:** Banking system
`Active → Withdraw → Active`

But suppose the user enters the wrong PIN three times:
`Active → Wrong PIN → Wrong PIN → Wrong PIN → Locked`

Entering a fourth PIN shouldn't behave as the first incorrect PIN.

`Wrong → Wrong → Right → Wrong → Wrong`

If the requirement says "3 consecutive failed attempts," it should not lock the account after the third wrong attempt overall, because the successful login reset the consecutive-failure count.

### E) Use Case Testing
A use case describes how a user interacts with a system to accomplish a goal.

**Example:**
**Use case:** User logs into the application.
**Basic Flow:** Open login page → Enter email → Enter password → Click login → System validates credentials → Dashboard

This is the primary flow, but it's also important to consider alternative flow (incorrect credentials) and exceptional flow (database not available).

### F) Error Guessing
Using your experience and intuition to predict where defects are likely to occur. There isn't necessarily a formal formula.

You ask: **"Where would a developer probably make a mistake?"**

For a login form, you might test: empty fields, spaces, very long input, special passwords, wrong password, case sensitivity, copy/paste, rapid repeated clicks, browser back button, refresh after login, session handling.

These are common places where software can behave unexpectedly.

### G) Exploratory Testing
Instead of following only predefined test cases, you learn about the application, design tests, and execute them simultaneously — exploring the application and following interesting behavior.

**Example:**
Open website → Try normal login → Notice unusual error → Investigate error → Try different input → Discover another behavior → Investigate further

> Exploratory testing is an approach where learning about the application, designing tests, and executing tests happen simultaneously rather than strictly following predefined test cases.

---

## Ad-hoc Testing

Informal testing performed without predefined test cases or a structured test plan. The tester uses experience, intuition, and experimentation to find defects (similar to error guessing).

---

## Alpha Testing (Internal)

Performed before the software is released to external users, usually in a controlled environment.

Typically involves:
- Internal QA
- Developers
- Internal stakeholders/users

**Flow:** Development → Alpha testing → Beta testing → Production release

**Goal:** Identify major defects before the software reaches external users.

---

## Beta Testing (External)

Performed by a limited group of external or real users before the final release.

**Example:** A company might release a beta version of an application to 1,000 selected users. Those users use it in real-world environments and provide feedback.

---

## Compatibility Testing

Verifies whether an application works correctly across different environments — browsers, OS, devices.

---

## Usability Testing

Focuses on **"How easy and comfortable is the application for users to use?"**

A feature can be functionally correct but have poor usability.

**Example:** Login works, but:
- Login button is difficult to find
- Error messages are confusing
- Password requirements aren't clear
- Navigation is difficult

It's always necessary to consider ease of use, navigation, learnability, clarity, accessibility, and user satisfaction.

---

## Reliability Testing

Checks whether the system can perform its intended functions consistently over time without failure.

Imagine an API that works perfectly once — that doesn't necessarily mean it's reliable.

**Example:** An API may work for 100 or 200 requests but may fail at 1,000 requests or after running for a few hours. A banking application should not randomly crash after running continuously for several hours.

---

## Recovery Testing

Checks how the system behaves after a failure.

**Possible failures:** server crash, database failure, network failure, application crash, power failure

**Example:** How does the system behave when it's restarted after a system failure?

---

## Installation Testing

Verifies that software can be installed, configured, upgraded, and uninstalled safely and correctly.

---

## Internationalization (i18n)

Designing the software so that it can support different languages, regions, currencies, date formats, and other regional requirements. The main idea is to prepare software to support multiple locales.

**Example:** An application can be designed to support Nepali, English, French, German, etc.

---

## Localization (l10n)

Adapting the software for a specific language or region.

**Example:** For Nepal — language = Nepali, currency = NPR, date formats adjusted accordingly.

---

## Severity vs Priority

- **Severity:** "How seriously does the defect affect the system?"
  - Example: Payment completely crashes → **High Severity**
- **Priority:** "How urgently should the defect be fixed?"
  - Example: Company logo is wrong on the homepage one hour before a major launch → **Low severity, but very high priority**