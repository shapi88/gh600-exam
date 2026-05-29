# GH-600 Exam Use Cases — Version 1

**File version:** v1  
**Date:** 2026-05-29  
**Topics covered:** Agent Architecture & SDLC, Tool Use & Environment Interaction, Memory State & Execution, Multi-Agent Coordination, Guardrails & Accountability  
**Format:** Scenario-based case studies — each use case presents a realistic problem narrative followed by a full worked solution.

> **How to use these use cases:** Read the Problem section first and try to identify what went wrong and how you would fix it before reading the Solution.

---

## Use Case 1 — Agent Architecture & SDLC: Runaway Autonomy

**Key concepts:** `topics/01-agent-architecture-and-sdlc.md` · Autonomy levels · Role separation · Branch protection · CODEOWNERS

---

### Problem

A startup's platform team wants to speed up routine refactors by connecting a GitHub Copilot agent to their monorepo. They configure the agent with:

- A `copilot-instructions.md` that describes the refactor rules (Autonomy Level 1 behavior).
- A workflow that grants the agent `contents: write` on all branches **including `main`** and calls `gh pr merge --auto` after CI passes.

During a weekend, the agent processes a large rename refactor. CI passes in 4 minutes. The auto-merge fires before any engineer has a chance to review. The refactor renames an internal RPC symbol that a downstream service depends on, causing a production outage that takes 3 hours to recover.

**Key symptoms to spot:**
1. The agent was operating at Autonomy Level 1 (planning + drafting) in its instructions, yet had Level 3 execution rights (direct repo writes + auto-merge) in its workflow.
2. No human-in-the-loop checkpoint existed between CI passing and the merge landing on `main`.
3. CODEOWNERS did not assign reviewers for the affected source paths, so no automatic review request was triggered.

---

### Solution

**Step 1 — Correct the autonomy level mismatch.**  
Remove `gh pr merge --auto` from the workflow. The agent's role is to *propose* changes, not to merge them. A Copilot agent operating at Autonomy Level 1 should open a **draft PR** only; a human promotes it to ready-for-review when satisfied.


**Step 2 — Restrict branch permissions.**  
Change the workflow `permissions:` block so `contents: write` applies only to the agent's working branch (`copilot/*`), not to `main`. Use branch protection rules on `main` to require at least one human approving review before merge.

```yaml
# Correct permissions block — least privilege for a Level 1 agent
permissions:
  contents: write        # agent branch only (enforced by branch protection on main)
  pull-requests: write   # open and comment on PRs
```

**Step 3 — Add CODEOWNERS for high-impact paths.**  
Create or update `CODEOWNERS` to assign the platform team as required reviewers for any files the agent is likely to touch during refactors:

```
# CODEOWNERS excerpt
/src/rpc/           @platform-team
/src/internal/      @platform-team
```

With CODEOWNERS in place, any PR touching those paths automatically requests a human review, blocking auto-merge even if CI passes.

**Step 4 — Add a branch protection rule.**  
In repository **Settings → Branches → Add rule** for `main`:
- Enable **Require a pull request before merging**.
- Set **Required approvals** to at least 1.
- Enable **Require review from Code Owners**.
- Enable **Dismiss stale pull request approvals when new commits are pushed**.

**Root cause summary:** The team conflated an Autonomy Level 1 configuration (instructions-only) with Autonomy Level 3 execution rights (direct merge). Platform-level controls — branch protection and CODEOWNERS — are the mandatory enforcement layer; prompt instructions alone are advisory and cannot substitute for them.

---

## Use Case 2 — Tool Use & Environment Interaction: Token Overpermission

**Key concepts:** `topics/02-tool-use-and-environment-interaction.md` · Least privilege · `permissions:` block · `persist-credentials: false` · Blast radius

---

### Problem

A team adds a Copilot agent workflow to automate issue triage, changelog generation, and deployment status comments. To avoid having to think about permissions, the workflow lead sets:

```yaml
permissions: write-all
```

Later, the agent is extended to also create tags. During that extension, a developer stores the workflow token in a step output so it can be reused in a later job:

```yaml
- name: Export token
  id: token_export
  run: echo "tok=${{ secrets.GITHUB_TOKEN }}" >> $GITHUB_OUTPUT
```

A security audit flags both issues as critical violations.

