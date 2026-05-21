---
title: Computer use — advanced
sidebar_position: 25
description: Beyond click-and-type — Tesseract.js OCR for fast offline text recognition, multi-display screenshots and clicks, and macro record + playback.
lastUpdated: 2026-05-21
---

# Computer use — advanced

The [basic computer use guide](computer-use.md) covers Horizon's six
core primitives (screenshot, click, type, scroll, press, drag).
Sprint 7D added three deeper capabilities that turn the agent from a
"vision LLM with a mouse" into something that scales to real
workflows: **OCR**, **multi-display**, and **macros**.

This page walks each one — what it does, when to use it, and the CLI
commands you can call directly.

## OCR — read text out of any window

The naive "smart click" flow calls a vision LLM on every screenshot
to find a button. That's slow (1–3 seconds per turn) and costs
tokens. OCR replaces the vision call with a local
[Tesseract.js](https://tesseract.projectnaptha.com/) worker that
runs offline, returns word-level bounding boxes, and costs zero API
spend.

Tesseract.js is an **optional dependency** (~50 MB). On first use,
Horizon checks if it's installed and gracefully falls back to vision
LLM if not. Install it with:

```bash
npm install tesseract.js
# or, if you installed via brew/scoop, the package manager already pulled it
```

The worker is **lazy-loaded** — it doesn't initialise until the
first OCR call, so it doesn't slow down startup if you never use it.
Once loaded, the worker is cached for the lifetime of the process,
so subsequent calls are ~200 ms instead of ~1 s.

### CLI commands

```bash
# OCR an image file
horizon ocr screenshot.png

# OCR the live screen (Electron / CLI both supported)
horizon ocr screen

# Find a word on screen, get coordinates back
horizon find "Submit"
# → Submit at (1240, 860) 80x32 (conf 94)

# JSON output for scripts
horizon find "Save" --json
# → { "ok": true, "match": { "x": 1240, "y": 860, "w": 80, "h": 32, "text": "Save", "confidence": 94 } }
```

The agent uses the same module internally. When you ask "click the
Submit button", it first runs OCR, finds the bounding box, and
clicks the centre — no vision LLM call required.

### When to use OCR vs vision

| Use OCR when | Use vision LLM when |
|---|---|
| Looking for known text ("Submit", "OK") | Identifying a non-text element (an icon, a button without a label) |
| You need many lookups in a row | You need a holistic description of the screen |
| You want zero ongoing API cost | One-off semantic reasoning ("which window is the chat?") |
| The text might be in any of many places | The layout matters more than the words |

## Multi-display

Plug in a second monitor and the agent should still know what's on
which screen. The `multiDisplay` module wraps Electron's
`screen.getAllDisplays()` and `desktopCapturer`, exposes display
records with coordinates and scale factors, and routes screenshot
and click ops to the right physical screen.

```bash
# List all attached displays
horizon screen capture --list-displays

# Screenshot a specific display
horizon screen capture --display 1 --out left-monitor.png
horizon screen capture --display 2 --out right-monitor.png

# Agent tools automatically pick the right display from per-call hints
horizon "what's on my left monitor"
horizon "click the start menu on display 1"
```

Display IDs come from Electron — they're stable across screenshots
in the same session but can change when you reconnect a monitor.
Run `--list-displays` again after rearranging.

**Linux caveat:** `xdotool` (the default click backend on most
distros) doesn't support per-display click routing. Horizon
translates display-relative coords to global desktop coords and
sends those to `xdotool`, which works on most setups because the
global coordinate space spans all monitors.

## Macros — record once, replay forever

A macro captures a sequence of mouse and keyboard events, saves it
to disk as a small JSON file, and replays it on demand. Useful for:

- Repetitive UI tasks that don't justify writing a script (open a
  specific app, navigate to a specific tab, paste content)
- Demo workflows for screen recordings
- Setting up a known-good state before each test run

### Recording

```bash
# Start recording — perform actions, then Ctrl+C
horizon macro record open-vscode-and-search

# List saved macros
horizon macro list
# → open-vscode-and-search   12 events   4.2s   last: never

# Inspect a macro's event timeline
horizon macro show open-vscode-and-search
```

### Playback

```bash
# Default 1x speed, 1 repeat
horizon macro play open-vscode-and-search

# Faster, repeated 3 times
horizon macro play open-vscode-and-search --speed 2.0 --repeat 3

# Dry-run — print events without firing native actions
horizon macro play open-vscode-and-search --dry-run
```

### Macro file format

Saved at `<userData>/macros/<name>.json`:

```json
{
  "name": "open-vscode-and-search",
  "version": "1.0",
  "createdAt": 1716239900000,
  "duration": 4200,
  "events": [
    { "t": 0,    "type": "mouse_move",  "x": 100, "y": 200 },
    { "t": 250,  "type": "mouse_click", "x": 100, "y": 200, "button": "left" },
    { "t": 800,  "type": "key",         "key": "Tab" },
    { "t": 1100, "type": "type",        "text": "Hello world" },
    { "t": 4200, "type": "end" }
  ]
}
```

Macros are portable across machines but tied to screen resolution
— a macro recorded at 1920×1080 won't replay correctly on a 4K
display unless the UI scales identically. For machine-portable
workflows, prefer the OCR + `find` approach: it adapts to layout
shifts.

### Recording limitations

- **Best-effort capture.** Horizon uses a native global-hook module
  when available, otherwise falls back to polling cursor position at
  50 ms intervals (mouse-only, no click detection).
- **macOS:** recording is marked experimental. Playback is fully
  supported.
- **Sensitive input:** macros do not capture passwords or other
  protected text. The keyboard hook respects OS-level secure input
  flags.

## Putting it together

The three capabilities compose. A robust automation looks like:

```bash
# 1. Snap to a known-good state
horizon macro play reset-to-blank-doc

# 2. Use OCR to find the target
horizon find "New Document"

# 3. Let the agent take over from there
horizon "type the meeting notes into the new document"
```

## Where to go next

- [Computer use basics](computer-use.md) — the six core primitives
- [Voice & wake word](voice-and-wake-word.md) — hands-free agent runs
- [Durable agents](durable-agents.md) — long-running automations on the Kanban board
- [Full CLI commands](../reference/cli-commands.md)
