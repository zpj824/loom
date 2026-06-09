---
name: loom-serve
description: Start a local web viewer + mock API server for a project's Loom JSON Schema API docs. Use when the user wants to preview/browse API documentation or run a mock API server backed by the schema files under docs/. No API key or LLM access is required. Trigger keywords - loom, loom serve, api docs, api documentation, preview api, browse endpoints, mock server, mock api, fake api, stub api, swagger ui, openapi viewer, json schema docs, docs viewer.
allowed-tools: Bash, Read
---

# loom serve

`loom serve` boots a single local server that combines a **web documentation viewer** and a **mock API server**, both backed by the JSON Schema files a project keeps under `docs/` (`*.schema.json` + `docs/entities/*.entity.schema.json`).

It is **read-only and offline**: it does not call any LLM and needs **no API key or config file**. The only inputs are the schema files on disk.

## When to use

- The user wants to **preview / browse** their API docs in a browser.
- The user wants a **mock API** that returns sample data generated from the schemas (for frontend dev, testing, etc.).
- There is a project directory containing `docs/*.schema.json` files (authored by Loom).

Do **not** use this skill to author or edit schemas — that is the interactive `loom chat` TUI, which is for humans.

## Prerequisites

- Node.js ≥ 18.
- A target project directory that contains Loom schema files at `<dir>/docs/*.schema.json`. If there are none, the viewer loads but shows nothing and no mock routes are registered.

`loom` is invoked via `npx @vegamo/loom` (no install step needed).

## How to run

The server is **long-running and blocking** — start it in the **background** so you don't hang waiting on it.

```bash
npx @vegamo/loom serve --port 3000 --dir /path/to/project
```

- `--port <number>` — port to listen on (defaults to `3000` when omitted). Pass it explicitly to avoid surprises.
- `--dir <path>` — target project root; the server serves `<dir>/docs/`. Defaults to the current working directory.

In Claude Code, launch it with background execution (e.g. the Bash tool's `run_in_background: true`) rather than blocking the turn.

On startup it logs:

```
[ok] Loom Server running at http://localhost:3000
[info]   Web Viewer:  http://localhost:3000/
[info]   Mock Server: http://localhost:3000/mock/...
[info]   N mock routes registered
[info]   Serving docs from: /path/to/project/docs
```

## What it exposes (one port)

| Path | What |
|------|------|
| `/` | React web viewer (browse endpoints & entities) |
| `/api/docs`, `/api/schemas`, `/api/entities`, `/api/mocks/*` | JSON APIs the viewer consumes |
| `/mock/<your endpoint paths>` | the mock API — each schema endpoint becomes a live route returning generated sample data |

To confirm it is up, GET `http://localhost:<port>/api/docs` — it should return JSON. The `N mock routes registered` line tells you how many endpoints the mock server mounted.

## How to stop

Kill the background process (e.g. terminate the background Bash task, or `kill <pid>`). There is no separate stop command for the CLI server.

## Notes / gotchas

- **No API key, no LLM, no config required** — safe to run anywhere with just the schema files.
- Long-running: always background it; never run it as a blocking foreground command in an automated flow.
- If you change schema files while it is running, restart the server to pick them up.
- `serve` = viewer + mock on one port. If you only want one, `loom view` (viewer only) and `loom mock` (mock only) exist, but `serve` is the usual choice.
