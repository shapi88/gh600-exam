# GH-600 Exam Study Package

Production-oriented, safety-first prep for **GitHub Certified: Agentic AI Developer (GH-600)** with GitHub as the control plane for agentic SDLC workflows.

> [!TIP]
> If you are time-constrained, start with the **5-week study plan** below, work through each topic file in `topics/`, then review the **cheat sheets** and **practice questions** for daily drills.

<details>
<summary>⚡ 1-Hour Crash Course — most exam-critical bullets</summary>

1. **Role separation is non-negotiable.** Planner ≠ Executor ≠ Reviewer. No agent approves its own PR.
2. **GitHub is the control plane.** Every important decision lives in an Issue, PR, Actions log, or artifact — never in transient chat.
3. **Least privilege on every job.** Always add an explicit `permissions:` block; never rely on default token scopes.
4. **Human gates at three points.** Plan approval (before execution), PR merge (branch protection + required review), production deploy (environment protection rule with required reviewer).
5. **Hard stops are platform-enforced.** `.github/workflows/`, `CODEOWNERS`, and `copilot-instructions.md` are blocked by CODEOWNERS + required review — not just by prompt instructions.
6. **Memory belongs in the right store.** Secrets → never stored. Conventions → `copilot-instructions.md` (repository memory). Checkpoint progress → PR description or artifact. One-off debug flags → prompt context only.
7. **Idempotency before retries.** Check for existence before creating. Running the same step twice must produce the same state, not duplicates.
8. **4-dimension rubric.** Every eval scenario must pass: Correctness, Scope Control, Safety (no Partial allowed), Auditability. One Safety Fail = scenario Fail regardless of other scores.
9. **Multi-agent: serialize writes, parallelize reads.** Use `needs:` and artifacts for handoffs. Conflict on the same file → escalate to human.
10. **Prompts guide; platform controls enforce.** Any policy stated only in `copilot-instructions.md` is advisory. Back every hard requirement with branch protection, CODEOWNERS, or an environment rule.

> See [Cheat Sheets](topics/cheat-sheets.md) and the [Workflow Diagram](topics/workflow-diagram.md) for the full reference tables.

</details>

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
| — | [Workflow Diagram](topics/workflow-diagram.md) | Complete lifecycle diagram: all actors, phases, human gates, hard stops, and artifacts |
| — | [Cheat Sheets](topics/cheat-sheets.md) | All summary tables in one place for rapid review |
| — | [Resources and Final Exam Strategy](topics/resources.md) | Official docs, practice links, exam strategy tips |

Each topic file contains: key concepts, common pitfalls, a step-by-step tutorial, practical exercises, exam-style Q&amp;A, and a **Complete Working Example** section with copy-pasteable commands and full workflow YAML.

## Deep Dives

Each deep-dive file is a companion to its topic. It links every concept to the authoritative GitHub documentation section, adds implementation detail, and provides additional exercises.

| # | Deep Dive file | What it adds |
| --- | --- | --- |
| 1 | [Agent Architecture and SDLC — Deep Dive](deep-dives/01-agent-architecture-sdlc-deep.md) | Coding agent lifecycle API mapping, branch protection matrix, CODEOWNERS ordering rules |
| 2 | [Tool Use and Environment Interaction — Deep Dive](deep-dives/02-tool-use-environment-deep.md) | Full permissions matrix, MCP threat model, setup steps authoring, runner isolation tradeoffs |
| 3 | [Memory, State, and Execution — Deep Dive](deep-dives/03-memory-state-execution-deep.md) | `copilot-instructions.md` limits, artifact retention, cache vs. artifact, idempotency patterns |
| 4 | [Evaluation, Error Analysis, and Tuning — Deep Dive](deep-dives/04-evaluation-error-tuning-deep.md) | Re-run modes, code scanning API gates, check run rubric, GitHub Models SDK evals |
| 5 | [Multi-Agent Coordination — Deep Dive](deep-dives/05-multi-agent-coordination-deep.md) | Reusable workflows as handoff contracts, PR review API, environment trust gates, conflict resolution |
| 6 | [Guardrails and Accountability — Deep Dive](deep-dives/06-guardrails-accountability-deep.md) | Environments API, audit log queries, push protection, Dependabot triage, Copilot org policies |
| — | [GitHub Docs Index](deep-dives/docs-index.md) | All GitHub documentation URLs organised by topic for rapid look-up |

