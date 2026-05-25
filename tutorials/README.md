# Hands-On Labs — Navigation Index

Each lab is self-contained and builds on a fresh GitHub repository. Complete them in order; each lab creates the setup that the next lab extends.

> [!NOTE]
> **Start with Lab 00.** It installs the GitHub CLI, creates your practice repo, and sets up Node.js and Python. No other lab will work without it.

---

## Lab Sequence

| Lab | File | Prerequisite | What you will build | Estimated time |
| --- | --- | --- | --- | --- |
| 00 | [Prerequisites](00-prerequisites.md) | GitHub account | GitHub CLI, practice repo, fine-grained PAT, Node.js + Python | 30 min |
| 01 | [Agent Architecture & SDLC](01-agent-architecture-lab.md) | Lab 00 | Protected `main`, CODEOWNERS, governed `issue → branch → PR` flow | 45 min |
| 02 | [Tool Use & Environment Interaction](02-tool-use-lab.md) | Lab 01 | Least-privilege workflow, environment gate, approved MCP config | 45 min |
| 03 | [Memory, State & Execution](03-memory-state-lab.md) | Lab 02 | `copilot-instructions.md`, PR checklist checkpoints, artifact upload, idempotent retries | 45 min |
| 04 | [Evaluation, Error Analysis & Tuning](04-evaluation-lab.md) | Lab 03 | Golden scenarios, rubric, versioned prompt results, scope-control and secret-scan workflow | 60 min |
| 05 | [Multi-Agent Coordination](05-multi-agent-lab.md) | Lab 04 | Planner → executor → reviewer → coordinator pipeline with artifact handoffs | 60 min |
| 06 | [Guardrails & Accountability](06-guardrails-lab.md) | Lab 05 | Platform-enforced guardrails, production deployment with approval gate, rollback runbook | 45 min |

---

## What Each Lab Produces

After completing all labs your practice repository will have:

- `main` branch with branch protection (required review + status checks + no admin bypass)
- `CODEOWNERS` gating workflow files, infra, and security-related paths
- `.github/copilot-instructions.md` defining agent standing orders at Autonomy Level 2
- Two workflows: a least-privilege agent workflow and a multi-agent pipeline
- `evals/` folder with golden scenarios, rubric, and two versioned result files
- A deployment workflow gated behind an `environment: production` approval
- Incident response runbook and rollback procedure

---

## Tips for Getting the Most Value

- **Build, don't just read.** Every concept is reinforced by running the commands, not by reading them.
- **Break things on purpose.** After completing a lab, try bypassing a control (e.g., push directly to `main`) to confirm the protection actually fires.
- **Reference the topic file.** Each lab links to the corresponding `topics/` file that explains the theory behind what you built.
- **Reuse across exam prep.** The practice repo you build is your study artifact — revisit it when reviewing the cheat sheets.
