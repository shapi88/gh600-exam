# Lab 05: Multi-Agent Coordination — Hands-On Setup

## Goal

Build a GitHub Actions workflow that implements the planner → executor → reviewer pattern using serialized jobs, structured artifact handoffs, and a coordinator that posts a synthesized PR comment.

---

## Prerequisites

Complete [Lab 04](04-evaluation-lab.md). Confirm you are on the `main` branch.

```bash
cd gh600-practice
git checkout main && git pull
```

---

## Step 1 — Create the multi-job planner → executor → reviewer workflow

Each job represents one agent role. `needs:` enforces serialization. Artifacts carry the handoff contract between jobs.

```bash
cat > .github/workflows/multi-agent-pipeline.yml << 'EOF'
name: multi-agent-pipeline

on:
  workflow_dispatch:
    inputs:
      task_description:
        description: "Describe the task for the planner agent"
        required: true
        default: "Add a farewell function to src/greet.py and write a test"
  pull_request:
    branches: [main]

permissions:
  contents: read
  pull-requests: write

# ── Job 1: Planner ───────────────────────────────────────────────────────────
jobs:
  planner:
    name: "Agent: Planner"
    runs-on: ubuntu-latest
    permissions:
      contents: read
    outputs:
      plan_artifact: ${{ steps.plan.outputs.artifact_name }}
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false

      - name: Generate implementation plan
        id: plan
        run: |
          ARTIFACT_NAME="plan-${{ github.run_id }}"
          mkdir -p agent-artifacts

          # In a real agent workflow, the LLM call would happen here.
          # This step simulates the planner output with a structured JSON plan.
          cat > agent-artifacts/plan.json << 'PLAN'
          {
            "task": "${{ github.event.inputs.task_description || 'Add a farewell function to src/greet.py and write a test' }}",
            "steps": [
              {
                "id": 1,
                "action": "add_function",
                "file": "src/greet.py",
                "description": "Add farewell(name) that returns 'Goodbye, <name>!'"
              },
              {
                "id": 2,
                "action": "add_test",
                "file": "tests/test_greet.py",
                "description": "Add test_farewell_happy_path asserting output"
              }
            ],
            "risk_areas": ["none identified"],
            "human_approval_required": false,
            "estimated_files_changed": 2
          }
          PLAN

          echo "artifact_name=$ARTIFACT_NAME" >> "$GITHUB_OUTPUT"
          echo "Plan generated:"
          cat agent-artifacts/plan.json

      - name: Upload plan artifact
        uses: actions/upload-artifact@v4
        with:
          name: plan-${{ github.run_id }}
          path: agent-artifacts/plan.json
          retention-days: 7

# ── Job 2: Executor ──────────────────────────────────────────────────────────
  executor:
    name: "Agent: Executor"
    runs-on: ubuntu-latest
    needs: planner
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false

      - name: Download plan artifact
        uses: actions/download-artifact@v4
        with:
          name: plan-${{ github.run_id }}
          path: agent-artifacts/

      - name: Read plan and simulate execution
        run: |
          echo "Executor reading plan:"
          cat agent-artifacts/plan.json

          # Validate the plan before executing
          FILES=$(cat agent-artifacts/plan.json | python3 -c "
          import json,sys
          plan = json.load(sys.stdin)
          count = plan['estimated_files_changed']
          print(count)
          ")
          echo "Plan calls for $FILES file(s) to be changed."

          if [ "$FILES" -gt 20 ]; then
            echo "::error::Plan exceeds 20-file scope limit. Stopping execution."
            exit 1
          fi

          echo "Plan validated. Execution would proceed here."

          mkdir -p execution-output
          cat > execution-output/execution-result.json << 'EXEC'
          {
            "run_id": "${{ github.run_id }}",
            "steps_completed": [1, 2],
            "files_modified": ["src/greet.py", "tests/test_greet.py"],
            "status": "completed",
            "diff_lines_added": 8,
            "diff_lines_removed": 0
          }
          EXEC
          echo "Execution result:"
          cat execution-output/execution-result.json

      - name: Upload execution result artifact
        uses: actions/upload-artifact@v4
        with:
          name: execution-result-${{ github.run_id }}
          path: execution-output/execution-result.json
          retention-days: 7

# ── Job 3: Reviewer ──────────────────────────────────────────────────────────
  reviewer:
    name: "Agent: Reviewer"
    runs-on: ubuntu-latest
    needs: executor
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false

      - name: Download plan and execution artifacts
        uses: actions/download-artifact@v4
        with:
          pattern: "*-${{ github.run_id }}"
          merge-multiple: true
          path: agent-artifacts/

      - name: Review execution result
        run: |
          echo "Reviewer reading plan:"
          cat agent-artifacts/plan.json

          echo "Reviewer reading execution result:"
          cat agent-artifacts/execution-result.json

          # Cross-check: verify the executor changed only the planned files
          PLANNED=$(cat agent-artifacts/plan.json | python3 -c "
          import json,sys
          plan = json.load(sys.stdin)
          planned = sorted([s['file'] for s in plan['steps']])
          print('\n'.join(planned))
          ")

          EXECUTED=$(cat agent-artifacts/execution-result.json | python3 -c "
          import json,sys
          result = json.load(sys.stdin)
          executed = sorted(result['files_modified'])
          print('\n'.join(executed))
          ")

          if [ "$PLANNED" != "$EXECUTED" ]; then
            echo "::error::Reviewer: file scope mismatch. Planned: $PLANNED. Executed: $EXECUTED."
            exit 1
          fi

          echo "Review passed: executor stayed within planned scope."

          mkdir -p review-output
          cat > review-output/review-findings.json << 'REVIEW'
          {
            "run_id": "${{ github.run_id }}",
            "scope_check": "pass",
            "safety_check": "pass",
            "correctness_check": "pass",
            "findings": [],
            "recommendation": "approve"
          }
          REVIEW
          echo "Review findings:"
          cat review-output/review-findings.json

      - name: Upload review findings artifact
        uses: actions/upload-artifact@v4
        with:
          name: review-findings-${{ github.run_id }}
          path: review-output/review-findings.json
          retention-days: 7

# ── Job 4: Coordinator ───────────────────────────────────────────────────────
  coordinator:
    name: "Agent: Coordinator"
    runs-on: ubuntu-latest
    needs: reviewer
    if: github.event_name == 'pull_request'
    permissions:
      pull-requests: write
      contents: read
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false

      - name: Download all artifacts
        uses: actions/download-artifact@v4
        with:
          pattern: "*-${{ github.run_id }}"
          merge-multiple: true
          path: agent-artifacts/

      - name: Post synthesized summary to PR
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          PR_NUMBER: ${{ github.event.pull_request.number }}
        run: |
          RECOMMENDATION=$(cat agent-artifacts/review-findings.json | python3 -c "
          import json,sys
          findings = json.load(sys.stdin)
          print(findings['recommendation'])
          ")

          SCOPE=$(cat agent-artifacts/review-findings.json | python3 -c "
          import json,sys
          findings = json.load(sys.stdin)
          print(findings['scope_check'])
          ")

          gh pr comment "$PR_NUMBER" \
            --body "## 🤖 Multi-Agent Pipeline Summary (Run \`${{ github.run_id }}\`)

| Agent | Status |
| --- | --- |
| Planner | ✅ Plan generated |
| Executor | ✅ Execution completed |
| Reviewer | ✅ Review passed |

**Scope check:** ${SCOPE}
**Recommendation:** ${RECOMMENDATION}

_This comment was posted by the coordinator job. A human reviewer must still approve the PR before merge._" \
            --repo "${{ github.repository }}"
EOF

git add .github/workflows/multi-agent-pipeline.yml
git commit -m "Add multi-agent pipeline workflow with planner/executor/reviewer/coordinator (Lab 05)"
git push origin main
```

