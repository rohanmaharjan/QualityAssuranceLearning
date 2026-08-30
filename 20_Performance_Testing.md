markdown
# Performance Testing

Performance Testing evaluates how a system behaves under various load conditions — speed, responsiveness, stability, and scalability. This chapter goes deeper than the brief mention in Step 3, covering concepts, metrics, test types, and practical tooling.

---

# 1. What is Performance Testing?

Performance Testing is a type of non-functional testing that determines how a system performs in terms of responsiveness and stability under a particular workload.

> It doesn't ask "does the feature work?" — it asks **"does the feature work well, fast enough, and reliably, under real-world conditions?"**

## Why Performance Testing Matters
A feature that works correctly but takes 15 seconds to load is still a failure from a user's perspective
Systems that work fine with 10 users may completely break with 10,000 users
Performance issues are often discovered too late — only after real users experience slowness or crashes
Poor performance directly impacts revenue, user retention, and brand reputation (e.g., abandoned checkouts)

---

# 2. Key Performance Metrics

| Metric | Meaning |
|---|---|
| **Response Time** | Time taken to receive a response after sending a request |
| **Throughput** | Number of requests/transactions the system can handle per unit of time |
| **Latency** | Delay before a transfer of data begins following an instruction |
| **Concurrent Users** | Number of users actively using the system at the same time |
| **Error Rate** | Percentage of requests that fail under load |
| **CPU/Memory Utilization** | How much server resource is consumed under load |
| **Throughput vs Load Curve** | How throughput changes as load increases (identifies breaking points) |

### Example — Interpreting Metrics

Scenario: Checkout API under increasing load

Users: 100 → Avg Response Time: 200ms → Error Rate: 0%
Users: 1,000 → Avg Response Time: 450ms → Error Rate: 0.1%
Users: 5,000 → Avg Response Time: 2,500ms → Error Rate: 3%
Users: 10,000 → Avg Response Time: 8,000ms → Error Rate: 22%

Observation: The system starts degrading significantly between 1,000-5,000 users,
and becomes largely unusable at 10,000 users — this identifies the system's
practical capacity limit.


---

# 3. Types of Performance Testing

## 1. Load Testing
Verifies system behavior under **expected/normal** load conditions.

Example: Can the system handle 5,000 concurrent users during typical business hours
without significant degradation?


## 2. Stress Testing
Pushes the system **beyond** its normal capacity to find the breaking point and observe how it fails.

Example: Gradually increase load from 5,000 to 50,000 users until the system fails.
At what point does it crash? Does it fail gracefully (clear error messages)
or catastrophically (data corruption, complete outage)?


## 3. Spike Testing
Tests how the system handles a **sudden, sharp increase** in load.

Example: A flash sale causes traffic to jump from 500 to 20,000 users
within 60 seconds. Does the system scale up fast enough, or does it crash
before auto-scaling can respond?


## 4. Soak Testing (Endurance Testing)
Tests system behavior under sustained load over an **extended period** (hours or days).

Example: Run the system at 70% capacity continuously for 48 hours.
Does memory usage grow over time (memory leak)? Does performance
gradually degrade, even without additional load?


## 5. Scalability Testing
Determines how well the system scales — whether adding more resources (servers, database capacity) proportionally improves performance.

Example: Does doubling the number of application servers roughly double
the throughput the system can handle? Or does it plateau due to a bottleneck
elsewhere (e.g., database connection limits)?


## 6. Volume Testing
Tests system behavior with a **large volume of data** (not necessarily many users).

Example: How does search performance behave when the product catalog
grows from 10,000 to 10 million items?


## 7. Capacity Testing
Determines the maximum load a system can handle while still meeting performance requirements (related to, but distinct from, stress testing — capacity testing finds the "comfortable maximum," stress testing finds the "breaking point").

---

# 4. Performance Testing Comparison Table

