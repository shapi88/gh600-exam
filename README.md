# GH-600 Exam Study Package

Production-oriented, safety-first prep for **GitHub Certified: Agentic AI Developer (GH-600)** with GitHub as the control plane for agentic SDLC workflows.

> [!TIP]
> If you are time-constrained, start with the **5-week study plan** below, work through each topic file in `topics/`, then review the **cheat sheets** and **practice questions** for daily drills.

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

Each topic file contains: key concepts, common pitfalls, a step-by-step tutorial, practical exercises, and exam-style Q&amp;A.

## High-Level Study Plan (5 Weeks)

| Week | Focus areas | Time target | Deliverables |
| --- | --- | --- | --- |
| 1 | Agent architecture, SDLC integration, planning vs execution boundaries, human-in-the-loop | 6-8 hrs | One-page architecture notes, autonomy matrix, branch protection + review checklist |
| 2 | Tool use, MCP, permissions, runner environments, workflow design | 8-10 hrs | One repo with an agent-safe workflow, MCP config, least-privilege permissions examples |
| 3 | Memory, state, execution control, retries, resumability, observability | 6-8 hrs | Memory strategy table, state diagram, incident playbook for stuck/failed runs |
| 4 | Evaluation, error analysis, prompt tuning, multi-agent coordination | 8-10 hrs | Eval rubric, failure taxonomy, planner/executor/reviewer workflow examples |
| 5 | Guardrails, accountability, governance, end-to-end review, mock exam drills | 8-10 hrs | Full mock runbook, 30-question self-test, final cheat sheet |
