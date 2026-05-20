# GH-600 Practice Exam — Version 3 (Hard)

**Exam version:** v3  
**Date:** 2026-05-19  
**Difficulty:** Hard  
**Topics covered:** All 6 (Agent Architecture & SDLC, Tool Use & Environment Interaction, Memory State & Execution, Evaluation & Error Analysis, Multi-Agent Coordination, Guardrails & Accountability)  
**Total questions:** 42 (7 per topic)  
**Suggested time:** 120 minutes (~2.9 min per question)  
**Passing score:** 80% (34/42)

> **Instructions:** Choose the single best answer for each question. All questions require multi-concept reasoning or scenario analysis. Answers and explanations are in the [Answer Key](#answer-key) at the bottom.

---

## Topic 1 — Agent Architecture and SDLC Processes

**Q1.** An agent at Autonomy Level 2 is configured to auto-approve its own PRs using a service account that is listed as a CODEOWNERS reviewer for all file paths. This was done to reduce friction during testing. What is the fundamental governance violation?

- A) Service accounts cannot be added to CODEOWNERS entries  
- B) An agent self-approving its own PRs eliminates the human-in-the-loop requirement, defeating the entire purpose of CODEOWNERS and required-review governance  
- C) Auto-approval requires an Autonomy Level 3 configuration  
- D) CODEOWNERS review requirements apply only to non-agent branches  

---

**Q2.** A startup uses a single monorepo for all services. An agent added for Team A can edit any file in the repository. Six months later a second agent is added for Team B. Team A's agent accidentally modifies Team B's service directory. Which architectural change prevents cross-team agent interference going forward?

- A) Assign both agents the same `GITHUB_TOKEN` so their changes are attributed identically  
- B) Partition the monorepo with per-team CODEOWNERS entries and scope each agent's permitted file paths to its team's directories only  
- C) Run both agents in the same workflow job so they can detect each other's changes  
- D) Give Team A's agent read-only access to the entire repository  

---

**Q3.** A team has configured their agent to: open issues during backlog grooming, create branches during sprint planning, draft PRs during development, and merge approved PRs after review. The team now wants the agent to also trigger production deployments immediately after merge. Which governance boundary does this final step cross?

- A) Production deployment requires Autonomy Level 4 — Deployment-affecting actions — which exceeds the authorized autonomy of most governed agents  
- B) Deployment is permitted at Level 2 if all CI checks pass  
- C) Only the merge step itself requires elevated permissions  
- D) Staging and production deployments are equivalent from a governance perspective  

---

**Q4.** An engineering manager wants to trace which SDLC decision points an agent participated in over the past quarter. The repository has branch protection and CI, but no structured audit logging for agent actions. What is the minimum required artifact set to enable this traceability?

- A) The git commit history alone provides sufficient traceability  
- B) Each agent-initiated PR must include a description linking the originating issue, the approach rationale, and a log of what the agent verified before opening the PR  
- C) The manager should access the agent's local context window history  
- D) Quarterly sprint reports replace the need for per-action audit records  

---

**Q5.** A CTO mandates that agents in production-adjacent workflows must pass a security review before activation. The existing onboarding process only validates prompt quality. Which step must be added to the security review?

- A) Test the agent exclusively on synthetic data before enabling it  
- B) Review the agent's permission scope, tool list, MCP servers it connects to, and hard-stop conditions — each independently validated against the principle of least privilege  
- C) Require the agent to write its own security assessment  
- D) Run the agent at Level 0 for 30 days before elevating to Level 2  

---

**Q6.** An agent creates a branch, opens a PR, and requests review. The reviewer approves without examining the diff because the agent's PR description reads "routine maintenance." After merge, a critical configuration file has been altered. What process control failed?

- A) The agent should have been prevented from creating PRs for configuration files  
- B) The PR description did not accurately reflect the scope of changes; a required changed-files summary or mandatory diff-acknowledgment policy would have surfaced the discrepancy  
- C) The reviewer's approval was technically invalid because it was given too quickly  
- D) Routine-maintenance PRs should be auto-merged to reduce reviewer burden  

---

**Q7.** An agent at Autonomy Level 2 is tasked with setting up a new repository: create the repo, configure branch protection, add CODEOWNERS, and create the initial CI workflow. It completes the first three steps but the CI workflow step requires writing to `.github/workflows/`. How should the agent handle the remaining work?

