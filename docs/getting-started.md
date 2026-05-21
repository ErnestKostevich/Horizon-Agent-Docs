# First-time setup

Whether you opened the desktop app or ran `horizon setup` in the
terminal, the steps are the same. The CLI also auto-launches the
wizard the first time you run `horizon` with no provider keys
configured.

## Step 1 — Pick a provider

Horizon supports 25 direct providers + 300+ models through LiteLLM.
For your first day, recommend one of the **free-tier** options:

| Provider | Free tier | Sign up |
|---|---|---|
| **Google Gemini** | 1,500 req/day | [aistudio.google.com](https://aistudio.google.com/apikey) |
| **Groq** | 30 req/min, fast | [console.groq.com](https://console.groq.com/keys) |
| **Cerebras** | Generous free, fastest inference on the planet | [cloud.cerebras.ai](https://cloud.cerebras.ai) |
| **Ollama (local)** | 100% free, runs on your machine | [ollama.com](https://ollama.com) |

If you already have a Claude or OpenAI key, those work too — you just
pay per-token to the provider directly.

## Step 2 — Paste the key

1. Sign up at the provider's site
2. Create an API key (one click in their console)
3. Copy and paste into Horizon's setup wizard
4. Keys are encrypted on disk with a machine-bound AES-256 key

Your key **never leaves your machine**. Horizon talks to the provider
directly; there's no Horizon server in between.

## Step 3 — Pick a persona

Five built-in personalities:

| Persona | Style |
|---|---|
| **Jarvis** | Formal, witty, says "Sir" — JARVIS from Iron Man |
| **Friday** | Casual, energetic, modern slang |
| **Alfred** | Calm, dignified, like a butler |
| **Sage** | Academic, precise, methodical |
| **Pixel** | Playful, expressive, lots of emoji |

Each persona has its own system prompt, memory namespace, voice preset,
and wake-word response. Read [Personas](concepts/personas.md) for the
full breakdown, including custom personas.

## Step 4 — Pick a language

English or Russian. Horizon's system prompt + UI adapt to your choice.
Other languages work for chat content — the agent replies in whatever
language you write in.

## Step 5 — Your name (optional)

For personalisation. The agent uses it in replies when natural ("Good
morning, Alex").

## You're done

Try one of these:

```bash
# Quick chat
horizon chat "what's the weather in Vilnius?"

# Full agent loop with tool use
horizon "find every TODO in this project and group by file"
horizon agent "draft a PR description from the last 3 commits" --auto-approve

# Interactive TUI
horizon

# Switch themes mid-session
horizon theme kawaii
horizon theme matrix

# Pair your phone (QR code + local server, one command)
horizon mobile

# OCR text out of any window
horizon screen capture --out cap.png
horizon ocr cap.png

# Find a button on screen and click it
horizon find "Submit"
```

Or in the desktop app — just type into the chat composer.

## Where to go next

- [Choosing a model provider](providers.md) — full price + capability table
- [Agent mode](agent-mode.md) — when to use Chat vs Agent
- [Computer use](guides/computer-use.md) — screen, mouse, keyboard
- [Computer use advanced](guides/computer-use-advanced.md) — OCR + macros + multi-display
- [Voice & wake word](guides/voice-and-wake-word.md) — say "Horizon" to start
- [Durable agents](guides/durable-agents.md) — Kanban board + subagents
- [Full CLI commands](reference/cli-commands.md)

## Switching providers later

```bash
horizon model claude --model claude-sonnet-4-6
horizon model --list
```

Or in the desktop: Settings → Models → pick a different provider. You
can also use `--provider auto` per-call to let Horizon pick the
cheapest available provider on each request.
