# Evaluation Results — Template

Copy this file to `results-v<N>.md`, fill in every field, and commit it to `evals/`.
Do not edit existing `results-v*.md` files — always create a new version.

---

**Prompt version:** <!-- e.g. v3 -->
**Date:** <!-- ISO 8601, e.g. 2026-05-19 -->
**Evaluator:** <!-- GitHub username -->
**Scenarios file:** <!-- e.g. evals/agent-template-scenarios.md -->
**Rubric file:** <!-- e.g. evals/agent-template-rubric.md -->
**Change from previous version:** <!-- One sentence: what was tuned? -->

---

## Results Table

| Scenario | Correctness | Safety | Auditability | Scope Control | Overall | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| T1: PR review — safe diff | _fill_ | _fill_ | _fill_ | _fill_ | _fill_ | |
| T2: PR review — sensitive-path stop | _fill_ | _fill_ | _fill_ | _fill_ | _fill_ | |
| T3: Issue triage — standard | _fill_ | _fill_ | _fill_ | _fill_ | _fill_ | |
| T4: Eval run — artifact produced | _fill_ | _fill_ | _fill_ | _fill_ | _fill_ | |
| T5: Multi-agent plan — executor blocked | _fill_ | _fill_ | _fill_ | _fill_ | _fill_ | |

---

## Summary

| Metric | Value |
| --- | --- |
| Total scenarios | 5 |
| Pass count | _fill_ |
| Fail count | _fill_ |
| Pass rate | _fill_% |
| Gate threshold | 80% |
| **Gate result** | **_fill_ — Pass / Fail** |

---

## Root Causes for Failures

<!-- For each Fail or Partial, describe: -->
<!-- - Which scenario and dimension failed -->
<!-- - What the agent did vs. what was expected -->
<!-- - The root cause (missing stop condition, over-permission, no attribution, etc.) -->

---

## Tuning Actions for Next Version

<!-- One tuning action per root cause. Only change one variable at a time. -->

| Root cause | Tuning action | Target file |
| --- | --- | --- |
| | | |

---

## Raw Observations

<!-- Paste the raw agent output or workflow log excerpt for each scenario here. -->

### T1 Raw Output

```
(paste here)
```

### T2 Raw Output

```
(paste here)
```

### T3 Raw Output

```
(paste here)
```

### T4 Raw Output

```
(paste here)
```

### T5 Raw Output

```
(paste here)
```
