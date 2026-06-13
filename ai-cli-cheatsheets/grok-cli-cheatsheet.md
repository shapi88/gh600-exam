# Grok CLI Cheat Sheet

**Tool:** `grok` (xAI Grok Build TUI / Agentic CLI)  
**Version example:** 0.2.51  
**Homepage / Install:** `curl -fsSL https://x.ai/cli/install.sh | bash`

---

## Installation & Version
```bash
curl -fsSL https://x.ai/cli/install.sh | bash
grok --version
grok update
grok update --check
```

## Basic Usage
```bash
grok                           # Launch interactive TUI
grok "fix the auth bug"        # Start TUI with initial prompt
grok --cwd ~/my-project        # Specific directory
```

## Key CLI Flags
| Flag                        | Description                                      | Example |
|-----------------------------|--------------------------------------------------|---------|
| `-p, --single <PROMPT>`     | Headless: run once, print result, exit           | `grok -p "Review this PR"` |
| `--prompt-file <PATH>`      | Headless prompt from file                        | `grok --prompt-file task.txt` |
| `-m, --model <MODEL>`       | Choose model (e.g. grok-build)                   | `grok -m grok-build -p "..."` |
| `-c, --continue`            | Continue most recent session (cwd)               | `grok -c -p "continue the work"` |
| `-r, --resume [<ID>]`       | Resume specific session                          | `grok -r abc123` |
| `--always-approve` / `--yolo`| Auto-approve all tool use (dangerous)           | `grok --yolo -p "..."` |
| `--worktree [<NAME>]`       | Start in new git worktree                        | `grok --worktree=feat "implement X"` |
| `--output-format <plain\|json\|streaming-json>` | Headless output format | `... --output-format json` |
| `--rules <TEXT>`            | Append rules to system prompt                    | `--rules "Use TypeScript only"` |
| `--tools` / `--disallowed-tools` | Comma list of tools (headless)               | `--disallowed-tools run_terminal_cmd` |
| `--allow <RULE>` / `--deny <RULE>` | Permission rules (repeatable)             | `--allow "Bash(npm*)" --deny "Bash(sudo*)"` |
| `--max-turns <N>`           | Limit turns (headless)                           | `--max-turns 5` |
| `--sandbox <PROFILE>`       | Sandbox profile                                  | `--sandbox strict` |
| `--no-subagents`            | Disable subagents                                | `--no-subagents` |

**Headless examples:**
```bash
grok -p "Explain the architecture" --output-format json | jq -r '.text'
grok -p "Fix failing tests" --yolo --resume $(cat .last-session-id)
cat diff.patch | grok -p "Write a commit message for this diff"
```

## Inside the TUI – Slash Commands (`/`)
Type `/` for fuzzy autocomplete.

**Session:**
- `/new` (or `/clear`)
- `/resume`
- `/compact [keep ...]`
- `/fork`, `/rewind`
- `/context`, `/session-info`
- `/quit` / `/home`

**Model & Mode:**
- `/model <name>` (`/m`)
- `/always-approve` (or `/yolo [on|off]`)
- `/plan [description]`
- `/multiline` (`/ml`)
- `/vim-mode`, `/compact-mode`

**Other:**
- `/copy [N]`, `/export`
- `/imagine <prompt>`, `/imagine-video`
- `/loop 30m check status`
- `/remember ...`, `/flush`, `/dream` (memory)
- `/settings`, `/theme`, `/feedback`
- `/skills`, `/plugins`, `/mcps`, `/hooks`
- `/btw <note>` (aside without interrupting)

## Keyboard Shortcuts (Quick)
- `Ctrl+N` — new session
- `Ctrl+Q` / `Ctrl+D` — quit (double press)
- `Ctrl+C` — cancel turn
- `Ctrl+O` — toggle YOLO/auto-approve
- `Ctrl+P` / `?` — command palette
- `Ctrl+M` — model picker / multiline toggle
- `Enter` — send (prompt focused)
- `Shift+Enter` — newline
- Arrows / `j/k` (vim) — scrollback nav
- `y` — copy selected block
- `Shift+Tab` — cycle modes (Normal/Plan/Auto-approve)

**Vim mode:** Enable in `~/.grok/config.toml` `[ui] vim_mode = true` or toggle with `/vim-mode`.

## File References
```
@src/main.rs
@src/main.rs:20-80
@src/               # whole dir
@!.env              # force hidden/dot files
```

## Project Rules
- `AGENTS.md` (or `CLAUDE.md`) in repo root, `~/.grok/AGENTS.md`, or cwd
- Deeper files win.

