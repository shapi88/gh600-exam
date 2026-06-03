# Copilot Configuration Manual

> **Purpose:** A complete, step-by-step guide to configuring all GitHub Copilot capabilities and agentic SDLC controls from scratch in any repository.
> Follow the steps in order. Each step links to the relevant template file in this folder.

---

## Prerequisites

Before starting, confirm you have:

- [ ] A GitHub repository with `main` as the default branch.
- [ ] Admin or owner access to the repository.
- [ ] GitHub Copilot enabled for your account or organisation (Settings → Copilot).
- [ ] GitHub CLI (`gh`) installed and authenticated (`gh auth login`).
- [ ] The files in this `templates/` folder available locally (clone or download this repo).

---

## Overview — Setup steps

Steps 1–5 are required for a fully guardrailed agent. Steps 6 and 7 are strongly recommended but can be added at any time.

| Step | What you configure | Required? | Files created |
| --- | --- | --- | --- |
| 1 | Agent standing orders + persistent memory | ✅ Required | `.github/copilot-instructions.md`, `.github/copilot-memory.md` |
| 2 | Ownership + branch protection | ✅ Required | `CODEOWNERS`, `.github/branch-protection.md` |
| 3 | MCP server registry + agent skills | ✅ Required | `.github/mcp-config.json`, `.github/skills/*.yml` |
| 4 | Guardrail workflows + incident runbook | ✅ Required | `.github/workflows/`, `.github/guardrails.md`, `.github/INCIDENT_RESPONSE.md` |
| 5 | GitHub UI: environment protection gate | ✅ Required | Settings → Environments → `agent-execution` |
| 6 | Issue and PR templates | Recommended | `.github/ISSUE_TEMPLATE/`, `.github/PULL_REQUEST_TEMPLATE.md` |
| 7 | Copilot setup steps workflow | Optional | `.github/workflows/copilot-setup-steps.yml` |

---

## Step 1 — Agent Standing Orders and Memory

### 1a. Create `.github/copilot-instructions.md`

This file is your agent's constitution. Every agentic workflow reads it before acting.

```bash
mkdir -p .github
```

Copy [`templates/copilot-instructions-example.md`](copilot-instructions-example.md) and customise it:

```bash
cp templates/copilot-instructions-example.md .github/copilot-instructions.md
```

**Mandatory fields to replace:**

| Placeholder | Replace with |
| --- | --- |
| `@shapi88` | Your GitHub username or team slug |
| `copilot/` branch prefix | Your preferred branch naming convention |
| Autonomy level | `1` (suggest only) through `4` (deployment-affecting); start at `2` |
| MAY NOT list | Add any repo-specific prohibitions |
| Stop conditions | Add triggers specific to your project |

> **Rule:** This file must be listed in `CODEOWNERS` so that only designated humans can modify it.

### 1b. Create `.github/copilot-memory.md`

This file provides persistent context that survives across separate conversations.

Copy [`templates/copilot-memory-example.md`](copilot-memory-example.md) and customise it:

```bash
cp templates/copilot-memory-example.md .github/copilot-memory.md
```

**Mandatory sections to fill in:**

- **File Map** — list the key directories and what they contain.
- **Conventions** — naming rules, code style, test framework, branch prefix.
- **Approved MCP Servers** — mirror the allowlist from `.github/mcp-config.json`.
- **State Checkpoints** — the 5 standard checkpoints (pre-write, post-edit, pre-PR, post-PR, pre-deploy).

---

## Step 2 — Ownership and Branch Protection

### 2a. Create `CODEOWNERS`

`CODEOWNERS` is the primary platform control. It ensures that changes to governance files require human approval even if an agent opens the PR.

Create `.github/CODEOWNERS` (or `CODEOWNERS` at the root):

```
# Governance files — require human review for any change
.github/copilot-instructions.md   @<your-username>
.github/copilot-memory.md         @<your-username>
.github/guardrails.md             @<your-username>
.github/mcp-config.json           @<your-username>
CODEOWNERS                        @<your-username>
.github/workflows/                @<your-username>
```