- A) Complete all steps autonomously since the repository is new and has no existing governance to violate  
- B) Complete repository creation, branch protection, and CODEOWNERS, then stop at the CI workflow step and post a human-review request specifying exactly which workflow file requires human authorship  
- C) Create a placeholder workflow file that contains only comments and no executable steps  
- D) Request a temporary Autonomy Level 3 elevation to complete the workflow creation  

---

## Topic 2 — Tool Use and Environment Interaction

**Q8.** A workflow step uses `${{ github.event.issue.body }}` directly in a `run:` shell command: `run: ./process.sh "${{ github.event.issue.body }}"`. A user opens an issue with the body `"; rm -rf /workspace; echo "`. What vulnerability is this?

- A) SQL injection via the GitHub API  
- B) Shell injection (command injection); user-controlled data from `github.event.*` must never be interpolated directly into `run:` commands — it must be passed via environment variables with proper quoting  
- C) Cross-site scripting in the workflow UI  
- D) Path traversal via the shell argument  

---

**Q9.** An agent workflow uses `actions/checkout@v4` and then calls `git push`. The workflow triggers on `pull_request` events from external forks. What security risk arises if the workflow uses a PAT with write permissions instead of the default `GITHUB_TOKEN`?

- A) External fork PRs cannot trigger the `pull_request` event  
- B) A fork PR workflow that uses an elevated PAT could execute the `git push` step with write credentials, allowing a malicious fork to push content to the base repository  
- C) `actions/checkout@v4` does not support checking out fork branches  
- D) `git push` is blocked by default on all `pull_request`-triggered workflows  

---

**Q10.** A workflow uses `if: ${{ secrets.DEPLOY_TOKEN != '' }}` to gate a deployment step. A developer notices that setting a repository _variable_ (not a secret) named `DEPLOY_TOKEN` does not affect the condition. Why not, and what is the real risk to guard against?

- A) Repository variables and secrets share the same namespace so the condition would be affected  
- B) Secrets (`secrets.*`) and variables (`vars.*`) have separate namespaces; the condition correctly uses only the secret namespace. The real risk is accidentally storing the token as a variable rather than a secret, which would expose its value in plain text  
- C) The `if:` expression is never evaluated for secrets-based conditions  
- D) Only organization secrets can be read with `secrets.*` in a condition  

---

**Q11.** An agent workflow calls an MCP server that requires an API key stored in `secrets.MCP_API_KEY`. A workflow run from an external fork PR reaches the step that uses the MCP server. What security control must be in place to prevent secret exposure?

- A) External fork PRs automatically have access to all repository secrets  
- B) External fork PRs must not have access to repository secrets; workflows that use sensitive secrets must run on `pull_request_target` with explicit human-approval gates, or restrict MCP calls to trusted base-branch workflows only  
- C) MCP servers in GitHub Actions do not require API key authentication  
- D) The secret must be stored as a repository variable to be readable by fork PRs  

---

**Q12.** A developer pins all GitHub Actions to commit SHAs rather than version tags (e.g., `uses: actions/checkout@abc1234`). A security researcher notes this does not fully eliminate supply chain risk. Why?

- A) Commit SHAs change when a repository branch is force-pushed  
- B) SHA pinning guarantees only the action code at that commit; if the action fetches its own runtime dependencies (e.g., npm packages) at runtime rather than bundling them at the pinned commit, malicious transitive packages can still be injected  
- C) GitHub does not support SHA pinning for actions hosted outside `github.com`  
- D) SHA pinning is inherently less secure than semantic version tag pinning  

---

**Q13.** An agent needs to write files to an S3 bucket from GitHub Actions. The team configures an IAM role with `s3:*` on all buckets, authenticated via OIDC. A security review flags this configuration. What is the least-privilege corrective action?

- A) Replace OIDC with a long-lived IAM access key that also has `s3:*` permissions  
- B) Create a new IAM role with only `s3:PutObject` scoped to the target bucket ARN, and configure the OIDC trust policy to accept only the specific repository and branch that runs this workflow  
- C) Grant the IAM role `s3:*` but restrict the OIDC subject claim to the repository name only  
- D) Store static S3 credentials in GitHub environment secrets rather than using OIDC  

---

**Q14.** A GitHub Actions workflow uses `continue-on-error: true` on a mandatory security-scan step. When the security scan fails, the workflow continues to deployment. What governance violation does this create?

