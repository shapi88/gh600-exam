---
name: Agent Incident Report
about: Report and document a harmful or unexpected action caused by an agentic AI workflow
title: "[INCIDENT] <short description>"
labels: incident, agent-action
assignees: shapi88
---

## Incident Summary

**Date/time discovered (UTC):** <!-- e.g. 2026-05-19T14:30Z -->
**Discovered by:** <!-- GitHub username -->
**Severity:** <!-- P1 Critical | P2 High | P3 Medium | P4 Low -->
**Status:** <!-- Open | Contained | Resolved -->

---

## What Happened

<!-- One paragraph: what did the agent do, and why was it harmful? -->

---

## Impact

- **Files modified:** <!-- List affected files or "none" -->
- **PRs or issues affected:** <!-- List PR/issue numbers -->
- **Credentials exposed:** <!-- Yes / No — if Yes, have they been rotated? -->
- **Production affected:** <!-- Yes / No -->
- **Users affected:** <!-- Describe or "none" -->

---

## Timeline

| Time (UTC) | Event |
| --- | --- |
| | Agent action triggered |
| | Harm discovered |
| | Containment started |
| | Containment completed |
| | Root cause identified |
| | Controls updated |
| | Incident closed |

---

## Root Cause

**Which stop condition was not triggered?**  
<!-- Refer to .github/copilot-instructions.md -->

**Which guardrail was not enforced?**  
<!-- Refer to .github/guardrails.md -->

**Which skill or workflow was involved?**  
<!-- Skill file + workflow file name -->

**Root cause category:**
- [ ] Missing stop condition in `copilot-instructions.md`
- [ ] Over-permissive `permissions:` block in workflow
- [ ] Missing CODEOWNERS coverage for affected path
- [ ] Missing environment protection / approval gate
- [ ] Unapproved MCP server call
- [ ] Skill YAML had incorrect scope
- [ ] Other: <!-- describe -->

---

## Containment Actions Taken

- [ ] Harmful PR closed / reverted
- [ ] Agent branch deleted
- [ ] Credential rotated (if applicable)
- [ ] Harmful artefact removed from history (if applicable)
- [ ] Affected users notified (if applicable)

---

## Control Changes Made

<!-- List every file updated to prevent recurrence and describe the change -->

| File | Change made |
| --- | --- |
| `.github/copilot-instructions.md` | <!-- describe or "no change" --> |
| `.github/guardrails.md` | <!-- describe or "no change" --> |
| `CODEOWNERS` | <!-- describe or "no change" --> |
| `.github/workflows/<name>.yml` | <!-- describe or "no change" --> |
| `.github/mcp-config.json` | <!-- describe or "no change" --> |

---

## Verification

- [ ] `main` is in a clean, expected state
- [ ] Branch protection settings verified intact
- [ ] Guardrails-check workflow passes
- [ ] CODEOWNERS-designated maintainer sign-off: <!-- @username -->

---

## Lessons Learned

<!-- What would have prevented this incident? One or two sentences. -->

---

> Filed per `.github/INCIDENT_RESPONSE.md` runbook.
