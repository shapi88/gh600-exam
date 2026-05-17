# Topic 2: Implement Tool Use and Environment Interaction

## Key Concepts and Sub-Topics

- **Tool selection vs model-only reasoning** — tools extend what an agent can do (read files, call APIs, run commands); use them only when necessary and with the narrowest scope
- **MCP (Model Context Protocol) servers** — provide agents access to external systems; each server expands the attack surface and must be governed
- **Permissions design:**
  - `GITHUB_TOKEN` — short-lived token issued per workflow run; scope it explicitly
  - Fine-grained job-level permissions using the `permissions:` block
  - Environment secrets — only exposed to jobs that target that environment
- **Runner environment design:**
  - Preinstalled tools must be audited and approved
  - Deterministic setup reduces drift and surprises
  - Network boundaries restrict which external endpoints agents can reach
- **Tool failure handling and fallbacks** — distinguish transient failures (retry) from permanent failures (escalate)

---

## Common Pitfalls / Anti-Patterns

- Granting default write permissions to all jobs (GitHub Actions defaults to `contents: write` in some org settings).
- Allowing unrestricted network or tool access when only repository reads are needed.
- Connecting MCP tools without governance, auth review, or input validation.
- Letting prompts choose tools arbitrarily instead of using policy and explicit allowlists.

---

## Step-by-Step Tutorial

**Goal:** Configure a workflow and tool surface that let an agent act without over-permissioning it.

1. **Start from a workflow with no explicit permissions** and identify its risk (implicit broad token scope).

2. **Add a workflow-level `permissions:` block** with only the scopes needed for read or comment actions:
   ```yaml
   permissions:
     contents: read
     pull-requests: write
     checks: read
   ```

3. **Separate read-only jobs from write-capable jobs.** A job that summarizes a diff should not share a runner or token with a job that pushes commits.

4. **Add an environment block** for any deployment-like action:
   ```yaml
   jobs:
     deploy-prod:
       environment: production
   ```
   This allows environment protection rules (required reviewers, wait timers) to gate execution.

5. **Document which MCP servers are approved,** what auth they use, and what systems they can reach. Treat this as part of your repository's security posture.

---

## Practical Examples

### Least-privilege agent workflow

```yaml
name: agent-safe-pr

on:
  workflow_dispatch:
  pull_request:
    types: [opened, synchronize, reopened]

permissions:
  contents: read
  pull-requests: write
  checks: read

jobs:
  review-context:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false
      - name: Summarize changed files
        run: git diff --name-only origin/${{ github.base_ref }}...HEAD
      - name: Enforce diff size
        run: |
          FILE_COUNT=$(git diff --name-only origin/${{ github.base_ref }}...HEAD | wc -l)
          test "$FILE_COUNT" -le 20
```

**Why it is exam-relevant:**
- Explicit permissions
- Limited write scope
- `persist-credentials: false` prevents accidental reuse
- Guardrail on change size

### MCP configuration with approved tools only

```json
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
```

**What to remember:**
- MCP expands capability, so governance must also expand.
- Token scope should match the exact tasks performed.
- Only approved servers should be enabled in enterprise contexts.

---

## Practical Exercises

- Convert a broad-permission workflow into a least-privilege version: remove wildcard scopes and add only the permissions each job actually needs.
- Decide whether a given task should use local repo context, GitHub APIs, or an MCP server — and justify the boundary.
- List three failure cases caused by poor tool configuration and name the specific guardrail for each.
- Given a code-review agent and a deployment agent, write the `permissions:` block for each.

**What a strong answer includes:**
- Explicit token scopes
- Job/environment separation
- Approved tool list
- Auth and network boundaries

---

## Exam-Style Questions

**Q1.** A code-review agent only needs to inspect PR metadata and diffs.  
**A:** Grant `contents: read` and `pull-requests: read` only. No secrets, no write scopes. Least privilege reduces blast radius.

**Q2.** An agent needs production cloud access.  
**A:** Use an environment-gated deployment job plus OIDC (`id-token: write`) or tightly scoped credentials. Static broad secrets are riskier and harder to govern.

**Q3.** An MCP server can reach internal systems and the public internet.  
**A:** Restrict to approved servers, document allowed operations, and validate auth and data boundaries. MCP expands the attack and failure surface.

**Q4.** A workflow uses default token permissions.  
**A:** Replace with explicit job/workflow-level `permissions:`. Exam questions consistently reward least-privilege configuration.

**Q5.** A tool occasionally fails due to rate limits.  
**A:** Add bounded retries for transient failures only and preserve failure logs. Reliability controls must distinguish transient from permanent failures.

---

## Relevant Resources

- [GitHub Docs — Automatic token authentication and `GITHUB_TOKEN`](https://docs.github.com/en/actions/security-for-github-actions/security-guides/automatic-token-authentication)
- [GitHub Docs — Workflow syntax (permissions, environments, jobs)](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [GitHub Docs — Managing environments for deployment](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment)
- [GitHub Docs — About OIDC in Actions](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect)
- [GitHub Docs — About MCP](https://docs.github.com/en/copilot/concepts/about-mcp)
- [GitHub Docs — Extend the coding agent with MCP](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/extend-coding-agent-with-mcp)
- [Official Model Context Protocol docs](https://modelcontextprotocol.io/introduction)

---

## Deep Dive

Go further with **[Deep Dive 2 — Tool Use and Environment Interaction](../deep-dives/02-tool-use-environment-deep.md)**, which covers:
- Complete `GITHUB_TOKEN` permissions matrix with agent-specific guidance
- MCP server governance, threat model, and configuration examples
- `copilot-setup-steps.yml` — what to pre-install and why it prevents tool failures
- Self-hosted vs. GitHub-hosted runners — environment isolation tradeoffs for agents
- Tool failure handling patterns: retry vs. escalate, with workflow examples
- Additional exercises with authoritative GitHub Docs links for each sub-topic
