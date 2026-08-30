# Docker for QA

Docker is a containerization platform that lets QA engineers run applications, databases, and test environments in isolated, consistent, portable containers — solving the classic "works on my machine" problem and making test environments reproducible across local machines and CI/CD pipelines.

---

# 1. What is Docker?

Docker packages an application along with everything it needs to run (code, dependencies, libraries, configuration) into a single unit called a **container**.

## Why QA Needs Docker
Spin up a consistent test environment in seconds (app + database + dependencies)
Avoid "works on my machine" issues — same container runs identically everywhere
Run multiple versions of a database/service side by side without conflicts
Match the CI/CD environment locally (same container used in GitHub Actions, Step 17)
Quickly reset environments to a clean state between test runs
Test against different service versions (e.g., MySQL 5.7 vs 8.0) without installing both locally

---

# 2. Containers vs Virtual Machines

| Virtual Machine | Container |
|---|---|
| Runs a full guest OS on top of a hypervisor | Shares the host OS kernel, isolates only the application layer |
| Heavy (GBs), slow to start (minutes) | Lightweight (MBs), starts in seconds |
| Fully isolated OS-level environment | Process-level isolation |
| Good for running entirely different OS types | Good for fast, repeatable, disposable environments |

Virtual Machine:
Hardware → Hypervisor → Guest OS → App

Container:
Hardware → Host OS → Docker Engine → App (isolated, shares host kernel)


> Containers are faster and lighter, which is exactly why they're preferred for spinning up disposable test environments repeatedly in CI/CD.

---

# 3. Core Docker Concepts

| Term | Meaning |
|---|---|
| **Image** | A read-only blueprint/template for a container (e.g., "python:3.11", "postgres:15") |
| **Container** | A running instance of an image |
| **Dockerfile** | A text file with instructions to build a custom image |
| **Registry** | A place to store/share images (e.g., Docker Hub) |
| **Volume** | Persistent storage that survives even if the container is deleted |
| **Network** | How containers communicate with each other |
| **docker-compose** | A tool for defining and running multi-container applications together |

### Analogy

Image = Recipe (instructions for how to make the dish)
Container = The actual dish, made from following the recipe


---

# 4. Basic Docker Commands

```bash
# Pull an image from Docker Hub
docker pull python:3.11

# List downloaded images
docker images

# Run a container from an image
docker run python:3.11

# Run a container in the background (detached mode)
docker run -d nginx

# Run and give the container an interactive terminal
docker run -it python:3.11 bash

# List running containers
docker ps

# List ALL containers (including stopped ones)
docker ps -a

# Stop a running container
docker stop <container_id>

# Remove a container
docker rm <container_id>

# Remove an image
docker rmi python:3.11

# View logs from a container
docker logs <container_id>

# Execute a command inside a running container
docker exec -it <container_id> bash
```

---

# 5. Running a Database in Docker (Practical QA Example)

Instead of installing PostgreSQL/MySQL locally (which can conflict with other projects or versions), QA can spin up a clean database instantly.

```bash
docker run -d \
  --name test-postgres \
  -e POSTGRES_USER=testuser \
  -e POSTGRES_PASSWORD=testpass \
  -e POSTGRES_DB=testdb \
  -p 5432:5432 \
  postgres:15
```

-d → run in background
--name test-postgres → name the container for easy reference
-e ... → environment variables (credentials, database name)
-p 5432:5432 → map container's port 5432 to host's port 5432
postgres:15 → the image and version to use


### Connecting to It (from Python, building on Step 11)

```python
import psycopg2

conn = psycopg2.connect(
    host="localhost",
    port=5432,
    user="testuser",
    password="testpass",
    dbname="testdb"
)
```

### Resetting Between Test Runs

```bash
docker stop test-postgres
docker rm test-postgres
# Then re-run the docker run command above — completely fresh database, no leftover test data
```

> This solves a very real QA problem: test data accumulating across runs and causing false positives/negatives. A disposable containerized database gives a **truly clean state** every time.

---

# 6. Dockerfile — Building a Custom Image

A Dockerfile defines exactly how to build an image, useful for packaging your automation framework itself so it runs identically anywhere.

### Example — Dockerfile for a pytest/Playwright Automation Project

```dockerfile
# Start from an official Python base image
FROM python:3.11-slim

# Set working directory inside the container
WORKDIR /app

# Copy dependency file first (leverages Docker's caching layer)
COPY requirements.txt .

# Install dependencies
RUN pip install -r requirements.txt

# Install Playwright browsers
RUN playwright install --with-deps chromium

# Copy the rest of the test code
COPY . .

# Default command when the container runs
CMD ["pytest", "tests/", "-v"]
```

### Building and Running the Image

```bash
# Build the image from the Dockerfile
docker build -t qa-automation:latest .

# Run tests inside the container
docker run qa-automation:latest

# Run with a specific marker
docker run qa-automation:latest pytest -m smoke
```

