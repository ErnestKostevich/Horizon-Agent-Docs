---
title: Install via npm / brew / scoop
sidebar_position: 12
description: Install the Horizon CLI through your distro's package manager — npm for any platform, Homebrew for macOS/Linux, Scoop for Windows.
lastUpdated: 2026-05-21
---

# Install via package manager

The Horizon CLI ships as a single binary you can `curl` down from
GitHub Releases — but if you prefer your distro's package manager,
we publish to **npm**, **Homebrew**, and **Scoop**. Each one wires
the binary onto your `$PATH` automatically and handles upgrades.

> Looking for the GUI desktop app instead? See the
> [installation page](../installation.md) — installers are at
> [GitHub Releases](https://github.com/ErnestKostevich/horizon-genesis/releases/latest).

## npm — cross-platform

Works on Windows, macOS, and Linux. Requires **Node 22 LTS** (the
same Node version the desktop app bundles).

```bash
# Global install
npm install -g @horizonai/cli

# Verify
horizon version

# First-time setup wizard
horizon setup
```

To upgrade:

```bash
npm install -g @horizonai/cli@latest
# or use the built-in:
horizon update
```

The npm package wraps the same single-binary that GitHub Releases
ships. It's the easiest path for users who already have a Node
environment.

### Node version manager users

If you use `nvm`, `fnm`, `volta`, or `asdf`, install Horizon **under
your active Node** — not as root. Global installs under a Node
version manager land in the user-local prefix, so no sudo is needed.

```bash
nvm use 22
npm install -g @horizonai/cli
```

Switching Node versions later won't break Horizon (the binary is
self-contained), but `horizon update` won't be on your path until
you re-install under the new Node version.

## Homebrew — macOS and Linux

We publish a tap for first-class brew support:

```bash
# Add the tap (one-time)
brew tap ernestkostevich/horizon

# Install
brew install horizon

# Verify
horizon version
```

To upgrade:

```bash
brew update
brew upgrade horizon
```

Brew handles dependencies (Node, native modules) and writes the
binary to `/opt/homebrew/bin/horizon` (Apple Silicon) or
`/usr/local/bin/horizon` (Intel Mac / Linux).

### Shell completion

Brew automatically wires completion if your shell honours
`$(brew --prefix)/etc/bash_completion.d` (bash) or
`$(brew --prefix)/share/zsh/site-functions` (zsh). For fish or
PowerShell:

```bash
horizon completion fish > ~/.config/fish/completions/horizon.fish
horizon completion pwsh  > $PROFILE
```

## Scoop — Windows

For Windows users who prefer Scoop over MSI/EXE installers:

```powershell
# Add the bucket (one-time)
scoop bucket add horizon https://github.com/ErnestKostevich/scoop-horizon

# Install
scoop install horizon

# Verify
horizon version
```

To upgrade:

```powershell
scoop update horizon
```

Scoop drops the binary into `%USERPROFILE%\scoop\apps\horizon\current\`
and shims it onto your `$PATH`. No admin rights required.

### PowerShell completion

```powershell
horizon completion pwsh | Out-File -Append $PROFILE
. $PROFILE
```

## Choosing between methods

| Method | Best for |
|---|---|
| **GitHub Releases binary** | You want zero extra dependencies. Just `curl` and run. |
| **npm** | You're already a Node user, want easy `horizon update`. |
| **Homebrew** | You're on macOS/Linux and brew is already in your workflow. |
| **Scoop** | You're on Windows and don't want a system-wide installer. |

All four install the same binary. Memory, settings, and keys all
live in the same `<userData>` directory regardless of how you
installed.

## Auto-update behaviour

Every method supports the in-place updater:

```bash
horizon update             # update to latest
horizon update --check     # just check, don't apply
```

The updater downloads a fresh binary from GitHub Releases and
swaps it in atomically. After the swap completes you re-run
whatever command you had pending.

**Note:** package managers and `horizon update` can disagree on
version. If you installed via brew but ran `horizon update`, the
brew formula will overwrite the in-place updater binary on the next
`brew upgrade`. Pick one method and stick with it.

## Uninstall

| Method | Uninstall |
|---|---|
| GitHub Releases binary | Delete the binary + the data folder |
| npm | `npm uninstall -g @horizonai/cli` |
| brew | `brew uninstall horizon` |
| scoop | `scoop uninstall horizon` |

The data folder (`%APPDATA%\horizon-ai\` on Windows,
`~/Library/Application Support/horizon-ai/` on macOS,
`~/.config/horizon-ai/` on Linux) is **not** removed by any of the
package-manager uninstall steps — your memory, keys, and skills
survive a reinstall. Delete it manually if you want a clean reset:

```bash
# Linux
rm -rf ~/.config/horizon-ai

# macOS
rm -rf ~/Library/Application\ Support/horizon-ai

# Windows (PowerShell)
Remove-Item -Recurse -Force $env:APPDATA\horizon-ai
```

## Troubleshooting

**`horizon: command not found` after install**
Restart your shell, or `source ~/.zshrc` (zsh) / `source ~/.bashrc`
(bash). Scoop users: open a new PowerShell window.

**`EACCES: permission denied` from npm**
You ran `npm install -g` without a user-prefix. Either use a Node
version manager (recommended) or `sudo npm install -g` (not
recommended). Best fix: install nvm and re-install Node under your
own user.

**Mixed installs**
If `which horizon` shows multiple paths (e.g. `/opt/homebrew/bin/horizon`
and `/usr/local/bin/horizon` on macOS), uninstall the older one.
The newer entries on `$PATH` win, which can be confusing when
debugging "I upgraded but `horizon version` shows the old one".

## Where to go next

- [First-time setup](../getting-started.md) — wizard walk-through
- [Full CLI commands](../reference/cli-commands.md)
- [CLI themes](cli-themes.md) — switch palettes
