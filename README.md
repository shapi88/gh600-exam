# GH-600 Exam Study Package

Production-oriented, safety-first prep for **GitHub Certified: Agentic AI Developer (GH-600)** with GitHub as the control plane for agentic SDLC workflows.

> [!TIP]
> If you are time-constrained, start with the **5-week study plan** below, work through each topic file in `topics/`, then review the **cheat sheets** and **practice questions** for daily drills.

## Hands-On Labs

> [!NOTE]
> **Complete [Lab 00 — Prerequisites](tutorials/00-prerequisites.md) before starting any lab.** It covers installing the GitHub CLI, creating your practice repository, and setting up Node.js and Python.

Each lab is a standalone, step-by-step guide you can follow in a fresh GitHub repository with zero prior configuration. **Run Lab 01 before reading Topic 1** — the lab builds the exact setup described in the topic.

| Lab | File | What you will build |
| --- | --- | --- |
| 00 | [Prerequisites](tutorials/00-prerequisites.md) | GitHub account, GitHub CLI, practice repo, fine-grained PAT |
| 01 | [Agent Architecture & SDLC](tutorials/01-agent-architecture-lab.md) | Protected `main`, CODEOWNERS, governed `issue → branch → PR` flow |
| 02 | [Tool Use & Environment Interaction](tutorials/02-tool-use-lab.md) | Least-privilege workflow, environment gate, approved MCP config |
| 03 | [Memory, State & Execution](tutorials/03-memory-state-lab.md) | `copilot-instructions.md`, PR checklist checkpoints, artifact upload, idempotent retries |
| 04 | [Evaluation, Error Analysis & Tuning](tutorials/04-evaluation-lab.md) | Golden scenarios, rubric, versioned prompt results, scope-control and secret-scan workflow |
| 05 | [Multi-Agent Coordination](tutorials/05-multi-agent-lab.md) | Planner → executor → reviewer → coordinator pipeline with artifact handoffs |
| 06 | [Guardrails & Accountability](tutorials/06-guardrails-lab.md) | Platform-enforced guardrails, production deployment with approval gate, rollback runbook |

### Evaluation support files

The `evals/` directory contains ready-to-use templates for the evaluation framework covered in Topic 4 and Lab 04:

| File | Purpose |
| --- | --- |
| [evals/scenarios.md](evals/scenarios.md) | 5 golden test scenarios |
| [evals/rubric.md](evals/rubric.md) | 4-dimension scoring rubric |
| [evals/results-v1.md](evals/results-v1.md) | Example results for prompt version 1 (fails scope check) |
| [evals/results-v2.md](evals/results-v2.md) | Example results for prompt version 2 (passes all checks) |

---

## Topics

| # | Topic file | What it covers |
| --- | --- | --- |
| 1 | [Agent Architecture and SDLC Processes](topics/01-agent-architecture-and-sdlc.md) | Agent roles, SDLC placement, human-in-the-loop, GitHub control plane |
| 2 | [Tool Use and Environment Interaction](topics/02-tool-use-and-environment-interaction.md) | MCP, permissions design, runner environments, tool failure handling |
| 3 | [Memory, State, and Execution](topics/03-memory-state-and-execution.md) | Memory types, durable vs ephemeral state, checkpointing, idempotency |
| 4 | [Evaluation, Error Analysis, and Tuning](topics/04-evaluation-error-analysis-and-tuning.md) | Rubrics, golden scenarios, error categories, prompt iteration |
| 5 | [Multi-Agent Coordination](topics/05-multi-agent-coordination.md) | Orchestration patterns, handoff contracts, conflict resolution |
| 6 | [Guardrails and Accountability](topics/06-guardrails-and-accountability.md) | Autonomy levels, approval gates, audit trails, incident response |
| — | [Cheat Sheets](topics/cheat-sheets.md) | All summary tables in one place for rapid review |
| — | [Resources and Final Exam Strategy](topics/resources.md) | Official docs, practice links, exam strategy tips |

Each topic file contains: key concepts, common pitfalls, a step-by-step tutorial, practical exercises, exam-style Q&amp;A, and a **Complete Working Example** section with copy-pasteable commands and full workflow YAML.

## High-Level Study Plan (5 Weeks)

| Week | Focus areas | Time target | Deliverables |
| --- | --- | --- | --- |
| 1 | Agent architecture, SDLC integration, planning vs execution boundaries, human-in-the-loop | 6-8 hrs | One-page architecture notes, autonomy matrix, branch protection + review checklist |
| 2 | Tool use, MCP, permissions, runner environments, workflow design | 8-10 hrs | One repo with an agent-safe workflow, MCP config, least-privilege permissions examples |
| 3 | Memory, state, execution control, retries, resumability, observability | 6-8 hrs | Memory strategy table, state diagram, incident playbook for stuck/failed runs |
| 4 | Evaluation, error analysis, prompt tuning, multi-agent coordination | 8-10 hrs | Eval rubric, failure taxonomy, planner/executor/reviewer workflow examples |
| 5 | Guardrails, accountability, governance, end-to-end review, mock exam drills | 8-10 hrs | Full mock runbook, 30-question self-test, final cheat sheet |
