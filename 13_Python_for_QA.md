markdown
# Python for QA

This chapter covers the Python fundamentals QA engineers actually need for test automation — not a full programming course, but the practical subset used in writing test scripts, working with pytest (Step 13), and building automation frameworks (Step 15).

---

# 1. Why Python for QA?
Simple, readable syntax — faster to learn than Java/C#
Huge ecosystem of testing libraries (pytest, requests, Selenium, Playwright)
Widely used for API testing, UI automation, and data-driven testing
Strong community support and documentation

---

# 2. Variables and Data Types

```python
# Variables (no need to declare type explicitly)
email = "test@example.com"        # string
age = 25                           # integer
price = 49.99                      # float
is_active = True                   # boolean
user = None                        # represents "no value"
```

## Common Data Types

| Type | Example | Use in QA |
|---|---|---|
| `str` | `"test@example.com"` | Test data, assertions |
| `int` | `25` | Counts, IDs, status codes |
| `float` | `49.99` | Prices, response times |
| `bool` | `True`/`False` | Flags, condition checks |
| `list` | `[1, 2, 3]` | Collections of test data |
| `dict` | `{"key": "value"}` | JSON-like data (API responses) |
| `tuple` | `(1, 2)` | Fixed, unchangeable collections |
| `set` | `{1, 2, 3}` | Unique values only |

---

# 3. Strings (Heavily Used in Test Data & Assertions)

```python
email = "Test@Example.com"

email.lower()                  # "test@example.com"
email.upper()                  # "TEST@EXAMPLE.COM"
email.strip()                  # removes leading/trailing whitespace
email.replace("Test", "User")  # "User@Example.com"
email.split("@")               # ["Test", "Example.com"]
len(email)                     # 17

# f-strings (formatted strings) — very common in test scripts
name = "John"
print(f"Welcome, {name}!")     # "Welcome, John!"

# Checking content
"@gmail.com" in email          # False
email.startswith("Test")       # True
email.endswith(".com")         # True
```

### Practical QA Example

```python
def generate_unique_email():
    import time
    timestamp = int(time.time())
    return f"testuser{timestamp}@example.com"

print(generate_unique_email())  # testuser1756540800@example.com
```

---

# 4. Lists (Collections of Test Data)

```python
browsers = ["Chrome", "Firefox", "Edge", "Safari"]

browsers[0]              # "Chrome"
browsers[-1]              # "Safari" (last item)
browsers.append("Opera")  # add to end
browsers.remove("Edge")   # remove specific item
len(browsers)              # count of items

# Looping through a list — common for data-driven testing
for browser in browsers:
    print(f"Testing on {browser}")
```

### Practical QA Example — Data-Driven Test Inputs

```python
invalid_emails = [
    "",                    # empty
    "plainaddress",        # no @
    "@missingusername.com",# no username
    "test@.com",           # missing domain
    "test test@example.com" # space in email
]

for email in invalid_emails:
    print(f"Testing invalid email: {email}")
    # In real automation, this would call a validate_email() function
    # and assert that it correctly rejects each one
```

---

# 5. Dictionaries (Key-Value Data — Mirrors JSON)

Dictionaries are essential for QA because API responses are typically JSON, which maps directly to Python dictionaries.

```python
user = {
    "id": 101,
    "email": "test@example.com",
    "is_active": True
}

user["email"]                  # "test@example.com"
user.get("phone", "N/A")        # "N/A" (default if key doesn't exist)
user["role"] = "admin"          # add new key
del user["is_active"]           # remove a key

# Looping through a dictionary
for key, value in user.items():
    print(f"{key}: {value}")
```

### Practical QA Example — Working with API Response

```python
response_data = {
    "status": "success",
    "user": {
        "id": 101,
        "email": "test@example.com"
    },
    "token": "eyJhbGciOiJIUzI1NiIs..."
}

assert response_data["status"] == "success"
assert response_data["user"]["email"] == "test@example.com"
assert "token" in response_data
```

---

# 6. Conditionals

```python
status_code = 401

if status_code == 200:
    print("Success")
elif status_code == 401:
    print("Unauthorized")
elif status_code == 404:
    print("Not Found")
else:
    print("Unexpected status code")
```

### Practical QA Example

```python
def check_login_response(status_code, response_body):
    if status_code == 200 and "token" in response_body:
        return "Login successful"
    elif status_code == 401:
        return "Invalid credentials"
    elif status_code == 429:
        return "Too many attempts - rate limited"
    else:
        return f"Unexpected response: {status_code}"
```

