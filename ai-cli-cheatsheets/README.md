# AI CLI Command Cheat Sheets

This folder contains focused command cheat sheets (with practical examples) for the major agentic coding / terminal AI CLIs as of 2026.

## Files

| File | CLI | Provider | Command | Focus |
|------|-----|----------|---------|-------|
| [grok-cli-cheatsheet.md](./grok-cli-cheatsheet.md) | Grok Build | xAI | `grok` | Full TUI + headless, tools, MCP, sessions |
| [gemini-cli-cheatsheet.md](./gemini-cli-cheatsheet.md) | Gemini CLI | Google | `gemini` | Agentic terminal, @-files, slash commands, YOLO |
| [claude-code-cli-cheatsheet.md](./claude-code-cli-cheatsheet.md) | Claude Code | Anthropic | `claude` | Agentic coding REPL, GitHub workflows, strong reasoning |
| [chatgpt-codex-cli-cheatsheet.md](./chatgpt-codex-cli-cheatsheet.md) | Codex CLI (ChatGPT) | OpenAI | `codex` | Fast Rust-based coding agent, pipes, structured output |

## Quick Start (all of them)

Most follow the same modern pattern:

```bash
# Interactive
<tool>

# With initial task
<tool> "do the thing"

# Headless / scripting (one-shot)
<tool> -p "do the thing" --yolo

# Pipe context
git diff | <tool> "write a commit message"
cat logs.txt | <tool> "debug this"
```

Common power features across the family:
- `@path` or `@dir` to attach files/directories
- `/` slash commands for meta actions (compact, model, plan, etc.)
- `Shift+Tab` to cycle approval modes (Normal → Plan → Auto/YOLO)
- `AGENTS.md` (or tool-specific equivalent) for project instructions
- MCP servers for extra tools
- Headless mode for CI / automation
- Session persistence + resume

## Notes for gh600-exam

These tools represent the current generation of **terminal-native agentic coding assistants**. They all:
- Understand the local filesystem and git context
- Can read, edit, and run code with permission gating
- Support long-running multi-turn work with compaction
- Are scriptable for evals, pre-commit, and autonomous workflows
- Integrate with MCP and custom rules

Use the individual sheets as quick references when comparing capabilities, permission models, output formats, and integration patterns.

## Daily AI CLI Companion Quickstart

Treat these as always-available pair programmers / engineering teammates that live in your terminal.

**Safe daily launch habits (all tools):**
```bash
# Preferred safe entry (plan first)
<tool> --cwd . "Plan the changes for <task>. Do not edit files yet."

# Or use built-in plan mode where available (Shift+Tab often cycles to it)
<tool>
# inside: /plan or Shift+Tab → Plan mode

# Quick one-off (still gated)
git diff --staged | <tool> "Review for bugs and security. Reply with OK or a short list."
```

**Recommended shell aliases (add to ~/.zshrc or ~/.bashrc, macOS friendly):**
```bash
# Grok (xAI) - strong all-rounder + imagination
alias g='grok'
alias gp='grok --plan'                 # or just use /plan inside
alias gsafe='grok'                     # explicit: no yolo
alias ghead='grok -p'                  # headless prefix

# Claude Code (Anthropic) - often praised for code quality + GitHub flows
alias cc='claude'                      # community favorite short alias
alias ccp='claude --plan'              # or start and Shift+Tab /plan
alias ccreview='claude "Act as a strict senior reviewer. ..."'  # wrapper idea

# Gemini CLI (Google) - excellent long context + Google ecosystem via MCP
alias gm='gemini'
alias gmp='gemini --plan'              # or native Plan Mode
alias gmhead='gemini -p'

# Codex / ChatGPT CLI (OpenAI) - fast Rust agent, great with existing ChatGPT sub
alias cx='codex'
alias cxp='codex --plan'               # plan first
alias cxhead='codex -p'                # or the exec/headless form in your version
```

After editing rc: `source ~/.zshrc`. Then `cc`, `gm`, `g`, `cx` become muscle memory.

**One powerful daily pattern (works across all):**
```bash
# 1. Plan
<tool> "Create a detailed plan.md for <feature/bug>. Use todo list. Stay in plan/read-only mode."

# 2. Review the plan yourself (or pipe to another CLI for adversarial review)
cat plan.md | <other-tool> "Review this plan for scope, safety, and completeness."

# 3. Execute one step at a time (or after human approval)
<tool> "Follow plan.md. Implement ONLY the first unchecked todo item. Then stop and show the diff."
```

See the individual sheets for tool-specific flags, custom commands (`/retro`, custom .toml skills, etc.), and more ritual examples.

## Which Companion for Which Daily Task? (Companion Fitness Comparison)

