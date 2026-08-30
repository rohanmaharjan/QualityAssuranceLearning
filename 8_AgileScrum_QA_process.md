markdown
# Agile, Scrum & QA Process

This chapter covers how QA fits into Agile development, with a specific focus on Scrum — the most widely used Agile framework in the industry.

---

# 1. What is Agile?

Agile is a software development approach based on **iterative, incremental delivery**, close collaboration, and adapting to change rather than following a fixed plan.

## Agile Manifesto — Core Values

Individuals and interactions over Processes and tools
Working software over Comprehensive documentation
Customer collaboration over Contract negotiation
Responding to change over Following a plan


> This doesn't mean the items on the right (processes, documentation, contracts, plans) don't matter — it means the items on the left are valued **more** when priorities conflict.

## 12 Agile Principles (Key Ones for QA)

- Deliver working software frequently (weeks, not months)
- Welcome changing requirements, even late in development
- Business people and developers must work together daily
- Working software is the primary measure of progress
- Continuous attention to technical excellence and good design

---

# 2. What is Scrum?

Scrum is a specific **framework** for implementing Agile. It organizes work into fixed-length iterations called **Sprints**, with defined roles, events, and artifacts.

## Scrum Roles

| Role | Responsibility |
|---|---|
| **Product Owner (PO)** | Owns the product backlog, defines priorities, represents business/customer needs |
| **Scrum Master** | Facilitates the process, removes blockers, ensures the team follows Scrum practices |
| **Development Team** | Cross-functional team (developers, QA, designers) who build the product |

> In modern Scrum, QA engineers are usually part of the **Development Team**, not a separate silo — testing happens continuously alongside development, not after it.

## Scrum Artifacts

| Artifact | Description |
|---|---|
| **Product Backlog** | Complete list of features, enhancements, and fixes needed for the product |
| **Sprint Backlog** | Subset of product backlog items selected for the current sprint |
| **Increment** | The working, potentially shippable product at the end of a sprint |

## Scrum Events (Ceremonies)

Sprint Planning
↓
Daily Standup (repeated daily during sprint)
↓
Sprint Review
↓
Sprint Retrospective
↓
(Next Sprint begins)


### Sprint Planning
The team decides what backlog items will be worked on in the upcoming sprint, and breaks them into tasks.

**QA's role:** Understand upcoming stories, ask clarifying questions, estimate testing effort, flag potential risks early.

### Daily Standup (Daily Scrum)
A short (usually 15-minute) daily meeting where each team member answers:
What did I do yesterday?
What will I do today?
Are there any blockers?

**QA's role:** Report testing progress, mention blocked test cases (e.g., waiting on a build fix), flag defects found.

### Sprint Review
At the end of the sprint, the team demonstrates completed work to stakeholders and gets feedback.

**QA's role:** Confirm tested features are demo-ready; sometimes QA helps demonstrate tested functionality.

### Sprint Retrospective
The team reflects on the sprint itself — what went well, what didn't, and what to improve.

**QA's role:** Share testing-related issues (e.g., "builds were delivered too late for proper regression testing") and suggest process improvements.

---

# 3. What is a Sprint?

A **Sprint** is a fixed time-box (commonly 1–4 weeks, most often 2 weeks) during which a specific set of backlog items is developed, tested, and made potentially shippable.

### Typical 2-Week Sprint Flow

Day 1 → Sprint Planning
Day 2-8 → Development + Testing happen in parallel
Day 9 → Regression Testing, Bug fixing
Day 10 → Sprint Review + Retrospective


> Unlike Waterfall, testing does **not** wait until development is "finished" — QA tests each story as it becomes ready within the same sprint.

---

# 4. User Stories

A **User Story** is a short, simple description of a feature written from the perspective of the end user.

### Standard Format

As a [type of user]
I want to [perform some action]
So that [I achieve some benefit/goal]


### Example

As a registered user
I want to reset my password
So that I can regain access to my account if I forget it


## INVEST Criteria (Qualities of a Good User Story)

| Letter | Meaning |
|---|---|
| **I** | Independent — can be developed/tested without depending on other stories |
| **N** | Negotiable — details can be discussed, not a rigid contract |
| **V** | Valuable — delivers real value to the user/business |
| **E** | Estimable — team can estimate the effort required |
| **S** | Small — small enough to complete within a sprint |
| **T** | Testable — clear enough that test cases can be written from it |

> QA plays an important role in ensuring stories are **testable** — if a story is too vague to derive test cases from, QA should flag it during backlog refinement.

---

# 5. Acceptance Criteria

Acceptance Criteria define the specific conditions that must be met for a user story to be considered "done." They form the foundation for writing test cases.

### Example (continuing the password reset story)
User can request a reset link using their registered email
Reset link is valid for 30 minutes
Reset link can only be used once
New password must meet complexity requirements (min 8 chars, 1 number, 1 special char)
User receives a confirmation after successful reset

### Given-When-Then Format (Behavior-Driven Development style)

A common way to write acceptance criteria, especially when following BDD practices:

Given [initial context/state]
When [an action is performed]
Then [expected outcome]


**Example:**

Given a registered user is on the login page
When they enter valid credentials and click "Login"
Then they should be redirected to the dashboard

Given a user requests a password reset
When the reset link has expired
Then the system should show an "expired link" error message


This format naturally maps to test case structure (precondition → action → expected result).

---

# 6. Definition of Ready (DoR) vs Definition of Done (DoD)

## Definition of Ready (DoR)
Criteria a user story must meet **before** it can be pulled into a sprint.

