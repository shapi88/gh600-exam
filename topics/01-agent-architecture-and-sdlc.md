# Topic 1: Prepare Agent Architecture and SDLC Processes

## Key Concepts and Sub-Topics

- **Agent roles:**
  - **Planner** — proposes steps, decomposes tasks, identifies risks
  - **Executor** — implements approved steps, changes state or code
  - **Reviewer** — validates output quality, scope, and policy compliance
  - **Policy/guardrail agent** — enforces constraints and escalation rules

- **SDLC placement:**
  - Backlog grooming (labeling, acceptance-criteria drafting)
  - Implementation (code generation on scoped branches)
  - CI triage (failure summarization, flaky-test detection)
  - Code review (diff analysis, style checks)
  - Release preparation (changelog, release notes)

- **Planning vs execution boundaries** — a planning agent should not also have unrestricted execution rights

- **Human-in-the-loop checkpoints** — required approval steps before merging, deploying, or making sensitive changes

- **GitHub as the control plane:**
  - Issues → intent capture and acceptance criteria
  - Pull requests → proposals, diffs, review, and audit trail
  - GitHub Actions → observable, auditable execution path
  - Environments → deployment approval gates
  - Branch protection → enforce review and status checks
  - CODEOWNERS → route sensitive paths to human reviewers

---

## Common Pitfalls / Anti-Patterns

- Giving the same agent both broad planning authority and unrestricted execution.
- Letting an agent merge or deploy without required checks and human approval.
- Hiding decisions in chat instead of durable GitHub artifacts like PR comments, reviews, or workflow logs.
- Using agent autonomy where deterministic automation (a simple workflow or script) is enough.

---

## Step-by-Step Tutorial

**Goal:** Design a safe agent-assisted delivery flow for a small repository.

1. **Create a sample issue** with acceptance criteria, out-of-scope items, and a risk note. This becomes the agent's starting artifact.

2. **Define role boundaries.** Decide which parts of the flow are:
   - Planning-only (agent proposes, nothing changes in the repo)
   - Execution-capable (agent opens branches and commits code)
   - Human-approved (no merge or deploy without a human gate)

3. **Map the GitHub control plane objects involved:**
   ```
   issue → branch → draft PR → CI checks → human review → merge
   ```
   Label each step with the responsible party: agent, automation, or human.

4. **Add branch protection rules** for `main`:
   - Require at least one human review.
   - Require all status checks to pass.
   - Do not allow bypassing the above rules for agents.

5. **Write a short escalation policy** describing when the agent must stop and wait for a human:
   - Touching files listed in `CODEOWNERS`
   - Exceeding a diff-size threshold
   - Encountering a secret or credentials-related file
   - Failing more than N retries

---

## Practical Exercises

- Draw a planner → executor → reviewer flow and label every handoff artifact (issue, PR comment, review, log).
- Given two tasks — a docs-only change and an infra change — assign a safe autonomy level (0–4) to each and justify it.
- Rewrite the risky requirement "agent merges fixes automatically" into a governed workflow with explicit controls.
- Identify three SDLC stages where an agent adds value without execution rights and explain why that boundary is appropriate.

**What a strong answer includes:**
- Explicit role boundaries
- Required human checkpoints
- Durable audit trail in GitHub artifacts
- Least privilege aligned to the SDLC stage

---

## Exam-Style Questions

**Q1.** A team wants an agent to take issues directly to merge in one run.  
**A:** Split responsibilities across planning, execution, CI, and human review. Require protected branches and status checks before merge. The risk is excessive autonomy with no accountability boundary.

**Q2.** An agent can generate code and approve its own PR.  
**A:** Remove self-approval and require independent human or policy review. Approval must be independent for trustworthy governance.

**Q3.** Product managers want faster backlog grooming.  
**A:** Use an agent for summarization, labeling, and acceptance-criteria drafting. Keep prioritization and commitment with humans. This matches low-risk, high-value augmentation.

**Q4.** A repo has frequent small documentation updates.  
**A:** Allow higher autonomy for scoped documentation changes with standard checks. Autonomy should reflect risk and reversibility.

**Q5.** A team asks where to store the agent's rationale for a code change.  
**A:** In the PR description or PR comments tied to the run. GitHub artifacts provide auditability; transient chat does not.

---

## Relevant Resources

- [GitHub Docs — Copilot overview](https://docs.github.com/en/copilot)
- [GitHub Docs — Use GitHub Copilot agents](https://docs.github.com/en/copilot/how-tos/use-copilot-agents)
- [GitHub Docs — About CODEOWNERS](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners)
- [GitHub Docs — Workflow syntax (permissions, environments, jobs)](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
