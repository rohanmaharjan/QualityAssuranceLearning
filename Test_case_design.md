# Test Case Design Techniques

Test Case Design Techniques are structured methods used to design effective test cases that provide maximum coverage with a minimum number of test cases.

> Without a technique, testers tend to either test too little (missing bugs) or test everything blindly (wasting time). These techniques give a **systematic way** to choose the right test inputs.

Test design techniques are generally divided into three categories:

Test Design Techniques
│
├── Black-Box Techniques (based on requirements/behavior)
│
├── White-Box Techniques (based on internal code structure)
│
└── Experience-Based Techniques (based on tester's intuition/experience)


---

# Part 1: Black-Box Techniques

These techniques design tests based on **what the system should do**, without looking at the internal code.

## 1. Equivalence Partitioning (EP)

Divides input data into groups (partitions) where the system is expected to behave the **same way** for any value in that group. Instead of testing every possible value, we test one representative from each partition.

### Example

Suppose age must be between 18 and 60 to register.

A beginner might test: 18, 19, 20, 21 ... 60 → unnecessary and repetitive.

Instead, divide into partitions:

| Partition | Range | Type |
|---|---|---|
| P1 | < 18 | Invalid |
| P2 | 18 – 60 | Valid |
| P3 | > 60 | Invalid |

We just need **one representative value** from each partition:

P1 → 10 (invalid)
P2 → 30 (valid)
P3 → 70 (invalid)


### Why it works
We **assume** that if one value in a partition works/fails correctly, other values in the same partition will behave the same way — this assumption is what reduces the number of test cases needed.

### Types of Equivalence Classes
- **Valid Equivalence Class:** Inputs that should be accepted.
- **Invalid Equivalence Class:** Inputs that should be rejected.

---

## 2. Boundary Value Analysis (BVA)

Focuses specifically on the **edges/boundaries** of input ranges, since defects commonly occur at boundaries due to off-by-one errors in code (e.g., `>` instead of `>=`).

### Example

Age range: 18 – 60

Important boundary values to test:

17 → just below minimum (invalid)
18 → exact minimum (valid)
19 → just above minimum (valid)
59 → just below maximum (valid)
60 → exact maximum (valid)
61 → just above maximum (invalid)


### Why boundaries matter

A common coding mistake:

if age > 18: # WRONG — excludes 18, should be >=
if age >= 18: # CORRECT


BVA is almost always used **together** with Equivalence Partitioning — EP tells you which regions to test, BVA tells you exactly which values within/around those regions to focus on.

### BVA Rule of Thumb
For a range **min to max**, test:

min - 1, min, min + 1, max - 1, max, max + 1


---

## 3. Decision Table Testing

Used when the system's behavior depends on **multiple conditions combined** (AND/OR logic). A decision table lists all possible combinations of conditions and their expected outcomes.

### Example

Login system rules:
- Email must be registered
- Password must be correct
- Account must be active

**Decision Table:**

| Rule | Registered Email? | Correct Password? | Account Active? | Expected Result |
|---|---|---|---|---|
| R1 | Yes | Yes | Yes | Login Success |
| R2 | Yes | Yes | No | Account Locked message |
| R3 | Yes | No | Yes | Invalid password message |
| R4 | Yes | No | No | Invalid password message |
| R5 | No | — | — | Invalid email message |

### When to use
- When requirements contain multiple conditions combined with AND/OR
- Discount rules ("if member AND order > $50, apply 10% off")
- Permission/access rules
- Complex authentication rules

### Why it's useful
Ensures **no combination of conditions is missed** — a common source of bugs is an untested combination of conditions (e.g., "correct password but inactive account").

---

## 4. State Transition Testing

Used when a system's behavior depends on its **current state**, and the same input can produce different outputs depending on that state. This technique tests transitions from one state to another based on events/actions.

### Example — Banking System (PIN Attempts)

Active --Withdraw--> Active
Active --Wrong PIN--> Active (1 fail)
Active (1 fail) --Wrong PIN--> Active (2 fails)
Active (2 fails) --Wrong PIN--> Locked


**Important edge case:**

Wrong → Wrong → Right → Wrong → Wrong

If the rule is "3 **consecutive** failed attempts locks the account," it should **not** lock here — because the successful login in the middle resets the failure count. Testing this exact sequence is a classic State Transition test case.

### State Transition Diagram (Simple Representation)
    Wrong PIN         Wrong PIN         Wrong PIN

[Active] ----------> [1 Fail] ----------> [2 Fails] ----------> [Locked]
↑ |
|__Correct PIN|


### What to test
- Valid state transitions (the "happy path" flow)
- Invalid transitions (trying an action not allowed in current state)
- Boundary transitions (the exact event that changes state, e.g., 3rd wrong attempt)
- Sequences of events, not just single actions

### Common Use Cases
- Login/account lock mechanisms
- Order status flows (Placed → Shipped → Delivered → Returned)
- Bug/defect lifecycle (New → Assigned → Fixed → Closed)
- Workflow-based systems (approvals, multi-step forms)

---

## 5. Use Case Testing

Designs test cases based on **how a real user interacts with the system to accomplish a goal**, covering the primary flow plus alternate and exceptional flows.

### Example

**Use Case:** User logs into the application.

**Basic (Primary) Flow:**

Open login page → Enter email → Enter password → Click login → System validates credentials → Dashboard


**Alternate Flow (incorrect credentials):**

Open login page → Enter email → Enter wrong password → Click login → Error message shown → User retries


**Exceptional Flow (system failure):**

Open login page → Enter valid credentials → Click login → Database unavailable → System shows error/timeout


### Why it's useful
It ensures testing isn't limited to just the "ideal" scenario — it forces testers to think about alternate paths and failure conditions that real users will actually encounter.

---

# Part 2: White-Box Techniques

These are used mainly by developers (and sometimes SDETs), based on the internal code structure. Included here briefly since QA engineers should understand the concept even if they don't always write these tests themselves.

## 1. Statement Coverage
Ensures every line/statement in the code is executed at least once during testing.

## 2. Branch/Decision Coverage
Ensures every possible branch (true/false path of an `if` condition, for example) is executed at least once.

### Example
```python
if age >= 18:
    allow_access()
else:
    deny_access()
```
Branch coverage requires testing **both** the `True` and `False` paths of this condition, not just one.

## 3. Path Coverage
Ensures every possible **path** through the code (every combination of branches) is tested — the most thorough but also most expensive coverage type.

## 4. Condition Coverage
Ensures each individual boolean sub-condition (in compound conditions like `if (A and B)`) is tested for both `True` and `False`.

---

# Part 3: Experience-Based Techniques

These rely on the tester's skill, intuition, and past experience rather than a formal, documented method.

## 1. Error Guessing

Uses experience and intuition to predict where defects are likely to occur — there's no formal step-by-step process.

You ask: **"Where would a developer probably make a mistake?"**

### Common areas to "guess" errors in a login form:

Empty fields
Leading/trailing spaces
Very long input
Special characters / SQL injection / XSS payloads
Wrong password
Case sensitivity issues
Copy/paste behavior
Rapid repeated clicks (double submission)
Browser back button after login
Refresh after login
Session expiry / timeout behavior


## 2. Exploratory Testing

Learning about the application, designing tests, and executing them **simultaneously** — rather than strictly following predefined test cases.

### Example Flow

Open website → Try normal login → Notice unusual error → Investigate error
→ Try different input → Discover another behavior → Investigate further


Exploratory testing is especially useful for:
- New/undocumented features
- Finding unexpected bugs that scripted tests would miss
- Time-boxed testing sessions ("test this feature for 30 minutes and report findings")

## 3. Ad-hoc Testing

Informal testing performed **without any predefined test cases or structured plan**. Very similar to error guessing, but even less structured — often just "let me try random things and see what breaks."

### Difference from Exploratory Testing
| Ad-hoc Testing | Exploratory Testing |
|---|---|
| Completely unstructured | Semi-structured (has a charter/goal) |
| No documentation of approach | Learning, designing, and executing happen together with some direction |
| Relies purely on random attempts | Relies on systematic investigation guided by findings |

---

# Choosing the Right Technique

| Situation | Best Technique |
|---|---|
| A range of valid/invalid inputs (e.g., age, quantity) | Equivalence Partitioning + Boundary Value Analysis |
| Multiple combined conditions (AND/OR business rules) | Decision Table Testing |
| Behavior depends on previous actions/current state | State Transition Testing |
| Testing a full user journey/workflow | Use Case Testing |
| Time-limited testing, need to find bugs fast | Exploratory Testing |
| No requirements/documentation available, just explore | Ad-hoc Testing |
| Need to predict "sneaky" bugs from experience | Error Guessing |
| Need to verify internal code logic is tested | White-box techniques (Statement/Branch/Path Coverage) |

---

## Real-World Combined Example

**Feature:** Discount coupon system
> Rule: "If user is a Premium member AND order total > $100, apply 20% discount. Coupon code must also be valid and not expired."

**Applying multiple techniques together:**

1. **Equivalence Partitioning** on order total:
   - Valid: > $100
   - Invalid: ≤ $100

2. **Boundary Value Analysis** on order total:
   - $99.99 (just below) → no discount
   - $100.00 (exact boundary) → check requirement carefully (is it > or >=?)
   - $100.01 (just above) → discount applies

3. **Decision Table** for combined conditions:

| Premium Member? | Order > $100? | Valid Coupon? | Not Expired? | Result |
|---|---|---|---|---|
| Yes | Yes | Yes | Yes | 20% discount applied |
| Yes | Yes | Yes | No | Discount rejected (expired) |
| Yes | No | Yes | Yes | No discount (order too low) |
| No | Yes | Yes | Yes | No discount (not premium) |
| Yes | Yes | No | — | Invalid coupon error |

4. **Error Guessing:**
   - What if the coupon has extra spaces?
   - What if the user applies the coupon twice?
   - What if order total changes after coupon is applied (item removed from cart)?

This shows how in real QA work, techniques are rarely used in isolation — they're **combined** to get thorough coverage.

---

## Key Takeaways

- **Equivalence Partitioning** reduces test cases by grouping inputs that should behave the same way.
- **Boundary Value Analysis** targets the edges of ranges, where bugs commonly hide.
- **Decision Table Testing** ensures every combination of multiple conditions is covered.
- **State Transition Testing** is essential when the same input can behave differently depending on the system's current state.
- **Use Case Testing** ensures real user workflows — including alternate and exceptional flows — are tested, not just the happy path.
- **White-box techniques** (statement, branch, path, condition coverage) verify the internal code structure is exercised by tests.
- **Error Guessing, Exploratory, and Ad-hoc Testing** rely on tester experience and are valuable for finding bugs that formal techniques miss.
- In real projects, techniques are almost always **combined** — e.g., EP + BVA + Decision Tables together — for the best coverage.