---
title: Providers — full reference
description: All 25 AI providers Horizon talks to, with signup URLs, pricing, default model, available models, and setup steps.
lastUpdated: 2026-05-20
---

# Providers — full reference

Horizon talks directly to 25 different AI providers (plus 300+ models
reachable indirectly through LiteLLM). This page is the complete
catalogue: every provider, what to sign up for, what the costs look
like, and how to wire it up.

The shorter "which one should I pick" intro lives at
[Choosing a provider](../providers.md). This page is the deep dive —
read it when you need to pick a model or debug an error.

> **All providers are BYOK (Bring Your Own Key).** Horizon doesn't
> mark up tokens or proxy your traffic. You hold the key; the AI
> provider bills you directly.

## How to set a key

Three equivalent ways:

```bash
# CLI
horizon connect provider --<provider> <KEY>

# Desktop app
# Settings → Connections → <Provider> → paste key → Save

# Environment variable (one-off override)
ANTHROPIC_API_KEY=YOUR_KEY_HERE horizon ask "..."
```

Keys are stored encrypted in `horizon-keys.json` (AES-256-GCM,
machine-bound).

## Premium frontier models

### Anthropic Claude

- **Sign up:** [console.anthropic.com](https://console.anthropic.com)
- **Pricing tier:** Paid (no free tier)
- **Setting:** Settings → Connections → Claude
- **Env var:** `ANTHROPIC_API_KEY`
- **Default model:** `claude-sonnet-4-6`

Available models:

| Model | Tagline | Cost (input / output per M tokens) |
|---|---|---|
| `claude-sonnet-4-6` | Balanced flagship | $3 / $15 |
| `claude-opus-4-7` | Most capable, deep reasoning | $15 / $75 |
| `claude-haiku-4-5` | Fastest, cheapest | $0.80 / $4 |
| `claude-3-5-sonnet-latest` | Legacy stable | $3 / $15 |
| `claude-3-5-haiku-latest` | Legacy fast | $0.80 / $4 |

Best for: coding assistance, careful long-form writing, anything that
needs the model to follow nuanced instructions across many turns.

### OpenAI

- **Sign up:** [platform.openai.com](https://platform.openai.com)
- **Pricing tier:** Paid (small monthly free credit on new accounts)
- **Env var:** `OPENAI_API_KEY`
- **Default model:** `gpt-5.4`

Available models:

| Model | Tagline | Cost (per M) |
|---|---|---|
| `gpt-5.4` | Flagship | $1.25 / $10 |
| `gpt-5.4-mini` | Cheaper, fast | $0.15 / $0.60 |
| `gpt-5.3-codex` | Code-specialised | listed |
| `gpt-5.2` | Reliable older flagship | listed |
| `o4-mini` | Reasoning, cheaper | listed |
| `o3` | Deep reasoning | listed |
| `gpt-4o` | Multimodal legacy | $2.50 / $10 |
| `gpt-4o-mini` | Cheap legacy | $0.15 / $0.60 |

Best for: general-purpose, image input, fastest reasoning models (`o3`).

### xAI Grok

- **Sign up:** [console.x.ai](https://console.x.ai)
- **Pricing tier:** Paid (some free tier for new accounts)
- **Env var:** `XAI_API_KEY`
- **Default model:** `grok-4`

Available models:

| Model | Tagline |
|---|---|
| `grok-4` | Flagship |
| `grok-4-fast` | Fast variant |
| `grok-code-fast` | Code-specialised |
| `grok-3` | Legacy |

Best for: current-events queries (Grok has web access), Twitter/X data.

### Mistral

- **Sign up:** [console.mistral.ai](https://console.mistral.ai)
- **Pricing tier:** Paid (free tier with credit card validation)
- **Env var:** `MISTRAL_API_KEY`
- **Default model:** `mistral-large-latest`

Available models:

| Model | Tagline |
|---|---|
| `mistral-large-latest` | Flagship |
| `mistral-medium-latest` | Balanced |
| `mistral-small-latest` | Fast, cheap |
| `codestral-latest` | Code-specialised |
| `ministral-8b-latest` | Tiny, edge |

Best for: European data residency, multilingual.

## Free-tier providers (best for starting out)

### Google Gemini

- **Sign up:** [aistudio.google.com/apikey](https://aistudio.google.com/apikey)
- **Pricing tier:** Generous free tier — **1,500 requests/day** on Flash
- **Env var:** `GOOGLE_API_KEY`
- **Default model:** `gemini-2.5-flash`

Available models:

| Model | Tagline |
|---|---|
| `gemini-2.5-flash` | Free tier, fast |
| `gemini-3.1-pro` | Flagship |
| `gemini-3.1-flash` | Fast, cheaper |
| `gemini-3.0-pro` | Stable mid-tier |
| `gemini-2.5-pro` | 2M context |
| `gemini-1.5-pro` | Legacy 2M context |
| `gemini-1.5-flash` | Legacy fast |

Best for: getting started (free), long-context (1-2M tokens).

### Groq

- **Sign up:** [console.groq.com/keys](https://console.groq.com/keys)
- **Pricing tier:** Generous free tier with rate limits
- **Env var:** `GROQ_API_KEY`
- **Default model:** `llama-3.3-70b-versatile`

Available models:

| Model | Tagline |
|---|---|
| `llama-3.3-70b-versatile` | Flagship open |
| `llama-3.1-8b-instant` | Free tier, very fast |
| `openai/gpt-oss-120b` | Open weights |
| `moonshotai/kimi-k2-instruct` | Long-context |
| `mixtral-8x7b-32768` | Mixtral 8x7B |

Best for: speed (Groq's LPUs are *fast* — 500+ tokens/sec on small
models), Whisper transcription (also used by Horizon's voice loop).

### Cerebras

- **Sign up:** [cloud.cerebras.ai](https://cloud.cerebras.ai)
- **Pricing tier:** Generous free tier
- **Env var:** `CEREBRAS_API_KEY`
- **Default model:** `llama-3.3-70b`

Available models:

| Model | Tagline |
|---|---|
| `llama-3.3-70b` | Extremely fast inference |
| `llama3.1-8b` | Free tier, fastest |

Best for: extreme inference speed (Cerebras WSE chips do > 2000 tok/s
on small models). Best free option for latency-sensitive workflows.

## Cheap paid hosts (open models)

### DeepSeek

- **Sign up:** [platform.deepseek.com](https://platform.deepseek.com)
- **Pricing tier:** Paid (cheapest non-local — $0.14 / $0.28 per M)
- **Env var:** `DEEPSEEK_API_KEY`
- **Default model:** `deepseek-chat`

Available models:

| Model | Tagline | Cost |
|---|---|---|
| `deepseek-chat` | General-purpose | $0.14 / $0.28 |
| `deepseek-reasoner` | Reasoning-tuned | listed |

Best for: massive bulk inference on a budget.

### DeepInfra

- **Sign up:** [deepinfra.com](https://deepinfra.com)
- **Pricing tier:** Paid (~$0.23 / M for Llama 70B)
- **Env var:** `DEEPINFRA_API_TOKEN`
- **Default model:** `meta-llama/Llama-3.3-70B-Instruct`

Available models:

| Model | Tagline | Cost |
|---|---|---|
| `meta-llama/Llama-3.3-70B-Instruct` | Cheap | $0.23 / M |
| `meta-llama/Meta-Llama-3.1-405B-Instruct` | Massive |  |
| `Qwen/Qwen2.5-72B-Instruct` | Solid open |  |
| `deepseek-ai/DeepSeek-V3` | Latest DeepSeek |  |

Best for: cheap access to large open models.

### Fireworks AI

- **Sign up:** [fireworks.ai](https://fireworks.ai)
- **Pricing tier:** Paid
- **Env var:** `FIREWORKS_API_KEY`
- **Default model:** `accounts/fireworks/models/llama-v3p3-70b-instruct`

Available models:

| Model | Tagline |
|---|---|
| `accounts/fireworks/models/llama-v3p3-70b-instruct` | Flagship open |
| `accounts/fireworks/models/deepseek-v3` | Open MoE |
| `accounts/fireworks/models/qwen2p5-coder-32b-instruct` | Code |

Best for: low-latency open-model serving with custom fine-tunes.

### Together AI

- **Sign up:** [api.together.ai](https://api.together.ai)
- **Pricing tier:** Paid (small free credit)
- **Env var:** `TOGETHER_API_KEY`
- **Default model:** `meta-llama/Llama-3.3-70B-Instruct-Turbo`

Available models:

| Model | Tagline |
|---|---|
| `meta-llama/Llama-3.3-70B-Instruct-Turbo` | Fast, cheap |
| `meta-llama/Meta-Llama-3.1-405B-Instruct-Turbo` | Massive open |
| `mistralai/Mixtral-8x22B-Instruct-v0.1` | MoE |
| `Qwen/Qwen2.5-72B-Instruct-Turbo` | Solid open |

Best for: long catalog of open models, especially the "Turbo" speedups.

### SambaNova

- **Sign up:** [sambanova.ai](https://sambanova.ai)
- **Pricing tier:** Paid (some free credit for new accounts)
- **Env var:** `SAMBANOVA_API_KEY`
- **Default model:** `Meta-Llama-3.3-70B-Instruct`

Available models:

| Model | Tagline |
|---|---|
| `Meta-Llama-3.3-70B-Instruct` | Flagship |
| `Meta-Llama-3.1-405B-Instruct` | Massive |
| `Meta-Llama-3.1-8B-Instruct` | Fast, cheap |

Best for: SambaNova's RDU chips deliver competitive speeds on big
Llamas.

### Nebius AI

- **Sign up:** [studio.nebius.ai](https://studio.nebius.ai)
- **Pricing tier:** Paid
- **Env var:** `NEBIUS_API_KEY`
- **Default model:** `meta-llama/Meta-Llama-3.1-70B-Instruct`

Available models:

| Model |
|---|
| `meta-llama/Meta-Llama-3.1-70B-Instruct` |
| `meta-llama/Meta-Llama-3.1-405B-Instruct` |
| `Qwen/Qwen2.5-72B-Instruct` |

Best for: European-region hosting, NVIDIA-backed cluster.

### Alibaba Qwen

- **Sign up:** [dashscope.aliyun.com](https://dashscope.aliyun.com) (or `dashscope-intl` for non-CN)
- **Pricing tier:** Paid (limited free for some models)
- **Env var:** `DASHSCOPE_API_KEY`
- **Default model:** `qwen-plus`

Available models:

| Model | Tagline |
|---|---|
| `qwen-plus` | Balanced flagship |
| `qwen-max` | Most capable |
| `qwen-turbo` | Fast, cheap |
| `qwen2.5-coder-32b-instruct` | Code-specialised |

Best for: Chinese-language tasks, code (Qwen2.5-Coder).

### Moonshot (Kimi)

- **Sign up:** [platform.moonshot.ai](https://platform.moonshot.ai)
- **Pricing tier:** Paid
- **Env var:** `MOONSHOT_API_KEY`
- **Default model:** `kimi-k2-0905-preview`

Available models:

| Model | Tagline |
|---|---|
| `kimi-k2-0905-preview` | Flagship long-context |
| `moonshot-v1-128k` | 128k context |
| `moonshot-v1-8k` | 8k context, cheaper |

Best for: long-document analysis (Kimi K2 handles 200k tokens cleanly).

### Z.AI (GLM)

- **Sign up:** [open.bigmodel.cn](https://open.bigmodel.cn)
- **Pricing tier:** Paid
- **Env var:** `ZHIPU_API_KEY`
- **Default model:** `glm-4-plus`

Available models:

| Model | Tagline |
|---|---|
| `glm-4-plus` | Flagship |
| `glm-4` | Base |
| `glm-4-flash` | Fast, cheap |

Best for: Chinese-language; cheap alternative to GPT-4o.

## Specialised

### Perplexity

- **Sign up:** [perplexity.ai](https://www.perplexity.ai)
- **Pricing tier:** Paid
- **Env var:** `PERPLEXITY_API_KEY`
- **Default model:** `sonar-pro`

Available models:

| Model | Tagline |
|---|---|
| `sonar-pro` | With web search |
| `sonar` | Cheaper, with web search |
| `sonar-reasoning-pro` | Reasoning + search |

Best for: queries that need current web data baked in (every Perplexity
call does retrieval + cites sources).

### Cohere

- **Sign up:** [cohere.com](https://cohere.com)
- **Pricing tier:** Paid (trial key)
- **Env var:** `COHERE_API_KEY`
- **Default model:** `command-a-03-2025`

Available models:

| Model | Tagline |
|---|---|
| `command-a-03-2025` | Flagship |
| `command-r-plus` | RAG-tuned |
| `command-r` | Cheaper, RAG |

Best for: retrieval-augmented generation (Cohere's models are
specifically tuned for it) and re-ranking workflows.

## Aggregators

### OpenRouter

- **Sign up:** [openrouter.ai](https://openrouter.ai)
- **Pricing tier:** Pay-as-you-go across 300+ models behind one key
- **Env var:** `OPENROUTER_API_KEY`
- **Default model:** `openai/gpt-5.4-mini`

A selection of routable models:

| Model | Notes |
|---|---|
| `openai/gpt-5.4-mini` | Router default |
| `anthropic/claude-sonnet-4-6` | Claude via OR |
| `anthropic/claude-opus-4-7` | Opus via OR |
| `google/gemini-3.1-pro` | Gemini via OR |
| `meta-llama/llama-3.3-70b-instruct` | Open via OR |
| `deepseek/deepseek-chat` | DeepSeek via OR |
| `x-ai/grok-4` | Grok via OR |

Best for: trying many models without signing up for each (you pay OR's
small markup but skip per-vendor account setup). Useful for plugins
that want to support "any model the user has via OR".

### LiteLLM router

- **Sign up:** Self-host or use OR-style proxy
- **Pricing tier:** N/A (router only)
- **Env var:** depends on backend keys
- **Default model:** `openai/gpt-4o-mini`

LiteLLM is a routing layer — model id `provider/model` resolves to the
matching real provider. Horizon supports it as a "virtual provider" so
you can address 300+ models with one consistent prefix:

| Model id | Resolves to |
|---|---|
| `openai/gpt-4o-mini` | OpenAI direct |
| `anthropic/claude-sonnet-4-6` | Claude direct |
| `gemini/gemini-3.1-pro` | Gemini direct |
| `groq/llama-3.3-70b-versatile` | Groq direct |

Best for: complex routing rules (e.g. "use Claude for code, Gemini for
everything else") implemented in LiteLLM's config rather than Horizon.

See [docs.litellm.ai](https://docs.litellm.ai) for the router setup.

### Azure OpenAI

- **Sign up:** Azure portal + OpenAI service request
- **Pricing tier:** Pay-as-you-go (enterprise pricing)
- **Env var:** `AZURE_OPENAI_KEY`
- **Default model:** Whatever you named your deployment

Setup:

```bash
horizon set azureEndpoint https://my-resource.openai.azure.com
horizon set azureDeployment my-gpt-5-deployment
horizon set azureApiVersion 2024-10-21
horizon connect provider --azure <KEY>
```

Best for: enterprise scenarios with data-residency / compliance
requirements that the public OpenAI API can't satisfy.

### Custom (OpenAI-compatible)

- **Sign up:** Whatever endpoint you control
- **Env var:** none required if endpoint is open
- **Default model:** Set with `horizon set customModel <id>`

Setup:

```bash
horizon set customUrl https://my-private-endpoint.example.com
horizon set customModel my-fine-tune-v1
horizon connect provider --custom <KEY_IF_REQUIRED>
```

Best for: self-hosted vLLM / TGI / TensorRT-LLM servers, internal
inference endpoints, or any "looks like OpenAI" API not in this list.

## Local (no key required)

### Ollama, LM Studio, LocalAI

All three are documented in [Local models](../guides/local-models.md).
Quick summary:

| Provider | Default URL | Recommended for |
|---|---|---|
| `ollama` | `http://127.0.0.1:11434` | Simplest CLI runner |
| `lmstudio` | `http://127.0.0.1:1234` | GUI-driven model picking |
| `localai` | `http://127.0.0.1:8080` | Docker / self-hosted server |

Setup:

```bash
horizon model ollama         # uses default URL
horizon set ollamaUrl http://192.168.1.42:11434  # override
```

## Smart auto routing

`--provider auto` walks this order, picking the first one with a
configured key (or a reachable local URL):

```
1.  ollama, lmstudio, localai   (local — free)
2.  gemini, groq, cerebras       (free tier)
3.  deepinfra, deepseek, fireworks, together,
    sambanova, nebius, moonshot, mistral, zai  (cheap paid)
4.  openrouter                   (aggregator)
5.  claude, openai, grok         (premium)
```

Set once: `horizon model auto`. Or per-call: `--provider auto`.

See [Smart auto routing](../auto-routing.md) for the full algorithm.

## Cost tracking

Every call is logged with token counts + dollar estimate (via the
provider's published list price):

```bash
horizon cost                     # last 30 days, bar chart
horizon cost --days 7
horizon cost --json | jq .total
```

## Common errors

| Error | Likely cause | Fix |
|---|---|---|
| `<provider> key not set` | No key configured | `horizon connect provider --<provider> <KEY>` |
| `401 invalid api key` | Wrong key or revoked | Re-paste; verify on provider's dashboard |
| `429 rate limit exceeded` | Free tier daily cap | Wait or switch with `--provider auto` |
| `model not found` | Wrong model name (typo, deprecated) | `horizon model --list`; pick from catalog |
| `context length exceeded` | Sent more tokens than model accepts | Switch model, trim history (`horizon checkpoints save` then start fresh) |
| `connection refused` (local) | Runner not started | Start Ollama / LM Studio / LocalAI |
| `Azure endpoint not set` | Missing `azureEndpoint` | `horizon set azureEndpoint ...` |
| `Custom URL not set` | Missing `customUrl` | `horizon set customUrl ...` |

## Related

- [Choosing a provider](../providers.md) — the short version
- [Local models](../guides/local-models.md) — Ollama / LM Studio /
  LocalAI deep dive
- [Cost tracking](../cost.md) — `horizon cost` reference
- [Smart auto routing](../auto-routing.md) — `--provider auto` details
