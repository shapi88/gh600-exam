# Topic 1: Prepare Agent Architecture and SDLC Processes

## Key Concepts and Sub-Topics

- **Agent roles:**
  - **Planner** — proposes steps, decomposes tasks, identifies risks
  - **Executor** — implements approved steps, changes state or code
  - **Reviewer** — validates output quality, scope, and policy compliance
  - **Policy/guardrail agent** — enforces constraints and escalation rules

- **SDLC placement:**
  - Backlog grooming (labeling, acceptance-criteria drafting)
  - Implementation (code generation on scoped branches)
  - CI triage (failure summarization, flaky-test detection)
  - Code review (diff analysis, style checks)
  - Release preparation (changelog, release notes)

- **Planning vs execution boundaries** — a planning agent should not also have unrestricted execution rights

- **Human-in-the-loop checkpoints** — required approval steps before merging, deploying, or making sensitive changes

- **GitHub as the control plane:**
  - Issues → intent capture and acceptance criteria
  - Pull requests → proposals, diffs, review, and audit trail
  - GitHub Actions → observable, auditable execution path
  - Environments → deployment approval gates
  - Branch protection → enforce review and status checks
  - CODEOWNERS → route sensitive paths to human reviewers

---

## Common Pitfalls / Anti-Patterns

- Giving the same agent both broad planning authority and unrestricted execution.
- Letting an agent merge or deploy without required checks and human approval.
- Hiding decisions in chat instead of durable GitHub artifacts like PR comments, reviews, or workflow logs.
- Using agent autonomy where deterministic automation (a simple workflow or script) is enough.

---

## Step-by-Step Tutorial

**Goal:** Design a safe agent-assisted delivery flow for a small repository.

1. **Create a sample issue** with acceptance criteria, out-of-scope items, and a risk note. This becomes the agent's starting artifact.

2. **Define role boundaries.** Decide which parts of the flow are:
   - Planning-only (agent proposes, nothing changes in the repo)
   - Execution-capable (agent opens branches and commits code)
   - Human-approved (no merge or deploy without a human gate)

3. **Map the GitHub control plane objects involved:**
   ```
   issue → branch → draft PR → CI checks → human review → merge
   ```
   Label each step with the responsible party: agent, automation, or human.

4. **Add branch protection rules** for `main`:
   - Require at least one human review.
   - Require all status checks to pass.
   - Do not allow bypassing the above rules for agents.

5. **Write a short escalation policy** describing when the agent must stop and wait for a human:
   - Touching files listed in `CODEOWNERS`
   - Exceeding a diff-size threshold
   - Encountering a secret or credentials-related file
   - Failing more than N retries

---

## Practical Exercises

- Draw a planner → executor → reviewer flow and label every handoff artifact (issue, PR comment, review, log).
- Given two tasks — a docs-only change and an infra change — assign a safe autonomy level (0–4) to each and justify it.
- Rewrite the risky requirement "agent merges fixes automatically" into a governed workflow with explicit controls.
- Identify three SDLC stages where an agent adds value without execution rights and explain why that boundary is appropriate.

**What a strong answer includes:**
- Explicit role boundaries
- Required human checkpoints
- Durable audit trail in GitHub artifacts
- Least privilege aligned to the SDLC stage

---

## Exam-Style Questions

**Q1.** A team wants an agent to take issues directly to merge in one run.  
**A:** Split responsibilities across planning, execution, CI, and human review. Require protected branches and status checks before merge. The risk is excessive autonomy with no accountability boundary.

**Q2.** An agent can generate code and approve its own PR.  
**A:** Remove self-approval and require independent human or policy review. Approval must be independent for trustworthy governance.

**Q3.** Product managers want faster backlog grooming.  
**A:** Use an agent for summarization, labeling, and acceptance-criteria drafting. Keep prioritization and commitment with humans. This matches low-risk, high-value augmentation.

**Q4.** A repo has frequent small documentation updates.  
**A:** Allow higher autonomy for scoped documentation changes with standard checks. Autonomy should reflect risk and reversibility.

**Q5.** A team asks where to store the agent's rationale for a code change.  
**A:** In the PR description or PR comments tied to the run. GitHub artifacts provide auditability; transient chat does not.

---

## Complete Working Example

The following commands set up a fully governed repository from scratch using the `gh` CLI. Copy-paste each block in order.

### 1. Create a repository and protect `main`