---

## Step 2 — Trigger the workflow manually and observe the job graph

```bash
gh workflow run multi-agent-pipeline.yml \
  --field task_description="Add a farewell function to src/greet.py and write a test"

echo "Waiting for run to start..."
sleep 10
gh run list --workflow multi-agent-pipeline.yml --limit 3
```

Open the run URL to see the four jobs in sequence: **Planner → Executor → Reviewer → Coordinator**. Notice that each job waits for the previous one to complete before starting.

```bash
RUN_ID=$(gh run list --workflow multi-agent-pipeline.yml --limit 1 --json databaseId --jq '.[0].databaseId')
gh run watch "$RUN_ID"
```

---

## Step 3 — Download and inspect the artifact handoff chain

```bash
RUN_ID=$(gh run list --workflow multi-agent-pipeline.yml --limit 1 --json databaseId --jq '.[0].databaseId')
mkdir -p /tmp/agent-artifacts
gh run download "$RUN_ID" --dir /tmp/agent-artifacts

echo "=== Plan ==="
cat /tmp/agent-artifacts/plan-*/plan.json 2>/dev/null || find /tmp/agent-artifacts -name "plan.json" -exec cat {} \;

echo ""
echo "=== Execution Result ==="
find /tmp/agent-artifacts -name "execution-result.json" -exec cat {} \;

echo ""
echo "=== Review Findings ==="
find /tmp/agent-artifacts -name "review-findings.json" -exec cat {} \;
```