**Key symptoms to spot:**
1. `write-all` grants the agent write access to deployments, packages, security events, and more — far beyond what issue triage or changelog generation requires.
2. Echoing `GITHUB_TOKEN` into a step output exposes it to any subsequent step or job that reads the output, and may leak it into logs if `set -x` or debug mode is active.
3. If the agent is compromised or produces unexpected output, `write-all` maximises the blast radius of any mistake.

---

### Solution

**Step 1 — Replace `write-all` with explicit, minimal permissions.**  
Audit each job to determine exactly what API calls it makes, then grant only those scopes:

```yaml
# Before (violation)
permissions: write-all

# After (compliant)
permissions:
  contents: write        # commit changelog
  issues: write          # apply labels, post comments
  pull-requests: write   # post triage comments on PRs
  # deployments, packages, security-events: NOT granted (not needed)
```

**Step 2 — Remove the token export step entirely.**  
The `GITHUB_TOKEN` is available in every step automatically via `${{ secrets.GITHUB_TOKEN }}`. There is never a legitimate reason to echo it into a step output.

```yaml
# Remove this step completely:
# - name: Export token
#   id: token_export
#   run: echo "tok=${{ secrets.GITHUB_TOKEN }}" >> $GITHUB_OUTPUT
```

**Step 3 — Use `persist-credentials: false` on checkout steps.**  
If the workflow uses `actions/checkout`, prevent the checkout token from persisting into subsequent steps:

```yaml
- uses: actions/checkout@v4
  with:
    persist-credentials: false
```

**Step 4 — Understand blast radius.**  
With `write-all`, a misbehaving agent could: delete packages, trigger deployments, modify GitHub Actions secrets (via the secrets API), or alter security alert state. Scoped permissions contain any such failure to only the resources the agent legitimately needs.

**Root cause summary:** `write-all` is a convenience shortcut that violates the principle of least privilege. Every agent workflow must enumerate its required scopes explicitly. Tokens must never be echoed or forwarded — the runtime injects them where needed.

---

## Use Case 3 — Memory, State & Execution: Non-Idempotent Changelog Update

**Key concepts:** `topics/03-memory-state-and-execution.md` · Idempotency · Artifact-based checkpointing · Durable state

---

### Problem

A release automation agent runs on every push to a release branch. Its job is to append a new entry to `CHANGELOG.md` summarising the commits since the last tag. The relevant workflow step is:

```bash
# Step: append-changelog (simplified)
git log $LAST_TAG..HEAD --oneline >> CHANGELOG.md
git commit -am "chore: update changelog for $RELEASE_TAG"
git push
```

A flaky test in a downstream job causes the overall workflow run to be re-triggered twice in the same hour. Because `LAST_TAG` is still the same git tag, the agent appends the identical changelog block a second time, producing duplicate entries. Reviewers must manually clean up `CHANGELOG.md` before the release ships.

