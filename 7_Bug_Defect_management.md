# Bug/Defect Management

Bug (or Defect) Management is the structured process of identifying, reporting, tracking, fixing, and verifying defects throughout the software testing lifecycle. A defect that isn't properly reported or tracked can easily get lost, ignored, or reintroduced later.

---

# 1. Key Terminology Recap

| Term | Meaning |
|---|---|
| **Error** | A mistake made by a person (developer writes `>` instead of `>=`) |
| **Defect/Bug** | The resulting mistake in the software code |
| **Failure** | The software actually behaving incorrectly when the defect is triggered |

> An error in code becomes a defect. A defect, when executed, causes a failure that the user/tester observes.

---

# 2. Defect Life Cycle (Bug Life Cycle)

The Defect Life Cycle describes the various states a defect goes through, from the moment it's identified until it's closed.

## Standard Defect Life Cycle States

New
↓
Assigned
↓
Open (In Progress)
↓
Fixed
↓
Retest / Pending Retest
↓
Verified
↓
Closed


With branch paths for:

Reopened → If retest fails
Rejected → If not a valid defect
Deferred → If fix is postponed to a later release
Duplicate → If already reported
Not Reproducible → If defect can't be recreated


## Detailed Explanation of Each State

### 1. New
Tester finds a defect and logs it for the first time. It hasn't been reviewed yet.

### 2. Assigned
The defect is reviewed (often by a lead or the development team) and assigned to a specific developer to fix.

### 3. Open / In Progress
The developer is actively working on fixing the defect.

### 4. Fixed
The developer has made a code change believed to resolve the defect and has deployed it to the test build.

### 5. Pending Retest
The defect is waiting for QA to verify the fix.

### 6. Retest
QA re-executes the original failing test case to check whether the issue is actually resolved.

### 7. Verified
QA confirms the fix works correctly — the defect is closed as verified.

### 8. Closed
The defect is fully resolved and confirmed. No further action needed.

### 9. Reopened
If the retest fails (bug still exists or fix caused a new problem), the defect goes back to the developer.

Retest → Fails → Reopened → back to Assigned/Open


### 10. Rejected
The development team determines this is not actually a defect — e.g., it's working as designed, or it's a misunderstanding of requirements.

### 11. Deferred
The defect is valid but the team decides to postpone the fix to a future release (often low priority/severity, or time constraints).

### 12. Duplicate
The defect has already been reported previously under a different ID.

### 13. Not Reproducible
The developer cannot recreate the issue using the steps provided — needs more information from QA, or may not be a real defect (e.g., environment-specific issue).

## Full Defect Life Cycle Diagram
                ┌────────┐
                │  New   │
                └───┬────┘
                    ↓
             ┌──────────────┐
             │  Assigned    │
             └──────┬───────┘
                    ↓
          ┌─────────────────┐
          │  Open/In Progress │
          └─────────┬─────────┘
                    ↓
    ┌───────────────┼───────────────┬───────────────┐
    ↓               ↓               ↓               ↓
┌───────┐      ┌──────────┐    ┌──────────┐   ┌────────────────┐
│ Fixed │      │ Rejected │    │ Deferred │   │ Not Reproducible│
└───┬───┘      └──────────┘    └──────────┘   └────────────────┘
    ↓

┌───────────┐
│ Retest │
└─────┬─────┘
↓
┌────┴─────┐
↓ ↓
Pass Fail
↓ ↓
┌────────┐ ┌──────────┐
│Verified│ │ Reopened │──────┐
└───┬────┘ └──────────┘ │
↓ │
┌────────┐ │
│ Closed │ │
└────────┘ │
↑ │
└────────────────────────┘
(back to Assigned/Open after reopening)


---

# 3. Components of a Good Defect/Bug Report

A defect report must contain enough information for a developer to **understand, reproduce, and fix** the issue without needing to ask the tester follow-up questions every time.

