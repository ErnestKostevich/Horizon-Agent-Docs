# Troubleshooting

## "No API key configured" / "Provider not set up"

Run the setup wizard:

```bash
horizon setup
```

Or set them manually:

```bash
horizon model gemini       # pick a provider
# then paste the key into Settings → Connections or:
node -e "require('./src/main/runtime/store-shim').initStores().keysStore.set('k_gemini', 'AIza...')"
```

## "horizon: command not found" after install

The binary directory isn't in your PATH.

**Linux/macOS**: add `~/.local/bin` (or wherever you installed) to PATH:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

**Windows**: open a new PowerShell window — Windows caches PATH per
shell. Or check that `%LOCALAPPDATA%\horizon-cli\bin-shims` is in
your user PATH.

## "Token rejected" when using the mobile PWA

The PWA's saved token doesn't match what `horizon serve` is using.

Two fixes:

1. Click "Disconnect server" in the PWA drawer, then re-pair with the
   QR URL printed by `horizon serve`.
2. Make sure you start `horizon serve` with `--token YOUR_TOKEN` —
   without it the token is randomised on each restart.

## "Docker not found" when executionMode=docker

Install Docker Desktop and make sure it's running before starting
Horizon. Then `horizon doctor` should show `docker ✓`.

If you don't want Docker, switch to `executionMode=host` in Settings →
Models, or use the SSH/Modal/Daytona backends.

## Agent never reaches the answer

Check the step rail / inspector — the agent might be:
- Stuck in a loop (try `--max-steps 12` for harder tasks)
- Blocked by permission denials (`--auto-approve` for trusted tasks)
- Rate-limited (provider quota — switch to `--provider auto`)

## Memory not persisting between sessions

Memory file is at:
- Windows: `%APPDATA%\horizon-ai\horizon_memory.json`
- macOS: `~/Library/Application Support/horizon-ai/horizon_memory.json`
- Linux: `~/.config/horizon-ai/horizon_memory.json`

If multiple Horizon instances run on different `--user-data-dir`
flags, they have separate memories. Use `horizon profile use default`
to return to the main profile.

## Voice / wake word doesn't trigger

Wake word needs:
- Microphone permission granted to the desktop app
- A speech provider configured (Deepgram or Groq voice)
- The chip "Wake" enabled in the chat composer

In Settings → Voice you can test the wake-word detection manually.

## Embeddings not working

`horizon doctor` will flag this. Common causes:

- No `k_openai` or `k_gemini` set — embeddings need one of those
- Model name changed — Horizon defaults to `gemini-embedding-001` /
  `text-embedding-3-small`; if the provider deprecated them, update
  in `embeddings.js` or open an issue

## Settings page won't open / blank

Hard reload: `Ctrl+Shift+R` (Cmd+Shift+R on macOS). If that doesn't
work, open DevTools (`Ctrl+Shift+I`) and check the console for errors.

## Mobile PWA service worker stuck on old version

In the PWA drawer → "Disconnect server" — that clears localStorage
and unregisters the service worker. Then reload.

## Cron daemon stopped firing

Cron entries are stored in `horizon-settings.json` under `cli.cron`.
If a daemon process was killed, no fires happen until you restart it
with `horizon cron daemon` or `horizon serve --enable-cron`.

For VPS deploys, use the systemd unit in [deploy.md](deploy.md) — it
auto-restarts on crash.

## Still stuck?

[FAQ](faq.md) → [GitHub issues](https://github.com/ErnestKostevich/horizon-genesis/issues) → email ernest2011kostevich@gmail.com
