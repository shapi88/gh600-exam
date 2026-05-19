# Example: Guardrails Policy

> **How to use:** Copy the structure below to `.github/guardrails.md` in your repository.
> Customise the autonomy matrix, hard stops, and rollback procedures for your context.
> CODEOWNERS review is required to modify this file.

---

## Autonomy Matrix

<!-- ANNOTATION: Define what each level is allowed to do IN THIS REPO.
     Start conservatively. Escalate levels only with approval gates in place. -->

| Level | Capability | Allowed actions | Required controls |
| --- | --- | --- | --- |
| 0 — Suggest only | Read + propose text | Read files, post comments | No write access |
| 1 — Draft content | Create drafts, plans | Open draft PRs, upload artifacts | Human approval before state change |
| **2 — Limited changes** | Create/edit files, open PRs | Branch writes, PR comments | Branch restriction, required checks, human review |
| 3 — Broad automation | Labels, multi-file edits | Issues:write, broader writes | Audit trail, approvals, rollback plan |
| 4 — Deployment-affecting | Trigger deploys | Actions:write, env access | Environments API, required reviewers, OIDC |

**Active level for this repository:** <!-- REPLACE: Level 2 -->

---

## Hard Stops — Never-Do List

<!-- ANNOTATION: These are unconditional. No task justifies bypassing them. -->

| # | Prohibited action | Reason |
| --- | --- | --- |
| 1 | Modify `.github/workflows/` | Privilege escalation risk |
| 2 | Modify `CODEOWNERS` | Removes all other guardrails |
| 3 | Modify `.github/copilot-instructions.md` | Undermines standing orders |
| 4 | Push directly to `main` | Bypasses required review |
| 5 | Expose a secret in any output | Irreversible credential leak |
| 6 | Call an unapproved MCP server | Unreviewed attack surface |
| 7 | Approve own PRs | Removes human-in-the-loop |
| 8 | <!-- REPLACE: add repo-specific prohibition --> | <!-- REPLACE: reason --> |

---

## Required Audit Trail

<!-- ANNOTATION: Define what evidence is mandatory for each action type. -->

| Action type | Required evidence |
| --- | --- |
| File creation or edit | PR with non-empty description |
| Issue label applied | Issue comment with rationale |
| Workflow run triggered | Workflow summary with task, trigger, actor |
| Artifact uploaded | Artifact metadata logged |
| Multi-agent handoff | Handoff artifact per `templates/artifact-schema.json` |

---

## Rollback Procedures

<!-- ANNOTATION: Define specific commands for your repo. Test them before you need them. -->

### Bad commit on agent branch

```bash
git revert <sha> --no-edit
git push origin <branch>
gh pr close <pr-number> --delete-branch
```

### Bad commit merged to main

```bash
git checkout -b revert/incident-<sha>
git revert <sha> --no-edit
git push origin revert/incident-<sha>
gh pr create --title "Revert: incident <sha>" --body "See incident issue #<number>."
```

### Leaked credential

1. Rotate the credential **immediately** — before any other step.
2. Revoke the old token in the relevant service.
3. Remove from git history using `git filter-repo` or BFG.
4. Force-push with admin override.
5. File incident report.

---

## Policy Enforcement Mapping

<!-- ANNOTATION: For every policy in the hard-stop list, show the GitHub control that enforces it. -->

| Policy | Enforcement mechanism |
| --- | --- |
| No direct push to main | Branch protection — restrict pushes |
| Human review required | Branch protection — required approving reviews |
| Sensitive-path review | CODEOWNERS + code owner review requirement |
| Status checks pass | Branch protection — required status checks |
| No admin bypass | Branch protection — enforce_admins: true |
| Least-privilege token | `permissions:` block in every workflow |
| MCP allowlist enforced | `.github/mcp-config.json` + agent stop condition |
| Deployment approval gate | `environment:` with Required Reviewer |

---

## Review Schedule

<!-- ANNOTATION: Set a realistic review cadence. Agent policies drift without regular review. -->

- After any security incident.
- When a new autonomy level is requested.
- When new MCP servers are added.
- At minimum: every 6 months.

Last reviewed: <!-- REPLACE: YYYY-MM-DD by @username -->
