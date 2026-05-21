# What is Horizon?

Horizon is a personal AI agent that runs **on your machine**. Not in
the cloud, not on someone else's server — on your computer, with your
files, with your API keys.

You bring the model (Claude, GPT, Gemini, Groq, or 21 other providers
— or fully local with Ollama). Horizon brings everything around it:

- A polished **desktop app** with chat, voice, and computer use
- A **terminal CLI + TUI** with 50+ subcommands for power users
- A **headless HTTP API** for cron jobs and remote machines
- A **mobile PWA** companion that pairs with your desktop (`horizon mobile`)
- **9 layers of memory** — JSON file + SQLite + FTS5 + 256-dim embeddings hybrid — so it actually learns who you are
- A **plugin marketplace** with crypto payouts for authors
- **Real computer use** — see your screen, click, type, take screenshots, OCR text out of any window
- **Macro record + playback** — capture a sequence once, replay it forever
- **8 built-in CLI themes** — switch palettes with `horizon theme kawaii`
- **38+ built-in skills** that the agent auto-imports when relevant
- **25 AI providers** + 300+ models via LiteLLM router

## Why not just use ChatGPT or Claude.ai?

| | Horizon | ChatGPT / Claude.ai |
|---|---|---|
| Where it runs | Your machine | Their cloud |
| Your API keys | Local, encrypted | Sent to them |
| Conversation history | Local file you own | Their database |
| Computer use | First-class (+ OCR + macros + multi-display) | Limited / none |
| Voice + wake word | Built-in | Mobile-only |
| Plugins | Marketplace + crypto payouts to authors | First-party only |
| Personas | 5 built-in + custom | One style |
| Cost tracking | Per-call, free routing | None visible |
| Cron / scheduling | Yes — `horizon cron` | No |
| Durable agent queue | SQLite Kanban board, survives reboots | No |
| Messaging channels | TG / Discord / Slack / WhatsApp / Signal / iMessage / Notion / Linear / GitHub | None |

## Why bring your own key?

Three reasons:

1. **You stay in control.** Your keys never leave your machine; nobody
   else can read your conversations.
2. **You pay actual cost.** No "credits" markup, no monthly cap. Some
   providers (Gemini, Groq, Cerebras) have generous free tiers.
3. **Provider freedom.** Don't like Claude this week? Switch to GPT.
   Want Llama? Use Groq. Want zero API spend? Run local Ollama.

## What's in the box

- **Desktop app** (Electron, Windows / macOS / Linux installers)
- **CLI binary** — single executable, no Node required
- **Mobile PWA** — pairs with `horizon serve` over Wi-Fi or LAN
- **Plugin SDK** — TypeScript types + `@horizonai/plugin-cli` for authors

See [installation](installation.md) for the matrix of install options.

## License & pricing

Horizon ships under the **Business Source License 1.1** — source-available,
free for personal evaluation and internal use, with a paid commercial
tier and a 5-day trial for new users. Commercial resale needs a written
licence agreement.

See [pricing](pricing.md) for the trial + subscription details.

## Built by

Horizon Genesis is built by [Ernest Kostevich](https://github.com/ErnestKostevich).

Bring your own key. Bring your own ethics. Keep your data.
