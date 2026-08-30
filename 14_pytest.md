markdown
# pytest

pytest is the most widely used Python testing framework, and it's what turns the raw Python + `requests` scripts from Step 12 into structured, reportable, maintainable automated tests. This chapter covers how QA engineers actually use pytest in real projects.

---

# 1. What is pytest?

pytest is a Python testing framework that allows you to write simple, readable test functions and run them automatically, with built-in support for assertions, fixtures, parameterization, and rich reporting.

## Why pytest (Over Python's Built-in `unittest`)
Simpler syntax — no need for a class or self.assertEqual(), just plain assert
Powerful fixtures for setup/teardown and reusable test data
Easy parameterization for data-driven testing
Rich plugin ecosystem (pytest-html, pytest-xdist, pytest-rerunfailures, etc.)
Detailed, readable failure output
Auto-discovers test files and functions — minimal configuration needed

---

# 2. Installing pytest

```bash
pip install pytest
```

---

# 3. Writing Your First Test

pytest automatically discovers files and functions following its naming convention.

## Naming Conventions

Test files → must start with test_ or end with test.py
Test functions → must start with test
Test classes → must start with Test (and have no init method)


### Example — `test_login.py`

```python
def add(a, b):
    return a + b

def test_add_two_numbers():
    result = add(2, 3)
    assert result == 5

def test_add_negative_numbers():
    result = add(-2, -3)
    assert result == -5
```

### Running Tests

```bash
pytest                      # run all tests in the current directory
pytest test_login.py        # run a specific file
pytest test_login.py::test_add_two_numbers   # run a specific test
pytest -v                   # verbose output (shows each test name)
```

---

# 4. Writing API Tests with pytest (Building on Step 12)

This is where pytest and the `requests` library from the previous chapter come together into real, structured test suites.

### Example — `test_login_api.py`

```python
import requests

BASE_URL = "https://api.example.com/v1"

def test_login_with_valid_credentials():
    payload = {"email": "test@example.com", "password": "Test@1234"}
    response = requests.post(f"{BASE_URL}/login", json=payload)
    
    assert response.status_code == 200
    assert "token" in response.json()

def test_login_with_invalid_password():
    payload = {"email": "test@example.com", "password": "WrongPass"}
    response = requests.post(f"{BASE_URL}/login", json=payload)
    
    assert response.status_code == 401
    assert response.json()["message"] == "Invalid email or password"

def test_login_with_missing_email():
    payload = {"password": "Test@1234"}
    response = requests.post(f"{BASE_URL}/login", json=payload)
    
    assert response.status_code == 400
```

---

# 5. Assertions in pytest

pytest uses Python's plain `assert` keyword — no special assertion methods needed. pytest automatically provides detailed failure messages by inspecting the assertion.

```python
def test_examples():
    assert 2 + 2 == 4
    assert "hello".upper() == "HELLO"
    assert [1, 2, 3] == [1, 2, 3]
    assert "test" in "this is a test string"
    assert None is None
    assert not False
```

### Example Failure Output

```python
def test_status_code():
    status_code = 404
    assert status_code == 200
```
def test_status_code():
    status_code = 404
  assert status_code == 200

E assert 404 == 200

test_example.py:3: AssertionError


> pytest automatically shows you the actual vs expected values without needing to write a custom message — this is a major usability advantage over other frameworks.

### Custom Assertion Messages

```python
def test_status_code():
    status_code = 404
    assert status_code == 200, f"Expected 200 but got {status_code}"
```

---

# 6. Fixtures (Setup and Teardown)

Fixtures provide reusable setup logic (and optional cleanup) that can be injected into any test function that needs it — replacing repetitive setup code.

### Basic Fixture Example

```python
import pytest
import requests

@pytest.fixture
def base_url():
    return "https://api.example.com/v1"

def test_get_products(base_url):
    response = requests.get(f"{base_url}/products")
    assert response.status_code == 200
```

### Fixture with Setup and Teardown

```python
@pytest.fixture
def logged_in_token():
    # Setup: log in before the test runs
    response = requests.post("https://api.example.com/v1/login", json={
        "email": "test@example.com",
        "password": "Test@1234"
    })
    token = response.json()["token"]
    
    yield token   # this value is passed to the test
    
    # Teardown: runs after the test completes (cleanup)
    print("Test finished, logging out or cleaning up session")

def test_get_orders(logged_in_token):
    headers = {"Authorization": f"Bearer {logged_in_token}"}
    response = requests.get("https://api.example.com/v1/orders", headers=headers)
    assert response.status_code == 200
```

> Everything before `yield` runs as setup; everything after `yield` runs as teardown, even if the test fails — this ensures cleanup always happens.

### Fixture Scope

Controls how often a fixture is created/reused.