---

# 7. Loops

```python
# for loop
for i in range(5):
    print(i)   # 0, 1, 2, 3, 4

# while loop
attempts = 0
while attempts < 3:
    print(f"Attempt {attempts + 1}")
    attempts += 1
```

### Practical QA Example — Retry Logic

```python
import time

def wait_for_element(check_function, max_attempts=5, delay=2):
    attempts = 0
    while attempts < max_attempts:
        if check_function():
            return True
        time.sleep(delay)
        attempts += 1
    return False  # element never appeared within max attempts
```

---

# 8. Functions

```python
def add_to_cart(product_id, quantity=1):
    """Adds a product to the cart with a default quantity of 1."""
    return {"product_id": product_id, "quantity": quantity}

result = add_to_cart(45)               # uses default quantity=1
result2 = add_to_cart(45, quantity=3)  # explicit quantity
```

### Practical QA Example — Reusable Test Helper

```python
def is_valid_email(email):
    """Basic email format validation for test purposes."""
    if "@" not in email:
        return False
    if email.count("@") != 1:
        return False
    local, domain = email.split("@")
    if not local or not domain:
        return False
    if "." not in domain:
        return False
    return True

# Usage in tests
assert is_valid_email("test@example.com") == True
assert is_valid_email("plainaddress") == False
assert is_valid_email("test@@example.com") == False
```

---

# 9. Modules and Imports

```python
import requests          # for API calls
import json               # for JSON handling
import time                # for delays/timestamps
import random               # for random test data
from datetime import datetime  # for date/time handling
```

### Practical QA Example — Random Test Data Generation

```python
import random
import string

def generate_random_password(length=10):
    characters = string.ascii_letters + string.digits + "!@#$%"
    return ''.join(random.choice(characters) for _ in range(length))

print(generate_random_password())  # e.g., "aB3!kL9@qZ"
```

---

# 10. Working with JSON

Since API testing revolves around JSON, this is one of the most-used skills.

```python
import json

# Convert Python dict to JSON string
user = {"id": 101, "email": "test@example.com"}
json_string = json.dumps(user)
print(json_string)   # '{"id": 101, "email": "test@example.com"}'

# Convert JSON string back to Python dict
parsed = json.loads(json_string)
print(parsed["email"])  # "test@example.com"
```

---

# 11. Making API Requests with the `requests` Library

This is the most important library for QA doing API testing/automation in Python — it directly builds on Step 10 (API Testing + Postman) but lets you automate what you did manually in Postman.

## Installing
```bash
pip install requests
```

## GET Request

```python
import requests

response = requests.get("https://api.example.com/v1/products")

print(response.status_code)     # 200
print(response.json())           # parsed JSON response body
```

## POST Request

```python
import requests

url = "https://api.example.com/v1/login"
payload = {
    "email": "test@example.com",
    "password": "Test@1234"
}
headers = {
    "Content-Type": "application/json"
}

response = requests.post(url, json=payload, headers=headers)

print(response.status_code)      # 200
print(response.json())            # {"status": "success", "token": "..."}
```

## Using Headers/Authorization (Bearer Token)

```python
token = "eyJhbGciOiJIUzI1NiIs..."
headers = {
    "Authorization": f"Bearer {token}"
}

response = requests.get("https://api.example.com/v1/orders", headers=headers)
```

## Full Example — Login Then Fetch Protected Data

```python
import requests

BASE_URL = "https://api.example.com/v1"

# Step 1: Login
login_response = requests.post(f"{BASE_URL}/login", json={
    "email": "test@example.com",
    "password": "Test@1234"
})

assert login_response.status_code == 200
token = login_response.json()["token"]

# Step 2: Use token to fetch protected resource
headers = {"Authorization": f"Bearer {token}"}
orders_response = requests.get(f"{BASE_URL}/orders", headers=headers)

assert orders_response.status_code == 200
orders = orders_response.json()
print(f"Found {len(orders)} orders")
```

> Notice this replicates exactly what you'd do manually in Postman — login, capture token, chain into the next request — but now it's scriptable and repeatable, which is the foundation for Step 13 (pytest) and Step 15 (Automation Framework Design).

---

# 12. Exception Handling (try/except)

Important for writing robust automation scripts that don't crash on unexpected failures — instead, they handle errors gracefully or report them clearly.

