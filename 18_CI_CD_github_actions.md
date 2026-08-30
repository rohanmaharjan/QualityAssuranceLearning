markdown
# CI/CD & GitHub Actions

This chapter covers how automated tests (built in Steps 12–16) get integrated into a CI/CD pipeline — running automatically on every code change, rather than manually on someone's laptop.

---

# 1. What is CI/CD?

**CI/CD** stands for **Continuous Integration / Continuous Delivery (or Deployment)** — practices that automate building, testing, and releasing software.

## Continuous Integration (CI)
Developers frequently merge code changes into a shared repository, and each change is automatically **built and tested**.

Developer pushes code → Automated build → Automated tests run → Pass/Fail reported


## Continuous Delivery (CD)
Code that passes all tests is automatically prepared for release (but a human still approves the final deployment).

## Continuous Deployment (also CD)
Code that passes all tests is automatically deployed to production — no manual approval step.

Continuous Integration → Automatically test every change
Continuous Delivery → Automatically prepare a release, human approves deploy
Continuous Deployment → Automatically deploy, no human approval needed


## Why This Matters for QA
Tests run automatically on every push/PR — no one forgets to run them
Bugs are caught within minutes of being introduced, not days later
Broken builds are visible to the whole team immediately
Enables frequent, confident releases (a core Agile principle from Step 7)
Removes "it works on my machine" problems — tests run in a consistent environment

---

# 2. The CI/CD Pipeline (General Flow)

Code Pushed / PR Opened
↓
Build (install dependencies, compile if needed)
↓
Static Analysis / Linting (optional)
↓
Unit Tests
↓
Integration/API Tests
↓
UI/E2E Tests (Playwright)
↓
Deploy to Staging (if all tests pass)
↓
Smoke Tests on Staging
↓
Deploy to Production (manual approval or automatic)


> Notice this mirrors the **Testing Pyramid** (Step 3/15) — fast tests (unit, API) run early and often; slower tests (full UI/E2E) run later in the pipeline, since they take longer and are more expensive to run on every single commit.

---

# 3. What is GitHub Actions?

GitHub Actions is GitHub's built-in CI/CD tool, letting you define automated workflows directly in your repository using YAML files.

## Key Concepts

| Term | Meaning |
|---|---|
| **Workflow** | An automated process defined in a YAML file (e.g., "run tests on every PR") |
| **Event/Trigger** | What causes the workflow to run (push, pull request, schedule, manual) |
| **Job** | A set of steps that run on the same machine (runner) |
| **Step** | An individual task within a job (e.g., "install dependencies", "run tests") |
| **Runner** | The virtual machine that executes the workflow (GitHub-hosted or self-hosted) |
| **Action** | A reusable, pre-built unit of code (e.g., `actions/checkout` to pull the repo) |

---

# 4. Basic GitHub Actions Workflow File

Workflows live in `.github/workflows/*.yml` in your repository.

### Example — `.github/workflows/api-tests.yml`

```yaml
name: API Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  run-api-tests:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          pip install -r requirements.txt

      - name: Run API tests
        run: |
          pytest tests/api -m smoke --junitxml=reports/junit.xml

      - name: Upload test report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-report
          path: reports/junit.xml
```

### What Each Part Does

on: → defines the trigger (push or PR to main branch)
jobs.run-api-tests: → the job name
runs-on: → the OS/environment (ubuntu-latest is common)
steps: → sequential tasks executed in order
actions/checkout@v4 → pulls the repository code into the runner
actions/setup-python@v5 → installs the specified Python version
run: → executes shell commands directly
if: always() → uploads the report even if tests failed (important!)


---

# 5. Adding UI Tests (Playwright) to the Pipeline

```yaml
name: UI Tests

on:
  pull_request:
    branches: [main]

jobs:
  run-ui-tests:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          playwright install --with-deps

      - name: Run UI tests
        run: |
          pytest tests/ui -m smoke --browser chromium --headed=false \
            --screenshot=only-on-failure --video=retain-on-failure

      - name: Upload Playwright report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/
```

> `--headed=false` (headless mode) is essential for CI — there's no display available on the runner to show an actual browser window. `if: always()` on the artifact upload ensures screenshots/videos are saved **even when tests fail** — which is exactly when you need them most.

---

# 6. Running Different Test Suites at Different Triggers

Not every test needs to run on every single push — this connects directly to the Smoke/Regression/Markers concepts from Step 13.

```yaml
name: Test Suite

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 2 * * *'   # runs daily at 2 AM

jobs:
  smoke-tests:
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pytest -m smoke

  full-regression:
    if: github.event_name == 'schedule'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pytest -m regression
```

Every PR → runs quick smoke tests (fast feedback)
Every night (scheduled) → runs the full regression suite (thorough, slower)


> This is a common real-world pattern: fast feedback on every change, thorough coverage on a schedule — balancing speed against comprehensiveness.

---

# 7. Environment Variables and Secrets

Sensitive data (API keys, passwords, tokens) should never be hardcoded in workflow files — GitHub provides **Secrets** for this, directly addressing the "never commit credentials" principle from Step 16.

### Setting a Secret (via GitHub UI)

Repository → Settings → Secrets and variables → Actions → New repository secret
Example: TEST_USER_PASSWORD = "SomeSecurePassword123"


### Using Secrets in a Workflow

```yaml
steps:
  - name: Run tests
    env:
      TEST_USER_EMAIL: ${{ secrets.TEST_USER_EMAIL }}
      TEST_USER_PASSWORD: ${{ secrets.TEST_USER_PASSWORD }}
      BASE_URL: ${{ secrets.STAGING_BASE_URL }}
    run: |
      pytest tests/api -m smoke
```