- A) `continue-on-error: true` is not permitted in deployment workflows  
- B) `continue-on-error: true` on a security gate effectively disables it; required security checks must fail the entire workflow when they fail — a bypassed gate provides false assurance that the security requirement was met  
- C) Security scans must always run after deployment, not before  
- D) The deployment step must be in a separate workflow to be a valid gate  

---

## Topic 3 — Memory, State, and Execution

**Q15.** An agent reads a feature-flag configuration file at the start of each run. A concurrent automated PR updates and merges the same file while the agent is mid-run. The agent continues using the old flag values for the remainder of the run. What is the most accurate description of this failure mode and the best mitigation?

- A) Race condition between a concurrent write and the agent's cached read; the agent should re-read configuration at each decision point where flag values are critical, or implement a run-level configuration lock  
- B) The feature-flag system should be replaced with environment variables  
- C) The agent should abort if any repository file changes during a run  
- D) The configuration file should be stored in a secret rather than a repository file  

---

**Q16.** A multi-run agent stores all its decisions in PR comments. The PR is squash-merged, collapsing the comment history. The next run finds no PR comments to reference. Which memory design is most resilient to this kind of information loss?

- A) Always use merge commits instead of squash merge  
- B) Persist critical agent decisions as committed files (e.g., `docs/agent-decisions/`) before closing the PR, so the information survives regardless of merge strategy  
- C) Store decisions in the workflow run logs, which are always retained  
- D) Keep the PR permanently in draft mode to prevent squash merges  

---

**Q17.** An agent caches model weights between runs using `actions/cache` with a key of `model-weights-${{ github.sha }}`. A new cache entry is created after every commit and old entries are never reused. Storage fills up and older entries are evicted. What cache key strategy would improve hit rate while maintaining freshness?

- A) Use a fixed key such as `model-weights-v1` that never expires  
- B) Base the key on a slow-changing property like a dedicated weights version file (`model-weights-${{ hashFiles('model-version.txt') }}`), combined with a restore-key fallback, so the cache is only invalidated when the weights actually change  
- C) Disable caching entirely and re-download weights on every run  
- D) Use `github.run_number` as part of the cache key  

---

**Q18.** A stateful agent uses a GitHub issue body as a task queue by appending JSON task objects. After 100 tasks the issue body exceeds GitHub's character limit and new tasks are silently dropped. Which architectural change addresses this scalability ceiling?

- A) Increase the issue body character limit via the GitHub Enterprise API  
- B) Use the Issues API for task metadata only (title, labels, state) and store task payloads as structured artifact files or in an external queue; never use issue bodies as a high-volume data store  
- C) Create a new issue for each task  
- D) Compress the JSON payload before appending it to the issue body  

---

**Q19.** An agent checks for existing PRs before creating a new one using the PR title as the deduplication key. A user intentionally creates a PR with the same title but a different purpose. The agent finds the matching title, skips creation, and the intended work is never done. What deduplication strategy is more robust?

- A) Use the PR body content for deduplication  
- B) Use a combination of the source branch name, base branch name, and a unique identifier derived from the originating task or issue number — not just the title  
- C) Require all PR titles to be repository-unique at the platform level  
- D) Disable idempotency checking and always create a new PR  

---

**Q20.** An agent workflow fails on step 7 of 10. On retry it successfully re-runs idempotent steps 1–7 before failing again on step 8. Steps 1–7 are slow and make unnecessary API calls on each retry. What optimization preserves idempotency while reducing redundant work?

- A) Remove idempotency checks from steps 1–7 to speed up retries  
- B) Upload a validated checkpoint artifact after step 7; on retry, download the checkpoint and skip steps 1–7 entirely  
- C) Run all 10 steps in parallel to reduce wall-clock time on the first attempt  
- D) Increase the job timeout so retries have more time to complete  

---

**Q21.** An agent reads from a central "known issues" file in a shared configuration repository using a `raw.githubusercontent.com` URL. Repository A's agent updates the file, but Repository B's agent reads it 2 minutes later and still sees the old version due to CDN caching. What is the root cause and the correct fix?

- A) Repository B's agent should poll the URL more frequently  
- B) Raw file CDN URLs can serve stale content; agents must read via the GitHub Contents API (`/repos/{owner}/{repo}/contents/{path}`) to retrieve the latest committed version, bypassing the CDN cache  
- C) The shared configuration repository should be in the same organization as both agents  
- D) Use Git submodules to synchronize the file across repositories  

