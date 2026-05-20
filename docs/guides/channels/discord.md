---
title: Discord bot
description: Connect Horizon as a Discord bot via Developer Portal — gateway + slash commands.
lastUpdated: 2026-05-20
---

# Discord bot

Horizon connects to Discord as a regular bot account using the
Gateway protocol — a persistent WebSocket connection that delivers
events the moment they happen. No public webhook URL, no polling.

Setup takes about 10 minutes total: create the app in Discord's
Developer Portal, copy the bot token + OAuth URL, paste into Settings,
flip "Live" on.

## Setup (≈ 10 minutes)

### 1. Create the application

Open [discord.com/developers/applications](https://discord.com/developers/applications) and
sign in with the Discord account that will *own* the bot.

Click **New Application**, give it a name (e.g. "My Horizon"), accept
the Developer Terms.

### 2. Add a bot user

Sidebar → **Bot** → **Add Bot** → confirm.

Optional, but recommended:

- **Public Bot** (under Authorization Flow): **OFF** if you want only
  you to be able to invite the bot to servers.
- **Privileged Gateway Intents**: enable **Message Content Intent**.
  Without it, the bot can read messages that @-mention it but not
  free-form messages in channels it's a member of. (Discord requires
  verification for this intent on bots in 100+ servers, but you're
  fine below that.)

![Discord developer portal screenshot](./img/connect-discord-portal.png)

### 3. Copy the bot token

On the same Bot page, click **Reset Token** (or **Copy** if it's the
first time). Copy the long string that appears — it looks like:

```
YOUR_DISCORD_BOT_TOKEN_HERE
```

**Save it now** — Discord won't show it to you again. If you lose it,
you reset and re-paste in Horizon.

### 4. Build an invite URL

Sidebar → **OAuth2** → **URL Generator**.

Check:

- **Scopes**: `bot`, `applications.commands`
- **Bot Permissions** (under the Scopes selection):
  - `Send Messages`
  - `Read Message History`
  - `Use Slash Commands`
  - `Embed Links` (for richer replies)
  - `Attach Files` (if you want the bot to send files)
  - `Add Reactions` (optional)

Copy the **Generated URL** at the bottom. Open it in a new tab, pick
a server you have "Manage Server" on, and authorise.

The bot now appears in your server's member list — offline until you
start the runtime.

### 5. Paste the token into Horizon

#### Desktop app

Settings → Connections → Discord → paste token → Save.

#### CLI

```bash
horizon connect discord --token YOUR_DISCORD_BOT_TOKEN_HERE
```

The token is stored encrypted in `horizon-keys.json`.

### 6. Start the live runtime

Either:

**Desktop GUI** — Settings → Connections → Discord → "Live" toggle on.

**Or headless server:**

```bash
horizon serve --port 18789 --token mysecret --enable-discord
```

Stderr should show:

```
discord runtime: started
```

The bot's member-list status flips from offline to online within
a second or two.

### 7. Test

In your Discord server, @-mention the bot:

```
@MyHorizon what's 12!
```

Reply within ~1–3 s depending on provider.

If you also enabled Message Content Intent, the bot reads any
message in any channel it's a member of — you don't have to
@-mention.

## Slash commands

Horizon registers a small set of slash commands per server. Type `/`
in the chat box to see them:

| Command | What it does |
|---|---|
| `/horizon ask <question>` | One-turn chat reply |
| `/horizon persona <id>` | Switch persona for this server |
| `/horizon model <provider>` | Switch model provider |
| `/horizon memory` | Send the user a DM with recent memories |
| `/horizon help` | List commands |

Slash commands appear in any channel the bot can read. Discord
auto-completes the arguments — no typos.

## Channels & permissions

You usually don't want the bot replying to *every* message in *every*
channel. Two ways to restrict:

### By Discord role / channel permission

Standard Discord — set the bot's "View Channel" permission to false
on channels it shouldn't see. Done in the channel settings, no
Horizon config needed.

### By Horizon allow-list