| Test Type | Purpose | Load Pattern |
|---|---|---|
| Load Testing | Normal expected usage | Steady, expected load |
| Stress Testing | Find breaking point | Load increased beyond capacity |
| Spike Testing | Sudden traffic surge | Sharp, sudden increase |
| Soak Testing | Long-term stability | Sustained moderate load over time |
| Scalability Testing | Resource scaling effectiveness | Load increased alongside added resources |
| Volume Testing | Large data handling | Large data volume, normal user load |

---

# 5. The Performance Testing Process
Identify the test environment (hardware, software, network configuration)
Identify performance acceptance criteria (e.g., "95% of requests under 500ms")
Plan and design tests (identify key user scenarios/workflows to simulate)
Configure the test environment
Implement test design (create scripts using a performance tool)
Run the tests
Analyze results, report, retest after fixes

## Example — Defining Performance Acceptance Criteria

Requirement: "The checkout API must respond within 1 second for 95% of requests,
under a load of 2,000 concurrent users, with an error rate below 1%."

This becomes the pass/fail benchmark for performance testing —
without criteria like this, "is it fast enough?" has no objective answer.


---

# 6. Common Performance Testing Tools

| Tool | Notes |
|---|---|
| **Apache JMeter** | Free, open-source, widely used, GUI + scriptable, supports many protocols |
| **k6** | Modern, developer-friendly, JavaScript-based scripting, great CI/CD integration |
| **Gatling** | Scala-based, high-performance, good reporting |
| **Locust** | Python-based, write test scenarios as Python code |
| **Apache Bench (ab)** | Simple command-line tool for quick basic load tests |
| **BlazeMeter** | Cloud-based, built on top of JMeter, good for large-scale distributed tests |

> Given your Python background from Step 12, **Locust** is often the easiest performance tool to pick up next — test scenarios are written in familiar Python syntax.

---

# 7. Example — Basic Load Test with Locust

```python
# locustfile.py
from locust import HttpUser, task, between

class WebsiteUser(HttpUser):
    wait_time = between(1, 3)   # simulates realistic pauses between actions

    @task(3)   # weight: this task runs 3x more often than weight-1 tasks
    def view_products(self):
        self.client.get("/api/products")

    @task(1)
    def login(self):
        self.client.post("/api/login", json={
            "email": "test@example.com",
            "password": "Test@1234"
        })

    @task(2)
    def add_to_cart(self):
        self.client.post("/api/cart", json={
            "product_id": 45,
            "quantity": 1
        })
```

```bash
# Run Locust (opens a web UI at localhost:8089 to configure and monitor the test)
locust -f locustfile.py --host=https://api.example.com

# Run headless with specific parameters (useful for CI/CD)
locust -f locustfile.py --host=https://api.example.com \
  --users 1000 --spawn-rate 50 --run-time 5m --headless
```

--users 1000 → simulate 1,000 concurrent users
--spawn-rate 50 → add 50 new users per second until reaching 1,000
--run-time 5m → run the test for 5 minutes
--headless → run without the web UI, output results to console/file


---

# 8. Example — Basic Load Test with JMeter (Conceptual Structure)

JMeter uses a GUI-based Test Plan structure (though it can also be scripted/run via CLI):

Test Plan
│
├── Thread Group (defines number of users, ramp-up time, loop count)
│ │
│ ├── HTTP Request (e.g., POST /api/login)
│ ├── HTTP Request (e.g., GET /api/products)
│ └── HTTP Request (e.g., POST /api/cart)
│
├── Listeners (View Results Tree, Summary Report, Graph Results)
└── Assertions (verify response codes, content)


### Key JMeter Concepts

Thread Group → Represents a group of virtual users
Number of Threads → Number of concurrent users to simulate
Ramp-Up Period → Time taken to start all threads (avoids all users hitting at exact same millisecond)
Loop Count → How many times each user repeats their actions


**Example:** 500 threads, 100-second ramp-up → JMeter starts 5 new users every second until reaching 500 total.

---

# 9. Analyzing Performance Test Results

