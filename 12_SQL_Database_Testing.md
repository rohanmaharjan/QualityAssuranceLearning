markdown
# SQL & Database Testing

Database testing verifies the data layer of an application — ensuring data is stored, retrieved, updated, and deleted correctly, and that it stays consistent, accurate, and secure. This chapter covers the SQL fundamentals QA engineers need, plus how database testing fits into the overall testing process.

---

# 1. Why QA Needs to Know SQL
Verify data actually gets saved/updated/deleted correctly in the backend
(not just that the UI "looks" successful)
Set up test data directly, faster than going through the UI repeatedly
Verify data integrity, especially after bulk operations or migrations
Investigate bugs by checking the actual state of data
Validate that API/UI actions produce the correct database changes
Clean up test data after automated test runs

> A UI might show "Order placed successfully!" but if the order wasn't actually saved to the database correctly, that's a serious bug the UI alone won't reveal — SQL lets QA verify the ground truth.

---

# 2. Types of Databases (Brief Overview)

| Type | Examples | Structure |
|---|---|---|
| **Relational (SQL)** | MySQL, PostgreSQL, Oracle, SQL Server | Tables with rows/columns, relationships via keys |
| **Non-Relational (NoSQL)** | MongoDB, Cassandra, Redis | Documents, key-value pairs, or graphs — flexible schema |

> This chapter focuses on **SQL/relational databases**, as they remain the most common for QA testing fundamentals.

---

# 3. Basic Database Concepts

| Term | Meaning |
|---|---|
| **Database** | A collection of organized data |
| **Table** | A structured set of data organized in rows and columns |
| **Row (Record)** | A single entry in a table |
| **Column (Field)** | A specific attribute of data in a table |
| **Primary Key (PK)** | A unique identifier for each row in a table |
| **Foreign Key (FK)** | A column that references the primary key of another table (creates relationships) |
| **Schema** | The structure/blueprint of the database (tables, columns, relationships) |
| **Index** | A structure that speeds up data retrieval |

### Example — Simple Table Structure

**`users` table**

| id (PK) | email | password_hash | created_at |
|---|---|---|---|
| 1 | test@example.com | $2b$10$abc... | 2026-01-15 |
| 2 | john@example.com | $2b$10$xyz... | 2026-02-20 |

**`orders` table**

| id (PK) | user_id (FK) | total | status |
|---|---|---|---|
| 101 | 1 | 49.99 | Completed |
| 102 | 1 | 29.99 | Pending |

> Here, `user_id` in the `orders` table is a **foreign key** referencing `id` in the `users` table — this relationship lets us find all orders belonging to a specific user.

---

# 4. SQL Basics — CRUD Operations

**CRUD = Create, Read, Update, Delete** — the four basic operations performed on data.

## SELECT (Read)

```sql
-- Get all users
SELECT * FROM users;

-- Get specific columns
SELECT id, email FROM users;

-- Get a specific user
SELECT * FROM users WHERE id = 1;

-- Get users matching a condition
SELECT * FROM users WHERE email = 'test@example.com';
```

## INSERT (Create)

```sql
INSERT INTO users (email, password_hash, created_at)
VALUES ('newuser@example.com', '$2b$10$hash...', '2026-08-30');
```

## UPDATE

```sql
-- Update a specific user's email
UPDATE users
SET email = 'updated@example.com'
WHERE id = 1;
```

> ⚠️ **Always use a `WHERE` clause with UPDATE** — without it, ALL rows in the table get updated. This is a real mistake even experienced testers make when directly querying test databases.

## DELETE

```sql
-- Delete a specific user
DELETE FROM users WHERE id = 1;
```

> ⚠️ Same warning applies to DELETE — without `WHERE`, it deletes **every row** in the table.

---

# 5. Filtering and Conditions

```sql
-- Comparison operators
SELECT * FROM orders WHERE total > 50;
SELECT * FROM orders WHERE total >= 50;
SELECT * FROM orders WHERE status != 'Completed';

-- Multiple conditions
SELECT * FROM orders WHERE status = 'Pending' AND total > 20;
SELECT * FROM orders WHERE status = 'Pending' OR status = 'Completed';

-- Range
SELECT * FROM orders WHERE total BETWEEN 20 AND 100;

-- Matching a list
SELECT * FROM orders WHERE status IN ('Pending', 'Processing');

-- Pattern matching
SELECT * FROM users WHERE email LIKE '%@gmail.com';   -- ends with @gmail.com
SELECT * FROM users WHERE email LIKE 'test%';           -- starts with 'test'

-- Checking for NULL
SELECT * FROM users WHERE password_hash IS NULL;
```