| Aspect (Daily Lens)              | Grok (xAI)                              | Gemini CLI (Google)                          | Claude Code (Anthropic)                     | Codex CLI (OpenAI)                         |
|----------------------------------|-----------------------------------------|----------------------------------------------|---------------------------------------------|--------------------------------------------|
| **Quick tasks / fast feedback** | Excellent                               | Very good (esp. flash models)                | Excellent (strong reasoning even on smaller) | Best-in-class speed (Rust) + subscription |
| **Deep refactors / architecture** | Strong (imagination + tools)            | Excellent long context + planning            | Often top-rated for thorough code quality   | Very strong, especially with good prompts |
| **Multi-day / long sessions**   | Great session resume + memory cmds      | Strong (GEMINI.md + checkpoints)             | Outstanding (CLAUDE.md + retro habits)      | Good (AGENTS.md + config) + loops         |
| **GitHub-heavy workflows**      | Good (MCP + gh integration via tools)   | Good + extensions                            | Outstanding native gh + issue→PR flows     | Strong (especially with Codex ecosystem)  |
| **Google / Drive / ecosystem**  | Via MCP                                 | Native advantage via MCP/extensions          | Via MCP                                     | Via MCP                                   |
| **Safety / guardrails ergonomics** | Excellent (detailed --allow/--deny, plan mode, yolo toggle, rules) | Strong native Plan Mode + readonly customs   | Excellent (plan mode, CLAUDE.md, skills)    | Clear modes + AGENTS.md + subscription YOLO awareness |
| **Custom commands / extensibility** | Skills + / commands + MCP               | Excellent .toml custom commands + MCP        | Skills / plugins + custom slash             | Skills catalog + MCP + config             |
| **Headless / scripting / CI**   | Mature (json, streaming, python wrappers, resume) | Good (GitHub Action exists)                  | Good (pipes, automation patterns)           | Strong (exec mode, npm, cloud ties)       |
| **GH-600 alignment (guardrails, audit, least-priv)** | Top tier (sandbox, allow/deny, worktree, export, plan) | Very good (plan mode, custom workflows)      | Excellent (deep GitHub controls + plan)     | Strong (modes + artifacts + subscription) |
| **Best paired reviewer**        | Any of the others                       | Good for broad research                      | Often paired with Codex for adversarial review | Excellent as "second brain" reviewer      |

**Rule of thumb for a typical dev day:**
- Exploration / research / Google stuff → Gemini
- High-stakes refactors, docs, careful GitHub work → Claude Code
- Fast iteration, existing ChatGPT sub, speed-critical → Codex
- Balanced creative + agentic + strong safety tooling → Grok
- Many people run 2 in parallel (one for planning/execution, one for review) using tmux + different worktrees.

## Unified Companion Recommendations (for Daily Use)

**Persistent memory that survives days (use all of them where possible):**
- `AGENTS.md` (or `CLAUDE.md`, `GEMINI.md`, `.codex/AGENTS.md` etc.) at repo root or `~/.grok/AGENTS.md` — put standing orders, style, "always run tests before claiming done", repo map, non-sensitive conventions.
- Per-CLI dotfiles/settings for checkpoints, temperature, custom commands.
- At end of every focused session: ask for a "retro" or "what did we learn / decisions made" and append (or link) to a `session-retros.md` or the project memory file. Keeps institutional knowledge without bloating one context.

**Shared starter AGENTS.md snippet** (works across all four; adapt slightly):
```markdown
# Standing orders for AI coding companions
- Always start complex work in plan / read-only mode.
- Make the smallest safe change that moves us forward. One logical step per turn when possible.
- Never edit .github/workflows/, CODEOWNERS, or AGENTS.md / copilot-instructions.md without explicit human instruction.
- Run relevant tests (or the test command I specify) and show output before claiming a task complete.
- Use git worktree or a feature branch for anything non-trivial.
- Export or note key decisions so a human (or future session) can audit.
- Respect .gitignore and never touch secrets.
```

**MCP servers worth having for daily dev (least-privilege scopes):**
- GitHub (read issues/PRs, create comments, limited writes)
- Filesystem with path restrictions
- Terminal / shell (with allow-lists)
- Any internal company tools or docs MCPs

**Terminal setup tips:**
- Ghostty / WezTerm / iTerm2 with good truecolor + mouse.
- tmux (or zellij) + named sessions/worktrees lets you run multiple companions in parallel on the same or different branches (planner in one pane, executor in another, reviewer in a third).
- Many power users keep a "companion" tmux window always running.

**Maintenance:**
- Update the CLIs regularly (`grok update`, `npm update -g @google/gemini-cli`, etc.).
- Periodically review your AGENTS.md / memory files for staleness.
- Use the "retro at end of session" habit — it compounds.

See the individual sheets for the exact commands, flags, and tool-specific rituals that turn these into true daily companions.

---

*Generated as part of the gh600-exam materials. Enriched for practical day-to-day use while staying aligned with safe agentic development principles.*

