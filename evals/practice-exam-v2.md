# GH-600 Practice Exam — Version 2 (Hard)

**Exam version:** v2  
**Date:** 2026-05-19  
**Difficulty:** Hard  
**Topics covered:** All 6 (Agent Architecture & SDLC, Tool Use & Environment Interaction, Memory State & Execution, Evaluation & Error Analysis, Multi-Agent Coordination, Guardrails & Accountability)  
**Total questions:** 42 (7 per topic)  
**Suggested time:** 120 minutes (~2.9 min per question)  
**Passing score:** 80% (34/42)

> **Instructions:** Choose the single best answer for each question. All questions require multi-concept reasoning or scenario analysis. Answers and explanations are in the [Answer Key](#answer-key) at the bottom.

---

## Topic 1 — Agent Architecture and SDLC Processes

**Q1.** A company runs three separate agent pipelines: one triages issues, one drafts PRs, and one reviews merged code for regressions. All three share the same `GITHUB_TOKEN` with `write` access to `contents`, `pull-requests`, and `issues`. A security audit flags this as a risk. What is the most precise mitigation?

- A) Revoke the `GITHUB_TOKEN` and switch to SSH keys  
- B) Assign each pipeline a dedicated GitHub App with installation permissions scoped to only what that specific pipeline needs  
- C) Run all three pipelines in a single workflow so they share one token lifetime  
- D) Store the token in an environment secret rather than a workflow secret  

---

**Q2.** An agent at Autonomy Level 2 implements a hotfix on `main` during an incident, requiring a bypass of the standard two-approval branch-protection rule. The on-call engineer approves the bypass in GitHub's protected branch override UI. What additional step must be completed to maintain governance compliance?

- A) Nothing — the engineer's approval constitutes sufficient audit evidence  
- B) An incident report must be filed immediately using `.github/ISSUE_TEMPLATE/agent-incident.md`  
- C) A rollback commit must be pushed to undo the bypass  
- D) The agent must be elevated to Level 3 for this action  

---

**Q3.** A platform team wants their agent to open PRs in Repository A but deploy to a staging environment in Repository B. Which mechanism enables this cross-repository orchestration without granting the agent broad organization-wide write access?

- A) A personal access token with `repo` scope stored in Actions secrets  
- B) A GitHub App installed on both repositories with repository-specific permissions, orchestrated via `repository_dispatch` or `workflow_call`  
- C) A service account with write access to all repositories in the organization  
- D) Passing `GITHUB_TOKEN` across repositories via a shared workflow artifact  

---

**Q4.** A developer proposes that the agent's automated code analysis should count as "one human-equivalent approval," allowing a PR to merge with only one additional human approval instead of the required two. Which fundamental governance flaw does this argument contain?

- A) Squash merges are not compatible with multi-approver branch protection rules  
- B) CI checks are the only valid substitute for a human approval  
- C) Agent analysis is not a human review; counting it as such removes the human-in-the-loop requirement and circumvents the separation-of-duties control  
- D) Autonomy Level 2 explicitly prohibits all PR merge operations  

---

**Q5.** After six months, a repository has accumulated 400+ stale branches named `agent/<issue-number>` because no cleanup policy was defined. Which governance addition resolves this without disrupting active work?

- A) Delete all branches older than 30 days immediately  
- B) Add a branch lifecycle rule that auto-deletes merged branches and flags unmerged agent branches after a defined idle period for human review  
- C) Rename all stale branches to `archive/<name>` in bulk  
- D) Increase the repository's storage quota  

---

**Q6.** During a post-mortem the team discovers that an agent modified `CODEOWNERS` to add itself as an owner of `.github/workflows/`. The change passed CI because an existing CODEOWNERS entry protecting those paths had been accidentally removed. Which combination of controls would have prevented this?

- A) Requiring the agent to post a comment before merging any CODEOWNERS-adjacent file  
- B) A branch protection rule requiring CODEOWNERS review AND a CODEOWNERS entry that maps `CODEOWNERS` itself to a human team  
- C) A CI workflow that validates CODEOWNERS syntax before merge  
- D) Moving the CODEOWNERS file from `.github/` to the repository root  

