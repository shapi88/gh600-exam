# Deep Dive 4: Evaluation, Error Analysis, and Tuning

> **Companion to:** [Topic 4 — Evaluation, Error Analysis, and Tuning](../topics/04-evaluation-error-analysis-and-tuning.md)

This file goes beyond the topic summary. It links every concept to the authoritative GitHub documentation section, adds implementation detail, and provides additional exercises for each sub-topic.

---

## 1. Workflow Re-Runs and Failure Isolation

### GitHub Docs reference
- [Re-running workflows and jobs](https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-workflow-runs/re-running-workflows-and-jobs)
- [Using the GitHub CLI in workflows](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/using-github-cli-in-workflows)

### Extended explanation
GitHub Actions supports three re-run modes. Knowing which mode to use is an evaluation skill — using the wrong mode either wastes compute or misses a failure class.

| Re-run mode | What re-runs | When to use |
| --- | --- | --- |
| Re-run all jobs | Every job from the start | Transient infrastructure failure that affected multiple jobs |
| Re-run failed jobs only | Failed jobs and their dependents | Partial failure — preserve successful job outputs/artifacts |
| Re-run a single job | One specific job | Targeted re-run during debugging without disturbing upstream |

**CLI commands:**
```bash
# Re-run all jobs in a workflow run
gh run rerun <run-id>

# Re-run only failed jobs
gh run rerun <run-id> --failed

# Re-run a specific job
gh run rerun <run-id> --job <job-id>
```

**Re-run behavior for evaluation:**
- Artifacts from successful jobs are preserved and available to re-run jobs that download them.
- Re-runs inherit the same `github.sha` and context — so test results are directly comparable to the first attempt.
- Each re-run increments `github.run_attempt`, allowing evaluation tooling to distinguish attempt 1 from attempt 2.

**Evaluation pattern — comparing run attempts:**
```bash
# Get status of each attempt for a given run
gh run view <run-id> --json attempts
```

Use `run_attempt` in artifact names (see Deep Dive 3) to keep results from each attempt separate and comparable.

### Additional exercise
Write a shell script that:
1. Lists the last 10 workflow runs for `eval-guardrails.yml`
2. For each run, prints: `run_id`, `status`, `run_attempt`, and `conclusion`
3. Calculates the pass rate across the 10 runs

Use: `gh run list --workflow eval-guardrails.yml --json databaseId,status,conclusion,attempt`.

---

## 2. Code Scanning and Secret Scanning Alert APIs as Quality Gates