**Example checklist:**
Story is written in proper format
Acceptance criteria are defined
Dependencies are identified
Story is estimated by the team
Design/mockups are available (if needed)

## Definition of Done (DoD)
Criteria that must be met **before** a story is considered complete.

**Example checklist:**
Code is developed and reviewed
Unit tests written and passing
QA has tested the feature (functional + regression)
No critical/high defects remain open
Acceptance criteria are met
Documentation updated (if applicable)
Feature is deployed to staging/demo environment

> **QA's role:** QA is often the gatekeeper for DoD — a story shouldn't be marked "Done" until QA has verified it meets the acceptance criteria.

---

# 7. QA Activities Throughout a Sprint

Backlog Refinement
↓
Sprint Planning
↓
Story Development Begins
↓
QA Writes Test Cases (in parallel with development)
↓
Build Ready for Testing
↓
QA Executes Test Cases
↓
Defects Found → Reported → Fixed → Retested
↓
Regression Testing
↓
Story Meets Definition of Done
↓
Sprint Review Demo
↓
Sprint Retrospective


## Backlog Refinement (Backlog Grooming)

An ongoing activity (often a separate meeting) where the team reviews upcoming backlog items, clarifies requirements, and estimates effort — **before** they're planned into a sprint.

**QA's role:** Ask clarifying questions early ("What should happen if the reset link is used twice?"), identify missing acceptance criteria, flag testability concerns.

---

# 8. Types of Testing Performed Within a Sprint

| Testing Type | When Performed |
|---|---|
| **Functional Testing** | As each story is developed |
| **Regression Testing** | Near the end of sprint, or automated continuously |
| **Exploratory Testing** | Throughout, especially for new features |
| **Smoke Testing** | On every new build deployed to test environment |
| **API Testing** | As backend features become available |
| **Automation** | Ongoing — new automated tests added for completed stories |

> Because sprints are short, **automation becomes critical** — manually regression-testing the entire application every 2 weeks isn't sustainable as the product grows. This is why later steps in your roadmap (Playwright, CI/CD) build directly on this need.

---

# 9. Common Agile/Scrum Terms

| Term | Meaning |
|---|---|
| **Epic** | A large body of work that can be broken down into multiple user stories |
| **Story Point** | A relative unit of estimation representing effort/complexity (not hours) |
| **Velocity** | Average number of story points a team completes per sprint |
| **Burndown Chart** | Visual tracking of remaining work vs time left in the sprint |
| **Spike** | A time-boxed research task to investigate something before committing to work |
| **Technical Debt** | Shortcuts taken during development that will need to be fixed later |
| **Backlog Grooming/Refinement** | Ongoing process of reviewing and clarifying backlog items |

### Example — Epic Breakdown

Epic: User Account Management
│
├── User Story: User can register with email
├── User Story: User can log in
├── User Story: User can reset password
└── User Story: User can update profile information


---

# 10. Agile Testing Quadrants (Brian Marick's Model)

A common framework describing the different types of tests needed in Agile, organized into four quadrants.
            Business-Facing
                   │
Q2: Functional Tests    │    Q3: Exploratory Testing,
Story Tests               │    Usability Testing,
Prototypes                │    UAT
(Support the team)         │    (Critique the product)

────────────────────────────────────────────────────
Q1: Unit Tests │ Q4: Performance Tests,
Component Tests │ Security Tests,
(Support the team) │ "-ility" Tests
│ (Critique the product)
Technology-Facing


- **Q1 (Technology-facing, supports team):** Unit tests, component tests — usually automated, written by developers.
- **Q2 (Business-facing, supports team):** Functional tests, story tests — verify the feature meets acceptance criteria.
- **Q3 (Business-facing, critiques product):** Exploratory testing, usability testing, UAT — manual, human-judgment-based.
- **Q4 (Technology-facing, critiques product):** Performance, security, and other non-functional "-ility" tests.

> This model helps QA teams think about **balancing** different types of testing across a sprint rather than over-focusing on just functional testing.

---

# 11. Challenges of QA in Agile
Short sprint cycles leave limited time for thorough testing
Requirements/stories can be incomplete or change mid-sprint
Regression testing burden grows as the product grows
Automation needs continuous investment and maintenance
Testing environments may not always be ready when a story is "code complete"
Balancing speed of delivery with quality

### How QA Adapts
Get involved early (during backlog refinement, not after development)
Write test cases from acceptance criteria as soon as a story is ready
Prioritize automation for regression-heavy areas
Use risk-based testing to focus effort where it matters most
Communicate continuously with developers rather than waiting for a "handoff"

---

## Key Takeaways

- **Agile** values working software, collaboration, and responding to change over rigid processes and plans.
- **Scrum** is a framework for Agile with defined roles (PO, Scrum Master, Dev Team), artifacts (Product Backlog, Sprint Backlog, Increment), and events (Sprint Planning, Daily Standup, Sprint Review, Retrospective).
- QA is embedded in the **Development Team** and tests continuously within each sprint, not after all development is "done."
- **User Stories** follow "As a / I want / So that" format and should meet **INVEST** criteria — especially **Testable**.
- **Acceptance Criteria** define what "done" means for a story and form the direct basis for test cases; **Given-When-Then** is a common format.
- **Definition of Ready** gates what enters a sprint; **Definition of Done** gates what's considered complete — QA is usually the gatekeeper for DoD.
- Short sprint cycles make **automation** essential for sustainable regression testing as the product grows.
- The **Agile Testing Quadrants** model helps balance functional, exploratory, unit, and non-functional testing across a sprint.
- QA in Agile succeeds by engaging early (backlog refinement), communicating continuously, and prioritizing based on risk.