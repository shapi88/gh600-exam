# Deep Dive 2: Tool Use and Environment Interaction

> **Companion to:** [Topic 2 — Tool Use and Environment Interaction](../topics/02-tool-use-and-environment-interaction.md)

This file goes beyond the topic summary. It links every concept to the authoritative GitHub documentation section, adds implementation detail, and provides additional exercises for each sub-topic.

---

## 1. GitHub Actions Permission Scopes — Complete Reference

### GitHub Docs reference
- [Automatic token authentication](https://docs.github.com/en/actions/security-for-github-actions/security-guides/automatic-token-authentication)
- [Workflow syntax — permissions](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#permissions)
- [Security hardening for GitHub Actions](https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions)

### Extended explanation
`GITHUB_TOKEN` is a short-lived credential generated at the start of every workflow run. Its default permissions depend on the organization/repository settings, which makes the exam pattern clear: **always declare permissions explicitly** rather than relying on defaults.

**Complete permissions matrix for agents:**

| Scope | Read | Write | When agents need write |
| --- | --- | --- | --- |
| `contents` | Read repo files | Push commits, create tags | Executor agents that commit code |
| `pull-requests` | Read PR metadata | Open PRs, post reviews, merge | Coordinator agents that summarize and surface results |
| `issues` | Read issues | Create/comment/label issues | Planner agents that create follow-up issues |
| `checks` | Read check results | Create check runs | CI agents that report custom quality gates |
| `statuses` | Read commit statuses | Create commit statuses | Legacy CI status integrations |
| `id-token` | — | Request OIDC JWT | Deployment agents using cloud OIDC auth |
| `deployments` | Read deployments | Create deployment records | Release automation |
| `secrets` | — | — | **Never accessible via token — use `secrets.*`** |
| `actions` | Read workflow runs | Cancel/re-run workflows | Advanced orchestration only |
| `packages` | Read packages | Publish packages | Release automation |

**Key exam patterns:**
1. A code-review agent that only reads diffs needs `contents: read, pull-requests: read` — nothing else.
2. An agent that posts a PR comment needs `pull-requests: write` — not `contents: write`.
3. Setting `permissions: {}` (empty object) gives the token zero permissions — useful for jobs that need no GitHub API access.

### Least-privilege exercise

Rewrite this overly permissive workflow using job-level least-privilege scopes:

```yaml
# BEFORE — avoid this
name: agent-pr-review
on:
  pull_request:
permissions: write-all
jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Summarize diff
        run: git diff origin/main...HEAD --stat
      - name: Post comment
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: gh pr comment ${{ github.event.pull_request.number }} --body "Review complete"
```

**Answer — after:**

```yaml
name: agent-pr-review
on:
  pull_request:
permissions: {}          # deny all at workflow level
jobs:
  review:
    runs-on: ubuntu-latest
    permissions:
      contents: read         # checkout only
      pull-requests: write   # post comment only
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false
      - name: Summarize diff
        run: git diff origin/${{ github.base_ref }}...HEAD --stat
      - name: Post comment
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: gh pr comment ${{ github.event.pull_request.number }} --body "Review complete"
```

---

## 2. MCP Server Configuration for GitHub Copilot Agents

### GitHub Docs reference
- [About MCP](https://docs.github.com/en/copilot/concepts/about-mcp)
- [Extending the coding agent with MCP](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/extend-coding-agent-with-mcp)
- [Managing MCP servers for your organization](https://docs.github.com/en/copilot/managing-copilot/managing-github-copilot-in-your-organization/managing-mcp-servers)

### Extended explanation
Model Context Protocol (MCP) lets agents call external tools through a standardized interface. GitHub hosts a built-in MCP server for GitHub itself (file access, PR operations, issue management) and allows organizations to connect custom MCP servers.

**MCP threat model — what must be governed:**

| Risk | Governance control |
| --- | --- |
| MCP server can exfiltrate repository content | Restrict to approved servers list in org settings |
| MCP server receives unsanitized agent input (prompt injection) | Validate and sanitize all inputs at the MCP server boundary |
| MCP server has broader API access than the workflow token | Audit the OAuth scopes/PAT used by the MCP server |
| Unapproved server installed by an individual developer | Organization-level MCP server allowlist policy |

**GitHub-hosted MCP server vs. user-configured servers:**
- The GitHub MCP server is provisioned automatically for the coding agent and uses the `GITHUB_TOKEN` — no additional credentials needed.
- User-configured MCP servers (e.g., a Jira or Slack server) require explicit configuration in the repository's Copilot settings and credentials provided through secrets.

**Configuration in `.github/copilot-mcp.json` (coding agent MCP config):**

```json
{
  "mcpServers": {
    "github": {
      "type": "github",
      "tools": ["get_file_contents", "create_pull_request", "search_code"]
    },
    "internal-docs": {
      "type": "http",
      "url": "https://internal-mcp.example.com",
      "headers": {
        "Authorization": "Bearer ${input:internal_mcp_token}"
      },
      "tools": ["search_docs", "get_doc_page"]
    }
  }
}
```

### Additional exercise
Given an agent that needs to:
- Search the codebase for a function definition
- Read a documentation page from an internal wiki
- Post a summary comment on a PR

Write the minimal MCP configuration and the corresponding `permissions:` block for the workflow. Justify each tool and each scope.

---

## 3. Copilot Setup Steps — Pre-Installing Tools Before Agent Execution

### GitHub Docs reference
- [Customizing the development environment for Copilot coding agent](https://docs.github.com/en/copilot/using-github-copilot/customizing-copilot-coding-agent)

### Extended explanation
`copilot-setup-steps.yml` is a special GitHub Actions workflow file that runs *before* the coding agent starts its task. It installs dependencies, sets environment variables, and configures tools that the agent will need.

**Why this matters for agents:**
- If the agent tries to run `pytest` and Python test dependencies are not installed, the agent will encounter a tool failure and may retry, hallucinate a workaround, or stop.
- Pre-installing tools deterministically eliminates a large class of transient "environment not ready" failures.
- The setup steps run in the same runner environment as the agent, so any file or environment variable set here is available to the agent.

**Minimal example — Python project:**

```yaml
# .github/workflows/copilot-setup-steps.yml
name: "Copilot Setup Steps"
on:
  workflow_dispatch:

jobs:
  copilot-setup-steps:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: pip

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Install dev tools
        run: pip install pytest ruff
```

**What to include vs. exclude:**
- ✅ Include: language runtimes, test frameworks, linters, build tools
- ✅ Include: `apt` packages the agent's scripts may need
- ❌ Exclude: secrets — use `secrets.*` at runtime, not in setup steps
- ❌ Exclude: network calls to external services with auth — these belong in the agent task itself

### Additional exercise
A Node.js project's agent consistently fails because `eslint` is not available. Write the complete `copilot-setup-steps.yml` that:
1. Sets up Node.js 20 with npm cache
2. Runs `npm ci` to install dependencies
3. Verifies `eslint` is available by running `npx eslint --version`

---

## 4. Self-Hosted vs. GitHub-Hosted Runners — Environment Isolation

### GitHub Docs reference
- [About GitHub-hosted runners](https://docs.github.com/en/actions/using-github-hosted-runners/using-github-hosted-runners/about-github-hosted-runners)
- [About self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners)
- [Security guides for self-hosted runners](https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions#hardening-for-self-hosted-runners)

### Extended explanation
For agent workflows, the choice of runner type directly affects the trust model:

| Property | GitHub-hosted runner | Self-hosted runner |
| --- | --- | --- |
| Isolation | Fresh VM per run; fully ephemeral | Persistent; state can leak between runs |
| Pre-installed tools | Standard image (ubuntu-latest, windows-latest, macos-latest) | Fully customizable |
| Network access | Public internet + GitHub APIs | Configurable; can reach internal networks |
| Secret exposure | Token scoped to the run | Persistent runner process can retain tokens in memory |
| Agent suitability | **Preferred for most agent workflows** — strong isolation | Use only when internal network access is required |

**Critical security note for self-hosted runners with agents:** If a self-hosted runner processes both trusted and untrusted code (e.g., PRs from external contributors), a compromised workflow can persist state on the runner between runs. Always use GitHub-hosted runners (or ephemeral self-hosted runners) for agent workflows that process untrusted input.

**Larger runners:** GitHub-hosted larger runners (more CPU/RAM) are available for compute-intensive agent tasks. Reference: [Using larger runners](https://docs.github.com/en/actions/using-github-hosted-runners/using-larger-runners).

### Additional exercise
A company wants agents to access an internal npm registry and a private database schema. Design the runner strategy:
1. Should the agent use a GitHub-hosted or self-hosted runner? Justify.
2. What network controls should be in place on the runner?
3. How do you prevent state from leaking between two different agent runs on the same runner?

---

## 5. Tool Failure Handling and Fallbacks — Deep Dive

### GitHub Docs reference
- [Workflow syntax — `continue-on-error`](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepscontinue-on-error)
- [Workflow syntax — `retry` (via `if` and step outcomes)](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsif)

### Extended explanation
Agent tool failures fall into two categories with different responses:

| Failure type | Examples | Correct response |
| --- | --- | --- |
| Transient | Rate limit, network timeout, runner resource contention | Retry with backoff, bounded retry count |
| Permanent | Invalid credentials, missing required resource, policy block | Do not retry; escalate to human or fail with clear error |

**Retry pattern in GitHub Actions:**

```yaml
- name: Call external API with retry
  id: api_call
  run: |
    for i in 1 2 3; do
      if curl -sf "https://api.example.com/data" -o /tmp/result.json; then
        echo "succeeded on attempt $i"
        exit 0
      fi
      echo "Attempt $i failed. Retrying in $((i * 10))s..."
      sleep $((i * 10))
    done
    echo "::error::API call failed after 3 attempts"
    exit 1
```

**Stop and escalate pattern — agent stop condition:**

```yaml
- name: Check policy block
  run: |
    BLOCKED=$(gh api repos/${{ github.repository }}/branches/main/protection \
      --jq '.required_pull_request_reviews.required_approving_review_count')
    if [ "$BLOCKED" -gt 0 ]; then
      echo "::notice::Branch protection requires human review. Agent stopping."
      echo "STOP_REASON=policy_block" >> "$GITHUB_ENV"
      exit 0   # graceful exit — not a failure
    fi
```

### Additional exercise
Design a tool failure matrix for a planner → executor pipeline. For each failure below, specify: retry (Y/N), retry count, escalation action:

| Failure | Retry? | Max retries | Escalation |
| --- | --- | --- | --- |
| GitHub API rate limit (429) | | | |
| Missing secret (env var empty) | | | |
| `npm test` flaky failure | | | |
| Branch protection blocks merge | | | |
| MCP server connection timeout | | | |

---

## Relevant Resources

| Resource | URL |
| --- | --- |
| Automatic token authentication | https://docs.github.com/en/actions/security-for-github-actions/security-guides/automatic-token-authentication |
| Workflow syntax — permissions | https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#permissions |
| Security hardening for GitHub Actions | https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions |
| About MCP | https://docs.github.com/en/copilot/concepts/about-mcp |
| Extending coding agent with MCP | https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/extend-coding-agent-with-mcp |
| Managing MCP servers for your org | https://docs.github.com/en/copilot/managing-copilot/managing-github-copilot-in-your-organization/managing-mcp-servers |
| Customizing the coding agent environment | https://docs.github.com/en/copilot/using-github-copilot/customizing-copilot-coding-agent |
| About GitHub-hosted runners | https://docs.github.com/en/actions/using-github-hosted-runners/using-github-hosted-runners/about-github-hosted-runners |
| About self-hosted runners | https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners |
| Using larger runners | https://docs.github.com/en/actions/using-github-hosted-runners/using-larger-runners |
| Managing environments for deployment | https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment |
| About OIDC in Actions | https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect |
