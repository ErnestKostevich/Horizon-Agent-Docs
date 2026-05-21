---
title: Plugin development — quick intro
sidebar_position: 50
description: A 60-second intro to building Horizon plugins — pointer to the official SDK, the canonical execute() form, and the marketplace flow.
lastUpdated: 2026-05-21
---

# Plugin development — quick intro

Horizon plugins extend the agent with **new tools** and **new
connectors**. The agent itself, personality system, memory, and
computer use stay in core — plugins add the verbs.

This page is a 60-second intro. The full developer documentation,
TypeScript types, scaffolder, and examples live in the official
SDK repo:

> **Plugin SDK:** [github.com/ErnestKostevich/horizon-plugin-sdk](https://github.com/ErnestKostevich/horizon-plugin-sdk)

## A complete plugin in 30 lines

The canonical entry point is a single `execute()` function that
receives a tool name, an args object, and a context object, and
returns whatever the tool produces:

```javascript
// main.js — built into handler.js inside the .hzplugin zip
'use strict';

module.exports = {
  async execute(tool, args, ctx) {
    if (tool === 'currency_convert') {
      const { amount, from, to } = args;
      const url = `https://api.exchangerate.host/convert?from=${from}&to=${to}&amount=${amount}`;
      const res = await fetch(url);
      const data = await res.json();
      return { converted: data.result, rate: data.info.rate };
    }
    throw new Error(`Unknown tool: ${tool}`);
  },
};
```

That's the entire plugin runtime contract. Everything else — manifest,
TypeScript types, scaffolder, marketplace publish flow — is dev-time
ergonomics on top of this contract.

## Plugin bundle layout

```
my-plugin/
  manifest.json       — id, version, permissions, tool list
  main.js             — module.exports = { execute(tool, args, ctx) }
  package.json
  README.md
```

The build step packs this into a `.hzplugin` zip. The canonical
entry filename inside the zip is `handler.js` (renamed from `main.js`
at pack time). The host loader (`src/main/pluginManager.js`) loads
`handler.js` first and falls back to `main.js` for legacy bundles.

## 60-second quick start

```bash
# 1. Scaffold a new plugin (TypeScript, ready to build)
npx @horizonai/plugin-cli init my-plugin
cd my-plugin

# 2. Edit src/index.ts — add a tool
#    Type-safe via @horizonai/plugin-types

# 3. Build + dev
npm run build              # tsc → dist/
npm run dev                # watch mode

# 4. Sideload into Horizon for testing
npx @horizonai/plugin-cli pack         # → my-plugin-0.1.0.zip
# Then: Horizon GUI → Plugins → Install from file

# 5. Publish to the public marketplace
npx @horizonai/plugin-cli publish      # asks for credentials once
# → appears on horizonaai.dev/browse, installable via horizon:// deep links
```

## What you can build

| Category | Examples already shipping |
|---|---|
| **Service connectors** | Slack, Notion, Linear, Telegram, Discord (built-in); Trello, ClickUp, Airtable, Zendesk (community) |
| **Domain tools** | clipboard utilities, system-monitor, web-fetch, screenshot, crypto-pulse, currency-converter |
| **API wrappers** | Spotify, Google Maps, OpenWeather, IMDB, Wikipedia |
| **Workflow blocks** | scheduled jobs, conditional branches, custom triggers |
| **Provider adapters** | new LLM backends not in the bundled 25 |

Anything you can do in a Node.js function — file ops, HTTP requests,
child processes, native bindings — is fair game inside a plugin, as
long as your `manifest.json` declares the matching permissions.

## Permissions

`manifest.json` declares what your plugin needs. The host enforces
each at runtime, and the user sees the list on install:

```json
{
  "id": "my-currency-plugin",
  "version": "0.1.0",
  "tools": ["currency_convert"],
  "permissions": ["network:fetch"]
}
```

Common permissions:

| Permission | Grants |
|---|---|
| `network:fetch` | Outbound HTTP calls |
| `fs:read` / `fs:write` | File system access |
| `child_process` | Spawn subprocesses |
| `clipboard` | Read/write the system clipboard |
| `keys:read` | Read encrypted API keys from the host's key store |
| `notifications` | Show desktop notifications |

Declare only what you need. Users see your permissions before
install — bloated lists kill conversion.

## Revenue share

The marketplace splits revenue **70% to author, 30% to platform**.
Payouts are in crypto (USDC) on a monthly cadence. Authors keep
copyright; the marketplace is a distribution channel, not a
work-for-hire model.

See the SDK README for the publish flow, payout setup, and tax
considerations.

## Where to find what

| You want | Where it lives |
|---|---|
| TypeScript types | [`@horizonai/plugin-types`](https://www.npmjs.com/package/@horizonai/plugin-types) |
| Scaffolder CLI | [`@horizonai/plugin-cli`](https://www.npmjs.com/package/@horizonai/plugin-cli) |
| Full developer docs | [SDK README + docs/](https://github.com/ErnestKostevich/horizon-plugin-sdk) |
| Example plugins | [`examples/`](https://github.com/ErnestKostevich/horizon-plugin-sdk/tree/main/examples) |
| Marketplace | [horizonaai.dev/browse](https://horizonaai.dev/browse) |
| Tool API reference | The TypeScript types — they're the canonical source |

## Where to go next

- [Plugin SDK on GitHub](https://github.com/ErnestKostevich/horizon-plugin-sdk) — full developer documentation
- [Browse the marketplace](https://horizonaai.dev/browse)
- [Full CLI commands](../reference/cli-commands.md) — `horizon plugins list/show/enable/disable` for runtime management