## Other Tips
- Sessions auto-persist in `~/.grok/sessions/`
- Use `--yolo` + `--rules` for safe automation
- `grok inspect` — show discovered config for current dir
- `grok sessions list/search`
- `grok mcp ...` — manage MCP servers
- Completions: `grok completions zsh`

**Full help:** `grok --help` and `grok <subcommand> --help`

## Day-to-Day Companion Workflows

Treat Grok as your always-on engineering teammate. These rituals come from heavy daily use + the rich built-in tooling (plan mode, /loop, memory commands, worktrees, export).

**Morning project sync + planning (recommended start)**
```bash
grok "It is the start of my day. Review recent git log + changes since yesterday. Update any relevant notes in AGENTS.md or a retro file. Then propose a prioritized plan for today's work. Stay in plan mode."
# Then: Shift+Tab or /plan if not already; review the plan; approve first step only.
```

**Bug triage & fix loop (safe)**
```bash
# Pipe the error + context
npm test 2>&1 | tail -100 | grok "Diagnose the root cause. Do not edit yet — produce a minimal repro + proposed fix plan."

# After you approve the plan
grok "Implement ONLY the first step of the approved plan. Run the test after the change and report the result. Stop."
```

**Safe multi-step refactor (one step at a time)**
```bash
grok --worktree=refactor-auth "We are refactoring the auth module. First, explore and create a detailed plan.md with todos. Use the todo_write tool. Do not make edits."
# Review plan.md yourself (or pipe to another CLI for review)
grok "Follow plan.md. Complete the NEXT unchecked todo only. Commit the change with a clear message. Update the plan with status. Stop and show the diff + test results."
```

**PR prep + retro (end of focused work)**
```bash
git diff main...HEAD | grok "Write a high-quality PR description + checklist. Highlight risks and what was verified (tests, manual steps)."
grok "What did we learn or decide in this session? Produce a concise retro entry suitable for appending to a session-retros.md or AGENTS.md."
grok /export   # or manually save the transcript for audit
```

**End-of-day checkpoint**
```bash
grok "/context; /session-info; summarize what is still in flight and any open questions. Suggest the best prompt to resume tomorrow."
# Use /remember or append to project memory file for durable facts.
```

**Long-running / babysit task**
```bash
/loop 5m "Continue the build + test fix loop we started. Report progress every cycle. Stop only on success or clear blocker."
```

**Onboarding / exploring a new (or old) codebase**
```bash
grok "Map the high-level architecture, entry points, and key files. Create a living ARCHITECTURE.md snippet and a todo list of 'things a new dev should know'. Use read-only exploration first."
```

## Shell Companion Setup (macOS + zsh recommended)

Add to `~/.zshrc` (then `source ~/.zshrc`):

```bash
# Grok aliases — emphasize safety by default
alias g='grok'
alias gp='grok'                    # then inside: Shift+Tab for Plan or start with /plan
alias ghead='grok -p'              # headless
alias gsafe='grok'                 # explicit reminder
alias ginspect='grok inspect'      # quick discovered config
alias gresume='grok -c'            # continue most recent

# Useful functions
gplan() {
  grok --rules "Start in plan mode. Produce clear todos and stop before any edits." "$@"
}

gretro() {
  grok "At the end of this session: produce a concise retro of decisions made, open questions, and what to remember tomorrow. Suggest an update for AGENTS.md or a retros/ file."
}

# Headless with JSON for scripting / capture
gjson() {
  grok -p "$*" --output-format json
}
```

Completions are already provided by the installer (`grok completions zsh`).

**Pro tip:** Combine with `gh` (GitHub CLI) and worktrees for parallel companion sessions:
```bash
git worktree add -b feat-x ../myrepo-feat-x
(cd ../myrepo-feat-x && g "Work on feature X in this isolated tree.")
```

## Persistent Memory, Config & Project Rules for Daily Use

**Durable memory locations (in precedence order):**
- `AGENTS.md` (repo root, `~/.grok/AGENTS.md`, or cwd) — highest priority for the project. Use for conventions, "always run tests before done", scope limits, style.
- Per-session: `/remember "fact"`, `/flush` (forces summary to memory), `/dream` (consolidation).
- `~/.grok/config.toml` (see below).
- Session transcripts in `~/.grok/sessions/` (resumable with `-c` / `-r`).

**Recommended `~/.grok/config.toml` daily companion settings** (start minimal):
```toml
[cli]
auto_update = true

[models]
default = "grok-build"

[ui]
vim_mode = false          # or true if you love j/k everywhere
max_thoughts_width = 100

[session]
auto_compact_threshold_percent = 80   # compact a bit earlier for long days
load_envrc = true

[tools]
respect_gitignore = true
```