## Agentic AI Template

> [!TIP]
> Use this template to turn any repository into a production-ready, guardrailed agentic AI environment in 5 steps. Every file links to the GH-600 lesson that governs it.

### Quickstart — 5 steps to a running, guardrailed agent

| Step | Action | File(s) to create |
| --- | --- | --- |
| 1 | Copy the agent standing orders and memory hints | `.github/copilot-instructions.md`, `.github/copilot-memory.md` |
| 2 | Define ownership and branch protection | `CODEOWNERS`, `.github/branch-protection.md` |
| 3 | Register approved MCP servers and define skills | `.github/mcp-config.json`, `.github/skills/*.yml` |
| 4 | Activate the guardrail workflows | `.github/workflows/guardrails-check.yml`, `agent-pr-review.yml`, `agent-issue-triage.yml` |
| 5 | Configure the `agent-execution` environment with a Required Reviewer | GitHub UI: Settings → Environments |

### Configuration Files (deploy to your repo)

| File | GH-600 topic | What it does |
| --- | --- | --- |
| [.github/copilot-instructions.md](.github/copilot-instructions.md) | Topic 1, 6 | Agent standing orders: MAY / MAY NOT / stop conditions / autonomy level |
| [.github/copilot-memory.md](.github/copilot-memory.md) | Topic 3 | Persistent context: file map, conventions, baselines, checkpoints |
| [.github/guardrails.md](.github/guardrails.md) | Topic 6 | Autonomy matrix, hard stops, audit trail requirements, rollback procedures |
| [.github/mcp-config.json](.github/mcp-config.json) | Topic 2 | Approved MCP server registry with scopes and deny list |
| [.github/branch-protection.md](.github/branch-protection.md) | Topic 1 | Auditable record of required branch protection settings |
| [CODEOWNERS](CODEOWNERS) | Topic 1, 6 | Human review requirements for governance and workflow files |
| [.github/INCIDENT_RESPONSE.md](.github/INCIDENT_RESPONSE.md) | Topic 6 | Step-by-step runbook for bad agent actions |
| [.github/ISSUE_TEMPLATE/agent-incident.md](.github/ISSUE_TEMPLATE/agent-incident.md) | Topic 6 | Structured incident report template |

### Skills (`.github/skills/`)

| Skill | GH-600 topic | Autonomy | What it does |
| --- | --- | --- | --- |
| [review-pr.yml](.github/skills/review-pr.yml) | Topic 1, 2 | Level 2 | Reads PR diff; posts structured review comment |
| [triage-issue.yml](.github/skills/triage-issue.yml) | Topic 1, 2 | Level 2 | Classifies new issues and applies a label |
| [generate-docs.yml](.github/skills/generate-docs.yml) | Topic 3 | Level 2 | Drafts a topic, tutorial, or deep-dive file on a new branch |
| [run-eval.yml](.github/skills/run-eval.yml) | Topic 4 | Level 1 | Runs scenarios against a rubric and uploads a results artifact |
| [multi-agent-planner.yml](.github/skills/multi-agent-planner.yml) | Topic 5 | Level 1 | Decomposes a task into a plan artifact; never executes |

### Workflows (`.github/workflows/`)

| Workflow | Trigger | What it does |
| --- | --- | --- |
| [guardrails-check.yml](.github/workflows/guardrails-check.yml) | Every push | Secret scan, scope control, attribution check |
| [agent-pr-review.yml](.github/workflows/agent-pr-review.yml) | `pull_request` | Posts structured review comment; hard-stops on sensitive paths |
| [agent-issue-triage.yml](.github/workflows/agent-issue-triage.yml) | `issues: opened` | Applies label and posts triage comment; detects secret patterns |
| [agent-eval.yml](.github/workflows/agent-eval.yml) | `workflow_dispatch` | Runs evaluation scenarios and uploads results artifact |
| [agent-multi-agent-planner.yml](.github/workflows/agent-multi-agent-planner.yml) | `workflow_dispatch` | Planner → human approval gate → executor → reviewer pipeline |

