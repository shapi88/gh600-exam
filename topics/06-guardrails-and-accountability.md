# Topic 6: Implement Guardrails and Accountability

## Key Concepts and Sub-Topics

- **Autonomy levels and approval gates** — five levels from suggest-only to deployment-affecting, each requiring progressively stronger controls
- **Policy enforcement** — backing prompt-level guidelines with enforceable platform controls (branch protection, required reviews, environments)
- **Audit trails** — every agent action traceable through PR comments, review records, workflow logs, and artifacts
- **Sensitive-path protections** — CODEOWNERS, restricted file policies, and branch protection for workflow files and infrastructure code
- **Incident response for bad agent actions** — rollback procedures, post-mortems, and attribution requirements
- **Safe execution defaults** — workflows and agents default to the minimum capability required; write access must be explicitly added

---

## Common Pitfalls / Anti-Patterns

- Allowing silent writes with no attribution (no PR, no log, no comment).
- Missing approval steps for production, secrets, or compliance-bound systems.
- Storing policy only in prompts with no enforceable repository controls.
- Treating logs as optional rather than a required part of every agent-assisted action.

---

## Step-by-Step Tutorial

**Goal:** Enforce safety with platform controls, not just prompt instructions.

1. **List the highest-risk actions in the repo:** workflow file edits, secrets access, production deploys, CODEOWNERS changes, repository settings changes.

2. **Map each risk to a specific GitHub control:**

   | Risk | GitHub control |
   | --- | --- |
   | Workflow file edits | CODEOWNERS + required reviewers |
   | Secrets access | Environment protection rules |
   | Production deploys | `environment: production` with required reviewer |
   | Merge without review | Branch protection — required reviews |
   | Bypass status checks | Branch protection — required status checks |

3. **Add stop conditions to agent prompts** that force the agent to pause before sensitive actions:
   - "Stop and request human review before editing any file listed in CODEOWNERS."
   - "Stop if the diff touches files in `.github/workflows/`."
   - "Stop if a required secret or deployment credential is missing."

4. **Ensure all runs are attributable.** Every agent-assisted change should produce at least one of:
   - A PR with a description explaining the rationale
   - A PR comment or review from the agent's identity
   - A workflow log entry for the action

5. **Create a rollback and incident-response checklist:**
   - Identify the harmful commit or PR.
   - Revert using `git revert` or close and delete the branch.
   - Review the workflow logs to understand what triggered the action.
   - Update controls and instructions to prevent recurrence.
   - Document the incident in an issue or post-mortem.

---

## Autonomy Levels Reference

| Level | Agent capability | Recommended controls |
| --- | --- | --- |
| 0 | Suggest only | Human executes everything; agent has no write access |
| 1 | Draft plans/content | Human approval before any state change |
| 2 | Limited repo changes | Branch restriction, required checks, human review |
| 3 | Broad repo automation | Tight permissions, audit trail, approvals, rollback plan |
| 4 | Deployment-affecting actions | Environments, required reviewers, OIDC, incident controls |

---

## Practical Examples

### Repository custom instructions

```markdown
# .github/copilot-instructions.md

- Always propose a short plan before making code changes.
- Use the smallest possible diff that fully solves the task.
- Never bypass required tests, reviews, or environment approvals.
- Prefer read-only inspection tools before write actions.
- For sensitive files, stop and request human review before editing.
```

**Exam angle:** Repository-level instructions improve consistency, but they are **not** a substitute for enforceable platform controls.

### Guarded deployment with human approval

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

**What matters:**
- `environment: production` enforces human approval via environment protection rules.
- `id-token: write` supports short-lived cloud auth with OIDC.
- The deployment is observable and auditable in the Actions log.

---

## Practical Exercises

- For three high-risk actions (workflow file edit, production deploy, secrets access), name the exact GitHub control that gates each one.
- Evaluate whether a prompt-only policy is sufficient and explain why platform controls are necessary to back it up.
- Draft a short accountability checklist for agent-generated PRs (what must every PR include to be considered accountable?).
- Given a scenario where an agent made a harmful change with no audit trail, describe the steps to improve accountability going forward.

**What a strong answer includes:**
- Enforceable platform controls (not just prompt text)
- Approval gates for sensitive operations
- Attributable, auditable decisions
- Rollback and incident readiness

---

## Exam-Style Questions

**Q1.** An agent can edit workflow files in a protected repo.  
**A:** Require CODEOWNERS review and protected branch rules for workflow files. Workflow changes can expand privileges and must be tightly governed.

**Q2.** A manager wants fewer human approvals for production deploys.  
**A:** Keep environment-based approval gates for high-impact actions. Risk, not convenience, should drive autonomy limits.

**Q3.** A security review agent needs to report findings but not push code.  
**A:** Grant comment/report permissions only; do not grant repository write access. Match privileges to the exact action required.

**Q4.** An agent made a harmful change and there is no record of why.  
**A:** Improve auditability using PR rationale fields, workflow logs, and artifacts. Accountability depends on traceability.

**Q5.** A policy is written only in prompt text.  
**A:** Back it with enforceable GitHub controls: required reviews, branch protection, environments, and explicit `permissions:`. Prompts guide behavior; platform controls enforce it.

---

## Relevant Resources

- [GitHub Docs — About branch protection rules](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)
- [GitHub Docs — Managing environments for deployment](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment)
- [GitHub Docs — About OIDC in Actions](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect)
- [GitHub Docs — About CODEOWNERS](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners)
- [GitHub Docs — Automatic token authentication](https://docs.github.com/en/actions/security-for-github-actions/security-guides/automatic-token-authentication)

---

## Deep Dive

Go further with **[Deep Dive 6 — Guardrails and Accountability](../deep-dives/06-guardrails-accountability-deep.md)**, which covers:
- GitHub Environments — full protection rule matrix and REST API configuration
- GitHub Audit Log API — querying agent-attributed events and streaming to a SIEM
- Secret scanning push protection — how it works at the git layer and how agents should handle rejections
- Dependabot and code scanning auto-triage workflows — what agents may and may not do
- Copilot org/enterprise policy settings — seat assignment, MCP allowlists, content exclusions
- Complete hard-stop workflow implementation for PRs touching `.github/workflows/`
- Additional exercises with authoritative GitHub Docs links for each sub-topic
