---
title: Computer use
description: Vision-based screen control — screenshots, smart clicking, typing, and keyboard automation.
lastUpdated: 2026-05-20
---

# Computer use

Horizon can see your screen and operate your computer the way you do — by
looking, clicking, typing, and reading the result. Tell it "open Chrome and
search for the closest pharmacy" and it captures the screen, finds the right
icon, clicks it, types in the address bar, and reads back what loaded.

This is the same family of capability you'll see called "Computer Use" by
Anthropic or "Agent-S" in research papers. The implementation is local to
your machine — Horizon never streams your screen to a third party except
when it asks an LLM to interpret one frame.

## When Horizon triggers computer use

You don't switch a mode. The agent decides per turn based on what you asked:

- **Auto-screenshot triggers** — phrases like "what's on screen", "what do
  you see", "describe this window", "the button in the corner", "is X
  visible" cause Horizon to capture and attach a screenshot before
  answering.
- **UI control triggers** — "click", "open", "navigate to", "type", "press
  enter", "scroll down" produce one or more of the action tools below.
- **Workflows** — a multi-step request ("log in to GitHub and star the
  horizon-genesis repo") becomes a planned chain: screenshot → smart_click
  on login → type → press_key → screenshot → verify → smart_click on star.

You can always ask for an explicit capture with `horizon screen capture`
or `horizon screen describe`.

## The tools

Six primitives compose every workflow. The agent picks them itself; you
just describe the goal.

### `screenshot`

Grabs the active display as a PNG, base64-encodes it, and either returns
it to the agent for visual reasoning, saves it to a file, or both.

The CLI shortcut:

```bash
horizon screen capture --out ~/Desktop/cap.png
horizon screen describe         # captures + asks the agent to describe
```

### `smart_click`

Vision-based click. You give it a description; it asks the vision model
to find the centre coordinates of the matching UI element and clicks
there.

```js
// Internal tool call (agent emits this; you don't usually write it)
smart_click({ target: "the blue Send button in the bottom-right of the chat panel" })
```

The vision model returns `{ found, element, x, y, confidence, action }`.
If `confidence < 0.5`, the click is refused and the agent has to
re-screenshot or pick a different target. This guards against confident
mis-clicks on unrelated elements.

`smart_click` is preferred over raw `mouse_click(x, y)` for anything UI —
absolute coordinates break the moment the window moves or resolution
changes.

### `mouse.click(x, y)`

Raw click at a pixel. Use when the agent already knows exact coordinates
(e.g. from a previous screenshot it analysed) or when there's no visual
target to describe.

```js
mouse_click({ x: 540, y: 320, button: "left", double: false })
```

`button` accepts `left | right | middle`. `double: true` produces a
double-click.

### `keyboard.type`

Types literal text into whatever's focused. Supports unicode (Cyrillic,
emoji, etc.) on all platforms.

```js
type_text({ text: "hello world", enter: false })
```

Set `enter: true` to append a newline and submit the field — equivalent
to typing then pressing Enter.

### `keyboard.press`

A single keystroke or chord:

```js
press_key({ key: "Enter" })
press_key({ key: "Ctrl+L" })   // address bar in most browsers
press_key({ key: "Cmd+Tab" })  // macOS app switcher
```

Modifiers: `Ctrl`, `Alt`, `Shift`, `Cmd` (mapped to Win key on Windows
unless you set `Meta`).

### `scroll`

Scrolls the active window. `direction` is `up | down | left | right`,
`amount` is in mouse-wheel ticks (default 3).

```js
scroll({ direction: "down", amount: 5 })
```

### `move_window` (planned)

Move the foreground window to a different region of the desktop. Not yet
wired in the public agent tools but accessible via `run_code` with a
platform shell command (`xdotool`, `osascript`, or PowerShell).

## Permission gates

Every action that touches the keyboard, mouse, or file system goes through
a permission check before it runs. In the desktop GUI this is a small
popup:

```
Horizon wants to: click on "the Send button"
[Allow once]  [Allow for this session]  [Deny]
```

The three modes are persisted per-tool in `settingsStore`:

- **Allow once** — this single call. Next time you'll be asked again.
- **Allow for this session** — all calls of this tool until you close
  Horizon.
- **Allow always** — write a permanent allow into settings. Revoke from
  Settings → Permissions.

In server mode (`horizon serve`), permission gating is `auto-approve`
by default because there's no human to click — production deployments
should gate by token scope or wire push notifications (see
[HTTP API](../reference/http-api.md)).

To deny a tool globally:

```bash
horizon rules add "Never use smart_click or mouse_click without explicit confirmation in the same turn."
```