---

## Topic 4 — Evaluation, Error Analysis, and Tuning

**Q22.** An agent achieves a 100% pass rate across all 10 golden scenarios for the fifth consecutive prompt version. An evaluator recommends retiring the evaluation process since the agent has reached maximum quality. Why is this recommendation dangerous?

- A) A 100% pass rate is statistically impossible for a probabilistic model  
- B) Sustained 100% on a fixed scenario set may indicate the prompt has been overfitted to those exact inputs; new unseen scenarios must be introduced to validate generalization, and the evaluation process should never be retired for an active agent  
- C) The evaluation should continue but with a lower passing threshold to surface marginal failures  
- D) Five consecutive perfect runs are statistically insufficient for a retirement decision  

---

**Q23.** An agent generates syntactically correct Python code that passes all unit tests, but the code uses an API method deprecated and scheduled for removal in the next minor release. How should the evaluator grade this?

- A) Pass — the code is correct and all tests pass  
- B) Correctness should be graded Partial (or Fail if the acceptance criteria explicitly required current API usage), with a Note flagging the deprecated method as a tuning target  
- C) Safety should be graded Fail because deprecated APIs introduce security risk  
- D) The result is inconclusive until the next minor version is released  

---

**Q24.** A prompt engineer observes a 5% Correctness improvement after tuning. Before declaring success, which additional checks are required per GH-600 evaluation best practices?

- A) Run the new prompt on at least 100 additional novel scenarios  
- B) Confirm no other dimension regressed, run the full scenario set (not a subset), record results in a new versioned results file, and ensure the model version is identical to the previous evaluation run  
- C) Request a second independent evaluator to confirm the Correctness scores  
- D) Deploy the new prompt to staging and measure real-world performance before finalizing  

---

**Q25.** The rubric uses "at least Partial" as the minimum passing grade for Correctness. An agent returns the correct file change but also adds an unrequested explanatory comment to the PR description. The evaluator grades Correctness=Pass. A reviewer argues the extra content should penalize the Correctness score. Which principle resolves the dispute?

- A) The reviewer is correct; any output beyond the literal requirements is a Correctness deduction  
- B) The rubric should be applied literally: if the acceptance criterion only requires the correct file change, and the extra PR comment doesn't violate any other dimension, Correctness=Pass is defensible; the extra content is assessed under the Scope dimension, not Correctness  
- C) The evaluator and reviewer should average their independent grades  
- D) The scenario acceptance criteria must be rewritten before a final verdict is possible  

---

**Q26.** An organization evaluates three models (A, B, C) on the same prompt and scenario set. Model A scores 90%, B scores 80%, C scores 85%. The team immediately selects Model A. An evaluator objects. What is the most likely basis for the objection?

- A) The scores should be averaged before making a selection decision  
- B) A single evaluation run may be insufficient given model non-determinism; each model should be evaluated across multiple runs with consistency metrics before a selection decision is made  
- C) Model selection should be driven by cost per token, not evaluation scores  
- D) All three models exceed the 80% passing threshold so they are equivalent choices  

---

**Q27.** An evaluator notices that an agent's Auditability scores are consistently high because the agent copies its input instructions verbatim into the PR description rather than providing genuine decision rationale. What evaluation action is required?

- A) Accept the high Auditability scores since the instructions are present in the PR description  
- B) Clarify the Auditability grade criteria to require genuine decision rationale (what was decided and why) rather than verbatim instruction copy; re-grade affected runs under the clarified criteria and update the scenarios  
- C) Add a new Auditability sub-dimension specifically for "genuine vs. verbatim" rationale  
- D) Discard all existing Auditability scores and restart evaluation from scratch  

---

**Q28.** A team's agent always scores Scope Control=Fail on monorepo scenarios because the agent modifies shared utility files that are technically "unrelated" to the immediate task but required by the build system. How should the evaluation process handle this?

- A) Override Scope Control to Pass for all monorepo scenarios as a blanket exception  
- B) Refine the scenario acceptance criteria to explicitly list which shared utility files are considered in-scope for each task, so the rubric can be applied precisely  
- C) Remove the Scope Control dimension entirely from monorepo evaluations  
- D) Accept Scope Control=Partial as the new baseline passing grade for all monorepo scenarios  

