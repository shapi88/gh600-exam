# Copilot Agent Memory

This file provides persistent context hints for agentic AI assistants operating in this repository.
Read this file before acting on any task. It complements `.github/copilot-instructions.md`.

---

## Repository Identity

| Field | Value |
| --- | --- |
| Repository | `shapi88/gh600-exam` |
| Purpose | GH-600 Agentic AI Developer exam study package |
| Default branch | `main` |
| Autonomy level | **Level 2** — limited repo changes; human review required for merges |

---

## Key File Locations

| File / Path | Purpose |
| --- | --- |
| `README.md` | Top-level index and 5-week study plan |
| `topics/` | 6 topic files + workflow diagram + cheat sheets + resources |
| `tutorials/` | 7 hands-on lab files (00–06) + README navigation index |
| `deep-dives/` | 7 deep-dive files + docs index |
| `evals/` | Evaluation scenarios, rubric, and versioned results |
| `templates/` | Copy-paste-ready templates for agent configuration |
| `.github/copilot-instructions.md` | Agent standing orders (MAY / MAY NOT / stop conditions) |
| `.github/guardrails.md` | Autonomy matrix, hard stops, audit trail requirements |
| `.github/mcp-config.json` | Approved MCP server registry |
| `.github/skills/` | Skill YAML definitions (5 skills) |
| `.github/INCIDENT_RESPONSE.md` | Runbook for handling bad agent actions |
| `CODEOWNERS` | Human review requirements for sensitive paths |
| `.github/branch-protection.md` | Documented branch protection settings |

---

## Repo Conventions

- All content files use Markdown (`.md`).
- Topic files follow the pattern: `topics/0N-<slug>.md`.
- Tutorial files follow the pattern: `tutorials/0N-<slug>-lab.md`.
- Deep-dive files follow the pattern: `deep-dives/0N-<slug>-deep.md`.
- Evaluation files live in `evals/`; results files are named `results-v<N>.md`.
- Template files live in `templates/` and are named `<purpose>-example.<ext>` or `<purpose>-template.<ext>`.
- Agent branches must be prefixed `copilot/` or `agent/`.
- Every agent PR must have a non-empty description including rationale and scope.

---

## Autonomy Constraints (quick reference)

- **MAY**: Read files, open branches, create/edit docs outside protected paths, post PR/issue comments, apply labels, upload artifacts.
- **MAY NOT**: Modify `.github/workflows/`, `CODEOWNERS`, `.github/copilot-instructions.md`; push to `main`; call unapproved MCP servers; merge own PRs.
- **STOP CONDITIONS**: See `.github/copilot-instructions.md` — 7 hard stops listed.

---

## Evaluation Baselines

| File | Purpose | Current version |
| --- | --- | --- |
| `evals/scenarios.md` | 5 golden test scenarios | v1 (original) |
| `evals/rubric.md` | 4-dimension scoring rubric | v1 (original) |
| `evals/results-v1.md` | Prompt v1 results (scope-control fail) | v1 |
| `evals/results-v2.md` | Prompt v2 results (all pass) | v2 |
| `evals/agent-template-scenarios.md` | Template-specific scenarios | v1 |
| `evals/agent-template-rubric.md` | Template-specific rubric | v1 |

Do not modify existing `evals/results-v*.md` files — append new versions (`v3`, `v4`, etc.) instead.

---

## Approved MCP Servers (summary)

Full registry: `.github/mcp-config.json`

| Name | Allowed scopes |
| --- | --- |
| `github-rest` | `contents:read`, `issues:read`, `pull-requests:read`, `checks:read` |
| `github-actions` | `actions:read`, `actions:write` |
| `github-graphql` | `contents:read`, `issues:read`, `pull-requests:read` |

Default policy: **deny** — any server not listed above is prohibited.

---

## State Checkpoints

Use these checkpoints to resume a multi-step task safely:

1. **Before first write**: Confirm branch name follows `copilot/<task-slug>` convention.
2. **After each file edit**: Record the file path and reason in the PR description.
3. **Before opening PR**: Verify the PR checklist (`templates/pr-checklist.md`) is complete.
4. **After opening PR**: Post the agent review summary comment.
5. **Before any deployment action**: Verify `environment: agent-execution` approval is configured.

---

## Idempotency Rule

All agent actions must be safe to re-run without causing double side-effects:

- Use `--force` only when idempotent (e.g., label application).
- Check if a PR or branch already exists before creating one.
- Artifact uploads with the same name overwrite the previous version — this is intentional.
