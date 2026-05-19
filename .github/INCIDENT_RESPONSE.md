# Incident Response Runbook — Agentic AI

Use this runbook whenever an agent-assisted action in this repository has caused or may have caused harm.

> **Goal:** Contain the harm quickly, restore a safe state, then improve controls to prevent recurrence.

---

## Severity Classification

| Severity | Examples | Response time |
| --- | --- | --- |
| **P1 — Critical** | Credential leaked, production broken, data deleted | Immediate (< 15 min) |
| **P2 — High** | Wrong merge to main, scope violation, secret in log | < 1 hour |
| **P3 — Medium** | Incorrect label, wrong PR comment, minor wrong edit | < 24 hours |
| **P4 — Low** | Cosmetic error, wrong tone in comment | Next working day |

---

## Step 1 — Identify the Harmful Action

1. Locate the PR, issue, commit, or workflow run that caused the problem.
2. Record the commit SHA, PR number, and workflow run ID.
3. Determine which agent skill and autonomy level was active.
4. Check `.github/guardrails.md` — was a hard stop violated?

```bash
# Find recent agent-authored commits
git log --oneline --author="copilot" | head -20

# Find recent workflow runs
gh run list --limit 20 --json name,status,createdAt,databaseId
```

---

## Step 2 — Contain the Harm

### If the bad change is on a PR (not yet merged):

```bash
gh pr close <pr-number>
gh api repos/{owner}/{repo}/git/refs/heads/<branch> --method DELETE
```

### If the bad change is merged to `main`:

```bash
git checkout -b revert/incident-<short-sha>
git revert <bad-sha> --no-edit
git push origin revert/incident-<short-sha>
gh pr create \
  --title "Revert: incident <short-sha>" \
  --body "Emergency revert. See incident issue #<number>." \
  --base main
# Request immediate review from CODEOWNERS
```

### If a credential was exposed:

1. **Rotate the credential immediately** — do not wait for any other step.
2. Revoke the old token in the relevant service.
3. Check workflow logs to determine who may have seen the credential.
4. Notify affected parties if required by your security policy.
5. Follow Step 3 to remove the credential from git history.

---

## Step 3 — Remove the Harmful Artefact (if applicable)

### Remove a file from git history (credential leak):

```bash
# Install BFG or use git filter-repo
# BFG example:
java -jar bfg.jar --delete-files <filename> .
git reflog expire --expire=now --all
git gc --prune=now --aggressive
git push --force origin main
# Requires branch protection override — coordinate with admin.
```

---

## Step 4 — Investigate Root Cause

1. Review the workflow run logs for the triggering event.
2. Read `.github/copilot-instructions.md` — which stop condition was not triggered?
3. Read `.github/guardrails.md` — which policy was not enforced?
4. Check `.github/mcp-config.json` — did the agent call an unapproved server?
5. Determine whether the failure is in: the prompt, the skill YAML, the workflow, or the platform controls.

```bash
# View workflow run logs
gh run view <run-id> --log

# View PR diff that caused harm
gh pr diff <pr-number>
```

---

## Step 5 — Update Controls

After identifying the root cause, update at least one of the following to prevent recurrence:

| Root cause type | Update target |
| --- | --- |
| Agent did not stop when it should | Add stop condition to `.github/copilot-instructions.md` |
| Agent had permissions it should not have | Narrow `permissions:` block in the relevant workflow |
| No human review gate | Add environment protection or required reviewer to the workflow |
| MCP server called without approval | Add to deny list in `.github/mcp-config.json` |
| CODEOWNERS did not cover the path | Add the path to `CODEOWNERS` |
| Skill had incorrect scope | Update the skill YAML in `.github/skills/` |

---

## Step 6 — File Post-Mortem Issue

Create an issue using the template at `.github/ISSUE_TEMPLATE/agent-incident.md`.

The issue must include:

- Date and time of discovery
- Severity (P1–P4)
- Summary of what the agent did
- Root cause
- Control changes made
- Verification that the harm is contained
- Sign-off from a CODEOWNERS-designated maintainer

---

## Step 7 — Verify Recovery

```bash
# Confirm main is in a clean state
git log --oneline main | head -5

# Confirm branch protection is intact
gh api repos/{owner}/{repo}/branches/main/protection \
  --jq '{enforce_admins: .enforce_admins.enabled, required_reviews: .required_pull_request_reviews.required_approving_review_count}'

# Re-run guardrails check
gh workflow run guardrails-check.yml
```

---

## Reference

- [GitHub Docs — Reverting a pull request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/incorporating-changes-from-a-pull-request/reverting-a-pull-request)
- [GitHub Docs — Removing sensitive data from a repository](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository)
- [GitHub Docs — Audit log events](https://docs.github.com/en/organizations/keeping-your-organization-secure/managing-security-settings-for-your-organization/audit-log-events-for-your-organization)
- `.github/guardrails.md` — policy and autonomy matrix
- `.github/copilot-instructions.md` — agent standing orders
