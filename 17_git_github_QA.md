# Git & GitHub for QA

Version control is essential for QA engineers who write automation code — this chapter covers the Git fundamentals needed to collaborate on test repositories, manage code changes, and work within a team's development workflow.

---

# 1. What is Git?

Git is a distributed version control system that tracks changes to files over time, allowing multiple people to collaborate on the same codebase without overwriting each other's work.

## Why QA Needs Git
Store and version automation test code (not just app code)
Collaborate with other QA engineers and developers on the same test suite
Track history of test changes (useful when investigating "when did this test start failing?")
Work on new tests/features without affecting the main stable test suite
Integrate automated tests into CI/CD pipelines (Step 17), which rely entirely on Git

## Git vs GitHub

| Git | GitHub |
|---|---|
| Version control **software** installed locally | A **cloud platform** for hosting Git repositories |
| Works entirely offline | Provides remote hosting, collaboration tools, PRs, CI/CD (Actions) |
| Command-line tool | Web-based interface + integrations |

> Similar platforms: GitLab, Bitbucket — the underlying Git concepts are the same across all of them.

---

# 2. Basic Git Concepts

| Term | Meaning |
|---|---|
| **Repository (repo)** | A project folder tracked by Git |
| **Commit** | A saved snapshot of changes, with a message describing what changed |
| **Branch** | An independent line of development (e.g., for a new feature or test) |
| **Merge** | Combining changes from one branch into another |
| **Remote** | A version of the repository hosted elsewhere (e.g., on GitHub) |
| **Clone** | Downloading a copy of a remote repository to your local machine |
| **Pull** | Fetching and merging changes from the remote into your local branch |
| **Push** | Sending your local commits to the remote repository |
| **Working Directory** | The files you're currently editing |
| **Staging Area (Index)** | Changes marked to be included in the next commit |

---

# 3. Basic Git Workflow

Working Directory → Staging Area → Local Repository → Remote Repository
(edit file) (git add) (git commit) (git push)


## Essential Commands

```bash
# Initialize a new repository
git init

# Clone an existing repository
git clone https://github.com/company/qa-automation.git

# Check current status (what's changed, staged, etc.)
git status

# Stage changes
git add test_login.py           # stage a specific file
git add .                        # stage all changed files

# Commit staged changes
git commit -m "Add login negative test cases"

# Push commits to remote
git push origin main

# Pull latest changes from remote
git pull origin main

# View commit history
git log
git log --oneline               # condensed view
```

---

# 4. Branching (Critical for QA Collaboration)

Branches let you work on new tests or changes without affecting the stable, working version of the test suite that others rely on.

```bash
# Create a new branch
git branch feature/login-negative-tests

# Switch to a branch
git checkout feature/login-negative-tests

# Create and switch in one command
git checkout -b feature/login-negative-tests

# Newer syntax (Git 2.23+)
git switch -c feature/login-negative-tests

# List all branches
git branch

# Switch back to main branch
git checkout main
```

### Typical QA Branching Workflow

main (stable, working test suite)
│
├── feature/checkout-api-tests (new tests being developed)
├── bugfix/flaky-login-test (fixing a broken/flaky test)
└── feature/add-playwright-cart-tests


### Example Real Workflow

```bash
# 1. Make sure main is up to date
git checkout main
git pull origin main

# 2. Create a new branch for your work
git checkout -b feature/cart-api-tests

# 3. Write your tests, then stage and commit
git add tests/api/test_cart_api.py
git commit -m "Add API tests for cart add/remove functionality"

# 4. Push the branch to GitHub
git push origin feature/cart-api-tests

# 5. Open a Pull Request on GitHub for review
```

---

# 5. Naming Conventions for Branches

Consistent branch naming helps teams quickly understand what a branch contains.

feature/short-description → new tests/functionality
bugfix/short-description → fixing a broken test or automation bug
hotfix/short-description → urgent fix
chore/short-description → maintenance (e.g., updating dependencies)

