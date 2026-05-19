# Exam Category — Index & Taxonomy

This file defines the **exam** category within the `evals/` evaluation taxonomy and lists all exam-form files in this repository.

---

## Purpose

An **exam form** is a structured, fillable QA sheet used to score a single agent evaluation run against a defined set of scenarios. It produces a numeric pass rate and a clear Pass/Fail gate verdict.

Exam forms are the primary deliverable of each evaluation cycle. They are version-controlled in this directory so that evaluators can compare results across prompt versions and agent configurations.

---

## Who Fills It Out

| Role | Responsibility |
| --- | --- |
| Evaluator | Runs the agent against each scenario, grades each dimension, and commits the filled-in form |
| Reviewer | Validates grades against evidence in the Notes column and approves the PR |
| Prompt author | Reads Section E to identify root causes and propose the next tuning action |

---

## When to Create a New Exam Form

Create a new versioned copy of `exam-form.md` (e.g. `exam-form-v2.md`) whenever:

- A new prompt version is being evaluated.
- The scenario set changes.
- The rubric weights or gate threshold are updated.

Do **not** edit a committed exam form — create a new version instead to preserve the audit trail.

---

## Difference Between a Rubric and an Exam Form

| Document | Purpose |
| --- | --- |
| `rubric.md` / `agent-template-rubric.md` | Defines how to grade each dimension (the answer key) |
| `exam-form.md` | The fillable sheet an evaluator completes for one run (the answer sheet) |
| `results-template.md` | Captures raw agent output and run metadata alongside the exam form findings |

---

## Exam Form Files

| File | Prompt version | Date | Status |
| --- | --- | --- | --- |
| [exam-form.md](exam-form.md) | — | — | Template (do not fill in directly) |
| [practice-exam-v1.md](practice-exam-v1.md) | v1 | 2026-05-19 | Published — 30 questions, Medium difficulty, all 6 topics, ~60 min |
| [practice-exam-v2.md](practice-exam-v2.md) | v2 | 2026-05-19 | Published — 42 questions, Hard difficulty, all 6 topics, ~120 min |
| [practice-exam-v3.md](practice-exam-v3.md) | v3 | 2026-05-19 | Published — 42 questions, Hard difficulty, all 6 topics, ~120 min (alternate question set) |

> Add a row here each time you commit a completed `exam-form-v<N>.md` or `practice-exam-v<N>.md`.

---

## Related Files

- [rubric.md](rubric.md) — general evaluation rubric
- [agent-template-rubric.md](agent-template-rubric.md) — rubric for the agentic AI template
- [scenarios.md](scenarios.md) — golden scenario set
- [agent-template-scenarios.md](agent-template-scenarios.md) — agent template scenario set
- [results-template.md](results-template.md) — raw-output results template