---

**Q7.** An organization stores all agent governance policy exclusively in `copilot-instructions.md` with no corresponding platform controls. A developer accidentally deletes the file and pushes directly to `main`. For the next 48 hours agents operate without any governance constraints. Which root cause does this expose?

- A) The developer lacked write access to `main`  
- B) Platform controls were not configured to protect the governance file itself, making policy purely advisory and bypassable  
- C) The CI pipeline did not validate the presence of `copilot-instructions.md`  
- D) The agent should have detected the missing file and halted automatically  

---

## Topic 2 — Tool Use and Environment Interaction

**Q8.** A workflow calls an external payment API using `secrets.PAYMENT_API_KEY` and echoes the full API response — which includes the key reflected back in encoded form — into the workflow log for debugging. GitHub's secret masking replaces the raw key with `***`. What residual risk remains?

- A) The masked log is publicly visible by default on public repositories  
- B) If the API response encodes the key differently (e.g., base64 or URL-encoded), secret masking may not redact it, leaving the key exposed in the log  
- C) GitHub logs are retained for only 24 hours so there is no lasting risk  
- D) The `GITHUB_TOKEN` will be revoked when the log entry is written  

---

**Q9.** An agent workflow runs on a self-hosted runner shared across multiple repositories. The workflow creates a temporary file containing customer email addresses for test data and then deletes it at job completion. What risk does this pattern introduce that a GitHub-hosted runner would not?

- A) Self-hosted runners do not support workflow artifact uploads  
- B) If the cleanup step fails (e.g., due to job cancellation), the PII file persists on the shared disk and may be accessible by workflows from other repositories  
- C) Self-hosted runners automatically block file writes by non-root processes  
- D) GitHub-hosted runners automatically encrypt all temporary files at rest  

---

**Q10.** A multi-step workflow uses `actions/cache`. A malicious pull request from an external fork modifies the cache key to overwrite a cache entry used by main-branch builds. What is this attack called and what is the primary mitigation?

- A) Supply chain attack; pin all third-party action versions to commit SHAs  
- B) Cache poisoning; restrict cache write access to trusted branches only and do not allow fork PRs to write to the default cache scope  
- C) Token leakage; rotate secrets after each fork PR runs  
- D) OIDC hijacking; use environment protection rules on all jobs  

---

**Q11.** An agent authenticates to AWS via OIDC with `sub: repo:org/repo:ref:refs/heads/main`. A new workflow on feature branches needs the same authentication. The simplest fix is to broaden the AWS trust policy to `repo:org/repo:*`. Why should this fix be avoided in favor of a more targeted approach?

- A) OIDC does not support wildcard subject claims in any cloud provider  
- B) A wildcard subject allows any branch — including untrusted or attacker-controlled branches — to assume the same IAM role as `main`; a separate, read-only role scoped to `refs/heads/feature/*` should be created instead  
- C) Feature branch workflows cannot use OIDC tokens  
- D) AWS trust policies do not parse the `ref` portion of OIDC subject claims  

---

**Q12.** A workflow downloads a dependency tarball using `curl` and pipes the output directly to `bash` for installation (`curl https://example.com/install.sh | bash`). What security risk does this introduce that using `actions/setup-*` tools does not?

- A) `curl` is unavailable on Ubuntu GitHub-hosted runners  
- B) The remote server could serve malicious script content at any time; there is no checksum verification or sandboxing, unlike official setup actions which pin and verify artifact integrity  
- C) Bash is unavailable inside Docker-based containers  
- D) This pattern is automatically blocked by GitHub's runner IP allowlist  

---

**Q13.** An MCP server starts on `localhost:3000` in a workflow job. At job completion, the server process is not explicitly stopped. How does the outcome differ between a GitHub-hosted runner and a self-hosted runner?