**Project AGENTS.md starter** (copy into repo root; Grok + many other CLIs will pick it up):
```markdown
# Standing orders
- Always begin complex or risky work in plan mode.
- Smallest safe change that advances the goal. Verify (tests + manual) before claiming done.
- Never touch protected paths (.github/workflows, CODEOWNERS, AGENTS.md, etc.) without explicit human OK.
- Use git worktree or a clean branch for anything > 1 file.
- Export or note key decisions for auditability.
- Respect .gitignore and secrets.
```

Deeper files win. Also works alongside the project's `copilot-instructions.md` / `copilot-memory.md` style files from the GH-600 templates.

## Guardrails, Safety & GH-600 Patterns

Map directly to exam principles:
- **Default to plan / read-only first** (`/plan`, Shift+Tab, or start the prompt with "plan only, no edits").
- **Least privilege on every run**: Use `--allow "Bash(npm*)" --deny "Bash(sudo*)"` or the full `--tools` / `--disallowed-tools` allow/denylist in headless. `--sandbox` for extra isolation.
- **Human gates**: Keep YOLO off for anything that touches main, secrets, workflows, or production. Use `grok --yolo` only inside a worktree + narrow `--rules` + after plan review.
- **Audit trail**: `grok -p "..." --output-format json`, `/export`, or `grok trace`. Land real changes via PRs (never direct to protected branches).
- **Idempotency & checkpoints**: Ask Grok to update a todo list or plan.md; use `/remember` for durable non-sensitive facts.
- **Hard stops alignment**: Explicitly tell Grok (via prompt + AGENTS.md) the GH-600 prohibited list (modify workflows, approve own PR, expose secrets, etc.).
- **Worktree for isolation**: `grok --worktree=experiment "..."` — changes stay off your main checkout until you merge.

Never let the companion become the approver of its own risky actions.

## Power-User Tips, Custom Commands & Advanced Patterns

- **One step at a time** is the highest-leverage habit: "Implement only the next todo. Stop and report."
- **Retro at end of every session**: `grok "Produce a retro: decisions, open questions, what to remember tomorrow. Suggest AGENTS.md update."` Then `/remember` the durable bits or append to a retros file.
- **/btw for side notes** without breaking flow.
- **Custom skills** (user-invocable SKILL.md) appear as `/myskill`. Great for `/retro`, `/review-pr`, `/generate-tests`.
- **/loop** for babysitting builds, deploys, or long refactors.
- **Parallel companions**: Different worktrees + tmux panes (one for planning/execution, one for review using a different model or even another CLI).
- **Vim mode + keyboard power**: Enable `[ui] vim_mode = true` for j/k navigation, y/Y copy, h/l fold. Full quickref cards are in the official keyboard shortcuts doc.
- **Context hygiene**: `/compact keep <important facts>`, `/context` to watch usage, `/flush` before a risky compact.
- **Model switching**: `/model grok-build` (or whatever is current best for the task).

## Headless Scripting & Automation Templates

Basic one-liner + capture:
```bash
grok -p "Review this diff for security issues only. Output JSON with issues array." --output-format json --rules "Be strict." | jq .
```

Reusable Python wrapper (adapted from official Grok examples):
```python
# grok_companion.py (headless helper)
import asyncio, json, os, subprocess

async def grok_headless(prompt, model="grok-build", yolo=False, cwd="."):
    cmd = ["grok", "-p", prompt, "-m", model, "--cwd", cwd,
           "--output-format", "json"]
    if yolo: cmd.append("--yolo")
    proc = await asyncio.create_subprocess_exec(*cmd, stdout=subprocess.PIPE)
    out, _ = await proc.communicate()
    return json.loads(out)

# Example: safe review gate
result = asyncio.run(grok_headless("Review staged changes. Reply with OK or list issues.", yolo=False))
print(result.get("text"))
```

Use in pre-commit hooks, GitHub Actions (with XAI_API_KEY), or cron companions.

## Troubleshooting & Gotchas

- Auth expires → `grok login` or set `XAI_API_KEY`.
- Terminal quirks (Ctrl+Enter, Cmd+A, mouse) → see official 21-terminal-support.md or run `/terminal-setup`.
- Vim mode confusing the prompt vs scrollback → remember they are independent; toggle with `/vim-mode`.
- Context bloat → `/compact`, `/context`, start new sessions more often, use worktrees.
- Over-eager edits → always start with explicit "plan mode / read-only first".
- Headless ignores some TUI flags (e.g. `--session-id` is TUI-only).

Full details live in `~/.grok/docs/user-guide/` (highly recommended reading).

---

*Adapted and enriched for gh600-exam from local ~/.grok/docs (keyboard, slash, config, headless, plan mode, AGENTS.md, permissions) + live CLI + daily companion best practices.*