Examples:
feature/checkout-regression-suite
bugfix/fix-flaky-login-test
chore/update-playwright-version


---

# 6. Merging Branches

## Fast-Forward Merge
When the target branch hasn't changed since the feature branch was created — Git simply moves the pointer forward.

## Three-Way Merge
When both branches have diverged (changes made on both) — Git creates a new "merge commit" combining both histories.

```bash
git checkout main
git merge feature/cart-api-tests
```

### Merge Conflicts

Occurs when the same lines of code were changed differently on two branches — Git can't automatically decide which version to keep.

```python
<<<<<<< HEAD
def test_login_valid():
    assert login("test@example.com", "Test@1234") == 200
=======
def test_login_valid():
    assert login("test@example.com", "Test@1234").status_code == 200
>>>>>>> feature/cart-api-tests
```

**Resolving a conflict:**
Open the conflicted file
Manually decide which version (or a combination) is correct
Remove the conflict markers (<<<<<<<, =======, >>>>>>>)
Stage the resolved file: git add test_login.py
Complete the merge: git commit

---

# 7. Pull Requests (PRs) — The Real-World Collaboration Workflow

A Pull Request is a request to merge changes from one branch into another, typically reviewed by teammates before merging — this is how most real QA teams actually work on GitHub.

## Typical PR Workflow
Push your feature branch to GitHub
Open a Pull Request (feature branch → main)
Add a clear title and description of what changed and why
Teammates/leads review the code, leave comments
Address feedback (push more commits to the same branch)
Once approved, merge the PR into main
Delete the feature branch (optional cleanup)

### Example PR Description (Good Practice)

```markdown
## What this PR does
Adds negative test cases for the login API, covering:
- Invalid email format
- Unregistered email
- Missing required fields
- SQL injection payload in email field

## How to test
Run: `pytest tests/api/test_login_api.py -v`

## Related
Closes JIRA-245
```

> Linking to the related Jira ticket (from Step 8) keeps traceability between code changes and requirements/bugs — connecting directly back to earlier chapters.

---

# 8. Code Review (QA-Specific Considerations)

