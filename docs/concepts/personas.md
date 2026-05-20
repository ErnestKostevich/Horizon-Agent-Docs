---
title: Personas
description: Five built-in AI personalities, per-persona memory, and how to write your own.
lastUpdated: 2026-05-20
---

# Personas

A persona is the voice Horizon speaks in. It changes the system prompt, the
wake-word response, the colour accent in the GUI, and which memory slice the
agent reads from. Five are built in; you can edit any of them or create new
ones from scratch.

The active persona is one setting — `persona` in `settingsStore` — and every
chat, agent run, voice reply, and channel reply uses it. Switching is
instant, no restart.

## The five built-in personas

| id | Name | Icon | Tone | Default greeting (EN) | Default greeting (RU) |
|---|---|---|---|---|---|
| `jarvis` | J.A.R.V.I.S. | 🤖 | Professional, efficient, "Sir" | At your service, Sir. All systems nominal. | К вашим услугам, Сэр. Системы в норме. |
| `friday` | F.R.I.D.A.Y. | 💙 | Friendly, warm, supportive | Hey there! I'm here, how can I help? | Привет! Я тут, чем могу помочь? |
| `alfred` | Alfred | 🎩 | Refined English butler, sophisticated | Good day. How may I be of assistance? | Добрый день. Чем могу быть полезен? |
| `sage` | Sage | 🧙 | Philosophical, thoughtful, metaphorical | Greetings, traveler. What weighs upon your mind? | Приветствую, путник. Какой вопрос тревожит твой разум? |
| `pixel` | Pixel | ✨ | Creative, Gen-Z energy, slang | Yoo! Pixel in the house! Let's make something awesome! | Йоу! Pixel в деле! Го, давай что-нибудь крутое! |

`jarvis` is the default for a fresh install.

## What a persona actually changes

When you switch persona, five things change at the same time:

1. **System prompt** — each persona has a bilingual (EN/RU) prompt that
   describes the role and tone. The active language follows the user's recent
   message language (Cyrillic in the last 5 user turns → Russian; otherwise
   English).
2. **Wake-word response** — what Horizon says when it hears its name (see
   [Voice and wake word](../guides/voice-and-wake-word.md)). Each persona has
   four to five canned responses; one is picked at random.
3. **Colour accent** — the GUI uses the persona colour for the prompt arrow,
   wake-bar pulse, and inspector highlights:
   - `jarvis` `#6c8cff` (blue-violet)
   - `friday` `#34d399` (green)
   - `alfred` `#a78bfa` (purple)
   - `sage` `#fbbf24` (amber)
   - `pixel` `#f472b6` (pink)
4. **Memory namespace** — every memory and conversation is tagged with the
   persona id that wrote it. When the active persona is Alfred, recall is
   boosted (default 1.2x) for memories Alfred saw. The other personas' notes
   are still visible but lower-ranked.
5. **Allowed tools (optional)** — a persona can carry an `allowedTools` list
   that restricts what the agent may call while it's active. Default is
   `null` (all tools enabled).

## Switching persona

### GUI

Settings → Personas → click the card. The colour accent flips immediately and
the next reply uses the new voice.

### CLI

```bash
# Show current persona
horizon persona

# Switch
horizon persona alfred

# List all personas (built-in + custom)
horizon persona --list
```

### TUI slash command

While inside `horizon` (the TUI):

```
/persona sage
```

### Per-call override

Most CLI commands accept `--persona` to use a different persona for one call
only — the default is unchanged:

```bash
horizon chat "explain async/await" --persona sage
horizon agent "review src/auth.js"   --persona alfred
```

The HTTP API takes the same field on `POST /api/chat`, `/api/agent`, and
`/api/skill/:id/run` bodies. See [HTTP API](../reference/http-api.md).

## Per-persona memory

Each persona has its own memory layer on top of the shared store. The
overlay records:

- **Long-term notes** — short bullet facts that get injected into the system
  prompt every turn. The Inspector → Personas panel lets you add/remove
  them by hand.
