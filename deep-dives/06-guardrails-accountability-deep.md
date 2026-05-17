# Deep Dive 6: Guardrails and Accountability

> **Companion to:** [Topic 6 — Guardrails and Accountability](../topics/06-guardrails-and-accountability.md)

This file goes beyond the topic summary. It links every concept to the authoritative GitHub documentation section, adds implementation detail, and provides additional exercises for each sub-topic.

---

## 1. GitHub Environments — Required Reviewers, Wait Timers, and Deployment Protection Rules

### GitHub Docs reference
- [Using environments for deployment](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment)
- [Environment protection rules](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment#environment-protection-rules)
- [Reviewing deployments](https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-workflow-runs/reviewing-deployments)
- [Custom deployment protection rules](https://docs.github.com/en/actions/deployment/protecting-deployments/creating-custom-deployment-protection-rules)

### Extended explanation
Environments are the primary enforcement mechanism for separating agents with different trust levels from production systems. Every job that targets an environment must pass its protection rules before starting — these rules are enforced by GitHub's platform, not by prompts.

**Full protection rule matrix:**

| Rule | Configuration | When to use |
| --- | --- | --- |
| Required reviewers | 1–6 named users or teams | Any deployment-affecting or production-touching action |
| Wait timer | 0–43,200 minutes before job starts | Compliance buffer for security review or incident window awareness |
| Deployment branches | `main` only, or `protected` | Prevent agents from deploying from arbitrary or unreviewed branches |
| Custom deployment protection rules | GitHub App | Third-party policy engine (e.g., ticketing system approval, compliance check) |

**Setting up environments via the REST API:**

```bash
# Create a production environment with required reviewer and wait timer
OWNER="your-org"
REPO="your-repo"
USER_ID=12345   # get with: gh api users/{username} --jq .id

gh api "repos/$OWNER/$REPO/environments/production" \
  --method PUT \
  --field wait_timer=10 \
  --field reviewers="[{\"type\":\"User\",\"id\":$USER_ID}]" \
  --field deployment_branch_policy='{"protected_branches":true,"custom_branch_policies":false}'
```

**Hard-stop workflow — reject PRs that touch `.github/workflows/` without a named approver:**

```yaml
name: workflow-file-guard

on:
  pull_request:
    paths:
      - '.github/workflows/**'

permissions:
  pull-requests: write
  contents: read

jobs:
  require-workflow-approval:
    runs-on: ubuntu-latest
    environment: workflow-file-changes   # requires named security team reviewer
    steps:
      - name: Confirm workflow file change is approved
        run: echo "Workflow file change approved by required reviewer. Proceeding."
```

**Key exam point:** The `environment:` key in a job definition is what enforces the gate. Without it, environment protection rules have no effect on that job.

### Additional exercise
Set up a three-environment protection hierarchy for an agent pipeline:
1. `staging` — no required reviewers, wait timer of 5 minutes, main branch only
2. `pre-production` — 1 required reviewer from `@org/qa-team`, no wait timer
3. `production` — 2 required reviewers (1 from `@org/qa-team`, 1 from `@org/security-team`), 10-minute wait timer, protected branches only

Write the three `gh api` calls to configure these environments.

---

## 2. GitHub Audit Log API — Querying Agent-Attributed Events

### GitHub Docs reference
- [Reviewing the audit log for your organization](https://docs.github.com/en/organizations/keeping-your-organization-secure/managing-security-settings-for-your-organization/reviewing-the-audit-log-for-your-organization)
- [Audit log events for your organization](https://docs.github.com/en/organizations/keeping-your-organization-secure/managing-security-settings-for-your-organization/audit-log-events-for-your-organization)
- [REST API — Audit log](https://docs.github.com/en/rest/orgs/orgs#get-the-audit-log-for-an-organization)
- [Streaming the audit log for your organization](https://docs.github.com/en/organizations/keeping-your-organization-secure/managing-security-settings-for-your-organization/streaming-the-audit-log-for-your-organization)

### Extended explanation
The audit log records every security-relevant event in the organization with actor identity, timestamp, and resource. For agent accountability, it provides the immutable record of who did what, when — even for actions taken by automation.

**Key audit log event categories for agents:**

| Event category | Example events | Accountability value |
| --- | --- | --- |
| `repo` | Branch protection changes, repo settings changes | Detect unauthorized configuration changes by agents |
| `workflows` | Workflow runs, approval events | Trace which run triggered a deployment |
| `protected_branch` | Bypass events, rule changes | Flag any agent bypassing branch protection |
| `oauth_application` | Token creation, scope changes | Detect agents acquiring broader permissions |
| `secret_scanning` | Alerts dismissed, push protection bypassed | Detect secrets leaks by agent-generated code |
| `dependabot` | Auto-merge, auto-dismiss | Track automated dependency decisions |

**Querying the audit log for agent actions:**

```bash
# Get all workflow-related audit events from the last 7 days
gh api "orgs/{org}/audit-log" \
  --field phrase="action:workflows" \
  --field include=all \
  --field per_page=100 \
  --jq '.[] | select(.created_at > (now - 604800)) | {action: .action, actor: .actor, repo: .repo, at: (.created_at | todate)}'

# Find all branch protection bypasses (agents should never appear here)
gh api "orgs/{org}/audit-log" \
  --field phrase="action:protected_branch.policy_override" \
  --jq '.[] | {actor: .actor, repo: .repo, at: (.created_at | todate)}'
```

**Audit log streaming:** For compliance-sensitive organizations, stream audit log events to an external SIEM (Splunk, Azure Monitor, Datadog) to enable real-time alerting when agents take sensitive actions. Reference: [Streaming the audit log](https://docs.github.com/en/organizations/keeping-your-organization-secure/managing-security-settings-for-your-organization/streaming-the-audit-log-for-your-organization).

### Additional exercise
Write a scheduled workflow that:
1. Queries the audit log for any branch protection bypass events in the last 24 hours
2. Checks whether the actor in each event is a known human reviewer (vs. a bot/agent identity)
3. Creates a GitHub issue if any bypass was performed by a non-human actor

Use the audit log API and `gh issue create`.

---

## 3. Secret Scanning Push Protection — Blocking Credential Writes at Commit Time

### GitHub Docs reference
- [About secret scanning](https://docs.github.com/en/code-security/secret-scanning/introduction/about-secret-scanning)
- [About push protection](https://docs.github.com/en/code-security/secret-scanning/protecting-pushes-with-secret-scanning)
- [Enabling push protection for your repository](https://docs.github.com/en/code-security/secret-scanning/protecting-pushes-with-secret-scanning#enabling-push-protection-for-a-repository)
- [Secret scanning patterns](https://docs.github.com/en/code-security/secret-scanning/secret-scanning-patterns)

### Extended explanation
Push protection is the strongest defense against agents accidentally committing secrets. It operates at the git receive layer — before the commit reaches the repository — so the secret never appears in git history at all.

**How push protection works with agents:**
1. An agent generates a commit containing a string that matches a known secret pattern (e.g., a GitHub PAT, AWS key, or Slack webhook URL).
2. On `git push`, GitHub's push protection service scans the incoming diff.
3. If a match is found, the push is **rejected** with an error explaining which secret type was detected and where.
4. The agent (or the human who triggered the agent) must remove the secret before the push is accepted.

**Enabling push protection via the API:**
```bash
# Enable secret scanning push protection on a repository
gh api repos/{owner}/{repo} \
  --method PATCH \
  --field security_and_analysis='{"secret_scanning_push_protection":{"status":"enabled"}}'
```

**Secret scanning alert API — monitoring for bypassed alerts:**
```bash
# List open secret scanning alerts (bypasses in history)
gh api repos/{owner}/{repo}/secret-scanning/alerts \
  --field state=open \
  --jq '.[] | {type: .secret_type_display_name, location: .locations_url, created: .created_at}'

# List resolved/dismissed alerts (review for inappropriate dismissals)
gh api repos/{owner}/{repo}/secret-scanning/alerts \
  --field state=resolved \
  --field resolution=used_in_tests \
  --jq '.[] | {type: .secret_type_display_name, dismissed_by: .resolved_by.login}'
```

**Stop condition in `copilot-instructions.md` for secret safety:**
```markdown
## Security rules
- Never include tokens, passwords, API keys, or any credential in code, comments, or documentation.
- If a task requires a credential, read it from `secrets.*` at runtime — never hardcode it.
- If you cannot complete a task without embedding a credential, stop and request human guidance.
```

### Additional exercise
A repository has push protection enabled. An agent's generated code accidentally includes a string `ghp_abc123...` (a GitHub PAT format). Walk through:
1. What happens when the agent attempts to push this commit
2. How the agent should be configured to handle push rejection gracefully
3. What audit log event is generated
4. How to confirm the secret never reached the repository's git history

---

## 4. Dependabot and Code Scanning Auto-Triage Workflows

### GitHub Docs reference
- [About Dependabot alerts](https://docs.github.com/en/code-security/dependabot/dependabot-alerts/about-dependabot-alerts)
- [Configuring Dependabot security updates](https://docs.github.com/en/code-security/dependabot/dependabot-security-updates/configuring-dependabot-security-updates)
- [About Dependabot auto-triage rules](https://docs.github.com/en/code-security/dependabot/dependabot-auto-triage-rules/about-dependabot-auto-triage-rules)
- [REST API — Dependabot alerts](https://docs.github.com/en/rest/dependabot/alerts)

### Extended explanation
Dependabot and code scanning generate alerts that agents can triage, label, and escalate — but agents must not dismiss security alerts without human approval.

**Dependabot auto-triage rules:** GitHub supports configurable rules that automatically dismiss low-signal Dependabot alerts (e.g., dev-only dependencies with no path to production). These rules are set at the organization level and apply automatically — no agent involvement needed. Use them to reduce noise before agents see the alert queue.

**Agent-assisted Dependabot triage workflow:**

```yaml
name: dependabot-triage

on:
  schedule:
    - cron: '0 8 * * 1'  # every Monday morning

permissions:
  security-events: read
  issues: write

jobs:
  triage:
    runs-on: ubuntu-latest
    steps:
      - name: List critical Dependabot alerts
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          CRITICAL=$(gh api repos/${{ github.repository }}/dependabot/alerts \
            --field severity=critical \
            --field state=open \
            --jq 'length')

          echo "Open critical Dependabot alerts: $CRITICAL"

          if [ "$CRITICAL" -gt 0 ]; then
            DETAILS=$(gh api repos/${{ github.repository }}/dependabot/alerts \
              --field severity=critical \
              --field state=open \
              --jq '[.[] | {package: .dependency.package.name, severity: .security_advisory.severity, cve: .security_advisory.cve_id}]')

            gh issue create \
              --title "Security triage: $CRITICAL critical Dependabot alert(s) open" \
              --body "## Critical Dependabot Alerts

            $(echo $DETAILS | python3 -c 'import json,sys; [print(f"- **{a[\"package\"]}**: {a[\"cve\"]} ({a[\"severity\"]})") for a in json.load(sys.stdin)]')

            These require human review before dismissal." \
              --label "security,triage"
          fi
```

**What agents may do vs. may not do with security alerts:**

| Action | Agent may do | Requires human |
| --- | --- | --- |
| List and count alerts | ✅ | |
| Label/comment on triage issues | ✅ | |
| Propose a fix PR for a Dependabot alert | ✅ | |
| Dismiss an alert as false positive | ❌ | ✅ |
| Merge a Dependabot PR to production | ❌ | ✅ |
| Change alert severity | ❌ | ✅ |

---

## 5. Copilot Policy Settings at Org/Enterprise Level

### GitHub Docs reference
- [Managing policies for GitHub Copilot in your organization](https://docs.github.com/en/copilot/managing-copilot/managing-github-copilot-in-your-organization/managing-policies-for-copilot-in-your-organization)
- [Granting access to GitHub Copilot for members of your organization](https://docs.github.com/en/copilot/managing-copilot/managing-github-copilot-in-your-organization/granting-access-to-copilot-for-members-of-your-organization)
- [Reviewing GitHub Copilot activity in your organization](https://docs.github.com/en/copilot/managing-copilot/managing-github-copilot-in-your-organization/reviewing-activity-related-to-github-copilot-in-your-organization)
- [About enterprise policies for GitHub Copilot](https://docs.github.com/en/copilot/managing-copilot/managing-copilot-for-your-enterprise/managing-policies-and-features-for-copilot-in-your-enterprise)

### Extended explanation
Organization administrators control which Copilot features are available to agents — and this governance layer is above the repository level. Even a perfectly configured repository cannot use a feature that has been disabled at the org or enterprise level.

**Key policy controls for agentic AI workflows:**

| Policy | Scope | Impact on agents |
| --- | --- | --- |
| Copilot coding agent enabled/disabled | Org/Enterprise | Determines whether issues can be assigned to Copilot at all |
| Allowed MCP servers | Org | Restricts which MCP servers agents can connect to |
| Copilot in GitHub Actions | Org | Controls whether Copilot can be used within workflow steps |
| Seat assignment | Org | Only licensed members can trigger agent runs |
| Content exclusions | Org/Repo | Specific files or directories excluded from Copilot context |
| Suggestions from public code | Org | Controls whether suggestions include matches to public code |

**Content exclusions — protecting sensitive files from agent context:**
```yaml
# .copilotignore or organization content exclusion config
# Files that must never enter Copilot's context window
- "**/.env"
- "**/secrets/**"
- "**/*.pem"
- "**/credentials/**"
```

Reference: [Configuring content exclusions for GitHub Copilot](https://docs.github.com/en/copilot/managing-copilot/configuring-and-auditing-content-exclusion/configuring-content-exclusions-for-github-copilot).

**Reviewing Copilot activity:** Organization admins can access usage metrics showing which seats are active, which repositories are using Copilot, and aggregate usage trends. This supports compliance reporting for enterprises with data governance requirements.

### Additional exercise
A new regulation requires your organization to ensure that agents never process files containing PII. Design the governance configuration:
1. Which content exclusion patterns should be set at the organization level?
2. Which `copilot-instructions.md` rules should reinforce the content exclusion?
3. How do you audit that the exclusion is working as expected?
4. What is the fallback if an agent still encounters a PII-containing file?

---

## 6. Hard-Stop Workflow — Rejecting PRs That Touch Workflow Files Without Named Approver

### GitHub Docs reference
- [About CODEOWNERS](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners)
- [Managing environments for deployment](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment)

### Complete implementation

The following combination of CODEOWNERS, branch protection, and an environment gate creates a hard stop for workflow file changes:

**Step 1 — CODEOWNERS: route `.github/workflows/` changes to security team**
```gitignore
# .github/CODEOWNERS
.github/workflows/  @org/security-team @org/platform-leads
```

**Step 2 — Branch protection: require CODEOWNERS review for `main`**
```bash
gh api repos/{owner}/{repo}/branches/main/protection \
  --method PUT \
  --field required_pull_request_reviews='{
    "required_approving_review_count": 1,
    "require_code_owner_reviews": true,
    "dismiss_stale_reviews": true
  }' \
  --field required_status_checks='{"strict":true,"contexts":["workflow-file-guard"]}' \
  --field enforce_admins=true \
  --field restrictions=null
```

**Step 3 — Workflow guard: environment gate for PRs touching workflow files**
```yaml
name: workflow-file-guard

on:
  pull_request:
    paths:
      - '.github/workflows/**'
      - '.github/CODEOWNERS'
      - '.github/copilot-instructions.md'

permissions:
  pull-requests: write
  statuses: write

jobs:
  alert-and-gate:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
    steps:
      - name: Flag workflow file change on PR
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          gh pr comment ${{ github.event.pull_request.number }} \
            --body "⚠️ **Workflow file guard:** This PR modifies files in \`.github/workflows/\` or other governed paths.

          Required before merge:
          - [ ] Review by \`@org/security-team\` (auto-requested via CODEOWNERS)
          - [ ] Confirm no privilege escalation in modified workflows
          - [ ] Confirm no MCP server additions without governance review"

  require-security-approval:
    needs: alert-and-gate
    runs-on: ubuntu-latest
    environment: workflow-file-changes   # requires @org/security-team approval
    steps:
      - name: Security team approved workflow change
        run: echo "Workflow file change approved. Status check will pass."
```

**Why this works:**
- CODEOWNERS auto-requests security team review (but review alone is not sufficient).
- The `environment: workflow-file-changes` gate requires an explicit approval action from a required reviewer *in addition* to the CODEOWNERS review.
- Branch protection requires `workflow-file-guard` status to pass, which only passes after the environment gate is cleared.
- No agent — regardless of permissions — can merge a PR that touches workflow files without a human approval.

---

## Relevant Resources

| Resource | URL |
| --- | --- |
| Using environments for deployment | https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment |
| Custom deployment protection rules | https://docs.github.com/en/actions/deployment/protecting-deployments/creating-custom-deployment-protection-rules |
| Reviewing the audit log | https://docs.github.com/en/organizations/keeping-your-organization-secure/managing-security-settings-for-your-organization/reviewing-the-audit-log-for-your-organization |
| Audit log events | https://docs.github.com/en/organizations/keeping-your-organization-secure/managing-security-settings-for-your-organization/audit-log-events-for-your-organization |
| Streaming the audit log | https://docs.github.com/en/organizations/keeping-your-organization-secure/managing-security-settings-for-your-organization/streaming-the-audit-log-for-your-organization |
| About push protection | https://docs.github.com/en/code-security/secret-scanning/protecting-pushes-with-secret-scanning |
| About Dependabot alerts | https://docs.github.com/en/code-security/dependabot/dependabot-alerts/about-dependabot-alerts |
| Dependabot auto-triage rules | https://docs.github.com/en/code-security/dependabot/dependabot-auto-triage-rules/about-dependabot-auto-triage-rules |
| REST API — Dependabot alerts | https://docs.github.com/en/rest/dependabot/alerts |
| Managing Copilot policies for your org | https://docs.github.com/en/copilot/managing-copilot/managing-github-copilot-in-your-organization/managing-policies-for-copilot-in-your-organization |
| Configuring content exclusions | https://docs.github.com/en/copilot/managing-copilot/configuring-and-auditing-content-exclusion/configuring-content-exclusions-for-github-copilot |
| About CODEOWNERS | https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners |
| About protected branches | https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches |
