# Lab 06: Guardrails & Accountability — Hands-On Setup

## Goal

Enforce safety with platform controls (not just prompt text): branch protection, CODEOWNERS, a production deployment with an environment approval gate, and a rollback runbook. Verify that every agent action leaves an auditable trail.

---

## Prerequisites

Complete [Lab 05](05-multi-agent-lab.md). Confirm you are on the `main` branch.

```bash
cd gh600-practice
git checkout main && git pull
```

---

## Step 1 — Verify `.github/copilot-instructions.md` stop conditions are in place

Stop conditions tell the agent when it must pause and wait for a human. These were created in Lab 03. Confirm they are present and extend them for guardrail completeness.

```bash
cat .github/copilot-instructions.md
```

If the file exists from Lab 03, add the deployment stop condition:

```bash
cat >> .github/copilot-instructions.md << 'EOF'

## Deployment stop conditions
- Stop before any action that targets the `production` environment.
- Stop if any workflow file in `.github/workflows/` would be created or modified.
- Stop if `CODEOWNERS` would be modified.
- Stop if more than 3 files in `.github/` would be changed in a single run.
EOF

git add .github/copilot-instructions.md
git commit -m "Add deployment stop conditions to copilot-instructions.md (Lab 06)"
git push origin main
```

---

## Step 2 — Harden branch protection (no admin bypass)

The branch protection configured in Lab 01 should already exist. This step adds the admin enforcement setting, which prevents repository owners from bypassing required checks.

**Via GitHub UI:**

1. Go to **Settings** → **Branches** → click **Edit** on the `main` rule.
2. Confirm these are checked:
   - ✅ **Require a pull request before merging** (1 required approval)
   - ✅ **Require status checks to pass** (lint, scope-control, secret-scan)
   - ✅ **Do not allow bypassing the above settings**
3. Also check:
   - ✅ **Restrict who can push to matching branches** → add only yourself or a CI bot
4. Click **Save changes**.

**Verify via CLI:**

```bash
gh api repos/{owner}/{repo}/branches/main/protection \
  --jq '{
    enforce_admins: .enforce_admins.enabled,
    required_reviews: .required_pull_request_reviews.required_approving_review_count,
    required_checks: [.required_status_checks.contexts[]]
  }'
```

Expected output:

```json
{
  "enforce_admins": true,
  "required_reviews": 1,
  "required_checks": ["lint", "scope-control", "secret-scan"]
}
```

---

## Step 3 — Add CODEOWNERS rule for workflow files

Workflow file edits can escalate permissions. Route them to a human reviewer automatically.

```bash
# Confirm the existing CODEOWNERS file and add workflow-specific rule if not present
if ! grep -q ".github/workflows/" .github/CODEOWNERS; then
  cat >> .github/CODEOWNERS << 'EOF'

# All workflow files require explicit owner review
/.github/workflows/ @<your-username>

# Deployment scripts
/scripts/ @<your-username>
EOF
  echo "CODEOWNERS updated."
else
  echo "CODEOWNERS already contains the workflow rule."
fi

cat .github/CODEOWNERS

git add .github/CODEOWNERS
git commit -m "Add workflow and scripts paths to CODEOWNERS (Lab 06)"
git push origin main
```

---

## Step 4 — Create the production environment with a required reviewer

```bash
# 1. Create the production environment
# This must be done via the GitHub UI or API:

gh api repos/{owner}/{repo}/environments/production \
  --method PUT \
  --field wait_timer=0 \
  --field reviewers='[{"type":"User","id":'$(gh api user --jq .id)'}]' \
  --field deployment_branch_policy=null

echo "Production environment created with required reviewer."
```

If the API call above fails due to permissions, use the GitHub UI:

1. Go to **Settings** → **Environments** → **New environment**.
2. Name: `production`.
3. Under **Protection rules** → **Required reviewers** → add your username.
4. Click **Save protection rules**.

---

## Step 5 — Create the guarded deployment workflow

