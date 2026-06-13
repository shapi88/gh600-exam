# Claude Code CLI Cheat Sheet

**Tool:** `claude` (Anthropic Claude Code – agentic coding assistant in the terminal)  
**Package:** `@anthropic-ai/claude-code`  
**Install options:**
```bash
npm install -g @anthropic-ai/claude-code
# or the official installer (often recommended)
curl -fsSL https://claude.ai/install.sh | bash
```
**Verify:** `claude --version`

**Official:** https://code.claude.com/ | https://github.com/anthropics/claude-code

---

## Basic Usage
```bash
claude                           # Launch interactive REPL / TUI
claude "implement the new login flow using OAuth"
claude --help
```

**Piped input (very common pattern):**
```bash
cat error.log | claude
npm test 2>&1 | claude "Diagnose the failing tests and fix them"
git diff | claude "Write a high-quality commit message and PR description"
```

## Key CLI Patterns & Flags
Claude Code follows the same modern agentic CLI paradigm as Grok and Gemini CLI.

| Command / Flag              | Description                                      | Example |
|-----------------------------|--------------------------------------------------|---------|
| `claude`                    | Interactive coding session (REPL/TUI)            | `claude` |
| `claude "<prompt>"`         | Start with an initial task                       | `claude "refactor utils.ts for performance"` |
| `claude -p` or similar      | Headless / one-shot (check `claude --help`)      | `claude -p "..."` (varies by version) |
| Pipe to `claude`            | Feed context via stdin                           | `... \| claude` |
| Approval / auto mode        | YOLO / auto-approve (Shift+Tab cycle often)      | Enable in session or via flags |
| `--yolo` / auto-approve     | Full auto (use with care)                        | `claude --yolo "..."` |
| Worktree / cwd support      | Often supported via flags or inside TUI          | Check current `claude --help` |
| Model selection             | Via slash or config (Claude 3.5/4 Sonnet/Opus etc.) | Inside session or flags |

**Headless / scripting examples:**
```bash
# One-liner review
git diff --cached | claude "Review these staged changes for bugs. Reply OK or list issues." | grep -q "^OK" || exit 1

# Structured output
claude -p "Analyze the repo and output a JSON plan with tasks" --yolo > plan.json
```

## Inside the Session – Slash Commands
Claude Code has powerful built-in slash commands and a command palette.

Common / typical commands (from guides and usage):
- `/help` — available commands
- `/compact`, `/compress`, or context management commands
- `/model` — switch model
- `/yolo` or approval mode toggles
- `/plan` — enter plan mode
- GitHub integration slash commands (issues → PRs, etc.)
- `/resume`, session management
- Custom prompts via configuration or MCP

It integrates deeply with GitHub (via gh CLI), runs shell, edits files with diffs, etc.

**File attachment style:** Uses `@` for files/directories (consistent with the family of tools):
```
@src/components/Login.tsx
@.                     # project root
```

## Approval Modes & Safety
- Default: permission prompts for edits and dangerous commands
- `Shift+Tab` (common binding): cycle Normal → Plan → Auto-Approve (YOLO)
- Global/project config for trusted paths or auto-approve rules
- Use `--yolo` (or equivalent) + very specific prompts for CI/automation

## Project Instructions & Memory
- Supports `AGENTS.md` (and similar convention files) at repo root
- Many users also use `CLAUDE.md` or `.claude/` directory for instructions
- Persistent memory / checkpoints configurable
- Session history and forking often available

## Keyboard Shortcuts (Typical for the family)
- `Enter` — submit
- `Shift+Enter` — multiline
- `Ctrl+C` — cancel current turn
- `Ctrl+O` or similar — toggle auto-approve (exact keys can vary)
- `Shift+Tab` — cycle modes
- `/` — open slash command menu / command palette
- Arrow keys + `j/k` navigation in history
- `y` / accept keys for diffs

Run inside the TUI for the exact current bindings: usually `?` or a help command.

## MCP, Extensions & Integrations
- Full MCP server support (like Grok & Gemini)
- GitHub integration (read issues, create PRs, etc.)
- Works alongside your existing terminal tools (gh, git, npm, etc.)
- Can be used from IDEs, Slack, web, in addition to pure terminal

## Useful Top-Level Commands
```bash
claude --version
claude --help
claude                    # interactive
# Inside: /help for session commands
```

**Pro patterns from community:**
```bash
# Pre-commit style guard
git diff --staged | claude "Are there obvious bugs? Reply only OK or ISSUES: ..." 

# Full autonomous run (with guardrails)
claude "Migrate the project to the new API. Create a plan first, then implement step by step." --yolo
```

## Comparison Notes
- Extremely similar workflow to **Grok CLI** and **Gemini CLI**.
- Often praised for strong reasoning, tool use, and code quality on complex tasks.
- Excellent GitHub workflow integration (issues → code → PRs).
- Token / plan management is first-class.

**Resources:**
- https://code.claude.com/
- GitHub: anthropics/claude-code
- Community cheat sheets & guides (Shipyard, etc.)

## Day-to-Day Companion Workflows

After 6+ months of daily use, the highest-leverage habits reported across the community are consistent: plan mode first, one step at a time, retro at the end of every session, CLAUDE.md as living project memory, and treating the agent as an orchestrator/reviewer more than a typist.

**Start-of-day planning ritual**
```bash
claude "Good morning. Review git activity since last session + any open issues or TODOs. Update CLAUDE.md with new context if needed. Then produce a clear, prioritized plan for today with a todo list. Remain in plan mode."
# Review the plan (yourself or feed to a second model/CLI for adversarial review). Approve only the first step.
```