| Field | Purpose |
|---|---|
| **Defect ID** | Unique identifier |
| **Title/Summary** | Short, clear one-line description |
| **Description** | Detailed explanation of the issue |
| **Steps to Reproduce** | Exact numbered steps to recreate the bug |
| **Expected Result** | What should happen |
| **Actual Result** | What actually happened |
| **Severity** | How seriously it affects the system |
| **Priority** | How urgently it needs to be fixed |
| **Environment** | Browser, OS, device, build/version |
| **Screenshots/Videos/Logs** | Visual or technical evidence |
| **Reported By / Date** | Who found it and when |
| **Status** | Current state in the defect life cycle |

## Example — Well-Written Bug Report

Defect ID: BUG_1042
Title: Login fails with valid credentials when "Remember Me" is checked
Severity: High
Priority: High
Environment: Chrome 128, Windows 11, Staging build v2.3.1

Description:
When a user checks the "Remember Me" checkbox before logging in,
the login fails even with valid credentials.

Steps to Reproduce:

Navigate to the login page
Enter valid registered email: test@example.com
Enter valid password: Test@1234
Check the "Remember Me" checkbox
Click "Login"

Expected Result:
User should be logged in successfully and redirected to the dashboard.

Actual Result:
"Invalid credentials" error is displayed, and user remains on the login page.

Additional Notes:

Issue does NOT occur when "Remember Me" is left unchecked
Reproduced consistently across 5 attempts
Screenshot and browser console log attached

Reported By: [Tester Name]
Date: [Date]
Status: New


### Why this report is good
- Title is specific (mentions the exact condition that triggers it), not just "Login doesn't work"
- Steps are precise and reproducible
- Includes the important detail that isolates the cause ("Remember Me" checked vs unchecked)
- Includes environment info so the developer can match conditions

---

# 4. Severity vs Priority (In Depth)

These two are commonly confused, and interviewers frequently test this distinction.

| | Severity | Priority |
|---|---|---|
| **Definition** | How seriously the defect affects the system | How urgently it needs to be fixed |
| **Decided by** | QA/Tester (based on technical impact) | Product Manager/Business (based on business urgency) |
| **Focus** | Technical impact | Business impact/urgency |

## Severity Levels

| Level | Meaning | Example |
|---|---|---|
| **Critical** | System crash, data loss, complete feature failure | Application crashes on launch |
| **High** | Major functionality broken, no workaround | Payment fails for all users |
| **Medium** | Functionality broken but workaround exists | Sorting doesn't work, but filtering does |
| **Low** | Minor issue, cosmetic | Button color slightly off |

## Priority Levels

| Level | Meaning |
|---|---|
| **P1 (Urgent)** | Must be fixed immediately |
| **P2 (High)** | Fix soon, before release |
| **P3 (Medium)** | Fix when possible |
| **P4 (Low)** | Fix if time permits |

## Four Classic Severity/Priority Combinations

### High Severity + High Priority
Payment gateway completely crashes during checkout.
→ Major impact, needs immediate fix.

### High Severity + Low Priority
A rarely-used admin report crashes, but it's used only once a year and doesn't affect regular users.
→ Serious technically, but not urgent right now.

### Low Severity + High Priority
Company logo is broken on the homepage, one hour before a major product launch.
→ Minor technically, but urgent due to business/branding impact.

### Low Severity + Low Priority
A typo in a rarely-visited help page.
→ Minor and not urgent.

> **Interview tip:** The classic example interviewers expect is the "logo is wrong before a big launch" case — it perfectly demonstrates that severity and priority are independent of each other.

---

# 5. Defect Classification (Types of Defects)

| Category | Description | Example |
|---|---|---|
| **Functional Defect** | Feature doesn't work as expected | Login button doesn't respond |
| **UI/Cosmetic Defect** | Visual issue, doesn't affect functionality | Misaligned button, wrong font size |
| **Performance Defect** | System is slow or unresponsive | Page takes 15 seconds to load |
| **Security Defect** | Vulnerability found | SQL injection possible in search field |
| **Compatibility Defect** | Works on one environment but not another | Layout breaks in Safari but works in Chrome |
| **Usability Defect** | Confusing or hard to use, even if functionally correct | Error message unclear to users |
| **Logical Defect** | Incorrect business logic | Discount calculated incorrectly |

