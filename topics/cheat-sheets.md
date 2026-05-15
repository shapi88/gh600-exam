# Cheat Sheets and Summary Tables

Quick-reference tables covering every major decision area for the GH-600 exam. Use these during daily review and before mock exam sessions.

---

## Memory Types

| Memory type | Best use | Lifetime | Risks | Good exam answer |
| --- | --- | --- | --- | --- |
| Prompt context | Current task instructions | Single run | Lost on reset; token pressure | Use for transient task data |
| Repository memory | Durable repo conventions | Multi-run | Stale or overbroad facts | Store reusable, non-sensitive repository rules |
| User preference memory | Personal workflow preferences | Multi-run | Privacy or scope mistakes | Store only durable, non-sensitive user preferences |
| PR/issue comments | Decisions and approvals | Durable | Can become noisy | Best for auditable checkpoints |
| Artifacts/logs | Machine outputs, evidence | Run-scoped or retained | Retention cost, leakage | Good for reproducibility and debugging |
| External DB/store | Large structured state | Durable | Security and sync complexity | Use only when GitHub-native stores are insufficient |

---

## Orchestration Patterns

| Pattern | Best for | Strength | Weakness | Use when |
| --- | --- | --- | --- | --- |
| Single constrained agent | Small, bounded tasks | Lowest complexity | Limited specialization | One task, low ambiguity |
| Planner → executor | Moderate implementation | Clear separation of intent/action | More latency | You need explicit review of the plan |
| Planner → executor → reviewer | PR-quality work | Strong safety and quality | More coordination overhead | You need both execution and independent validation |
| Specialist swarm + coordinator | Broad analysis | Parallel coverage | Synthesis complexity | Researching many independent areas |
| Human-supervised loop | Sensitive or high-impact changes | Highest accountability | Slowest | Infra, security, compliance, production |

---

## Autonomy Levels

| Level | Agent capability | Recommended controls |
| --- | --- | --- |
| 0 | Suggest only | Human executes everything; agent has no write access |
| 1 | Draft plans/content | Human approval before any state change |
| 2 | Limited repo changes | Branch restriction, required checks, human review |
| 3 | Broad repo automation | Tight permissions, audit trail, approvals, rollback plan |
| 4 | Deployment-affecting actions | Environments, required reviewers, OIDC, incident controls |

---

## Key Permissions and Settings

| Control | Why it matters | Safer default |
| --- | --- | --- |
| `permissions:` in workflow | Limits token scope for every job | Set all scopes explicitly; deny by default |
| `persist-credentials: false` | Prevents accidental credential reuse after checkout | Enable unless push is explicitly required |
| Environments | Gate secrets and require deployment approval | Use for staging and production |
| Branch protection | Prevents bypassing review or status checks | Require reviews and required status checks |
| CODEOWNERS | Forces expert review for sensitive paths | Add for workflows, infra, and security files |
| OIDC | Avoids long-lived cloud secrets | Prefer over static credentials for cloud auth |

---

## Decision Framework

| Question | If yes | If no |
| --- | --- | --- |
| Does the task require external data or tooling? | Consider MCP or API tooling with narrow scope | Keep it in-repo only |
| Is the action write-capable? | Add guardrails, approvals, and logs | Read-only may be sufficient |
| Is the task sensitive or production-affecting? | Require human gate and environment controls | Lower autonomy level may suffice |
| Is context durable and reusable? | Store non-sensitive durable memory | Keep it ephemeral |
| Would retries be dangerous? | Add idempotency and checkpoints first | Standard bounded retry may be fine |

---

## Error Category Quick Reference

| Error type | Description | Mitigation |
| --- | --- | --- |
| Hallucinated tools | Agent calls a tool that does not exist | Restrict tool list; validate tool names before execution |
| Stale context | Agent acts on outdated file or branch state | Refresh context at each step; checkpoint timestamps |
| Excessive changes | Agent modifies files outside the intended scope | Add diff-size evaluation; tighten instructions |
| Policy violations | Agent bypasses a required check or approval | Enforce with platform controls, not prompt text alone |
| Unsafe output | Output exposes secrets or violates compliance | Add output scanning; treat safety failures as hard failures |

---

## GitHub Control Plane Quick Map

| Agent action | GitHub artifact or control |
| --- | --- |
| Capture intent | Issue with acceptance criteria |
| Propose changes | Draft PR |
| Record rationale | PR description or PR comment |
| Block unsafe merges | Branch protection + required status checks |
| Approve deploys | Environment protection rules |
| Route sensitive reviews | CODEOWNERS |
| Audit execution | Workflow logs and artifacts |
| Limit token scope | `permissions:` block |
| Avoid long-lived credentials | OIDC |
