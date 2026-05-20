---
title: Telegram bot
description: Connect Horizon to Telegram via BotFather — five minutes, no webhooks.
lastUpdated: 2026-05-20
---

# Telegram bot

Horizon talks to Telegram through a regular bot account. Five minutes
of BotFather, paste one token into Settings, and your AI is reachable
from any phone, watch, or laptop that has Telegram installed.

No webhook setup, no public IP, no port forwarding — Horizon polls
Telegram's long-poll endpoint, so it works behind NAT, on a laptop, on
a Raspberry Pi at home.

> **One-bot model.** A Telegram bot is tied to a single Horizon
> instance. If you start two `horizon serve` processes with the same
> token, Telegram will return `Conflict: terminated by other getUpdates`
> and one will lose. Run one instance per token, period.

## Setup (≈ 5 minutes)

### 1. Create the bot in BotFather

Open Telegram and find [@BotFather](https://t.me/BotFather).

```
/start
/newbot
```

BotFather asks for:

- **Display name** — what shows in chat headers. Pick anything, e.g.
  "My Horizon".
- **Username** — must end in `bot`. e.g. `my_horizon_bot`. Must be
  globally unique on Telegram.

BotFather replies with a token that looks like:

```
8123456789:AAH-AbCdEf_ghIjKlMnOpQrStUvWxYz1234
```

Save that — it's the bot's password. Treat it like an API key.

### 2. Configure Horizon

#### Desktop app

Settings → Connections → Telegram → paste the token → Save.

The connection panel turns green and shows the bot's username.

![Telegram settings screenshot](./img/connect-telegram-settings.png)

#### CLI

```bash
horizon connect telegram --token 8123456789:AAH-AbCdEf_ghIjKlMnOpQrStUvWxYz1234
```

The token is stored encrypted in `horizon-keys.json` (AES-256-GCM,
machine-bound).

### 3. Start the live runtime

Two options.

**Option A — desktop GUI:** Settings → Connections → Telegram → "Live"
toggle on. The runtime starts polling immediately.

**Option B — headless server:**

```bash
horizon serve --port 18789 --token mysecret --enable-tg
```

Stderr should show:

```
telegram runtime: started
```

### 4. Test

Open Telegram, find your bot (by the username from step 1), tap
**Start**, and send a message:

```
You: hi
Bot: At your service, Sir. (active persona: jarvis)
You: what time is it in Tokyo?
Bot: 02:47 JST, which is 14 hours ahead of UTC.
```

The bot reads your active persona, active provider, and active
memory — same as the desktop and CLI. Anything you've told Horizon
elsewhere is available here too.

## Allow-list (optional but recommended)

By default any Telegram user who finds your bot can message it.
That's fine if you keep the username private, but Telegram usernames
are guessable. Lock it down to known users:

```bash
# Comma-separated Telegram user IDs (not usernames — IDs)
horizon set connection.telegram_bot.allowed_user_ids 123456789,987654321
```

To find your Telegram ID: message [@userinfobot](https://t.me/userinfobot).
It replies with your numeric ID.

Once allow-list is set, the bot silently drops messages from any
other user ID. They get no response, no error — they don't even know
the bot is live.

## Slash commands

Horizon registers a few commands with Telegram so they auto-complete:

| Command | What it does |
|---|---|
| `/start` | Greeting + active persona |
| `/persona <id>` | Switch persona for this chat session |
| `/model <provider>` | Switch model provider |
| `/memory` | List recent memories |
| `/help` | Available commands |

Anything else is treated as a chat message routed to the agent.

## Group chats

Bots in Telegram groups are constrained by **privacy mode**. By
default a bot only sees:

- Messages that mention it (`@my_horizon_bot ...`)
- Direct replies to its messages
- Slash commands (`/...`)

If you want the bot to read everything in a group (e.g. for a
"summarise the standup" workflow), disable privacy mode in BotFather:

```
/setprivacy
@my_horizon_bot
Disable
```

Then re-add the bot to the group (privacy changes don't apply
retroactively).

## Outgoing messages from the agent

The agent can send Telegram messages via the `conn_telegram_send_message`
tool. Useful for chains like "summarise this and message it to my
saved messages":

```bash
horizon agent "summarise CHANGELOG.md and DM it to Telegram chat 123456789"
```

The agent will request a `conn_telegram_send_message` tool call; the
permission gate confirms unless you've granted "always allow" for
that tool.

## Limits & quotas

Telegram's Bot API has a few rate limits worth knowing about:

| Limit | Value |
|---|---|
| Outgoing messages per second (per bot, all chats) | 30 |
| Outgoing messages per second per chat | 1 |
| Outgoing messages per minute per group | 20 |
| Message length | 4096 characters |
| Inline button text | 64 characters |
| File upload size | 50 MB (bot) / 2 GB (premium user via bot) |

Horizon's runtime auto-batches outgoing messages and chunks long
replies on the 4096-char boundary at word breaks (it doesn't cut in
the middle of a code block).

If you hit `429 Too Many Requests`, the runtime backs off per
Telegram's `retry_after` and resumes — no manual intervention needed.

## Persistence

The conversation isn't a one-off — every Telegram message is written
into the same 9-layer memory as desktop chat, with `source:
"telegram"` and a `chatId` tag. Search across them:

```bash
horizon mem search "what did I tell the Telegram bot about coffee"
```

## Privacy notes

- **The token is the keys-to-the-kingdom.** Anyone with the token can
  impersonate the bot. Revoke it via BotFather (`/revoke`) if leaked.
- **Telegram sees every message.** If "no third party reads my chats"
  is a hard requirement, Telegram is the wrong channel; consider
  [Signal](../connect-signal.md) instead.
- **Allow-list users early.** Username squatting is rare but real;
  a public username on a personal bot is a tiny attack surface.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Bot doesn't respond | Live toggle off | Settings → Connections → Telegram → Live ON |
| `Conflict: terminated by other getUpdates` in logs | Another instance is polling with the same token | Stop the other instance; only one runtime per token |
| Long-poll loop never starts | Token format wrong | Should look like `<digits>:<base64-ish>`. Re-copy from BotFather |
| Bot replies but with stale persona | You switched persona mid-stream | Open desktop app to confirm; reload runtime: `horizon connect test telegram_bot` |
| Group chat ignores everything | Privacy mode still on | BotFather → `/setprivacy` → Disable + re-add bot |
| Message split awkwardly | Reply longer than 4096 chars | Expected — Horizon chunks at word boundaries |
| Bot answers anyone | No allow-list | Set `allowed_user_ids` (see above) |
| Network error on every poll | Outbound 443 blocked | Telegram Bot API is `api.telegram.org`; allow it through your firewall |

### Diagnostic commands

```bash
# Ping the Telegram getMe endpoint with the stored token
horizon connect test telegram_bot

# Show runtime status
horizon connect list

# Tail the last 100 Telegram-runtime log lines
horizon logs telegram --tail 100
```

## Related

- [Discord bot](./discord.md) — same shape, different protocol
- [WhatsApp via Twilio](../../connect-whatsapp.md) — webhook-based channel
- [HTTP API](../../reference/http-api.md) — `/api/status` includes
  `liveRunning` for each connection
- [Personas](../../concepts/personas.md) — per-chat persona overrides