---

## Topic 5 — Multi-Agent Coordination

**Q29.** A coordinator dispatches tasks to five specialists and collects results via artifacts. One specialist uploads an artifact with an unexpected schema version. The coordinator silently processes the malformed artifact and produces corrupted output. What defensive design was missing?

- A) The coordinator should automatically retry the specialist that uploaded the wrong schema version  
- B) The coordinator must validate every incoming artifact against the expected schema version before processing; mismatched versions must be rejected and escalated rather than silently processed  
- C) Specialists should post results as PR comments instead of uploading artifacts  
- D) Schema versions should be removed from artifacts to increase pipeline flexibility  

---

**Q30.** In a planner-executor architecture, the planner produces a plan that requires 2.5 hours to execute. The executor job hits the organization's 2-hour job timeout after completing 80% of the plan. The partial execution leaves the system in an inconsistent state. Which design pattern would have prevented this?

- A) Request a job timeout exemption from GitHub support  
- B) Break the plan into discrete, independently resumable sub-tasks with checkpoint artifacts; each executor job must fit within the timeout boundary, handing unfinished tasks to subsequent triggered jobs  
- C) Run the executor on a self-hosted runner with no enforced timeout  
- D) Increase `timeout-minutes` in the workflow YAML to accommodate the full plan  

---

**Q31.** Two specialist agents simultaneously upgrade a shared library dependency — Agent A to v2.0.0 and Agent B also to v2.0.0 but with different configuration options. The coordinator merges both PRs without detecting the incompatibility. The build breaks. What coordination mechanism was missing?

- A) Both specialists should have operated on separate repository forks  
- B) A dependency-lock conflict detection step in CI combined with a coordinator rule that prevents merging a PR touching shared dependency configuration while another open PR modifies the same dependency  
- C) Agent A and Agent B should have communicated directly via API calls  
- D) The shared library should be removed to eliminate coordination overhead  

---

**Q32.** A monitoring agent can trigger a rollback by dispatching a `repository_dispatch` event. The trigger condition occasionally fires on benign metric spikes, causing three false-positive rollbacks in one week. Which design improvement reduces false positives while preserving the rollback capability?

- A) Disable the monitoring agent during normal business hours  
- B) Add a human confirmation gate before the rollback executes, and improve the anomaly detection logic to distinguish known benign spike patterns from genuine regressions  
- C) Lower the rollback trigger threshold to reduce sensitivity  
- D) Replace the monitoring agent with a manual monitoring dashboard  

---

**Q33.** In an agent pipeline, a planner produces 50 tasks as a JSON artifact. An executor processes them sequentially, uploading checkpoints every 10 tasks. At task 30, an upstream API becomes rate-limited for 1 hour. The executor job times out waiting for the rate limit to lift and the 30 completed tasks are lost. What checkpoint strategy would have preserved the completed work?

- A) Process all 50 tasks in parallel to avoid the rate-limit impact  
- B) When the executor detects the rate-limit condition, it should immediately upload the current checkpoint artifact (tasks 1–30 complete) and fail the step gracefully so the next retry job can download the checkpoint and resume from task 31  
- C) Retry the rate-limited API call indefinitely until the limit resets  
- D) Pre-fetch all API responses in a separate step before running the executor  

---

**Q34.** A coordinator compiles deployment parameters by merging outputs from four specialists, each output signed with a shared HMAC. The coordinator validates each HMAC. A sophisticated attacker compromises one specialist's environment and replaces its output artifact after signing but before the coordinator downloads it. What additional control detects this artifact tampering?

- A) Switch from HMAC to RSA digital signatures  
- B) In addition to HMAC validation, the coordinator should verify the artifact's GitHub-provided SHA-256 checksum via the API after download; a mismatch indicates post-signature tampering  
- C) The coordinator should re-run all specialists before consuming their outputs  
- D) HMAC validation alone is sufficient if the shared secret is sufficiently long  

---

**Q35.** An organization runs nightly multi-agent pipelines that each produce 5–10 artifacts. After six months, artifact storage costs have tripled. No artifact retention policy exists. Which governance addition most directly addresses this?

- A) Reduce the number of agents in each pipeline to lower artifact volume  
- B) Define a repository-level artifact retention policy aligned with compliance requirements, implement a scheduled cleanup workflow for artifacts older than the retention window, and document the policy in the governance files  
- C) Compress all artifacts before uploading to reduce storage size  
- D) Switch to self-hosted runners, which include free artifact storage  

