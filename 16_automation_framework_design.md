# Automation Framework Design

This chapter ties together everything from Python (Step 12), pytest (Step 13), and Playwright (Step 14) into a properly structured, maintainable automation framework — the kind used in real QA/SDET jobs, not just standalone test scripts.

---

# 1. What is a Test Automation Framework?

A framework is a structured set of guidelines, tools, and reusable components that organizes how automated tests are written, executed, and maintained — rather than a collection of scattered, one-off scripts.

## Why Not Just Write Standalone Scripts?

Without a framework:

Locators duplicated across dozens of test files
No consistent way to manage test data or environments
Hard to know what's already been automated
Difficult for new team members to understand or contribute
Small UI changes break many unrelated tests
No standard reporting or CI/CD integration

With a framework:

Centralized, reusable components (Page Objects, API clients, utilities)
Consistent structure that scales as the test suite grows
Easy onboarding for new QA engineers
Clean separation between test logic and implementation details

---

# 2. Types of Automation Frameworks

| Framework Type | Description |
|---|---|
| **Linear (Record & Playback)** | Simple recorded scripts, no reusability — rarely used in real projects |
| **Modular** | Application broken into independent modules, each with its own reusable functions |
| **Data-Driven** | Test logic separated from test data (data comes from CSV/JSON/Excel) |
| **Keyword-Driven** | Test steps represented as keywords (e.g., "Click", "Type") that map to functions — often used by non-coders |
| **Page Object Model (POM)** | UI elements/actions represented as page classes (covered in Step 14) |
| **Hybrid** | Combines multiple approaches (e.g., POM + Data-Driven + Keyword-Driven) |
| **BDD (Behavior-Driven Development)** | Tests written in plain language (Gherkin) that map to automation code |

> Most modern real-world frameworks are **Hybrid** — typically POM + Data-Driven, sometimes with BDD layered on top.

---

# 3. Recommended Project Structure

A well-organized framework typically looks like this:

project/
│
├── tests/
│ ├── api/
│ │ ├── test_login_api.py
│ │ ├── test_orders_api.py
│ │ └── test_products_api.py
│ │
│ └── ui/
│ ├── test_login_ui.py
│ ├── test_checkout_ui.py
│ └── test_cart_ui.py
│
├── pages/ ← Page Object Model classes
│ ├── login_page.py
│ ├── cart_page.py
│ └── checkout_page.py
│
├── api_clients/ ← Reusable API wrapper classes
│ ├── auth_client.py
│ └── orders_client.py
│
├── utils/ ← Helper functions
│ ├── data_generator.py
│ ├── db_helper.py
│ └── logger.py
│
├── test_data/ ← External test data files
│ ├── users.json
│ └── products.csv
│
├── config/ ← Environment configurations
│ ├── dev_config.py
│ ├── staging_config.py
│ └── prod_config.py
│
├── reports/ ← Generated test reports (usually .gitignored)
│
├── conftest.py ← Shared pytest fixtures
├── pytest.ini ← pytest configuration
├── requirements.txt ← Python dependencies
└── README.md


---

# 4. Configuration Management (Environment Handling)

Frameworks need to run against different environments (dev, staging, production) without changing code — building on the environment concept from Postman (Step 10).

### Example — `config/config.py`

```python
import os

class Config:
    ENVIRONMENTS = {
        "dev": {
            "base_url": "https://dev-api.example.com",
            "ui_url": "https://dev.example.com"
        },
        "staging": {
            "base_url": "https://staging-api.example.com",
            "ui_url": "https://staging.example.com"
        },
        "prod": {
            "base_url": "https://api.example.com",
            "ui_url": "https://example.com"
        }
    }

    @staticmethod
    def get(env=None):
        env = env or os.getenv("TEST_ENV", "staging")
        return Config.ENVIRONMENTS[env]
```

### Usage

```python
from config.config import Config

env_config = Config.get()
BASE_URL = env_config["base_url"]
```

### Running Against a Specific Environment

```bash
TEST_ENV=dev pytest
TEST_ENV=staging pytest
```

> This lets the exact same test suite run against dev, staging, or production just by changing an environment variable — no code changes needed, which is essential for CI/CD pipelines (Step 17).

---

# 5. API Client Layer (Reusable API Wrappers)

Instead of scattering `requests.get()`/`requests.post()` calls across test files, wrap them into reusable client classes — extending the pattern introduced in Step 12.

### Example — `api_clients/auth_client.py`

```python
import requests

class AuthClient:
    def __init__(self, base_url):
        self.base_url = base_url

    def login(self, email, password):
        return requests.post(f"{self.base_url}/login", json={
            "email": email,
            "password": password
        })

    def logout(self, token):
        headers = {"Authorization": f"Bearer {token}"}
        return requests.post(f"{self.base_url}/logout", headers=headers)
```

### Example — `api_clients/orders_client.py`

