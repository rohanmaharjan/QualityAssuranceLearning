# Testing Types and Levels

Testing types tell us **what aspect** of the software we are testing (functionality, performance, security, etc.), while testing levels tell us **at what stage/scope** of the software we are testing (a single function, a group of components, or the whole system).

> Think of it this way:
> **Testing Levels** = "How much of the system am I testing?" (scope)
> **Testing Types** = "What quality am I checking?" (nature)

---

# Part 1: Testing Levels

Testing levels represent the different stages at which testing is performed during the SDLC, moving from the smallest unit of code to the fully integrated system.

## Levels Overview

Unit Testing
↓
Integration Testing
↓
System Testing
↓
Acceptance Testing


---

## 1. Unit Testing

Testing the smallest individual component (a function, method, or class) of the software in isolation.

- Usually performed by **developers**.
- Other components (database, frontend, external services) are not involved — they are often **mocked** or **stubbed**.
- Uses white-box testing techniques.

### Example

A backend has:

validate_email()
check_password()
authenticate_user()
create_session()


A developer tests `validate_email()` alone:

Input: "test@gmail.com" → Expected: valid
Input: "test.com" → Expected: invalid
Input: "" → Expected: invalid
Input: "test@@gmail.com" → Expected: invalid


### Why it matters
- Catches bugs at the earliest and cheapest point to fix.
- Makes refactoring safer later on.
- Forms the base of the testing pyramid (most tests should be unit tests).

---

## 2. Integration Testing

Testing performed after combining two or more components/modules to check whether they work correctly **together**.

- Individual components may pass their own unit tests but still fail when combined — usually due to incorrect data formats, wrong assumptions, or communication issues.

### Example

For login:

Frontend → Login API → Authentication Service → Database


Flow:

User enters credentials
↓
Frontend sends request
↓
API receives it
↓
API queries database
↓
Database returns user data
↓
API responds
↓
Frontend redirects to dashboard


Even if each piece works alone, integration testing checks things like:
- Does the API correctly parse what the frontend sends?
- Does the database return data in the format the API expects?
- What happens if the authentication service times out?

### Approaches to Integration Testing
- **Big Bang:** Integrate all components at once, then test. Simple but hard to isolate defects.
- **Top-Down:** Test from the top (UI) downward, using stubs for lower modules not yet built.
- **Bottom-Up:** Test from the bottom (database/services) upward, using drivers for higher modules not yet built.
- **Incremental:** Integrate and test module by module, gradually building up the system.

---

## 3. System Testing

Testing the **complete, integrated application** as a whole, against the specified requirements.

- Performed in an environment that closely resembles production.
- Usually black-box testing — testers don't need to know the internal code.

### Example

For an e-commerce website:

Login → Search → Product → Cart → Checkout → Payment → Order → Email Confirmation


System testing checks the entire user journey end-to-end, not just individual pieces.

### What's covered here
- Functional testing (does it do what it should?)
- Non-functional testing (performance, security, usability — as part of full-system checks)
- Compatibility across browsers/devices
- Full business workflows

---

## 4. Acceptance Testing

Asks: **"Does this system satisfy the business/user requirements well enough to be accepted?"**

- Often called **UAT — User Acceptance Testing**.
- Performed by the client, end-users, or business stakeholders — not just QA.
- This is the final validation before release.

### Example

An e-commerce client requested:
> "Users must be able to purchase products using credit cards."

The system may technically pass all system tests, but the business still needs to confirm it actually satisfies **their** real-world need.

### Types of Acceptance Testing
- **UAT (User Acceptance Testing):** End users verify the system meets their needs.
- **Business Acceptance Testing (BAT):** Business checks if it meets business goals/ROI.
- **Alpha Testing:** Internal acceptance testing before releasing to real users.
- **Beta Testing:** External users test a near-final version in real environments.
- **Contract Acceptance Testing:** Verifies the system meets criteria agreed upon in a contract.
- **Regulation Acceptance Testing:** Verifies the system meets legal/regulatory standards.

---

## Testing Levels Comparison Table

| Level | Tested By | Scope | Testing Type Used |
|---|---|---|---|
| Unit Testing | Developers | Single function/component | White-box |
| Integration Testing | Developers/QA | Group of components | Grey-box |
| System Testing | QA | Entire application | Black-box |
| Acceptance Testing | Client/End-users/QA | Business requirements | Black-box |

---

# Part 2: Testing Types

Testing types are categorized based on **what** is being tested — functionality, or how well the system performs.

## Overview

Testing Types
│
├── Functional Testing (What it does)
│
└── Non-Functional Testing (How well it does it)


---

## A) Functional Testing

Tests **what the system does** — verifying features work according to requirements.

**Example:** Does clicking Login authenticate the user?

### Common areas covered:
- Login / Registration
- Search functionality
- Checkout / Payment
- Password reset
- Form validations
- Business logic / calculations

### Sub-types of Functional Testing

| Type | Description |
|---|---|
| **Smoke Testing** | Quick check to verify the build is stable enough for further testing |
| **Sanity Testing** | Quick check to verify a specific fix/feature works after a small change |
| **Regression Testing** | Verifies that new changes haven't broken existing functionality |
| **Retesting** | Verifies that a specific previously-failed defect is now fixed |
| **Integration Testing** | Verifies components work together (also a level, listed above) |
| **Interface Testing** | Verifies communication between two systems (e.g., API contracts) |

---

## B) Non-Functional Testing

Tests **how well the system performs**, rather than what it does.

### 1. Performance Testing

Checks speed, responsiveness, and stability under a workload.

- **Load Testing:** How does the system behave under expected user load?
  - Example: Can 10,000 users browse the site simultaneously?
- **Stress Testing:** How does the system behave beyond normal capacity, or under extreme conditions?
  - Example: What happens at 50,000 concurrent users — does it crash gracefully or corrupt data?
- **Scalability Testing:** Can the system scale up (handle more load) effectively?
- **Spike Testing:** How does the system react to a sudden, sharp increase in load?
  - Example: Flash sale causes traffic to jump from 500 to 20,000 users in a minute.
- **Endurance/Soak Testing:** Can the system sustain expected load over a long period without degrading?
  - Example: Running for 72 hours straight without memory leaks or slowdowns.

### 2. Security Testing

Checks whether the application protects data and resources from unauthorized access and vulnerabilities.

**Example checks:**
- Can an unauthorized user access another user's account?
- Is sensitive data (passwords, tokens) encrypted?
- Is the app vulnerable to SQL injection or XSS?
- Are authentication and authorization implemented correctly?

### 3. Usability Testing

Checks **"How easy and comfortable is the application for users to use?"**

**Example:** Login works, but:
- Login button is difficult to find
- Error messages are confusing
- Password requirements aren't clear
- Navigation is difficult

Considers: ease of use, navigation, learnability, clarity, accessibility, user satisfaction.

### 4. Compatibility Testing

Verifies whether an application works correctly across different environments.

**Checked across:**
- Browsers (Chrome, Firefox, Edge, Safari)
- Operating systems (Windows, macOS, Linux, Android, iOS)
- Devices (mobile, tablet, desktop)
- Screen resolutions

### 5. Reliability Testing

Checks whether the system performs its intended functions consistently over time without failure.

**Example:** An API may work fine for 100–200 requests but fail after 1,000 requests or after running for several hours. A banking application should not randomly crash after continuous operation.

### 6. Recovery Testing

Checks how the system behaves after a failure and whether it can recover properly.

**Possible failures tested:** server crash, database failure, network failure, application crash, power failure.

**Example:** If the server crashes mid-transaction, does the system recover without losing/corrupting data?

### 7. Installation Testing

Verifies that software can be installed, configured, upgraded, and uninstalled safely and correctly.

### 8. Internationalization (i18n) Testing

Verifies the software is **designed** to support different languages, regions, currencies, and date formats (multi-locale readiness).

**Example:** Checking that the UI can support Nepali, English, French, German, etc., without hardcoded text or layout breaks.

### 9. Localization (l10n) Testing

Verifies the software is correctly **adapted** for a specific language/region.

**Example:** For Nepal — language = Nepali, currency = NPR, correct date format used.

### 10. Portability Testing

Checks how well the software can be transferred from one environment/platform to another (related to compatibility, but focused on the transition itself).

---

## Functional vs Non-Functional — Quick Comparison

| Aspect | Functional Testing | Non-Functional Testing |
|---|---|---|
| Focuses on | What the system does | How well the system performs |
| Based on | Business requirements | Quality attributes |
| Example question | Does login work with valid credentials? | Can login handle 10,000 concurrent users? |
| Techniques | Black-box mostly | Performance tools, security scanners, etc. |
| Found via | Manual/automated functional test cases | Specialized tools (JMeter, OWASP ZAP, etc.) |

---

## How Levels and Types Relate

It's important to understand that **levels** and **types** are two different dimensions — you can apply many testing *types* at different testing *levels*.

**Example:** Security testing can happen at multiple levels:

Unit Level → Testing if a single function sanitizes input correctly
Integration Level → Testing if API properly authenticates before hitting DB
System Level → Testing if the full app blocks unauthorized access end-to-end
Acceptance Level → Client verifies the app meets their security compliance needs


Similarly, functional testing happens across **all levels** — from unit tests checking one function's logic, up to acceptance tests checking full business workflows.

---

## Testing Pyramid (How Levels Are Typically Balanced)
    ▲
   / \        Few — Acceptance/UI Tests (slow, expensive, high confidence)
  /---\
 /     \      Some — Integration Tests
/-------\

/ \ Many — Unit Tests (fast, cheap, run constantly)
/___________\


- **Unit tests** should be the most numerous — fast, cheap, catch bugs early.
- **Integration tests** are fewer — check that pieces work together.
- **System/Acceptance/UI tests** are the fewest — slow and expensive, but give the highest confidence that the real user experience works.

> A common mistake is having too many UI-level tests and too few unit tests — this makes the test suite slow, brittle, and expensive to maintain.

---

## Key Takeaways

- **Testing Levels** define **scope**: Unit → Integration → System → Acceptance.
- **Testing Types** define **quality aspect**: Functional vs Non-Functional (Performance, Security, Usability, Compatibility, Reliability, Recovery, Installation, i18n/l10n, Portability).
- Functional testing asks "does it work?" — Non-functional testing asks "does it work *well*?"
- The same testing type (e.g., security) can be tested at multiple levels.
- The testing pyramid suggests: many unit tests, fewer integration tests, even fewer system/UI tests — for a fast and reliable test suite.