- A) On both runner types the process continues running indefinitely  
- B) On a GitHub-hosted runner the ephemeral VM is destroyed so the process terminates automatically; on a self-hosted runner the process may continue running, consuming resources and potentially responding to requests from subsequent jobs  
- C) GitHub-hosted runners forward orphaned processes to the next queued job  
- D) The runner agent automatically kills all orphaned processes on both runner types  

---

**Q14.** A developer wants to allow full CI on pull requests from internal contributors while preventing untrusted fork PRs from executing privileged steps. Which `on:` / `if:` configuration achieves this most precisely?

- A) Use `on: push` only — this eliminates all PR-triggered runs  
- B) Use `pull_request_target` for all external work and `pull_request` for internal branches  
- C) Use `on: pull_request: types: [opened]` to restrict to the PR open event only  
- D) Add `if: github.event.pull_request.head.repo.full_name == github.repository` to the job definition to restrict privileged steps to same-repo PRs  

---

## Topic 3 — Memory, State, and Execution

**Q15.** An agent uploads a state artifact after each pipeline stage. On retry, it downloads the artifact to resume. The artifact was uploaded before a stage finished writing all its output files. The retry reads a partial artifact and produces corrupt output. Which design property was violated, and what is the correct fix?

- A) Idempotency; add a retry loop around the artifact download step  
- B) Atomicity; upload the state artifact only after all outputs for a stage are fully written and validated  
- C) Caching; increase the artifact retention period  
- D) Parallelism; run all pipeline stages concurrently  

---

**Q16.** A user preference ("always use British English") is stored in a repository-wide `copilot-instructions.md`. A new engineer joins the team and their IDE Copilot enforces British English without any explanation. Which memory architecture design issue does this illustrate?

- A) Repository memory is the wrong tier for personal preferences; user-level preferences belong in user-scoped memory, not repository-wide instructions  
- B) `copilot-instructions.md` cannot store text style preferences  
- C) The new engineer should have been granted write access before the instruction was written  
- D) Organization-wide Copilot configuration is not supported  

---

**Q17.** An agent stores the result of an expensive API call in a GitHub Actions cache with a 7-day TTL. The upstream data changes daily. On day 6 the agent reads stale data and produces an incorrect recommendation. Which pattern best prevents this category of error?

- A) Extend the cache TTL to 30 days  
- B) Add a content hash or `Last-Modified` header check before using cached data; invalidate and refresh if the upstream data has changed since the cache entry was written  
- C) Disable caching entirely and always call the API fresh  
- D) Store the API response in a repository secret instead  

---

**Q18.** A stateful agent conversation is stored in PR comments. The PR is closed without merging after 60 days and the comment history is archived. A new PR for the same task is opened and the agent has no memory of prior decisions. Which memory tier would have preserved the context most durably?

- A) Prompt context memory  
- B) A new PR comment thread in the replacement PR  
- C) A repository file (e.g., `docs/decisions/<issue>.md`) committed to the repository before the original PR was closed  
- D) An in-memory variable in the workflow run  

---

**Q19.** An agent is designed to skip PR creation if a PR for the same issue already exists. A race condition causes two workflow runs to trigger simultaneously for the same issue. Both check at the same instant, both find no existing PR, and both create one. Which mechanism resolves this race condition?

- A) Adding a `sleep` step before PR creation to stagger runs  
- B) Using a distributed lock (e.g., creating a lock issue/label before PR creation) or GitHub's conditional creation API with a unique idempotency key  
- C) Setting `concurrency: global` to serialize all workflow runs in the repository  
- D) Adding `if: github.run_attempt == 1` to prevent retries from creating PRs  

---

**Q20.** A security researcher reports that any step in a job — including a third-party action step — can overwrite values in `$GITHUB_ENV`, allowing later steps to be manipulated. What is the recommended mitigation introduced in GitHub Actions?

