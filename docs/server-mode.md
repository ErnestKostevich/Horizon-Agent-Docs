# Server mode — run Horizon as a 24/7 bot

The same Horizon binary that runs as a desktop app can also run as a
headless server. One process, no window, exposing an HTTP API and
running your messaging-channel adapters around the clock.

This is what lets Horizon match the Hermes-style "server agent" workflow
— you SSH into a VPS, start the server, and your Telegram / Discord /
Email bot keeps answering messages even when your laptop is closed.

## What `horizon serve` actually does

```bash
horizon serve --port 18789 --token mysecret
```

- Boots the **same** agent loop, memory layers, skills, and personas
  the desktop app uses.
- Reads / writes the **same** `<userData>/horizon_memory.json` +
  `memory.sqlite` + `horizon_embeddings.json` files.
- Exposes an HTTP API on the port you chose:
  - `POST /api/chat` — single-turn chat (JSON or SSE stream)
  - `POST /api/agent` — full agent loop (plan → act → reflect)
  - `POST /api/mem/search` — semantic + keyword recall
  - `GET  /api/skills` — list installed skills
  - `GET  /api/personas` — list personas
  - `GET  /api/health` — liveness probe
  - …plus a dozen more, see `horizon serve --help`.
- All endpoints gate on `Authorization: Bearer <token>` if `--token` was
  set (or `HORIZON_TOKEN` env var). Without a token, the server binds
  to `127.0.0.1` only and generates a random token on each boot so
  localhost clients can still talk to it but the LAN can't.

## Three real use cases

### 1. Mobile companion at home

You're sitting at your PC with `horizon serve` running. From your phone
on the same Wi-Fi, you open the Horizon PWA (or just `http://192.168.x.x:18789`)
and chat with the same agent. It still has access to your machine —
because it's running on your machine.

### 2. 24/7 Telegram bot on a VPS

Rent a $5/mo box (DigitalOcean, Hetzner, Vultr, etc.). Copy the CLI
binary up there, set your provider keys, start the server with
adapters enabled:

```bash
horizon serve \
  --port 18789 \
  --token $HORIZON_TOKEN \
  --enable-telegram \
  --enable-discord \
  --enable-email
```

Now your Telegram bot, Discord bot, and email account all answer 24/7
even when your laptop is asleep. The agent's memory lives on the VPS —
you can `scp` the JSON file home periodically if you want a local backup.

### 3. Cron + webhook automations

```bash
# Daily morning briefing
0 9 * * * curl -sS -X POST http://localhost:18789/api/agent \
  -H "Authorization: Bearer $HORIZON_TOKEN" \
  -d '{"task":"check email, summarise unread, save the digest"}'
```

The agent runs end-to-end on your schedule and reports back via
whatever channel you wired (email, Telegram, a file, etc.).

## How is this different from "Horizon for me on my laptop"?

It's the *same* Horizon. The only thing that's different is whether
there's an Electron window opening on screen.

- **Desktop mode** — Electron window, voice, canvas, screen capture,
  computer use.
- **Server mode** — no window. The agent still works, but features
  that need a screen (voice, computer use, vision) are off.

If you want both, you can run both: the Electron app on your laptop,
`horizon serve` on a VPS. They're independent installs with independent
memory files. Some teams pair them — the VPS bot is the "always
listening" inbox; the desktop app is the "active driver" they use for
coding, browsing, etc.

## Channel adapter flags

```
--enable-telegram     # uses k_telegram_bot from your keys store
--enable-discord      # uses k_discord_bot
--enable-slack        # uses k_slack
--enable-whatsapp     # uses Twilio SID/token in settings
--enable-signal       # uses signal-cli bridge URL in settings
--enable-email        # uses email.imap.* + email.smtp.* settings
--enable-imessage     # macOS only — needs Messages.app automation perm
```

You can omit `--token` for localhost-only; the server picks a random
one and prints it once on startup.

## Security notes

- Always set `--token` if the server is reachable from the LAN. Without
  it, anyone on your network could hit `/api/agent`.
- For VPS deployment, put it behind a reverse proxy with HTTPS
  (Caddy / nginx-acme), and add a firewall rule that only opens the
  proxy port. Don't expose `18789` directly to the public internet.
- Your provider API keys live on the VPS in `~/.config/horizon-ai/` —
  encrypted with a machine-bound key. If the box gets compromised the
  attacker can use the keys but can't extract them clean. Rotate keys
  in your provider dashboard if you suspect compromise.

## What this is NOT

- Not a service we host. You install and run it yourself.
- Not paid. Same BSL 1.1 license as the rest of Horizon.
- Not a separate product. It's the same binary, `--serve` flag.
- Not for hosting agents that aren't yours. Each user runs their own.
