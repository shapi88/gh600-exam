# Deep Dive 1: Agent Architecture and SDLC Processes

> **Companion to:** [Topic 1 — Agent Architecture and SDLC Processes](../topics/01-agent-architecture-and-sdlc.md)

This file goes beyond the topic summary. It links every concept to the authoritative GitHub documentation section, adds implementation detail, and provides additional exercises for each sub-topic.

---

## 1. GitHub Copilot Coding Agent — Under the Hood

### What it is
The GitHub Copilot coding agent is a fully autonomous agent that can read issues, plan implementation steps, open branches, push commits, and create pull requests — all without leaving GitHub.

### GitHub Docs reference
- [Using the GitHub Copilot coding agent](https://docs.github.com/en/copilot/using-github-copilot/using-copilot-coding-agent)
- [About GitHub Copilot coding agent](https://docs.github.com/en/copilot/concepts/about-copilot-coding-agent)
- [Best practices for using the coding agent](https://docs.github.com/en/copilot/using-github-copilot/best-practices-for-using-copilot-coding-agent)

### Extended explanation
The coding agent runs in an isolated, ephemeral environment provisioned by GitHub Actions. It is not a long-lived process — every invocation creates a fresh runner, executes a bounded task, and terminates. This architecture has deliberate implications:

| Property | Implication |
| --- | --- |
| Ephemeral runner | State from one run is not automatically available to the next — use artifacts or GitHub objects |
| Runs as a GitHub App | All actions appear under the Copilot identity, making attribution automatic |
| Actions-based | Every step is logged in the Actions tab; the full audit trail is built-in |
| Branch-scoped | The agent works on a feature branch; merging to `main` always requires the configured branch protection checks |

### Key configuration points
1. **Issue assignment** — assign an issue to Copilot to trigger agent execution. The issue body becomes the agent's primary context.
2. **`copilot-setup-steps.yml`** — pre-install tools, configure the environment, and set environment variables before the agent starts. See [Customizing the development environment for Copilot coding agent](https://docs.github.com/en/copilot/using-github-copilot/customizing-copilot-coding-agent).
3. **`.github/copilot-instructions.md`** — provides persistent repository-level context read by the agent on every run. Keep instructions concise, non-sensitive, and version-controlled.

### Additional exercise
Map the coding agent lifecycle to GitHub API calls. For each step below, identify the GitHub REST API endpoint or UI action the agent uses:

| Step | GitHub API endpoint or UI action |
| --- | --- |
| Reads the assigned issue | `GET /repos/{owner}/{repo}/issues/{issue_number}` |
| Creates a feature branch | `POST /repos/{owner}/{repo}/git/refs` |
| Commits a file change | `PUT /repos/{owner}/{repo}/contents/{path}` |
| Opens a draft pull request | `POST /repos/{owner}/{repo}/pulls` |
| Posts a progress comment | `POST /repos/{owner}/{repo}/issues/{issue_number}/comments` |
| Requests a review | `POST /repos/{owner}/{repo}/pulls/{pull_number}/requested_reviewers` |

Complete the table with documentation: [GitHub REST API — Pulls](https://docs.github.com/en/rest/pulls), [GitHub REST API — Git refs](https://docs.github.com/en/rest/git/refs).

---

## 2. GitHub Issues and Projects as Planning Artifacts

### GitHub Docs reference
- [About issues](https://docs.github.com/en/issues/tracking-your-work-with-issues/about-issues)
- [About GitHub Projects](https://docs.github.com/en/issues/planning-and-tracking-with-projects/learning-about-projects/about-projects)
- [GitHub REST API — Issues](https://docs.github.com/en/rest/issues)

### Extended explanation
Issues are the primary planning artifact for agents. A well-structured issue gives the agent bounded, parseable context. Poorly structured issues lead to stale context errors and scope creep.

**Anatomy of an agent-ready issue:**

```markdown
## Acceptance criteria
- [ ] Specific, verifiable outcome
- [ ] Another verifiable outcome

## Out of scope
- Explicitly list what must NOT be changed

## Risk note
Describe any sensitive paths, dependencies, or compliance concerns
```

**GitHub Projects as an orchestration layer:** GitHub Projects can track agent-managed issues on a board. Use custom fields (status, assigned agent, risk level) to give a coordinator agent a structured view of the backlog. The Projects API (`POST /graphql`) enables programmatic updates to project items.

### Additional exercise
Write a `gh` CLI command that:
1. Creates an issue with the agent-ready template above
2. Adds it to a specific GitHub Project
3. Sets a custom "Agent assigned" status field

Reference: [Using the GitHub CLI with Projects](https://docs.github.com/en/issues/planning-and-tracking-with-projects/automating-your-project/using-the-api-to-manage-projects).

---

## 3. Branch Protection — Deep Dive

### GitHub Docs reference
- [About protected branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)
- [Managing a branch protection rule](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/managing-a-branch-protection-rule)
- [About rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets)

### Extended explanation
Branch protection rules have multiple layers. The table below maps each layer to its exam-relevant behavior:

| Protection layer | What it enforces | API field |
| --- | --- | --- |
| Required reviews | Minimum number of human approvals | `required_approving_review_count` |
| Dismiss stale reviews | Re-review required after new commits | `dismiss_stale_reviews` |
| Code owner review | CODEOWNERS must approve affected paths | `require_code_owner_reviews` |
| Required status checks | CI must pass before merge | `required_status_checks.contexts` |
| Strict updates | Branch must be up to date before merge | `required_status_checks.strict` |
| Restrict pushes | Only named users/apps can push | `restrictions` |
| Enforce for admins | Even admins must follow rules | `enforce_admins` |
| Allow force pushes | Default: disabled; keep disabled for agents | `allow_force_pushes` |

**Rulesets vs. classic branch protection:** GitHub Rulesets (Organization-level) provide more granular bypass permissions and can target multiple branches/repos from one rule. For enterprise contexts, rulesets are preferred over per-repo classic protection rules.

### Full API example — set protection with all critical fields

```bash
gh api repos/{owner}/{repo}/branches/main/protection \
  --method PUT \
  --field required_status_checks='{"strict":true,"contexts":["ci/lint","ci/test"]}' \
  --field enforce_admins=true \
  --field required_pull_request_reviews='{
    "required_approving_review_count": 1,
    "dismiss_stale_reviews": true,
    "require_code_owner_reviews": true
  }' \
  --field restrictions=null \
  --field allow_force_pushes=false \
  --field allow_deletions=false
```

### Additional exercise
Compare classic branch protection with Repository Rulesets by answering:
1. Which setting blocks an agent from merging its own PR even with admin rights?
2. Which bypass mechanism lets a specific GitHub App skip required reviews?
3. How do you audit which bypass was used on a given merge? (Hint: check the audit log.)

Reference: [Managing rulesets for a repository](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/managing-rulesets-for-a-repository).

---

## 4. CODEOWNERS — Syntax and Auto-Review Routing

### GitHub Docs reference
- [About code owners](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners)

### Extended explanation
CODEOWNERS uses `.gitignore`-style patterns but with an important difference: the **last matching rule wins**. This means order matters significantly.

**Pattern examples:**

```gitignore
# Default owner for everything
*                     @org/team-leads

# Override for workflow files — must come after wildcard
.github/workflows/    @security-team @org/platform-leads

# Override for infrastructure
infrastructure/       @org/sre-team

# Override for secrets/credentials files — most specific, place last
**/*secret*           @security-team
**/*credential*       @security-team
.env*                 @security-team
```

**Rules for agent governance:**
- Always add `.github/workflows/` to CODEOWNERS to prevent agents from escalating their own privileges by editing workflow files.
- Add `CODEOWNERS` itself to CODEOWNERS so agents cannot modify the owners file.
- Add `*` as a catch-all to ensure every path has at least one human owner.

**Limitations to know for the exam:**
- CODEOWNERS only triggers if "Require review from Code Owners" is enabled in branch protection.
- CODEOWNERS does not gate `git push` directly — it gates the PR review process.
- Binary files and files in submodules are not covered by CODEOWNERS.

### Additional exercise
Given this CODEOWNERS file, determine which team is auto-requested as reviewer for each changed file:

```gitignore
*                 @org/all-engineers
src/auth/         @security-team
src/              @backend-team
docs/             @docs-team
.github/          @platform-team
```

Files changed in a PR: `src/auth/login.py`, `src/api/routes.py`, `docs/api.md`, `.github/workflows/ci.yml`, `README.md`. Write the expected reviewer for each.

---

## 5. Agent-Assisted PR Lifecycle — Full Trace

### End-to-end lifecycle with GitHub control plane mapping

The following traces a complete coding-agent-assisted change from issue assignment to merge:

```
1. Issue assigned to Copilot
   └── GitHub App webhook triggers agent execution

2. Agent reads issue
   └── GET /repos/{owner}/{repo}/issues/{n}
   └── GET /repos/{owner}/{repo}/contents/.github/copilot-instructions.md

3. Agent creates branch
   └── POST /repos/{owner}/{repo}/git/refs
   └── branch: copilot/issue-{n}-{slug}

4. Agent makes commits
   └── GET /repos/{owner}/{repo}/contents/{path}   (read file)
   └── PUT /repos/{owner}/{repo}/contents/{path}   (write file)

5. Agent opens draft PR
   └── POST /repos/{owner}/{repo}/pulls
   └── draft: true, body includes task checklist

6. Agent requests review
   └── POST /repos/{owner}/{repo}/pulls/{n}/requested_reviewers

7. CI runs (GitHub Actions)
   └── on: pull_request — status checks created

8. Human reviews and approves
   └── POST /repos/{owner}/{repo}/pulls/{n}/reviews
   └── event: APPROVE

9. Status checks pass
   └── Branch protection enforces: reviews + checks = mergeable

10. Merge
    └── PUT /repos/{owner}/{repo}/pulls/{n}/merge
```

### Additional exercise
Using the lifecycle trace above and the [GitHub REST API docs](https://docs.github.com/en/rest), write a shell script that:
1. Lists all open PRs created by the `github-actions[bot]` or `copilot` app
2. For each PR, prints whether required status checks are passing
3. Prints the name of any missing required check

Use `gh pr list --app copilot` and `gh api repos/{owner}/{repo}/commits/{sha}/check-runs`.

---

## Relevant Resources

| Resource | URL |
| --- | --- |
| Copilot coding agent overview | https://docs.github.com/en/copilot/using-github-copilot/using-copilot-coding-agent |
| About Copilot coding agent (concepts) | https://docs.github.com/en/copilot/concepts/about-copilot-coding-agent |
| Customizing the coding agent environment | https://docs.github.com/en/copilot/using-github-copilot/customizing-copilot-coding-agent |
| Best practices for coding agent | https://docs.github.com/en/copilot/using-github-copilot/best-practices-for-using-copilot-coding-agent |
| About protected branches | https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches |
| Managing branch protection rules | https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/managing-a-branch-protection-rule |
| About rulesets | https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets |
| About code owners | https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners |
| About issues | https://docs.github.com/en/issues/tracking-your-work-with-issues/about-issues |
| GitHub REST API — Pulls | https://docs.github.com/en/rest/pulls |
| GitHub REST API — Git refs | https://docs.github.com/en/rest/git/refs |
