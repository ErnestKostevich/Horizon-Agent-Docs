# Installation

Three ways to run Horizon — pick the one that fits your workflow.

## Option 1 — Desktop app (recommended for most users)

The full GUI with chat, voice, computer use, plugin marketplace.

Download the latest installer for your OS from
[GitHub Releases](https://github.com/ErnestKostevich/horizon-genesis/releases/latest):

| Platform | File |
|---|---|
| **Windows x64** | `Horizon-AI-Setup-*.exe` (NSIS installer) |
| **Windows portable** | `Horizon-AI-Portable-*.exe` (no installer) |
| **macOS Intel + Apple Silicon** | `Horizon-AI-*.dmg` |
| **Linux** | `Horizon-AI-*.AppImage` (portable) or `horizon-ai_*_amd64.deb` |

After installation:

1. Launch Horizon.
2. The first-run wizard asks for one provider key (Gemini and Groq are free).
3. Start chatting.

**macOS note**: builds are unsigned. Right-click the .dmg → Open the
first time, otherwise Gatekeeper warns. After that it remembers.

## Option 2 — Terminal CLI (for terminal-first users)

Single binary, no Node.js install needed.

**macOS / Linux**
```bash
curl -L https://github.com/ErnestKostevich/horizon-genesis/releases/latest/download/horizon-linux-x64 -o /usr/local/bin/horizon
chmod +x /usr/local/bin/horizon
horizon setup
```

**Windows (PowerShell)**
```powershell
iwr https://github.com/ErnestKostevich/horizon-genesis/releases/latest/download/horizon-win-x64.exe -OutFile horizon.exe
.\horizon.exe setup
```

After `horizon setup` you can:

```bash
horizon                          # launch the interactive TUI
horizon "find every TODO"        # one-shot agent run
horizon chat "what's 2+2?"       # quick chat
horizon serve --port 18789       # headless HTTP API for mobile/cron
```

See the [full CLI reference](cli-reference.md) for all 50+ commands.

## Option 3 — From source

If you want to read the code, contribute, or build a custom version.

```bash
git clone https://github.com/ErnestKostevich/horizon-genesis
cd horizon-genesis
npm ci
npm start                 # launch the desktop app
# or
node bin/horizon.js       # the CLI
```

You need **Node 22 LTS** for source installs. Source builds open a
"preview window" — they remind you that running from source isn't an
official build. Use the GitHub Releases binaries for production.

## Verifying downloads

Every CLI binary ships with a matching `.sha256` file:

```bash
# Linux/macOS
sha256sum -c horizon-linux-x64.sha256

# Windows PowerShell
Get-FileHash horizon-win-x64.exe -Algorithm SHA256
# compare manually with the .sha256 file content
```

## What's installed where

After installation, Horizon stores everything under:

| Platform | Data location |
|---|---|
| Windows | `%APPDATA%\horizon-ai\` |
| macOS | `~/Library/Application Support/horizon-ai/` |
| Linux | `~/.config/horizon-ai/` |

This folder holds your encrypted API keys, memory file, installed
skills/plugins, and cost log. Nothing leaves your machine without your
explicit action.

## Uninstall

- **Desktop app**: standard OS uninstaller. To also clean settings:
  delete the data folder listed above.
- **CLI binary**: `rm /usr/local/bin/horizon` (or whatever path you
  picked) + delete the data folder.

## Next

- [First-time setup wizard](getting-started.md)
- [Pick a model provider](providers.md)
