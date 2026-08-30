markdown
# Playwright Automation

Playwright is a modern browser automation framework used for UI testing — this chapter covers how QA engineers automate real user interactions in a browser, building on Python (Step 13) and pytest (Step 14).

---

# 1. What is Playwright?

Playwright is an open-source browser automation library (originally by Microsoft) that allows you to control real browsers programmatically — clicking buttons, filling forms, navigating pages, and verifying what appears on screen.

## Why Playwright (vs Selenium)
Faster and more reliable than Selenium
Built-in auto-waiting (no manual sleep/wait needed in most cases)
Supports Chromium, Firefox, and WebKit (Safari engine) with one API
Built-in support for mobile viewport emulation
Handles modern web features well (iframes, shadow DOM, single-page apps)
Built-in tools: Trace Viewer, Codegen, and screenshot/video recording
Native async support and strong parallelization

---

# 2. Installing Playwright (Python)

```bash
pip install pytest-playwright
playwright install          # downloads browser binaries (Chromium, Firefox, WebKit)
```

---

# 3. Locators — Finding Elements on a Page

Locators are how Playwright finds elements (buttons, input fields, links) on a webpage to interact with them. Choosing good locators is one of the most important automation skills.

## Recommended Locator Strategies (in order of preference)

```python
# 1. By role (most robust, mirrors how users/assistive tech perceive the page)
page.get_by_role("button", name="Login")

# 2. By label (great for form fields)
page.get_by_label("Email")

# 3. By placeholder
page.get_by_placeholder("Enter your email")

# 4. By text content
page.get_by_text("Forgot Password?")

# 5. By test ID (best when developers add data-testid attributes)
page.get_by_test_id("login-submit-button")

# 6. CSS selector (use when nothing else fits)
page.locator("#email-input")

# 7. XPath (last resort — least readable, most fragile)
page.locator("//input[@id='email']")
```

### Why Locator Choice Matters

❌ Fragile: page.locator("div > div:nth-child(3) > button")
(breaks easily if the page layout changes even slightly)

✅ Robust: page.get_by_role("button", name="Login")
(survives layout/CSS changes, tied to meaning, not structure)


> A common real-world problem: tests break not because of a real bug, but because a `<div>` was added somewhere in the DOM. Choosing semantic locators (role, label, test-id) instead of brittle CSS/XPath paths avoids this.

---

# 4. Basic Playwright Test Structure (with pytest)

```python
# test_login.py
import re
from playwright.sync_api import Page, expect

def test_successful_login(page: Page):
    page.goto("https://example.com/login")
    
    page.get_by_label("Email").fill("test@example.com")
    page.get_by_label("Password").fill("Test@1234")
    page.get_by_role("button", name="Login").click()
    
    expect(page).to_have_url(re.compile(".*dashboard"))
    expect(page.get_by_text("Welcome, Test User")).to_be_visible()
```

> Notice `page` is a **fixture** automatically provided by `pytest-playwright` — no manual browser setup needed, following the fixture pattern from Step 13.

---

# 5. Common Actions

```python
# Navigation
page.goto("https://example.com")
page.go_back()
page.go_forward()
page.reload()

# Typing and filling
page.get_by_label("Email").fill("test@example.com")   # clears field first, then types
page.get_by_label("Search").type("laptop")              # types character by character (simulates real typing)

# Clicking
page.get_by_role("button", name="Submit").click()
page.get_by_role("checkbox", name="Remember me").check()
page.get_by_role("checkbox", name="Remember me").uncheck()

# Dropdowns
page.get_by_label("Country").select_option("Nepal")

# Keyboard
page.get_by_label("Search").press("Enter")

# Hovering
page.get_by_text("Menu").hover()

# File upload
page.get_by_label("Upload photo").set_input_files("photo.jpg")
```

---

# 6. Assertions with `expect()`

Playwright's `expect()` function provides **auto-retrying assertions** — it automatically waits and rechecks the condition until it's true or a timeout is reached, instead of failing instantly.

```python
from playwright.sync_api import expect

# Visibility
expect(page.get_by_text("Login successful")).to_be_visible()
expect(page.get_by_text("Loading...")).to_be_hidden()

# Text content
expect(page.get_by_role("heading")).to_have_text("Dashboard")
expect(page.get_by_role("heading")).to_contain_text("Dash")

# URL
expect(page).to_have_url("https://example.com/dashboard")
expect(page).to_have_title("My Dashboard")

# Element state
expect(page.get_by_role("button", name="Submit")).to_be_enabled()
expect(page.get_by_role("button", name="Submit")).to_be_disabled()
expect(page.get_by_role("checkbox")).to_be_checked()

# Count
expect(page.get_by_role("listitem")).to_have_count(5)

# Input value
expect(page.get_by_label("Email")).to_have_value("test@example.com")
```