---

## Step 4 — Demonstrate the file-ownership conflict problem and the fix

This step shows why parallel writes must be serialized or partitioned.

```bash
# Create a branch that demonstrates two jobs writing to the same file
cat > /tmp/conflict-demo.yml << 'EOF'
name: conflict-demo

on:
  workflow_dispatch:

permissions:
  contents: read

jobs:
  agent-a:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false
      - name: Agent A writes to shared file
        run: |
          echo "Change from Agent A" > shared-output.txt
          echo "Agent A finished writing."
          cat shared-output.txt

  agent-b:
    # Problem: runs in parallel with agent-a, both writing to shared-output.txt
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false
      - name: Agent B also writes to shared file
        run: |
          echo "Change from Agent B" > shared-output.txt
          echo "Agent B finished writing."
          cat shared-output.txt
EOF
echo "Without 'needs:', agent-a and agent-b run in parallel and overwrite each other."
echo ""

# Fixed version: serialize with needs, or partition the files
cat > /tmp/conflict-fixed.yml << 'EOF'
name: conflict-fixed

on:
  workflow_dispatch:

permissions:
  contents: read

jobs:
  agent-a:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false
      - name: Agent A writes to its own file (partitioned ownership)
        run: |
          echo "Change from Agent A" > output-agent-a.txt
          cat output-agent-a.txt
      - uses: actions/upload-artifact@v4
        with:
          name: output-a
          path: output-agent-a.txt

  agent-b:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false
      - name: Agent B writes to its own file (partitioned ownership)
        run: |
          echo "Change from Agent B" > output-agent-b.txt
          cat output-agent-b.txt
      - uses: actions/upload-artifact@v4
        with:
          name: output-b
          path: output-agent-b.txt

  coordinator:
    runs-on: ubuntu-latest
    needs: [agent-a, agent-b]   # serialized synthesis after both complete
    steps:
      - uses: actions/download-artifact@v4
        with:
          merge-multiple: true
          path: combined/
      - name: Synthesize outputs
        run: |
          echo "=== Combined output ==="
          cat combined/output-agent-a.txt
          cat combined/output-agent-b.txt
EOF
echo "Fix: each agent writes to its own file. The coordinator synthesizes after both finish."
```

> **Exam point:** Parallel writes to the same file create non-deterministic conflicts. The fix is either serialization with `needs:` or partitioning file ownership so each agent writes only to its own output file.

---

## Step 5 — Test the PR coordinator comment

```bash
git checkout -b test/lab05-multiagent
echo "# Lab 05 test" > lab05-test.md
git add lab05-test.md
git commit -m "Lab 05: test multi-agent pipeline PR comment"
git push origin test/lab05-multiagent

gh pr create \
  --title "Lab 05: multi-agent pipeline test" \
  --body "Testing the coordinator PR comment." \
  --base main
```

The `multi-agent-pipeline.yml` workflow triggers on the PR, and after the pipeline completes the coordinator job posts a summary comment on the PR.

```bash
PR_NUMBER=$(gh pr list --head test/lab05-multiagent --json number --jq '.[0].number')
gh pr view "$PR_NUMBER" --comments
```

---

## Expected outcome

After completing this lab your repository will have:

| Artefact | Status |
| --- | --- |
| `multi-agent-pipeline.yml` with Planner, Executor, Reviewer, Coordinator jobs | ✅ Committed |
| Artifact handoffs: `plan.json` → `execution-result.json` → `review-findings.json` | ✅ Demonstrated |
| Coordinator posts a synthesized comment to the PR | ✅ Demonstrated |
| Conflict-demo scripts showing the problem and the fix | ✅ Documented |

---

## Verify it works

```bash
# Confirm workflow file exists
cat .github/workflows/multi-agent-pipeline.yml | grep "name:"

# Confirm jobs are serialized correctly
grep "needs:" .github/workflows/multi-agent-pipeline.yml

# List recent pipeline runs
gh run list --workflow multi-agent-pipeline.yml --limit 5
```

---

## Teardown

```bash
git checkout main
git branch -d test/lab05-multiagent 2>/dev/null || true
```

---

## Next step

Proceed to [Lab 06 — Guardrails & Accountability](06-guardrails-lab.md).
