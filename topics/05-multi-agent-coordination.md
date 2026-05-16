# Topic 5: Orchestrate Multi-Agent Coordination

## Key Concepts and Sub-Topics

- **Common patterns:**
  - **Planner → executor** — planner decomposes and proposes; executor implements approved steps
  - **Planner → executor → reviewer** — adds independent validation before any result is accepted
  - **Specialist swarm with coordinator** — parallel domain experts (security, docs, tests) synthesized by a coordinator
  - **Parallel research + centralized synthesis** — multiple read-heavy agents gather data; one agent writes the final output
  - **Human escalation gate** — any agent can pause and route a decision to a human

- **Shared context contracts** — explicit schema for what one agent passes to the next (PR comment template, structured artifact, workflow output)

- **Handoff boundaries and ownership** — each agent has a clear input, output, and scope; no two agents own the same file simultaneously

- **Conflict resolution and deduplication** — when agents produce overlapping or contradictory outputs, a synthesis step applies prioritization rules

- **Cost, latency, and failure-domain tradeoffs** — more agents mean more coordination overhead; parallelize only when the benefit justifies the complexity

---

## Common Pitfalls / Anti-Patterns

- Multiple agents editing the same files simultaneously without coordination.
- Undefined ownership between planner and executor (both try to change the same scope).
- No synthesis layer, causing contradictory outputs to reach humans or merge.
- Overusing multi-agent workflows when a single constrained agent is enough.

---

## Step-by-Step Tutorial

**Goal:** Decide when multiple agents help and how to coordinate them safely.

1. **Split a complex task** into research, implementation, validation, and synthesis phases. Identify which phases can run in parallel and which must serialize.

2. **Determine parallelism:**
   - Research (read-only) → safe to parallelize
   - Implementation (write) → serialize or partition by file/module
   - Validation (read-only after writes) → safe to parallelize
   - Synthesis (writes final output) → serialize

3. **Assign each agent a contract:**
   - Input format (issue body, PR diff, artifact path)
   - Output format (PR comment, structured JSON artifact, workflow summary)
   - Permissions (read-only, comment-only, branch-scoped write)
   - Stop conditions (token limit, diff size, missing required data)

4. **Define a shared handoff format.** Example: each specialist agent posts a structured PR comment with a severity-tagged findings list. The coordinator reads all comments and synthesizes a final report.

5. **Choose one accountable decision-maker** — typically a human or a gated review step — who approves or rejects the final synthesized output before merge or deploy.

---

## Orchestration Patterns Cheat Sheet

| Pattern | Best for | Strength | Weakness | Use when |
| --- | --- | --- | --- | --- |
| Single constrained agent | Small, bounded tasks | Lowest complexity | Limited specialization | One task, low ambiguity |
| Planner → executor | Moderate implementation | Clear separation of intent/action | More latency | You need explicit review of the plan |
| Planner → executor → reviewer | PR-quality work | Strong safety and quality | More coordination overhead | You need both execution and independent validation |
| Specialist swarm + coordinator | Broad analysis | Parallel coverage | Synthesis complexity | Researching many independent areas |
| Human-supervised loop | Sensitive or high-impact changes | Highest accountability | Slowest | Infra, security, compliance, production |

---

## Sample Prompts

### Planner prompt

```text
Read the linked issue and repository instructions. Produce a 5-step implementation plan.
Do not modify files. Identify required tests, risk areas, and human approval points.
```

### Executor prompt

```text
Implement only the approved plan steps. Use the smallest possible diff.
Do not change unrelated files. Stop if a required secret, permission, or policy decision is missing.
```

### Reviewer prompt

```text
Review the proposed diff for correctness, unnecessary scope, missing tests, policy violations, and unsafe actions.
Return findings grouped by severity with concrete remediation guidance.
```

---

## Practical Exercises

- Design a specialist swarm for a large PR involving documentation, tests, and security concerns. Define the contract for each specialist and the coordinator.
- Identify one scenario where multi-agent coordination is worse than a single constrained agent and explain the overhead cost.
- Write a coordinator checklist for resolving conflicting findings from two specialist agents.
- Map the planner → executor → reviewer pattern onto a GitHub Actions workflow using jobs and artifacts for handoff.

**What a strong answer includes:**
- Clear ownership and input/output contracts
- Controlled, explicit handoffs
- Serialized writes; parallelized reads
- One accountable decision-maker for merge or deploy

---

## Exam-Style Questions

**Q1.** Security, docs, and test review all need to happen quickly on a large PR.  
**A:** Use specialist agents in parallel for read-heavy analysis and a coordinator to synthesize findings before surfacing to a human. Parallel read-heavy work is an ideal multi-agent use case.

**Q2.** Two agents keep overwriting each other's code changes.  
**A:** Serialize write access or partition file ownership clearly. Parallel writes without coordination create conflicts and unreliability.

**Q3.** The planner proposes steps the executor cannot perform with current permissions.  
**A:** Validate the plan against available tools and permissions before execution. Feasibility checks belong at the handoff boundary.

**Q4.** A single small task is given to three agents "for safety."  
**A:** Prefer one constrained agent. Multi-agent overhead is not justified for low-complexity work; it adds latency and coordination risk without a proportional safety benefit.

**Q5.** Findings from multiple agents conflict.  
**A:** Route them through a synthesis or review stage with explicit prioritization rules. Coordination needs a decision owner, not just more outputs.

---

## Complete Working Example

### Full multi-job planner → executor → reviewer → coordinator workflow

This workflow serializes four agent roles using `needs:`, passes structured artifacts as handoffs, and posts a synthesized summary to the PR.

