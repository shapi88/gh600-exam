# Gemini CLI Cheat Sheet

**Tool:** `gemini` (Google Gemini CLI – open-source AI agent in your terminal)  
**Package:** `@google/gemini-cli`  
**Install:** `npm install -g @google/gemini-cli`  
**Alternative (no global install):** `npx @google/gemini-cli` or `npx https://github.com/google-gemini/gemini-cli`  
**Docs:** https://geminicli.com/ | https://github.com/google-gemini/gemini-cli

---

## Installation & Basics
```bash
npm install -g @google/gemini-cli
gemini --version
gemini --help
gemini update   # or re-run npm install -g @google/gemini-cli@latest
```

Launch:
```bash
gemini                    # interactive TUI/REPL
gemini "refactor the auth module"
```

## Key CLI Flags & Usage Patterns
From official patterns and community:

| Usage / Flag                  | Description                                      | Example |
|-------------------------------|--------------------------------------------------|---------|
| `gemini`                      | Start interactive session                        | `gemini` |
| `gemini -p "<PROMPT>"`        | Headless / non-interactive (single prompt)       | `gemini -p "Summarize the codebase"` |
| `gemini "<PROMPT>"`           | Start interactive with initial prompt            | `gemini "fix the tests"` |
| `cat file \| gemini`          | Pipe content into Gemini                         | `git diff \| gemini "write commit message"` |
| `--prompt` / `-p`             | Explicit prompt for headless                     | See above |
| `--yolo` / YOLO mode          | Auto-approve edits & commands                    | `gemini --yolo -p "..."` or Ctrl+Y toggle |
| `--cwd <path>`                | Run in specific directory                        | `gemini --cwd ~/project` |
| Model / config flags          | Vary by version (see `gemini --help`)            | Check current with `gemini --help` |

**Headless / scripting examples:**
```bash
gemini -p "Review changes for bugs and security" --yolo > review.md

# JSON-ish or structured output via prompt engineering
gemini -p "Return ONLY valid JSON: { \"summary\": \"...\", \"issues\": [] }" --yolo

# Continue context in CI (capture state or use project memory)
gemini -p "Continue from previous work on the payment flow"
```

Piped + project context:
```bash
npm test 2>&1 | gemini -p "These tests are failing. Diagnose and propose fixes."
```

## Inside the TUI – Slash Commands (`/`)
Gemini CLI has a rich set of built-in slash commands (fuzzy searchable).

Common ones (from docs & cheatsheets):
- `/help` — list all commands
- `/compress` or `/compact` — summarize/reduce context (save tokens)
- `/copy` — copy last response to clipboard
- `/stats` (or `/stats model`) — token usage, quota, session stats
- `/mcp` — list/manage configured MCP servers & tools
- `/settings` or config-related commands
- Custom slash commands (via `.toml` files in `.gemini/commands/` or project)

**Custom commands example** (`.gemini/commands/review.toml`):
```toml
description = "Reviews a pull request"
prompt = """
Please provide a detailed pull request review for issue/PR: {{args}}.
Focus on security, performance, and tests.
"""
```

Invoke: `/review 123`

**At commands (`@`)** — very similar to Grok:
```
@src/main.rs
@src/
@README.md Explain this file
@.  # current dir (git-aware filtering usually)
```

Hidden files: some versions support `!` prefix or settings.

## Approval Modes & YOLO
- Default: asks before edits / shell commands
- Cycle modes: often `Shift+Tab` (Normal → Auto-Edit / Plan → YOLO)
- `--yolo` flag or `Ctrl+Y` (or similar) to toggle full auto-approve
- Dangerous: use with `--rules` or scoped prompts in automation

## Project Configuration & Rules
- Global: `~/.gemini/settings.json` or similar
- Project: `.gemini/settings.json`, `GEMINI.md`, or `AGENTS.md` style files (many users repurpose `AGENTS.md`)
- Checkpoints / memory: enable in settings for safer long sessions
- Extensions / MCP: `gemini extensions ...` or `/mcp`

**Example settings snippet** (from community):
```json
{
  "general": {
    "enablePromptCompletion": true
  },
  "checkpointing": { "enabled": true }
}
```

## Keyboard & Interaction (Typical)
- `Enter` — send
- `Shift+Enter` — newline
- Arrow keys / vim-like nav in scrollback (varies)
- `Ctrl+C` — cancel current generation
- `y` (contextual) — accept / copy patterns
- Command palette or `/` for everything
- Mouse support in most modern terminals

## Headless / Automation Tips
- Combine with `--yolo` + strict prompt + allowed paths in wrappers for CI
- Use for GitHub Actions (there is an official `google-github-actions/run-gemini-cli`)
- Capture output with structured prompts (`Return JSON only...`)
- Many wrappers exist for safe sandboxed multi-turn headless use

## Useful Commands
```bash
gemini --help
gemini extensions list | install | update
gemini -p "..." | jq   # when output is parseable
```

## Comparison Notes (with Grok / Claude Code)
- Very similar UX to Grok CLI and Claude Code: TUI + tools + file editing + shell + @-attachments + slash commands.
- Strong Google ecosystem integration (Drive, etc. via MCP/extensions).
- Good at long-context and planning.
- Quota / tier notes: unpaid/Google One users may see replacement notices in 2026.

**Official quick ref:**  
https://geminicli.com/docs/reference/commands/  
https://geminicli.com/docs/get-started/

## Day-to-Day Companion Workflows

Gemini CLI shines at long-context exploration and structured planning. Power users combine native Plan Mode, custom commands, checkpoints, and the official GitHub Action.

**Morning architecture / research sync**
```bash
gemini "Start of day. Analyze recent changes + open issues. Update GEMINI.md with any new conventions discovered. Produce a clear prioritized plan for today (use Plan Mode / read-only tools only)."
# Then switch out of plan mode only after review.
```

