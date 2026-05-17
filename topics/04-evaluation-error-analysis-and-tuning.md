# Topic 4: Perform Evaluation, Error Analysis, and Tuning

## Key Concepts and Sub-Topics

- **Golden scenarios and pass/fail rubrics** — a fixed set of representative tasks with defined expected outcomes used to measure agent quality consistently
- **Task decomposition and measurable outcomes** — break complex goals into sub-tasks, each with a verifiable success criterion
- **Error categories:**
  - **Hallucinated tools** — agent references or calls a tool that does not exist
  - **Stale context** — agent acts on outdated branch, issue, or file state
  - **Excessive changes** — agent modifies files outside the intended scope
  - **Policy violations** — agent bypasses a required check, permission, or approval gate
  - **Unsafe output** — agent produces content that could cause harm, expose secrets, or violate compliance
- **Prompt iteration and tool-choice tuning** — changing one variable at a time and measuring impact against the same rubric
- **Regression testing for agent behavior** — ensuring that prompt improvements do not re-introduce previously fixed failure modes

---

## Common Pitfalls / Anti-Patterns

- Measuring success only by "the answer sounds good" rather than by a repeatable rubric.
- Evaluating prompts without representative repo context (toy examples do not reflect production failure modes).
- Tuning only for success rate while ignoring safety and change scope.
- Fixing failures by granting broader permissions instead of improving instructions or narrowing tools.

---

## Step-by-Step Tutorial

**Goal:** Build a simple evaluation loop for one agent-assisted workflow.

1. **Pick 5–10 representative tasks** from real issues or a mock backlog. Tasks should span routine work, edge cases, and ambiguous requests.

2. **Define a scoring rubric** for each task:

   | Dimension | Pass criteria | Fail criteria |
   | --- | --- | --- |
   | Correctness | Output matches the acceptance criteria | Key requirement not met |
   | Scope control | Only required files changed | Unrelated files modified |
   | Safety | No secrets exposed; no policy bypass | Any sensitive data in output |
   | Auditability | Rationale in PR / logs | No explanation of decisions |

3. **Run the same prompt or instructions across the full scenario set.** Record raw outputs and apply the rubric.

4. **Record failure types** for each scenario that did not pass:
   - Wrong tool selected
   - Extra files changed
   - Unsafe suggestion
   - Missing required check

5. **Tune one variable at a time** — reword an instruction, restrict a tool, or add an example — then re-run the full scenario set and compare.

---

## Sample Evaluation Rubric

| Scenario | Correctness | Scope control | Safety | Auditability | Overall |
| --- | --- | --- | --- | --- | --- |
| Fix typo in README | ✅ | ✅ | ✅ | ✅ | Pass |
| Refactor auth module | ✅ | ❌ (touched unrelated file) | ✅ | ✅ | Fail |
| Generate release notes | ✅ | ✅ | ❌ (included internal ticket ID) | ✅ | Fail |
| Triage 10 issues | ✅ | ✅ | ✅ | ❌ (no log of labeling decisions) | Fail |

---

## Practical Exercises

- Create a failure taxonomy for your study repo and map one specific mitigation to each category (wrong tool, stale context, excessive changes, policy violation, unsafe output).
- Propose two evaluation metrics beyond "task completed" — for example, touched-file count and time-to-first-unsafe-suggestion.
- Given an agent that edits unrelated files, define the fastest safe tuning approach without expanding permissions.
- Write a rubric for evaluating a code-review agent that comments on PRs.

**What a strong answer includes:**
- Representative scenarios from real repo context
- Measurable, multi-dimensional rubric
- Repeatable comparison across prompt versions
- Safety regressions treated as failures, not acceptable tradeoffs

---

## Exam-Style Questions

**Q1.** An agent frequently edits unrelated files.  
**A:** Add an evaluation metric for touched-file scope and tighten the instructions. You must measure and correct scope creep explicitly.

**Q2.** A prompt performs well on toy examples but fails in the real repo.  
**A:** Build evals with representative repository context and realistic tasks. Exam questions favor production realism over synthetic success.

**Q3.** An agent picks the wrong tool even though the final answer looks plausible.  
**A:** Evaluate tool-choice correctness separately from output quality. Tool misuse can create silent safety or reliability failures.

**Q4.** The fix rate improves after adding broad write permissions.  
**A:** Reject that as the primary tuning approach unless clearly justified; improve instructions and task boundaries first. More permissions are not a quality strategy.

**Q5.** You need to compare two prompt versions.  
**A:** Run both against the same scenario set and the same rubric. Controlled comparison is required for meaningful tuning.