---

# 6. Sorting and Limiting

```sql
-- Sort ascending (default)
SELECT * FROM orders ORDER BY total ASC;

-- Sort descending
SELECT * FROM orders ORDER BY total DESC;

-- Limit number of results
SELECT * FROM orders ORDER BY created_at DESC LIMIT 10;
```

---

# 7. Aggregate Functions

Used to perform calculations across multiple rows — very useful for verifying totals, counts, and averages match what the UI displays.

```sql
-- Count total number of users
SELECT COUNT(*) FROM users;

-- Total revenue from completed orders
SELECT SUM(total) FROM orders WHERE status = 'Completed';

-- Average order value
SELECT AVG(total) FROM orders;

-- Highest and lowest order value
SELECT MAX(total) FROM orders;
SELECT MIN(total) FROM orders;
```

### Grouping Data

```sql
-- Count orders per status
SELECT status, COUNT(*) as order_count
FROM orders
GROUP BY status;
```

**Example Result:**

| status | order_count |
|---|---|
| Completed | 145 |
| Pending | 23 |
| Cancelled | 8 |

### HAVING (Filtering Grouped Data)

```sql
-- Find users with more than 5 orders
SELECT user_id, COUNT(*) as order_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 5;
```

> **Note:** `WHERE` filters rows before grouping; `HAVING` filters groups after aggregation. This distinction is a common interview question.

---

# 8. JOINs (Combining Data from Multiple Tables)

Since related data is often split across multiple tables (like `users` and `orders`), JOINs let us combine them in a single query.

## INNER JOIN
Returns only rows that have matching data in both tables.

```sql
SELECT users.email, orders.total, orders.status
FROM users
INNER JOIN orders ON users.id = orders.user_id;
```

**Result:** Only users who have at least one order.

## LEFT JOIN
Returns all rows from the left table, plus matching rows from the right table (NULL where no match exists).

```sql
SELECT users.email, orders.total
FROM users
LEFT JOIN orders ON users.id = orders.user_id;
```

**Result:** All users, including those with zero orders (their order fields show NULL).

## Visual Difference

INNER JOIN: Only matching rows from BOTH tables
LEFT JOIN: ALL rows from left table + matches from right (NULL if no match)


### Testing Relevance

Scenario: "Show all users, including those who haven't placed any orders yet"
→ Requires LEFT JOIN. If the developer mistakenly used INNER JOIN,
users with zero orders would silently disappear from reports — a real bug
that's easy to catch by understanding JOIN types.


---

# 9. Database Testing — What QA Actually Verifies

## 1. Data Integrity
Verify data remains accurate and consistent across related tables.

```sql
-- Check for "orphaned" orders (referencing a user that doesn't exist)
SELECT * FROM orders
WHERE user_id NOT IN (SELECT id FROM users);
```

If this returns any rows, it's a data integrity issue — often caused by improper deletion (deleting a user without handling their related orders).

## 2. Data Validation (Business Rules)
Verify data conforms to expected rules.

```sql
-- Check for orders with negative totals (should never happen)
SELECT * FROM orders WHERE total < 0;

-- Check for duplicate emails (should be unique)
SELECT email, COUNT(*) 
FROM users 
GROUP BY email 
HAVING COUNT(*) > 1;
```

## 3. CRUD Verification (Backend Confirmation)
After performing an action via UI/API, confirm the database actually reflects the change.

Test: User updates their email via the UI settings page
Verify via SQL:
SELECT email FROM users WHERE id = 101;
→ Confirm the email in the database matches what was entered in the UI


## 4. Stored Procedures / Triggers Testing
If the application uses stored procedures or triggers (automatic actions on certain events), verify they execute correctly.

Example: A trigger that automatically updates a "last_modified" timestamp
whenever a row is updated — verify this timestamp actually changes.


## 5. Data Migration Testing
When data is moved between systems/versions, verify no data is lost, duplicated, or corrupted.
Compare record counts between old and new systems
Spot-check specific records for accuracy
Verify relationships (foreign keys) remain intact after migration

---

# 10. Transactions and ACID Properties

A **transaction** is a sequence of database operations treated as a single unit — either all operations succeed, or none do.

