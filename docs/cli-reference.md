# CLI command reference

50+ subcommands across 8 groups. Run `horizon help` for the compact
4-group default, or `horizon help --all` for the full surface.

> **Looking for the complete list?** See the [full CLI commands page](reference/cli-commands.md)
> — every subcommand grouped by category, including Sprint 7 additions
> (`macro`, `ocr`, `find`, `agents board`, `theme`, `mobile`, `skill import`).

## Core

| Command | What it does |
|---|---|
| `horizon` | Launches the TUI |
| `horizon "task"` | Shorthand for `horizon agent "task"` |
| `horizon setup` | First-time interactive wizard |
| `horizon agent "task"` | Full agent loop with tool use + reflection |
| `horizon chat "msg"` | Single-turn chat (streaming, markdown) |
| `horizon version` | Print version + key health summary |
| `horizon help` | Inline help |

## Memory & skills

| Command | What it does |
|---|---|
| `horizon mem search "q"` | Semantic + FTS memory search |
| `horizon mem dump [--type X]` | Export NDJSON |
| `horizon mem profile` | Print user profile (Big Five etc.) |
| `horizon mem forget --memory/--fact <id>` | Delete one item |
| `horizon mem stats` | Counts + embedding state |
| `horizon skill list [--scope]` | List skills (workspace/user/builtin) |
| `horizon skill show <id>` | Print SKILL.md |
| `horizon skill new <id>` | Scaffold a new SKILL.md |
| `horizon skill enable/disable <id>` | Toggle |
| `horizon skill import <url\|path>` | Import skill from agentskills.io / ClawHub / GitHub / local path with security scan |

## Models & personas

| Command | What it does |
|---|---|
| `horizon model [provider]` | Read / set active provider |
| `horizon model --list` | All 25 providers + key health |
| `horizon persona [id]` | Read / set persona |
| `horizon persona --list` | All 5 built-in personas |
| `horizon cost [--days N]` | Token + dollar spend with bar chart |

## Connections

| Command | What it does |
|---|---|
| `horizon connect list` | All connections + status |
| `horizon connect test <id>` | Ping a connection |
| `horizon connect telegram --token <bot>` | Save Telegram bot token |
| `horizon connect discord --token <bot>` | Save Discord bot token |
| `horizon connect slack --token xoxb-...` | Save Slack token |
| `horizon connect notion --token secret_...` | Save Notion |
| `horizon connect linear --token lin_api_...` | Save Linear |
| `horizon connect github --token ghp_...` | Save GitHub |
| `horizon connect whatsapp --twilio-sid X --twilio-token Y --from whatsapp:+N` | WhatsApp via Twilio |
| `horizon connect signal --url <bridge> --number +N` | Signal via signal-cli bridge |
| `horizon connect imessage` | macOS-only iMessage enable |
| `horizon connect ssh --host user@h [--port N] [--key path]` | SSH executor |
| `horizon connect modal --token-id X --token-secret Y` | Modal serverless executor |
| `horizon connect daytona --server URL --key X --workspace ID` | Daytona workspaces |

## AI helpers

| Command | What it does |
|---|---|
| `horizon ask "..."` | Quick one-liner (terse, plain output) |
| `horizon explain <file\|->` | Explain code or text |
| `horizon summarize <file\|->` | Bullet summary |
| `horizon translate <file\|-> --to <lang>` | Translate (default Russian) |
| `horizon review <file>` | Code review (bugs/security/maintenance) |
| `horizon refactor <file> [--goal X]` | Suggest refactoring |
| `horizon test <file>` | Generate unit-test scaffold |
| `horizon diff <file\|->` | Explain a unified diff |
| `horizon search "query"` | Semantic memory search |
| `horizon brief` | Morning briefing |

## Dev helpers

| Command | What it does |
|---|---|
| `horizon git commit` | AI-generated commit message from staged diff |
| `horizon git review [target]` | Code review on a commit/file |
| `horizon git log [query]` | Semantic search through git log |
| `horizon git blame <file>` | Author contribution percentages |
| `horizon shell "what to do"` | Shell command suggestion (`--run` to execute) |
| `horizon web "query"` | Web search (Tavily or Perplexity) |
| `horizon image "prompt"` | Image generation (DALL-E / Imagen) |
| `horizon screen capture/describe` | Screenshot or AI-described screenshot |
| `horizon explain-error <log\|->` | Explain a stack trace |

## Management

| Command | What it does |
|---|---|
| `horizon doctor [--fix]` | Health check + auto-fix |
| `horizon profile list/use/create/show/delete/rename` | Multi-profile (work/personal) |
| `horizon completion bash\|zsh\|fish\|pwsh` | Shell autocomplete script |
| `horizon update [--check]` | Self-update from GitHub Releases |
| `horizon mcp list/add/remove/enable/disable/tools` | MCP server config |
| `horizon plugins list/show/enable/disable` | Installed plugin inspection |
| `horizon rules show/edit/add "rule"` | `.horizon/rules.md` editor |
| `horizon ws show/init/path/memory show\|edit` | Workspace `.horizon/` |