**Bug fix with self-correction loop**
```bash
cat error.log | claude "Diagnose the root cause. Do not edit files yet. Produce a plan + minimal repro."
# After approval
claude "Implement the first step of the plan. Let the tests or build run. If they fail, fix your own mistake and try again before asking me. Show the final diff and test output."
```

**Complex feature with explicit retro**
```bash
claude --worktree=feat-x "Build feature X following the spec in specs/x.md. Create a plan.md first (plan mode). After each logical chunk, run tests and update the plan status. At the very end, produce a detailed retro of decisions and open questions."
```

**PR preparation + memory capture**
```bash
git diff main...HEAD | claude "Write an excellent PR description, highlight risks and verification steps, and suggest any CLAUDE.md updates."
claude "Session retro: what was accomplished, what we learned, what a future session (or human) should know. Append-worthy format."
```

**End-of-day or handoff**
```bash
claude "Summarize current state, open todos, and the single best prompt to resume this work tomorrow. Update any relevant memory files."
```

**Long-running or babysat task**
```bash
# Inside session
/loop 5m "Continue the work. Report progress each cycle. Use the plan as your source of truth."
```

Many power users run Claude Code + another agent (often Codex) in parallel tmux panes / worktrees: one executes, one reviews.

## Shell Companion Setup (macOS + zsh)

```bash
# Claude Code (very common short alias in the community)
alias cc='claude'
alias ccp='claude'                     # then Shift+Tab or /plan inside
alias cchead='claude'                  # pipe or prompt-based headless patterns
alias ccreview='claude "You are a strict, security-conscious senior reviewer. ... "'

# Safe plan-first wrapper
ccplan() {
  claude --rules "Begin and stay in plan mode until explicitly told to implement. Produce clear todos." "$@"
}
```

Some users alias `cc='claude --dangerously-skip-permissions'` only inside trusted worktrees after building deep trust (the flag name is intentionally scary).

## Persistent Memory, Config & Project Rules for Daily Use

**Primary durable memory:**
- `CLAUDE.md` (or `AGENTS.md`) at repo root — the single most mentioned "game changer" for daily use. High-level facts, decisions, conventions, "always run tests", scope rules.
- Many teams also keep per-feature `retros/` or `session-notes.md` files and link to them from CLAUDE.md (avoids monolithic context bloat).
- Sub-agents / skills and project config for repeated procedures.

**CLAUDE.md starter** (highly recommended by daily users):
```markdown
# Claude Code standing orders for this repo
- Always propose a plan (or use Plan Mode) for anything non-trivial. Get explicit approval before writing code.
- One logical step at a time. Stop after each step, run verification, and report.
- Never modify .github/workflows, CODEOWNERS, CLAUDE.md, or AGENTS.md without human instruction.
- Run the project's test/build command(s) and show output before claiming a task is complete.
- Prefer git worktree or a dedicated branch for changes > a couple of files.
- At the end of every focused session, produce a retro and suggest memory updates.
```

## Guardrails, Safety & GH-600 Patterns

- Plan Mode (and the common "plan first" prompt discipline) is the closest thing to a built-in read-only safety net.
- Skills + custom commands let you codify "always plan", "run reviewer checklist", "retro" as first-class `/commands`.
- Community strongly recommends keeping dangerous auto-approve off by default and using it only after building trust + with worktree isolation.
- GitHub integration is excellent — use it to read issues, post structured reviews, create branches/PRs, but still respect branch protection + required human review for merges/deploys.
- Encode the exam's hard stops explicitly in CLAUDE.md + prompts.
- Use worktrees liberally for parallel/experimental work so mistakes don't pollute your main checkout.
- Export or capture session output + PR descriptions as durable audit artifacts.

## Power-User Tips, Custom Commands & Advanced Patterns (from heavy daily users)

- **Plan → one step → verify → retro** is the dominant successful loop.
- **CLAUDE.md + separate retro notes** beats dumping everything into one giant context.
- **Two models are better than one**: many run Claude for execution and feed plans/diffs to Codex (or another Claude instance) for adversarial review.
- Custom skills/plugins for repeated rituals (`/retro`, `/review-pr`, `/generate-tests-with-coverage`).
- `/loop` (or external schedulers) for long autonomous stretches.
- Worktrees + tmux for true parallel agents on the same machine.
- "Don't fix it yourself — let Claude fix its own mistakes" so it builds a better mental model.
- Visual work (when using desktop companion) + terminal for the actual edits.

## Headless Scripting & Automation Templates

Pipe-heavy usage is first-class:
```bash
git diff --cached | claude "Review for obvious bugs and security. Reply with OK or a short bullet list." | tee review.txt
```

Wrap in pre-commit or CI guards the same way shown in the basic examples. Many teams also drive Claude Code via the broader Codex desktop/app surface or MCP when more orchestration is needed.

## Troubleshooting & Gotchas

- Plan Mode can feel slow at first — the time is usually recovered in far fewer bad edits.
- Large CLAUDE.md files bloat context; prefer pointers to focused notes.
- GitHub auth / gh CLI must be set up for the best issue/PR flows.
- Skills and custom commands have their own loading/trust model — review them like any other extension.
- Token usage and "when to compact / start fresh" is a daily skill.

The community resources (especially long "50 tips" and "workflow after 6 months" posts) are gold for advanced daily patterns.

---

*Enriched for gh600-exam (2026) from official sources, GitHub (anthropics/claude-code), and extensive community daily-use reports (plan mode first, CLAUDE.md, one-step, retro, skills, two-model review, tmux+worktrees, aliases, guardrails). Original patterns preserved.*