```python
# In the test code
import os
password = os.getenv("TEST_USER_PASSWORD")
```

---

# 8. Matrix Builds (Testing Multiple Configurations)

Runs the same job multiple times with different configurations — useful for cross-browser or cross-version testing (connects to Compatibility Testing, Step 3).

```yaml
jobs:
  cross-browser-tests:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        browser: [chromium, firefox, webkit]

    steps:
      - uses: actions/checkout@v4
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - run: |
          pip install -r requirements.txt
          playwright install --with-deps
      - name: Run tests on ${{ matrix.browser }}
        run: pytest tests/ui --browser ${{ matrix.browser }}
```

> This automatically runs the entire UI test suite **three times in parallel** — once per browser — giving cross-browser compatibility results without writing three separate workflows.

---

# 9. Parallel Execution (Speeding Up the Pipeline)

Combining `pytest-xdist` (from Step 13) with CI:

```yaml
- name: Run tests in parallel
  run: pytest tests/api -n 4
```

Or splitting jobs entirely:

```yaml
jobs:
  api-tests:
    runs-on: ubuntu-latest
    steps:
      - run: pytest tests/api

  ui-tests:
    runs-on: ubuntu-latest
    steps:
      - run: pytest tests/ui
```

> Since `api-tests` and `ui-tests` don't depend on each other, GitHub Actions runs them **simultaneously** by default (unless you explicitly link them with `needs:`), reducing total pipeline time.

---

# 10. Job Dependencies (`needs`)

Sometimes one job must complete before another starts (e.g., deploy only after tests pass).

```yaml
jobs:
  run-tests:
    runs-on: ubuntu-latest
    steps:
      - run: pytest tests/

  deploy-to-staging:
    needs: run-tests          # waits for run-tests to succeed first
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying to staging..."
```

If run-tests FAILS → deploy-to-staging is automatically skipped


This is exactly how CI/CD prevents broken code from being deployed — a fundamental quality gate.

---

# 11. Status Checks and Branch Protection

Teams typically configure GitHub so that a Pull Request **cannot be merged** unless the CI workflow passes — enforcing quality without relying on manual discipline.

Repository → Settings → Branches → Branch protection rules → main
✔ Require status checks to pass before merging
✔ Select: "API Tests", "UI Tests" as required checks


> This means even if a reviewer approves a PR, GitHub will **block the merge** if the automated tests are failing — a strong, enforced quality gate that connects directly back to the Definition of Done concept from Step 7.

---

# 12. Notifications on Failure

```yaml
- name: Notify on failure
  if: failure()
  run: |
    curl -X POST -H 'Content-type: application/json' \
    --data '{"text":"❌ API tests failed on main branch!"}' \
    ${{ secrets.SLACK_WEBHOOK_URL }}
```

> Many teams integrate Slack/Teams notifications so the team is alerted immediately when the pipeline breaks, rather than discovering it hours later.

---

# 13. Test Reports in CI/CD

Connecting back to Step 13's reporting formats:

```yaml
- name: Run tests
  run: pytest --junitxml=reports/junit.xml --html=reports/report.html

- name: Publish Test Results
  uses: dorny/test-reporter@v1
  if: always()
  with:
    name: Test Results
    path: reports/junit.xml
    reporter: java-junit
```

This displays a readable pass/fail summary **directly in the GitHub PR interface**, so reviewers can see test results without downloading and opening report files manually.

---

# 14. Complete Example — Realistic Multi-Stage Pipeline

```yaml
name: Full CI Pipeline

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  api-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - run: pip install -r requirements.txt
      - name: Run API tests
        env:
          BASE_URL: ${{ secrets.STAGING_BASE_URL }}
          TEST_USER_PASSWORD: ${{ secrets.TEST_USER_PASSWORD }}
        run: pytest tests/api -m smoke --junitxml=reports/api-junit.xml
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: api-report
          path: reports/api-junit.xml

  ui-tests:
    runs-on: ubuntu-latest
    needs: api-tests
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - run: |
          pip install -r requirements.txt
          playwright install --with-deps
      - name: Run UI tests
        run: pytest tests/ui -m smoke --screenshot=only-on-failure
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: ui-report
          path: playwright-report/

  notify-on-failure:
    runs-on: ubuntu-latest
    needs: [api-tests, ui-tests]
    if: failure()
    steps:
      - name: Send Slack alert
        run: |
          curl -X POST -H 'Content-type: application/json' \
          --data '{"text":"❌ CI pipeline failed on ${{ github.ref }}"}' \
          ${{ secrets.SLACK_WEBHOOK_URL }}
```

---

## Key Takeaways

- **CI** automatically builds and tests every code change; **CD** automatically prepares (Delivery) or deploys (Deployment) code that passes those tests.
- **GitHub Actions** defines automated workflows in YAML files under `.github/workflows/`, triggered by events like `push` or `pull_request`.
- Pipelines typically run **fast tests first** (unit, API/smoke) and **slower tests later** (full UI/regression) — mirroring the Testing Pyramid.
- **Secrets** keep credentials out of code, directly enforcing the security practices from Step 16.
- **Matrix builds** run the same suite across multiple configurations (browsers, versions) in parallel — automating Compatibility Testing.
- **`needs:`** creates job dependencies, so deployment only happens if tests pass — a core CI/CD quality gate.
- **Branch protection rules** can require CI checks to pass before a PR can be merged, enforcing Definition of Done automatically rather than relying on manual discipline.
- Reports (JUnit XML, HTML, screenshots/videos) should always upload even on failure (`if: always()`), since failure evidence is exactly what's needed to debug issues.
- Notifications (Slack/Teams) on pipeline failure keep the whole team aware immediately, supporting the fast-feedback goal of both Agile (Step 7) and CI/CD.