Replace `@<your-username>` with your GitHub username or a team slug (e.g., `@your-org/security-team`).

### 2b. Document branch protection in `.github/branch-protection.md`

This file is an auditable record of the branch protection rules you configure in the GitHub UI. It is not machine-enforced on its own — apply the settings in GitHub Settings → Branches.

Copy [`templates/guardrails-example.md`](guardrails-example.md) sections or use the text below as a starting point:

```markdown
# Branch Protection — main

Configured: 2026-01-15 by @<your-username>   <!-- Replace with actual date (YYYY-MM-DD) -->

## Required settings

- Require a pull request before merging: ON
- Required approving reviews: 1
- Dismiss stale reviews on new push: ON
- Require review from Code Owners: ON
- Require status checks to pass before merging: ON
  - Required checks: guardrails-check (or equivalent CI job)
- Restrict who can push to matching branches: ON (only maintainers)
- Do not allow bypassing the above settings: ON
```

**Apply these settings in the GitHub UI:**

1. Go to Settings → Branches → Add rule.
2. Branch name pattern: `main`.
3. Enable every item listed above.
4. Click Save.

---

## Step 3 — MCP Server Registry and Agent Skills

### 3a. Create `.github/mcp-config.json`

This file is the allowlist of Model Context Protocol servers the agent may contact.

Copy [`templates/mcp-config-example.json`](mcp-config-example.json):

```bash
cp templates/mcp-config-example.json .github/mcp-config.json
```

**Required fields for each server entry:**

| Field | Description |
| --- | --- |
| `id` | Short unique identifier (e.g., `github-rest`) |
| `description` | What this server is used for |
| `allowed_scopes` | List of permission scopes this server may use |
| `deny` | `true` — add any server here to explicitly block it |

> **Default policy: deny all.** Only servers explicitly listed with `deny: false` are permitted.
> Add new servers using the blank entry in [`templates/mcp-server-entry.json`](mcp-server-entry.json).

### 3b. Create agent skills in `.github/skills/`

Skills are reusable, scoped action definitions. Create one YAML file per skill.

```bash
mkdir -p .github/skills
cp templates/skill-example.yml .github/skills/my-skill.yml
```

**Recommended starter skills:**

| Skill file | What it does | Autonomy level |
| --- | --- | --- |
| `review-pr.yml` | Reads PR diff; posts structured review comment | 2 |
| `triage-issue.yml` | Classifies new issues and applies a label | 2 |
| `generate-docs.yml` | Drafts documentation on a new branch | 2 |

For each skill, set the `autonomy_level` field to match your repository's active level from `.github/copilot-instructions.md`.

---

## Step 4 — Guardrail Workflows and Incident Runbook

### 4a. Create guardrail workflows in `.github/workflows/`

> **Note:** Workflow files are protected by CODEOWNERS. Only humans may create or modify them.

Copy the example workflow templates and rename them:

```bash
mkdir -p .github/workflows
cp templates/workflow-agent-safe-example.yml .github/workflows/guardrails-check.yml
cp templates/workflow-multi-agent-example.yml .github/workflows/agent-multi-agent-planner.yml
```

**Minimum set of workflows to activate:**

| Workflow file | Trigger | Purpose |
| --- | --- | --- |
| `guardrails-check.yml` | Every push to any branch | Secret scan, scope control, attribution check |
| `agent-pr-review.yml` | `pull_request` | Posts review comment; hard-stops on sensitive paths |
| `agent-issue-triage.yml` | `issues: opened` | Applies label; detects secret patterns |

Each workflow must include a least-privilege `permissions:` block. See [`templates/workflow-agent-safe-example.yml`](workflow-agent-safe-example.yml) for the exact pattern.

### 4b. Create `.github/guardrails.md`

