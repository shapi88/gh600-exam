# Lab 03: Memory, State & Execution — Hands-On Setup

## Goal

Implement the three key durability patterns for agent runs: repository-scoped instructions (durable memory), a PR-description checklist as an explicit checkpoint, and a GitHub Actions artifact as a machine-readable run output. Also demonstrate a retry-safe issue-creation pattern and the stale-context risk.

---

## Prerequisites

Complete [Lab 02](02-tool-use-lab.md). Confirm you are on the `main` branch with the CI workflow from previous labs in place.

```bash
cd gh600-practice
git checkout main && git pull
```

---

## Step 1 — Create `.github/copilot-instructions.md` (durable repository memory)

Repository memory stores conventions that persist across runs and inform every agent session.

```bash
cat > .github/copilot-instructions.md << 'EOF'
# Repository: gh600-practice
# Copilot / Agent Instructions

## Coding conventions
- All Python files use 4-space indentation and type annotations.
- Test files live in `tests/` and are prefixed with `test_`.
- All functions must have a one-line docstring.

## Workflow rules
- Always propose a short plan before making code changes.
- Use the smallest possible diff that fully solves the task.
- Never bypass required tests, reviews, or environment approvals.
- Prefer read-only inspection before any write action.

## Stop conditions
- Stop and request human review before editing any file listed in CODEOWNERS.
- Stop if the diff touches `.github/workflows/`.
- Stop if a required secret or credential is missing.
- Stop after 3 consecutive failed retries and post a summary of what failed.

## Sensitive information
- Never store tokens, passwords, or PII in memory or in any committed file.
- If a task requires a secret, read it from `secrets.*` at runtime only.
EOF

git add .github/copilot-instructions.md
git commit -m "Add copilot-instructions.md with durable repo conventions (Lab 03)"
git push origin main
```

> **Why this is durable memory:** Unlike in-prompt context, `copilot-instructions.md` is committed to the repo and survives session resets. Every agent run in this repo has access to these conventions automatically.

---

## Step 2 — Use a PR description checklist as a checkpoint

A checklist in the PR description acts as an explicit, human-readable checkpoint log that any agent or human can inspect.

```bash
# Create a feature branch to simulate a multi-step agent run
git checkout -b feature/multi-step-task

# Simulate step 1: create a docs file
mkdir -p docs
echo "# Architecture overview" > docs/architecture.md
git add docs/architecture.md
git commit -m "Step 1: add architecture doc"

# Push and open a draft PR with an explicit checklist
git push origin feature/multi-step-task

gh pr create \
  --title "Multi-step task: add docs and tests" \
  --body "## Task checklist (agent checkpoint log)

- [x] Step 1: Create \`docs/architecture.md\`
- [ ] Step 2: Add a usage example to \`docs/architecture.md\`
- [ ] Step 3: Add integration test
- [ ] Step 4: Human review

**Last checkpoint:** Step 1 complete. Resuming from Step 2." \
  --draft \
  --base main
```

Note the PR number from the output (e.g., `3`).

**Simulate resuming from Step 2:**

```bash
# Step 2: update the doc
cat >> docs/architecture.md << 'EOF'

## Usage example

```python
from src.greet import greet
print(greet("World"))  # Hello, World!
```
EOF

git add docs/architecture.md
git commit -m "Step 2: add usage example to architecture doc"
git push origin feature/multi-step-task

# Update the PR description to reflect progress
PR_NUMBER=$(gh pr list --head feature/multi-step-task --json number --jq '.[0].number')
gh pr edit "$PR_NUMBER" --body "## Task checklist (agent checkpoint log)

- [x] Step 1: Create \`docs/architecture.md\`
- [x] Step 2: Add a usage example to \`docs/architecture.md\`
- [ ] Step 3: Add integration test
- [ ] Step 4: Human review

**Last checkpoint:** Step 2 complete. Resuming from Step 3."
```

> **Recovery pattern:** If the agent crashes, any future run (or human) can open the PR, read the checklist, and know exactly where to resume — without re-running destructive steps.

---

## Step 3 — Upload a run artifact with GitHub Actions

Artifacts are the durable machine output for a run: diffs, reports, evaluation results.

```bash
cat > .github/workflows/artifact-demo.yml << 'EOF'
name: artifact-demo

on:
  workflow_dispatch:
  pull_request:
    branches: [main]

permissions:
  contents: read

jobs:
  generate-report:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false

      - name: Generate a simple run report
        run: |
          mkdir -p reports
          cat > reports/run-report.json << 'REPORT'
          {
            "run_id": "${{ github.run_id }}",
            "sha": "${{ github.sha }}",
            "files_changed": 3,
            "status": "completed",
            "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
          }
          REPORT
          echo "Report generated:"
          cat reports/run-report.json

      - name: Upload run report as artifact
        uses: actions/upload-artifact@v4
        with:
          name: run-report-${{ github.run_id }}
          path: reports/
          retention-days: 14
EOF

git add .github/workflows/artifact-demo.yml
git commit -m "Add artifact-demo workflow (Lab 03)"
git push origin main

# Trigger the workflow
gh workflow run artifact-demo.yml
echo "Waiting for run to start..."
sleep 15
gh run list --workflow artifact-demo.yml --limit 3
```

