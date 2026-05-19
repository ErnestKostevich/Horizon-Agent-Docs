# Agent mode

Two modes side-by-side in Horizon:

| Mode | What it does | When to use |
|---|---|---|
| **Chat** | Single-turn reply. Pure language model. | Quick questions, brainstorming, writing |
| **Agent** | Multi-step plan → execute tools → reflect → reply | Anything that needs to *do* something on your machine |

Switch with the **Agent** button in the chat header (or `/agent` in the
TUI, or `horizon agent "..."` in the CLI).

## What agent mode gives the AI

When you switch to Agent mode, Horizon shows a consent modal listing
every capability the agent gets:

- 🖥️ **See your screen** — screenshots on demand or automatic when
  your task mentions the UI
- 🖱️ **Move the mouse & click** — `computer.click`, `smart_click`
  (vision-guided), drag, scroll
- ⌨️ **Type on the keyboard** — `computer.type`, `keyboard.press` for
  any key combination
- 📂 **Read & write files** — under your workspace, with a permission
  prompt for every write
- 🐚 **Run shell commands** — Python / Node / shell / PowerShell,
  per-call permission gate
- 🌐 **Browse the web** + connector tools (Slack, Notion, Linear,
  WhatsApp, Signal, ...)
- 🔁 **Self-reflect & retry** — up to 8 multi-step iterations per task

You see this list **once**. After consent, a sticky banner
("AGENT IN CONTROL · screen · mouse · keyboard · files · shell ·
network") appears whenever Agent mode is active.

## The permission gate

Every destructive action prompts. By default:

| Action | Prompt? |
|---|---|
| `read_file`, `list_dir`, `search_files`, `web_search` | No (read-only) |
| `recall`, `get_facts`, `remember` | No (your own data) |
| `run_code` (any language), `run_shell` | **Yes** |
| `write_file`, `delete_file`, `move_file` | **Yes** |
| `mouse_click`, `keyboard_type`, `smart_click` | **Yes** |
| Send a message via Telegram/Discord/Slack/WhatsApp/Signal/iMessage | **Yes** |

You can:
- **Approve once** — that call goes through
- **Approve always for this skill** — adds to your allow-list
- **Deny** — the agent sees the failure and adapts

CLI flags for automation:
- `--auto-approve` — say yes to everything (cron-safe but powerful)
- `--never-approve` — read-only mode for safe exploration

## Auto-screenshot on screen-related tasks

When your task mentions:
*screen, screenshot, window, UI, button, click, press, tap, open...*

Horizon automatically captures the screen **before** the first turn
and feeds it to the AI as a vision input. Saves 3–5 seconds because
the model doesn't have to call the `screenshot` tool separately.

You can disable this in DevTools console: `window.HORIZON_AUTO_SCREENSHOT = false`.

## The agent loop

For each turn, the agent runs this loop (max 8 iterations by default):

1. **Plan** — list the steps it thinks will solve your task
2. **Pick a tool** — choose the next concrete action
3. **Execute** — run the tool (with permission gate)
4. **Observe** — read the result
5. **Reflect** — did this work? Goal met / partial / not met?
6. **Loop or finalise** — keep going if not done, or answer

Every step is visible in the **Inspector panel** (desktop) or the
**step rail** (CLI/TUI). Tool calls show as `→ tool_name(args)`,
results as `✓` or `✗` with a snippet of the output.

## Smart subagents

The agent can spawn sub-agents for parallel work (`spawn_subagent`
tool). Each subagent runs its own loop with isolated memory + history,
returns a compact summary to the parent. Useful for:

- "Search 3 different keywords in parallel"
- "Open 3 files and pick the most relevant one"
- "Try 3 hypotheses and report which worked"

Subagent depth is capped at 2 levels deep (no infinite recursion).
Concurrent subagents capped at 4.

## Cost control

Agent runs typically use 3–5× more tokens than a single chat turn.
Track spend with:

```bash
horizon cost                 # last 30 days
horizon cost --days 1        # today
horizon cost --provider claude
```

For routine tasks, use `--provider auto` — Horizon picks the cheapest
available provider that has a key (Gemini/Groq free tiers first).

## When NOT to use agent mode

- For pure brainstorming or summarisation — use Chat mode, it's 5× cheaper
- When you don't want a permission prompt every few seconds — Chat doesn't gate
- For very short questions ("what is X?") — Agent overkill, Chat returns faster

## Next

- [Computer use details](computer-use.md)
- [Voice + wake word for hands-free agent runs](voice.md)
- [Workspace memory — teach the agent about your project](workspace-memory.md)
