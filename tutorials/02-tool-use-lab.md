# Lab 02: Tool Use & Environment Interaction — Hands-On Setup

## Goal

Harden a GitHub Actions workflow from broad default permissions to an explicit least-privilege configuration, then add an environment gate for deployment-like jobs and document an approved MCP server configuration.

---

## Prerequisites

Complete [Lab 01](01-agent-architecture-lab.md). You should be inside the `gh600-practice` repository with the CI workflow already committed.

```bash
cd gh600-practice
git checkout main && git pull
```

---

## Step 1 — Observe the risk of a no-`permissions` workflow

Create a deliberately broad workflow to see what the default token allows:

```bash
cat > /tmp/broad-workflow.yml << 'EOF'
name: broad-permissions-example

on:
  workflow_dispatch:

# No permissions block — the token inherits the repository default,
# which is often "contents: write" and can include other broad scopes.

jobs:
  example:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Show token scopes (for demonstration only)
        run: |
          echo "GITHUB_TOKEN is available but its scope is unspecified."
          echo "In many org configurations this defaults to contents:write."
          echo "An agent operating under this token could push commits."
EOF
cat /tmp/broad-workflow.yml
```

> **Exam point:** Default permissions vary by organisation settings. Always set them explicitly; never rely on the default.

---

## Step 2 — Build the least-privilege version

Replace the broad workflow with an explicit, minimal-scope version:

```bash
mkdir -p .github/workflows
cat > .github/workflows/agent-safe-pr.yml << 'EOF'
name: agent-safe-pr

on:
  workflow_dispatch:
  pull_request:
    types: [opened, synchronize, reopened]

# Workflow-level: most permissive scope any job in this file may need.
# Jobs that need less should override with their own permissions block.
permissions:
  contents: read
  pull-requests: write
  checks: read

jobs:
  # ── Job 1: Read-only analysis ─────────────────────────────────────────────
  review-context:
    runs-on: ubuntu-latest
    permissions:
      contents: read      # this job only reads
    outputs:
      file_count: ${{ steps.count.outputs.file_count }}
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false   # prevent accidental credential reuse
      - name: List changed files
        run: git diff --name-only origin/${{ github.base_ref }}...HEAD || echo "(not a PR context)"
      - name: Count changed files and enforce limit
        id: count
        run: |
          COUNT=$(git diff --name-only origin/${{ github.base_ref }}...HEAD 2>/dev/null | wc -l || echo 0)
          echo "file_count=$COUNT" >> "$GITHUB_OUTPUT"
          echo "Changed file count: $COUNT"
          if [ "$COUNT" -gt 20 ]; then
            echo "::error::Diff exceeds 20 files ($COUNT). Reduce scope before proceeding."
            exit 1
          fi

  # ── Job 2: Write action — separated from the read job ─────────────────────
  post-summary:
    runs-on: ubuntu-latest
    needs: review-context
    if: github.event_name == 'pull_request'
    permissions:
      pull-requests: write   # only this job needs write access
      contents: read
    steps:
      - name: Post file-count summary to PR
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          FILE_COUNT: ${{ needs.review-context.outputs.file_count }}
        run: |
          gh pr comment "${{ github.event.pull_request.number }}" \
            --body "🔍 **Diff size check:** ${FILE_COUNT} file(s) changed (limit: 20)." \
            --repo "${{ github.repository }}"
EOF
```

Commit and push:

```bash
git add .github/workflows/agent-safe-pr.yml
git commit -m "Add least-privilege agent-safe PR workflow (Lab 02)"
git push origin main
```

---

## Step 3 — Add an environment gate for deployment-like jobs

1. **Create the staging environment in the GitHub UI:**
   - Go to **Settings** → **Environments** → **New environment**.
   - Name it `staging`.
   - Under **Protection rules**, check **Required reviewers** and add yourself.
   - Click **Save protection rules**.

2. **Create a deployment workflow that targets the environment:**

