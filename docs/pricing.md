# Pricing

Horizon is a paid product with a 5-day free trial. You also pay your
chosen AI provider directly (BYOK) — Horizon doesn't mark up tokens.

## Trial

- **5 days, free**, no payment method required to start.
- Full functionality: desktop app + CLI + serve + every plugin.
- Trial starts on first launch; counted on your machine.

## Subscription

After the trial, continued use of commercial features requires a
subscription. Pricing details and the manage-subscription flow live
on the dashboard:

→ [horizonaai.dev/pricing](https://horizonaai.dev/pricing)

## What you also pay outside Horizon

Your AI provider charges per-token. Examples (mid-2026 list prices):

| Provider | Free tier | Paid range |
|---|---|---|
| Google Gemini | 1,500 req/day | $0.10/M (Flash) — $5/M (Pro) |
| Groq | 30 req/min | $0.05/M (Llama 8B) — $0.79/M (70B) |
| Cerebras | Generous | $0.85/M Llama 70B (fastest) |
| Anthropic Claude | — | $3/M input + $15/M output (Sonnet) |
| OpenAI | — | $1.25/M input + $10/M output (gpt-5.4) |
| DeepSeek | — | $0.14/M (cheapest non-local) |
| Ollama (local) | Unlimited | $0 — runs on your CPU/GPU |

`horizon cost` shows your spend with a daily bar chart.

`--provider auto` automatically routes to free-tier / cheap providers
when possible — typical user spends ~$0–5/month on AI tokens at
moderate use.

## What's included in your subscription

- The desktop app + CLI + TUI + headless serve + Mobile PWA
- All 25 direct providers + LiteLLM router (300+ models reachable)
- 8-layer memory + semantic recall + workspace memory
- Plugin marketplace access + ability to publish your own
- Voice / wake word / continuous talk mode
- Computer use (vision + mouse + keyboard)
- All sandbox backends: host / Docker / SSH / Modal / Daytona
- Channel adapters: Telegram / Discord / Slack / Notion / Linear /
  GitHub / WhatsApp / Signal / iMessage
- Multi-profile + cron + cost tracking
- Updates + bug fixes for as long as you're subscribed

## Plugin authors

If you publish a paid plugin in the marketplace, you keep **70%** of
each sale. Horizon takes 30% to cover hosting + payment processing.
Payouts in USDT (TRC20/BSC/TON/SOL), weekly, $30 minimum.

Free plugins are also welcome and don't cost the author anything.

## Refunds

Within 7 days of subscribing, contact
ernest2011kostevich@gmail.com for a no-questions-asked refund.

## Team / enterprise

For teams >5 seats, self-hosting Horizon-as-a-service, or running on
internal infrastructure beyond what BSL allows: contact
ernest2011kostevich@gmail.com for a commercial license.

## Why not free?

Horizon is built by one person ([Ernest Kostevich](https://github.com/ErnestKostevich))
without VC funding. Subscriptions pay for:

- Continued development + new features
- Marketplace hosting + payment processing for plugin authors
- 7×24 uptime of horizonaai.dev + the marketplace
- Security audits + signed releases

Source code is visible under BSL-1.1 (and will convert to AGPL-3.0
after the change date). You can audit every line; you can contribute;
you can fork for non-commercial use. The paid tier exists so this
project survives.
