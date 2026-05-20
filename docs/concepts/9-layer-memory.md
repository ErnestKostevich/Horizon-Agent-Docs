---
title: 9-layer memory
description: How Horizon remembers — facts, episodic, conversations, semantic embeddings, FTS, profile, persona, workspace, and dialectic.
lastUpdated: 2026-05-20
---

# The 9-layer memory architecture

Horizon doesn't have one "memory" — it has nine, each tuned for a
different kind of remembering. When you ask "what was that yerba mate
brand I liked?", the retrieval pipeline consults all of them and ranks
the candidates together.

This page walks each layer top-to-bottom: what it stores, how it's
written, how it's read, and where it lives on disk.

## Why nine layers?

Each layer fixes a failure mode of the layer above it:

- A flat key-value store of "facts" forgets when you said something.
- Adding timestamped "memories" loses context across a long chat.
- Saving full "conversations" doesn't scale to fuzzy lookup.
- Adding embeddings handles fuzzy lookup but is slow + costs API calls.
- Adding FTS gives instant lexical matches but misses synonyms.
- Adding a Big Five "profile" captures stable traits FTS can't index.
- Persona-tagged memory lets each voice keep its own knowledge.
- Workspace memory pins per-project rules outside the chat.
- A dialectic model captures evolving beliefs that don't fit any of
  the above.

Each layer is independent — drop any one and the others still work.

## Layer 1 — Facts (key-value)

The simplest store. `{ key: value }` written by `remember` tools and
the `learnFromTurn` extractor.

```js
// Anatomy
facts: {
  "favourite_coffee": "Tim Hortons double-double",
  "city": "Vilnius",
  "wake_time": "08:30"
}
```

Use it for stable, single-value preferences. Querying:

```bash
horizon mem search "favourite coffee"
```

Storage: `AgentMemory._data.facts` mirrored to SQLite `facts` table + FTS5 index.

## Layer 2 — Episodic memories

Timestamped bullets of "what happened". The main long-term store.

```js
memories: [
  {
    id: "m_abc",
    text: "User finished the auth refactor at 23:11.",
    created: 1747000000000,
    importance: 0.6,
    tags: ["work", "milestone"],
    personaId: "jarvis",
    source: "chat"
  }
]
```

Written by the agent at the end of any turn the LLM flags as worth
remembering, by the manual `/remember` slash command, and by the
`learnFromTurn` extractor running over each completed conversation.

Recall combines keyword score + recency + importance + persona match.

Storage: JSON file + SQLite `memories` table + FTS5 index +
embeddings sidecar.

## Layer 3 — Conversation transcripts

Full message history per session. Available for re-summarisation when
you `/checkpoint restore` or run `horizon insights`.

```js
conversations: [
  {
    id: "c_def",
    role: "user",
    content: "Can you summarise yesterday's standup?",
    created: 1747000000000,
    personaId: "alfred"
  },
  // ...
]
```

This layer is the source of truth for retrieval — Layer 2 (memories)
is essentially an LLM-curated summary of Layer 3.

Storage: SQLite `conversations` table with FTS5.

## Layer 4 — Semantic embedding index

Every memory + every fact + every conversation chunk is embedded with
the active embedding provider (OpenAI `text-embedding-3-small` by
default, falls back to Gemini, then a local model). The vectors live
in a sidecar JSON file:

```
~/.horizon/memory.embeddings.json
```

Recall computes cosine similarity between the query embedding and
candidates. Combined with Layer 5 (FTS) via Reciprocal Rank Fusion.

```bash
# Force an embedding refresh
horizon mem reindex
```

Status:

```bash
horizon mem stats
# embeddings: 1247 vectors, dim=1536, provider=openai
```

If no embedding provider key is set, this layer is skipped and recall
falls back to FTS + recency.

## Layer 5 — Full-text search (FTS5)

SQLite FTS5 virtual tables over memories, facts, and conversations.
BM25 ranking, intersection across columns, instant lookup.