## ACID Properties

| Property | Meaning |
|---|---|
| **Atomicity** | All operations in a transaction succeed, or all fail (no partial updates) |
| **Consistency** | Database moves from one valid state to another, respecting all rules |
| **Isolation** | Concurrent transactions don't interfere with each other |
| **Durability** | Once committed, changes persist even after a system crash |

### Example — Why This Matters for Testing

Scenario: Money transfer between two bank accounts
Step 1: Deduct $100 from Account A
Step 2: Add $100 to Account B

If the system crashes between Step 1 and Step 2 without proper transaction handling:
→ Account A loses $100, but Account B never receives it — data corruption!

A properly implemented transaction ensures BOTH steps happen together, or NEITHER happens.


**Test scenario derived from this:**
Simulate a failure mid-transaction (if testable) and verify no partial update occurs
Verify account balances always sum correctly after any transfer operation
Test concurrent transfers on the same account (isolation) to check for race conditions

---

# 11. SQL Injection (Security Testing Overlap)

SQL Injection is a security vulnerability where malicious SQL code is inserted into input fields, potentially allowing attackers to manipulate or extract database data.

### Example Vulnerable Query (Backend Code)

```sql
-- Vulnerable: directly inserting user input into the query
SELECT * FROM users WHERE email = '" + userInput + "' AND password = '" + passwordInput + "'
```

If a user enters this into the email field:

' OR '1'='1


The query becomes:
```sql
SELECT * FROM users WHERE email = '' OR '1'='1' AND password = '...'
```

Since `'1'='1'` is always true, this could bypass authentication entirely.

### As a Tester, What to Check
Try injection payloads in every input field (login, search, forms)
Verify the application uses parameterized queries / prepared statements (ask developers)
Verify error messages don't reveal database structure/details
Confirm special characters are properly escaped/sanitized

> This connects directly to Step 20 (Security Testing Basics) — SQL injection is one of the most common and dangerous web vulnerabilities (OWASP Top 10).

---

# 12. Database Testing in API/UI Test Automation

In practice, database checks are often built directly into automated test scripts to confirm backend state after an action.

### Example — Python + SQL in an Automated Test (Preview for Step 12/13)

```python
# After calling the "create order" API
response = create_order(user_id=101, items=[...])
assert response.status_code == 201

# Verify the order actually exists in the database
cursor.execute("SELECT * FROM orders WHERE user_id = %s ORDER BY id DESC LIMIT 1", (101,))
order = cursor.fetchone()
assert order is not None
assert order['status'] == 'Pending'
```

This pattern — **API/UI action → SQL verification** — is extremely common in real QA automation and gives much stronger confidence than checking the UI/API response alone.

---

# 13. Common Database Testing Scenarios (Checklist)
Verify data saved correctly after Create operations
Verify data updates correctly after Update operations (and unchanged fields stay unchanged)
Verify data is actually removed after Delete operations (or soft-deleted, if applicable)
Verify unique constraints are enforced (e.g., duplicate email rejected)
Verify foreign key constraints prevent orphaned records
Verify default values are applied correctly when fields are omitted
Verify NULL handling behaves as expected
Verify data types match schema (e.g., price stored as decimal, not string)
Verify cascading deletes/updates work as intended (or don't, if not intended)
Verify large data volumes don't cause performance issues (basic check)
Verify transactions roll back properly on failure

---

## Key Takeaways

- QA uses SQL to verify the **actual backend state** of data, not just what the UI/API claims happened.
- Core SQL operations: **SELECT, INSERT, UPDATE, DELETE** — always use `WHERE` carefully with UPDATE/DELETE.
- **Aggregate functions** (`COUNT`, `SUM`, `AVG`, `GROUP BY`, `HAVING`) help verify totals and summaries match UI displays.
- **JOINs** (especially INNER vs LEFT) matter for testing — using the wrong join type can silently hide or duplicate data.
- Database testing covers **data integrity, validation, CRUD verification, migration testing**, and more.
- **ACID properties** (Atomicity, Consistency, Isolation, Durability) explain why transactions matter, especially for operations like financial transfers.
- **SQL Injection** is a critical security testing concern — testers should try injection payloads in every input field and verify proper sanitization.
- In real automation, the pattern **"perform action → verify via SQL"** gives stronger test confidence than checking the UI/API response alone — this connects directly into Python (Step 12) and pytest (Step 13) automation work ahead.