### GitHub Docs reference
- [About code scanning](https://docs.github.com/en/code-security/code-scanning/introduction-to-code-scanning/about-code-scanning)
- [REST API — Code scanning](https://docs.github.com/en/rest/code-scanning)
- [About secret scanning](https://docs.github.com/en/code-security/secret-scanning/introduction/about-secret-scanning)
- [REST API — Secret scanning](https://docs.github.com/en/rest/secret-scanning)

### Extended explanation
Code scanning and secret scanning results can be used as programmatic quality gates in your evaluation workflow. Rather than only checking "did CI pass?", you can check "did this PR introduce new alerts?"

**Code scanning alert API — key operations:**

```bash
# List open code scanning alerts for a ref
gh api \
  "repos/{owner}/{repo}/code-scanning/alerts?ref=refs/heads/{branch}&state=open" \
  --jq '.[] | {rule_id: .rule.id, severity: .rule.severity, location: .most_recent_instance.location.path}'

# Count new alerts introduced by a PR
NEW_ALERTS=$(gh api \
  "repos/{owner}/{repo}/code-scanning/alerts?ref=$HEAD_SHA&state=open" \
  --jq 'length')
```

**Using code scanning as a PR quality gate in a workflow:**

```yaml
name: eval-code-scanning-gate

on:
  pull_request:
    branches: [main]

permissions:
  contents: read
  security-events: read
  pull-requests: write

jobs:
  code-scanning-gate:
    runs-on: ubuntu-latest
    steps:
      - name: Check for new high-severity alerts
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          ALERTS=$(gh api \
            "repos/${{ github.repository }}/code-scanning/alerts" \
            --field ref="${{ github.event.pull_request.head.sha }}" \
            --field state=open \
            --field severity=high \
            --jq 'length')

          echo "High-severity alerts on this PR: $ALERTS"

          if [ "$ALERTS" -gt 0 ]; then
            echo "::error::$ALERTS high-severity code scanning alert(s) introduced by this PR."
            exit 1
          fi
```

**Secret scanning push protection:** When push protection is enabled, GitHub blocks pushes that contain detected secrets *before* they reach the repository. This is the strongest safety control — it operates at the git layer before the agent's workflow even runs. Reference: [About push protection](https://docs.github.com/en/code-security/secret-scanning/protecting-pushes-with-secret-scanning).

### Additional exercise
Build an evaluation workflow that posts a code scanning summary to a PR comment in the following format:

```
## Security Gate Summary

| Severity | New alerts | Existing alerts |
| --- | --- | --- |
| Critical | 0 | 2 |
| High | 1 | 5 |
| Medium | 3 | 12 |

**Gate result:** ❌ FAIL — 1 new high-severity alert detected.
```

Use the code scanning alert API, `gh pr comment`, and `github-script` or shell scripting.

---

## 3. PR Check Run Status API as a Pass/Fail Rubric Signal

### GitHub Docs reference
- [REST API — Check runs](https://docs.github.com/en/rest/checks/runs)
- [REST API — Check suites](https://docs.github.com/en/rest/checks/suites)
- [About status checks](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/collaborating-on-repositories-with-code-quality-features/about-status-checks)

### Extended explanation
Check runs are the individual CI/CD jobs and checks that run against a specific commit SHA. They are the machine-readable form of the rubric — each dimension of your evaluation (scope, safety, correctness, auditability) maps to one check run.

**Listing check runs for a commit:**
```bash
# All check runs for a commit SHA
gh api repos/{owner}/{repo}/commits/{sha}/check-runs \
  --jq '.check_runs[] | {name: .name, status: .status, conclusion: .conclusion}'

# Check if all required checks have passed
FAILED=$(gh api repos/{owner}/{repo}/commits/{sha}/check-runs \
  --jq '[.check_runs[] | select(.conclusion == "failure" or .conclusion == "action_required")] | length')
```

**Creating a custom check run from a workflow (rubric as code):**

```yaml
- name: Create rubric check run
  uses: actions/github-script@v7
  with:
    script: |
      await github.rest.checks.create({
        owner: context.repo.owner,
        repo: context.repo.repo,
        name: 'Eval Rubric — Scope Control',
        head_sha: context.payload.pull_request.head.sha,
        status: 'completed',
        conclusion: 'success',   // or 'failure'
        output: {
          title: 'Scope Control: PASS',
          summary: '3 files changed — within 20-file limit.',
          text: '### Details\nFiles changed: src/greet.py, tests/test_greet.py, README.md'
        }
      });
```

**Connecting rubric dimensions to check run names:**

| Rubric dimension | Suggested check run name |
| --- | --- |
| Correctness | `eval/correctness` |
| Scope control | `eval/scope-control` |
| Safety | `eval/safety` |
| Auditability | `eval/auditability` |

Set each as a required status check in branch protection — then the PR cannot merge unless every dimension passes.

### Additional exercise
Write a workflow that reads `evals/rubric.md` to extract pass criteria, runs each criterion as a script assertion, and creates one check run per rubric dimension. Use `actions/github-script` for the check run creation and shell commands for the assertions.

---

## 4. GitHub Models Playground for Prompt Iteration

### GitHub Docs reference
- [About GitHub Models](https://docs.github.com/en/github-models/about-github-models)
- [Prototyping with AI models](https://docs.github.com/en/github-models/prototyping-with-ai-models)

### Extended explanation
GitHub Models provides a playground and API endpoint for running prompts against multiple models directly from GitHub. For evaluation and tuning, it enables:

1. **Side-by-side model comparison** — run the same scenario against GPT-4o, Claude, and Llama with one click.
2. **Prompt version history** — save and compare prompt versions in the playground.
3. **SDK-based evaluation scripts** — use the Azure AI Inference SDK (compatible with GitHub Models) to run eval scenarios programmatically.

**API-based eval using GitHub Models:**

```python
import json
from azure.ai.inference import ChatCompletionsClient
from azure.ai.inference.models import SystemMessage, UserMessage
from azure.core.credentials import AzureKeyCredential

client = ChatCompletionsClient(
    endpoint="https://models.inference.ai.azure.com",
    credential=AzureKeyCredential(GITHUB_TOKEN),
)

def run_scenario(scenario: dict, prompt_version: str) -> dict:
    response = client.complete(
        model="gpt-4o",
        messages=[
            SystemMessage(content=prompt_version),
            UserMessage(content=scenario["input"]),
        ],
        max_tokens=2048,
    )
    return {
        "scenario_id": scenario["id"],
        "output": response.choices[0].message.content,
        "finish_reason": response.choices[0].finish_reason,
    }

# Load scenarios and run eval
with open("evals/scenarios.md") as f:
    # parse scenarios (simplified)
    scenarios = [{"id": 1, "input": "Fix the typo in README.md"}]

results = [run_scenario(s, PROMPT_V2) for s in scenarios]
print(json.dumps(results, indent=2))
```

**Key limits for GitHub Models (as of 2025):** Rate limits vary by model tier. Free tier has lower limits suitable for eval prototyping; paid tiers suit production-scale evals. Reference the [GitHub Models rate limits page](https://docs.github.com/en/github-models/prototyping-with-ai-models#rate-limits).

### Additional exercise
Build an eval workflow that:
1. Reads all 5 scenarios from `evals/scenarios.md`
2. Runs each scenario against prompt version 1 and version 2 using the GitHub Models API
3. Scores each result against `evals/rubric.md` criteria
4. Posts a comparison table to the PR as a comment

```markdown
## Prompt Version Comparison

| Scenario | v1 Correctness | v1 Scope | v2 Correctness | v2 Scope |
| --- | --- | --- | --- | --- |
| 1. Fix typo | ✅ | ✅ | ✅ | ✅ |
| 2. Refactor auth | ❌ | ❌ | ✅ | ✅ |
...
```

---

## 5. Regression Testing for Agent Behavior

### GitHub Docs reference
- [About required status checks](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/collaborating-on-repositories-with-code-quality-features/about-status-checks#types-of-status-checks-on-github)
- [Workflow syntax — `schedule`](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onschedule)

### Extended explanation
Prompt regressions are sneaky — fixing one failure mode often re-introduces a previously resolved one. A regression test suite for agent behavior runs the full scenario set on a schedule and fails the workflow if any previously-passing scenario now fails.

**Scheduled regression workflow:**

```yaml
name: agent-regression-tests

on:
  schedule:
    - cron: '0 6 * * 1-5'   # weekdays at 06:00 UTC
  workflow_dispatch:

permissions:
  contents: read
  issues: write   # create regression issue if tests fail

jobs:
  regression:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false

      - name: Run eval scenarios
        run: python3 scripts/run_evals.py --scenarios evals/scenarios.md --rubric evals/rubric.md

      - name: File regression issue on failure
        if: failure()
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          gh issue create \
            --title "Agent regression detected — $(date +%Y-%m-%d)" \
            --body "The nightly eval run failed. Check the Actions log for details." \
            --label "regression,agent"
```

**Tracking pass rates over time:** Store eval results as artifacts (one per run) and use the GitHub API to download and compare them across runs. This gives you a longitudinal view of agent quality without a separate database.

---

## Relevant Resources

| Resource | URL |
| --- | --- |
| Re-running workflows and jobs | https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-workflow-runs/re-running-workflows-and-jobs |
| About code scanning | https://docs.github.com/en/code-security/code-scanning/introduction-to-code-scanning/about-code-scanning |
| REST API — Code scanning | https://docs.github.com/en/rest/code-scanning |
| About secret scanning | https://docs.github.com/en/code-security/secret-scanning/introduction/about-secret-scanning |
| About push protection | https://docs.github.com/en/code-security/secret-scanning/protecting-pushes-with-secret-scanning |
| REST API — Check runs | https://docs.github.com/en/rest/checks/runs |
| About status checks | https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/collaborating-on-repositories-with-code-quality-features/about-status-checks |
| About GitHub Models | https://docs.github.com/en/github-models/about-github-models |
| Prototyping with AI models | https://docs.github.com/en/github-models/prototyping-with-ai-models |
| Workflow syntax — schedule | https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#onschedule |
