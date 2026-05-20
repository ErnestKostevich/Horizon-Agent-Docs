---
title: Local models
description: Run Horizon entirely on your own machine with Ollama, LM Studio, or LocalAI — no API spend, no data leaving the box.
lastUpdated: 2026-05-20
---

# Local models

Horizon's three local-provider integrations let you run the entire
stack on your own machine — no API keys, no cloud calls, no per-token
billing. Pick whichever runner you like and Horizon talks to it over
its existing OpenAI-compatible HTTP endpoint.

Three runners are supported out of the box:

- **Ollama** — easiest. `ollama pull <model>`, done.
- **LM Studio** — GUI-driven. Best for picking from a model catalogue
  and tuning sampling visually.
- **LocalAI** — Docker / self-hosted. Best for fleets, servers, and
  custom backends.

All three speak the same OpenAI-compatible `/v1/chat/completions` API,
so swapping between them is a single setting change.

## Ollama (recommended for laptops)

Ollama is the simplest path to a local model — single-binary, CLI,
auto-manages model downloads.

### Install

```bash
# macOS
brew install ollama

# Linux
curl -fsSL https://ollama.com/install.sh | sh

# Windows
# Download installer from https://ollama.com/download
```

### Pull a model

```bash
ollama pull llama3.1            # 4.7 GB, default in Horizon
ollama pull llama3.2:3b          # 2.0 GB, smaller for CPU-only
ollama pull qwen2.5-coder        # 4.5 GB, code-specialised
ollama pull deepseek-r1          # 4.7 GB, reasoning-tuned
ollama pull gemma2               # 5.5 GB, Google's open model
```

A 7-8B model fits comfortably on a 16 GB MacBook Pro / desktop with
8 GB GPU. For larger models (70B), you'll want >= 64 GB unified
memory or a serious dedicated GPU.

### Point Horizon at it

Ollama serves at `http://127.0.0.1:11434` by default. Horizon assumes
this URL out of the box — no setup needed.

```bash
# Switch Horizon to Ollama
horizon model ollama

# Use a specific model
horizon model ollama --model qwen2.5-coder

# Quick test
horizon ask "what's 12 factorial?" --provider ollama
```

#### Custom Ollama URL

If you run Ollama on a different port or another machine:

```bash
horizon set ollamaUrl http://192.168.1.42:11434
```

#### Inspect available models

```bash
ollama list

# Inside Horizon
horizon model ollama --list
```

> **Tip:** `ollama serve` runs in the background automatically when
> you `ollama run <model>`. If Horizon says "connection refused",
> start it explicitly with `ollama serve` and try again.

## LM Studio

LM Studio is a GUI app that's friendly for non-CLI users. Browse the
model catalogue, download with one click, tune sampling parameters
visually.

### Install

