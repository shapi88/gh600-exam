# GH-600 Practice Exam — Version 1

**Exam version:** v1  
**Date:** 2026-05-19  
**Topics covered:** All 6 (Agent Architecture & SDLC, Tool Use & Environment Interaction, Memory State & Execution, Evaluation & Error Analysis, Multi-Agent Coordination, Guardrails & Accountability)  
**Total questions:** 30 (5 per topic)  
**Passing score:** 80% (24/30)

> **Instructions:** Choose the single best answer for each question. Answers and explanations are in the [Answer Key](#answer-key) at the bottom.

---

## Topic 1 — Agent Architecture and SDLC Processes

**Q1.** A team wants to add a GitHub Copilot agent to their repository. The agent will read open issues, suggest a plan, and open a draft PR — but a human must approve the plan before any code is committed. Which autonomy level best describes this setup?

- A) Level 0 — Suggest only  
- B) Level 1 — Draft plans/content  
- C) Level 2 — Limited repo changes  
- D) Level 4 — Deployment-affecting actions  

---

**Q2.** Which of the following is the correct order of GitHub control plane objects in a governed agent-assisted delivery flow?

- A) PR → branch → issue → CI checks → merge  
- B) Issue → branch → draft PR → CI checks → human review → merge  
- C) Branch → issue → draft PR → merge → CI checks  
- D) Draft PR → issue → branch → human review → merge  

---

**Q3.** An agent is assigned both the planning role and unrestricted execution rights on `main`. Which risk does this most directly introduce?

- A) Increased token usage  
- B) Duplicate PR creation  
- C) Lack of role separation allowing the agent to bypass human-in-the-loop checkpoints  
- D) Slower CI pipeline execution  

---

**Q4.** A developer wants human experts to automatically receive review requests whenever the agent touches `.github/workflows/` files. Which GitHub feature should they configure?

- A) Branch protection required status checks  
- B) CODEOWNERS  
- C) Repository rulesets  
- D) Environment protection rules  

---

**Q5.** During backlog grooming, an agent labels issues and drafts acceptance criteria. At which SDLC stage is this activity most beneficial for an agent operating at Autonomy Level 1?

- A) Production deployment  
- B) Code review  
- C) Release preparation  
- D) Backlog grooming / planning  

---

## Topic 2 — Tool Use and Environment Interaction

**Q6.** An agent workflow uses `actions/checkout@v4`. To prevent the checkout token from being reused in later steps, which setting should be applied?

- A) `persist-credentials: false`  
- B) `fetch-depth: 0`  
- C) `token: ${{ secrets.GITHUB_TOKEN }}`  
- D) `submodules: true`  

---

**Q7.** A workflow grants an agent `write` permissions on `contents` and `pull-requests`, and `read` on everything else. Which principle does this configuration follow?

- A) Defense in depth  
- B) Principle of least privilege  
- C) Zero-trust networking  
- D) Separation of duties  

---

**Q8.** An MCP server registered in `.github/mcp-config.json` provides access to a cloud storage bucket. The agent needs only to read files, never write them. Which scope entry is most appropriate?

- A) `scopes: ["read", "write"]`  
- B) `scopes: ["admin"]`  
- C) `scopes: ["read"]`  
- D) `scopes: []`  

---

**Q9.** A workflow step calls an external API and the call fails with a transient 503 error. The agent retries without any delay or limit. What is the primary risk?

- A) The agent may produce duplicate outputs if the first call succeeded silently  
- B) The workflow will automatically cancel after 30 seconds  
- C) The GITHUB_TOKEN will be revoked  
- D) Branch protection rules will prevent the retry  

---

**Q10.** An agent needs to run tests that require a database. Which GitHub Actions runner configuration best isolates the database from the host environment?

- A) A self-hosted runner with the database installed globally  
- B) A `services:` container defined in the workflow YAML  
- C) A hard-coded connection string in the workflow environment variables  
- D) Storing the database password in a repository variable  

---

## Topic 3 — Memory, State, and Execution

**Q11.** Which memory type has the shortest lifetime and is lost when an agent's context window is reset?

- A) Repository memory  
- B) External database  
- C) Prompt context memory  
- D) PR/issue comments  

---

**Q12.** An agent uploads a JSON summary at the end of each workflow run so that the next run can resume from where it left off. This is an example of which pattern?

- A) User preference memory  
- B) Ephemeral in-memory caching  
- C) Durable artifact-based checkpointing  
- D) Branch-level conflict resolution  

---

**Q13.** A `copilot-instructions.md` file defines reusable agent standing orders for a repository. In which memory category does this file belong?

- A) Prompt context memory  
- B) Repository memory  
- C) User preference memory  
- D) Artifact storage  

---

**Q14.** An agent performs a file update and then fails mid-way. On retry it runs the update again, producing duplicate entries. Which design property was missing?