| Scope | Behavior |
|---|---|
| `function` (default) | Runs once per test function |
| `class` | Runs once per test class |
| `module` | Runs once per test file |
| `session` | Runs once for the entire test run |

```python
@pytest.fixture(scope="session")
def logged_in_token():
    # Login happens only ONCE for the entire test session,
    # instead of before every single test — much faster
    response = requests.post("https://api.example.com/v1/login", json={
        "email": "test@example.com",
        "password": "Test@1234"
    })
    return response.json()["token"]
```

> Using `session` scope for something like login avoids re-authenticating before every single test, significantly speeding up large test suites.

### conftest.py — Sharing Fixtures Across Files

Fixtures defined in a file named `conftest.py` are automatically available to all test files in that directory, without needing to import them manually.

project/
├── conftest.py ← shared fixtures live here
├── test_login.py
├── test_orders.py
└── test_cart.py


```python
# conftest.py
import pytest
import requests

@pytest.fixture
def api_token():
    response = requests.post("https://api.example.com/v1/login", json={
        "email": "test@example.com",
        "password": "Test@1234"
    })
    return response.json()["token"]
```

```python
# test_orders.py — no import needed, pytest finds it automatically
def test_get_orders(api_token):
    headers = {"Authorization": f"Bearer {api_token}"}
    response = requests.get("https://api.example.com/v1/orders", headers=headers)
    assert response.status_code == 200
```

---

# 7. Parameterization (Data-Driven Testing)

Instead of writing a separate test function for each input, `@pytest.mark.parametrize` lets you run the same test logic against multiple sets of data.

### Example — Testing Multiple Invalid Emails

```python
import pytest

def is_valid_email(email):
    return "@" in email and "." in email.split("@")[-1]

@pytest.mark.parametrize("email, expected", [
    ("test@example.com", True),
    ("plainaddress", False),
    ("@missingusername.com", False),
    ("test@example", False),
    ("", False),
])
def test_email_validation(email, expected):
    assert is_valid_email(email) == expected
```

**Running this produces 5 separate test results**, one per data set:

test_email_validation[test@example.com-True] PASSED
test_email_validation[plainaddress-False] PASSED
test_email_validation[@missingusername.com-False] PASSED
test_email_validation[test@example-False] PASSED
test_email_validation[-False] PASSED


### Practical QA Example — Login API Negative Tests

```python
import pytest
import requests

BASE_URL = "https://api.example.com/v1"

@pytest.mark.parametrize("email, password, expected_status", [
    ("test@example.com", "WrongPassword", 401),
    ("notregistered@example.com", "Test@1234", 401),
    ("", "Test@1234", 400),
    ("test@example.com", "", 400),
    ("", "", 400),
])
def test_login_negative_cases(email, password, expected_status):
    payload = {"email": email, "password": password}
    response = requests.post(f"{BASE_URL}/login", json=payload)
    assert response.status_code == expected_status
```

> This single test function replaces 5 nearly-duplicate test functions — a huge readability and maintenance win, and directly reflects the "test scenario → multiple test cases" concept from Step 5.

---

# 8. Markers (Organizing and Selecting Tests)

Markers let you tag tests, then run only specific groups when needed — very useful for separating smoke tests from full regression suites.

```python
import pytest

@pytest.mark.smoke
def test_homepage_loads():
    ...

@pytest.mark.regression
def test_full_checkout_flow():
    ...

@pytest.mark.slow
def test_large_data_export():
    ...
```

### Running Specific Marked Tests

```bash
pytest -m smoke          # run only smoke tests
pytest -m regression      # run only regression tests
pytest -m "not slow"      # run everything except slow tests
```

### Registering Custom Markers (in `pytest.ini`)

```ini
[pytest]
markers =
    smoke: quick tests to verify basic functionality
    regression: full regression test suite
    slow: tests that take a long time to run
```

> This directly supports the Smoke Testing and Regression Testing concepts from earlier chapters — now implemented as actual runnable test groups instead of just a testing strategy.

## Built-in Markers

```python
@pytest.mark.skip(reason="Feature not implemented yet")
def test_upcoming_feature():
    ...

@pytest.mark.skipif(condition, reason="Only runs on staging")
def test_staging_only_feature():
    ...

@pytest.mark.xfail(reason="Known bug, ticket JIRA-245")
def test_known_broken_feature():
    ...
```

> `xfail` is especially useful — it lets you keep a test for a **known** bug in the suite (marked as "expected to fail") without it breaking the build, while still tracking that the bug exists.

---

# 9. Test Classes (Grouping Related Tests)

While pytest doesn't require classes, grouping related tests into a class can improve organization for larger suites.

