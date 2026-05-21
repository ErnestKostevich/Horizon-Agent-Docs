---
title: CLI commands — full reference
sidebar_position: 30
description: Every Horizon subcommand grouped by category — 50+ verbs from chat and agent loops to OCR, macros, mobile pairing, and the Kanban agent board.
lastUpdated: 2026-05-21
---

# CLI commands — full reference

The Horizon CLI ships **50+ subcommands** split into nine functional
groups. Run `horizon help` for the four-group compact view that
matches what most people use day-to-day, or `horizon help --all` for
the entire surface. This page lists every command grouped by purpose,
with the most common flags and a one-liner of intent.

> Every command honours the [common flags](#common-flags) at the
> bottom of the page — `--json`, `--provider`, `--persona`,
> `--workspace`, `--profile`, and friends.

## Chat & agent

The four entry points to "ask the agent something":

| Command | What it does |
|---|---|
| `horizon` | Launches the interactive TUI (auto-runs `setup` on first run if no keys are configured) |
| `horizon "task"` | Shorthand for `horizon agent "task"` |
| `horizon agent "task"` | Full agent loop — plan, act, reflect, retry. Caps at 8 steps by default. |
| `horizon chat "msg"` | Single-turn chat (streaming, markdown) |
| `horizon ask "..."` | Quick one-liner (terse, plain output, no metadata) |
| `horizon tui` | Explicitly launch the TUI |
| `horizon setup` | First-time interactive wizard |
| `horizon version` | Print version + provider key health summary |
| `horizon help [--all]` | Inline help |

## Skills & memory

The 9-layer memory and skills system:

| Command | What it does |
|---|---|
| `horizon skill list [--scope]` | List skills (workspace / user / builtin) |
| `horizon skill show <id>` | Print the SKILL.md content |
| `horizon skill new <id>` | Scaffold a new SKILL.md from a template |
| `horizon skill enable/disable <id>` | Toggle a skill |
| `horizon skill import <url\|path>` | Import skill from agentskills.io / ClawHub / GitHub / local path with security scan |
| `horizon mem search "q"` | Semantic + FTS memory search |
| `horizon mem dump [--type X]` | Export memory as NDJSON |
| `horizon mem profile` | Print user profile (Big Five + extracted facts) |
| `horizon mem forget --memory/--fact <id>` | Delete one memory item |
| `horizon mem stats` | Counts + embedding state |
| `horizon mem review` | Periodic memory review/cleanup pass |
| `horizon sessions list/new/show/export/delete` | Multi-chat history |
| `horizon checkpoints save/list/restore` | Memory savepoints |
| `horizon backup snapshot/list/restore/prune` | Full userData snapshots |
| `horizon dialectic` | Honcho dialectic diff log: summary / recent / search |

See [9-layer memory](../concepts/9-layer-memory.md) for the architecture.

## Models, personas, themes

| Command | What it does |
|---|---|
| `horizon model [provider]` | Read / set active provider |
| `horizon model --list` | All 25 providers + key health |
| `horizon persona [id]` | Read / set persona |
| `horizon persona --list` | All 5 built-in personas |
| `horizon theme [name]` | Switch CLI/TUI theme |
| `horizon theme --list` | Show all 8 built-in themes |
| `horizon cost [--days N]` | Token + dollar spend with a bar chart |

## Connectors & channels

| Command | What it does |
|---|---|
| `horizon connect list` | All connections + status |
| `horizon connect test <id>` | Ping a connection |
| `horizon connect telegram --token <bot>` | Save Telegram bot token |
| `horizon connect discord --token <bot>` | Save Discord bot token |
| `horizon connect slack --token xoxb-...` | Save Slack token |
| `horizon connect notion --token <secret>` | Save Notion integration token |
| `horizon connect linear --token <token>` | Save Linear API key |
| `horizon connect github --token <pat>` | Save GitHub personal access token |
| `horizon connect whatsapp --twilio-sid X --twilio-token Y --from whatsapp:+N` | WhatsApp via Twilio |
| `horizon connect signal --url <bridge> --number +N` | Signal via signal-cli bridge |
| `horizon connect imessage` | macOS-only iMessage enable |
| `horizon connect ssh --host user@h [--port N] [--key path]` | SSH executor |
| `horizon connect modal --token-id X --token-secret Y` | Modal serverless executor |
| `horizon connect daytona --server URL --key X --workspace ID` | Daytona workspaces |

> **Don't paste real tokens into shared docs / chat logs.** Use
> placeholders like `YOUR_TOKEN_HERE` and rotate any token you
> accidentally leak.

## Server, scheduling, durable agents

| Command | What it does |
|---|---|
| `horizon serve [--port N] [--token X]` | Headless HTTP API |
| `horizon serve --enable-tg/--enable-discord/--enable-whatsapp/--enable-signal/--enable-imessage` | Plus live channel bots |
| `horizon mobile` | Spawn `horizon serve` + render a QR code so your phone can scan and pair |
| `horizon cron create "<expr>" "<task>"` | Schedule recurring agent tasks |
| `horizon cron list/show/run/pause/resume/remove` | Cron management |
| `horizon cron daemon` | Run the scheduler in the foreground |
| `horizon agents board` | Kanban board view (queued / running / done) of spawned subagent tasks |
| `horizon agents list/show/cancel/purge/stats/reclaim` | Manage the durable SQLite queue |
| `horizon hooks list/add/test/remove` | Life-cycle shell hooks |

See [durable agents](../guides/durable-agents.md) for the Kanban runtime details.

## Computer use (vision + automation)

| Command | What it does |
|---|---|
| `horizon screen capture [--display N] [--out FILE]` | Screenshot a specific display |
| `horizon screen describe` | Screenshot + AI description |
| `horizon ocr <imagepath>` | Run Tesseract.js OCR on an image, print extracted text |
| `horizon ocr screen` | OCR the active display |
| `horizon find "<text>"` | OCR-find first match on screen — prints `(x,y) wxh conf` |
| `horizon macro record <name>` | Record a sequence of mouse/keyboard events |
| `horizon macro play <name> [--speed N] [--repeat N] [--dry-run]` | Replay a saved macro |
| `horizon macro list/show/delete` | Manage saved macros |

See [computer use](../guides/computer-use.md) for the basics and
[computer use advanced](../guides/computer-use-advanced.md) for OCR,
multi-display, and macros.

## AI helpers

| Command | What it does |
|---|---|
| `horizon explain <file\|->` | Explain code or text |
| `horizon summarize <file\|->` | Bullet summary |
| `horizon translate <file\|-> --to <lang>` | Translate (default Russian) |
| `horizon review <file>` | Code review (bugs / security / maintenance) |
| `horizon refactor <file> [--goal X]` | Suggest refactoring |
| `horizon test <file>` | Generate unit-test scaffold |
| `horizon diff <file\|->` | Explain a unified diff |
| `horizon search "query"` | Semantic memory search |
| `horizon brief` | Morning briefing |

## Dev helpers

| Command | What it does |
|---|---|
| `horizon git commit` | AI-generated commit message from staged diff |
| `horizon git review [target]` | Code review on a commit / file |
| `horizon git log [query]` | Semantic search through git log |
| `horizon git blame <file>` | Author contribution percentages |
| `horizon shell "what to do"` | Shell command suggestion (`--run` to execute) |
| `horizon web "query"` | Web search (Tavily or Perplexity) |
| `horizon image "prompt"` | Image generation (DALL-E / Imagen) |
| `horizon explain-error <log\|->` | Explain a stack trace |

## Management & utilities

| Command | What it does |
|---|---|
| `horizon doctor [--fix]` | Health check + auto-fix |
| `horizon profile list/use/create/show/delete/rename` | Multi-profile (work / personal) |
| `horizon completion bash\|zsh\|fish\|pwsh` | Shell autocomplete script |
| `horizon update [--check]` | Self-update from GitHub Releases |
| `horizon mcp list/add/remove/enable/disable/tools` | MCP server config |
| `horizon plugins list/show/enable/disable` | Installed plugin inspection |
| `horizon rules show/edit/add "rule"` | `.horizon/rules.md` editor |
| `horizon ws show/init/path/memory show\|edit` | Workspace `.horizon/` management |
| `horizon canvas show/append/replace` | Live Canvas surface |
| `horizon notes list/add/show/rm` | Quick notes |
| `horizon todo list/add/done/rm/clear-done` | TODO with completion tracking |
| `horizon timer <minutes>` | Pomodoro with progress bar + bell |
| `horizon stats [--days N]` | Memory + skills + cost overview |
| `horizon clip [show\|len]` | Read clipboard, analyse with AI |
| `horizon env list/set/unset` | Per-install env vars |
| `horizon open <url\|path>` | OS-native opener |
| `horizon logs [type] [--tail N]` | Typed log views |
| `horizon insights [--days N]` | Usage analytics (top models, hour heatmap) |
| `horizon status` | Compact runtime status |

## Common flags

These work on most commands:

| Flag | Effect |
|---|---|
| `--json` | Machine-readable JSON output |
| `--human` | Pretty human output (default) |
| `--quiet` | Reply only, no metadata |
| `--plain` | Disable markdown |
| `--provider X \| auto` | Override AI provider for this call |
| `--model X` | Override model |
| `--persona X` | Override persona |
| `--workspace path` | Override `.horizon/` lookup root |
| `--profile X` | Use profile `<X>` for this call |
| `--max-steps N` | Cap agent loop iterations (default 8) |
| `--no-reflect` | Skip reflection epilogue |
| `--auto-approve` | Approve every tool call (cron-friendly, dangerous) |
| `--never-approve` | Reject every tool call (read-only exploration) |
| `--verbose` | Boot diagnostics to stderr |

## Where to go next

- [CLI themes — switch palettes](../guides/cli-themes.md)
- [Computer use](../guides/computer-use.md) + [advanced](../guides/computer-use-advanced.md)
- [Durable agents — Kanban board](../guides/durable-agents.md)
- [HTTP API reference](http-api.md) — the same verbs over JSON-over-HTTP
- [Full providers reference](providers-full.md)