### Key Things to Look For
Average, median, 90th/95th/99th percentile response times
(percentiles matter more than averages — an average can hide a small % of very slow requests)
Error rate trend as load increases
Throughput (requests per second) at different load levels
Resource utilization (CPU, memory, database connections) during the test
The exact point where response time starts degrading sharply (the "knee" of the curve)

### Why Percentiles Matter More Than Averages

Example: 100 requests, average response time = 300ms

But actual distribution:
95 requests → 100ms (fast)
5 requests → 4,000ms (very slow)

Average = 300ms (looks acceptable)
95th percentile = 4,000ms (reveals a real problem affecting 5% of users)

If the SLA promises "95% of requests under 1 second," this system FAILS
that requirement — even though the average looks fine.


---

# 10. Identifying Bottlenecks

When performance degrades, the cause could be in multiple layers — good performance testing involves narrowing down **where**.

Possible bottleneck locations:

Application code (inefficient algorithms, unnecessary processing)
Database (slow queries, missing indexes, connection pool exhaustion)
Network (bandwidth limits, latency between services)
Third-party services (slow external API calls)
Server resources (CPU, memory, disk I/O limits)
Caching (missing or ineffective caching layer)

### Practical Investigation Approach
Monitor response times broken down by API endpoint — which endpoint is slowest?
Check database query logs during the load test — are there slow queries? (connects to Step 11)
Monitor server CPU/memory during the test — is a resource maxed out?
Check if response time degrades linearly with load, or suddenly spikes at a threshold
(a sudden spike often indicates hitting a hard limit — e.g., a connection pool size)

---

# 11. Performance Testing in CI/CD

Performance tests are typically **not** run on every single commit (they're slower and resource-intensive) — more commonly run on a schedule or before major releases.

```yaml
name: Performance Test

on:
  schedule:
    - cron: '0 3 * * 0'   # weekly, Sunday at 3 AM

jobs:
  performance-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install Locust
        run: pip install locust
      - name: Run performance test
        run: |
          locust -f locustfile.py --host=https://staging.example.com \
            --users 1000 --spawn-rate 50 --run-time 10m --headless \
            --csv=results
      - name: Upload results
        uses: actions/upload-artifact@v4
        with:
          name: performance-results
          path: results*.csv
```

> This connects directly to Step 17 — automating performance tests to run periodically, catching performance regressions before they reach production, rather than relying on manual, occasional testing.

---

# 12. Performance Testing Checklist
Define clear, measurable performance acceptance criteria before testing
Test in an environment as close to production as possible (results from a tiny dev
environment don't reliably predict production behavior)
Test realistic user scenarios/workflows, not just single isolated endpoints
Include ramp-up periods to simulate realistic traffic patterns, not instant spikes
Monitor server-side metrics (CPU, memory, DB) alongside client-side response times
Look at percentiles (95th/99th), not just averages
Test with realistic data volumes (an empty test database gives misleadingly fast results)
Repeat tests to confirm consistency, not just a single run
Compare results against previous baselines to catch performance regressions over time

---

## Key Takeaways

- Performance Testing verifies **speed, stability, and scalability** — a functionally correct feature can still fail if it's too slow or unreliable under load.
- Key metrics: **response time, throughput, latency, concurrent users, error rate**, and resource utilization.
- Main test types: **Load** (expected traffic), **Stress** (find breaking point), **Spike** (sudden surge), **Soak** (long-term stability), **Scalability**, and **Volume** testing.
- Tools like **JMeter**, **k6**, **Gatling**, and **Locust** (Python-based, a natural fit given Step 12) let QA script and run these tests.
- **Percentiles (95th/99th)** reveal problems that averages hide — a small percentage of very slow requests can still violate SLAs even with a good average.
- Identifying **bottlenecks** requires looking across the whole stack: application code, database (Step 11), network, and server resources.
- Performance tests are usually run on a **schedule** or before releases in CI/CD (Step 17), rather than on every single commit, due to their resource cost.
- Always test against **clear acceptance criteria**, in a production-like environment, with realistic data volumes — vague "let's see if it's fast" testing doesn't produce actionable results.