> Copying `requirements.txt` and installing dependencies **before** copying the rest of the code is a deliberate optimization — Docker caches unchanged layers, so if only test code changes (not dependencies), rebuilding is much faster.

---

# 7. Docker Compose — Running Multi-Container Environments

Real applications usually need several services running together (app + database + API), all connected. `docker-compose` defines this entire setup in one YAML file.

### Example — `docker-compose.yml`

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - db
    environment:
      - DATABASE_URL=postgresql://testuser:testpass@db:5432/testdb

  db:
    image: postgres:15
    environment:
      - POSTGRES_USER=testuser
      - POSTGRES_PASSWORD=testpass
      - POSTGRES_DB=testdb
    ports:
      - "5432:5432"

  test-runner:
    build: .
    depends_on:
      - app
      - db
    command: pytest tests/ -v
    environment:
      - BASE_URL=http://app:8000
      - DATABASE_URL=postgresql://testuser:testpass@db:5432/testdb
```

### Commands

```bash
# Start all services defined in the file
docker-compose up

# Start in detached (background) mode
docker-compose up -d

# Stop and remove all containers, networks
docker-compose down

# View logs from all services
docker-compose logs

# Rebuild images before starting
docker-compose up --build
```

### Why This Matters for QA

One command (docker-compose up) spins up:

The application under test
Its database
The automation test suite itself

All properly networked together, in a consistent state, on any machine —
a teammate's laptop, a CI/CD runner, all behave identically.


> Notice `db` in `DATABASE_URL=postgresql://testuser:testpass@db:5432/testdb` isn't an IP address — Docker Compose automatically creates a network where containers can reach each other **by service name**.

---

# 8. Using Docker in CI/CD (Connecting to Step 17)

GitHub Actions can use Docker to ensure the exact same environment locally and in CI.

### Example — GitHub Actions Workflow Using Docker Compose

```yaml
name: Dockerized Test Suite

on: [pull_request]

jobs:
  run-tests:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Run tests with Docker Compose
        run: docker-compose up --build --abort-on-container-exit --exit-code-from test-runner

      - name: Tear down
        if: always()
        run: docker-compose down
```

--abort-on-container-exit → stops all services once the test-runner container finishes
--exit-code-from test-runner → the workflow step's pass/fail is based on the test-runner's exit code


> This means the CI pipeline uses the **exact same containers** a QA engineer would run locally — eliminating environment mismatches between "it passed on my machine" and "it failed in CI."

---

# 9. Volumes — Persisting or Sharing Data

By default, data inside a container is lost when the container is removed. Volumes solve this.

```bash
# Named volume (Docker manages storage location)
docker run -d -v pg_data:/var/lib/postgresql/data postgres:15

# Bind mount (maps a specific host folder into the container)
docker run -v $(pwd)/reports:/app/reports qa-automation:latest
```

### Practical Use — Getting Test Reports Out of a Container

```bash
docker run -v $(pwd)/reports:/app/reports qa-automation:latest pytest --html=reports/report.html
```

> Without the volume mount, the HTML report would be generated **inside** the container and disappear when the container is removed — the bind mount ensures it's saved to your actual local machine (or CI runner's file system) for later access.

---

# 10. Common QA Use Cases for Docker (Summary)
Spin up a disposable test database for isolated, repeatable test runs
Package the automation framework itself into a portable image
Run the full application stack locally for exploratory/manual testing
Match CI/CD environment exactly, avoiding "works on my machine" issues
Test against multiple versions of a dependency (e.g., two different DB versions)
Quickly reset to a clean environment between test cycles
Run mock/stub services in containers for isolated integration testing

---

# 11. Basic Troubleshooting Commands

```bash
# Check container resource usage
docker stats

# Inspect detailed container configuration
docker inspect <container_id>

# View real-time logs
docker logs -f <container_id>

# Check exposed ports
docker port <container_id>

# Remove all stopped containers (cleanup)
docker container prune

# Remove unused images (cleanup)
docker image prune
```

---

## Key Takeaways

- Docker packages applications and their dependencies into lightweight, isolated **containers**, solving environment inconsistency ("works on my machine") issues.
- **Images** are blueprints; **containers** are running instances of those images — containers are much lighter and faster to start than full virtual machines.
- QA can spin up disposable **databases** in seconds, giving a truly clean state for every test run instead of dealing with leftover test data.
- A **Dockerfile** packages the automation framework itself into a portable image that runs identically anywhere.
- **Docker Compose** orchestrates multi-container environments (app + database + test runner) with a single command and a single YAML file.
- Docker integrates directly into **CI/CD** (Step 17) — the same containers used locally can run in GitHub Actions, eliminating environment mismatches.
- **Volumes** persist data beyond a container's lifecycle and let you extract artifacts (like test reports) from inside a container to the host machine.
- Overall, Docker gives QA **fast, consistent, disposable environments** — a practical skill that ties directly into automation frameworks (Step 15) and CI/CD pipelines (Step 17).