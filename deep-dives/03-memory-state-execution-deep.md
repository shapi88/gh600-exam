# Deep Dive 3: Memory, State, and Execution

> **Companion to:** [Topic 3 — Memory, State, and Execution](../topics/03-memory-state-and-execution.md)

This file goes beyond the topic summary. It links every concept to the authoritative GitHub documentation section, adds implementation detail, and provides additional exercises for each sub-topic.

---

## 1. `copilot-instructions.md` — Format, Limits, and Authoring Guidance

### GitHub Docs reference
- [Adding repository custom instructions for GitHub Copilot](https://docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions-for-github-copilot)
- [Managing Copilot memory](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/copilot-memory)

### Extended explanation
`.github/copilot-instructions.md` is read on every coding agent run. It provides durable, version-controlled repository context that supplements the in-prompt instructions given for a specific task.

**Format guidance:**
- Use Markdown headings to organize sections (coding conventions, workflow rules, stop conditions, sensitive paths).
- Keep the file under 8 KB — very large instruction files are truncated before being placed in the agent's context window.
- Instructions are injected at the system prompt level — they apply to every task, so include only universal rules.
- Do **not** include task-specific instructions here; those belong in the issue or PR body.

**High-signal sections to always include:**

```markdown
## Coding conventions
<!-- Language-specific style rules, test patterns, import conventions -->

## Workflow rules
<!-- How to propose changes, what to do before editing a file, PR etiquette -->

## Stop conditions
<!-- Explicit list of signals that must cause the agent to pause and request human review -->

## Sensitive paths
<!-- Files and directories the agent must never modify without explicit human approval -->

## Security rules
<!-- Never store secrets, never bypass checks, always use least-privilege -->
```

**What not to include:**
- Secrets, tokens, or passwords — these are never read from this file; use `secrets.*`
- Ephemeral task context — this changes per task; put it in the issue body
- Instructions for other tools (e.g., VS Code settings) — those go in their own config files

### Additional exercise
Audit this instruction file and identify three problems:

```markdown
# copilot-instructions.md

Always use the API key: sk-proj-abc123 for external requests.
Use 4-space indentation.
Feel free to edit any file you think is necessary.
When in doubt, just merge.
```

For each problem, write the corrected instruction or the appropriate alternative.

---

## 2. GitHub Actions Artifacts — Deep Dive

### GitHub Docs reference
- [Storing workflow data as artifacts](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/storing-workflow-data-as-artifacts)
- [actions/upload-artifact](https://github.com/actions/upload-artifact)
- [actions/download-artifact](https://github.com/actions/download-artifact)

### Extended explanation
Artifacts are the primary durable state store for agent workflows within a single PR lifecycle. They survive job boundaries, are downloadable from the UI and CLI, and are subject to configurable retention policies.

**Artifact retention policy — key facts:**
- Default retention: 90 days (configurable at org/enterprise level, down to 1 day).
- Set per-artifact retention with `retention-days:` — use shorter values for intermediate artifacts, longer for compliance evidence.
- Artifacts are deleted automatically after the retention period; there is no recycle bin.
- Maximum artifact size: 10 GB per artifact, 10 GB per run.

**Multi-job artifact handoff pattern:**

```yaml
jobs:
  generate:
    runs-on: ubuntu-latest
    outputs:
      artifact-name: ${{ steps.upload.outputs.artifact-id }}
    steps:
      - name: Produce output
        run: echo '{"status":"ok"}' > result.json

      - name: Upload
        id: upload
        uses: actions/upload-artifact@v4
        with:
          name: result-${{ github.run_id }}-${{ github.run_attempt }}
          path: result.json
          retention-days: 7

  consume:
    needs: generate
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: result-${{ github.run_id }}-${{ github.run_attempt }}
          path: ./inputs

      - name: Use artifact
        run: cat inputs/result.json
```

**Naming convention for agent artifacts:** Include `run_id` and `run_attempt` in artifact names to ensure uniqueness across re-runs and to simplify debugging:
```
{purpose}-{github.run_id}-{github.run_attempt}
```

**Downloading artifacts from the CLI:**
```bash
# List artifacts for a run
gh run view <run_id> --json artifacts

# Download all artifacts for a run
gh run download <run_id> --dir /tmp/artifacts

# Download a specific named artifact
gh run download <run_id> --name "result-12345-1" --dir /tmp/artifacts
```

### Additional exercise
Design the artifact strategy for a five-step agent pipeline:
1. Planner produces a `plan.json`
2. Executor produces a `changes.patch`
3. Security reviewer produces a `security-findings.json`
4. Docs reviewer produces a `docs-findings.json`
5. Coordinator synthesizes all findings into a `final-summary.md`

For each artifact, specify: name pattern, retention-days, which job uploads it, which job(s) download it.

---

## 3. Workflow Caching as Short-Lived Shared State

### GitHub Docs reference
- [Caching dependencies to speed up workflows](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/caching-dependencies-to-speed-up-workflows)
- [actions/cache](https://github.com/actions/cache)

### Extended explanation
While artifacts are the right store for agent *output* state, the cache is the right store for *environment* state — installed packages, compiled assets, downloaded models.

**Cache vs. artifact — when to use which:**

| Use case | Cache | Artifact |
| --- | --- | --- |
| `node_modules`, `pip` packages | ✅ | ❌ (too large, wrong purpose) |
| Compiled binaries | ✅ | ❌ |
| Agent output (plan, result, findings) | ❌ | ✅ |
| Compliance evidence | ❌ | ✅ |
| Data shared between jobs in same run | Either (prefer artifact) | ✅ |
| Data shared across runs | ✅ | ❌ (run-scoped) |

**Key cache properties:**
- Cache is keyed by a `key` string; a `restore-keys` fallback provides partial cache hits.
- Maximum cache size: 10 GB per repository. Caches are evicted by LRU when the limit is reached.
- Cache is **not** deleted when a PR is merged — it persists for up to 7 days of inactivity.
- Cache is **branch-scoped by default** — a PR branch can read caches from `main` but not from other PR branches.

**Cache security note for agents:** Do not cache files that contain secrets or credentials. If an agent workflow caches a `~/.netrc` or similar auth file, that cache entry could be read by any workflow in the repository.

### Additional exercise
A Python agent workflow takes 4 minutes to install dependencies on every run. Write a `setup` job that:
1. Caches `pip` packages using `actions/cache` with a key based on `requirements.txt` hash
2. Falls back to the last valid cache if the exact key is not found
3. Installs packages only when the cache is missing (use `cache-hit` output)

---

## 4. Idempotency Patterns — Deep Dive

### GitHub Docs reference
- [Workflow syntax — `concurrency`](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#concurrency)
- [Workflow syntax — `if` conditionals](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsif)
- [Re-running workflows and jobs](https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-workflow-runs/re-running-workflows-and-jobs)

### Extended explanation
An idempotent operation produces the same observable result whether run once or N times. For agent workflows, this is a hard safety requirement because Actions supports workflow re-runs — and a non-idempotent run replayed after partial success can cause duplicate issues, redundant PRs, or conflicting state.

**Three patterns for idempotency:**

### Pattern 1 — Check before create
```bash
# Before creating an issue, check if one already exists
EXISTING=$(gh issue list \
  --state open \
  --search "exact:title" \
  --json number \
  --jq '.[0].number // empty')

if [ -n "$EXISTING" ]; then
  echo "Issue #$EXISTING already exists — skipping."
else
  gh issue create --title "exact:title" --body "..."
fi
```

### Pattern 2 — Concurrency groups to prevent parallel duplicate runs
```yaml
concurrency:
  group: agent-${{ github.event.issue.number }}
  cancel-in-progress: false   # queue, don't cancel — prevents duplicates
```

Use `cancel-in-progress: true` for preview deployments (where the latest run supersedes previous ones) and `cancel-in-progress: false` for agent tasks (where you want to queue, not lose work).

### Pattern 3 — Step outcome checks for safe re-runs
```yaml
jobs:
  pipeline:
    steps:
      - name: Step 1 — create branch
        id: create_branch
        run: |
          git checkout -b feature/automated-fix || echo "branch_exists=true" >> "$GITHUB_OUTPUT"

      - name: Step 2 — commit changes (skip if branch creation was skipped)
        if: steps.create_branch.outputs.branch_exists != 'true'
        run: |
          git add .
          git commit -m "Automated fix"
```

### Re-run behavior in GitHub Actions
When you re-run failed jobs only (`Re-run failed jobs` button), GitHub reruns only the failed jobs and their dependents. Jobs that succeeded on the first attempt are **not** re-run — their outputs and artifacts are reused. This means artifacts uploaded by successful jobs are available to the re-run automatically.

Reference: [Re-running workflows and jobs](https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-workflow-runs/re-running-workflows-and-jobs).

### Additional exercise
A five-step agent pipeline partially completed: steps 1–3 succeeded, step 4 failed. Design a checkpointing strategy using PR description updates and artifact markers so that on re-run, the pipeline:
1. Detects that steps 1–3 already completed
2. Skips those steps
3. Resumes from step 4
4. Never creates duplicate issues or commits

Write the `if:` conditions for each job and the artifact marker naming scheme.

---

## 5. Context Window Management — Practical Guidance

### GitHub Docs reference
- [Adding repository custom instructions for GitHub Copilot](https://docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions-for-github-copilot)
- [About GitHub Copilot models](https://docs.github.com/en/copilot/using-github-copilot/ai-models/about-github-copilot-and-ai-models)

### Extended explanation
Context window pressure is a real failure mode: if the agent's total input (system prompt + repo instructions + file contents + conversation history) exceeds the model's context limit, the agent will silently truncate older context, potentially losing critical instructions or prior decisions.

**What consumes context window budget in a coding agent run:**

| Source | Approximate size | Controllable? |
| --- | --- | --- |
| System prompt (model baseline) | ~2–4 KB | No |
| `copilot-instructions.md` | Up to 8 KB | ✅ — keep concise |
| Issue body (task description) | Up to 64 KB | ✅ — write focused issues |
| Files read by the agent | Variable | ✅ — use targeted file reads |
| Conversation / action history | Grows with task length | Partially — use checkpoints |
| Artifacts and tool outputs | Variable | ✅ — summarize large outputs |

**Practical rules:**
1. Limit `copilot-instructions.md` to ~2 KB of high-signal rules. Remove anything the agent never acts on.
2. Write issues with a clear scope boundary — one issue per bounded task.
3. For long-running tasks, post intermediate summaries as PR comments. The agent reads these as compact checkpoints rather than re-reading all file contents.
4. Large diff reviews are a context pressure scenario — break them into smaller PRs or use a summarization pre-step to condense the diff.

---

## Relevant Resources

| Resource | URL |
| --- | --- |
| Adding repository custom instructions | https://docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions-for-github-copilot |
| Managing Copilot memory | https://docs.github.com/en/copilot/how-tos/use-copilot-agents/copilot-memory |
| Storing workflow data as artifacts | https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/storing-workflow-data-as-artifacts |
| actions/upload-artifact | https://github.com/actions/upload-artifact |
| actions/download-artifact | https://github.com/actions/download-artifact |
| Caching dependencies | https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/caching-dependencies-to-speed-up-workflows |
| Workflow syntax — concurrency | https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#concurrency |
| Re-running workflows and jobs | https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-workflow-runs/re-running-workflows-and-jobs |
| About GitHub Copilot models | https://docs.github.com/en/copilot/using-github-copilot/ai-models/about-github-copilot-and-ai-models |