Download from [lmstudio.ai](https://lmstudio.ai) — installers for
macOS, Windows, Linux. Launch the app.

### Download + start a model

1. **Discover** tab → search for a model (e.g. "llama 3.1 8b instruct").
2. Click **Download**. Sizes are shown per quantisation level (`Q4_K_M`
   is a good default).
3. **Local Server** tab → select the downloaded model → **Start
   Server**. Default port is `1234`.

The LM Studio status bar should now say "Running on http://127.0.0.1:1234".

![LM Studio screenshot](./img/local-models-lmstudio.png)

### Point Horizon at it

```bash
horizon model lmstudio
horizon set lmStudioUrl http://127.0.0.1:1234   # default — only override if you changed it
```

That's it — the model id is whatever LM Studio is currently serving.

```bash
horizon ask "explain async/await" --provider lmstudio
```

#### Tuning

In LM Studio's Local Server tab, the right sidebar exposes:

- **Context Length** — how many tokens fit in a single request.
  Default is whatever the model was trained for (4k-128k typically).
- **GPU Offload** — number of layers offloaded to GPU. More = faster
  but more VRAM. LM Studio auto-detects a reasonable default.
- **Sampling** — temperature, top-p, top-k. Horizon passes its own
  values per request; LM Studio's UI values are the fallback for
  whatever Horizon doesn't specify.

## LocalAI

LocalAI is the heavy-duty option — a self-hosted server that supports
many backends (GGUF, GPTQ, AWQ, EXL2), runs in Docker, and exposes
the same OpenAI-compatible API.

### Quick start with Docker

```bash
docker run -p 8080:8080 \
  -v $PWD/models:/build/models \
  --name local-ai \
  localai/localai:latest
```

The server takes ~1 min to boot (loads the model registry). It's
ready when `curl http://127.0.0.1:8080/readyz` returns 200.

### Add a model

```bash
# Pull a GGUF from the catalogue
curl http://127.0.0.1:8080/models/apply \
  -H "Content-Type: application/json" \
  -d '{ "id": "llama3.1-instruct" }'
```

LocalAI downloads the GGUF and registers it. Verify:

```bash
curl http://127.0.0.1:8080/v1/models
```

### Point Horizon at it

```bash
horizon model localai
horizon set localAiUrl http://127.0.0.1:8080
horizon set localAiModel llama3.1-instruct
```

LocalAI is the right choice when you're running on a server, want
multi-user isolation, need an HTTP backend other tools can talk to,
or want to mix multiple models behind a router.

## Switching between local runners

All three providers store their URL in separate settings keys so
switching is a single command, no re-config:

```bash
horizon model ollama       # uses ollamaUrl
horizon model lmstudio     # uses lmStudioUrl
horizon model localai      # uses localAiUrl
```

Settings persist; the previous provider's URL stays in the store.

## Performance tips

Local inference quality and speed depend on three things, in order:

### 1. Model size vs RAM

Rough VRAM rule for 4-bit quantisation:

| Model | VRAM needed |
|---|---|
| 3B (Llama 3.2 3B, Gemma 2B) | 2–3 GB |
| 7-8B (Llama 3.1 8B, Mistral 7B) | 5–6 GB |
| 13B (Qwen2 14B) | 8–10 GB |
| 32B (Qwen2.5-Coder 32B) | 18–22 GB |
| 70B (Llama 3.3 70B) | 40+ GB |

CPU-only inference works but is ~5-10x slower than GPU. A modern
M-series Mac is the sweet spot for laptops — unified memory means a
14B model on a 32 GB M3 Pro runs comfortably.

### 2. Quantisation level

Lower bits = smaller + faster, at a quality cost:

| Level | Quality | Size (8B model) | Recommended for |
|---|---|---|---|
| `Q8_0` | Near-FP16 | ~8 GB | Quality-critical work |
| `Q5_K_M` | High | ~5.5 GB | Default for laptops |
| `Q4_K_M` | Good | ~4.5 GB | Balanced — Ollama default |
| `Q3_K_M` | Lossy | ~3.5 GB | RAM-constrained |
| `Q2_K` | Very lossy | ~2.5 GB | Demos only |

### 3. Context window

A 128k context model with 100k tokens loaded uses ~10x more memory
than the same model with 4k tokens. Cap context lower if you're
hitting OOM. In LM Studio: Settings → Context Length. In Ollama: not
exposed — uses the model's default; remove and re-pull with a
smaller `num_ctx` Modelfile if you need it.

## Combining with `--provider auto`

`--provider auto` walks a preference order. Local providers come
first — if Ollama is running, you'll never accidentally spend tokens
on Claude.

```
1. ollama   (if ollamaUrl reachable)
2. lmstudio (if lmStudioUrl reachable)
3. localai  (if localAiUrl reachable)
4. gemini   (free tier)
5. groq     (free tier)
6. cerebras (free tier)
...
```

```bash
horizon agent "review src/auth.js" --provider auto
# → falls through to ollama if you have a model loaded
```

Set in default: `horizon model auto` and Horizon does the right
thing per call.

## Limitations of local models

Local models in 2026 are good — not as good as Claude or GPT-5.4.
Things that consistently work less well:

- **Long-context reasoning** beyond ~16k tokens — even models with
  128k context "windows" degrade meaningfully past 16-32k.
- **Tool-calling reliability** — frontier models follow the
  tool-call protocol almost perfectly; 7B locals get it right ~80-90%
  of the time. Horizon's agent loop handles malformed responses
  gracefully but the round-trip costs latency.
- **Multi-turn instruction following** — staying on a complex
  multi-step instruction across many turns is harder for smaller
  models.
- **Code generation** — Claude / GPT-5 / Codex-class is noticeably
  ahead on code in 2026. Qwen2.5-Coder 32B locally is the closest;
  Llama 3.3 70B is in the same ballpark for analysis tasks.

Compensate by:

- Using `--provider auto` so smaller tasks go local + bigger ones go
  to a frontier model
- Bumping `--max-steps 12` for the agent loop with a local model
  (more iterations to recover from a malformed step)
- Switching to a code-specialised model (qwen2.5-coder) for
  coding tasks specifically

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `Connection refused` on Ollama | `ollama serve` not running | Start it manually: `ollama serve` |
| Replies extremely slow | Model didn't fit in VRAM, hit swap | Smaller model or lower-bit quantisation |
| `404` from LM Studio | No model loaded in Local Server tab | Click "Start Server" with a model selected |
| OOM crash | Context too big | Reduce context window in the runner |
| Garbage output | Wrong sampling for chat (e.g. temperature 0 with a base model) | Reset to defaults; use *Instruct* / *Chat* variants only |
| Tool calls never fire | Model too small or not instruction-tuned | Use 7B+ instruct models; avoid base / completion-only models |

### Diagnostic commands

```bash
# Show provider status with key/URL health
horizon model --list

# Direct test of local endpoint
curl http://127.0.0.1:11434/v1/models   # Ollama
curl http://127.0.0.1:1234/v1/models    # LM Studio
curl http://127.0.0.1:8080/v1/models    # LocalAI
```

## Related

- [Providers reference](../reference/providers-full.md) — full catalog
- [Choosing a provider](../providers.md) — overview
- [CLI reference](../cli-reference.md) — `horizon model` family