```python
class TestLoginAPI:
    
    def test_valid_login(self):
        response = requests.post(f"{BASE_URL}/login", json={
            "email": "test@example.com", "password": "Test@1234"
        })
        assert response.status_code == 200

    def test_invalid_password(self):
        response = requests.post(f"{BASE_URL}/login", json={
            "email": "test@example.com", "password": "wrong"
        })
        assert response.status_code == 401
```

```bash
pytest test_login_api.py::TestLoginAPI          # run entire class
pytest test_login_api.py::TestLoginAPI::test_valid_login  # run one method
```

---

# 10. Test Reports and Output

## Verbose and Summary Output

```bash
pytest -v                # verbose, shows each test name and result
pytest -q                # quiet, minimal output
pytest --tb=short         # shorter traceback on failures
pytest -x                 # stop after the first failure
pytest --lf               # rerun only tests that failed last time
```

## HTML Reports (via plugin)

```bash
pip install pytest-html
pytest --html=report.html
```

This generates a shareable HTML report — useful for showing test results to non-technical stakeholders (e.g., during Sprint Review).

## JUnit XML Reports (for CI/CD Integration)

```bash
pytest --junitxml=report.xml
```

> This format is what CI/CD tools (like GitHub Actions, Jenkins) read to display test results in their dashboards — directly setting up Step 17 (CI/CD).

---

# 11. Useful pytest Plugins for QA

| Plugin | Purpose |
|---|---|
| `pytest-html` | Generates HTML test reports |
| `pytest-xdist` | Runs tests in parallel (faster execution) |
| `pytest-rerunfailures` | Automatically retries failed tests (useful for flaky UI tests) |
| `pytest-cov` | Measures code coverage |
| `pytest-playwright` | Integrates Playwright browser automation with pytest |
| `pytest-mock` | Simplifies mocking dependencies in tests |

### Example — Running Tests in Parallel

```bash
pip install pytest-xdist
pytest -n 4          # runs tests using 4 parallel workers
```

### Example — Auto-Retry Flaky Tests

```bash
pip install pytest-rerunfailures
pytest --reruns 3 --reruns-delay 2   # retry failed tests up to 3 times, 2s apart
```

> Retry logic is commonly (and carefully) used for UI automation tests that can be flaky due to timing issues — but it should never be used to mask a **real** bug, only genuine environment/timing flakiness.

---

# 12. Complete Example — A Realistic Test File

```python
# test_orders_api.py
import pytest
import requests

BASE_URL = "https://api.example.com/v1"

@pytest.fixture(scope="module")
def auth_token():
    response = requests.post(f"{BASE_URL}/login", json={
        "email": "test@example.com",
        "password": "Test@1234"
    })
    assert response.status_code == 200
    return response.json()["token"]

@pytest.fixture
def auth_headers(auth_token):
    return {"Authorization": f"Bearer {auth_token}"}


@pytest.mark.smoke
class TestOrdersAPI:

    def test_get_all_orders(self, auth_headers):
        response = requests.get(f"{BASE_URL}/orders", headers=auth_headers)
        assert response.status_code == 200
        assert isinstance(response.json(), list)

    @pytest.mark.parametrize("order_id, expected_status", [
        (101, 200),
        (99999, 404),
        (-1, 400),
    ])
    def test_get_order_by_id(self, auth_headers, order_id, expected_status):
        response = requests.get(f"{BASE_URL}/orders/{order_id}", headers=auth_headers)
        assert response.status_code == expected_status

    def test_create_order_without_auth(self):
        response = requests.post(f"{BASE_URL}/orders", json={"items": []})
        assert response.status_code == 401
```

This example combines: fixtures (with scope), fixture chaining (`auth_headers` depends on `auth_token`), markers, parameterization, and clear assertions — the realistic building blocks of a professional pytest suite.

---

## Key Takeaways

- pytest uses plain `assert` statements and auto-discovers `test_*.py` files/functions — minimal boilerplate compared to other frameworks.
- **Fixtures** handle setup/teardown (like logging in before tests) and can be scoped (`function`, `module`, `session`) to control how often they run — critical for performance in large suites.
- **`conftest.py`** shares fixtures across multiple test files automatically.
- **Parameterization** (`@pytest.mark.parametrize`) turns repetitive negative/boundary test cases (from Step 4 & 5) into clean, data-driven single functions.
- **Markers** (`smoke`, `regression`, `skip`, `xfail`) let you tag and selectively run test subsets — directly implementing Smoke/Regression testing strategies as real, runnable commands.
- **Reports** (`pytest-html`, `--junitxml`) make results shareable with stakeholders and consumable by CI/CD tools.
- Plugins like `pytest-xdist` (parallel execution) and `pytest-rerunfailures` (retry flaky tests) are practical tools for scaling and stabilizing real automation suites.
- This chapter directly combines Python (Step 12) + API concepts (Step 10) into structured, professional, automated test suites — the foundation for Playwright (Step 14) and Framework Design (Step 15).