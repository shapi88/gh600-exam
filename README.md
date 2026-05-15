# GH-600 Exam Study Package

Production-oriented, safety-first prep for **GitHub Certified: Agentic AI Developer (GH-600)** with GitHub as the control plane for agentic SDLC workflows.

> [!TIP]
> If you are time-constrained, start with the **Table of Contents**, complete the **5-week study plan**, then use the **cheat sheets** and **practice questions** for daily review.

## Table of Contents

1. [High-Level Study Plan (5 Weeks)](#high-level-study-plan-5-weeks)
2. [Detailed Topic Breakdown](#detailed-topic-breakdown)
   1. [Prepare agent architecture and SDLC processes](#1-prepare-agent-architecture-and-sdlc-processes)
   2. [Implement tool use and environment interaction](#2-implement-tool-use-and-environment-interaction)
   3. [Manage memory, state, and execution](#3-manage-memory-state-and-execution)
   4. [Perform evaluation, error analysis, and tuning](#4-perform-evaluation-error-analysis-and-tuning)
   5. [Orchestrate multi-agent coordination](#5-orchestrate-multi-agent-coordination)
   6. [Implement guardrails and accountability](#6-implement-guardrails-and-accountability)
3. [Practical Examples and Hands-On Labs](#practical-examples-and-hands-on-labs)
4. [Cheat Sheets and Summary Tables](#cheat-sheets-and-summary-tables)
5. [Practice Questions](#practice-questions)
6. [Recommended Resources and References](#recommended-resources-and-references)
7. [Final Exam Strategy](#final-exam-strategy)

## High-Level Study Plan (5 Weeks)

| Week | Focus areas | Time target | Deliverables |
| --- | --- | --- | --- |
| 1 | Agent architecture, SDLC integration, planning vs execution boundaries, human-in-the-loop | 6-8 hrs | One-page architecture notes, autonomy matrix, branch protection + review checklist |
| 2 | Tool use, MCP, permissions, runner environments, workflow design | 8-10 hrs | One repo with an agent-safe workflow, MCP config, least-privilege permissions examples |
| 3 | Memory, state, execution control, retries, resumability, observability | 6-8 hrs | Memory strategy table, state diagram, incident playbook for stuck/failed runs |
| 4 | Evaluation, error analysis, prompt tuning, multi-agent coordination | 8-10 hrs | Eval rubric, failure taxonomy, planner/executor/reviewer workflow examples |
| 5 | Guardrails, accountability, governance, end-to-end review, mock exam drills | 8-10 hrs | Full mock runbook, 30-question self-test, final cheat sheet |

### Weekly Breakdown

#### Week 1 — Architecture and SDLC fit

- Map where agents can act safely: issue triage, code generation, review, release assistance, documentation, test analysis.
- Learn the difference between:
  - **planning agents** that propose steps
  - **execution agents** that change state or code
  - **supervisory controls** that approve, reject, or constrain actions
- Deliverable:
  - A diagram of agent entry points in a GitHub flow: issue -> plan -> branch -> PR -> checks -> review -> merge.

#### Week 2 — Tools, MCP, and environment interaction

- Study GitHub-hosted runners, repository permissions, secrets, environment approvals, and MCP integration.
- Practice configuring tools with:
  - explicit permissions
  - approved environments
  - minimal scopes
  - deterministic inputs
- Deliverable:
  - A repo containing one workflow with strict `permissions:` and one MCP-backed tool configuration.

#### Week 3 — Memory, state, and execution control

- Compare ephemeral context, repository memory, user preferences, artifacts, issues, and PR comments as memory/state stores.
- Practice safe retry, resume, checkpoint, and idempotent execution patterns.
- Deliverable:
  - A written policy for what should be stored in prompt context vs durable repository memory vs workflow artifacts.

#### Week 4 — Evals, tuning, and multi-agent systems

- Build an evaluation loop using scenario-based prompts and expected outcomes.
- Practice failure analysis:
  - wrong tool
  - wrong scope
  - stale context
  - excessive autonomy
  - missing review gate
- Deliverable:
  - A planner/executor/reviewer pattern with pass/fail criteria and escalation triggers.

#### Week 5 — Guardrails, governance, and mock exam rehearsal

- Review safety controls, auditability, logging, accountability, and approval gates.
- Simulate exam-style cases involving risky permissions, memory misuse, or tool misconfiguration.
- Deliverable:
  - Final condensed cheat sheet and two timed mock sessions.

## Detailed Topic Breakdown

## 1. Prepare agent architecture and SDLC processes

### Key concepts and sub-topics

- Agent roles:
  - planner
  - executor
  - reviewer
  - policy/guardrail agent
- SDLC placement:
  - backlog grooming
  - implementation
  - CI triage
  - code review
  - release preparation
- Planning vs execution boundaries
- Human-in-the-loop checkpoints
- GitHub as the control plane:
  - issues
  - pull requests
  - Actions
  - environments
  - branch protection
  - CODEOWNERS

### Common pitfalls / anti-patterns

- Giving the same agent both broad planning authority and unrestricted execution.
- Letting an agent merge or deploy without required checks and approval.
- Hiding decisions in chat instead of durable GitHub artifacts like PR comments, reviews, or workflow logs.
- Using agent autonomy where deterministic automation is enough.

### Exam-style scenario examples

- A team wants an agent to open PRs automatically from issues. Best answer: restrict the agent to branch-scoped changes, require CI + human review, and log rationale in the PR.
- An organization wants faster triage. Best answer: use an agent for labeling/summarization, not direct code changes, and preserve reviewer oversight.

### Best practices and GitHub-specific features

- Use **issues** for intent capture and acceptance criteria.
- Use **draft PRs** for agent proposals before review-ready changes.
- Use **branch protection** and **required checks** to enforce review.
- Use **CODEOWNERS** to route sensitive areas to humans.
- Prefer **GitHub Actions** to make the agent path observable and auditable.

## 2. Implement tool use and environment interaction

### Key concepts and sub-topics

- Tool selection vs model-only reasoning
- MCP servers and external system access
- Permissions design:
  - `GITHUB_TOKEN`
  - fine-grained job permissions
  - environment secrets
- Runner environment design:
  - preinstalled tools
  - deterministic setup
  - network boundaries
- Tool failure handling and fallbacks

### Common pitfalls / anti-patterns

- Granting default write permissions to all jobs.
- Allowing unrestricted network/tool access when only repository reads are needed.
- Connecting MCP tools without governance, auth review, or input validation.
- Letting prompts choose tools arbitrarily instead of using policy and explicit allowlists.

### Exam-style scenario examples

- A code-review agent needs read-only repo context. Best answer: `contents: read`, `pull-requests: read`, no secrets, no write scopes.
- A release-note agent needs issue and PR metadata but not code execution. Best answer: API access only; no self-hosted runner or deployment credentials.

### Best practices and GitHub-specific features

- Set explicit `permissions:` at workflow and job level.
- Use **environments** for deployment approvals and secret boundaries.
- Use **OIDC** over long-lived cloud credentials where possible.
- Keep MCP server permissions narrowly scoped and documented.
- Use **Copilot setup steps** or runner bootstrap only for approved tools.

## 3. Manage memory, state, and execution

### Key concepts and sub-topics

- Memory types:
  - in-prompt context
  - repository memory
  - user preferences
  - artifacts/logs
  - external state stores
- Durable vs ephemeral state
- Checkpointing and resumability
- Idempotency and replay safety
- Context window management and retrieval boundaries

### Common pitfalls / anti-patterns

- Storing secrets or sensitive data in memory.
- Using stale issue/PR context after the branch has changed.
- Making an agent depend on non-durable chat state for critical decisions.
- Re-running destructive tools without idempotency checks.

### Exam-style scenario examples

- A long-running refactor agent loses context. Best answer: checkpoint the plan and progress in durable repo artifacts or PR comments, then resume from explicit state.
- A team wants to "remember" reviewer preferences. Best answer: store durable, non-sensitive style preferences; never store secrets or transient one-off instructions.

### Best practices and GitHub-specific features

- Store durable decision points in PR descriptions, issues, comments, or repository memory.
- Use artifacts for intermediate machine outputs that must be auditable.
- Use issue/PR timelines as execution history.
- Design runs to be safe on retries.

## 4. Perform evaluation, error analysis, and tuning

### Key concepts and sub-topics

- Golden scenarios and pass/fail rubrics
- Task decomposition and measurable outcomes
- Error categories:
  - hallucinated tools
  - stale context
  - excessive changes
  - policy violations
  - unsafe output
- Prompt iteration and tool-choice tuning
- Regression testing for agent behavior

### Common pitfalls / anti-patterns

- Measuring success only by "the answer sounds good."
- Evaluating prompts without representative repo context.
- Tuning only for success rate while ignoring safety and change scope.
- Fixing failures with broader permissions instead of better instructions or narrower tools.

### Exam-style scenario examples

- An agent solves tasks but keeps editing unrelated files. Best answer: tighten instructions, add diff-size evaluation, and require justification for touched files.
- A workflow agent fails intermittently on tool calls. Best answer: inspect logs, classify error type, add retries for transient failures only, and preserve failure telemetry.

### Best practices and GitHub-specific features

- Use repeatable repo fixtures or sample tasks.
- Record outcomes in issues, discussions, or evaluation artifacts.
- Compare prompt versions with the same scenarios.
- Treat safety regressions as test failures, not acceptable tradeoffs.

## 5. Orchestrate multi-agent coordination

### Key concepts and sub-topics

- Common patterns:
  - planner -> executor -> reviewer
  - specialist swarm with coordinator
  - parallel research + centralized synthesis
  - human escalation gate
- Shared context contracts
- Handoff boundaries and ownership
- Conflict resolution and deduplication
- Cost, latency, and failure-domain tradeoffs

### Common pitfalls / anti-patterns

- Multiple agents editing the same files without coordination.
- Undefined ownership between planner and executor.
- No synthesis layer, causing contradictory outputs.
- Overusing multi-agent workflows when a single constrained agent is enough.

### Exam-style scenario examples

- A monorepo needs separate security, docs, and test analysis. Best answer: use specialist agents with a coordinator that merges findings and enforces a final review gate.
- A planner produces steps that the executor cannot safely perform. Best answer: validate the plan against tool permissions before execution.

### Best practices and GitHub-specific features

- Give each agent a clear contract: inputs, outputs, permissions, and stop conditions.
- Use PR comments, artifacts, or structured workflow outputs for handoff.
- Keep one final accountable owner for merge/deploy decisions.
- Parallelize read-heavy work; serialize write-heavy work.

## 6. Implement guardrails and accountability

### Key concepts and sub-topics

- Autonomy levels and approval gates
- Policy enforcement
- Audit trails
- Sensitive-path protections
- Incident response for bad agent actions
- Safe execution defaults

### Common pitfalls / anti-patterns

- Allowing silent writes with no attribution.
- Missing approval steps for production, secrets, or compliance-bound systems.
- Storing policy only in prompts with no enforceable repository controls.
- Treating logs as optional.

### Exam-style scenario examples

- An org wants agents to update infra code. Best answer: require strict reviewers, environment approvals, and policy checks before merge.
- A security agent needs to comment on findings but not push code. Best answer: grant PR comment permissions only; preserve logs and ownership.

### Best practices and GitHub-specific features

- Use **branch protection**, **required reviewers**, **required status checks**, and **environments**.
- Restrict secret access to the minimum jobs and environments.
- Use audit-friendly channels: PR comments, review summaries, workflow logs, artifacts.
- Build "stop conditions" into prompts and workflows.

## Practical Examples and Hands-On Labs

### Example 1: Least-privilege agent workflow

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

**Why it is exam-relevant**

- Explicit permissions
- Limited write scope
- Observable steps
- Guardrail on change size

### Example 2: MCP configuration with approved tools only

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

**What to remember**

- MCP expands capability, so governance must also expand.
- Token scope should match the exact tasks performed.
- Only approved servers should be enabled in enterprise contexts.

### Example 3: Repository custom instructions

```markdown
# .github/copilot-instructions.md

- Always propose a short plan before making code changes.
- Use the smallest possible diff that fully solves the task.
- Never bypass required tests, reviews, or environment approvals.
- Prefer read-only inspection tools before write actions.
- For sensitive files, stop and request human review before editing.
```

**Exam angle:** repository-level instructions improve consistency, but they are not a substitute for enforceable policy.

### Example 4: Guarded deployment with human approval

```yaml
name: deploy

on:
  workflow_dispatch:

permissions:
  contents: read
  deployments: write
  id-token: write

jobs:
  deploy-prod:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4
      - name: Deploy
        run: ./scripts/deploy.sh
```

**What matters**

- `environment: production` can enforce human approval.
- `id-token: write` supports short-lived cloud auth with OIDC.
- Deployment is still observable and reviewable in GitHub.

### Mini-project labs

#### Lab 1 — Single-agent safe coding workflow

1. Create an issue with acceptance criteria.
2. Add repository custom instructions.
3. Configure one workflow with least privilege.
4. Have the agent propose a plan in the PR description.
5. Require checks and human review before merge.

**Success criteria:** the agent can assist productively without direct merge or deploy rights.

#### Lab 2 — Planner/executor/reviewer pipeline

1. Planner agent reads the issue and outputs a 5-step plan.
2. Executor agent applies only approved steps.
3. Reviewer agent checks scope, tests, and policy compliance.
4. Human approves or rejects.

**Success criteria:** each handoff is explicit, attributable, and recoverable.

#### Lab 3 — Eval harness for agent quality

1. Create 10 representative tasks in issues.
2. Define a rubric: correctness, scope control, safety, auditability.
3. Run the same tasks across prompt versions.
4. Compare failure modes and update instructions.

**Success criteria:** prompt changes are justified by measured improvement, not guesswork.

### Sample prompts

#### Planner prompt

```text
Read the linked issue and repository instructions. Produce a 5-step implementation plan.
Do not modify files. Identify required tests, risk areas, and human approval points.
```

#### Executor prompt

```text
Implement only the approved plan steps. Use the smallest possible diff.
Do not change unrelated files. Stop if a required secret, permission, or policy decision is missing.
```

#### Reviewer prompt

```text
Review the proposed diff for correctness, unnecessary scope, missing tests, policy violations, and unsafe actions.
Return findings grouped by severity with concrete remediation guidance.
```

## Cheat Sheets and Summary Tables

### Memory types

| Memory type | Best use | Lifetime | Risks | Good exam answer |
| --- | --- | --- | --- | --- |
| Prompt context | Current task instructions | Single run | Lost on reset; token pressure | Use for transient task data |
| Repository memory | Durable repo conventions | Multi-run | Stale or overbroad facts | Store reusable, non-sensitive repository rules |
| User preference memory | Personal workflow preferences | Multi-run | Privacy or scope mistakes | Store only durable, non-sensitive user preferences |
| PR/issue comments | Decisions and approvals | Durable | Can become noisy | Best for auditable checkpoints |
| Artifacts/logs | Machine outputs, evidence | Run-scoped or retained | Retention cost, leakage | Good for reproducibility and debugging |
| External DB/store | Large structured state | Durable | Security and sync complexity | Use only when GitHub-native stores are insufficient |

### Orchestration patterns

| Pattern | Best for | Strength | Weakness | Use when |
| --- | --- | --- | --- | --- |
| Single constrained agent | Small tasks | Lowest complexity | Limited specialization | One bounded task, low ambiguity |
| Planner -> executor | Moderate implementation tasks | Clear separation of intent/action | More latency | You need explicit review of the plan |
| Planner -> executor -> reviewer | PR-quality work | Strong safety and quality | More coordination | You need both execution and independent validation |
| Specialist swarm + coordinator | Broad analysis | Parallel coverage | Synthesis complexity | Researching many independent areas |
| Human-supervised loop | Sensitive changes | Highest accountability | Slowest | Infra, security, compliance, prod |

### Autonomy levels

| Level | Agent capability | Recommended controls |
| --- | --- | --- |
| 0 | Suggest only | Human executes everything |
| 1 | Draft plans/content | Human approval before any state change |
| 2 | Limited repo changes | Branch restriction, required checks, human review |
| 3 | Broad repo automation | Tight permissions, audit trail, approvals, rollback |
| 4 | Deployment-affecting actions | Environments, approvals, OIDC, incident controls |

### Key permissions and settings

| Control | Why it matters | Safer default |
| --- | --- | --- |
| `permissions:` in workflow | Limits token scope | Set all explicitly |
| `persist-credentials: false` | Prevents accidental credential reuse | Enable unless push is required |
| Environments | Gate secrets and deployment approval | Use for staging/prod |
| Branch protection | Prevents bypassing review/checks | Require reviews and status checks |
| CODEOWNERS | Forces expert review | Add for sensitive paths |
| OIDC | Avoids long-lived cloud secrets | Prefer over static credentials |

### Decision framework

| Question | If yes | If no |
| --- | --- | --- |
| Does the task require external data/tooling? | Consider MCP or API tooling with narrow scope | Keep it in-repo only |
| Is the action write-capable? | Add guardrails, approvals, and logs | Read-only may be enough |
| Is the task sensitive or production-affecting? | Require human gate and environment controls | Lower autonomy may suffice |
| Is context durable and reusable? | Store non-sensitive durable memory | Keep it ephemeral |
| Would retries be dangerous? | Add idempotency/checkpoints first | Standard retry may be fine |

## Practice Questions

## 1. Prepare agent architecture and SDLC processes

1. **Scenario:** A team wants an agent to take issues directly to merge in one run.  
   **Correct answer:** Split responsibilities across planning, execution, CI, and human review. Require protected branches and status checks before merge.  
   **Why:** The risk is excessive autonomy with no accountability boundary.

2. **Scenario:** An agent can generate code and approve its own PR.  
   **Correct answer:** Remove self-approval and require independent human or policy review.  
   **Why:** Approval must be independent for trustworthy governance.

3. **Scenario:** Product managers want faster backlog grooming.  
   **Correct answer:** Use an agent for summarization, labeling, and acceptance-criteria drafting. Keep prioritization and commitment with humans.  
   **Why:** This matches low-risk, high-value augmentation.

4. **Scenario:** A repo has frequent small doc updates.  
   **Correct answer:** Allow higher autonomy for scoped documentation changes with standard checks.  
   **Why:** Autonomy should reflect risk and reversibility.

5. **Scenario:** A team asks where to store the agent’s rationale for a code change.  
   **Correct answer:** In a PR description or PR comments tied to the run.  
   **Why:** GitHub artifacts provide auditability better than transient chat.

## 2. Implement tool use and environment interaction

1. **Scenario:** A code review agent only needs to inspect PR metadata and diffs.  
   **Correct answer:** Grant read-only repository and PR scopes.  
   **Why:** Least privilege reduces blast radius.

2. **Scenario:** An agent needs production cloud access.  
   **Correct answer:** Use environment-gated deployment plus OIDC or tightly scoped credentials.  
   **Why:** Static broad secrets are riskier and harder to govern.

3. **Scenario:** An MCP server can reach internal systems and the public internet.  
   **Correct answer:** Restrict approved servers, document allowed operations, and validate auth and data boundaries.  
   **Why:** MCP expands the attack and failure surface.

4. **Scenario:** A workflow uses default token permissions.  
   **Correct answer:** Replace them with explicit job/workflow-level `permissions:`.  
   **Why:** Exam questions often reward least-privilege configuration.

5. **Scenario:** A tool occasionally fails due to rate limits.  
   **Correct answer:** Add bounded retries for transient failures and preserve logs.  
   **Why:** Reliability controls should distinguish transient vs permanent failures.

## 3. Manage memory, state, and execution

1. **Scenario:** An agent must remember repo coding conventions across runs.  
   **Correct answer:** Store non-sensitive repository conventions in durable repository memory or instructions.  
   **Why:** This is reusable, repo-scoped context.

2. **Scenario:** A user says, "Use this one-off debug flag for today only."  
   **Correct answer:** Keep it ephemeral; do not store durable memory.  
   **Why:** Task-specific or temporary instructions should not persist.

3. **Scenario:** A long-running agent crashes after step 4 of 8.  
   **Correct answer:** Resume from an explicit checkpoint recorded in an artifact, issue, or PR comment.  
   **Why:** Durable state prevents duplicate or destructive replay.

4. **Scenario:** A retry might re-open duplicate issues.  
   **Correct answer:** Make the operation idempotent by checking for an existing marker or issue first.  
   **Why:** Safe retries are a core production pattern.

5. **Scenario:** A proposed memory item contains a PAT or private customer data.  
   **Correct answer:** Do not store it.  
   **Why:** Sensitive data must not enter durable memory.

## 4. Perform evaluation, error analysis, and tuning

1. **Scenario:** An agent frequently edits unrelated files.  
   **Correct answer:** Add an evaluation metric for touched-file scope and tighten instructions.  
   **Why:** You must measure and correct scope creep explicitly.

2. **Scenario:** A prompt performs well on toy examples but fails in the real repo.  
   **Correct answer:** Build evals with representative repository context and realistic tasks.  
   **Why:** Exam questions favor production realism over synthetic success.

3. **Scenario:** An agent picks the wrong tool even though the final answer looks plausible.  
   **Correct answer:** Evaluate tool-choice correctness separately from output quality.  
   **Why:** Tool misuse can create silent safety or reliability failures.

4. **Scenario:** The fix rate improves after adding broad write permissions.  
   **Correct answer:** Reject that as the primary tuning approach unless justified; improve instructions and task boundaries first.  
   **Why:** More permissions are not a quality strategy.

5. **Scenario:** You need to compare two prompt versions.  
   **Correct answer:** Run both against the same scenario set and rubric.  
   **Why:** Controlled comparison is required for meaningful tuning.

## 5. Orchestrate multi-agent coordination

1. **Scenario:** Security, docs, and test review all need to happen quickly on a large PR.  
   **Correct answer:** Use specialist agents in parallel and a coordinator to synthesize findings.  
   **Why:** Parallel read-heavy analysis is an ideal multi-agent use case.

2. **Scenario:** Two agents keep overwriting each other’s code changes.  
   **Correct answer:** Serialize write access or partition file ownership clearly.  
   **Why:** Parallel writes without coordination create conflicts and unreliability.

3. **Scenario:** The planner proposes steps the executor cannot perform with current permissions.  
   **Correct answer:** Validate the plan against available tools/permissions before execution.  
   **Why:** Feasibility checks belong at the handoff boundary.

4. **Scenario:** A single small task is given to three agents "for safety."  
   **Correct answer:** Prefer one constrained agent.  
   **Why:** Multi-agent overhead is not justified for low-complexity work.

5. **Scenario:** Findings from multiple agents conflict.  
   **Correct answer:** Route them through a synthesis or review stage with explicit prioritization rules.  
   **Why:** Coordination needs a decision owner, not just more outputs.

## 6. Implement guardrails and accountability

1. **Scenario:** An agent can edit workflow files in a protected repo.  
   **Correct answer:** Require CODEOWNERS review, protected branches, and possibly restricted file policies.  
   **Why:** Workflow changes can expand privileges and must be tightly governed.

2. **Scenario:** A manager wants fewer human approvals for production deploys.  
   **Correct answer:** Keep environment-based approval gates for high-impact actions.  
   **Why:** Risk, not convenience, should drive autonomy limits.

3. **Scenario:** A security review agent needs to report findings only.  
   **Correct answer:** Grant comment/report permissions, not repository write access.  
   **Why:** Match privileges to the exact action required.

4. **Scenario:** An agent made a harmful change and there is no record of why.  
   **Correct answer:** Improve auditability using PR rationale, logs, and workflow artifacts.  
   **Why:** Accountability depends on traceability.

5. **Scenario:** A policy is written only in prompt text.  
   **Correct answer:** Back it with enforceable GitHub controls like reviews, branch protection, environments, and permissions.  
   **Why:** Prompts guide behavior; platform controls enforce it.

## Recommended Resources and References

GitHub and Microsoft documentation URLs can change over time. If any link below has moved, use the relevant docs home page or site search to locate the current article by title.

### Official

- GitHub Docs — Copilot overview: <https://docs.github.com/en/copilot>
- GitHub Docs — Use GitHub Copilot agents: <https://docs.github.com/en/copilot/how-tos/use-copilot-agents>
- GitHub Docs — About MCP: <https://docs.github.com/en/copilot/concepts/about-mcp>
- GitHub Docs — Extend the coding agent with MCP: <https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/extend-coding-agent-with-mcp>
- GitHub Docs — Automatic token authentication and `GITHUB_TOKEN`: <https://docs.github.com/en/actions/security-for-github-actions/security-guides/automatic-token-authentication>
- GitHub Docs — Workflow syntax (`permissions`, environments, jobs): <https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions>
- GitHub Docs — Managing environments for deployment: <https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment>
- GitHub Docs — About OIDC in Actions: <https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect>
- GitHub Docs — About CODEOWNERS: <https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners>
- GitHub Docs — Managing Copilot Memory: <https://docs.github.com/en/copilot/how-tos/use-copilot-agents/copilot-memory>

### Useful practice resources

- GitHub Skills courses for Actions and workflow automation
- GitHub Actions example repositories
- MCP Registry: <https://github.com/mcp>
- Official Model Context Protocol docs: <https://modelcontextprotocol.io/introduction>

### How to use these resources efficiently

1. Read the docs once for concepts.
2. Convert each concept into a GitHub control: permission, workflow rule, review rule, or approval gate.
3. Build a tiny example repo for each high-risk topic.
4. Rehearse scenario questions using the cheat-sheet tables.

## Final Exam Strategy

- Think **GitHub control plane first**: issues, PRs, Actions, environments, reviews, logs, artifacts.
- Prefer **least privilege** over convenience.
- Prefer **human approval** for high-risk or production actions.
- Prefer **durable audit trails** over chat-only reasoning.
- Prefer **single constrained agents** unless specialization clearly improves outcomes.
- When a question presents speed vs safety, the exam usually rewards the answer that keeps safety, observability, and accountability intact.

> [!IMPORTANT]
> A strong GH-600 answer is usually the one that combines **clear role boundaries**, **minimal permissions**, **durable state/auditability**, and **human oversight for risky actions**.
