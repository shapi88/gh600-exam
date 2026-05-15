# Topic 4: Perform Evaluation, Error Analysis, and Tuning

## Key Concepts and Sub-Topics

- **Golden scenarios and pass/fail rubrics** — a fixed set of representative tasks with defined expected outcomes used to measure agent quality consistently
- **Task decomposition and measurable outcomes** — break complex goals into sub-tasks, each with a verifiable success criterion
- **Error categories:**
  - **Hallucinated tools** — agent references or calls a tool that does not exist
  - **Stale context** — agent acts on outdated branch, issue, or file state
  - **Excessive changes** — agent modifies files outside the intended scope
  - **Policy violations** — agent bypasses a required check, permission, or approval gate
  - **Unsafe output** — agent produces content that could cause harm, expose secrets, or violate compliance
- **Prompt iteration and tool-choice tuning** — changing one variable at a time and measuring impact against the same rubric
- **Regression testing for agent behavior** — ensuring that prompt improvements do not re-introduce previously fixed failure modes

---

## Common Pitfalls / Anti-Patterns

- Measuring success only by "the answer sounds good" rather than by a repeatable rubric.
- Evaluating prompts without representative repo context (toy examples do not reflect production failure modes).
- Tuning only for success rate while ignoring safety and change scope.
- Fixing failures by granting broader permissions instead of improving instructions or narrowing tools.

---

## Step-by-Step Tutorial

**Goal:** Build a simple evaluation loop for one agent-assisted workflow.

1. **Pick 5–10 representative tasks** from real issues or a mock backlog. Tasks should span routine work, edge cases, and ambiguous requests.

2. **Define a scoring rubric** for each task:

   | Dimension | Pass criteria | Fail criteria |
   | --- | --- | --- |
   | Correctness | Output matches the acceptance criteria | Key requirement not met |
   | Scope control | Only required files changed | Unrelated files modified |
   | Safety | No secrets exposed; no policy bypass | Any sensitive data in output |
   | Auditability | Rationale in PR / logs | No explanation of decisions |

3. **Run the same prompt or instructions across the full scenario set.** Record raw outputs and apply the rubric.

4. **Record failure types** for each scenario that did not pass:
   - Wrong tool selected
   - Extra files changed
   - Unsafe suggestion
   - Missing required check

5. **Tune one variable at a time** — reword an instruction, restrict a tool, or add an example — then re-run the full scenario set and compare.

---

## Sample Evaluation Rubric

| Scenario | Correctness | Scope control | Safety | Auditability | Overall |
| --- | --- | --- | --- | --- | --- |
| Fix typo in README | ✅ | ✅ | ✅ | ✅ | Pass |
| Refactor auth module | ✅ | ❌ (touched unrelated file) | ✅ | ✅ | Fail |
| Generate release notes | ✅ | ✅ | ❌ (included internal ticket ID) | ✅ | Fail |
| Triage 10 issues | ✅ | ✅ | ✅ | ❌ (no log of labeling decisions) | Fail |

---

## Practical Exercises

- Create a failure taxonomy for your study repo and map one specific mitigation to each category (wrong tool, stale context, excessive changes, policy violation, unsafe output).
- Propose two evaluation metrics beyond "task completed" — for example, touched-file count and time-to-first-unsafe-suggestion.
- Given an agent that edits unrelated files, define the fastest safe tuning approach without expanding permissions.
- Write a rubric for evaluating a code-review agent that comments on PRs.

**What a strong answer includes:**
- Representative scenarios from real repo context
- Measurable, multi-dimensional rubric
- Repeatable comparison across prompt versions
- Safety regressions treated as failures, not acceptable tradeoffs

---

## Exam-Style Questions

**Q1.** An agent frequently edits unrelated files.  
**A:** Add an evaluation metric for touched-file scope and tighten the instructions. You must measure and correct scope creep explicitly.

**Q2.** A prompt performs well on toy examples but fails in the real repo.  
**A:** Build evals with representative repository context and realistic tasks. Exam questions favor production realism over synthetic success.

**Q3.** An agent picks the wrong tool even though the final answer looks plausible.  
**A:** Evaluate tool-choice correctness separately from output quality. Tool misuse can create silent safety or reliability failures.

**Q4.** The fix rate improves after adding broad write permissions.  
**A:** Reject that as the primary tuning approach unless clearly justified; improve instructions and task boundaries first. More permissions are not a quality strategy.

**Q5.** You need to compare two prompt versions.  
**A:** Run both against the same scenario set and the same rubric. Controlled comparison is required for meaningful tuning.
