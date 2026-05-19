# Example: `.github/copilot-instructions.md`

> **How to use this file:** Copy the content below to `.github/copilot-instructions.md` in your repository.
> Replace the annotations (text in `<!-- ... -->` comments) with values appropriate for your project.
> Then remove the annotation comments before committing.

---

```markdown
# Copilot Agent Standing Orders                <!-- Required: keep this heading -->

<!-- ANNOTATION: This file is your agent's constitution.
     Every agentic workflow reads it before acting.
     Be explicit — vague instructions produce inconsistent behavior. -->

## Default Posture

- Read before write. Inspect relevant files before proposing changes.
- Plan before code. Propose a written plan and wait for acknowledgement.
- Smallest diff. Change only what is necessary.
- Attribute every action. Every PR must have a non-empty description.

## Autonomy Level

<!-- ANNOTATION: Choose ONE level. Most repos should start at Level 1 or 2.
     Never set Level 3 or 4 without an environment protection gate in place. -->

This repository operates at **Autonomy Level 2** — limited repo changes.
Human review is required before any merge.

## What the Agent MAY Do

<!-- ANNOTATION: Be explicit. List actions by verb (read, create, edit, label, comment). -->

- Read any file, issue, PR, workflow log, or artifact.
- Open branches with prefix `copilot/` or `agent/`.
- Create or edit files outside the protected paths below.
- Post PR comments and issue comments.
- Apply labels to issues.
- Upload workflow artifacts.

## What the Agent MAY NOT Do

<!-- ANNOTATION: These are your hard stops. Add repo-specific prohibitions. -->

- Merge a PR into `main` without at least one human approval.
- Create, edit, or delete `.github/workflows/**`.
- Create, edit, or delete `CODEOWNERS`.
- Create, edit, or delete this file (`.github/copilot-instructions.md`).
- Access, print, or log any secret or credential.
- Push directly to `main` or any protected branch.
- Call an MCP server not listed in `.github/mcp-config.json`.

## Stop Conditions

<!-- ANNOTATION: List the specific triggers that must cause the agent to pause.
     Ordered from most critical to least critical. -->

Stop and post a comment requesting human review if:

1. The diff touches `.github/workflows/`.
2. The diff touches `CODEOWNERS` or `.github/copilot-instructions.md`.
3. The task requires a secret, credential, or deployment token.
4. The diff exceeds 400 lines across more than 5 files.
5. The task targets a production environment.
6. A required status check is failing and the task requires merging.

## Required Audit Trail

Every agent-initiated action must produce at least one of:

- A PR with a non-empty description.
- A PR or issue comment with rationale.
- A workflow log entry with a structured summary step.

## Rollback Instruction

If this agent makes a harmful change:
1. Close or revert the PR.
2. Delete the agent branch.
3. File an incident report using `.github/ISSUE_TEMPLATE/agent-incident.md`.
4. Update this file to prevent recurrence.
```

---

## Annotation Key

| Annotation | Meaning |
| --- | --- |
| `<!-- Required: ... -->` | Do not remove this element |
| `<!-- ANNOTATION: ... -->` | Remove before committing; this is guidance only |
| Replace `@shapi88` | Replace with your CODEOWNERS username or team slug |
| Replace `copilot/` prefix | Change to your preferred branch naming convention |