---

## Topic 6 — Guardrails and Accountability

**Q36.** An agent PR modifies both `src/auth.py` and `config/auth-config.yaml`. CODEOWNERS assigns `@security-team` to `config/` and `@dev-team` to `src/`. Both teams must approve. The security team approves; under deadline pressure the dev team's lead merges without the second approval. Which platform control would have prevented the premature merge?

- A) Branch protection must require ALL CODEOWNERS-designated teams to approve before merge is permitted; the merge happening without the dev team's approval indicates the "Require review from Code Owners" branch protection rule was not enforced or was bypassed  
- B) The security team should have blocked the merge by dismissing their own approval  
- C) CODEOWNERS does not support requiring multiple teams to approve a single PR  
- D) The agent should not have combined the two file types in a single PR  

---

**Q37.** A team's incident response runbook requires filing an `agent-incident.md` within 24 hours of any agent-caused incident. An incident is detected 6 hours after occurrence, but the template file is missing from the repository. What should the team do?

- A) Skip the incident report until the template is restored  
- B) File the report immediately using any structured format that captures the same information the template would have required (timeline, impact, root cause, remediation, prevention), and restore the template as part of the remediation  
- C) Wait until the template is restored before filing the report  
- D) File a generic bug report as a temporary placeholder  

---

**Q38.** An agent's PR description states "no security-sensitive files were modified." The actual diff includes changes to `config/secrets-rotation-policy.yaml`. The CODEOWNERS rule for this path was missing, so no security reviewer was auto-assigned, and the PR was merged. Which two controls together would have caught this?

- A) The agent should have been blocked from modifying any YAML files  
- B) CODEOWNERS must cover all security-sensitive file patterns, AND the agent must include a verifiable changed-files manifest in its PR description so reviewers can confirm the claimed scope matches the actual diff  
- C) Branch protection required status checks would have caught the divergence  
- D) The audit trail requirement was satisfied because the PR description was present  

---

**Q39.** A scheduled agent workflow generates weekly reports using `GITHUB_TOKEN`. Three months after deployment, the team discovers the workflow has been producing reports with stale data because an external API endpoint changed. The workflow always exits with code 0, so no alert was raised. What governance artifact was missing?

- A) An alerting rule configured in the external monitoring system  
- B) Scheduled agent workflows must include output validation checks (not just exit code), with failures posted to a notification channel; a periodic human review of scheduled agent outputs should also be part of governance policy  
- C) The `GITHUB_TOKEN` should have been replaced with a long-lived PAT  
- D) Scheduled workflows should not call external APIs  

---

**Q40.** A governance reviewer examines 20 agent-produced PRs and finds that 18 of them have identical descriptions: "Automated change by agent — see commit diff." Each technically has a PR description, but none contains a linked issue, a scope summary, or decision rationale. How should the governance policy be strengthened?

- A) The descriptions are compliant since they name the agent and point to the diff  
- B) The audit trail policy must require minimum content: the originating issue number, a scope summary listing which files changed and why, and the decision rationale — generic boilerplate that only references the diff is explicitly non-compliant  
- C) Add a minimum character count requirement for all PR descriptions  
- D) Require two human reviewers for every agent-produced PR  

---

**Q41.** An organization layers agent governance across three controls: (1) `copilot-instructions.md`, (2) CODEOWNERS, and (3) branch protection rules. A new team member accidentally deletes the CODEOWNERS file and pushes the deletion to `main`. For two hours, agent PRs are merged without any code-owner review. What should the governance policy have included to prevent this?

- A) The CODEOWNERS file should be stored in a separate repository  
- B) The CODEOWNERS file itself must appear in CODEOWNERS under a protected owner group, and its deletion must be blocked by a branch protection rule or repository ruleset requiring administrator approval  
- C) All team members should be restricted to read-only access to the repository  
- D) GitHub automatically protects the CODEOWNERS file and it cannot be deleted without admin rights  

---

**Q42.** An agent successfully completes a task, opens a PR with a detailed description, and the PR is reviewed by a human and merged. Three days later the change causes a performance regression and the team initiates rollback per the incident runbook. Under what condition is it acceptable for the agent to autonomously create the revert commit?