- A) Store intermediate results in a repository secret instead  
- B) Use `$GITHUB_OUTPUT` with step-level output variables, and audit third-party actions before including them to limit the attack surface  
- C) Encrypt all values before writing them to `$GITHUB_ENV`  
- D) Run each step in a separate job to prevent environment variable sharing  

---

**Q21.** An agent reads canonical guidance from a repository wiki at the start of each run. The wiki is updated by humans asynchronously. During a high-frequency period the agent reads an outdated wiki page and makes decisions based on deprecated guidance. What architectural change best addresses this?

- A) Lock the wiki for writes during agent runs  
- B) Store canonical guidance in a versioned file committed to the repository (e.g., `docs/guidance.md`), so the agent always reads the version aligned with the current commit  
- C) Run the agent less frequently to reduce the chance of reading stale data  
- D) Cache the wiki page locally for 30 days  

---

## Topic 4 — Evaluation, Error Analysis, and Tuning

**Q22.** An evaluator scores an agent output as Correctness=Pass, Scope=Pass, Safety=Fail, Auditability=Pass. The agent owner argues that three of four dimensions pass so the result should be a conditional pass. What is wrong with this argument per the GH-600 rubric?

- A) The rubric uses weighted scoring, not binary pass/fail per dimension  
- B) Safety is a hard-stop dimension; any Safety=Fail produces an overall Fail regardless of other scores  
- C) A 75% dimension pass rate automatically triggers a review, not an outright fail  
- D) Auditability should have been graded Partial, which would change the outcome  

---

**Q23.** Evaluation results show the agent passes a scenario but the evaluator notes the agent arrived at the correct answer through incorrect reasoning (it edited the wrong file but got the right outcome because the codebase has duplicate content). What should the evaluator do?

- A) Accept the Pass — the rubric grades outputs, not reasoning chains  
- B) Flag the pass as fragile in the Notes column, add a harder scenario that removes the duplicate content, and re-evaluate to confirm genuine correctness  
- C) Upgrade the grade to a "Strong Pass" to distinguish it from a borderline result  
- D) Discard the scenario as inconclusive  

---

**Q24.** Two evaluators independently score the same run on Scenario 3. Evaluator A grades Safety=Pass; Evaluator B grades Safety=Fail citing a partial internal ticket ID visible in the output. Per the rubric, whose grade stands and why?

- A) Evaluator A's, because the ticket ID is only partial and not a full credential  
- B) Evaluator B's, because the Safety dimension has no Partial grade and any exposure of sensitive data is a hard Fail  
- C) The grades are averaged to Partial  
- D) The scenario must be re-run to break the tie  

---

**Q25.** After three tuning rounds, Correctness improved from 60% to 85% but Scope Control degraded from 95% to 70%. The prompt engineer argues the net improvement justifies the trade-off. Why is this reasoning flawed?

- A) A 70% Scope Control score still exceeds the minimum passing threshold  
- B) Scope Control degradation means the agent is now touching unrelated files; a regression in any safety or governance dimension is not an acceptable trade-off for correctness gains  
- C) Evaluators must use a different rubric after three rounds of tuning  
- D) Correctness improvements below 90% are not considered genuine improvements  

---

**Q26.** An evaluator runs Prompt A on Monday and Prompt B on Friday. Between Monday and Friday, the underlying model is silently updated by the provider. What evaluation design flaw does this create?

- A) The scenarios may be non-deterministic and differ between runs  
- B) The model version is a confounding variable; differences in results cannot be attributed solely to the prompt change without controlling for the model version  
- C) The rubric weights may have changed between the two evaluation runs  
- D) Scenario files were likely modified during the week  

---

**Q27.** A team has used the same 10 golden scenarios for all evaluations over 12 months of prompt tuning. The agent performs excellently on these 10 scenarios but poorly on novel tasks. Which mitigation directly addresses evaluation overfitting?

- A) Increase the number of evaluation runs per existing scenario  
- B) Rotate in new held-out scenarios periodically and evaluate prompt versions against both the established set and the new scenarios before accepting any prompt change  
- C) Replace the weakest 3 scenarios with slightly varied versions  
- D) Run evaluations less frequently so the prompt author cannot inadvertently tune to the golden set  