### Why Auto-Waiting Matters

Problem in older tools (like early Selenium):
Click button → page starts loading → immediately check for text
→ FAILS because the page hasn't finished loading yet
→ Developer adds time.sleep(3) as a workaround → slow, unreliable tests

Playwright's approach:
expect(page.get_by_text("Success")).to_be_visible()
→ automatically retries checking for up to 30 seconds (configurable)
→ passes as soon as the condition is true, fails only if it never becomes true


> This directly solves a huge source of **flaky tests** in UI automation — no more guessing sleep durations.

---

# 7. Handling Waits (When Auto-Waiting Isn't Enough)

While `expect()` handles most waiting automatically, sometimes explicit waits are needed for specific scenarios.

```python
# Wait for a specific element to appear
page.wait_for_selector("#results-loaded")

# Wait for navigation to complete
with page.expect_navigation():
    page.get_by_role("button", name="Submit").click()

# Wait for a specific network response
with page.expect_response("**/api/login") as response_info:
    page.get_by_role("button", name="Login").click()
response = response_info.value
assert response.status == 200

# Wait for a certain load state
page.wait_for_load_state("networkidle")
```

> **Avoid** `page.wait_for_timeout(3000)` (a hard sleep) except as a last resort — it slows down tests and doesn't actually guarantee the condition you need is met.

---

# 8. Page Object Model (POM) — Structuring Automation Code

The Page Object Model is a design pattern where each page (or component) of the application is represented as a class, containing its locators and actions. This keeps tests clean and makes maintenance much easier — directly setting up Step 15 (Automation Framework Design).

### Without POM (Repetitive, Hard to Maintain)

```python
def test_login_1(page):
    page.goto("https://example.com/login")
    page.get_by_label("Email").fill("test@example.com")
    page.get_by_label("Password").fill("Test@1234")
    page.get_by_role("button", name="Login").click()

def test_login_2(page):
    page.goto("https://example.com/login")
    page.get_by_label("Email").fill("wrong@example.com")
    page.get_by_label("Password").fill("wrongpass")
    page.get_by_role("button", name="Login").click()
```

> If the login page's locators change (e.g., label text changes from "Email" to "Email Address"), **every single test** using these locators needs to be updated individually.

### With POM (Reusable, Maintainable)

```python
# pages/login_page.py
class LoginPage:
    def __init__(self, page):
        self.page = page
        self.email_input = page.get_by_label("Email")
        self.password_input = page.get_by_label("Password")
        self.login_button = page.get_by_role("button", name="Login")
        self.error_message = page.get_by_text("Invalid email or password")

    def goto(self):
        self.page.goto("https://example.com/login")

    def login(self, email, password):
        self.email_input.fill(email)
        self.password_input.fill(password)
        self.login_button.click()
```

```python
# test_login.py
from pages.login_page import LoginPage

def test_valid_login(page):
    login_page = LoginPage(page)
    login_page.goto()
    login_page.login("test@example.com", "Test@1234")
    
    expect(page).to_have_url(re.compile(".*dashboard"))

def test_invalid_login(page):
    login_page = LoginPage(page)
    login_page.goto()
    login_page.login("test@example.com", "wrongpass")
    
    expect(login_page.error_message).to_be_visible()
```

> Now if the locator changes, it's updated in **one place** (`login_page.py`) instead of every test file — this is the single most important pattern for maintainable UI automation.

---

# 9. Fixtures for Playwright + pytest

```python
# conftest.py
import pytest
from playwright.sync_api import sync_playwright

@pytest.fixture(scope="function")
def logged_in_page(page):
    page.goto("https://example.com/login")
    page.get_by_label("Email").fill("test@example.com")
    page.get_by_label("Password").fill("Test@1234")
    page.get_by_role("button", name="Login").click()
    page.wait_for_url("**/dashboard")
    yield page
```

```python
# test_dashboard.py
def test_dashboard_shows_welcome_message(logged_in_page):
    expect(logged_in_page.get_by_text("Welcome, Test User")).to_be_visible()
```

> This reuses the login flow across many tests without repeating the login steps in every single test function.

---

# 10. Handling Multiple Browser Contexts and Tabs

```python
# Testing multiple users interacting simultaneously (e.g., chat feature)
def test_two_users_can_chat(browser):
    user1_context = browser.new_context()
    user2_context = browser.new_context()
    
    user1_page = user1_context.new_page()
    user2_page = user2_context.new_page()
    
    user1_page.goto("https://example.com/chat")
    user2_page.goto("https://example.com/chat")
    
    # Simulate user1 sending a message, user2 receiving it
    user1_page.get_by_label("Message").fill("Hello!")
    user1_page.get_by_role("button", name="Send").click()
    
    expect(user2_page.get_by_text("Hello!")).to_be_visible()
```