This file documents the autonomy matrix, hard stops, audit trail requirements, and rollback procedures.

```bash
cp templates/guardrails-example.md .github/guardrails.md
```

Complete the `<!-- REPLACE -->` annotations for your project:

- Set the **active autonomy level** for your repository.
- Add any project-specific **hard stops** (row 8 in the Never-Do table).
- Fill in the **last reviewed** date and reviewer name.

### 4c. Create `.github/INCIDENT_RESPONSE.md`

This is the step-by-step runbook for responding to a bad agent action.

```bash
cp templates/incident-report-example.md .github/INCIDENT_RESPONSE.md
```

Customise the escalation contacts and repo-specific rollback commands.

---

## Step 5 — GitHub UI: Environment Protection Gate

This step requires the GitHub web UI. It cannot be completed via git commands alone.

1. Go to **Settings → Environments** in your repository.
2. Click **New environment** and name it `agent-execution`.
3. Under **Deployment protection rules**, enable **Required reviewers**.
4. Add yourself (or your team) as a required reviewer.
5. Under **Environment secrets**, add any secrets the agent workflows need (e.g., `AGENT_TOKEN`).
6. Click **Save protection rules**.

> **Why this matters:** The `agent-execution` environment is referenced in your workflows as a deployment gate. Any workflow job that targets this environment will pause and wait for human approval before proceeding.

---

## Step 6 — Issue and PR Templates

Create standard templates so that issues and PRs carry the structured information agents expect.

```bash
mkdir -p .github/ISSUE_TEMPLATE
```

**Agent incident issue template** (`.github/ISSUE_TEMPLATE/agent-incident.md`):

```markdown
---
name: Agent Incident Report
about: Report a harmful or unexpected agent action
labels: incident, agent
---

## What happened

## Files or data affected

## Immediate actions taken

## Root cause (if known)

## Prevention
```

**PR description template** (`.github/PULL_REQUEST_TEMPLATE.md`):

Copy [`templates/pr-checklist.md`](pr-checklist.md) to `.github/PULL_REQUEST_TEMPLATE.md`:

```bash
cp templates/pr-checklist.md .github/PULL_REQUEST_TEMPLATE.md
```

---

## Step 7 — Copilot Setup Steps (optional but recommended)

If your repository has build dependencies that the Copilot coding agent needs pre-installed, create a setup steps workflow:

```bash
mkdir -p .github/workflows
```

Create `.github/workflows/copilot-setup-steps.yml`:

```yaml
name: Copilot Setup Steps
on:
  workflow_dispatch:

jobs:
  setup:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up Node.js          # Replace with your language/toolchain
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - name: Install dependencies
        run: npm ci
      - name: Verify toolchain
        run: npx eslint --version     # Replace with your verification command
```

Adjust the language setup action and verification command for your stack (Python, Go, Java, etc.).

---

## Validation Checklist

After completing all steps, verify the setup:

**Steps 1–5 (required)**

- [ ] `.github/copilot-instructions.md` exists and has a non-empty `## What the Agent MAY NOT Do` section.
- [ ] `.github/copilot-memory.md` exists and includes the file map and conventions sections.
- [ ] `CODEOWNERS` (or `.github/CODEOWNERS`) lists all governance files with a human reviewer.
- [ ] Branch protection on `main` is enabled with required reviews and required status checks.
- [ ] `.github/mcp-config.json` exists and uses **deny-by-default** policy.
- [ ] At least one skill YAML exists in `.github/skills/`.
- [ ] `guardrails-check.yml` workflow is active and passing on a test push.
- [ ] `.github/guardrails.md` has an active autonomy level set.
- [ ] `agent-execution` environment exists with at least one required reviewer.

**Step 6 (recommended)**

- [ ] `.github/ISSUE_TEMPLATE/agent-incident.md` exists.
- [ ] `.github/PULL_REQUEST_TEMPLATE.md` contains the agent accountability checklist.

