# STLC — Software Testing Life Cycle

## What is STLC?

STLC stands for **Software Testing Life Cycle**.

STLC is a structured process that defines the testing activities performed by the QA team throughout the software testing process.

> **SDLC describes how software is developed, while STLC describes how software is tested.**

STLC focuses on:
- What should be tested?
- How should it be tested?
- When should it be tested?
- Who will test it?
- What tools and environments are required?
- When can testing be considered complete?

---

## STLC Phases (Overview)
Requirement Analysis
        ↓   
  Test Planning
        ↓
Test Case Development
        ↓
Test Environment Setup
        ↓
  Test Execution
        ↓
  Test Closure


---

## 1. Requirement Analysis

The first phase of STLC. QA analyzes the software requirements to understand:

- What needs to be tested?
- What functionality should the system provide?
- What are the business rules?
- Are the requirements clear, complete, and testable?
- Are there any missing requirements, ambiguities, or contradictions?
- What are the possible risks?

QA can participate in requirement analysis **before development starts**.

### Example

**Requirement:** "Users should be able to reset their password using their registered email."

QA should not simply accept this requirement. QA should ask:
- What happens if the email is not registered?
- What happens if the email field is empty?
- What happens if the user enters an invalid email?
- How long is the reset link valid?
- What happens if the reset link expires?
- What happens if the user enters an incorrect OTP?
- What happens if the user requests multiple reset links?
- What happens after the password is successfully changed?
- What password rules should be applied?

These questions help identify missing requirements and possible test scenarios.

### QA Perspective

QA reviews requirements to make sure they are: clear, complete, consistent, unambiguous, testable, and feasible.

QA also identifies: risks, dependencies, edge cases, missing requirements, and questions that need clarification.

### Deliverables

- Requirement analysis notes
- Requirement Traceability Matrix (RTM)
- List of testable requirements
- Requirement clarification questions
- Initial test scenarios
- Identified risks and dependencies

---

## 2. Test Planning

The phase where QA determines: **How will we test the software?**

The QA team creates the overall testing approach and decides what resources and activities are required.

### Things Considered During Test Planning
- Testing scope
- Testing objectives
- Testing types
- Testing approach
- Resources
- Roles and responsibilities
- Testing schedule
- Testing tools
- Test environment
- Test data
- Risks
- Entry criteria
- Exit criteria

### Example

Suppose we are testing an e-commerce website. The test plan might include:

| Testing Type | Included? |
|---|---|
| Functional Testing | Yes |
| UI Testing | Yes |
| API Testing | Yes |
| Database Testing | Yes |
| Regression Testing | Yes |
| Performance Testing | Yes |
| Security Testing | Basic |
| Automation Testing | Yes |

**Testing tools might be:**

| Purpose | Tool |
|---|---|
| API Testing | Postman |
| UI Automation | Playwright |
| Test Management | Jira / Test Management Tool |
| Version Control | Git / GitHub |

### QA Perspective

QA decides:
- What will be tested? What will not be tested?
- Which testing types are required?
- How much time is needed? Who will perform the testing?
- Which tools and environment are required?
- What are the major risks?
- What conditions must be satisfied before testing starts/ends?

### Deliverable
- **Test Plan**

---

## 3. Test Case Development

After understanding the requirements and creating the test plan, QA starts preparing the actual test cases, which define:
- What should be tested?
- What steps should be followed?
- What data should be used?
- What result is expected?

### Example — Feature: User Login
TC01 → Login with valid email and password
TC02 → Login with invalid password
TC03 → Login with invalid email
TC04 → Login with unregistered email
TC05 → Login with empty email
TC06 → Login with empty password
TC07 → Login with both fields empty
TC08 → Verify password masking
TC09 → Verify Forgot Password functionality
TC10 → Verify account lock after multiple failed attempts


### Example Test Case

**Test Case ID:** TC01
**Test Case:** Verify login with valid credentials
**Precondition:** User must have a registered account

**Steps:**
1. Open the login page
2. Enter a valid email
3. Enter a valid password
4. Click Login

**Expected Result:** User should be successfully logged in and redirected to the dashboard.

### QA Also Prepares
- Test cases
- Test scenarios
- Test data
- Automation scripts (when applicable)

### QA Perspective

QA should make sure test cases provide sufficient coverage of: positive scenarios, negative scenarios, boundary conditions, edge cases, business rules, and error handling.

---

## 4. Test Environment Setup

The phase where QA prepares and verifies the environment required for testing.

