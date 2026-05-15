# Lab 01: Agent Architecture & SDLC — Hands-On Setup

## Goal

Configure a GitHub repository so that an agent can only operate within safe, governed boundaries: a protected `main` branch, a `CODEOWNERS` file, and an explicit `issue → branch → PR` flow with required human review.

---

## Prerequisites

Complete [Lab 00 — Prerequisites](00-prerequisites.md) and confirm:

```bash
gh auth status        # logged in
cd gh600-practice     # inside your practice repo
```

---

## Step 1 — Protect the `main` branch via GitHub UI

Branch protection ensures no direct pushes to `main` — not even from an agent.

1. Open `https://github.com/<your-username>/gh600-practice`.
2. Click **Settings** → **Branches**.
3. Under **Branch protection rules**, click **Add rule** (or **Add branch protection rule**).
4. Set:
   - **Branch name pattern:** `main`
   - ✅ **Require a pull request before merging**
     - ✅ **Require approvals** — set to **1**
   - ✅ **Require status checks to pass before merging**
     - Search and add: `lint` (you will create this in Step 4)
   - ✅ **Do not allow bypassing the above settings**
5. Click **Create** (or **Save changes**).

**Equivalent via `gh` CLI (after the UI step creates the initial rule):**

```bash
gh api repos/{owner}/{repo}/branches/main/protection \
  --method PUT \
  --field required_status_checks='{"strict":true,"contexts":["lint"]}' \
  --field enforce_admins=true \
  --field required_pull_request_reviews='{"required_approving_review_count":1}' \
  --field restrictions=null
# Replace {owner} and {repo} with your values, e.g. myusername/gh600-practice
```

---

## Step 2 — Add a `CODEOWNERS` file

`CODEOWNERS` routes reviews for sensitive paths to designated humans (or teams) automatically.

```bash
mkdir -p .github
cat > .github/CODEOWNERS << 'EOF'
# All workflow files require review from the repository owner
/.github/workflows/ @<your-username>

# All YAML files require review from the repository owner
*.yml @<your-username>
*.yaml @<your-username>
EOF
```

Replace `@<your-username>` with your actual GitHub username (e.g., `@myusername`).

Commit the file:

```bash
git add .github/CODEOWNERS
git commit -m "Add CODEOWNERS to gate workflow and YAML file changes"
git push origin main
```

> **Why this matters for agents:** Even if an agent has write access, editing a CODEOWNERS-protected path automatically requests a human review on the PR. The agent cannot merge unilaterally.

---

## Step 3 — Create a sample issue with acceptance criteria

A well-structured issue is the agent's starting artifact.

```bash
gh issue create \
  --title "Add a greeting function to the codebase" \
  --body "## Acceptance criteria

- [ ] A function named \`greet(name)\` is added to \`src/greet.py\`
- [ ] The function returns \`Hello, <name>!\`
- [ ] A unit test covers the happy path and an empty-string edge case

## Out of scope
- CLI wiring
- Internationalization

## Risk note
Low risk. No external dependencies or state changes." \
  --label "enhancement"
```

Expected output:

```
https://github.com/<your-username>/gh600-practice/issues/1
```

Note the issue number — you will reference it in the next step.

---

## Step 4 — Add a minimal status-check workflow

Branch protection requires at least one passing status check. Create a minimal `lint` job so the protection rule can be satisfied.

```bash
mkdir -p .github/workflows
cat > .github/workflows/ci.yml << 'EOF'
name: ci

on:
  pull_request:
    branches: [main]

permissions:
  contents: read

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false
      - name: Check for trailing whitespace
        run: |
          if grep -rn " $" --include="*.py" --include="*.md" .; then
            echo "Trailing whitespace found."
            exit 1
          fi
          echo "No trailing whitespace found."
EOF
```

Commit directly to `main` (this is the one time a direct push is acceptable — before the protection rule is enforced):

```bash
git add .github/workflows/ci.yml
git commit -m "Add minimal CI workflow for branch protection status check"
git push origin main
```

---

## Step 5 — Demonstrate the `issue → branch → PR` flow

This is the governed delivery flow an agent operates within.

```bash
# 1. Create a feature branch linked to the issue
git checkout -b feature/greet-function-issue-1

# 2. Create the source file
mkdir -p src
cat > src/greet.py << 'EOF'
def greet(name: str) -> str:
    """Return a greeting string for the given name."""
    return f"Hello, {name}!"
EOF

# 3. Create a unit test
mkdir -p tests
cat > tests/test_greet.py << 'EOF'
from src.greet import greet

def test_greet_happy_path():
    assert greet("Alice") == "Hello, Alice!"

def test_greet_empty_string():
    assert greet("") == "Hello, !"
EOF

# 4. Commit the work
git add src/greet.py tests/test_greet.py
git commit -m "Add greet function and unit tests (closes #1)"

# 5. Push the branch
git push origin feature/greet-function-issue-1
```

Open a **draft PR** to signal the work is in progress (not ready for review):

```bash
gh pr create \
  --title "Add greet function (closes #1)" \
  --body "## Summary
Implements \`greet(name)\` in \`src/greet.py\` as specified in issue #1.

## Checklist
- [x] Function added
- [x] Unit tests added
- [ ] Human review complete

## Agent rationale
Used the narrowest possible change: one new file in \`src/\` and one test file. No existing files modified." \
  --draft \
  --base main
```

Expected output:

```
https://github.com/<your-username>/gh600-practice/pull/2
```

---

## Step 6 — Mark the PR ready and verify the protection gates

```bash
# Mark as ready for review
gh pr ready 2

# Attempt to merge without approval — this should fail
gh pr merge 2 --squash
# Expected error:
# X Pull request #2 is not mergeable: required review from CODEOWNERS not satisfied
# X Required status checks have not passed
```

The error confirms both gates are active.

---

## Expected outcome

After completing this lab your repository will have:

| Control | Status |
| --- | --- |
| `main` branch protection — required PR | ✅ Active |
| `main` branch protection — required review | ✅ Active |
| `main` branch protection — required status check (`lint`) | ✅ Active |
| `CODEOWNERS` routing `.github/` and `*.yml` | ✅ Active |
| Sample issue with acceptance criteria | ✅ Created |
| Draft PR linked to the issue | ✅ Opened |

---

## Verify it works

```bash
# Confirm branch protection is in place
gh api repos/{owner}/{repo}/branches/main/protection --jq '.required_pull_request_reviews,.required_status_checks'

# Confirm CODEOWNERS file exists
cat .github/CODEOWNERS

# Confirm the CI workflow is present
cat .github/workflows/ci.yml

# List open PRs
gh pr list
```

---

## Teardown

To reset the practice repo for the next lab, leave the protection rules in place — they are required by later labs. To remove the feature branch:

```bash
git checkout main
git branch -d feature/greet-function-issue-1
gh pr close 2 --delete-branch
```

---

## Next step

Proceed to [Lab 02 — Tool Use & Environment Interaction](02-tool-use-lab.md).