```bash
cat > .github/workflows/deploy-production.yml << 'EOF'
name: deploy-production

on:
  workflow_dispatch:
    inputs:
      version:
        description: "Version tag to deploy (e.g. v1.0.0)"
        required: true

permissions:
  contents: read
  deployments: write
  id-token: write   # required for OIDC short-lived cloud auth

jobs:
  validate:
    name: Pre-deployment validation
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false
      - name: Confirm version tag exists
        run: |
          TAG="${{ github.event.inputs.version }}"
          if git rev-parse "refs/tags/$TAG" >/dev/null 2>&1; then
            echo "Tag $TAG found. Proceeding."
          else
            echo "::error::Tag $TAG does not exist. Create the tag before deploying."
            exit 1
          fi

  deploy-prod:
    name: Deploy to production
    runs-on: ubuntu-latest
    needs: validate
    environment: production      # enforces required-reviewer human approval gate
    permissions:
      contents: read
      deployments: write
      id-token: write
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false

      - name: Record deployment start
        run: |
          echo "Deploying version ${{ github.event.inputs.version }} to production."
          echo "Triggered by: ${{ github.actor }}"
          echo "Run ID: ${{ github.run_id }}"

      - name: Simulate deployment
        run: |
          # Replace this with your actual deployment command.
          # e.g. ./scripts/deploy.sh ${{ github.event.inputs.version }}
          echo "Deployment of ${{ github.event.inputs.version }} complete."

      - name: Record deployment success
        run: |
          echo "Production deployment succeeded."
          echo "Version: ${{ github.event.inputs.version }}"
          echo "SHA: ${{ github.sha }}"
          echo "Actor: ${{ github.actor }}"
          echo "Timestamp: $(date -u +%Y-%m-%dT%H:%M:%SZ)"
EOF

git add .github/workflows/deploy-production.yml
git commit -m "Add guarded production deployment workflow with environment gate (Lab 06)"
git push origin main
```

---

## Step 6 — Trigger a deployment and approve it

```bash
# Create a version tag first (required by the validate job)
git tag v1.0.0
git push origin v1.0.0

# Trigger the deployment workflow
gh workflow run deploy-production.yml --field version=v1.0.0
echo "Waiting for the run to reach the approval gate..."
sleep 15

RUN_ID=$(gh run list --workflow deploy-production.yml --limit 1 --json databaseId --jq '.[0].databaseId')
echo "Run ID: $RUN_ID"
echo "Open: https://github.com/<your-username>/gh600-practice/actions/runs/$RUN_ID"
echo ""
echo "The deploy-prod job is now waiting for your approval in the Actions UI."
echo "Go to the URL above, click the pending deployment, and click 'Approve and deploy'."
```

After you approve:

```bash
gh run watch "$RUN_ID"
# Wait for the run to complete, then check the deployment log
gh run view "$RUN_ID" --log
```

---

## Step 7 — Create the rollback runbook

```bash
mkdir -p runbooks
cat > runbooks/rollback.md << 'EOF'
# Production Rollback Runbook

Use this runbook when a production deployment causes an incident and must be reverted.

---

## Step 1 — Identify the harmful commit

```bash
# List recent commits on main
git log --oneline -10 origin/main

# Or find the deployment run that caused the incident
gh run list --workflow deploy-production.yml --limit 10
```

Note the commit SHA that was deployed (visible in the Actions log of the deployment run).

---

## Step 2 — Create a revert PR (preferred — preserves audit trail)

```bash
BAD_SHA="<paste-the-commit-SHA-here>"

git fetch origin main
git checkout -b revert/incident-$(date +%Y%m%d)
git revert "$BAD_SHA" --no-edit
git push origin revert/incident-$(date +%Y%m%d)

gh pr create \
  --title "Revert: roll back $BAD_SHA (production incident)" \
  --body "## Incident summary
$(describe what went wrong)

## Root cause
$(describe root cause if known)

## Revert scope
Reverts commit $BAD_SHA.

## Verification
$(describe how you will verify the rollback is successful)" \
  --base main