### Handling a New Tab/Popup

```python
def test_opens_new_tab(page, context):
    with context.expect_page() as new_page_info:
        page.get_by_role("link", name="Open in new tab").click()
    new_page = new_page_info.value
    new_page.wait_for_load_state()
    expect(new_page).to_have_title("New Tab Page")
```

---

# 11. API Testing with Playwright (Bonus Capability)

Playwright can also send raw API requests — useful for setting up test data quickly before a UI test, without going through the UI (a common time-saving pattern).

```python
def test_checkout_flow(page, request):
    # Set up test data via API (faster than clicking through UI)
    response = request.post("https://api.example.com/v1/cart", data={
        "product_id": 45,
        "quantity": 2
    })
    assert response.status == 201
    
    # Now test the actual UI flow
    page.goto("https://example.com/checkout")
    expect(page.get_by_text("2 items in cart")).to_be_visible()
```

> This is a very common real-world pattern: use API calls to quickly set up preconditions (e.g., "user already has items in cart"), then use UI automation only to test the specific flow you actually care about.

---

# 12. Screenshots, Videos, and Traces (Debugging Failures)

```python
# Take a screenshot manually
page.screenshot(path="screenshot.png")

# Take a full-page screenshot
page.screenshot(path="full_page.png", full_page=True)
```

### Configuring Automatic Capture (via `pytest.ini` or CLI)

```bash
pytest --screenshot=only-on-failure --video=retain-on-failure --tracing=retain-on-failure
```

## Trace Viewer

Playwright can record a full **trace** of a test run (DOM snapshots, network requests, console logs, timeline) — extremely useful for debugging why a test failed, especially in CI environments where you can't watch it run live.

```bash
playwright show-trace trace.zip
```

> When a test fails in CI/CD (Step 17), the trace file lets you visually "replay" exactly what happened in the browser, step by step — a huge time-saver over trying to reproduce the failure locally.

---

# 13. Codegen — Auto-Generating Test Code

Playwright includes a tool that records your manual browser actions and generates corresponding Playwright code automatically — useful for quickly starting a test or discovering good locators.

```bash
playwright codegen https://example.com
```

This opens a browser; as you click and type, Playwright generates the equivalent code in real time. It's a great learning and starting-point tool, but generated code often needs cleanup (better locators, added assertions, POM structure) before being production-ready.

---

# 14. Handling Common UI Testing Challenges

## iframes

```python
frame = page.frame_locator("#payment-iframe")
frame.get_by_label("Card Number").fill("4111111111111111")
```

## Dynamic/Loading Content

```python
# Wait for a loading spinner to disappear before continuing
expect(page.get_by_text("Loading...")).to_be_hidden()
page.get_by_role("button", name="Continue").click()
```

## Handling Alerts/Dialogs

```python
def test_delete_confirmation(page):
    page.on("dialog", lambda dialog: dialog.accept())  # auto-accept confirm dialogs
    page.get_by_role("button", name="Delete").click()
```

---

# 15. Cross-Browser Testing

```python
# Run tests across multiple browsers via command line
```
```bash
pytest --browser chromium
pytest --browser firefox
pytest --browser webkit
pytest --browser chromium --browser firefox --browser webkit   # all three
```

### Practical QA Relevance

This directly implements Compatibility Testing (from Step 3) —
instead of manually checking a feature across Chrome/Firefox/Safari,
the same automated test suite runs against all three engines automatically.


---

## Key Takeaways

- Playwright automates real browser interactions and supports Chromium, Firefox, and WebKit with a single API.
- **Locator strategy matters** — prefer role/label/test-id locators over CSS/XPath for resilience against UI changes.
- `expect()` provides **auto-retrying assertions**, eliminating most flaky-test problems caused by manual `sleep()` calls.
- The **Page Object Model (POM)** centralizes locators and actions per page, so UI changes require updates in one place, not across every test.
- Fixtures (from Step 13) integrate naturally with Playwright — e.g., a `logged_in_page` fixture avoids repeating login steps across tests.
- Playwright can also fire raw **API requests**, letting tests set up data quickly via API and reserve UI automation for what actually needs UI verification.
- **Screenshots, videos, and Trace Viewer** make debugging failed tests (especially in CI/CD) far easier than trying to reproduce issues manually.
- **Codegen** is useful for quickly bootstrapping tests, but generated code typically needs cleanup before being production-quality.
- Running the same suite with `--browser chromium/firefox/webkit` directly implements automated **Compatibility Testing**.