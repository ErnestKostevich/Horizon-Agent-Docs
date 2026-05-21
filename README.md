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
# macOS / Linux — single binary, no Node required
curl -L https://github.com/ErnestKostevich/horizon-genesis/releases/latest/download/horizon-linux-x64 -o horizon
chmod +x horizon
./horizon setup
```

```powershell
# Windows (PowerShell)
iwr https://github.com/ErnestKostevich/horizon-genesis/releases/latest/download/horizon-win-x64.exe -OutFile horizon.exe
.\horizon.exe setup
```

Or [download the desktop app](https://github.com/ErnestKostevich/horizon-genesis/releases/latest) — installers for Windows / macOS / Linux. Package-manager users: see [install via npm / brew / scoop](docs/guides/install-via-package-manager.md).

## Table of contents

### Getting started

- [What is Horizon?](docs/what-is-horizon.md)
- [Installation](docs/installation.md)
- [Install via npm / brew / scoop](docs/guides/install-via-package-manager.md)
- [First-time setup](docs/getting-started.md)
- [Choosing a model provider](docs/providers.md)

### Concepts

- [9-layer memory architecture](docs/concepts/9-layer-memory.md)
- [Personas — five built-in voices](docs/concepts/personas.md)

### Desktop app & agent

- [Agent mode](docs/agent-mode.md)
- [Computer use — screen, mouse, keyboard](docs/guides/computer-use.md)
- [Computer use — advanced (OCR, multi-display, macros)](docs/guides/computer-use-advanced.md)
- [Durable agents — Kanban board + subagents](docs/guides/durable-agents.md)
- [Voice & wake word](docs/guides/voice-and-wake-word.md)

### Terminal CLI

- [CLI reference (legacy table)](docs/cli-reference.md)
- [Full CLI commands — every subcommand grouped by category](docs/reference/cli-commands.md)
- [CLI themes — 8 built-in palettes](docs/guides/cli-themes.md)

### Memory & models

- [9-layer memory architecture](docs/concepts/9-layer-memory.md)
- [Local models — Ollama / LM Studio / LocalAI](docs/guides/local-models.md)
- [All 25 AI providers — full reference](docs/reference/providers-full.md)

### Channels & integrations

- [Telegram bot setup](docs/guides/channels/telegram.md)
- [Discord bot setup](docs/guides/channels/discord.md)
- [WhatsApp via Twilio](docs/connect-whatsapp.md)

### Server / advanced

- [Server mode — run as a 24/7 bot](docs/server-mode.md)
- [HTTP API reference](docs/reference/http-api.md)

### Plugins

- [Plugin development — intro to the SDK](docs/guides/plugin-development.md)
- [Browse the marketplace](https://horizonaai.dev/browse)
- [Plugin SDK on GitHub](https://github.com/ErnestKostevich/horizon-plugin-sdk)

### Help

- [FAQ](docs/faq.md)
- [Troubleshooting](docs/troubleshooting.md)
- [Pricing](docs/pricing.md)
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