```

Get a human reviewer to approve the revert PR, then merge.

---

## Step 3 — Emergency direct revert (break-glass only)

Use this only if the PR review process is too slow and production is actively degraded.
This requires temporarily relaxing branch protection — document the action.

```bash
# Note: relaxing branch protection requires repository admin access.
# Document this action immediately after completing it.

git fetch origin main
git revert "$BAD_SHA" --no-edit
git push origin main   # requires admin override of branch protection
```

Immediately after: re-enable branch protection and create a post-mortem issue.

---

## Step 4 — Verify the rollback

```bash
# Confirm the revert commit is present
git log --oneline -5 origin/main

# Trigger a fresh deployment of the reverted main
gh workflow run deploy-production.yml --field version=<previous-good-tag>
```

---

## Step 5 — Post-mortem

```bash
gh issue create \
  --title "Post-mortem: production incident $(date +%Y-%m-%d)" \
  --body "## Timeline
(Fill in)

## Root cause
(Fill in)

## Actions taken
- Reverted commit $BAD_SHA via PR #(fill in)
- (other actions)

## Prevention
(What controls are being added to prevent recurrence?)

## Follow-up tasks
- [ ] (Add tasks)
" \
  --label "incident"
```

---

## Accountability checklist

Every production incident response must produce:
- [ ] A revert PR or merge commit with a clear description
- [ ] A workflow log showing who approved the rollback
- [ ] A post-mortem issue linked to the deployment run
- [ ] An update to this runbook or to controls if the process revealed a gap
EOF

git add runbooks/rollback.md
git commit -m "Add production rollback runbook (Lab 06)"
git push origin main
```

---

## Step 8 — Verify the full accountability trail

```bash
# Check deployment history for the production environment
gh api repos/{owner}/{repo}/deployments \
  --jq '[.[] | {id, sha, environment, creator: .creator.login, created_at}] | .[:5]'

# Check the Actions log for the deployment run
RUN_ID=$(gh run list --workflow deploy-production.yml --limit 1 --json databaseId --jq '.[0].databaseId')
gh run view "$RUN_ID" --log | tail -30
```

Every deployment shows: who triggered it, which SHA was deployed, who approved it (in the environment protection review record), and the full execution log.

---

## Expected outcome

After completing this lab your repository will have:

| Control | Status |
| --- | --- |
| `copilot-instructions.md` with deployment stop conditions | ✅ Updated |
| Branch protection — no admin bypass, required checks | ✅ Configured |
| `CODEOWNERS` covering `.github/workflows/` and `/scripts/` | ✅ Updated |
| `production` environment with required reviewer | ✅ Configured |
| `deploy-production.yml` with environment gate and OIDC | ✅ Committed |
| Deployment triggered, approved, and auditable in the log | ✅ Demonstrated |
| `runbooks/rollback.md` with exact revert commands | ✅ Committed |

---

## Verify it works

```bash
# Confirm all key files are in place
ls .github/CODEOWNERS .github/copilot-instructions.md runbooks/rollback.md
ls .github/workflows/deploy-production.yml

# Confirm the production environment exists
gh api repos/{owner}/{repo}/environments --jq '.environments[].name'

# Confirm CODEOWNERS covers workflows
grep "workflows" .github/CODEOWNERS

# Review the deployment log
gh run list --workflow deploy-production.yml --limit 5
```

---

## Teardown

```bash
# Remove the test tag (optional — keep it if you want to re-run the deployment)
git tag -d v1.0.0
git push origin :refs/tags/v1.0.0 2>/dev/null || true
```

---

## Congratulations

You have completed all six labs. Your `gh600-practice` repository now demonstrates every governance pattern tested in the GH-600 exam:

| Lab | Pattern |
| --- | --- |
| 01 | Governed `issue → branch → PR` flow with branch protection |
| 02 | Least-privilege workflows, environment gates, approved MCP config |
| 03 | Durable repo memory, PR checkpoints, artifacts, idempotent retries |
| 04 | Repeatable evaluation framework with scope and secret guardrails |
| 05 | Multi-agent pipeline with serialized jobs and artifact handoffs |
| 06 | Platform-enforced guardrails, deployment approval, rollback runbook |