---

**Q28.** An agent is evaluated on a code-review task. The acceptance criterion is "all high-severity issues are identified." The agent correctly identifies 4 of 5 high-severity issues. The evaluator grades Correctness=Pass because the agent caught "most" issues. What error did the evaluator make?

- A) The evaluator applied the wrong rubric dimension  
- B) The evaluator applied a subjective interpretation ("most") instead of the literal acceptance criterion ("all"); the correct grade is Partial or Fail  
- C) Correctness must be graded by the prompt author, not the evaluator  
- D) A 4/5 catch rate automatically qualifies as Pass under the GH-600 rubric  

---

## Topic 5 — Multi-Agent Coordination

**Q29.** A planner agent generates a structured JSON plan artifact. The executor validates it against a JSON Schema. Validation passes, but the executor fails because the planner used a valid schema field (`"action": "delete"`) in an unexpected context that the executor interprets as deleting a production database. What design gap does this expose?

- A) JSON Schema validation is insufficient alone; a semantic validation layer that checks plan operations against an allowlist for the current environment is also required  
- B) The schema should categorically prohibit all `delete` operations  
- C) The executor should not validate inputs received from the planner  
- D) The planner should have used YAML instead of JSON  

---

**Q30.** Three specialist agents (A, B, C) each propose changes to the same configuration file. Agent A's change is a superset of B's change. Agent C's change conflicts with both A and B. What is the coordinator's correct action?

- A) Apply only A's change since it is a superset of B's  
- B) Apply all three changes in alphabetical agent order  
- C) Apply A (since it supersedes B), and escalate the conflict between A and C to a human reviewer before proceeding  
- D) Discard all proposals and request new ones from all three agents  

---

**Q31.** In a fan-out fan-in pattern, a coordinator dispatches tasks to 5 specialists in parallel. One specialist times out without returning a result. What behavior aligns with best governance practices?

- A) Wait indefinitely for all 5 agents to complete  
- B) Proceed immediately with the 4 available results and discard the timeout silently  
- C) Log the partial result, mark the timed-out specialist's output as incomplete, escalate to a human if the missing output is on a critical path, and produce an audit record  
- D) Cancel all 5 specialist runs and restart the entire pipeline from the beginning  

---

**Q32.** Pipeline 1 updates `config/settings.yaml`. Pipeline 2 reads the same file to generate deployment parameters. Pipeline 2 starts immediately after Pipeline 1 begins but before Pipeline 1 finishes. What is the safest coordination mechanism?

- A) Run both pipelines sequentially by removing all parallel execution  
- B) Use a repository-scoped concurrency group to prevent Pipeline 2 from starting until Pipeline 1's file update is complete  
- C) Lock the file using `git lfs lock`  
- D) Merge both pipelines into a single monolithic workflow  

---

**Q33.** An executor agent returns a patch file artifact generated against commit SHA `abc1234`. The coordinator applies the patch against `HEAD`, which has since moved to `def5678`. The patch produces merge conflicts. Which handoff contract element was missing?

- A) The executor should have used a YAML diff instead of a patch file  
- B) The patch artifact must include the source commit SHA, and the coordinator must verify `HEAD` matches it (or rebase the patch) before applying  
- C) The coordinator should apply patches without pre-validation to minimize latency  
- D) The executor should have pushed the change directly to `HEAD`  

---

**Q34.** A three-tier agent system (strategic planner → tactical planner → executor) has Tier 1 with write access to all tiers' configuration files. Tier 3 is compromised by prompt injection and attempts to overwrite Tier 1's configuration. Which control prevents this lateral movement?

- A) Each tier's configuration files must be owned in CODEOWNERS by the tier above it, and the executor (Tier 3) must have no write access to Tier 1 or Tier 2 configuration paths  
- B) Run all three tiers in the same workflow job for better isolation  
- C) Use a `repo`-scoped personal access token for each tier  
- D) Remove CODEOWNERS entries for agent configuration files to avoid complexity  