```python
import requests

class OrdersClient:
    def __init__(self, base_url, token):
        self.base_url = base_url
        self.headers = {"Authorization": f"Bearer {token}"}

    def get_all_orders(self):
        return requests.get(f"{self.base_url}/orders", headers=self.headers)

    def get_order_by_id(self, order_id):
        return requests.get(f"{self.base_url}/orders/{order_id}", headers=self.headers)

    def create_order(self, payload):
        return requests.post(f"{self.base_url}/orders", json=payload, headers=self.headers)
```

### Usage in Tests

```python
def test_get_all_orders(auth_token):
    orders_client = OrdersClient(BASE_URL, auth_token)
    response = orders_client.get_all_orders()
    assert response.status_code == 200
```

> If the API changes (e.g., an endpoint path changes), you update it in **one place** — the client class — instead of every test file that calls it.

---

# 6. Test Data Management

## Approach 1: External Data Files

```json
// test_data/users.json
{
  "valid_user": {
    "email": "test@example.com",
    "password": "Test@1234"
  },
  "locked_user": {
    "email": "locked@example.com",
    "password": "Test@1234"
  }
}
```

```python
import json

def load_test_data(filename):
    with open(f"test_data/{filename}", "r") as f:
        return json.load(f)

def test_login_with_valid_user():
    users = load_test_data("users.json")
    valid_user = users["valid_user"]
    # use valid_user["email"], valid_user["password"] in the test
```

## Approach 2: Data Generation (Factories)

For dynamic data that must be unique each run (e.g., registration tests that can't reuse the same email twice).

```python
# utils/data_generator.py
import random
import string
import time

class TestDataFactory:
    @staticmethod
    def generate_email():
        timestamp = int(time.time())
        return f"testuser{timestamp}@example.com"

    @staticmethod
    def generate_password(length=10):
        chars = string.ascii_letters + string.digits + "!@#$"
        return ''.join(random.choice(chars) for _ in range(length))

    @staticmethod
    def generate_user():
        return {
            "email": TestDataFactory.generate_email(),
            "password": TestDataFactory.generate_password()
        }
```

```python
from utils.data_generator import TestDataFactory

def test_new_user_registration():
    new_user = TestDataFactory.generate_user()
    response = requests.post(f"{BASE_URL}/register", json=new_user)
    assert response.status_code == 201
```

---

# 7. Logging

Good frameworks log meaningful information during execution — critical for debugging failures, especially in CI/CD where you can't watch the test run live.

### Example — `utils/logger.py`

```python
import logging

def setup_logger(name="automation"):
    logger = logging.getLogger(name)
    logger.setLevel(logging.INFO)
    
    handler = logging.StreamHandler()
    formatter = logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')
    handler.setFormatter(formatter)
    logger.addHandler(handler)
    
    return logger

logger = setup_logger()
```

### Usage

```python
from utils.logger import logger

def test_login_flow(page):
    logger.info("Starting login test")
    page.goto("https://example.com/login")
    logger.info("Navigated to login page")
    
    page.get_by_label("Email").fill("test@example.com")
    logger.info("Filled email field")
    
    page.get_by_role("button", name="Login").click()
    logger.info("Clicked login button")
```

---

# 8. Database Helper Layer (Connecting to Step 11)

```python
# utils/db_helper.py
import psycopg2  # example: PostgreSQL driver

class DBHelper:
    def __init__(self, connection_string):
        self.conn = psycopg2.connect(connection_string)

    def execute_query(self, query, params=None):
        cursor = self.conn.cursor()
        cursor.execute(query, params or ())
        result = cursor.fetchall()
        cursor.close()
        return result

    def get_user_by_email(self, email):
        query = "SELECT * FROM users WHERE email = %s"
        return self.execute_query(query, (email,))

    def close(self):
        self.conn.close()
```

### Usage in a Test — Verifying Backend State

```python
def test_registration_saves_to_database(db_helper):
    new_user = TestDataFactory.generate_user()
    response = requests.post(f"{BASE_URL}/register", json=new_user)
    assert response.status_code == 201
    
    # Verify the user actually exists in the database
    db_user = db_helper.get_user_by_email(new_user["email"])
    assert db_user is not None
```

> This directly implements the "API/UI action → SQL verification" pattern introduced in Step 11, now as a reusable framework component.

---

# 9. Base Test Classes (Shared Setup Across Tests)

```python
# tests/base_test.py
import pytest
from api_clients.auth_client import AuthClient
from config.config import Config

class BaseAPITest:
    @pytest.fixture(autouse=True)
    def setup(self):
        self.config = Config.get()
        self.auth_client = AuthClient(self.config["base_url"])
        response = self.auth_client.login("test@example.com", "Test@1234")
        self.token = response.json()["token"]
```

```python
# tests/api/test_orders_api.py
from tests.base_test import BaseAPITest
from api_clients.orders_client import OrdersClient

class TestOrdersAPI(BaseAPITest):
    def test_get_orders(self):
        orders_client = OrdersClient(self.config["base_url"], self.token)
        response = orders_client.get_all_orders()
        assert response.status_code == 200
```

> `autouse=True` means this fixture runs automatically for every test in classes that inherit `BaseAPITest`, without needing to explicitly request it.

---

# 10. Combining API Setup with UI Testing (Hybrid Approach)

A powerful real-world pattern: use fast API calls to set up preconditions, then use UI automation only for what actually needs visual/interaction verification.

```python
def test_user_sees_items_in_cart(page, auth_token):
    # Setup precondition via API (fast)
    orders_client = OrdersClient(BASE_URL, auth_token)
    orders_client.add_to_cart(product_id=45, quantity=2)
    
    # Test the actual UI behavior (what we really want to verify)
    page.goto(f"{UI_URL}/cart")
    expect(page.get_by_text("2 items in cart")).to_be_visible()
```

> This is significantly faster and more stable than clicking through the UI to add items to the cart first — reserving UI automation for genuinely UI-specific verification.

---

# 11. Reporting Integration

```python
# pytest.ini
[pytest]
addopts = --html=reports/report.html --self-contained-html --junitxml=reports/junit.xml
markers =
    smoke: quick sanity checks
    regression: full regression suite
    api: API-level tests
    ui: UI-level tests
```

```bash
pytest -m smoke      # runs only smoke-marked tests, generates HTML + JUnit reports
```

---

# 12. Framework Design Principles (Best Practices)
DRY (Don't Repeat Yourself): centralize locators, API calls, and data generation
Single Responsibility: each class/module does one clear thing (e.g., LoginPage only handles login page logic)
Separation of Concerns: keep test logic, page objects, API clients, and data generation in separate layers
Fail Fast, Fail Clear: assertions and logs should make it obvious WHY a test failed
Environment Agnostic: tests should run against dev/staging/prod without code changes
Independent Tests: tests shouldn't depend on execution order or leftover state from other tests
Clean Test Data: use unique/generated data where needed, and clean up after tests when possible

### Example — Violating vs Following "Independent Tests"

```python
# ❌ BAD: test_2 depends on test_1 running first and leaving data behind
def test_1_create_order():
    global order_id
    order_id = create_order()

def test_2_delete_order():
    delete_order(order_id)   # breaks if run alone or out of order


# ✅ GOOD: each test sets up what it needs independently
def test_delete_order():
    order_id = create_order()   # create its own precondition
    response = delete_order(order_id)
    assert response.status_code == 204
```

---

# 13. Test Pyramid Applied to Framework Design

Revisiting the Testing Pyramid concept (Step 3) in the context of framework structure:
    ▲
   /UI\        Few UI tests (Playwright) — slow, high confidence
  /-----\
 / API   \      More API tests (pytest + requests) — fast, good coverage
/---------\

/ Unit \ Most unit tests (pytest, no network calls) — fastest
/_____________\


A well-designed framework should have **folders/structure reflecting this balance** — not an automation suite that's 90% slow UI tests, which becomes painfully slow and flaky as it grows.

---

# 14. CI/CD Readiness Checklist (Preview for Step 17)

A framework is "CI/CD ready" when it can:

✔ Run headlessly (no visible browser window needed)
✔ Run via a single command (e.g., pytest -m smoke)
✔ Accept environment configuration via environment variables
✔ Produce machine-readable reports (JUnit XML)
✔ Run reliably without manual intervention (no hard-coded waits, no leftover test data issues)
✔ Complete in a reasonable time (parallelized where needed)


```bash
# Example of a CI-friendly command
TEST_ENV=staging pytest -m smoke --headless --junitxml=reports/junit.xml -n 4
```

---

## Key Takeaways

- A framework organizes automation into reusable, maintainable layers instead of scattered scripts — most real-world frameworks are **Hybrid** (POM + Data-Driven, sometimes + BDD).
- A clear **project structure** (tests, pages, api_clients, utils, config, test_data) keeps large suites navigable and easy to onboard into.
- **Configuration management** (via environment variables) lets the same suite run against dev/staging/prod without code changes.
- **API client classes** and **Page Object classes** centralize change points — update one file instead of many when something in the app changes.
- **Test data management** uses either external files (for fixed data) or factories (for unique, dynamically generated data).
- **Logging** and **reporting** (HTML, JUnit XML) make failures debuggable and results consumable by both humans and CI/CD tools.
- Combining **API setup + UI verification** is a key hybrid pattern — fast preconditions via API, UI automation reserved for what truly needs visual/interaction testing.
- Core design principles: **DRY, Single Responsibility, Separation of Concerns, independent tests, environment-agnostic design.**
- A good framework structure reflects the **Testing Pyramid** — mostly fast unit/API tests, fewer slow UI tests.
- Framework "CI/CD readiness" (headless execution, single-command runs, machine-readable reports) directly sets up Step 17.