The agent re-reads `.horizon/rules.md` every turn and the LLM honours it.

## "AGENT IN CONTROL" overlay

While the agent is sending mouse or keyboard events, Horizon paints a thin
banner across the top of the screen:

```
◈ AGENT IN CONTROL — move mouse to take over
```

Moving the mouse pointer by more than ~30 px hard-cancels the active
agent turn. The banner exists for one reason: at any second you can grab
the mouse and the agent will stop. There's no "kill" button to find — you
just touch the input device.

The banner is configurable in Settings → Computer use → Overlay:

- Position: top / bottom / off (off only recommended on multi-monitor
  setups where the banner overlaps something important)
- Auto-cancel threshold (pixels): default 30
- Sound on activate / deactivate: off by default

## Safety boundaries

Horizon has three guard rails for computer use:

### Workspace boundary

Default workspace is the directory you launched from (`$PWD`). File-system
tools (`read_file`, `write_file`, `list_dir`, `search_files`) refuse paths
outside it unless you explicitly opt in:

```bash
horizon agent "summarise my desktop" --workspace ~/Desktop
```

Or per-call:

```bash
horizon "..." --allow-path /etc/hosts
```

### Deny-list

Some commands are denied even with full permission. `shell_command` only
runs a whitelisted set of read-only invocations (`ls`, `ipconfig`, `ping`,
`tasklist`, `systeminfo`, etc.). Anything that writes or executes goes
through `run_code`, which has its own per-call permission gate.

`rm -rf /`, `format`, `del /q`, `mkfs`, `dd`, and similar destroyers are
flat-out blocked regardless of permission state.

### No-input mode

For unattended runs (cron, server), launch with `--no-input`:

```bash
horizon serve --no-input  # never prompts for permission; tools that
                           # would have asked are skipped + logged
```

Combined with `--allow-tools read_file,recall,web_search,...` this lets
you run safe-by-construction batch jobs.

## Examples

### "Open Chrome and navigate to GitHub"

```bash
horizon "open Chrome and navigate to github.com/horizon-genesis"
```

Plan that the agent emits internally:

1. `screenshot` — what's currently on screen?
2. `press_key { key: "Cmd+Space" }` / `press_key { key: "Win" }` — open
   launcher
3. `type_text { text: "Chrome" }` and `press_key { key: "Enter" }`
4. `screenshot` — verify Chrome is now focused
5. `press_key { key: "Ctrl+L" }` — focus address bar
6. `type_text { text: "github.com/horizon-genesis", enter: true }`
7. `screenshot` — verify the page loaded; report back

You see each step in the GUI's "Inspector" pane while it's running.

### "Take a screenshot and describe what you see"

```bash
horizon screen describe
```

One-shot — capture, ask the vision model for a paragraph, print it. No
clicks or typing involved.

### "Star the horizon-genesis repo for me"

```bash
horizon agent "log into github.com and star horizon-genesis/horizon-genesis"
```

Assuming you're already logged in (Horizon won't ask for your password —
it would refuse if it had to), this resolves to:

1. `browser_open { url: "https://github.com/horizon-genesis/horizon-genesis" }`
2. `screenshot`
3. `smart_click { target: "the Star button at the top of the repo page" }`
4. `screenshot` — confirm it now says "Starred"

### "Reply to the top email"

```bash
horizon "open Gmail and tell me what's in the top email"
```

`smart_click` shines here because the email list reorders constantly —
absolute coordinates wouldn't survive five minutes. The vision model
finds "the first email row" regardless of its current y-position.

## Limitations

- **Vision latency** — every `smart_click` round trip is a vision model
  call (1–4 s typical). For fast clickfests, raw `mouse_click(x, y)` is
  ~50 ms per click.
- **Resolution scaling** — Retina / 4K displays sometimes confuse the
  vision model on small text. Bumping zoom (`Ctrl+=`) before agent
  control improves accuracy markedly.
- **OS dialogs** — UAC, sudo, Keychain, and similar password prompts are
  blocked by the OS for accessibility. Horizon will refuse to "type your
  password" and surface the prompt back to you.
- **Browsers in fullscreen** — fullscreen ChromeOS-style apps hide their
  chrome; `Ctrl+L` and similar shortcuts may not work. Exit fullscreen
  before agent control if you need address-bar navigation.

## Related

- [CLI reference](../cli-reference.md) — `horizon screen` family of
  commands
- [Agent mode](../agent-mode.md) — how the agent loop calls these tools
- [Voice and wake word](./voice-and-wake-word.md) — hands-free trigger
  for computer-use workflows