**Bug diagnosis with pipe + plan**
```bash
npm test 2>&1 | gemini -p "These tests are failing. Explore the relevant code (read-only). Create a minimal diagnosis + reproduction steps + a step-by-step fix plan. Do not edit yet."
```

**One disciplined step at a time (highly recommended)**
```bash
gemini "Follow the approved plan.md. Implement ONLY the next unchecked item. After the change, run the relevant test/build command and report results exactly. Stop and wait for further instruction."
```

**PR / review preparation + retro**
```bash
git diff main...HEAD | gemini "Act as a thorough senior reviewer + doc writer. Produce PR description, risks, and verification steps."
gemini "End-of-session retro: key decisions, open questions, what to remember for tomorrow. Suggest update to GEMINI.md or a retros/ file."
```

**Long babysit task**
```bash
# Use the /loop or equivalent recurring prompt, or the official GitHub Action for CI
gemini -p "Continue the ongoing migration. Check build status every few minutes via tools. Report progress."
```

**New codebase onboarding**
```bash
gemini "Map the repo: high-level architecture, build/test commands, key modules, and 'gotchas for new contributors'. Write a living ONBOARDING.md. Stay exploratory/read-only first."
```

**tmux + multi-agent pattern (popular with Gemini users)**
Run several Gemini (or mixed) instances in different panes / worktrees: one for planning, one for implementation in a feature tree, one for review. Use the companion VS Code extension for visual feedback when helpful.

## Shell Companion Setup (macOS + zsh)

```bash
# Gemini
alias gm='gemini'
alias gmp='gemini'                     # then use native Plan Mode or /settings to ensure it
alias gmhead='gemini -p'
alias gmext='gemini extensions'        # manage MCP/extensions quickly

# Function for safe plan-first launch
gmplan() {
  gemini --rules "Begin in Plan / read-only mode. Produce clear todos and do not modify files." "$@"
}
```

See `gemini --help` and the extensions subcommand for more.

## Persistent Memory, Config & Project Rules for Daily Use

**Key files:**
- `GEMINI.md` (or `AGENTS.md`) at repo root or home — primary durable memory (many users repurpose AGENTS.md successfully).
- `.gemini/settings.json` (project) or `~/.gemini/settings.json` (global) — checkpoints, prompt completion, model prefs, etc.
- Custom commands: `.gemini/commands/*.toml` (or project-scoped) — turn repeated prompts into `/review-pr`, `/retro`, `/generate-tests`, etc.

**Example settings for daily companion use** (community + official patterns):
```json
{
  "general": {
    "enablePromptCompletion": true
  },
  "checkpointing": { "enabled": true },
  "planMode": { "enabled": true }
}
```

**Custom command example** (`.gemini/commands/retro.toml`):
```toml
description = "End-of-session retro and memory update"
prompt = """
Produce a concise retro of what was accomplished, decisions made, open questions, and suggested updates for GEMINI.md or a retros file.
"""
```

Invoke with `/retro`.

## Guardrails, Safety & GH-600 Patterns

- Native **Plan Mode** (read-only by design) is a first-class citizen — use it liberally (Shift+Tab or settings). Perfect match for "plan before execute".
- Custom read-only workflows (community PRAR or readonly/plan/review/implement patterns using marker files + custom commands) give extra discipline.
- Permission model + YOLO (`--yolo` or Ctrl+Y) — treat like the others: default off for anything risky; combine with narrow prompts and AGENTS.md/GEMINI.md limits.
- GitHub Action exists for CI/headless automation — great for auditable runs.
- Scope work with worktrees or explicit path rules when using MCP/tools.
- Export / structured JSON output for audit trails.
- Explicitly encode hard stops (protected paths, own-PR approval, secret exposure, etc.) in GEMINI.md + prompt.

## Power-User Tips, Custom Commands & Advanced Patterns

- Plan Mode + custom commands for repeatable discipline (many users script their entire perceive → reason → act → refine flow).
- Checkpointing + GEMINI.md for multi-day continuity.
- `/loop` style recurring execution (or external schedulers) for long tasks.
- Pair with the official Gemini CLI Companion VS Code extension for visual / IDE context when needed.
- Use the rich MCP/extension system for Google services (Drive, Sheets) or GitHub without leaving the terminal.
- Community favorites: structured modes (readonly first), retro at end of every significant session, one-step-at-a-time, adversarial review by feeding plans to another model/CLI.
- Extensions command for managing the "companion toolkit".

## Headless Scripting & Automation Templates

```bash
gemini -p "Return ONLY compact JSON: {\"summary\": \"...\", \"risks\": []}" --yolo | jq .
```

Official GitHub Action makes it easy to run Gemini CLI agents as part of CI with proper scoping.

Many users wrap the CLI in small scripts that enforce "plan first" or inject a readonly marker before handing control to the agent.

## Troubleshooting & Gotchas

- Quota / tier surprises (especially unpaid/Google One users in 2026) — have a fallback model or CLI.
- Plan Mode feels limiting at first — that's the point; exit it only after explicit review.
- Custom commands are case-sensitive and live in `.gemini/commands/`.
- Extensions/MCP need explicit trust + scoping (aligns with GH-600 least-privilege).
- Long contexts still benefit from deliberate compaction and good GEMINI.md hygiene.

See the official docs for the latest on Plan Mode, custom commands, and the companion extension.

---

*Enriched for gh600-exam from official Gemini CLI docs (plan mode, commands, get-started, settings), GitHub repo, community workflows (PRAR/custom modes, tmux, retro, GEMINI.md, custom .toml), and cross-CLI patterns. Original compilation from 2026 sources preserved.*