```python
try:
    response = requests.get("https://api.example.com/v1/products", timeout=5)
    response.raise_for_status()   # raises an error if status code is 4xx/5xx
except requests.exceptions.Timeout:
    print("Request timed out")
except requests.exceptions.HTTPError as e:
    print(f"HTTP error occurred: {e}")
except requests.exceptions.RequestException as e:
    print(f"Request failed: {e}")
else:
    print("Request succeeded")
    data = response.json()
```

---

# 13. Assertions (Core to All Testing)

Python's built-in `assert` statement is the foundation of test validation — pytest (Step 13) builds directly on this.

```python
status_code = 200
assert status_code == 200, "Expected status code 200"

response_body = {"status": "success"}
assert response_body["status"] == "success", "Expected status to be success"

email = "test@example.com"
assert "@" in email, "Email should contain @"
```

> If the condition after `assert` is `False`, Python raises an `AssertionError` with the given message — this is exactly how automated tests report failures.

---

# 14. List Comprehensions (Common Pattern in Test Scripts)

A concise way to build lists, frequently seen in real automation code.

```python
# Traditional loop
valid_emails = []
for email in ["a@x.com", "invalid", "b@y.com"]:
    if "@" in email:
        valid_emails.append(email)

# Equivalent list comprehension
valid_emails = [email for email in ["a@x.com", "invalid", "b@y.com"] if "@" in email]
print(valid_emails)  # ["a@x.com", "b@y.com"]
```

### Practical QA Example — Extracting Data from API Response

```python
response_data = {
    "products": [
        {"id": 1, "name": "Shoes", "price": 49.99},
        {"id": 2, "name": "Bag", "price": 29.99},
        {"id": 3, "name": "Hat", "price": 15.00}
    ]
}

# Extract just the names
product_names = [p["name"] for p in response_data["products"]]
print(product_names)  # ["Shoes", "Bag", "Hat"]

# Extract products under $30
cheap_products = [p for p in response_data["products"] if p["price"] < 30]
```

---

# 15. Classes and Objects (Basics — Needed for Automation Frameworks)

Object-Oriented Programming (OOP) becomes important later for structuring automation frameworks (Step 15) and Page Object Models in Playwright (Step 14).

```python
class User:
    def __init__(self, email, password):
        self.email = email
        self.password = password
    
    def get_credentials(self):
        return {"email": self.email, "password": self.password}

# Usage
test_user = User("test@example.com", "Test@1234")
print(test_user.email)
print(test_user.get_credentials())
```

### Practical QA Example — Simple API Client Class

```python
import requests

class APIClient:
    def __init__(self, base_url):
        self.base_url = base_url
        self.token = None

    def login(self, email, password):
        response = requests.post(f"{self.base_url}/login", json={
            "email": email,
            "password": password
        })
        self.token = response.json().get("token")
        return response

    def get(self, endpoint):
        headers = {"Authorization": f"Bearer {self.token}"}
        return requests.get(f"{self.base_url}{endpoint}", headers=headers)

# Usage
client = APIClient("https://api.example.com/v1")
client.login("test@example.com", "Test@1234")
response = client.get("/orders")
print(response.json())
```

> This pattern — wrapping API calls into a reusable class — is exactly how real automation frameworks are structured, which Step 15 will expand on significantly.

---

# 16. File Handling (Reading Test Data from Files)

```python
# Reading a JSON file of test data
import json

with open("test_data.json", "r") as file:
    test_data = json.load(file)

print(test_data)

# Reading a CSV file (common for data-driven testing)
import csv

with open("users.csv", "r") as file:
    reader = csv.DictReader(file)
    for row in reader:
        print(row["email"], row["password"])
```

---

## Key Takeaways

- Python's simple syntax and rich ecosystem make it the standard language for QA automation (API testing, UI automation, frameworks).
- **Dictionaries** are essential since they map directly to JSON — the format most APIs use.
- **Lists** and **list comprehensions** support data-driven testing (looping through multiple test inputs).
- The **`requests` library** automates exactly what you'd do manually in Postman — sending requests, chaining tokens, checking status codes and response bodies.
- **Exception handling** (`try/except`) makes automation scripts robust against timeouts and unexpected failures.
- **`assert`** statements are the foundation of all automated test validation — pytest builds directly on this concept.
- **Classes** let QA structure reusable components (like an API client) — this is the foundation for automation framework design (Step 15) and Page Object Models (Step 14).
- File handling (JSON/CSV) supports reading external test data for data-driven test scripts.