- A) The agent can always create revert commits since reverting is inherently lower-risk than the original change  
- B) The agent may create the revert commit only if: the revert diff fits within the applicable scope limits, no protected files are touched, and a human explicitly authorizes the agent to perform the revert as part of the incident response approval process  
- C) Revert commits never require human oversight because they undo previously approved changes  
- D) The agent may create the revert commit only if it was also the author of the original commit  

---

## Answer Key

| Q | Answer | Explanation |
|---|--------|-------------|
| 1 | **B** | Self-approval by an agent using a service account it controls eliminates the human-in-the-loop checkpoint. CODEOWNERS and required reviews exist precisely to force a human to evaluate changes before they land. |
| 2 | **B** | CODEOWNERS scoped per-team directory, with each agent's writable paths constrained to its team's directories, prevents cross-team write interference without requiring manual coordination. |
| 3 | **A** | Production deployment is a Level 4 — Deployment-affecting action. An agent authorized only at Level 2 must not trigger production deployments autonomously. |
| 4 | **B** | Structured PR descriptions linking the originating issue, approach rationale, and pre-merge verification log provide the minimum evidence trail for retrospective traceability. |
| 5 | **B** | A security review must cover the full attack surface: permissions, tools, external integrations (MCP servers), and the hard-stop conditions that limit autonomous action. Prompt quality alone is insufficient. |
| 6 | **B** | A changed-files summary or mandatory diff-acknowledgment policy requires reviewers to confront the actual scope of a change before approving, preventing misleading PR descriptions from masking broad changes. |
| 7 | **B** | `.github/workflows/` is a hard-stop path. The agent must complete the steps it is authorized to perform, stop at the boundary, and clearly communicate to a human exactly what action is needed and why. |
| 8 | **B** | Interpolating `github.event.*` data directly into a `run:` command allows any event contributor (e.g., an issue author) to inject arbitrary shell commands. Environment variable passing with proper quoting neutralizes this. |
| 9 | **B** | A fork PR workflow that uses elevated credentials (PAT or a non-read-only token) could execute `git push` and write to the base repository. The default `GITHUB_TOKEN` for fork PRs is read-only, but misconfigured PAT use removes this protection. |
| 10 | **B** | `secrets.*` and `vars.*` are distinct namespaces. The condition is correct as written. The real risk is accidentally creating a variable with the same name as a secret, which would expose the value in plain text in workflow logs. |
| 11 | **B** | Repository secrets are not accessible to fork PR workflows by design. Workflows that require secrets must use `pull_request_target` with explicit human approval gates to prevent unauthorized secret access from external forks. |
| 12 | **B** | SHA pinning guarantees the action code but not its runtime dependencies. If the action installs npm packages at runtime, those packages are fetched fresh on every run and can be compromised between the pin date and the run date. |
| 13 | **B** | `s3:PutObject` on a specific bucket ARN with an OIDC trust policy scoped to the exact repository and branch applies both least-privilege permission scoping and identity verification at the cloud layer. |
| 14 | **B** | `continue-on-error: true` on a gate silently absorbs failures. A security scan that fails but allows the workflow to proceed provides false confidence and effectively removes the control it was meant to enforce. |
| 15 | **A** | The agent cached the configuration at run start. A concurrent merge updated the file mid-run, creating a race condition between the cached read and the external write. Re-reading at critical decision points or locking the file are the mitigations. |
| 16 | **B** | Committed files in the repository survive all merge strategies and PR lifecycle events. They are the most durable memory tier available in a GitHub-hosted workflow context. |
| 17 | **B** | Keying on the weights version file's checksum couples cache invalidation to actual weight changes rather than arbitrary commits. The restore-key fallback allows partial cache reuse when the version file hasn't changed. |
| 18 | **B** | Issue bodies are not a scalable data store. The GitHub API is designed for metadata (labels, state, titles). High-volume data belongs in artifacts or external queuing systems. |
| 19 | **B** | A PR title is easily duplicated intentionally or accidentally. A composite key based on source branch, base branch, and originating task ID is far more collision-resistant and harder to manipulate. |
| 20 | **B** | Uploading a checkpoint after fully validating steps 1–7 allows retries to resume from step 8 without repeating expensive idempotent work, reducing cost and latency while preserving correctness guarantees. |
| 21 | **B** | CDN-cached raw file URLs can serve content that is minutes to hours stale. The GitHub Contents API returns the latest committed version by default and supports commit-SHA-pinned requests for exact version control. |
| 22 | **B** | A fixed golden set can be overfitted through repeated prompt tuning. 100% on a known set indicates the agent may be narrow rather than broadly capable. New held-out scenarios are the only way to test for genuine generalization. |
| 23 | **B** | Correctness must be graded against the actual acceptance criteria. If the criteria require current API usage, deprecated API usage is at most Partial. Even without that explicit criterion, a deprecated API represents a known correctness risk worth flagging. |
| 24 | **B** | A 5% improvement is only meaningful if no other dimensions regressed, the full scenario set was used, the model version was identical, and results were committed to a new versioned file for audit purposes. |
| 25 | **B** | Rubric dimensions are graded independently. Extra PR description content that doesn't fail any stated criterion is assessed under Scope, not Correctness. Applying Correctness literally keeps dimension grades clean and comparable across runs. |
| 26 | **B** | Probabilistic model outputs vary across runs. A single evaluation run per model doesn't distinguish genuine capability differences from run-to-run variance. Multi-run evaluation with consistency metrics is required for reliable model selection. |
| 27 | **B** | Auditability requires genuine decision rationale, not verbatim instruction playback. Without clarifying the criteria, the metric is measuring copy-paste ability rather than reasoning transparency. Re-grading under the corrected criterion preserves evaluation integrity. |
| 28 | **B** | Scope Control is applied against stated acceptance criteria. Refining the criteria to list in-scope shared files gives the rubric a precise target, avoiding false failures and making evaluation results meaningful for monorepo workflows. |
| 29 | **B** | Schema validation without version checking allows processing of structurally valid but version-incompatible artifacts. Explicit version validation before processing prevents silent corruption from schema drift between agent versions. |
| 30 | **B** | Long-running plans must be decomposed into checkpoint-based sub-tasks that each fit within the job timeout. This allows progress to be preserved across multiple job executions without exceeding the organization's runtime policy. |
| 31 | **B** | Dependency conflicts between concurrent PRs are a known coordination risk in multi-agent systems. CI-level lock file conflict detection and a coordinator-enforced serialization rule for shared dependencies prevent this class of failure. |
| 32 | **B** | A human approval gate ensures that a rollback triggered by a benign spike is caught before execution. Improved anomaly detection reduces the false positive rate at the source. Both controls together provide defense in depth. |
| 33 | **B** | Detecting a rate-limit error and immediately checkpointing before the job times out preserves completed work. Graceful failure allows the retry job to resume from the checkpoint rather than restarting from task 1. |
| 34 | **B** | HMAC validates integrity at signing time. GitHub's artifact checksum (via the API) validates integrity post-upload. Using both detects tampering that occurs in the window between signing and coordinator download. |
| 35 | **B** | A repository-level retention policy prevents unbounded storage growth. A scheduled cleanup workflow enforces the policy automatically. Documenting the policy in governance files makes it discoverable and auditable. |
| 36 | **A** | "Require review from Code Owners" in branch protection must be enabled and must not be bypassable by the PR author. If this rule had been enforced, the merge could not have completed without the dev team's approval. |
| 37 | **B** | The 24-hour reporting requirement does not depend on the template file existing. The team must file the report using whatever structure captures the required information. Restoring the template is a parallel remediation task, not a prerequisite for filing. |
| 38 | **B** | CODEOWNERS must be comprehensive for security-sensitive paths, and the PR description must include a verifiable scope manifest. These two controls together expose the discrepancy between claimed and actual scope before merge. |
| 39 | **B** | Exit code 0 only confirms the workflow completed without error, not that the output is correct or fresh. Scheduled agent workflows must validate output quality and post failures to a notification channel so silent degradation is detectable. |
| 40 | **B** | A PR description that merely names the agent and references the diff satisfies the letter of "a description exists" but not the spirit of an audit trail. The policy must specify minimum required content to be enforceable. |
| 41 | **B** | The CODEOWNERS file must protect itself. Without a CODEOWNERS entry for `CODEOWNERS` mapped to a protected owner group — and without branch protection blocking unapproved deletion — any contributor with write access can remove all code-owner governance in a single push. |
| 42 | **B** | Revert commits can have unintended effects and should remain within governance scope limits. A human authorization step ensures that the revert is intentional, appropriately scoped, and part of a documented incident response rather than an autonomous action. |