**Step 7 (optional)**

- [ ] `.github/workflows/copilot-setup-steps.yml` exists and installs the required toolchain for your project.

**End-to-end smoke test**

- [ ] Open a draft PR from a `copilot/test` branch and confirm the guardrails workflow runs and passes.

---

## Quick Reference — File Map

| File | Required | Purpose |
| --- | --- | --- |
| `.github/copilot-instructions.md` | ✅ | Agent standing orders (MAY / MAY NOT / stop conditions) |
| `.github/copilot-memory.md` | ✅ | Persistent context (file map, conventions, checkpoints) |
| `CODEOWNERS` | ✅ | Human review gates on governance files |
| `.github/branch-protection.md` | ✅ | Auditable record of branch protection settings |
| `.github/mcp-config.json` | ✅ | Approved MCP server allowlist |
| `.github/guardrails.md` | ✅ | Autonomy matrix, hard stops, rollback procedures |
| `.github/INCIDENT_RESPONSE.md` | ✅ | Runbook for bad agent actions |
| `.github/skills/*.yml` | Recommended | Reusable agent skill definitions |
| `.github/workflows/guardrails-check.yml` | Recommended | Automated secret scan + scope control |
| `.github/workflows/agent-pr-review.yml` | Recommended | Automated PR review comment |
| `.github/workflows/agent-issue-triage.yml` | Recommended | Automated issue classification |
| `.github/workflows/copilot-setup-steps.yml` | Optional | Pre-install build tools for the coding agent |
| `.github/PULL_REQUEST_TEMPLATE.md` | Recommended | Structured PR descriptions with agent checklist |
| `.github/ISSUE_TEMPLATE/agent-incident.md` | Recommended | Structured incident reports |

---

## Troubleshooting

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| Agent opens PR but no review is requested | `CODEOWNERS` not configured or not in protected path | Add the file and enable "Require review from Code Owners" in branch protection |
| Guardrails workflow skipped | Workflow file not committed, or branch protection not enforcing required checks | Commit the workflow and add it to required status checks |
| Agent calls an unapproved MCP server | Stop condition not in `copilot-instructions.md` or not enforced | Add the server to the deny list in `mcp-config.json` and add a stop condition |
| Workflow fails with `permission denied` | Missing or overly restrictive `permissions:` block | Add an explicit `permissions:` block (see `workflow-agent-safe-example.yml`) |
| Environment gate never triggers | Workflow job does not reference `environment: agent-execution` | Add `environment: agent-execution` to the relevant job in your workflow |
| Agent modifies `CODEOWNERS` or workflow files | CODEOWNERS not enforced by branch protection | Enable "Require review from Code Owners" in branch protection settings |

---

## Related Templates

| Template | What it provides |
| --- | --- |
| [`copilot-instructions-example.md`](copilot-instructions-example.md) | Fully annotated standing orders template |
| [`copilot-memory-example.md`](copilot-memory-example.md) | Annotated memory file with field explanations |
| [`guardrails-example.md`](guardrails-example.md) | Autonomy matrix and rollback procedures |
| [`mcp-config-example.json`](mcp-config-example.json) | MCP config with 3 annotated example servers |
| [`skill-example.yml`](skill-example.yml) | Annotated skill YAML with every field explained |
| [`workflow-agent-safe-example.yml`](workflow-agent-safe-example.yml) | Least-privilege agent workflow |
| [`workflow-multi-agent-example.yml`](workflow-multi-agent-example.yml) | Planner → approval → executor → reviewer pipeline |
| [`pr-checklist.md`](pr-checklist.md) | Agent PR description with accountability checklist |
| [`incident-report-example.md`](incident-report-example.md) | Filled-out incident report example |
| [`artifact-schema.json`](artifact-schema.json) | JSON Schema for inter-agent handoff artifacts |
| [`mcp-server-entry.json`](mcp-server-entry.json) | Blank MCP server entry for registering a new tool |