```bash
# Create the practice repo
gh repo create gh600-practice --private --add-readme --clone
cd gh600-practice

# Add branch protection: require PR, 1 review, status checks, no admin bypass
gh api repos/{owner}/{repo}/branches/main/protection \
  --method PUT \
  --field required_status_checks='{"strict":true,"contexts":["lint"]}' \
  --field enforce_admins=true \
  --field required_pull_request_reviews='{"required_approving_review_count":1}' \
  --field restrictions=null
```

Expected output: `HTTP 200` and a JSON object describing the protection rule.

### 2. Add CODEOWNERS

```bash
mkdir -p .github
cat > .github/CODEOWNERS << 'EOF'
/.github/workflows/ @<your-username>
*.yml               @<your-username>
*.yaml              @<your-username>
EOF

git add .github/CODEOWNERS
git commit -m "Add CODEOWNERS to gate workflow and YAML changes"
git push origin main
```

### 3. Create the minimal CI workflow (status check)

```bash
mkdir -p .github/workflows
cat > .github/workflows/ci.yml << 'EOF'
name: ci

on:
  pull_request:
    branches: [main]

permissions:
  contents: read

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false
      - name: Check for trailing whitespace
        run: |
          if grep -rn " $" --include="*.py" --include="*.md" .; then
            echo "Trailing whitespace found."
            exit 1
          fi
          echo "No trailing whitespace found."
EOF

git add .github/workflows/ci.yml
git commit -m "Add minimal CI workflow for branch protection status check"
git push origin main
```

### 4. Demonstrate the governed `issue → branch → PR` flow

```bash
# Open an issue with acceptance criteria
gh issue create \
  --title "Add a greeting function to the codebase" \
  --body "## Acceptance criteria
- [ ] greet(name) added to src/greet.py
- [ ] Returns 'Hello, <name>!'
- [ ] Unit test added

## Out of scope
- CLI wiring

## Risk note
Low risk. No external dependencies." \
  --label "enhancement"

# Create a feature branch and implement
git checkout -b feature/greet-function-issue-1
mkdir -p src tests

cat > src/greet.py << 'EOF'
def greet(name: str) -> str:
    """Return a greeting string for the given name."""
    return f"Hello, {name}!"
EOF

cat > tests/test_greet.py << 'EOF'
from src.greet import greet

def test_greet_happy_path():
    assert greet("Alice") == "Hello, Alice!"

def test_greet_empty_string():
    assert greet("") == "Hello, !"
EOF

git add src/greet.py tests/test_greet.py
git commit -m "Add greet function and unit tests (closes #1)"
git push origin feature/greet-function-issue-1

# Open a draft PR with explicit agent rationale
gh pr create \
  --title "Add greet function (closes #1)" \
  --body "## Summary
Implements greet(name) as specified in issue #1.

## Agent rationale
Used the narrowest possible change: one new file in src/ and one test file.
No existing files modified.

## Checklist
- [x] Function added
- [x] Unit tests added
- [ ] Human review complete" \
  --draft --base main
```

### 5. Verify branch protection blocks a direct push

```bash
# Attempt a direct push to main — this will be rejected
echo "test" > direct-push-test.txt
git add direct-push-test.txt
git commit -m "Attempt direct push to main"
git push origin main
# Expected: remote: error: GH006: Protected branch update failed
```

### Shell commands reference

| Task | Command |
| --- | --- |
| Create branch protection | `gh api repos/{owner}/{repo}/branches/main/protection --method PUT ...` |
| View branch protection | `gh api repos/{owner}/{repo}/branches/main/protection` |
| Create issue | `gh issue create --title "..." --body "..."` |
| Open draft PR | `gh pr create --draft --base main` |
| Mark PR ready | `gh pr ready <number>` |
| List PRs | `gh pr list` |

---

## Hands-On Lab

Follow [Lab 01 — Agent Architecture & SDLC](../tutorials/01-agent-architecture-lab.md) to build this setup step-by-step in your own practice repository.

---

## Relevant Resources

- [GitHub Docs — Copilot overview](https://docs.github.com/en/copilot)
- [GitHub Docs — Use GitHub Copilot agents](https://docs.github.com/en/copilot/how-tos/use-copilot-agents)
- [GitHub Docs — About CODEOWNERS](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners)
- [GitHub Docs — Workflow syntax (permissions, environments, jobs)](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)

---

## Deep Dive

Go further with **[Deep Dive 1 — Agent Architecture and SDLC](../deep-dives/01-agent-architecture-sdlc-deep.md)**, which covers:
- GitHub Copilot coding agent internals and lifecycle API mapping
- GitHub Issues and Projects as structured planning artifacts
- Branch protection and rulesets — complete configuration reference
- CODEOWNERS syntax, ordering rules, and governance patterns
- Additional exercises with authoritative GitHub Docs links for each sub-topic