```bash
# Comma-separated Discord channel IDs the bot will read
horizon set connection.discord_bot.allowed_channel_ids 123,456,789
```

Anything outside the allow-list is silently dropped. Set to empty
string to disable the allow-list (read all channels you have
permission for).

To find a channel ID: enable Developer Mode in Discord (User Settings
→ Advanced → Developer Mode), right-click the channel → Copy ID.

## DMs

The bot accepts DMs from anyone who shares a server with it. You can
turn this off in the Bot page → "Bot User" section. DMs are private
1:1 chats; the bot replies in the same channel.

## Outgoing messages from the agent

The agent can send Discord messages via the
`conn_discord_send_message` tool. Use cases: post a summary to a
#notifications channel, ping yourself in DM when a cron job finishes.

```bash
horizon agent "summarise CHANGELOG.md and post it to Discord channel 123456"
```

The permission gate confirms unless you've granted "always allow" for
that tool.

## Embeds and code blocks

Horizon's Discord adapter detects code blocks in agent replies and
wraps them in Discord's syntax-highlighted code-block format:

````
```js
function hello() {
  console.log("Horizon on Discord");
}
```
````

Becomes a properly highlighted block in the Discord client.

Long replies (>2000 chars — Discord's message-length limit) are
auto-chunked at paragraph boundaries; nothing gets cut.

## Limits & quotas

Discord's rate limits:

| Limit | Value |
|---|---|
| Outgoing messages per channel | 5 per 5 s (per bot) |
| Outgoing messages per server | 50 per 10 s (per bot) |
| Slash commands per guild | 200 |
| Message length | 2000 chars (4000 for premium) |
| Slash command response window | 3 s (use deferred replies for longer) |
| Gateway WebSocket reconnects | Exponential backoff per Discord guidance |

The runtime handles rate-limit `429`s automatically with the
Discord-provided `retry_after`.

## Persistence

Every Discord message + every reply is written into the same 9-layer
memory as desktop chat, tagged `source: "discord"` with `guildId`,
`channelId`, `userId`. Search across them:

```bash
horizon mem search "what did I post to the dev channel about migrations"
```

## Privacy notes

- **Token = bot.** Anyone with the token controls the bot — they can
  read all channels it's in and post as it. Reset immediately if
  leaked (Bot page → Reset Token).
- **Message Content Intent reads everything.** Only enable it if
  you genuinely want passive reading. Without it, the bot only
  sees @-mentions and slash commands.
- **Discord stores everything.** Same caveat as Telegram — Discord
  Inc. has the cleartext.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Bot stays offline | Live toggle off, OR invalid token | Settings → Connections → Discord → re-paste token → Live ON |
| Bot online but doesn't respond | Message Content Intent disabled | Discord Dev Portal → Bot → enable Message Content Intent |
| Slash commands don't appear | Bot lacks `applications.commands` scope | Re-invite with the OAuth URL from step 4 (now with that scope) |
| `Disallowed intents` error in logs | Privileged intent toggled on portal but bot in 100+ servers | Submit verification or stay under 100 servers |
| Bot replies twice | Two instances running with same token | Stop the duplicate; Discord allows one Gateway connection per token |
| Markdown looks weird | Discord's flavoured markdown ≠ standard | Horizon escapes `*` and `_` automatically; code blocks pass through |
| 404 on slash command | Commands not synced after a code update | Restart the runtime — Horizon re-registers on boot |
| Reply timed out | Provider slow, slash command needs `defer` | Long replies use Discord's `deferReply` automatically; if you're hitting > 15 min, that's the model |

### Diagnostic commands

```bash
# Ping Discord's gateway with the stored token
horizon connect test discord_bot

# Show all connections + their live status
horizon connect list

# Tail Discord-runtime logs
horizon logs discord --tail 100
```

## Related

- [Telegram bot](./telegram.md) — same idea, different protocol
- [HTTP API](../../reference/http-api.md) — `/api/status` shows the
  Discord runtime state
- [Personas](../../concepts/personas.md) — per-server persona overrides
