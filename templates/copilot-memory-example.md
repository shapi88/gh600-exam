# Copilot Memory — Example and Annotated Template

> **File location:** `.github/copilot-memory.md`  
> **GH-600 topic:** Topic 3 — Memory, State, and Execution  
> **Autonomy constraint:** CODEOWNERS review required to modify (see `CODEOWNERS`)

This file provides persistent context that GitHub Copilot and agentic AI assistants load at the start of every session. It complements `.github/copilot-instructions.md` (standing orders) and survives across multiple runs.

The live example used by this repository is [`.github/copilot-memory.md`](../.github/copilot-memory.md). Copy the sections below as a starting template for your own repository.

---

## Why this file matters (exam note)

`copilot-instructions.md` tells the agent *how to behave*. `copilot-memory.md` tells it *where things are* and *what the repo's current state is*. Together they form the agent's durable repository memory — the `Repository memory` row in the memory-type cheat sheet.

**What to store here:**
- Repo identity and purpose
- Key file locations and naming conventions
- Autonomy constraints quick-reference
- Evaluation baselines and their current version
- State checkpoints for resumable multi-step tasks
- Approved MCP server summary

**What NOT to store here:**
- Secrets, tokens, or credentials (use `secrets.*` at runtime)
- One-off or task-specific instructions (use prompt context instead)
- Sensitive personal data (privacy violation)

---

## Template

```markdown
# Copilot Agent Memory

This file provides persistent context hints for agentic AI assistants operating in this repository.
Read this file before acting on any task. It complements `.github/copilot-instructions.md`.

---

## Repository Identity

| Field | Value |
| --- | --- |
| Repository | `<owner>/<repo>` |
| Purpose | (One sentence: what this repo is for) |
| Default branch | `main` |
| Autonomy level | **Level <N>** — (brief description) |

---

## Key File Locations

| File / Path | Purpose |
| --- | --- |
| `README.md` | Top-level index |
| `src/` | Application source code |
| `tests/` | Unit and integration tests |
| `.github/copilot-instructions.md` | Agent standing orders |
| `.github/guardrails.md` | Autonomy matrix and hard stops |
| `.github/mcp-config.json` | Approved MCP server registry |
| `CODEOWNERS` | Human review requirements |

---

## Repo Conventions

- (List your naming conventions, code style rules, test framework, etc.)
- Agent branches must be prefixed `copilot/` or `agent/`.
- Every agent PR must have a non-empty description including rationale and scope.

---

## Autonomy Constraints (quick reference)

- **MAY**: (list what the agent is allowed to do)
- **MAY NOT**: (list hard stops — mirror `.github/copilot-instructions.md`)
- **STOP CONDITIONS**: See `.github/copilot-instructions.md`.

---

## Evaluation Baselines

| File | Purpose | Current version |
| --- | --- | --- |
| `evals/scenarios.md` | Golden test scenarios | v1 |
| `evals/rubric.md` | 4-dimension scoring rubric | v1 |
| `evals/results-v1.md` | Prompt v1 results | v1 |

Do not modify existing `evals/results-v*.md` files — append new versions instead.

---

## Approved MCP Servers (summary)

Full registry: `.github/mcp-config.json`

| Name | Allowed scopes |
| --- | --- |
| `github-rest` | `contents:read`, `issues:read`, `pull-requests:read` |

Default policy: **deny** — any server not listed is prohibited.

---

## State Checkpoints

Use these checkpoints to resume a multi-step task safely:

1. **Before first write**: Confirm branch name follows `copilot/<task-slug>` convention.
2. **After each file edit**: Record the file path and reason in the PR description.
3. **Before opening PR**: Verify the PR checklist (`templates/pr-checklist.md`) is complete.
4. **After opening PR**: Post the agent review summary comment.
5. **Before any deployment action**: Verify environment approval is configured.

---

## Idempotency Rule

All agent actions must be safe to re-run without causing double side-effects:
- Check if a PR or branch already exists before creating one.
- Use existence checks before creating issues or labels.
- Artifact uploads with the same name overwrite the previous version — this is intentional.
```

---

## Annotation Guide

| Section | Why it exists | Exam relevance |
| --- | --- | --- |
| **Repository Identity** | Gives the agent its operating context at session start | Confirms the agent knows its autonomy level before acting |
| **Key File Locations** | Prevents the agent from searching for files it should already know about | Reduces stale-context errors (Topic 4 error category) |
| **Repo Conventions** | Ensures consistent coding style and naming across agent sessions | Stored as repository memory — durable, non-sensitive, reusable |
| **Autonomy Constraints** | Quick-reference copy of the most critical stop conditions | Backs up `copilot-instructions.md` in the agent's context window |
| **Evaluation Baselines** | Tells the agent which eval files are canonical; prevents overwriting | Hard stop: do not modify existing result baselines |
| **Approved MCP Servers** | Prevents the agent from calling unapproved servers | Matches the deny-by-default policy in `mcp-config.json` |
| **State Checkpoints** | Enables safe resumption after failure | Idempotency + checkpointing pattern (Topic 3) |
| **Idempotency Rule** | Prevents duplicate side-effects on retry | Directly tested in Topic 3 exam questions |
