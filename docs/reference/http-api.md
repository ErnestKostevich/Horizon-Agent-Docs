---
title: HTTP API
description: Every endpoint exposed by `horizon serve` — auth, request shape, examples, and SSE streaming.
lastUpdated: 2026-05-20
---

# HTTP API

`horizon serve` exposes a small JSON HTTP API over Node's stdlib `http`
module — no Express, no extra dependencies. The mobile PWA, the cron
runner, your own scripts, and anything that can speak Bearer-token HTTP
all use the same surface.

The default bind is `127.0.0.1:18789`. Pass `--host 0.0.0.0` and a
firewall rule if you want LAN access; expose to the public internet
behind a reverse proxy with TLS (nginx, Caddy, Cloudflare Tunnel — see
[Running on a VPS](../deploy.md)).

## Starting the server

```bash
horizon serve --port 18789 --token my-secret
```

The token is required on every `/api/*` call. If you omit `--token` and
don't set `HORIZON_TOKEN`, Horizon generates a random one and prints it
to stderr on start — fine for local poking, not what you want for a
service that survives a reboot.

```bash
# stable token + listen on all interfaces
HORIZON_TOKEN=$(openssl rand -hex 32) horizon serve --host 0.0.0.0 --port 18789
```

> **Why no Express?** The whole API is < 300 lines on `http.createServer`
> plus a tiny router. Skipping Express avoids a ~60-package transitive
> tree and keeps `horizon serve` cold-start in the < 200 ms range.

## Authentication

All `/api/*` paths require a token. Two ways to send it:

```bash
# Preferred — Authorization header
curl -H "Authorization: Bearer $HORIZON_TOKEN" http://127.0.0.1:18789/api/status

# Fallback — query string (for EventSource and browsers that strip headers)
curl "http://127.0.0.1:18789/api/status?token=$HORIZON_TOKEN"
```

Token comparison is constant-time (`crypto.timingSafeEqual`), so a
remote attacker can't byte-by-byte guess the secret by timing.

The Twilio WhatsApp webhook (`POST /api/whatsapp/webhook`) bypasses the
Bearer gate because Twilio doesn't speak custom headers — it's
authenticated via `X-Twilio-Signature` HMAC instead.

## Endpoints

### `GET /api/health`

Liveness probe. Returns `{ ok: true, ts: <epoch-ms> }` with no auth-side
work, suitable for a load balancer.

```bash
curl http://127.0.0.1:18789/api/health
# {"ok":true,"ts":1747765800000}
```

### `GET /api/version`

Full build info: package version, node version, platform, active
provider, persona, and counts.

```bash
curl -H "Authorization: Bearer $T" http://127.0.0.1:18789/api/version
```

```json
{
  "version": "0.0.1",
  "name": "horizon-genesis",
  "nodeVersion": "v22.10.0",
  "platform": "linux",
  "provider": "gemini",
  "persona": "jarvis",
  "memory": { "memories": 187, "facts": 24 },
  "skills": 11,
  "embeddings": { "available": true, "provider": "openai" },
  "executor": { "backend": "host" }
}
```

### `GET /api/status`

Mobile-friendly snapshot. Flat shape so a phone header can render it
without nested reads. The PWA polls this for the green / red dot.

```json
{
  "ok": true,
  "online": true,
  "provider": "gemini",
  "model": "gemini-2.5-flash",
  "persona": "jarvis",
  "lang": "en",
  "memoryCount": 187,
  "factCount": 24,
  "skillsCount": 11,
  "serverVersion": "0.0.1",
  "nodeVersion": "v22.10.0",
  "platform": "linux",
  "ts": 1747765800000
}
```

### `GET /api/personas`

Lists all personas (5 built-in + your custom ones).

```bash
curl -H "Authorization: Bearer $T" http://127.0.0.1:18789/api/personas
```

See [Personas](../concepts/personas.md) for the full schema.

### `GET /api/providers`

Lists all providers Horizon knows about, plus whether you have a key
configured for each. `auto` is included as a virtual choice that
resolves at call time.

```json
[
  { "id": "auto", "model": "", "defaultModel": "", "hasKey": true, "local": false, "active": false },
  { "id": "claude", "model": "claude-sonnet-4-6", "defaultModel": "claude-sonnet-4-6", "hasKey": true, "local": false, "active": false },
  { "id": "gemini", "model": "gemini-2.5-flash", "defaultModel": "gemini-2.5-flash", "hasKey": true, "local": false, "active": true },
  { "id": "ollama", "model": "llama3.1", "defaultModel": "llama3.1", "hasKey": false, "local": true, "active": false }
]
```

`hasKey` is `true` for local providers when an `ollamaUrl` /
`lmStudioUrl` / `localAiUrl` is set.

### `POST /api/chat`

Single-turn chat. Streaming markdown reply (when supported by the
provider) and usage stats.

```bash
curl -X POST http://127.0.0.1:18789/api/chat \
  -H "Authorization: Bearer $T" \
  -H "Content-Type: application/json" \
  -d '{
    "message": "What is the capital of Lithuania?",
    "history": [],
    "persona": "alfred"
  }'
```

Body fields:

| Field | Type | Default | Notes |
|---|---|---|---|
| `message` | string | required | The user's message |
| `history` | array | `[]` | Prior `{role, content}` turns to inject as context |
| `provider` | string | active | Override provider for this call |
| `model` | string | active | Override model |
| `persona` | string | active | Override persona |

Response:

```json
{ "reply": "Vilnius.", "model": "gemini-2.5-flash", "usage": { "in": 18, "out": 4 } }
```

### `POST /api/agent`