```sql
-- What runs under the hood
SELECT m.* FROM memories m
JOIN memories_fts f ON m.rowid = f.rowid
WHERE f MATCH ? ORDER BY bm25(f) LIMIT 20;
```

Used in parallel with semantic search and re-ranked together. FTS
catches exact-phrase matches that embeddings sometimes miss ("user
said 'yerba mate' verbatim"), embeddings catch synonyms FTS misses
("the South American tea I drink").

Storage: SQLite `*_fts` tables.

## Layer 6 — User profile (Big Five + style)

A Honcho-style stable model of who you are. Edited manually (Inspector
→ Learned → Profile) and nudged by `learnFromTurn` when it detects
strong signals.

```js
userProfile: {
  bigFive: {
    openness: 0.78, conscientiousness: 0.65,
    extraversion: 0.32, agreeableness: 0.60, neuroticism: 0.41
  },
  communicationStyle: {
    formality: 'casual',
    verbosity: 'medium',
    preferredAddress: 'Ernest',
    lang: 'en'
  },
  expertise: [
    { topic: "TypeScript", level: "expert", noticed: "2026-03-12" }
  ],
  goals: [
    { goal: "ship horizon-genesis v1", priority: "high", addedAt: "2026-05-01" }
  ],
  preferences: { coffee: "strong + black" },
  confidence: 0.72
}
```

This isn't searched like Layer 2 — it's *injected into the system
prompt every turn* so the model can adapt tone without a retrieval
hop.

```bash
horizon mem profile
```

Storage: `AgentMemory._data.userProfile` in the JSON file.

## Layer 7 — Per-persona memory overlay

Each persona (Jarvis / Friday / Alfred / Sage / Pixel / your custom
ones) has its own slice of long-term memory. Every memory is tagged
with the persona id of the active persona at write time.

When the active persona is, say, Alfred, recall multiplies the score
of memories tagged `personaId: "alfred"` by 1.2x. Other personas'
memories are still candidates, just lower-ranked.

```bash
horizon persona show alfred
# Dumps Alfred's personal notes
```

This lets you keep distinct knowledge bases per voice — Alfred
remembers dinner reservations, Sage remembers your reading list —
without managing separate accounts.

See [Personas](./personas.md) for the full overlay schema.

Storage: `personaId` column on `memories` + persona overlay store
under `settingsStore.customPersonas`.

## Layer 8 — Workspace memory

Per-project memory. Lives in `.horizon/memory.json` in the current
workspace and only loads when Horizon's `cwd` is inside that
directory.

```
my-project/
├── .horizon/
│   ├── memory.json     ← Layer 8 store
│   ├── rules.md         ← agent rules
│   └── skills/          ← workspace-scoped skills
├── src/
└── package.json
```

Why: facts about *this* repo (test command, deploy target, style
guide) shouldn't pollute your global memory; rules unique to *this*
project shouldn't apply to chat in your home directory.

```bash
horizon ws show
# .horizon/memory.json contents

horizon ws memory edit
# Opens the file in $EDITOR
```

The agent system prompt for any turn run from inside a workspace
includes the workspace memory + rules verbatim. Switching `cwd`
swaps the active workspace.

Storage: `.horizon/memory.json` (the file you `git commit` or
`.gitignore` per your choice).

## Layer 9 — Dialectic user model

The newest layer. A file-backed representation of your beliefs,
desires, knowledge state, and theory of mind — extracted by an LLM
from your turns rather than written by you.

```js
dialectic: {
  beliefs: [
    { text: "User believes TypeScript is more maintainable than JS at >5kLOC", confidence: 0.81 },
    { text: "User dislikes verbose corporate writing", confidence: 0.76 }
  ],
  desires: [
    { text: "Wants horizon-genesis to be a sustainable solo project", confidence: 0.9 }
  ],
  knowledge: [
    { topic: "Node.js streams", level: "expert", evidence: "explained backpressure correctly" }
  ],
  theoryOfMind: [
    { text: "User assumes the reader knows what BSL is", confidence: 0.6 }
  ],
  updatedAt: 1747000000000
}
```

Runs after each non-trivial turn (`dialecticExtract`). Slow — it's a
full LLM call — so it's gated to turns that look like genuine
disclosure (not "what time is it").

Inject into the prompt:

```
{persona system prompt}
{user profile}
{dialectic relevant slices}
{retrieved memories}
{conversation}
```

Storage: `AgentMemory._data.dialectic` in the JSON file, written by
`dialecticModel.js`.

## How retrieval combines them

When you ask the agent something, the retrieval pipeline runs roughly:

1. **Query embedding** — embed the user's message
2. **Semantic candidates** (Layer 4) — top 20 by cosine similarity
3. **FTS candidates** (Layer 5) — top 20 by BM25
4. **Persona boost** (Layer 7) — multiply by 1.2 if persona matches
5. **Recency decay** — newer memories weigh more
6. **Importance** — high-importance memories weigh more
7. **Reciprocal rank fusion** — combine semantic + FTS rankings
8. **Inject into context** — top N (typically 5-10) into the system
   prompt alongside the persona, user profile, and workspace memory.

```
┌─────────────────────────────────────────────────────────────┐
│              System prompt assembly                         │
├─────────────────────────────────────────────────────────────┤
│  Persona system prompt              ← Layer 7 (style)       │
│  User profile (Big Five + prefs)    ← Layer 6               │
│  Dialectic slices (relevant)        ← Layer 9               │
│  Workspace memory + rules           ← Layer 8               │
│  Retrieved memories (top N)         ← Layers 2 + 4 + 5 + 7  │
│  Recent conversation                ← Layer 3               │
│  Current turn                                               │
└─────────────────────────────────────────────────────────────┘
```

## Inspection

```bash
horizon mem stats
# Layer-by-layer counts, embedding status, last index time

horizon mem dump --type memories
# NDJSON export of Layer 2

horizon mem dump --type facts
# Export of Layer 1

horizon mem profile
# Layer 6

horizon ws show
# Layer 8 contents
```

The desktop GUI's **Inspector → Learned** panel shows everything at
once with colour-coded provenance: chat-source, agent-source,
manual-edit, profile-extract, dialectic.

## File locations

- macOS: `~/Library/Application Support/horizon-genesis/`
- Windows: `%APPDATA%\horizon-genesis\`
- Linux: `~/.config/horizon-genesis/`

Inside:
- `memory.json` — Layers 1, 2, 3 (mirror), 6, 9
- `memory.db` — Layers 1, 2, 3, 5 (SQLite + FTS5)
- `memory.embeddings.json` — Layer 4
- `<workspace>/.horizon/memory.json` — Layer 8 (per-project)
- `settingsStore.customPersonas` (in `config.json`) — Layer 7 overlays

The JSON file is human-editable; the SQLite + embeddings sidecar are
derived stores that get rebuilt if you delete them.

## Limitations

- **Embedding provider required for Layer 4** — without an OpenAI /
  Gemini / Cohere key set, semantic search is off and recall falls
  back to FTS + recency.
- **Dialectic is slow** — it's a per-turn LLM call. You can disable
  it in Settings → Memory → Dialectic if cost matters.
- **Workspace memory doesn't sync** — it's per-machine. Commit
  `.horizon/memory.json` to git if you want team-shared workspace
  memory; you'll lose the `git-ignore-private-stuff` benefit.

## Related

- [Personas](./personas.md) — Layer 7's overlay store
- [HTTP API](../reference/http-api.md) — `/api/memories`, `/api/mem/*`
- [Workspace memory](../workspace-memory.md) — Layer 8 deep dive
- [CLI reference](../cli-reference.md) — `horizon mem *` commands