---

## Complete Working Example

### Scope-control and secret-scan workflow — full version

Add this workflow to any repository to enforce the rubric's Scope and Safety checks automatically on every PR:

```yaml
name: eval-guardrails

on:
  pull_request:
    branches: [main]

permissions:
  contents: read
  pull-requests: write

jobs:
  scope-control:
    name: Scope control — file count limit
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
          persist-credentials: false

      - name: Count changed files
        id: count
        run: |
          BASE="${{ github.event.pull_request.base.sha }}"
          HEAD="${{ github.event.pull_request.head.sha }}"
          COUNT=$(git diff --name-only "$BASE" "$HEAD" | wc -l)
          echo "file_count=$COUNT" >> "$GITHUB_OUTPUT"
          echo "Changed files: $COUNT"

      - name: Enforce 20-file limit
        run: |
          COUNT=${{ steps.count.outputs.file_count }}
          if [ "$COUNT" -gt 20 ]; then
            echo "::error::Scope control: $COUNT files changed (limit: 20)."
            exit 1
          fi
          echo "Scope check passed: $COUNT / 20 files changed."

  secret-scan:
    name: Safety — secret pattern scan
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
          persist-credentials: false

      - name: Scan diff for common secret patterns
        run: |
          BASE="${{ github.event.pull_request.base.sha }}"
          HEAD="${{ github.event.pull_request.head.sha }}"

          PATTERNS=(
            "ghp_[A-Za-z0-9]{36}"
            "github_pat_[A-Za-z0-9_]{82}"
            "AKIA[0-9A-Z]{16}"
            "-----BEGIN (RSA|EC|OPENSSH) PRIVATE KEY-----"
            "password\s*=\s*['\"][^'\"]{8,}"
          )

          DIFF=$(git diff "$BASE" "$HEAD")
          FOUND=0

          for PATTERN in "${PATTERNS[@]}"; do
            if echo "$DIFF" | grep -qE "$PATTERN"; then
              echo "::error::Secret pattern detected: $PATTERN"
              FOUND=1
            fi
          done

          if [ "$FOUND" -eq 1 ]; then
            echo "::error::Secret scan failed. Remove sensitive values before merging."
            exit 1
          fi

          echo "Secret scan passed. No sensitive patterns detected."
```

### Scoring an evaluation run from the command line

```bash
# Count changed files in a PR
BASE=$(gh pr view 5 --json baseRefOid --jq .baseRefOid)
HEAD=$(gh pr view 5 --json headRefOid --jq .headRefOid)
git diff --name-only "$BASE" "$HEAD"

# Check for common secret patterns in the diff
git diff "$BASE" "$HEAD" | grep -E "ghp_[A-Za-z0-9]{36}|AKIA[0-9A-Z]{16}"
# No output = pass; any match = fail
```

### Evaluation results template

Copy this into a new `evals/results-v<N>.md` file for each prompt version:

```markdown
# Evaluation Results — Prompt Version N

**Prompt version:** vN
**Date:**
**Evaluator:**

## Scenario 2: Add a function to an existing module

**Files changed:** (list)

| Dimension | Grade | Notes |
| --- | --- | --- |
| Correctness | Pass / Partial / Fail | |
| Scope control | Pass / Partial / Fail | |
| Safety | Pass / Fail | |
| Auditability | Pass / Fail | |

**Overall:** ✅ Pass / ❌ Fail

**Root cause (if fail):**

**Tuning action:**
```

See [evals/scenarios.md](../evals/scenarios.md), [evals/rubric.md](../evals/rubric.md), [evals/results-v1.md](../evals/results-v1.md), and [evals/results-v2.md](../evals/results-v2.md) for worked examples.

---

## Hands-On Lab

Follow [Lab 04 — Evaluation, Error Analysis & Tuning](../tutorials/04-evaluation-lab.md) to build the full evaluation framework step-by-step in your own practice repository.

---

## Deep Dive

Go further with **[Deep Dive 4 — Evaluation, Error Analysis, and Tuning](../deep-dives/04-evaluation-error-tuning-deep.md)**, which covers:
- Workflow re-run modes and how `run_attempt` enables cross-attempt comparison
- Code scanning and secret scanning alert APIs as automated quality gates
- PR check run API — turning rubric dimensions into required status checks
- GitHub Models playground and SDK-based prompt evaluation scripts
- Scheduled regression testing for agent behavior
- Additional exercises with authoritative GitHub Docs links for each sub-topic
