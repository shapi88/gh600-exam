# Topic 3: Manage Memory, State, and Execution

## Key Concepts and Sub-Topics

- **Memory types:**
  - **In-prompt context** — information in the current conversation; lost when the session ends
  - **Repository memory** — durable, repo-scoped facts stored by Copilot memory features
  - **User preference memory** — personal, durable preferences stored across runs
  - **Artifacts/logs** — machine outputs uploaded to GitHub; auditable and retained per policy
  - **External state stores** — databases or services outside GitHub; use only when GitHub-native stores are insufficient

- **Durable vs ephemeral state** — ephemeral state lives only for one run; durable state survives runs and is accessible to future agents

- **Checkpointing and resumability** — recording progress so a failed or interrupted run can resume from a known good state without repeating destructive steps

- **Idempotency and replay safety** — designing operations so running them twice produces the same result as running them once

- **Context window management** — understanding token limits and choosing what to keep in the active prompt vs. retrieve on demand

---

## Common Pitfalls / Anti-Patterns

- Storing secrets or sensitive data (tokens, passwords, PII) in memory.
- Using stale issue/PR context after the branch has changed.
- Making an agent depend on non-durable chat state for critical decisions.
- Re-running destructive operations (creating issues, sending notifications) without idempotency checks.

---

## Step-by-Step Tutorial

**Goal:** Choose the right memory and state strategy for a multi-step agent run.

1. **Take a task that lasts longer than one interaction** — for example, a repo-wide refactor across multiple files.

2. **Decide what belongs where:**

   | Information | Best store |
   | --- | --- |
   | Current task instructions | Prompt context |
   | Step-by-step plan and progress | PR description or PR comment |
   | Repo coding conventions | Repository memory |
   | Run artifacts (diffs, reports) | GitHub Actions artifact |
   | Sensitive tokens or credentials | Never store; use secrets at runtime |

3. **Add a checkpoint after each major step** so the run can resume safely if it fails. For example, after completing step 3 of 8, post a PR comment: "✅ Steps 1–3 complete. Resuming from step 4."

4. **Identify idempotent operations** before allowing retries. For example: checking whether an issue with a given title already exists before creating a new one.

5. **Write a short "resume from failure" procedure:** identify the last checkpoint in the PR timeline, skip already-completed steps, and re-run only remaining steps.

---

## Memory Types Cheat Sheet

| Memory type | Best use | Lifetime | Risks | Good exam answer |
| --- | --- | --- | --- | --- |
| Prompt context | Current task instructions | Single run | Lost on reset; token pressure | Use for transient task data |
| Repository memory | Durable repo conventions | Multi-run | Stale or overbroad facts | Store reusable, non-sensitive repository rules |
| User preference memory | Personal workflow preferences | Multi-run | Privacy or scope mistakes | Store only durable, non-sensitive user preferences |
| PR/issue comments | Decisions and approvals | Durable | Can become noisy | Best for auditable checkpoints |
| Artifacts/logs | Machine outputs, evidence | Run-scoped or retained | Retention cost, leakage | Good for reproducibility and debugging |
| External DB/store | Large structured state | Durable | Security and sync complexity | Use only when GitHub-native stores are insufficient |

---

## Practical Exercises

- Classify five example facts as ephemeral context, durable repository memory, or "do not store":
  1. The current PR number → ephemeral context
  2. The team's preferred test framework → repository memory
  3. A personal access token → do not store
  4. A one-off debug flag for today → ephemeral context
  5. A reviewer's general style preferences → user preference memory
- Design a retry-safe pattern for a task that opens issues or comments on PRs (hint: check for existence before creating).
- Explain how you would recover from stale context after a branch has been updated mid-run.

**What a strong answer includes:**
- Separation of durable vs transient state
- No sensitive data in memory
- Explicit checkpoints
- Retry and resume safety

---

## Exam-Style Questions

**Q1.** An agent must remember repo coding conventions across runs.  
**A:** Store non-sensitive repository conventions in durable repository memory or `.github/copilot-instructions.md`. This is reusable, repo-scoped context.

**Q2.** A user says, "Use this one-off debug flag for today only."  
**A:** Keep it ephemeral; do not store durable memory. Task-specific or temporary instructions should not persist.

**Q3.** A long-running agent crashes after step 4 of 8.  
**A:** Resume from an explicit checkpoint recorded in an artifact, issue, or PR comment. Durable state prevents duplicate or destructive replay.

**Q4.** A retry might re-open duplicate issues.  
**A:** Make the operation idempotent by checking for an existing marker or issue first. Safe retries are a core production pattern.

**Q5.** A proposed memory item contains a PAT or private customer data.  
**A:** Do not store it. Sensitive data must never enter durable memory.

---

## Relevant Resources

- [GitHub Docs — Managing Copilot Memory](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/copilot-memory)
- [GitHub Docs — Use GitHub Copilot agents](https://docs.github.com/en/copilot/how-tos/use-copilot-agents)
- [GitHub Docs — Storing workflow data as artifacts](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/storing-workflow-data-as-artifacts)