### Evaluation Files (`evals/`)

| File | What it does |
| --- | --- |
| [evals/agent-template-scenarios.md](evals/agent-template-scenarios.md) | 5 golden scenarios for the template (PR review, triage, doc gen, eval, multi-agent) |
| [evals/agent-template-rubric.md](evals/agent-template-rubric.md) | 4-dimension rubric: correctness, safety, auditability, scope control |
| [evals/results-template.md](evals/results-template.md) | Blank results template — copy to `results-v<N>.md` for each run |

### Copy-Paste Templates (`templates/`)

| Template | What it provides |
| --- | --- |
| [templates/manual.md](templates/manual.md) | **Start here** — step-by-step guide to configure all Copilot capabilities from scratch |
| [templates/copilot-instructions-example.md](templates/copilot-instructions-example.md) | Fully annotated `copilot-instructions.md` with guidance |
| [templates/copilot-memory-example.md](templates/copilot-memory-example.md) | Annotated `copilot-memory.md` template with field-by-field explanation |
| [templates/mcp-config-example.json](templates/mcp-config-example.json) | MCP config with 3 annotated example servers and deny list |
| [templates/skill-example.yml](templates/skill-example.yml) | Annotated skill YAML with every field explained |
| [templates/workflow-agent-safe-example.yml](templates/workflow-agent-safe-example.yml) | Complete least-privilege agent workflow with guard step |
| [templates/workflow-multi-agent-example.yml](templates/workflow-multi-agent-example.yml) | Planner → approval gate → executor → reviewer pipeline |
| [templates/pr-checklist.md](templates/pr-checklist.md) | Agent PR description template with accountability checklist |
| [templates/artifact-schema.json](templates/artifact-schema.json) | JSON Schema for inter-agent handoff artifacts |
| [templates/guardrails-example.md](templates/guardrails-example.md) | Annotated guardrails policy with autonomy matrix |
| [templates/incident-report-example.md](templates/incident-report-example.md) | Filled-out incident report example |
| [templates/mcp-server-entry.json](templates/mcp-server-entry.json) | Blank MCP server entry for registering a new tool |

### GH-600 Alignment

| Lesson area | Template coverage |
| --- | --- |
| Topic 1 — Agent Architecture | `copilot-instructions.md`, autonomy levels, `CODEOWNERS`, `branch-protection.md`, `workflow-diagram.md` |
| Topic 2 — Tool Use & MCP | `mcp-config.json`, least-privilege workflows, network policy entries |
| Topic 3 — Memory & State | `copilot-memory.md`, `copilot-memory-example.md`, `pr-checklist.md`, `artifact-schema.json`, idempotency pattern |
| Topic 4 — Evaluation | `agent-template-scenarios.md`, `agent-template-rubric.md`, `results-template.md` |
| Topic 5 — Multi-Agent | Planner/executor/reviewer workflow, handoff artifact schema, skill definitions, `workflow-diagram.md` |
| Topic 6 — Guardrails | `guardrails.md`, `INCIDENT_RESPONSE.md`, incident issue template, `guardrails-check.yml`, `workflow-diagram.md` |

---

## High-Level Study Plan (5 Weeks)

| Week | Focus areas | Time target | Deliverables |
| --- | --- | --- | --- |
| 1 | Agent architecture, SDLC integration, planning vs execution boundaries, human-in-the-loop | 6-8 hrs | One-page architecture notes, autonomy matrix, branch protection + review checklist |
| 2 | Tool use, MCP, permissions, runner environments, workflow design | 8-10 hrs | One repo with an agent-safe workflow, MCP config, least-privilege permissions examples |
| 3 | Memory, state, execution control, retries, resumability, observability | 6-8 hrs | Memory strategy table, state diagram, incident playbook for stuck/failed runs |
| 4 | Evaluation, error analysis, prompt tuning, multi-agent coordination | 8-10 hrs | Eval rubric, failure taxonomy, planner/executor/reviewer workflow examples |
| 5 | Guardrails, accountability, governance, end-to-end review, mock exam drills | 8-10 hrs | Full mock runbook, 30-question self-test, final cheat sheet |
