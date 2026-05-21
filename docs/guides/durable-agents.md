---
title: Durable agents — Kanban board
sidebar_position: 35
description: Long-running spawned subagents that survive crashes, reboots, and worker failures — backed by a SQLite Kanban queue with heartbeat reclamation.
lastUpdated: 2026-05-21
---

# Durable agents — Kanban board

Sprint 7C replaced Horizon's ephemeral `spawn_subagent` tool with a
durable, SQLite-backed task queue. Every subagent is now a row on a
**Kanban board** with statuses (queued → running → done), heartbeats,
and automatic reclamation when a worker dies.

This means you can:

- Spawn 20 subagents and walk away — they finish on their own and
  results stay around
- See what's running and what's queued in a single board view
- Reboot mid-run without losing work
- Cancel a runaway task without killing the parent

This page walks the runtime, the tools, and the CLI commands.

## Why durable?

The old `spawn_subagent` lived in process memory. When the parent
agent finished or the process died, every running subagent died with
it. That's fine for "do these three things in parallel and tell me",
but useless for "run this on a VPS overnight and email me when done".

The Kanban queue fixes both failure modes:

1. **Persistence.** Tasks are SQLite rows. Reboot the machine, restart
   `horizon serve`, and the queue picks up where it left off.
2. **Reclamation.** Each running task heartbeats every 5 s. If a
   heartbeat is missing for >60 s, the watchdog flips the task back to
   `queued`, increments the attempts counter, and another worker
   picks it up.

## The three subagent tools

A running agent can spawn subagents through these built-in tools:

### `spawn_subagent`

Enqueue a new task. Returns a task id immediately — does not block.

```js
// Inside a skill or tool call
await ctx.callTool('spawn_subagent', {
  task: 'Search the codebase for unused imports in *.ts files',
  priority: 5,                // 1-10, higher = sooner
  provider: 'auto',
  model: 'claude-haiku-4-5',  // cheaper model for grep work
  persona: 'jarvis',
  maxSteps: 4,
});
// → { taskId: 'k_a1b2c3d4...' }
```

### `subagent_status`

Poll a task's status without blocking.

```js
await ctx.callTool('subagent_status', { taskId: 'k_a1b2c3d4...' });
// → { status: 'running', progress: 0.4, startedAt: 1716239900000 }
```

### `subagent_wait`

Block until a task finishes (or times out). Returns the full result.

```js
const result = await ctx.callTool('subagent_wait', {
  taskId: 'k_a1b2c3d4...',
  timeoutMs: 120_000,
});
// → { status: 'done', answer: '...', steps: [...], usage: {...} }
```

The parent agent can spawn N subagents in parallel, then `wait` on
all of them and merge results. The Kanban scheduler runs them
concurrently — bounded by `HORIZON_KANBAN_WORKERS` (default 4).

## CLI commands

The `horizon agents` subcommand surfaces the board for the human.

### `horizon agents board`

ASCII three-column view: queued, running, done. Newest first per
column. Refreshes once per call — wrap in `watch` for a live view.

```bash
horizon agents board
#
# ╭─ Horizon Kanban ─────────────────────────────────────────────────╮
# │ QUEUED (3)          │ RUNNING (2)         │ DONE (12)            │
# │ ─────────────────── │ ─────────────────── │ ─────────────────── │
# │ summarise overnight │ search TODOs in src │ explain stack trace  │
# │ scrape news headlin │ run security scan   │ generate README      │
# │ translate docs/en   │                     │ rebuild test fixture │
# ╰──────────────────────────────────────────────────────────────────╯

watch -n 2 horizon agents board   # macOS/Linux live refresh
```

### `horizon agents list`

Flat list, newest first. Filter by status.

```bash
horizon agents list
horizon agents list --status running
horizon agents list --status failed --json
```

### `horizon agents show <id>`

Full task detail including the full prompt, all tool calls, the
final answer (or error), token usage, and timing.

```bash
horizon agents show k_a1b2c3d4
```

### `horizon agents cancel <id>`

Cancel a queued or running task. Running tasks stop at their next
heartbeat tick (typically <5 s).

```bash
horizon agents cancel k_a1b2c3d4
horizon agents stop k_a1b2c3d4    # alias
```

### `horizon agents purge`

Wipe terminal-state rows older than N days (default 30). Keeps the
queue lean on long-running deployments.

```bash
horizon agents purge --older 7
```

### `horizon agents stats`

Per-status counts.

```bash
horizon agents stats
# → queued: 3, running: 2, done: 142, failed: 4, cancelled: 1
```

### `horizon agents reclaim`

Manually sweep stale running rows. Normally the watchdog runs this
every 30 s; the manual command is useful after a worker crash you
want to recover from immediately.

```bash
horizon agents reclaim
```

## Statuses

Every task moves through these states:

| Status | Meaning |
|---|---|
| `queued` | Waiting to be claimed by a worker |
| `running` | Currently executing; heartbeat refreshed every 5 s |
| `done` | Completed successfully; result JSON in `result` column |
| `failed` | Execution threw; error message in `error` column |
| `abandoned` | Running task with no heartbeat >60 s; auto-requeued by `reclaim()` |
| `cancelled` | User-initiated abort via `cancel` |

## Reclamation in action

When a worker dies mid-task (process kill, machine reboot, network
outage), here's what happens:

1. The task's `heartbeat_at` stops updating.
2. After 60 s with no heartbeat, the watchdog flips the row from
   `running` back to `queued` and increments `attempts`.
3. The next idle worker claims the row and starts over.
4. After `attempts >= 3`, the row goes to `failed` so we don't loop
   forever on a poisoned task.

You can see how many times a task has been retried in the show view:

```bash
horizon agents show k_a1b2c3d4 | grep attempts
# attempts: 2
```

## Disabling the queue

If you don't want SQLite or `better-sqlite3` won't compile on your
target platform:

```bash
HORIZON_KANBAN=off horizon serve
# Or in your shell profile
export HORIZON_KANBAN=off
```

`spawn_subagent` falls back to the legacy in-memory implementation.
Tasks won't survive crashes, but the API stays the same.

## Where to go next

- [Agent mode](../agent-mode.md) — when to use spawned subagents
- [Server mode](../server-mode.md) — running the queue on a VPS
- [HTTP API](../reference/http-api.md) — the queue is also exposed
  over JSON-over-HTTP
- [Full CLI commands](../reference/cli-commands.md)