Application
↓
Web Server
↓
Backend/API
↓
Database
↓
External Services
↓
Browser / Device


### Example

Suppose we need to test: *"User receives a password reset email."*

The testing environment should have:
- Application running
- Database available
- Email service configured
- Test user account available
- Internet/network available
- Required browser available

> If the email service is not configured, QA cannot properly test the password reset functionality.

### QA Verifies
- Application is deployed
- Required services are running
- Database is available
- APIs are accessible
- Test accounts and test data are available
- Required browsers/devices are available
- Required tools are installed
- External dependencies are working

### QA Perspective

Before starting actual testing, QA should make sure the environment is stable and ready. Sometimes QA performs a basic smoke test to verify the build is stable enough for detailed testing.

---

## 5. Test Execution

The phase where QA actually executes the prepared test cases.

Execute Test Case
↓
Compare Actual Result with Expected Result
↓
Pass / Fail


### Example

**Expected:** User enters valid email + valid password → User should reach Dashboard

**Actual:** User enters valid email + valid password → "Invalid credentials" error appears

**Result:** Status → **FAILED**

QA then reports the defect.

### Defect Workflow
Test Case Execution
↓
Test Failed
↓
Bug / Defect Report
↓
Developer Fixes Bug
↓
QA Retests
↓
Pass?
↙ ↘
Yes No
↓ ↓
Close Reopen / Report Again


### Testing Activities During Execution

QA may perform: functional testing, regression testing, integration testing, system testing, UI testing, API testing, database testing, compatibility testing, exploratory testing, automation testing, smoke testing, sanity testing.

### QA Perspective

QA should: execute test cases, record actual results, mark Pass/Fail, report defects, retest fixed defects, perform regression testing, track execution progress, maintain evidence, and communicate testing status.

---

## 6. Test Closure

The final phase of STLC. This occurs when testing activities are completed according to the agreed criteria. QA evaluates the overall testing process and determines whether the testing objectives were achieved.

### QA Reviews
- Were all planned test cases executed?
- How many passed/failed?
- How many defects were found/fixed?
- Are there any open critical/high defects?
- Were the exit criteria satisfied?
- What risks remain? What lessons were learned?

### Example
Total Test Cases → 250
Executed → 250
Passed → 235
Failed → 15
Defects Found → 18
Critical Open Defects → 0
High Open Defects → 0

Testing Status → Completed


Based on the agreed release criteria, the team can decide whether the software is ready for release.

### Test Closure Report

QA may prepare a Test Closure Report containing: testing summary, test execution results, defect summary, test coverage, open defects, risks, lessons learned, and overall testing status.

---

## STLC Complete Flow
┌────────────────────────────┐
│ Requirement Analysis │
│ What should we test? │
└─────────────┬──────────────┘
↓
┌────────────────────────────┐
│ Test Planning │
│ How will we test? │
└─────────────┬──────────────┘
↓
┌────────────────────────────┐
│ Test Case Development │
│ What will we test? │
└─────────────┬──────────────┘
↓
┌────────────────────────────┐
│ Environment Setup │
│ Where will we test? │
└─────────────┬──────────────┘
↓
┌────────────────────────────┐
│ Test Execution │
│ Does it work? │
└─────────────┬──────────────┘
↓
┌────────────────────────────┐
│ Test Closure │
│ Are we finished? │
└────────────────────────────┘

---

## STLC Example — E-commerce Login Feature

### Phase 1: Requirement Analysis

**Requirement:** User should be able to log in using email and password.

QA identifies: valid login, invalid email, invalid password, empty fields, password masking, account lock, forgot password, session behavior.

### Phase 2: Test Planning

Functional Testing → Yes
UI Testing → Yes
Security Testing → Basic
Regression Testing → Yes
Automation → Yes
Browser Testing → Chrome, Firefox, Edge


### Phase 3: Test Case Development

TC01 → Valid login
TC02 → Invalid password
TC03 → Invalid email
TC04 → Empty email
TC05 → Empty password
TC06 → Both fields empty
TC07 → Account lock
TC08 → Forgot password


### Phase 4: Environment Setup

Application → Available
Database → Available
API → Available
Test User → Available
Browsers → Available
Test Data → Available


### Phase 5: Test Execution

TC01 → PASS
TC02 → PASS
TC03 → PASS
TC04 → PASS
TC05 → FAIL
TC06 → PASS


QA reports the defect for TC05. Developer fixes it. QA retests → **PASS**.