- A) Parallelism  
- B) Idempotency  
- C) Caching  
- D) OIDC authentication  

---

**Q15.** A long-running agent workflow that deploys to multiple environments needs to resume from the last successful environment if it fails partway through. What is the recommended mechanism in GitHub Actions?

- A) Store progress as a repository secret  
- B) Use a caching action to preserve the runner disk  
- C) Upload a checkpoint artifact after each successful environment, download it on retry  
- D) Set `continue-on-error: true` on every step  

---

## Topic 4 — Evaluation, Error Analysis, and Tuning

**Q16.** An evaluator is grading an agent's response on the Safety dimension. The agent produced correct output but leaked a partial API key in a comment. What is the correct Safety grade?

- A) Pass  
- B) Partial  
- C) Fail  
- D) N/A — Safety only applies to deployment scenarios  

---

**Q17.** A golden scenario test shows that the agent modified three files outside the specified scope. Which error category best describes this behavior?

- A) Hallucinated tools  
- B) Stale context  
- C) Excessive changes / scope violation  
- D) Policy violation  

---

**Q18.** After tuning a prompt, an evaluator runs the same golden scenarios against the new prompt version. Where should the new results be recorded to preserve the audit trail?

- A) Overwrite the existing `results-v1.md` with the new scores  
- B) Create a new `results-v2.md` and add a row to `exam-category.md`  
- C) Update `rubric.md` to match the new scores  
- D) Post the results as a PR comment only, without committing a file  

---

**Q19.** Which of the following is the correct minimum passing condition for a single scenario in the GH-600 evaluation rubric?

- A) All dimensions must receive `Pass`  
- B) Safety must be `Pass`; Correctness, Auditability, and Scope Control each require at least `Partial`  
- C) The total points must be at least 6 out of 8  
- D) Two out of four dimensions must receive `Pass`  

---

**Q20.** An agent evaluation run shows that the agent consistently calls a tool named `get_branch_info` that does not exist in the approved tool list. Which mitigation addresses this root cause most directly?

- A) Increase the context window size  
- B) Add explicit retries  
- C) Restrict the tool list in the agent's configuration and validate tool names before execution  
- D) Upgrade the underlying model  

---

## Topic 5 — Multi-Agent Coordination

**Q21.** In a planner → executor → reviewer pipeline, the executor agent completes a task and needs to hand off its result to the reviewer. Which mechanism provides the most durable and auditable handoff?

- A) An in-memory variable shared across jobs  
- B) A workflow artifact uploaded by the executor and downloaded by the reviewer  
- C) A Slack message from the executor to the reviewer  
- D) A direct API call between agents in the same run  

---

**Q22.** A coordinator agent receives conflicting change proposals from two specialist agents that both modified the same file. What is the recommended resolution strategy?

- A) Apply the change from the agent that ran first  
- B) Merge all changes automatically without review  
- C) Escalate to a human reviewer and block execution until resolved  
- D) Delete the file and start over  

---

**Q23.** An agent workflow uses a reusable workflow (`workflow_call`) as the handoff contract between a planner and an executor. What is the key benefit of this approach?

- A) It reduces the number of workflow files in the repository  
- B) It enforces a defined input/output interface and allows independent versioning of each agent step  
- C) It allows the planner to bypass branch protection rules  
- D) It eliminates the need for CODEOWNERS  

---

**Q24.** A multi-agent pipeline is designed so the executor cannot run unless the planner's output artifact is present and passes a schema validation step. This gate enforces which principle?

- A) Parallelism  
- B) Dependency injection  
- C) Contract-based handoff with validation  
- D) Secret scanning  

---

**Q25.** In a specialist swarm with a coordinator pattern, what is the primary responsibility of the coordinator agent?

- A) Running all specialist tasks sequentially  
- B) Writing code on behalf of the specialists  
- C) Synthesizing results from specialists and resolving conflicts before final output  
- D) Replacing human review for production deployments  

---

## Topic 6 — Guardrails and Accountability

**Q26.** An agent attempts to edit `.github/workflows/deploy.yml`. According to GH-600 governance best practices, what should happen?

- A) The edit proceeds automatically if the workflow syntax is valid  
- B) The agent stops, posts a comment requesting human review, and waits for approval  
- C) The agent creates a new workflow file instead of modifying the existing one  
- D) The agent deletes the file and recreates it on a new branch  

---

**Q27.** A production deployment triggered by an agent fails and causes a service outage. The first step in the incident response runbook is to:

- A) Update the prompt instructions to prevent recurrence  
- B) Close or revert the PR and delete the agent branch  
- C) Escalate to the MCP server administrator  
- D) Increase the agent's autonomy level for faster fixes  

---

**Q28.** Which combination of GitHub controls provides the strongest enforcement for the following requirement: "No code may be deployed to production without a named human approving the deployment"?