```yaml
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
          mkdir -p agent-artifacts
          cat > agent-artifacts/plan.json << 'PLAN'
          {
            "steps": [
              {"id": 1, "file": "src/greet.py", "action": "add_function"},
              {"id": 2, "file": "tests/test_greet.py", "action": "add_test"}
            ],
            "estimated_files_changed": 2,
            "human_approval_required": false
          }
          PLAN
          echo "artifact_name=plan-${{ github.run_id }}" >> "$GITHUB_OUTPUT"
          cat agent-artifacts/plan.json

      - uses: actions/upload-artifact@v4
        with:
          name: plan-${{ github.run_id }}
          path: agent-artifacts/plan.json
          retention-days: 7

# ── Job 2: Executor ──────────────────────────────────────────────────────────
  executor:
    name: "Agent: Executor"
    runs-on: ubuntu-latest
    needs: planner          # waits for planner to complete
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false

      - uses: actions/download-artifact@v4
        with:
          name: plan-${{ github.run_id }}
          path: agent-artifacts/

      - name: Validate plan scope before executing
        run: |
          FILES=$(python3 -c "
          import json
          plan = json.load(open('agent-artifacts/plan.json'))
          print(plan['estimated_files_changed'])
          ")
          echo "Plan calls for $FILES file(s)."
          [ "$FILES" -le 20 ] || { echo "::error::Plan exceeds 20-file limit."; exit 1; }

      - name: Simulate execution and record result
        run: |
          mkdir -p execution-output
          cat > execution-output/execution-result.json << 'EXEC'
          {
            "run_id": "${{ github.run_id }}",
            "steps_completed": [1, 2],
            "files_modified": ["src/greet.py", "tests/test_greet.py"],
            "status": "completed"
          }
          EXEC
          cat execution-output/execution-result.json

      - uses: actions/upload-artifact@v4
        with:
          name: execution-result-${{ github.run_id }}
          path: execution-output/execution-result.json
          retention-days: 7

# ── Job 3: Reviewer ──────────────────────────────────────────────────────────
  reviewer:
    name: "Agent: Reviewer"
    runs-on: ubuntu-latest
    needs: executor         # waits for executor to complete
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false

      - uses: actions/download-artifact@v4
        with:
          pattern: "*-${{ github.run_id }}"
          merge-multiple: true
          path: agent-artifacts/

      - name: Cross-check planned vs executed files
        run: |
          PLANNED=$(python3 -c "
          import json
          plan = json.load(open('agent-artifacts/plan.json'))
          print('\n'.join(sorted([s['file'] for s in plan['steps']])))
          ")
          EXECUTED=$(python3 -c "
          import json
          result = json.load(open('agent-artifacts/execution-result.json'))
          print('\n'.join(sorted(result['files_modified'])))
          ")
          [ "$PLANNED" = "$EXECUTED" ] || {
            echo "::error::Scope mismatch: planned='$PLANNED' executed='$EXECUTED'"; exit 1
          }
          echo "Review passed: executor stayed within planned scope."

      - name: Write review findings
        run: |
          mkdir -p review-output
          cat > review-output/review-findings.json << 'REVIEW'
          {
            "run_id": "${{ github.run_id }}",
            "scope_check": "pass",
            "safety_check": "pass",
            "recommendation": "approve"
          }
          REVIEW

      - uses: actions/upload-artifact@v4
        with:
          name: review-findings-${{ github.run_id }}
          path: review-output/review-findings.json
          retention-days: 7

# ── Job 4: Coordinator ───────────────────────────────────────────────────────
  coordinator:
    name: "Agent: Coordinator"
    runs-on: ubuntu-latest
    needs: reviewer         # waits for reviewer to complete
    if: github.event_name == 'pull_request'
    permissions:
      pull-requests: write
      contents: read
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false

      - uses: actions/download-artifact@v4
        with:
          pattern: "*-${{ github.run_id }}"
          merge-multiple: true
          path: agent-artifacts/

      - name: Post synthesized summary to PR
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          RECOMMENDATION=$(python3 -c "
          import json
          findings = json.load(open('agent-artifacts/review-findings.json'))
          print(findings['recommendation'])
          ")
          gh pr comment "${{ github.event.pull_request.number }}" \
            --body "## 🤖 Multi-Agent Pipeline Summary

| Agent | Status |
| --- | --- |
| Planner | ✅ Plan generated |
| Executor | ✅ Execution completed |
| Reviewer | ✅ Review passed |

**Recommendation:** ${RECOMMENDATION}

_Human approval still required before merge._" \
            --repo "${{ github.repository }}"
```

### Key patterns to remember

| Pattern | YAML construct | Purpose |
| --- | --- | --- |
| Serialize jobs | `needs: [previous-job]` | Enforce planner → executor ordering |
| Pass data between jobs | `upload-artifact` / `download-artifact` | Structured handoff contract |
| Job-level outputs | `outputs:` + `echo "key=value" >> $GITHUB_OUTPUT` | Lightweight scalar handoffs |
| Coordinator synthesis | Final job with `needs: [a, b]` after parallel jobs | Merge findings from specialist agents |

---

## Hands-On Lab

Follow [Lab 05 — Multi-Agent Coordination](../tutorials/05-multi-agent-lab.md) to build the full pipeline step-by-step in your own practice repository.

---

## Relevant Resources

- [GitHub Docs — Use GitHub Copilot agents](https://docs.github.com/en/copilot/how-tos/use-copilot-agents)
- [GitHub Docs — Workflow syntax (jobs, needs, outputs)](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [GitHub Docs — Storing workflow data as artifacts](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/storing-workflow-data-as-artifacts)