- **Recall boost** — when the active persona is Pixel, the retrieval scorer
  multiplies the score of every memory tagged `personaId: 'pixel'` by 1.2.
  Memories from other personas are still candidates but rank lower.

This lets you keep distinct knowledge bases per voice — Alfred remembers
your dinner reservations, Sage remembers your reading list — without
maintaining separate accounts.

To dump the current persona's notes:

```bash
horizon persona show alfred
```

To clear them:

```bash
horizon persona reset alfred
```

`reset` on a built-in persona clears your overlay (custom notes, custom
prompt edits) and restores the factory defaults. `reset` on a custom
persona deletes it entirely.

## Editing a built-in

You can override the prompt, icon, colour, greeting, wake responses, or
notes on a built-in without losing the original. The next `reset` will
bring it back.

### From the GUI

Settings → Personas → click the card → Edit. Save writes a small overlay
to `settingsStore` under the `customPersonas` key.

### From a JSON patch (CLI)

```bash
horizon persona edit jarvis --file ./my-jarvis-edits.json
```

Where `my-jarvis-edits.json` is a partial — fields you omit stay as the
built-in default:

```json
{
  "prompt": {
    "en": "You are J.A.R.V.I.S. Speak in clipped, technical sentences. Never apologise.",
    "ru": "Ты J.A.R.V.I.S. Говори короткими техническими фразами. Никогда не извиняйся."
  },
  "color": "#00d4ff",
  "memories": [
    { "id": "m1", "text": "User prefers metric units.", "ts": 1737840000000 }
  ]
}
```

## Authoring a custom persona

Custom personas live in the same store as overlays — there's no separate
file path you copy into. To create one:

```bash
horizon persona create kane \
  --name "Kane" \
  --icon "⚔️" \
  --color "#ef4444" \
  --prompt-en "You are Kane, a no-nonsense ops engineer. Reply in bullet points. No filler." \
  --prompt-ru "Ты Кейн, инженер по эксплуатации. Отвечай списками. Никакой воды."
```

### Full custom persona schema

The overlay store accepts the following keys. Anything you omit gets a
sensible default.

```json
{
  "id": "kane",
  "name": "Kane",
  "icon": "⚔️",
  "color": "#ef4444",
  "builtin": false,
  "greeting": {
    "en": "Kane online. What's broken?",
    "ru": "Кейн на связи. Что сломалось?"
  },
  "prompt": {
    "en": "You are Kane, a no-nonsense ops engineer...",
    "ru": "Ты Кейн, инженер по эксплуатации..."
  },
  "wakeResponses": {
    "en": ["Kane.", "Listening.", "Go."],
    "ru": ["Кейн.", "Слушаю.", "Говори."]
  },
  "allowedTools": ["read_file", "list_dir", "shell_command", "run_code"],
  "memories": [
    { "id": "m1", "text": "User runs on Hetzner CPX21 boxes.", "ts": 1737840000000 }
  ]
}
```

### Tips for writing a good prompt

- Keep it under 200 words. The prompt prefixes every turn, so cost matters.
- Lead with role, then tone, then speech style, then concrete behaviour
  ("always suggest", "never apologise"). Mirror the built-ins' shape.
- Don't repeat capabilities ("you can search the web") — the agent already
  knows its tools. Repeating dilutes the personality direction.
- Use the same patterns in both languages. The persona should feel like the
  same person whether the user writes in English or Russian.

## File locations

Built-in persona definitions are baked into the binary. User overlays
(edits to built-ins + custom personas) live in `settingsStore` under the
`customPersonas` key:

- macOS: `~/Library/Application Support/horizon-genesis/config.json`
- Windows: `%APPDATA%\horizon-genesis\config.json`
- Linux: `~/.config/horizon-genesis/config.json`

You can hand-edit that file if you want — it's just JSON. Restart Horizon
to pick up changes.

## Related

- [Voice and wake word](../guides/voice-and-wake-word.md) — wake responses
  per persona
- [9-layer memory](./9-layer-memory.md) — how persona memory fits the
  retrieval stack
- [HTTP API](../reference/http-api.md) — `POST /api/persona`,
  `GET /api/personas`
