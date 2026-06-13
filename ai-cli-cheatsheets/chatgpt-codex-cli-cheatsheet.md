# ChatGPT / Codex CLI Cheat Sheet

**Tool:** `codex` (OpenAI Codex CLI – lightweight coding agent in the terminal, powered by ChatGPT models)  
**Alternative names / perception:** Often referred to as the "ChatGPT CLI" or "OpenAI terminal agent" for coding.  
**Install (recommended standalone):**
```bash
curl -fsSL https://chatgpt.com/codex/install.sh | sh
```
**Other install methods:**
```bash
npm install -g @openai/codex
brew install --cask codex          # macOS
```
**Windows:** PowerShell installer from the same site.

**Verify:** `codex --version`  
**Repo:** https://github.com/openai/codex  
**Docs:** https://developers.openai.com/codex/cli

---

## Basic Usage
```bash
codex                           # Launch the interactive coding agent
codex "add rate limiting to the API endpoints"
```

**Piped / one-shot usage (very powerful):**
```bash
git diff | codex "Write a concise commit message"
cat failing_test.log | codex "Explain the failure and suggest a minimal fix"
npm test 2>&1 | codex "Fix all failing tests"
```

## Key CLI Flags & Patterns
Codex is designed as a fast, Rust-based peer to Claude Code and Gemini CLI.

| Usage                          | Description                                      | Example |
|--------------------------------|--------------------------------------------------|---------|
| `codex`                        | Start interactive TUI/agent session              | `codex` |
| `codex "<prompt>"`             | Start with a task / initial prompt               | `codex "refactor the payment module"` |
| `codex -p "<prompt>"`          | Headless / non-interactive mode (if supported in your build) | Check `codex --help` |
| Pipe input                     | Feed diffs, logs, code via stdin                 | `... \| codex "..."` |
| `--yolo` / auto-approve        | Skip permission prompts (use carefully)          | `codex --yolo "..."` |
| Approval mode cycling          | Usually `Shift+Tab` inside the agent             | Normal → Plan → Auto |
| `--cwd` / directory control    | Target specific project                          | Supported via flags or launch dir |
| Model / auth                   | Uses your ChatGPT login or API key (see docs)    | `codex login` or env config |

**Headless / CI examples:**
```bash
# Pre-commit / PR guard
git diff --staged | codex "Review for obvious bugs and security issues. Output OK or a bullet list." \
  | tee review.txt && grep -q "^OK" review.txt || exit 1

# Batch processing
for f in src/*.js; do
  codex -p "Convert $f to TypeScript with proper types" --yolo
done
```

**Capture structured results:**
```bash
codex -p "Analyze this repo and return a JSON object with: summary, main_files, risks" --yolo | jq .
```

## Inside the Agent – Slash Commands & Features
Codex follows the converged 2025–2026 agentic CLI design:

- `/help` and command palette (often `Ctrl+P` or `?`)
- `/compact` or context compression commands
- Model switching and plan mode
- Approval mode controls (`/yolo` or equivalent)
- Strong support for file editing with inline diffs
- Shell command execution with permission gating
- Git workflows, GitHub integration (via gh CLI)

**File / context references (`@` style, consistent family behavior):**
```
@src/app.ts
@src/
@package.json Describe the dependencies
```

## Authentication
- **ChatGPT subscription login** (recommended for latest models): browser-based or device flow on first run.
- **API key** for automation / CI (more consistent model access timing).
- See official docs for `codex login` and token management.

## Project Rules & Configuration (AGENTS.md style)
- Supports `AGENTS.md` (and similar) at the repository root — highly recommended.
- Per-project settings in `.codex/` or equivalent (check current version docs).
- Checkpointing / memory features for long-running tasks.
- MCP server support for extending tools (GitHub, databases, etc.).

## Safety & Approval Modes
- Default: asks before file writes and shell execution.
- `Shift+Tab`: cycle through modes (common binding across this class of tools).
- `--yolo` / full auto-approve for trusted automation.
- Best practice: combine YOLO with narrow prompts + `AGENTS.md` guardrails.

## Keyboard Shortcuts (Typical)
- `Enter` — send prompt
- `Shift+Enter` — insert newline
- `Ctrl+C` — interrupt / cancel
- `Shift+Tab` — change approval mode
- `/` — slash command menu
- Arrow / `j/k` navigation in scrollback and history
- Accept / reject keys for proposed edits (usually `y` / `n` or Enter/Esc)

Exact current bindings are shown in the TUI help or shortcuts bar.

## Useful Top-Level Commands
```bash
codex --help
codex --version
codex login
codex                 # interactive
# Inside session: use /help or the command palette
```

## Comparison with Siblings
- **Codex CLI** is OpenAI's direct answer to **Claude Code** and **Gemini CLI**.
- Built in Rust → very fast startup and low overhead.
- Excellent for quick "ChatGPT in the terminal" coding/agent loops.
- Pairs naturally with ChatGPT Plus / Team plans for the best models.
- Many teams standardize on one of {Grok, Claude Code, Gemini CLI, Codex} per project or preference.

**Resources:**
- Official install & docs: https://chatgpt.com/codex and https://developers.openai.com/codex/cli
- GitHub: https://github.com/openai/codex
- Community Codex CLI cheat sheets (Shipyard etc.)

## Day-to-Day Companion Workflows

