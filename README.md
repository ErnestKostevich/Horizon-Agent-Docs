<p align="center">
  <img src="https://raw.githubusercontent.com/ErnestKostevich/horizon-genesis/main/assets/icon.png" width="120" alt="Horizon logo" />
</p>

<h1 align="center">Horizon AI Docs</h1>

<p align="center">
  <strong>The official user documentation for Horizon AI.</strong><br/>
  <sub>Desktop app · Terminal CLI · Mobile PWA · Plugin marketplace</sub>
</p>

<p align="center">
  <a href="https://horizonaai.dev"><img src="https://img.shields.io/badge/web-horizonaai.dev-ec4899?style=flat-square" alt="Website"/></a>
  <a href="https://horizonaai.dev/docs"><img src="https://img.shields.io/badge/hosted%20docs-online-06b6d4?style=flat-square" alt="Hosted docs"/></a>
  <a href="https://github.com/ErnestKostevich/horizon-genesis"><img src="https://img.shields.io/badge/source-horizon--genesis-8b5cf6?style=flat-square" alt="Source"/></a>
  <a href="https://github.com/ErnestKostevich/horizon-genesis/releases"><img src="https://img.shields.io/github/v/release/ErnestKostevich/horizon-genesis?style=flat-square&color=f59e0b&label=latest" alt="Latest release"/></a>
</p>

> **First public release — v0.0.1** (May 2026). Docs follow the
> shipping software; if something here doesn't match the binary you
> downloaded, open an issue.

---

## Get started in 60 seconds

```bash
# macOS / Linux
curl -L https://github.com/ErnestKostevich/horizon-genesis/releases/latest/download/horizon-linux-x64 -o horizon
chmod +x horizon
./horizon setup
```

```powershell
# Windows (PowerShell)
iwr https://github.com/ErnestKostevich/horizon-genesis/releases/latest/download/horizon-win-x64.exe -OutFile horizon.exe
.\horizon.exe setup
```

Or [download the desktop app](https://github.com/ErnestKostevich/horizon-genesis/releases/latest) — installers for Windows / macOS / Linux.

## Table of contents

### Getting started

- [What is Horizon?](docs/what-is-horizon.md)
- [Installation](docs/installation.md)
- [First-time setup](docs/getting-started.md)
- [Choosing a model provider](docs/providers.md)

### Desktop app

- [The chat interface](docs/desktop-chat.md)
- [Agent mode](docs/agent-mode.md)
- [Voice & wake word](docs/voice.md)
- [Computer use (screen / mouse / keyboard)](docs/computer-use.md)
- [Personas](docs/personas.md)

### Terminal CLI

- [CLI overview](docs/cli.md)
- [All 50+ commands](docs/cli-reference.md)
- [TUI — interactive shell](docs/tui.md)
- [Shell completion](docs/cli-completion.md)
- [Profiles (multiple environments)](docs/cli-profiles.md)

### Memory & skills

- [How Horizon remembers](docs/memory.md)
- [Workspace memory (`.horizon/`)](docs/workspace-memory.md)
- [Skills system](docs/skills.md)
- [Writing custom skills](docs/skills-custom.md)

### Connections (messaging + tools)

- [Telegram bot setup](docs/connect-telegram.md)
- [Discord bot setup](docs/connect-discord.md)
- [WhatsApp via Twilio](docs/connect-whatsapp.md)
- [Signal via self-hosted bridge](docs/connect-signal.md)
- [iMessage (macOS)](docs/connect-imessage.md)
- [Slack / Notion / Linear / GitHub](docs/connect-tools.md)

### Cost & providers

- [Cost tracking — `horizon cost`](docs/cost.md)
- [Smart auto routing — `--provider auto`](docs/auto-routing.md)
- [All 25 AI providers](docs/providers-full.md)
- [LiteLLM router (300+ models)](docs/litellm.md)
- [Running locally with Ollama](docs/local-models.md)

### Server / advanced

- [Running on a VPS](docs/deploy.md)
- [Headless HTTP API](docs/http-api.md)
- [Mobile PWA companion](docs/mobile-pwa.md)
- [Cron scheduling](docs/cron.md)
- [Sandbox backends (SSH / Modal / Daytona)](docs/sandbox.md)

### Plugins & marketplace

- [Browse the marketplace](https://horizonaai.dev/browse)
- [Installing plugins](docs/plugins-install.md)
- [Building plugins (developer guide)](https://github.com/ErnestKostevich/horizon-plugin-sdk)
- [Publishing & getting paid](docs/plugins-publish.md)

### Help

- [FAQ](docs/faq.md)
- [Troubleshooting](docs/troubleshooting.md)
- [Pricing](docs/pricing.md)
- [Privacy & data](docs/privacy.md)
- [Contact support](docs/support.md)

## Stay updated

- Website: [horizonaai.dev](https://horizonaai.dev)
- Releases: [github.com/ErnestKostevich/horizon-genesis/releases](https://github.com/ErnestKostevich/horizon-genesis/releases)
- Source code: [horizon-genesis](https://github.com/ErnestKostevich/horizon-genesis)

## License

Horizon AI is © Ernest Kostevich, distributed under the Business Source
License 1.1 (source-visible, free for non-commercial + small-team use).

These documentation pages themselves are licensed CC BY-SA 4.0 — you
may quote, mirror, and translate them with attribution.

See [LICENSE](LICENSE) for terms.