## Server / scheduling

| Command | What it does |
|---|---|
| `horizon serve [--port N] [--token X]` | Headless HTTP API |
| `horizon serve --enable-tg/--enable-discord/--enable-signal/--enable-whatsapp` | Plus live channel bots |
| `horizon cron create "<expr>" "<task>"` | Schedule recurring agent tasks |
| `horizon cron list/show/run/pause/resume/remove` | Cron management |
| `horizon cron daemon` | Run the scheduler in foreground |
| `horizon sessions list/new/show/export/delete` | Multi-chat history |
| `horizon checkpoints save/list/restore` | Memory savepoints |
| `horizon backup snapshot/list/restore/prune` | Full userData snapshots |
| `horizon hooks list/add/test/remove` | Life-cycle shell hooks |
| `horizon agents board` | Kanban board (queued / running / done) for spawned subagents |
| `horizon agents list/show/cancel/purge/stats/reclaim` | Manage the durable SQLite queue |

## Computer use (Sprint 7D)

| Command | What it does |
|---|---|
| `horizon ocr <imagepath>` | Run Tesseract.js OCR on an image, print extracted text |
| `horizon ocr screen` | OCR the active display |
| `horizon find "<text>"` | OCR-find the first match on screen, print `(x,y) wxh conf` |
| `horizon macro record <name>` | Record a sequence of mouse/keyboard events |
| `horizon macro play <name> [--speed N] [--repeat N]` | Replay a saved macro |
| `horizon macro list/show/delete` | Manage saved macros |
| `horizon screen capture [--display N]` | Screenshot a specific display (multi-monitor) |
| `horizon screen describe` | Screenshot + AI description |

## Phone pairing

| Command | What it does |
|---|---|
| `horizon mobile [--port N] [--host X]` | Spawns `horizon serve`, renders a QR code in the terminal — scan it with your phone camera to open the PWA with the pairing token pre-filled |

## Theme

| Command | What it does |
|---|---|
| `horizon theme [name]` | Switch CLI/TUI theme — `default`, `mono`, `light`, `kawaii`, `matrix`, `vapor`, `mocha`, or `cyber` |
| `horizon theme --list` | Show all 8 built-in themes with previews |

## Utilities

| Command | What it does |
|---|---|
| `horizon notes list/add/show/rm` | Quick notes |
| `horizon todo list/add/done/rm/clear-done` | TODO with completion tracking |
| `horizon timer <minutes>` | Pomodoro with progress bar + bell |
| `horizon stats [--days N]` | Memory + skills + cost overview |
| `horizon clip [show\|len]` | Read clipboard, analyse with AI |
| `horizon env list/set/unset` | Per-install env vars |
| `horizon open <url\|path>` | OS-native opener |
| `horizon logs [type] [--tail N]` | Typed log views |
| `horizon insights [--days N]` | Usage analytics (top models, hour heatmap) |

## Common flags

Apply to most commands:

| Flag | Effect |
|---|---|
| `--json` | Machine-readable JSON output |
| `--human` | Pretty human output |
| `--quiet` | Reply only, no metadata |
| `--plain` | Disable markdown |
| `--provider X \| auto` | Override AI provider for this call |
| `--model X` | Override model |
| `--persona X` | Override persona |
| `--workspace path` | Override `.horizon/` lookup root |
| `--profile X` | Use profile <X> for this call |
| `--max-steps N` | Cap agent loop iterations (default 8) |
| `--no-reflect` | Skip reflection epilogue |
| `--auto-approve` | Approve every tool call |
| `--never-approve` | Reject every tool call (read-only) |
| `--verbose` | Boot diagnostics to stderr |

## Examples

```bash
# Routine
horizon ask "what's the capital of Lithuania?"
horizon chat "explain quantum entanglement" --plain

# Code work
horizon review src/auth.js
horizon refactor src/old-api.js --goal "split into smaller functions"
horizon test src/parser.js
horizon git commit                    # AI commit message
horizon explain-error <(npm test 2>&1)

# Search + memory
horizon search "yerba mate"
horizon mem profile                   # what does Horizon think it knows about me?

# Scheduling
horizon cron create "0 9 * * 1-5" "summarise overnight commits to main"
horizon cron daemon                   # run forever

# Server / mobile
horizon serve --port 18789 --token mysecret --enable-tg --enable-whatsapp

# Smart routing
horizon agent "find security issues in src/" --provider auto --auto-approve
```
