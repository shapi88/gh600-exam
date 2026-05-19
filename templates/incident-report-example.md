# Example: Filled-Out Incident Report

> This is a fully completed example incident report.
> Use it as a reference when filling in `.github/ISSUE_TEMPLATE/agent-incident.md`.
> Copy the template (not this example) when filing a real incident.

---

## Incident Summary

**Date/time discovered (UTC):** 2026-05-19T09:42Z  
**Discovered by:** @shapi88  
**Severity:** P2 High  
**Status:** Resolved

---

## What Happened

The `generate-docs` skill opened a PR that modified both `topics/01-agent-architecture-and-sdlc.md` (the intended file) and `README.md` (an unintended file). The README change was a minor formatting update the agent inferred would be helpful, but it was not requested and exceeded the skill's intended scope.

---

## Impact

- **Files modified:** `topics/01-agent-architecture-and-sdlc.md` ✅ (intended), `README.md` ❌ (unintended)
- **PRs affected:** #47
- **Credentials exposed:** No
- **Production affected:** No
- **Users affected:** None — PR was caught during review and not merged.

---

## Timeline

| Time (UTC) | Event |
| --- | --- |
| 2026-05-19T09:30Z | `generate-docs` skill triggered via `workflow_dispatch` |
| 2026-05-19T09:31Z | Agent opened PR #47 with 2 files changed |
| 2026-05-19T09:42Z | @shapi88 noticed README.md in diff during review |
| 2026-05-19T09:45Z | PR #47 closed; branch deleted |
| 2026-05-19T10:00Z | Root cause identified |
| 2026-05-19T10:15Z | Skill YAML updated with tighter scope constraint |
| 2026-05-19T10:20Z | Incident issue filed |
| 2026-05-19T10:30Z | Incident closed |

---

## Root Cause

**Which stop condition was not triggered?**  
No stop condition existed for "diff touches files outside the requested document path". The skill only checked for `.github/workflows/` and `CODEOWNERS`.

**Which guardrail was not enforced?**  
The scope-control check in `guardrails-check.yml` only detected workflow and CODEOWNERS changes. It did not verify that only the requested file was modified.

**Which skill or workflow was involved?**  
`.github/skills/generate-docs.yml` and `.github/workflows/agent-multi-agent-planner.yml` (executor job).

**Root cause category:**
- [x] Missing stop condition in `copilot-instructions.md`
- [ ] Over-permissive `permissions:` block in workflow
- [x] Skill YAML had incorrect scope
- [ ] Missing CODEOWNERS coverage
- [ ] Missing environment protection / approval gate
- [ ] Unapproved MCP server call
- [ ] Other

---

## Containment Actions Taken

- [x] Harmful PR closed / reverted — PR #47 closed, branch `copilot/gen-docs-topic-01` deleted
- [x] Agent branch deleted
- [ ] Credential rotated — N/A
- [ ] Harmful artefact removed from history — N/A
- [ ] Affected users notified — N/A

---

## Control Changes Made

| File | Change made |
| --- | --- |
| `.github/copilot-instructions.md` | Added stop condition: "Stop if the diff touches files outside the path specified in the task input." |
| `.github/skills/generate-docs.yml` | Added stop condition: "Output path is the ONLY path that may be modified." |
| `.github/workflows/guardrails-check.yml` | Added step: verify that agent branch diff touches only the expected file pattern |
| `.github/guardrails.md` | Updated scope-control section to include per-task path enforcement |

---

## Verification

- [x] `main` is in a clean, expected state — verified with `git log --oneline main | head -5`
- [x] Branch protection settings verified intact — `enforce_admins: true`, `required_reviews: 1`
- [x] Guardrails-check workflow passes — run ID: 9876543
- [x] CODEOWNERS-designated maintainer sign-off: @shapi88

---

## Lessons Learned

The scope-control guardrail needs per-skill path constraints, not just global path checks. When a skill is given a target path as input, it should only touch that path — this constraint must be encoded in both the skill YAML stop conditions and the workflow-level guardrails check.

---

> Filed per `.github/INCIDENT_RESPONSE.md` runbook.  
> Skill: `generate-docs`. Autonomy Level: 2. Workflow run: #9876543.