Codex CLI is frequently praised for speed and for being a natural extension of an existing ChatGPT subscription. Heavy users treat it as both a fast executor and a great "reviewer brain" to pair with Claude or Gemini.

**Morning status + plan**
```bash
codex "Start of my day. Summarize recent git activity and any failing checks. Update AGENTS.md if new conventions emerged. Then give me a clear, prioritized plan with todos (plan mode / read-only first)."
```

**Pipe-driven debug + fix (Codex is excellent here)**
```bash
npm test 2>&1 | codex "Diagnose the failure. Propose the smallest fix. Do not edit until I approve the plan."
# After review
codex "Apply the approved minimal fix. Run the test command. Show exact output and the diff."
```

**Feature work with verification gate**
```bash
codex "Work on <feature>. First produce (or follow) plan.md in plan mode. After each logical piece, run the project's test/build and report. Only mark a step complete after verification passes."
```

**PR description + retro (very strong at this)**
```bash
git diff main...HEAD | codex "Write a crisp PR description, call out risks, and list exactly what was verified."
codex "Session retro: decisions made, open questions, suggested AGENTS.md or retros/ updates. Concise, appendable format."
```

**Long autonomous stretch**
```bash
# Use loops or the exec/headless form (check your `codex --help`)
codex "Continue the migration according to the plan. Report progress every few minutes and stop on any blocker or after the next major milestone."
```

**Parallel reviewer pattern (community favorite)**
Run Codex in one pane/worktree as the executor and feed its plans/diffs to Claude (or another Codex instance) in another pane for review before merging the changes. The speed of Codex makes the "two brains" loop very practical.

## Shell Companion Setup (macOS + zsh)

```bash
# Codex (OpenAI)
alias cx='codex'
alias cxp='codex'                      # plan first inside or via prompt
alias cxhead='codex -p'                # or the exec/headless variant for your build
alias cxreview='codex "You are a fast, strict code reviewer. ... "'

# Safe daily wrapper
cxplan() {
  codex --rules "Plan/read-only mode first. Clear todos. No edits until approved." "$@"
}
```

Many users on ChatGPT Plus/Pro get a lot of mileage out of the subscription-based auth for both the web/desktop Codex surface and the CLI.

## Persistent Memory, Config & Project Rules for Daily Use

**Key files:**
- `AGENTS.md` (repo root or global) — the durable companion memory. Highly recommended.
- Per-project or global `config.toml` (Rust-style) for model prefs, behaviors, etc.
- Skills catalog and MCP configuration for repeated capabilities.

**AGENTS.md starter** (works great with Codex):
```markdown
# Standing orders for Codex CLI companion
- Plan first for anything non-trivial. Use clear todos.
- Smallest safe verifiable step. Run tests/build and show output before done.
- Never touch protected files (.github/workflows, CODEOWNERS, AGENTS.md, etc.) without human approval.
- Prefer worktree or feature branch for non-trivial changes.
- At end of session, produce a retro and memory suggestions.
- Respect secrets and .gitignore.
```

## Guardrails, Safety & GH-600 Patterns

- Clear approval mode cycling (Shift+Tab) + subscription-aware YOLO behavior.
- AGENTS.md + config.toml for standing orders and guardrails.
- Excellent fit for "planner / executor / reviewer" role separation (use one Codex for execution, another or Claude for review).
- Native support for structured output makes headless runs easy to audit (JSON, artifacts).
- Subscription login is convenient for daily interactive use; API key mode for CI/automation with more predictable access.
- Combine with git worktrees and explicit permission scoping (via prompts + AGENTS.md) for least-privilege daily practice.
- Encode the exam hard stops directly in AGENTS.md and prompts.

## Power-User Tips, Custom Commands & Advanced Patterns

- Speed makes "plan in one brain, execute/review in another" extremely practical.
- Skills + MCP for packaging daily rituals (review checklist, test generator, retro writer).
- Config.toml for persistent preferences (model choice per task type, etc.).
- Headless "exec" style usage inside scripts, make targets, or pre-commit for repeatable gates.
- Good multimodal / image + code workflows when using the broader Codex surface alongside the CLI.
- Community pattern: use Codex for fast implementation or review passes while reserving heavier reasoning models for architecture.

## Headless Scripting & Automation Templates

```bash
codex -p "Analyze the repo and return compact JSON: {\"summary\":..., \"main_risks\":[]}" | jq .
```

Subscription users can automate a surprising amount of repetitive coding and review work directly from the CLI without separate API billing in many cases. See official docs for the exact headless/exec forms and the broader Codex desktop/app integration.

## Troubleshooting & Gotchas

- Make sure you're using the correct package (`@openai/codex`) and recent Node for npm installs.
- Auth: ChatGPT subscription login is great for daily interactive; switch to API key for CI predictability.
- Context and "when to start fresh vs continue" is still a skill — use AGENTS.md + retros to keep sessions focused.
- As with all these tools, the highest-leverage safety habit is "plan (or read-only) first, one step at a time, verify."

Codex shines as the fast, always-available ChatGPT-powered terminal teammate and as the "second reviewer" in a multi-CLI daily setup.

---

*Enriched for gh600-exam. Based on official OpenAI Codex CLI docs + GitHub (openai/codex), community automation and daily-use patterns (AGENTS.md, skills, config.toml, speed + reviewer pairing, subscription vs API, headless exec), and alignment with the other three agentic CLIs (Grok / Gemini / Claude Code) as of 2026.*