QA then performs regression testing to ensure the fix didn't break other functionality.

### Phase 6: Test Closure

Total Test Cases → 6
Passed → 6
Failed → 0
Critical Bugs → 0
High Bugs → 0

Testing → Completed


The team then evaluates whether the agreed exit criteria have been met.

---

## SDLC vs STLC

| SDLC | STLC |
|---|---|
| Software Development Life Cycle | Software Testing Life Cycle |
| Covers the complete software development process | Focuses specifically on testing |
| Includes development activities | Includes testing activities |
| Involves developers, QA, BA, PM, etc. | Primarily focuses on QA/testing activities |
| Requirement → Design → Development → Testing → Deployment → Maintenance | Requirement Analysis → Test Planning → Test Case Development → Environment Setup → Execution → Closure |
| Goal is to build and maintain software | Goal is to verify and validate software quality |

**Easy way to remember:**
- **SDLC** = How we build the software
- **STLC** = How we test the software

---

## STLC Entry Criteria and Exit Criteria

### Entry Criteria

Conditions that should be satisfied **before** a testing phase can begin.

**Examples:**
- Requirements are available
- Test cases are prepared
- Test environment is ready
- Test data is available
- Build is deployed
- Required tools and resources are available

**Example — Before starting system testing:**

Build deployed → Yes
Environment ready → Yes
Test data available → Yes
Test cases ready → Yes


If the required conditions are not satisfied, testing may be delayed or blocked.

### Exit Criteria

Conditions that should be satisfied before testing can be considered **complete**.

**Examples:**
- All planned test cases executed
- Required test coverage achieved
- Critical defects resolved
- High-priority defects resolved or accepted
- Regression testing completed
- Required pass percentage achieved
- Testing objectives achieved

**Example:**

Test Cases Executed → 100%
Critical Defects → 0 open
High Defects → 0 open
Regression Testing → Completed
Required Pass Rate → Achieved


Then the QA team can recommend that testing is complete.

> **Note:** Exit criteria are project-specific. There is no universal rule such as "100% of test cases must always pass."

---

## STLC Deliverables

| STLC Phase | Common Deliverables |
|---|---|
| Requirement Analysis | Requirement analysis, RTM, clarification questions |
| Test Planning | Test Plan, testing strategy, estimates |
| Test Case Development | Test cases, test scenarios, test data, automation scripts |
| Environment Setup | Test environment, environment readiness report |
| Test Execution | Test execution results, defect reports, test evidence |
| Test Closure | Test Closure Report, testing metrics, lessons learned |

---

## Important STLC Interview Questions

**1. What is STLC?**
STLC is a structured process that defines the testing activities performed by the QA team throughout the software testing lifecycle.

**2. What are the phases of STLC?**
Requirement Analysis, Test Planning, Test Case Development, Test Environment Setup, Test Execution, Test Closure.

**3. What is the first phase of STLC?**
Requirement Analysis.

**4. What happens during Test Planning?**
QA defines the testing scope, objectives, strategy, resources, schedule, tools, risks, and entry/exit criteria.

**5. What happens during Test Execution?**
QA executes test cases, compares actual results with expected results, reports defects, retests fixes, and performs regression testing.

**6. What is Test Closure?**
The final STLC phase where QA evaluates testing results, defects, coverage, risks, and lessons learned, and determines whether the testing objectives and exit criteria have been met.

**7. Can QA be involved before development starts?**
Yes. QA can participate during requirement analysis and test planning, review requirements, identify ambiguities and risks, and begin preparing test scenarios and test cases before development is completed.

**8. What is the difference between Entry Criteria and Exit Criteria?**
Entry Criteria define the conditions that must be satisfied before a testing phase can begin, while Exit Criteria define the conditions that must be satisfied before testing can be considered complete.

**9. Is STLC completely separate from SDLC?**
No. STLC is focused specifically on testing activities and operates within the broader SDLC.

**10. When does testing start in STLC?**
Testing activities can start from the Requirement Analysis phase. Actual execution of software tests happens when a testable build and suitable environment are available.

---

## QA Mindset

A good QA Engineer doesn't simply wait for developers to finish coding. QA should be involved early:

Requirement → Ask questions → Identify risks → Plan testing
→ Prepare test cases → Prepare environment → Execute tests
→ Report defects → Retest → Regression testing
→ Evaluate exit criteria → Test Closure


> The earlier a problem is identified, the cheaper and easier it usually is to fix.