```bash
cat > .github/workflows/deploy-staging.yml << 'EOF'
name: deploy-staging

on:
  workflow_dispatch:

permissions:
  contents: read
  deployments: write

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: staging      # enforces the required-reviewer gate
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false
      - name: Simulate deployment
        run: |
          echo "Deploying to staging environment..."
          echo "Deployment complete."
EOF

git add .github/workflows/deploy-staging.yml
git commit -m "Add staging deployment workflow with environment gate (Lab 02)"
git push origin main
```

3. **Trigger the workflow and observe the approval gate:**

```bash
gh workflow run deploy-staging.yml
# After ~10 seconds, open the run URL and look for "Waiting for review"
gh run list --workflow deploy-staging.yml --limit 3
```

The run will pause at the `deploy` job and show **"Waiting for review"** until you (the required reviewer) approve it in the Actions UI.

---

## Step 4 — Trigger the agent-safe-pr workflow manually

```bash
gh workflow run agent-safe-pr.yml
gh run list --workflow agent-safe-pr.yml --limit 3
```

To see both jobs in action, open a PR against `main` from a test branch:

```bash
git checkout -b test/lab02-permissions
echo "# Lab 02 test file" > lab02-test.md
git add lab02-test.md
git commit -m "Test file for Lab 02 workflow"
git push origin test/lab02-permissions

gh pr create --title "Lab 02: permissions test" --body "Testing the agent-safe-pr workflow." --base main
```

Watch the Actions tab — you will see `review-context` run first, then `post-summary` after it completes.

---

## Step 5 — Document an approved MCP server configuration

MCP (Model Context Protocol) servers extend agent capabilities. Governance requires an explicit approved list. Create the configuration file used by Copilot/VS Code:

```bash
mkdir -p .vscode
cat > .vscode/mcp.json << 'EOF'
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${input:github_pat}"
      }
    },
    "docs": {
      "command": "python",
      "args": ["-m", "docs_mcp_server"],
      "cwd": "${workspaceFolder}/tools/docs-mcp"
    }
  }
}
EOF

# Document governance expectations alongside the config
cat > .vscode/mcp-governance.md << 'EOF'
# Approved MCP Servers

This file documents which MCP servers are approved for use in this repository, why they are approved, and what controls govern them.

| Server name | Command | Purpose | Auth mechanism | Approved by |
| --- | --- | --- | --- | --- |
| `github` | `npx @modelcontextprotocol/server-github` | Read and write GitHub issues, PRs, and code | Fine-grained PAT via VS Code input variable | Repo owner |
| `docs` | `python -m docs_mcp_server` | Query internal documentation | Local process, no external network | Repo owner |

## What is NOT approved

- Any MCP server that reaches production databases
- Any server with broad `repo` or `admin:org` GitHub token scopes
- Servers without documented auth and network boundaries
EOF

git add .vscode/
git commit -m "Add approved MCP server config and governance doc (Lab 02)"
git push origin main
```

---

## Expected outcome

After completing this lab your repository will have:

| Control | Status |
| --- | --- |
| `agent-safe-pr.yml` with explicit `permissions:` blocks | ✅ Committed |
| Read job and write job separated by `needs:` | ✅ Configured |
| `persist-credentials: false` on all `checkout` steps | ✅ Configured |
| `deploy-staging.yml` with `environment: staging` | ✅ Committed |
| `staging` environment with required reviewer | ✅ Configured |
| Approved MCP config in `.vscode/mcp.json` | ✅ Committed |
| MCP governance doc | ✅ Committed |

---

## Verify it works

```bash
# Confirm workflow permissions are explicit
grep -A5 "permissions:" .github/workflows/agent-safe-pr.yml

# Confirm persist-credentials is false
grep "persist-credentials" .github/workflows/agent-safe-pr.yml

# Confirm environment gate exists
grep "environment:" .github/workflows/deploy-staging.yml

# Confirm staging environment was created
gh api repos/{owner}/{repo}/environments --jq '.environments[].name'
```

---

## Teardown

```bash
git checkout main
git branch -d test/lab02-permissions
gh pr close --search "Lab 02: permissions test" 2>/dev/null || true
```

---

## Next step

Proceed to [Lab 03 — Memory, State & Execution](03-memory-state-lab.md).