**Key symptoms to spot:**
1. The step has no guard checking whether the changelog entry for `$RELEASE_TAG` already exists.
2. On retry the same commits are in scope (`LAST_TAG` hasn't moved), so the step appends the same block again.
3. The workflow does not use artifact-based checkpointing; there is no record of whether the changelog commit was already made.

---

### Solution

**Option A — Idempotency guard (preferred for simple cases).**  
Before appending, check whether the release entry already exists in `CHANGELOG.md`:

```bash
# Before (non-idempotent)
git log $LAST_TAG..HEAD --oneline >> CHANGELOG.md

# After (idempotent guard added)
if grep -qF "## $RELEASE_TAG" CHANGELOG.md; then
  echo "Changelog entry for $RELEASE_TAG already present — skipping."
  exit 0
fi
git log $LAST_TAG..HEAD --oneline | \
  sed "1s/^/## $RELEASE_TAG\n/" >> CHANGELOG.md
```

The guard exits early when the entry is already present, making repeated runs safe.

**Option B — Artifact-based checkpointing (preferred for multi-step pipelines).**  
1. Generate the changelog diff in memory and upload it as a workflow artifact keyed by the release tag:
   ```yaml
   - name: Generate changelog diff
     run: git log $LAST_TAG..HEAD --oneline > /tmp/changelog-diff.txt
   - uses: actions/upload-artifact@v4
     with:
       name: changelog-diff-${{ env.RELEASE_TAG }}
       path: /tmp/changelog-diff.txt
   ```
2. A separate "commit" job downloads the artifact and commits only if the artifact exists and the tag entry is absent from `CHANGELOG.md`. Artifact names are unique per release tag, so re-runs simply re-download the same file without duplicating work.

**Step 3 — Always key checkpoints on an immutable identifier.**  
Use `RELEASE_TAG` (or a commit SHA) — not a timestamp or run number — as the key. This ensures that re-runs of the same logical operation converge to the same result.

**Root cause summary:** Any agent step that writes to a shared file must be idempotent: running it twice must produce the same result as running it once. The simplest fix is an existence check before writing; for complex multi-job pipelines, artifact-based checkpointing with an immutable key is the recommended pattern.

---

## Use Case 4 — Multi-Agent Coordination: Conflicting Specialist Proposals

**Key concepts:** `topics/05-multi-agent-coordination.md` · Planner–executor pattern · Artifact handoff · Conflict detection · Environment gates

---

### Problem

A team uses a planner–executor–reviewer pipeline. The planner spawns two specialist agents in parallel:

- **Specialist A** is tasked with updating authentication logic in `src/auth.py`.
- **Specialist B** is tasked with updating the API documentation in `docs/api.md`.

Both specialists independently notice that `src/config.py` contains a constant they want to change. Specialist A renames `AUTH_TIMEOUT` to `SESSION_TIMEOUT`; Specialist B changes the same constant's value. Each uploads a JSON artifact describing their proposed file changes.

The coordinator job downloads both artifacts and applies them sequentially. It applies Specialist A's diff first, then Specialist B's diff. Specialist B's diff fails to apply cleanly (wrong line numbers after A's rename), leaving `src/config.py` in a broken state. The executor commits the partial result and opens a PR. CI fails; engineers spend two hours diagnosing the conflict.

**Key symptoms to spot:**
1. The coordinator applied diffs without checking for file overlap between specialist proposals.
2. No conflict escalation path existed — the coordinator assumed sequential application would always succeed.
3. The executor was not gated on a human approval environment, so the broken commit landed in a PR automatically.

---

### Solution

**Step 1 — Add a conflict-detection step to the coordinator job.**  
Before applying any diff, collect all file paths touched by each specialist and check for intersections:

```python
# Pseudocode — coordinator conflict check
files_a = set(artifact_a["changed_files"])
files_b = set(artifact_b["changed_files"])
conflicts = files_a & files_b

if conflicts:
    post_pr_comment(
        f"⚠️ Conflict detected: specialists propose changes to the same files: {conflicts}. "
        "Human review required before execution."
    )
    sys.exit(1)  # Fail the coordinator job; executor is blocked
```

**Step 2 — Extend the artifact schema to include a `conflict_files` field.**  
Update `templates/artifact-schema.json` (or your equivalent schema) to allow the coordinator to record which files are in conflict:

```json
{
  "task_id": "string",
  "changed_files": ["array of file paths"],
  "conflict_files": ["array of file paths that overlap with another specialist"],
  "requires_human_approval": true,
  "status": "conflict | ready | approved"
}
```

Setting `requires_human_approval: true` and `status: conflict` signals the executor job to halt.

**Step 3 — Gate the executor on the `agent-execution` environment.**  
In the executor workflow job:

```yaml
executor:
  needs: coordinator
  environment: agent-execution   # requires a named human to approve
  if: needs.coordinator.outputs.status == 'ready'
```

With the `agent-execution` environment configured to require a human reviewer, the executor cannot run until a person approves — even if the coordinator job exits cleanly.

**Step 4 — Post a human-readable escalation comment on the PR.**  
The coordinator should post a structured comment listing the conflicting files, the specialists involved, and the next step required from the reviewer:

```
⚠️ Multi-agent conflict — Human Review Required

Specialists A and B both propose changes to:
- src/config.py

Please review both proposals, resolve the conflict manually, and approve the
`agent-execution` environment to allow the executor to proceed.
```

**Root cause summary:** Parallel specialist agents operating on overlapping files produce merge conflicts that automated coordinators cannot safely resolve. The coordinator must detect overlaps before applying diffs, escalate to a human reviewer, and block the executor via an environment gate until the conflict is resolved.

---

## Use Case 5 — Guardrails & Accountability: Silent Workflow Edit

**Key concepts:** `topics/06-guardrails-and-accountability.md` · Sensitive-path hard-stop · CODEOWNERS · Branch protection · Audit trail

---

### Problem