Full agent loop with tool use, optional reflection, and an optional SSE
stream. The expensive one — calls `runAgentLoop` under the hood.

```bash
curl -X POST http://127.0.0.1:18789/api/agent \
  -H "Authorization: Bearer $T" \
  -H "Content-Type: application/json" \
  -d '{ "task": "review src/auth.js for security issues", "max_steps": 8 }'
```

Body fields:

| Field | Type | Default | Notes |
|---|---|---|---|
| `task` | string | required | The goal |
| `history` | array | `[]` | Prior context |
| `max_steps` | int | `8` | Iteration cap |
| `reflect` | bool | `true` | Run reflection epilogue |
| `provider` / `model` / `persona` | string | active | Per-call overrides |

#### SSE streaming

If you set `Accept: text/event-stream`, the response is a stream of
events — one per step — terminated by `event: end`.

```bash
curl -N -X POST http://127.0.0.1:18789/api/agent \
  -H "Authorization: Bearer $T" \
  -H "Content-Type: application/json" \
  -H "Accept: text/event-stream" \
  -d '{ "task": "summarise CHANGELOG.md" }'
```

Each event payload:

```
event: step
data: { "type": "thought", "text": "Reading CHANGELOG.md…" }

event: step
data: { "type": "tool_call", "tool": "read_file", "args": { "path": "CHANGELOG.md" } }

event: step
data: { "type": "tool_result", "tool": "read_file", "result": "..." }

event: end
data: { "ok": true, "answer": "...", "steps": 3, "stopped": false, "error": null }
```

> **Tool permissions in serve mode:** `runAgent` from the HTTP API
> auto-approves every tool call (`askPermission: () => true`). There's
> no human to click "allow", so the security model is the Bearer token
> itself — anyone with the token can run any tool. Plan accordingly.

### `GET /api/skills`

Returns the full skill list with metadata (id, scope, description,
allowedTools, tags).

```bash
curl -H "Authorization: Bearer $T" http://127.0.0.1:18789/api/skills
```

### `POST /api/skill/:id/run`

Runs the agent loop with one skill forced into the system prompt.
Useful for invoking a specific workflow without natural-language
hinting.

```bash
curl -X POST http://127.0.0.1:18789/api/skill/morning-briefing/run \
  -H "Authorization: Bearer $T" \
  -H "Content-Type: application/json" \
  -d '{ "task": "for the last 24h" }'
```

### `GET /api/memories?limit=N&q=query`

Recent memories OR search results. Default `limit=20`, max `200`.

```bash
# Last 20 memories
curl -H "Authorization: Bearer $T" "http://127.0.0.1:18789/api/memories?limit=20"

# Semantic search
curl -H "Authorization: Bearer $T" "http://127.0.0.1:18789/api/memories?q=yerba+mate"
```

Response:

```json
{
  "memories": [
    { "id": "m_abc", "content": "User drinks yerba mate every morning.", "ts": 1747000000000, "importance": 0.7, "tags": ["preference"] }
  ],
  "total": 187
}
```

### `GET /api/mem/profile`

Returns the user profile — Big Five + communication style + expertise +
goals + preferences. See [9-layer memory](../concepts/9-layer-memory.md).

### `POST /api/mem/search`

Programmatic search; takes `{ query, limit, semantic? }`.

```bash
curl -X POST http://127.0.0.1:18789/api/mem/search \
  -H "Authorization: Bearer $T" \
  -H "Content-Type: application/json" \
  -d '{ "query": "favourite coffee", "limit": 5, "semantic": true }'
```

### `POST /api/mem/forget`

Delete one memory or fact. Body: `{ memory: "<id>" }` or `{ fact: "<key>" }`.

### `POST /api/model`, `POST /api/persona`, `POST /api/settings`

Atomic setting writes. Bodies are `{ provider, model? }`, `{ id }`, and
`{ persona?, provider?, model?, lang? }` respectively. Returns
`{ ok: true }`.

### `POST /api/whatsapp/webhook`

Twilio inbound webhook. Auth via Twilio's HMAC signature, not the
Bearer token. See [WhatsApp via Twilio](../connect-whatsapp.md) for
configuration.

## Error responses

All errors are JSON. Common ones:

| Status | Body | Meaning |
|---|---|---|
| `401` | `{ "error": "unauthorized" }` | Missing or wrong token |
| `400` | `{ "error": "message required" }` | Bad request body |
| `404` | `{ "error": "not found", "path": "/api/foo" }` | Unknown endpoint |
| `500` | `{ "error": "..." }` | Internal error — check server stderr |
| `502` | `{ "error": "...", ... }` | Upstream AI provider failure |

## Example — minimal client

```js
const BASE  = 'http://127.0.0.1:18789';
const TOKEN = process.env.HORIZON_TOKEN;

async function chat(message) {
  const r = await fetch(`${BASE}/api/chat`, {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${TOKEN}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ message }),
  });
  return r.json();
}

console.log(await chat('What time is it in Tokyo?'));
```

## Example — streaming agent in Python

```python
import json, requests

with requests.post(
    "http://127.0.0.1:18789/api/agent",
    headers={
        "Authorization": f"Bearer {TOKEN}",
        "Accept": "text/event-stream",
    },
    json={"task": "list files in the current workspace"},
    stream=True,
) as r:
    for line in r.iter_lines(decode_unicode=True):
        if line.startswith("data:"):
            print(json.loads(line[5:].strip()))
```

## Related

- [Running on a VPS](../deploy.md) — production deployment behind nginx
- [Mobile PWA](../mobile-pwa.md) — the PWA that talks to this API
- [CLI reference](../cli-reference.md) — `horizon serve` flags
