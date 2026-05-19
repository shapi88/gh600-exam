# Copilot Agent Standing Orders

This file governs how GitHub Copilot and any agentic AI assistant operates in this repository.
It is enforced at **Autonomy Level 2** — limited repo changes; every merge requires human review.

---

## Default Posture

- **Read before write.** Inspect the relevant files, issues, and history before proposing any change.
- **Plan before code.** Propose a short written plan (max 10 bullet points) and wait for acknowledgement before opening a PR or editing files.
- **Smallest diff.** Make only the changes necessary to satisfy the task. Do not refactor, rename, or reformat unrelated code.
- **Attribute every action.** Every agent-produced PR must include a description explaining what changed and why.

---

## Autonomy Level

| Level | This repo setting |
| --- | --- |
| 0 — Suggest only | ❌ |
| 1 — Draft plans/content | ❌ |
| **2 — Limited repo changes** | ✅ Active |
| 3 — Broad repo automation | ❌ |
| 4 — Deployment-affecting | ❌ |

At Level 2 the agent **may** open branches and PRs, but **may not** merge, deploy, or modify governance files without a human approval.

---

## What the Agent MAY Do

- Read any file, issue, PR, workflow log, or artifact in this repository.
- Open a new branch (prefix: `copilot/` or `agent/`).
- Create or edit files outside the protected paths listed below.
- Post PR comments and review summaries.
- Apply labels to issues (triage skill only).
- Upload workflow artifacts.
- Run evaluation scenarios against existing rubric files.

---

## What the Agent MAY NOT Do

- Merge a PR into `main` without at least one human approval.
- Create, edit, or delete any file under `.github/workflows/`.
- Create, edit, or delete `CODEOWNERS`.
- Create, edit, or delete `.github/copilot-instructions.md` (this file).
- Access, print, or log any repository secret or environment variable that contains a credential.
- Push directly to `main` or any protected branch.
- Call an MCP server not listed in `.github/mcp-config.json`.
- Execute shell commands that install packages or modify the runner environment outside of an approved `copilot-setup-steps.yml`.

---

## Stop Conditions

Stop immediately and post a comment requesting human review if **any** of the following are true:

1. The proposed diff touches a file in `.github/workflows/`.
2. The proposed diff touches `CODEOWNERS` or `.github/copilot-instructions.md`.
3. The task requires accessing a secret, credential, or deployment token.
4. The diff size exceeds 400 lines across more than 5 files.
5. The task targets the `production` environment.
6. A required status check is failing and the task requires merging.
7. A conflict exists between two agent-authored changes on the same file.

---

## Required Audit Trail

Every agent-initiated action must produce at least one of:

- A PR with a non-empty description (rationale + scope summary).
- A PR comment summarising what was changed and why.
- A workflow log entry with a structured summary step.

Silent writes with no attribution are prohibited.

---

## Sensitive-Path Reference

| Path pattern | Protection |
| --- | --- |
| `.github/workflows/**` | CODEOWNERS + required review; agent hard-stop |
| `CODEOWNERS` | CODEOWNERS + required review; agent hard-stop |
| `.github/copilot-instructions.md` | CODEOWNERS + required review; agent hard-stop |
| `evals/**` | CODEOWNERS review recommended |
| `.github/mcp-config.json` | CODEOWNERS review recommended |

---

## Rollback Instruction

If a change introduced by this agent is found to be harmful:

1. Close or revert the PR (`git revert <sha>`).
2. Delete the agent branch.
3. File an incident report using `.github/ISSUE_TEMPLATE/agent-incident.md`.
4. Review and update this file to prevent recurrence.

See `.github/INCIDENT_RESPONSE.md` for the full runbook.