---

# 6. Root Cause Analysis (RCA)

After a defect is fixed, teams sometimes analyze **why** it happened in the first place — not just to fix the symptom, but to prevent similar defects in the future.

### Common Root Causes
Missing or unclear requirements
Miscommunication between team members
Incorrect assumptions made during development
Insufficient unit testing
Edge cases not considered during design
Environment/configuration issues
Third-party/dependency failures

### Example RCA

**Defect:** Discount not applied when order total is exactly $100 (boundary should be inclusive).

**Root Cause:** Developer used `total > 100` instead of `total >= 100`, based on ambiguous requirement wording that didn't clarify whether $100 exactly should qualify.

**Preventive Action:** Requirements should explicitly state boundary conditions (e.g., "orders of $100 or more") to avoid ambiguity in future features.

---

# 7. Defect Metrics (Used to Measure Testing Quality)

## Defect Density

Defect Density = Total Defects Found / Size of Module (e.g., per 1000 lines of code, or per feature)

Helps identify which modules are the most defect-prone.

## Defect Leakage

Defect Leakage = Defects Found in Production / Total Defects Found (Testing + Production)

Measures how many defects "escaped" testing and were only found after release — lower is better, and indicates stronger testing effectiveness.

## Defect Removal Efficiency (DRE)

DRE = (Defects Found During Testing / (Defects Found During Testing + Defects Found After Release)) × 100

Shows how effective the testing process was at catching issues before release.

## Example

Defects found during testing: 90
Defects found after release: 10

DRE = (90 / (90 + 10)) × 100 = 90%

A DRE of 90% means the QA process caught 90% of all defects before the software reached users.

---

# 8. Defect Prevention vs Defect Detection

| Defect Prevention | Defect Detection |
|---|---|
| Stopping defects before they happen | Finding defects after they exist |
| Requirement reviews, design reviews, coding standards | Test execution, exploratory testing |
| Proactive | Reactive |
| Cheaper (fixing early is cheaper) | More expensive as defects are found later |

> The earlier a defect is caught in the SDLC, the cheaper it is to fix — this is why static testing (reviews) and early QA involvement (from the Requirement Analysis phase) are so valuable.

### Cost of Fixing Defects by Phase (General Principle)

Requirement Phase → Cheapest to fix
Design Phase → Slightly more expensive
Development Phase → More expensive
Testing Phase → Even more expensive
Production Phase → Most expensive (data loss, reputation damage, hotfixes)


---

# 9. Defect Triage

A meeting/process where the team (QA lead, developers, product managers) reviews newly reported defects to decide:
- Is this a valid defect?
- What's the correct severity and priority?
- Who should fix it?
- Should it be fixed now, or deferred?

### Typical Triage Questions
Is this reproducible?
How many users does this affect?
Is there a workaround?
Does this block release?
Is this related to another known issue?

---

## Key Takeaways

- The **Defect Life Cycle** moves through New → Assigned → Open → Fixed → Retest → Verified → Closed, with branches for Reopened, Rejected, Deferred, Duplicate, and Not Reproducible.
- A good bug report includes precise steps to reproduce, expected vs actual result, severity, priority, and environment details — vague reports waste time.
- **Severity** is about technical impact (decided by QA); **Priority** is about business urgency (decided by PM/business) — the two are independent.
- Defects can be classified as functional, UI/cosmetic, performance, security, compatibility, usability, or logical.
- **Root Cause Analysis** looks beyond the fix itself to prevent similar defects from recurring.
- Key defect metrics: **Defect Density**, **Defect Leakage**, and **Defect Removal Efficiency (DRE)** measure testing effectiveness.
- Defects found earlier in the SDLC are cheaper to fix — this is why defect **prevention** (reviews) is valued alongside defect **detection** (testing).
- **Defect Triage** is a team process to validate, prioritize, and assign newly found defects.