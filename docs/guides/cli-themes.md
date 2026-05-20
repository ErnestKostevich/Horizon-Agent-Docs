---
title: CLI themes
description: Eight built-in colour schemes for the Horizon CLI and TUI — switch with one command.
lastUpdated: 2026-05-20
---

# CLI themes

The Horizon CLI and TUI ship with eight named themes — each is a small
palette (accent / success / warn / err / dim) plus a spinner-frame set
and a banner glyph. Pick one to match your terminal background, your
mood, or the rest of your dotfiles.

Themes affect the CLI (`horizon ask`, `horizon chat`, `horizon agent`)
and the TUI (`horizon` with no args). The desktop GUI has its own
theming and isn't affected by this setting.

> **Truecolor required.** Themes use 24-bit RGB. If your terminal only
> supports 256-colour mode you'll get a sensible ANSI fallback, but the
> palette won't be exact. Modern terminals (iTerm2, Windows Terminal,
> Alacritty, Kitty, WezTerm) all support truecolor by default.

## Listing & switching

```bash
# Show the current theme
horizon theme

# List all built-in themes
horizon theme --list

# Switch
horizon theme matrix

# Reset to default
horizon theme default
```

Setting is persistent — written to `settingsStore.cliTheme`. Takes
effect on the next CLI invocation; no daemon restart needed.

You can also set it via environment variable for one-off testing:

```bash
HORIZON_CLI_THEME=kawaii horizon ask "what's 12 factorial?"
```

## The eight themes

### `default` — deep blue-violet

The signature look. Accent `#7c6df2`, smooth Braille spinner, ⌁ glyph.
Designed to feel "between Linear and Vercel" — saturated but
work-appropriate.

```
⌁ Horizon
✓ Connected to gemini · 1,500 free req/day
⠹ Thinking…
```

![default theme](./img/cli-theme-default.png)

### `mono` — pure monochrome

Greys only. No colour. ASCII spinner (`|/-\`). For when you want the
output to look exactly the same on any terminal, anywhere.

```
· Horizon
✓ Connected to gemini · 1,500 free req/day
| Thinking…
```

Best paired with logging pipelines or muxed terminals where colours
fight other tools' output.

### `light` — for light terminals

Dark accents (deep violet `#581c87`) on the assumption your background
is light. The reverse of `default`'s contrast.

```
⌁ Horizon
✓ Connected to gemini
⠹ Thinking…
```

### `kawaii` — pink kaomoji

Hot pink everywhere, and the spinner is *not* a Braille graphic — it's
a rotating set of kaomoji faces:

```
✿ Horizon
✓ Connected to gemini · 1,500 free req/day
(◕ᴗ◕) Thinking…
(◕ᴗ◕✿) Thinking…
(✿◕ᴗ◕) Thinking…
(◕‿◕✿) Thinking…
```

There's a Hermes-style face rotation that cycles every ~2.5 s between
runs:

```
(◕ᴗ◕) → (◔‿◔) → (◕▿◕) → (◠‿◠) → (◡‿◡) → (♥ᴗ♥) → ...
```

> Originally added as a joke for a Twitter demo. Stayed because more
> people use it than expected.

### `matrix` — green on black

Bright `#00ff00` accent, eight-frame Braille spinner that resembles
code rain.

```
▮ Horizon
✓ Connected to gemini
⣾ Thinking…
⣽ Thinking…
⣻ Thinking…
```

![matrix theme](./img/cli-theme-matrix.png)

Best on a black background with a green-leaning system font (Hack,
JetBrains Mono).

### `retro-amber` — 80s CRT

`#ffb000` amber, half-circle spinner (`◐◓◑◒`), ▰ banner. Pairs nicely
with a terminal background of `#0a0500` and a font like IBM 3270.

```
▰ Horizon
✓ Connected to gemini
◐ Thinking…
◓ Thinking…
```

### `vapor` — synthwave

Hot pink + neon cyan + soft purple dim. Curvy spinner (`◜◠◝◞◡◟`),
◈ banner.

```
◈ Horizon
✓ Connected to gemini
◜ Thinking…
◠ Thinking…
```

If you have one of those retro-grid wallpapers, this is the one.

### `mocha` — Catppuccin

The Catppuccin Mocha palette — warm mauve accent (`#cba6f7`), green
success, soft yellow warn, rose red error. Spinner is the half-circle
set.

```
❀ Horizon
✓ Connected to gemini
◐ Thinking…
```

For users who run Catppuccin in their editor and want the CLI to
match.

## Where themes live

If you want to inspect or extend, the theme module is at:

```
bin/lib/themes.js
```

Each theme is a JavaScript object:

```js
default: {
  accent:  [124, 109, 242],
  success: [56, 211, 159],
  warn:    [255, 181, 102],
  err:     [255, 117, 117],
  dim:     [128, 128, 128],
  cyan:    [56, 189, 248],
  magenta: [217, 70, 239],
  spinnerFrames: ['⠋','⠙','⠹','⠸','⠼','⠴','⠦','⠧','⠇','⠏'],
  banner: '⌁',
  description: 'Deep blue-violet — between Linear and Vercel.',
}
```

The `kawaii` theme additionally carries a `kawaiiFaces` array used by
`GradientSpinner` for the rotating-face behaviour.

## Custom themes (advanced)

Adding a custom theme means editing `bin/lib/themes.js` and
re-bundling — there's no overlay store the way personas have. If
you'd like that, file an issue at
[github.com/ErnestKostevich/horizon-genesis/issues](https://github.com/ErnestKostevich/horizon-genesis/issues).

Per-run colour overrides are also available without modifying the
theme:

```bash
NO_COLOR=1 horizon ask "..."  # disable all colour
FORCE_COLOR=1 horizon ask "..."  # force colour even when piped
```

These standard env vars are honoured before the theme palette is read.

## Banner glyphs

Each theme has a small "banner glyph" that prefixes the Horizon
wordmark on boot:

| Theme | Glyph |
|---|---|
| default | ⌁ |
| mono | · |
| light | ⌁ |
| kawaii | ✿ |
| matrix | ▮ |
| retro-amber | ▰ |
| vapor | ◈ |
| mocha | ❀ |

A glyph that doesn't render in your terminal is a font issue, not a
theme issue. Make sure the font has decent Unicode coverage (any
Nerd Font, JetBrains Mono, Cascadia Code Nerd, Fira Code Nerd).

## Status-bar context windows

A side effect of theming: the TUI's status bar reads `ctxFor(model)`
from the same module to render a context-window meter for the active
model. Drop into the TUI and you'll see e.g.

```
gemini-2.5-flash · 1.0M ctx · 12k / 1.0M used (1%)
```

Switching theme doesn't change the bar — it's the same data — but the
colour pulse on a near-limit warning uses the theme's `warn` colour.

## Troubleshooting

| Symptom | Fix |
|---|---|
| Colours look washed out | Terminal isn't truecolor. Check `echo $COLORTERM` — needs to be `truecolor` |
| Spinner shows `?` boxes | Font missing Unicode glyphs — switch to a Nerd Font |
| Theme doesn't persist | `horizon theme <name>` writes settingsStore; if read-only filesystem (Docker), pass via `HORIZON_CLI_THEME=` |
| `default` looks dark on a light terminal | Use `horizon theme light` instead |
| Background colour wrong | Themes set foreground only — your terminal's background config is independent |

## Related

- [CLI reference](../cli-reference.md) — `horizon theme` command
- [Installation](../installation.md) — getting the CLI on your `$PATH`