- A) Branch protection required reviews only  
- B) `environment: production` with a required reviewer AND branch protection required reviews  
- C) A prompt instruction telling the agent not to deploy  
- D) CODEOWNERS on the deployment workflow file only  

---

**Q29.** An agent-produced PR has no description, no linked issue, and no PR comment explaining the change. Which governance requirement does this violate?

- A) OIDC authentication  
- B) Idempotency  
- C) Audit trail requirement — every agent action must produce an attributable record  
- D) Least-privilege permissions  

---

**Q30.** A team stores all governance policy exclusively in `copilot-instructions.md` with no corresponding branch protection rules or CODEOWNERS entries. What is the main risk of this approach?

- A) The instructions file will be too large for the context window  
- B) The policy is only advisory and can be bypassed if the agent ignores the instructions  
- C) Branch protection rules will conflict with the instructions  
- D) OIDC tokens will be invalidated  

---

## Answer Key

| Q | Answer | Explanation |
|---|--------|-------------|
| 1 | **B** | Drafting a plan and opening a draft PR without committing code matches Level 1 — Draft plans/content. A human approves before any state change. |
| 2 | **B** | The canonical governed flow is issue → branch → draft PR → CI checks → human review → merge. |
| 3 | **C** | Combining planning and unrestricted execution removes the separation of duties that enables human-in-the-loop checkpoints before code lands. |
| 4 | **B** | CODEOWNERS routes review requests to specified teams or individuals based on file paths. |
| 5 | **D** | Backlog grooming (labeling, acceptance-criteria drafting) is explicitly listed as an appropriate SDLC placement for an agent at Level 1. |
| 6 | **A** | `persist-credentials: false` prevents the checkout token from being stored and reused in subsequent steps. |
| 7 | **B** | Granting only the permissions actually needed is the principle of least privilege. |
| 8 | **C** | Read-only scope (`"read"`) matches the agent's actual need and avoids granting write access unnecessarily. |
| 9 | **A** | Unlimited retries on a call that may have silently succeeded can produce duplicate side effects (e.g., duplicate commits, duplicate API records). |
| 10 | **B** | A `services:` container runs in an isolated Docker network scoped to the workflow job, keeping the database off the host. |
| 11 | **C** | Prompt context memory exists only for the current run and is lost on context reset. |
| 12 | **C** | Uploading a JSON summary that the next run downloads is durable artifact-based checkpointing. |
| 13 | **B** | `copilot-instructions.md` is a repository-scoped persistent file — it is repository memory. |
| 14 | **B** | Idempotency means running the same operation multiple times produces the same result. Without it, retries cause duplicate entries. |
| 15 | **C** | Uploading a checkpoint artifact after each successful environment and downloading it on retry is the recommended pattern in GitHub Actions. |
| 16 | **C** | The Safety dimension has no Partial grade — leaking credentials is a hard Fail regardless of other dimension scores. |
| 17 | **C** | Modifying files outside the intended scope is classified as excessive changes / scope violation. |
| 18 | **B** | Results are always committed to a new versioned file (`results-v2.md`) to preserve the audit trail; existing result files are never overwritten. |
| 19 | **B** | The rubric requires Safety = Pass and each of Correctness, Auditability, Scope Control ≥ Partial for a scenario to pass. |
| 20 | **C** | Restricting and validating the tool list prevents the agent from calling non-existent tools (hallucinated tool error). |
| 21 | **B** | A workflow artifact is durable, stored in GitHub, and downloadable by downstream jobs — the most auditable handoff mechanism. |
| 22 | **C** | Conflicting proposals on the same file require human judgment; the agent must escalate rather than arbitrarily apply one change. |
| 23 | **B** | Reusable workflows enforce a typed input/output interface and can be versioned independently, providing a stable handoff contract. |
| 24 | **C** | Requiring an upstream artifact and validating its schema before proceeding is contract-based handoff with validation. |
| 25 | **C** | The coordinator synthesizes specialist results, resolves conflicts, and produces the final output — it does not execute specialist tasks itself. |
| 26 | **B** | Editing `.github/workflows/` is a hard-stop condition; the agent must pause and request human review. |
| 27 | **B** | The first step in the incident response runbook is to stop the damage: close/revert the PR and delete the agent branch. |
| 28 | **B** | Environment protection with a required reviewer enforces the approval gate at deploy time; branch protection required reviews enforce it at merge time. Both together provide the strongest coverage. |
| 29 | **C** | Every agent action must produce an attributable record (PR description, PR comment, or workflow log). A silent write with no attribution violates the audit trail requirement. |
| 30 | **B** | Policy stored only in prompts is advisory and cannot be mechanically enforced. An agent that ignores or misinterprets the instructions can bypass it. Platform controls (branch protection, CODEOWNERS) are mandatory to make governance enforceable. |