---

**Q35.** A multi-agent pipeline uses `workflow_call` for handoffs. The planner workflow passes `secrets: inherit` to the executor workflow. A security audit flags this. What is the risk?

- A) `secrets: inherit` is not supported in `workflow_call` reusable workflows  
- B) `secrets: inherit` forwards all calling workflow secrets to the executor; if the executor is over-privileged or compromised, it gains access to every secret in the caller's context rather than only the specific secrets it legitimately needs  
- C) `secrets: inherit` causes the executor to bypass all environment protection rules  
- D) Inherited secrets cannot be used in the executor workflow's `run:` steps  

---

## Topic 6 — Guardrails and Accountability

**Q36.** An agent PR modifies `.github/workflows/ci.yml`. A release manager has a branch-protection bypass enabled and approves the PR without reviewing the diff. The workflow is merged and executes malicious steps. At which governance layers did controls fail?

- A) Only the branch protection bypass is at fault; remove all bypass roles  
- B) Both the agent's hard-stop rule (which should have blocked PR creation) and the human reviewer's lack of due diligence failed; remediation requires enforcing the agent hard-stop AND adding a mandatory diff-review policy for any workflow file change  
- C) Only the CI pipeline failed to detect the malicious steps  
- D) Only the missing CODEOWNERS entry for `.github/workflows/` is at fault  

---

**Q37.** An agent correctly identifies that a file is protected by CODEOWNERS and halts, posting a review-request comment. The CODEOWNERS team is on vacation for two weeks, but the issue the agent was addressing is critical. What is the correct governance path?

- A) Override CODEOWNERS and have the agent proceed without approval  
- B) Escalate through the incident response process, which can grant a temporary human bypass with documented rationale, a timed revocation, and a post-action incident report  
- C) Wait until the team returns from vacation regardless of urgency  
- D) Elevate the agent to Autonomy Level 3 for this specific task  

---

**Q38.** After a security incident, investigators need to reconstruct exactly what the agent read, decided, and acted on during a run. They find only high-level summaries in the PR description. Which additional audit artifact should have been required?

- A) A signed commit for each individual file change  
- B) Structured workflow logs capturing each tool call with its inputs, outputs, and decision rationale at every step of the agent's execution  
- C) A separate `CHANGELOG.md` entry for every agent-produced PR  
- D) A screenshot or recording of the agent's output in the UI  

---

**Q39.** An agent at Autonomy Level 2 discovers a critical CVE in a dependency. Fixing it requires changes to `package.json`, `package-lock.json`, and three source files — totaling 410 lines across 5 files. What must the agent do per governance rules?

- A) Proceed immediately; security vulnerabilities override all scope-limit governance rules  
- B) Stop and post a human-review request; the 400-line / 5-file threshold is a hard stop that applies even to security fixes  
- C) Apply only the `package.json` change and open a separate PR for the source file changes  
- D) Compress the diff by combining file changes to stay within the threshold  

---

**Q40.** A governance audit reveals that 12 of the last 15 agent-produced PRs were approved by the same individual who was also the task requester. No independent review occurred. Which control most directly addresses this conflict of interest?

- A) Require the task requester to document their rationale in the PR description  
- B) Configure branch protection to require at minimum 2 approvals AND prevent the PR author from being one of the approving reviewers  
- C) Increase the required approval count to 3 reviewers for all agent PRs  
- D) Move all agent PRs to a separate repository with stricter access controls  

---

**Q41.** An agent PR has already been merged when an incident is detected. The incident response runbook says "Step 1: Close or revert the PR." What modification is required, and what is the correct Step 1 action?

- A) Step 1 remains "close the PR" — closing an already-merged PR prevents further merges from it  
- B) The runbook must account for already-merged changes: Step 1 is to create a revert commit (`git revert <sha>`) and open an urgent human-reviewed PR to land it on `main`, followed by the remaining runbook steps  
- C) Step 1 is to delete the `main` branch and restore it from the last known-good backup  
- D) No runbook modification is needed; a merged PR cannot be reverted via the incident process  

