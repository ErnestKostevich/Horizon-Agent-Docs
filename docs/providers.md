# Choosing a model provider

Horizon supports 25 direct providers + 300+ models through LiteLLM
routing. For a new user the **shortest path to a working setup**:

## If you just want it to work cheaply

1. **Google Gemini** — 1,500 free requests/day → [aistudio.google.com](https://aistudio.google.com/apikey)
2. **Groq** — free tier with very fast inference → [console.groq.com](https://console.groq.com/keys)

Either gets you running in under 2 minutes.

## If you want maximum quality (paid)

| Use case | Recommended |
|---|---|
| Coding assistance | Claude Sonnet 4.6 |
| Long context (books) | Gemini 2.5 Pro |
| Reasoning hard problems | OpenAI o3 / DeepSeek-Reasoner |
| Cheapest non-local | DeepSeek Chat ($0.14/M input) |
| Fastest inference | Cerebras Llama 70B |
| Multilingual | Mistral Large |

## If you want zero API spend

Run **Ollama** locally:
1. Install from [ollama.com](https://ollama.com)
2. `ollama pull llama3.1`
3. Horizon → Settings → Models → provider = Ollama

For a Llama 3.1 8B model on a modern laptop you get ~30 tokens/sec on
CPU, ~150 on GPU.

## All 25 direct providers

claude · openai · gemini · groq · deepseek · grok · mistral · qwen ·
perplexity · cohere · openrouter · together · fireworks · deepinfra ·
cerebras · sambanova · moonshot · zai · nebius · azure · custom ·
ollama · lmstudio · localai · litellm

For the full provider list with default models + cost per million
tokens, see [providers-full.md](providers-full.md).

## Smart routing

Don't want to decide? Use `--provider auto`. Horizon walks this order:

1. Local Ollama / LM Studio (if URL configured)
2. Free-tier providers (Gemini, Groq, Cerebras)
3. Cheap paid (DeepInfra, DeepSeek, Fireworks, ...)
4. Aggregator (OpenRouter)
5. Premium (Claude, OpenAI)

First match with a configured key wins. Set Claude as the only key →
auto falls through to Claude. Add Gemini → auto uses Gemini for free.

See [auto-routing](auto-routing.md) for the full algorithm.

## Switching mid-conversation

You can change provider per-call:

```bash
horizon chat "..." --provider claude --model claude-haiku-4-5
horizon agent "..." --provider auto
```

Or globally:

```bash
horizon model claude --model claude-sonnet-4-6
```

Or in the desktop: Settings → Models → pick.
