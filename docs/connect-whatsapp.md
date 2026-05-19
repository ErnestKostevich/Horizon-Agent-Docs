# WhatsApp via Twilio (BYOK)

Horizon sends and receives WhatsApp messages through your own Twilio
account. Twilio's free sandbox gets you running in ~5 minutes.

## Why Twilio?

WhatsApp's first-party Business Platform API requires a Facebook
Business Manager account + verified phone number + template approval —
typically 1–2 weeks. Twilio's sandbox skips all that. Once you're in
production, you can switch to a dedicated WhatsApp number through
Twilio without changing any Horizon settings.

## Setup (5 minutes)

### 1. Sign up at twilio.com

Free account, no credit card needed for the sandbox.

### 2. Join the WhatsApp sandbox

In Twilio Console:
**Messaging → Try WhatsApp → Sandbox**

You'll see a number (usually `+1 415 523 8886`) and a "join code"
(like `join sunset-bicycle`). Send that exact text from your phone's
WhatsApp to the sandbox number. After Twilio confirms the join, your
phone is whitelisted to receive messages from the sandbox.

### 3. Copy your credentials

In Twilio Console dashboard, copy:
- **Account SID** (starts with `AC...`)
- **Auth Token**

### 4. Configure Horizon

**Desktop app**:
Settings → Connections → WhatsApp via Twilio
- Account SID: `AC...`
- Auth Token: `...`
- From: `whatsapp:+14155238886` (Twilio's sandbox number)
- Save

**CLI**:
```bash
horizon connect whatsapp --twilio-sid AC... --twilio-token X --from whatsapp:+14155238886
```

### 5. Test outgoing

In the desktop app, in any chat: `/agent send "hi from Horizon" via WhatsApp to whatsapp:+1234567890`.

Or in the TUI: `conn_whatsapp_send` is a tool the agent can call.

You should receive the message on the phone that joined the sandbox.

### 6. Enable inbound (optional)

For Horizon to **receive** WhatsApp messages and reply automatically,
you need a public webhook URL — Twilio sends each inbound message
there.

**For server deployments** running `horizon serve --port 18789` behind
nginx with TLS:

```
https://horizon.example.com/api/whatsapp/webhook
```

**For laptop testing**: use ngrok or Cloudflare Tunnel to expose your
local server:

```bash
ngrok http 18789
# copy the https URL ngrok prints
```

Then in Twilio Console:
**Messaging → WhatsApp Sandbox Settings**
- **When a message comes in**: `https://your-host/api/whatsapp/webhook`
- HTTP method: POST

Run Horizon with `--enable-whatsapp`:

```bash
horizon serve --port 18789 --token mysecret --enable-whatsapp
```

Now WhatsApp messages from the joined phone → Horizon → AI reply →
delivered back via Twilio.

## Going to production

Twilio's sandbox is great for testing but has limits (only whitelisted
numbers, message templates required after 24h of silence, daily quota).
For real use:

1. Buy a Twilio number with WhatsApp enabled
2. Submit a WhatsApp Business Profile (Twilio Console walks you through)
3. Get approved (24–72h)
4. Update Horizon's "From" to your new number

No code changes needed in Horizon.

## Costs

Twilio's WhatsApp pricing (2026): ~$0.005–0.05 per message depending
on conversation category and recipient country. The sandbox is free.

Horizon doesn't charge anything extra for WhatsApp — your subscription
covers the adapter, you pay Twilio directly.

## Security

- Twilio webhook requests are signed (X-Twilio-Signature header).
  Horizon verifies the signature against your saved auth token. Bogus
  POSTs return 403.
- The token is stored encrypted in your local horizon-keys.json (AES-256-GCM,
  machine-bound).

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Message not received on phone | Phone didn't join the sandbox | Send `join <code>` from WhatsApp again |
| `403 forbidden` on webhook | Wrong auth token saved | Re-paste in Settings → Connections |
| Twilio shows "delivery failed" | Wrong recipient format | Use `whatsapp:+1234567890`, not `+1234567890` |
| Inbound never triggers | Webhook URL not reachable from internet | Test with `curl https://your-host/api/health` from another machine |
| Random "session expired" replies | 24h silence between messages | Twilio policy — send a fresh message to re-open |

## Next

- [Signal via self-hosted bridge](connect-signal.md)
- [Telegram bot setup](connect-telegram.md)
- [Running Horizon on a server](deploy.md) — for production webhook hosting