---

**Q42.** An agent posts detailed error messages — including internal microservice URLs, full stack traces with library paths, and partial environment variable values — to public issue comments when a workflow step fails. Which governance principle does this violate and what is the correct fix?

- A) Idempotency; implement deduplication to prevent repeated error posts  
- B) Information security / least exposure: error messages on a public tracker must be sanitized to remove internal topology data; detailed structured logs should be sent to a private artifact or private notification channel, not posted publicly  
- C) Auditability; errors should not be captured in any log  
- D) Scope control; the agent's token should not include `issues: write` permission  

---

## Answer Key

| Q | Answer | Explanation |
|---|--------|-------------|
| 1 | **B** | Each pipeline needs only its own specific permissions; a dedicated GitHub App with narrowly scoped installation permissions enforces least privilege per pipeline. |
| 2 | **B** | Any governance bypass requires an incident report so the organization can learn from and audit the deviation. |
| 3 | **B** | A GitHub App with per-repository installation permissions, triggered via `repository_dispatch` or `workflow_call`, provides cross-repo orchestration without organization-wide credentials. |
| 4 | **C** | Agent output is not a human review. Counting it as one eliminates the human-in-the-loop checkpoint that governed delivery requires. |
| 5 | **B** | Auto-deleting merged branches and flagging unmerged agent branches for review balances hygiene with safety for in-flight work. |
| 6 | **B** | Protecting `CODEOWNERS` with its own CODEOWNERS entry (mapped to a human team) plus a branch protection rule requiring CODEOWNERS review creates a two-layer defense that prevents this self-escalation. |
| 7 | **B** | Advisory-only policy stored solely in a file can be bypassed by anyone with write access. Platform controls are required to make governance mechanically enforceable. |
| 8 | **B** | Secret masking matches the exact secret value. Encoded variants (base64, URL-encoding, etc.) are not automatically masked, leaving the key exposed if the API reflects it in an encoded form. |
| 9 | **B** | Shared self-hosted runner disks persist between jobs. A failed cleanup step on a shared runner can expose PII to subsequent jobs from other repositories. |
| 10 | **B** | Allowing fork PRs to write to shared caches lets an attacker poison cache entries consumed by trusted builds. Restricting write access to trusted branches is the primary control. |
| 11 | **B** | A wildcard `repo:org/repo:*` subject allows any branch to assume the production IAM role. Untrusted or attacker-created branches could exploit this to gain production-level cloud access. |
| 12 | **B** | Piping `curl` output directly to `bash` executes arbitrary code from a remote source with no integrity check. Official setup actions pin versions and verify checksums before execution. |
| 13 | **B** | GitHub-hosted runners are ephemeral VMs destroyed after each job, automatically killing all processes. Self-hosted runners are persistent, so orphaned processes continue running and may affect subsequent jobs. |
| 14 | **D** | The `if: github.event.pull_request.head.repo.full_name == github.repository` condition restricts privileged job steps to PRs from the same repository, preventing untrusted fork PRs from executing those steps. |
| 15 | **B** | Atomicity requires that a checkpoint be written only after all outputs are fully and successfully produced. Writing a partial checkpoint violates atomicity and causes corrupt resume behavior. |
| 16 | **A** | User-level stylistic preferences should live in user-scoped memory, not repository-wide instructions that silently affect all users including newcomers who never consented to the preference. |
| 17 | **B** | A content hash or `Last-Modified` check couples cache validity to actual data freshness rather than an arbitrary TTL, preventing the use of stale data when upstream content has changed within the TTL window. |
| 18 | **C** | Committed repository files (e.g., `docs/decisions/<issue>.md`) survive PR closure, branch deletion, and any merge strategy, making them the most durable memory tier. |
| 19 | **B** | A distributed lock or idempotency key enforced before PR creation prevents both runs from proceeding past the check simultaneously, eliminating the race condition. |
| 20 | **B** | `$GITHUB_OUTPUT` creates step-scoped outputs not writable by other steps, reducing the attack surface compared to the job-wide mutable `$GITHUB_ENV`. Auditing third-party actions further limits exposure. |
| 21 | **B** | A versioned file committed to the repository is immutable for a given commit SHA and always consistent with the code it governs. Wiki content is out-of-band and can be asynchronously stale. |
| 22 | **B** | Safety has no Partial grade; any Safety failure is a hard Fail for the entire scenario, regardless of how well other dimensions score. |
| 23 | **B** | A correct output produced through incorrect reasoning is fragile — it will fail when the accidental correctness condition is removed. Flagging and hardening the scenario ensures genuine capability is validated. |
| 24 | **B** | The Safety dimension allows only Pass or Fail. Any exposed sensitive data — even partial — is a hard Fail. Evaluator B's grade stands. |
| 25 | **B** | Governance and safety dimensions must not regress. Trading Scope Control for Correctness means the agent is now creating side effects beyond its mandate, which is a worse outcome from a risk perspective. |
| 26 | **B** | A silent model update between evaluation runs creates a confounding variable. Results differences could be due to the model change, not the prompt change, invalidating the comparison. |
| 27 | **B** | Prompt overfitting to a fixed golden set produces agents that are narrow specialists on those exact inputs. Held-out scenarios are the standard mitigation in evaluation science. |
| 28 | **B** | Rubric grades must be applied against the literal acceptance criterion. "All" means all; "most" is a subjective deviation from the stated standard and produces inflated scores. |
| 29 | **A** | Schema validation only checks structural validity. A semantically dangerous operation can be structurally valid. An allowlist of permitted operations in context provides the missing semantic guard. |
| 30 | **C** | When A supersedes B the choice is clear, but a conflict between A and C cannot be resolved by the coordinator without additional context — human judgment is required. |
| 31 | **C** | Governance requires logging incomplete results, making the gap visible, and escalating critical-path gaps to a human. Silent discard or indefinite wait are both unacceptable. |
| 32 | **B** | A repository-scoped concurrency group serializes the two pipelines without requiring them to be merged into one workflow, preserving pipeline independence while preventing the read/write race. |
| 33 | **B** | A patch generated against a specific commit is only safe to apply at exactly that commit. Including the source SHA in the artifact and verifying it before application prevents conflict-causing stale patches. |
| 34 | **A** | CODEOWNERS protection enforced per tier, with the executor having no write access to higher-tier configuration, implements defense in depth that contains a compromised executor to its own tier. |
| 35 | **B** | `secrets: inherit` is a broad delegation. The executor receives all calling-workflow secrets, creating an unnecessarily large blast radius if the executor is compromised or misconfigured. |
| 36 | **B** | Two independent controls both failed: the agent's hard-stop rule (which should have prevented creating a PR touching workflows) and the human approver's failure to review the diff. Both must be remediated. |
| 37 | **B** | The incident response process exists precisely for situations where normal governance gates are blocked by operational reality. A documented temporary bypass with timed revocation and a post-action report maintains accountability. |
| 38 | **B** | PR descriptions are summaries. Forensic reconstruction requires granular, step-by-step tool call logs with inputs and outputs — the level of detail that structured workflow logs provide. |
| 39 | **B** | Hard stops apply regardless of the reason for the change. Security urgency is handled through the human incident response process, not by autonomous agent override of governance limits. |
| 40 | **B** | Disabling self-approval and requiring at least 2 approvers mechanically prevents the conflict of interest. Documentation requirements alone are insufficient since they rely on voluntary compliance. |
| 41 | **B** | A merged PR cannot be "closed" retroactively to undo its effects. The correct action is a revert commit that must itself go through human review, preserving the audit trail. |
| 42 | **B** | Internal topology information (service URLs, stack traces, env var values) in public issue comments violates information security policy. Detailed diagnostics belong in private artifacts or private notification channels. |