After the run completes, download the artifact:

```bash
RUN_ID=$(gh run list --workflow artifact-demo.yml --limit 1 --json databaseId --jq '.[0].databaseId')
gh run download "$RUN_ID" --dir /tmp/artifacts
cat "/tmp/artifacts/run-report-${RUN_ID}/run-report.json"
```

---

## Step 4 — Write a retry-safe issue-creation snippet

Running an agent twice without idempotency checks can create duplicate issues. This pattern prevents that.

```bash
cat > /tmp/idempotent-issue.sh << 'EOF'
#!/usr/bin/env bash
# idempotent-issue.sh — creates an issue only if one with the same title doesn't exist.

set -euo pipefail

TITLE="Automated: weekly dependency audit"
REPO="${GITHUB_REPOSITORY:-$(gh repo view --json nameWithOwner --jq .nameWithOwner)}"

echo "Checking for existing issue: '$TITLE' in $REPO ..."

EXISTING=$(gh issue list \
  --repo "$REPO" \
  --state open \
  --search "$TITLE" \
  --json number,title \
  --jq ".[] | select(.title == \"$TITLE\") | .number")

if [ -n "$EXISTING" ]; then
  echo "Issue already exists: #$EXISTING — skipping creation."
  exit 0
fi

echo "No existing issue found. Creating ..."
gh issue create \
  --repo "$REPO" \
  --title "$TITLE" \
  --body "This issue was created by the automated weekly audit workflow.
  
**Run:** $GITHUB_RUN_ID
**Timestamp:** $(date -u +%Y-%m-%dT%H:%M:%SZ)" \
  --label "maintenance"
echo "Issue created successfully."
EOF

chmod +x /tmp/idempotent-issue.sh
# Run it twice — the second run should skip creation
bash /tmp/idempotent-issue.sh
bash /tmp/idempotent-issue.sh
```

Expected output on second run:

```
Checking for existing issue: 'Automated: weekly dependency audit' in myusername/gh600-practice ...
Issue already exists: #4 — skipping creation.
```

---

## Step 5 — Demonstrate the stale-context risk

Stale context occurs when an agent's prompt references an outdated file state.

```bash
# Simulate the agent reading the current greet.py content into its context
AGENT_CONTEXT=$(cat src/greet.py)
echo "Agent context captured at $(date):"
echo "$AGENT_CONTEXT"

# Now, another engineer modifies the file on the branch
cat > src/greet.py << 'EOF'
def greet(name: str, language: str = "en") -> str:
    """Return a localised greeting string for the given name."""
    greetings = {"en": "Hello", "es": "Hola", "fr": "Bonjour"}
    prefix = greetings.get(language, "Hello")
    return f"{prefix}, {name}!"
EOF

git add src/greet.py
git commit -m "Add language parameter to greet() — agent context is now stale"
git push origin feature/multi-step-task

echo ""
echo "⚠ The agent's cached context is now stale:"
echo "$AGENT_CONTEXT"
echo ""
echo "Current file content:"
cat src/greet.py
echo ""
echo "Mitigation: always re-read file contents from the branch HEAD, not from cached prompt context."
```

> **Exam point:** Agents must refresh file content from the current branch HEAD before generating diffs. Stale context causes correctness failures that are hard to detect because the output looks plausible.

---

## Expected outcome

After completing this lab your repository will have:

| Artefact | Status |
| --- | --- |
| `.github/copilot-instructions.md` with durable conventions and stop conditions | ✅ Committed |
| Draft PR with a step-by-step checkpoint checklist | ✅ Open |
| `artifact-demo` workflow that uploads a JSON run report | ✅ Committed |
| Idempotent issue-creation script (local, not committed) | ✅ Demonstrated |
| Stale-context demonstration on `feature/multi-step-task` | ✅ Demonstrated |

---

## Verify it works

```bash
# Confirm copilot-instructions.md exists
cat .github/copilot-instructions.md | head -10

# Confirm the artifact workflow ran
gh run list --workflow artifact-demo.yml --limit 3

# Confirm the PR checklist exists
PR_NUMBER=$(gh pr list --head feature/multi-step-task --json number --jq '.[0].number')
gh pr view "$PR_NUMBER" --json body --jq .body
```

---

## Teardown

```bash
git checkout main
git branch -d feature/multi-step-task 2>/dev/null || true
```

---

## Next step

Proceed to [Lab 04 — Evaluation, Error Analysis & Tuning](04-evaluation-lab.md).
