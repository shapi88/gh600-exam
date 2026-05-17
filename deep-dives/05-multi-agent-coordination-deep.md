# Deep Dive 5: Multi-Agent Coordination

> **Companion to:** [Topic 5 — Multi-Agent Coordination](../topics/05-multi-agent-coordination.md)

This file goes beyond the topic summary. It links every concept to the authoritative GitHub documentation section, adds implementation detail, and provides additional exercises for each sub-topic.

---

## 1. Reusable Workflows as Handoff Contract Primitives

### GitHub Docs reference
- [Reusing workflows](https://docs.github.com/en/actions/sharing-automations/reusing-workflows)
- [Workflow syntax — `workflow_call`](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onworkflow_call)
- [Passing inputs and outputs between reusable workflows](https://docs.github.com/en/actions/sharing-automations/reusing-workflows#using-outputs-from-a-reusable-workflow)

### Extended explanation
`workflow_call` turns a workflow file into a reusable function with typed inputs and outputs. For multi-agent coordination, this creates a **versioned, testable handoff contract** between agents — better than ad-hoc artifact conventions.

**Key properties of `workflow_call`:**
- The called workflow runs in the caller's repository context (same secrets, same token).
- Inputs are typed (`string`, `boolean`, `number`, `choice`). Type errors are caught before execution.
- Outputs can pass scalar values from the called workflow back to the caller.
- The called workflow can be in a different repository, enabling shared agent libraries across an organization.

**Example — planner as a reusable workflow:**

```yaml
# .github/workflows/agent-planner.yml
name: "Agent: Planner"

on:
  workflow_call:
    inputs:
      issue_number:
        required: true
        type: number
      max_files:
        required: false
        type: number
        default: 20
    outputs:
      plan_artifact_name:
        description: "Name of the artifact containing the implementation plan"
        value: ${{ jobs.plan.outputs.artifact_name }}
      estimated_files:
        description: "Number of files the plan expects to change"
        value: ${{ jobs.plan.outputs.estimated_files }}

jobs:
  plan:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      issues: read
    outputs:
      artifact_name: ${{ steps.upload.outputs.artifact-id }}
      estimated_files: ${{ steps.generate.outputs.estimated_files }}
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false

      - name: Read issue and generate plan
        id: generate
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          ISSUE=$(gh issue view ${{ inputs.issue_number }} --json body,title)
          # Agent logic: produce plan.json from issue context
          mkdir -p artifacts
          echo "{\"steps\": [], \"estimated_files\": 3}" > artifacts/plan.json
          echo "estimated_files=3" >> "$GITHUB_OUTPUT"

      - name: Upload plan
        id: upload
        uses: actions/upload-artifact@v4
        with:
          name: plan-${{ github.run_id }}-${{ github.run_attempt }}
          path: artifacts/plan.json
          retention-days: 7
```

**Calling the planner from an orchestrator workflow:**

```yaml
# .github/workflows/orchestrate.yml
name: Agent Orchestrator

on:
  issues:
    types: [assigned]

jobs:
  call-planner:
    uses: ./.github/workflows/agent-planner.yml
    with:
      issue_number: ${{ github.event.issue.number }}
      max_files: 20

  call-executor:
    needs: call-planner
    uses: ./.github/workflows/agent-executor.yml
    with:
      plan_artifact_name: ${{ needs.call-planner.outputs.plan_artifact_name }}
      estimated_files: ${{ needs.call-planner.outputs.estimated_files }}
```

### Additional exercise
Convert the inline four-job pipeline from Topic 5's Complete Working Example into a set of four reusable workflow files:
- `.github/workflows/agent-planner.yml`
- `.github/workflows/agent-executor.yml`
- `.github/workflows/agent-reviewer.yml`
- `.github/workflows/agent-coordinator.yml`
- `.github/workflows/orchestrate.yml` (the caller)

Define the typed `inputs` and `outputs` for each. Verify that the coordinator can access the reviewer's `recommendation` output.

---

## 2. `workflow_dispatch` Inputs for Structured Agent Triggers

### GitHub Docs reference
- [Workflow syntax — `workflow_dispatch`](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onworkflow_dispatch)
- [Manually running a workflow](https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-workflow-runs/manually-running-a-workflow)
- [Triggering a workflow](https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/triggering-a-workflow)

### Extended explanation
`workflow_dispatch` with typed, validated inputs is the safest way to trigger a multi-agent pipeline manually or from another orchestration system. It provides an explicit, auditable trigger with a structured payload.

**Input types and validation:**

```yaml
on:
  workflow_dispatch:
    inputs:
      task_type:
        description: "Type of agent task"
        required: true
        type: choice
        options:
          - implement-issue
          - review-pr
          - triage-backlog
          default: implement-issue

      issue_number:
        description: "Issue number to work on (required for implement-issue)"
        required: false
        type: number

      pr_number:
        description: "PR number to review (required for review-pr)"
        required: false
        type: number

      dry_run:
        description: "Simulate actions without writing to repo"
        required: false
        type: boolean
        default: false
```

**Triggering via CLI with input validation:**
```bash
# Trigger with structured inputs
gh workflow run orchestrate.yml \
  --field task_type=implement-issue \
  --field issue_number=42 \
  --field dry_run=false

# List recent dispatch runs
gh run list --workflow orchestrate.yml --event workflow_dispatch
```

**Triggering from another workflow (cross-workflow orchestration):**
```yaml
- name: Trigger downstream agent
  env:
    GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  run: |
    gh workflow run agent-executor.yml \
      --field plan_artifact="${{ steps.plan.outputs.artifact_name }}" \
      --field issue_number="${{ inputs.issue_number }}"
```

**Audit value:** Every `workflow_dispatch` trigger is recorded in the Actions audit log with the actor, inputs, and timestamp — giving you full provenance for why an agent was triggered.

---

## 3. PR Review API — Programmatic Approval and Request-Changes

### GitHub Docs reference
- [REST API — Pull request reviews](https://docs.github.com/en/rest/pulls/reviews)
- [REST API — Pull request review comments](https://docs.github.com/en/rest/pulls/comments)
- [About pull request reviews](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/about-pull-request-reviews)

### Extended explanation
In a multi-agent pipeline, programmatic reviews are the handoff signal between the reviewer agent and the human/merge gate. A reviewer agent posting `REQUEST_CHANGES` blocks merge until addressed; posting `APPROVE` (from an identity that satisfies branch protection) signals readiness.

**Review submission events:**

| Event | Meaning | When agent uses it |
| --- | --- | --- |
| `COMMENT` | General feedback; no block | Soft signal — findings noted but not blocking |
| `REQUEST_CHANGES` | Blocks merge until dismissed or re-approved | Agent found policy violation or scope creep |
| `APPROVE` | Signals readiness for merge | Agent passed all checks — still requires human approval |
| `PENDING` | Review draft (not submitted) | Do not use in automated flows |

**Submitting a review from a workflow:**

```yaml
- name: Submit reviewer agent findings
  uses: actions/github-script@v7
  with:
    script: |
      const findings = JSON.parse(
        require('fs').readFileSync('artifacts/review-findings.json', 'utf8')
      );

      const event = findings.recommendation === 'approve'
        ? 'COMMENT'    // never auto-approve; use COMMENT + human approval gate
        : 'REQUEST_CHANGES';

      const body = findings.recommendation === 'approve'
        ? `## ✅ Reviewer Agent: All checks passed\n\nThis PR is ready for human review.`
        : `## ❌ Reviewer Agent: Issues found\n\n${findings.details}`;

      await github.rest.pulls.createReview({
        owner: context.repo.owner,
        repo: context.repo.repo,
        pull_number: context.payload.pull_request.number,
        event,
        body,
      });
```

**Important:** An agent identity should use `COMMENT` rather than `APPROVE` unless the organization has explicitly configured agents as valid reviewers for branch protection. Automatic approval by agents without additional human review is a governance anti-pattern.

**Dismissing a stale agent review:**
```bash
# If the agent review is stale after new commits, dismiss it
gh api repos/{owner}/{repo}/pulls/{pr}/reviews/{review_id}/dismissals \
  --method PUT \
  --field message="New commits pushed — re-review required."
```

### Additional exercise
Design the complete review flow for a three-agent pipeline:
1. Reviewer agent posts `REQUEST_CHANGES` with a structured list of findings
2. Executor agent addresses each finding and re-pushes commits
3. Reviewer agent re-runs and posts `COMMENT` (not `APPROVE`) when findings are resolved
4. Human reviewer approves and merges

Write the API calls and workflow `if:` conditions that implement this loop.

---

## 4. GitHub Actions Environments as Coordination Gates

### GitHub Docs reference
- [Using environments for deployment](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment)
- [Reviewing deployments](https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-workflow-runs/reviewing-deployments)
- [Environment protection rules](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment#environment-protection-rules)

### Extended explanation
Environments add a mandatory human approval gate between agents with different trust levels. The key insight: a job that targets an `environment` **pauses** until the required reviewers approve via the GitHub UI or API.

**Environment protection rules:**

| Rule | Purpose | Multi-agent use |
| --- | --- | --- |
| Required reviewers | Named humans must approve before the job runs | Gate between executor agent and deployment |
| Wait timer | Mandatory delay (0–43,200 minutes) | Buffer time for security review |
| Deployment branches | Only specific branches can deploy | Prevent agents from deploying from arbitrary branches |
| Custom deployment protection rules | GitHub App-based approval | Third-party policy engines as gates |

**Three-environment coordination pattern for agent pipelines:**

```yaml
jobs:
  plan:
    runs-on: ubuntu-latest
    # No environment — planner runs immediately
    ...

  review-gate:
    needs: plan
    runs-on: ubuntu-latest
    environment: agent-review   # requires human approval
    steps:
      - name: Human confirmed plan is safe
        run: echo "Plan approved. Proceeding to execution."

  execute:
    needs: review-gate
    runs-on: ubuntu-latest
    environment: agent-execute   # optionally add wait timer
    ...

  deploy-gate:
    needs: execute
    runs-on: ubuntu-latest
    environment: production      # requires named production approvers
    ...
```

**Setting up environments via CLI:**
```bash
# Create environments (requires API)
gh api repos/{owner}/{repo}/environments/agent-review \
  --method PUT \
  --field wait_timer=0 \
  --field reviewers='[{"type":"User","id":<user_id>}]'
```

### Additional exercise
Design an environment strategy for a five-agent pipeline that processes a PR touching both source code and `.github/workflows/` files. Requirements:
1. Workflow file changes must be approved by the security team before execution
2. Production deployments require two named approvers and a 10-minute wait timer
3. The reviewer agent can run without approval gates but cannot trigger deployments

Map each agent to an environment (or no environment) and list the protection rules for each.

---

## 5. Conflict Resolution Between Agents — Synthesis Patterns

### GitHub Docs reference
- [REST API — Pull request review comments](https://docs.github.com/en/rest/pulls/comments)
- [About required status checks](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/collaborating-on-repositories-with-code-quality-features/about-status-checks)

### Extended explanation
When multiple specialist agents post findings on the same PR, conflicts arise: the security agent flags a dependency, but the compatibility agent says it is required. A coordinator agent must apply prioritization rules rather than simply aggregating all findings.

**Priority rules for conflict resolution:**

| Conflict type | Resolution rule |
| --- | --- |
| Safety vs. correctness | Safety always wins — block merge, file issue |
| Scope findings from two agents disagree | Take the more restrictive (smaller scope) recommendation |
| One agent approves, another requests changes | `REQUEST_CHANGES` wins — PR stays blocked |
| Coverage gap (agents have complementary findings) | Merge both finding lists without deduplication |
| Duplicate findings (same issue flagged twice) | Deduplicate by rule ID or location hash before posting |

**Deduplication pattern:**
```python
import json, hashlib

def deduplicate_findings(finding_lists: list[list[dict]]) -> list[dict]:
    seen = set()
    merged = []
    for findings in finding_lists:
        for f in findings:
            # Create a stable key from rule + location
            key = hashlib.sha256(
                json.dumps({"rule": f["rule_id"], "path": f["path"], "line": f.get("line")},
                           sort_keys=True).encode()
            ).hexdigest()
            if key not in seen:
                seen.add(key)
                merged.append(f)
    return merged
```

**Coordinator synthesis workflow snippet:**
```yaml
- name: Synthesize findings from all specialists
  run: |
    python3 - << 'EOF'
    import json, os

    security = json.load(open("artifacts/security-findings.json"))
    docs     = json.load(open("artifacts/docs-findings.json"))
    tests    = json.load(open("artifacts/test-findings.json"))

    all_findings = security["findings"] + docs["findings"] + tests["findings"]
    blocking     = [f for f in all_findings if f["severity"] in ("critical","high")]
    advisory     = [f for f in all_findings if f["severity"] not in ("critical","high")]

    summary = {
        "total": len(all_findings),
        "blocking": len(blocking),
        "recommendation": "REQUEST_CHANGES" if blocking else "COMMENT",
    }
    json.dump(summary, open("artifacts/coordinator-summary.json", "w"), indent=2)
    EOF
```

---

## Relevant Resources

| Resource | URL |
| --- | --- |
| Reusing workflows | https://docs.github.com/en/actions/sharing-automations/reusing-workflows |
| Workflow syntax — workflow_call | https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onworkflow_call |
| Workflow syntax — workflow_dispatch | https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onworkflow_dispatch |
| Manually running a workflow | https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-workflow-runs/manually-running-a-workflow |
| REST API — Pull request reviews | https://docs.github.com/en/rest/pulls/reviews |
| About pull request reviews | https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/about-pull-request-reviews |
| Using environments for deployment | https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment |
| Reviewing deployments | https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-workflow-runs/reviewing-deployments |