When reviewing a teammate's automation PR, QA-specific things to check:
Are locators robust (role/label/test-id) or fragile (deep CSS/XPath)? (Step 14)
Are assertions specific and meaningful, not just "status_code == 200"?
Is test data properly generated/cleaned up, or hardcoded/reused unsafely?
Are tests independent (don't rely on execution order)? (Step 15)
Is there unnecessary duplication that should be refactored into a shared fixture/helper?
Are negative/edge cases covered, not just the happy path? (Step 4/5)
Does the PR include both positive and negative test scenarios where relevant?

---

# 9. .gitignore (Keeping the Repo Clean)

Prevents unnecessary or sensitive files from being tracked/committed.

### Example — `.gitignore` for a Python Automation Project
Python

pycache/
*.pyc
venv/
.env

Test artifacts

reports/
screenshots/
videos/
traces/
.pytest_cache/

IDE

.vscode/
.idea/

Playwright

playwright-report/
test-results/


> Reports, screenshots, and videos are generated fresh on every test run — committing them clutters the repository and causes unnecessary merge conflicts. Similarly, `.env` files often contain secrets/credentials that should **never** be committed.

---

# 10. Handling Sensitive Data (Credentials, Tokens)

❌ Never commit passwords, API keys, or tokens directly in code:
password = "RealProdPassword123" # BAD

✅ Use environment variables instead:
password = os.getenv("TEST_USER_PASSWORD")


If credentials are accidentally committed, simply deleting them in a later commit is **not enough** — they remain in Git history and must be properly rotated (changed) and purged from history using tools like `git filter-repo` or GitHub's secret scanning remediation.

---

# 11. Undoing Changes (Common QA Scenarios)

```bash
# Discard uncommitted changes to a file (careful — this is permanent)
git checkout -- test_login.py

# Unstage a file (keep the changes, just remove from staging)
git reset test_login.py

# Undo the last commit but keep the changes (uncommitted)
git reset --soft HEAD~1

# Completely undo the last commit and its changes (dangerous)
git reset --hard HEAD~1

# Revert a specific commit (creates a NEW commit that undoes it — safer for shared branches)
git revert <commit-hash>

# See what changed in a specific commit
git show <commit-hash>

# Temporarily stash uncommitted changes (e.g., to switch branches quickly)
git stash
git stash pop     # bring the stashed changes back
```

> `git revert` is generally preferred over `git reset --hard` on shared branches (like `main`), because it doesn't rewrite history that others may have already pulled — rewriting shared history causes major problems for the whole team.

---

# 12. Git Log and History Investigation

Useful for QA investigating **when** a test started failing or **who** changed a specific line.

```bash
# View commit history for a specific file
git log tests/api/test_login_api.py

# See exactly what changed in each commit
git log -p tests/api/test_login_api.py

# Find who last modified each line (useful for asking "why was this changed?")
git blame tests/api/test_login_api.py

# Search commit messages for a keyword
git log --grep="login"

# Compare differences between two branches
git diff main feature/cart-api-tests
```

### Practical QA Example

Scenario: A test that used to pass is now failing after a recent merge.

git log --oneline tests/api/test_checkout_api.py
→ Shows recent commits touching this file

git show <suspicious-commit-hash>
→ Shows exactly what changed

This often reveals whether the test itself was changed incorrectly,
or whether it's now correctly catching a real regression in the app.


---

# 13. Working with Remotes

```bash
# View configured remotes
git remote -v

# Add a remote
git remote add origin https://github.com/company/qa-automation.git

# Fetch changes without merging (see what's new first)
git fetch origin

# Fetch AND merge (pull = fetch + merge)
git pull origin main
```

---

# 14. Tags (Marking Releases)

Tags mark specific important points in history, often used to mark a version of the test suite that corresponds to a specific application release.

```bash
git tag v1.2.0
git tag -a v1.2.0 -m "Test suite for release 1.2.0"
git push origin v1.2.0
git push origin --tags     # push all tags
```

---

# 15. Common Git Workflows Used by Teams

| Workflow | Description |
|---|---|
| **Feature Branch Workflow** | Each new feature/test gets its own branch, merged via PR |
| **GitFlow** | Structured workflow with `main`, `develop`, `release`, and `hotfix` branches — more complex, used in larger/traditional teams |
| **Trunk-Based Development** | Very short-lived branches, frequent merges to `main`, often paired with feature flags — common in fast-moving Agile/CI-CD teams |

> Most modern Agile teams (aligning with Step 7) favor **Feature Branch Workflow** or **Trunk-Based Development** for their simplicity and compatibility with continuous integration.

---

## Key Takeaways

- **Git** tracks code changes locally; **GitHub** hosts repositories remotely and enables team collaboration (PRs, code review, CI/CD).
- Core workflow: edit → `git add` (stage) → `git commit` (save snapshot) → `git push` (send to remote).
- **Branching** lets QA work on new tests without breaking the stable suite others rely on — use clear naming (`feature/`, `bugfix/`, `hotfix/`).
- **Pull Requests** are the real-world collaboration mechanism — include clear descriptions, link related Jira tickets, and go through code review before merging.
- QA-specific code review should check locator robustness, assertion quality, test independence, and negative/edge case coverage.
- **.gitignore** keeps generated artifacts (reports, screenshots) and secrets (`.env`) out of the repository.
- **Never commit credentials** — use environment variables, and properly rotate any accidentally-committed secrets (deleting isn't enough).
- `git revert` is safer than `git reset --hard` on shared branches, since it doesn't rewrite history others depend on.
- `git log`, `git blame`, and `git diff` are practical tools for investigating **when and why** a test started failing.
- Understanding Git well is a direct prerequisite for **CI/CD** (Step 17), since pipelines are triggered by and built around Git events (pushes, PRs).