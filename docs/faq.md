# FAQ

## Is Horizon free?

There's a 5-day free trial when you install the desktop app or CLI.
After that you need a subscription to use commercial features. Personal
non-commercial use stays evaluable for free under the BSL license — see
[pricing](pricing.md).

If you only run local models with Ollama and never touch a paid
provider, Horizon itself is the only paid component (your "compute"
is free).

## Where are my API keys stored?

In a file on your machine:
- Windows: `%APPDATA%\horizon-ai\horizon-keys.json`
- macOS: `~/Library/Application Support/horizon-ai/horizon-keys.json`
- Linux: `~/.config/horizon-ai/horizon-keys.json`

Encrypted with AES-256-GCM using a key derived from your machine ID
(hostname + platform + CPU model). The encrypted file can't be read
on a different machine.

## Where is my conversation history stored?

Same folder: `horizon_memory.json` (text), `horizon_embeddings.json`
(vector index), `horizon_chats.json` (multi-chat).

Nothing leaves your machine unless you explicitly send a message — at
which point only that message goes to whatever AI provider you've
chosen (and not to any Horizon server).

## Can Horizon read all my files?

Only when you explicitly grant it. Read tools (`read_file`,
`list_dir`, `search_files`) run without prompt by default but only
within your current **workspace** (the folder you opened). Writing
files, running shell, or moving files all gate behind a per-call
permission prompt unless you pass `--auto-approve`.

The `.horizon/rules.md` file in your workspace lets you set
project-specific restrictions ("never modify files in `production/`").

## What happens if I lose my computer?

- API keys: lost (the encryption key is machine-bound). You'll need
  to re-enter them on the new machine.
- Memory + chats: also lost unless you ran `horizon backup snapshot`
  beforehand (creates a snapshot under `~/.config/horizon-ai/backups/`).
  You can also commit `.horizon/memory.json` to git for workspace-bound
  knowledge.
- Skills + plugins: re-installable from the marketplace.

We recommend `horizon backup snapshot` weekly + the workspace
`.horizon/` folder committed in git.

## Can I use Horizon offline?

Yes, if you use a **local provider** (Ollama, LM Studio, LocalAI). The
agent, memory, skills, executor, and computer use all work fully
offline. Only the AI completion call needs network — and a local
model removes even that.

## What's the difference between Chat and Agent mode?

| Mode | What it does | Cost |
|---|---|---|
| **Chat** | Single-turn reply, pure LLM | Cheap |
| **Agent** | Plans, runs tools, reflects, retries up to 8 times | 3-5× more tokens |

Use Chat for questions, Agent for tasks that need *doing*. See
[agent mode](agent-mode.md).

## Does Horizon work with my favourite model?

Probably yes. Horizon supports 25 direct providers + 300+ models
through the LiteLLM-style router:

Claude · OpenAI · Gemini · Groq · DeepSeek · Mistral · Qwen ·
Perplexity · Cohere · Grok · OpenRouter · Together AI · Fireworks ·
DeepInfra · Cerebras · SambaNova · Moonshot Kimi · Z.AI/GLM ·
Nebius · Azure OpenAI · custom OpenAI-compatible · Ollama · LM Studio
· LocalAI.

If your model isn't in this list, try `--provider litellm --model
"<provider-prefix>/<model>"` — see [LiteLLM router](litellm.md).

## How do I switch models?

```bash
horizon model claude --model claude-sonnet-4-6
```

Or in the desktop app: Settings → Models → pick a provider + model.

Or use auto-routing for the cheapest available:

```bash
horizon chat "..." --provider auto
```

## Can I run Horizon on my server?

Yes — see [running on a VPS](deploy.md). Install the CLI binary, run
`horizon serve --port 18789 --token mysecret`, point a mobile PWA or
a cron job at it.

## How is Horizon different from ChatGPT or Claude.ai?

Local-first. Bring your own keys. Memory + plugins + voice + computer
use are all built in. See [What is Horizon?](what-is-horizon.md) for
the comparison table.

## How do I get help?

- Read these docs: [horizonaai.dev/docs](https://horizonaai.dev/docs)
- File a GitHub issue: [horizon-genesis/issues](https://github.com/ErnestKostevich/horizon-genesis/issues)
- Email Ernest: ernest2011kostevich@gmail.com

## How do I cancel my subscription?

In the desktop app: Settings → Account → Manage subscription. Cancels
at the end of the current period; data stays on your machine.

## Can I host Horizon for my team?

The BSL license requires a commercial license for redistributing or
deploying Horizon as a service for multiple users. Contact
ernest2011kostevich@gmail.com for team/enterprise pricing.