An agent is reviewing a PR that includes a one-character typo fix in `.github/workflows/deploy.yml` — the word "Deploying" was misspelled as "Deploiyng" in a `run: echo` step. The agent determines the change is low-risk (it only affects a log message) and posts an `approve` recommendation. The PR author merges immediately, bypassing any further review.

Two weeks later a security auditor runs a supply-chain review. They discover:

1. The workflow file was modified without a named human approving the PR — the only approval came from the agent.
2. No CODEOWNERS entry existed for `/.github/workflows/`, so the agent's approval counted as a required review.
3. The `copilot-instructions.md` listed `.github/workflows/` as a sensitive path, but the agent evaluated the change as "trivial" and proceeded anyway.

The auditor flags this as a critical governance failure: a workflow file was changed without human oversight.

**Key symptoms to spot:**
1. The agent evaluated risk subjectively ("the change is trivial") instead of applying the hard-stop rule unconditionally.
2. No platform-level control (CODEOWNERS or branch protection) prevented the agent's approval from satisfying the merge requirement.
3. The audit trail shows only an agent approval — there is no human attribution for the merge decision.

---

### Solution

**Step 1 — Enforce the hard-stop rule in the agent unconditionally.**  
The agent must never approve or recommend approval for any PR that touches `.github/workflows/`, regardless of how small the change appears. The stop condition is binary — it fires on path match, not on a risk assessment:

```
# copilot-instructions.md — Stop Condition #1 (must never be relaxed)
If ANY changed file matches `.github/workflows/**`:
  - Post: "🛑 Agent Review — Human Review Required"
  - Reference: Stop Condition #1 — sensitive workflow path detected
  - Do NOT post an `approve` recommendation
  - Do NOT merge or suggest merging
```

The agent's prompt instructions must make clear that "trivial" is not a valid override for a hard-stop condition.

**Step 2 — Add CODEOWNERS for `.github/workflows/`.**  
Add a CODEOWNERS entry that requires a named human team to approve any change to workflow files. The agent is not a valid CODEOWNERS reviewer:

```
# CODEOWNERS
/.github/workflows/   @platform-team @security-team
```

With this entry in place, GitHub automatically requests a review from `@platform-team` and `@security-team` on any PR touching that path. The merge button remains locked until a member of one of those teams approves.

**Step 3 — Add a branch protection rule requiring CODEOWNERS review.**  
In repository **Settings → Branches → Branch protection rules** for `main`:
- Enable **Require review from Code Owners**.
- Enable **Restrict who can push to matching branches** (exclude the agent's identity).

This makes CODEOWNERS approval a required check, not just a requested review — so the PR cannot be merged by approving review count alone if the CODEOWNERS review is still pending.

**Step 4 — Verify the audit trail.**  
After the corrective controls are in place, any future PR touching `.github/workflows/` will show:
- An agent comment flagging the sensitive path (agent attribution).
- A CODEOWNERS review request to a human team (platform-level enforcement).
- A human approval from a named team member (human attribution).
- A merge commit showing the human approver's identity (full audit trail).

**Root cause summary:** Hard-stop conditions must be enforced by platform controls — CODEOWNERS and branch protection — not only by prompt instructions. Prompt instructions are advisory; an agent that reasons about whether a hard-stop applies can rationalise its way around them. The platform enforcement layer must make the hard-stop mechanically unavoidable.

---

## Quick Reference: Use Case Answer Summary

| # | Topic | Root Cause | Primary Fix |
|---|-------|-----------|-------------|
| 1 | Agent Architecture & SDLC | Autonomy level mismatch — Level 1 instructions with Level 3 execution rights | Remove auto-merge; add branch protection + CODEOWNERS |
| 2 | Tool Use & Environment | `write-all` permissions + token echoed to step output | Explicit minimal `permissions:` block; remove token export step |
| 3 | Memory State & Execution | Non-idempotent changelog step — no existence check before writing | Idempotency guard or artifact-based checkpointing keyed on release tag |
| 4 | Multi-Agent Coordination | No file-overlap detection before applying parallel specialist diffs | Coordinator conflict-detection step + `conflict_files` schema field + environment gate |
| 5 | Guardrails & Accountability | Hard-stop applied subjectively; no CODEOWNERS for workflow paths | Unconditional hard-stop in agent + CODEOWNERS + branch protection requiring CODEOWNERS review |
