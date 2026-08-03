---
name: sdd-helper-serve
description: Start a local web viewer + mock API server for a project's sdd-helper JSON Schema API docs. Use when the user wants to preview/browse API documentation or run a mock API server backed by the schema files under docs/. No API key or LLM access is required. Trigger keywords - sdd-helper, sdd-helper serve, api docs, api documentation, preview api, browse endpoints, mock server, mock api, fake api, stub api, swagger ui, openapi viewer, json schema docs, docs viewer.
allowed-tools: Bash, Read
---

# sdd-helper serve

`sdd-helper serve` boots a single local server that combines a **web documentation viewer** and a **mock API server**, both backed by the JSON Schema files a project keeps under `docs/` (`*.schema.json` + `docs/entities/*.entity.schema.json`).

It is **read-only and offline**: it does not call any LLM and needs **no API key or config file**. The only inputs are the schema files on disk.

## When to use

- The user wants to **preview / browse** their API docs in a browser.
- The user wants a **mock API** that returns sample data generated from the schemas (for frontend dev, testing, etc.).
- There is a project directory containing `docs/*.schema.json` files (authored by sdd-helper).

Do **not** use this skill to author or edit schemas — that is the interactive `sdd-helper chat` TUI, which is for humans.

## Prerequisites

- Node.js ≥ 18.
- A target project directory that contains sdd-helper schema files at `<dir>/docs/*.schema.json`. If there are none, the viewer loads but shows nothing and no mock routes are registered.

`sdd-helper` is invoked via `npx @vegamo/sdd-helper` (no install step needed).

## How to run

The server is **long-running and blocking** — start it in the **background** so you don't hang waiting on it.

```bash
npx @vegamo/sdd-helper serve --port 3000 --dir /path/to/project
```

- `--port <number>` — port to listen on (defaults to `3000` when omitted). Pass it explicitly to avoid surprises.
- `--dir <path>` — target project root; the server serves `<dir>/docs/`. Defaults to the current working directory.

In Claude Code, launch it with background execution (e.g. the Bash tool's `run_in_background: true`) rather than blocking the turn.

On startup it logs:

```
[ok] sdd-helper server running at http://localhost:3000
[info]   Web Viewer:  http://localhost:3000/
[info]   Mock Server: http://localhost:3000/mock/...
[info]   N mock routes registered
[info]   Serving docs from: /path/to/project/docs
```

## Online publishing

sdd-helper can also publish the same `docs/` package to the hosted server. Use this when the user wants a shareable online documentation URL instead of only a local preview/mock server.

Default hosted endpoint:

```
https://loom-server.vegamo.cn
```

The CLI auto-creates a stable publish token on first use and stores only that token in `~/.loom/auth.json`. The server URL is built into sdd-helper and is not exposed as a normal publish command parameter. The existing `.loom` storage path is retained for identity and publishing compatibility.

Two equivalent entry styles are supported:

```bash
# Outer CLI commands
npx @vegamo/sdd-helper plans
npx @vegamo/sdd-helper purchase basic_annual
npx @vegamo/sdd-helper publish
```

```text
# Inside `sdd-helper chat`
/plans
/purchase basic_annual
/publish
```

Publishing creates or reuses `<project>/.loom/project.json`, which pins the stable `projectSlug` for that local docs project. Reusing the same `projectSlug` updates the same hosted project; changing it is treated as a new hosted project and may require another `basic_annual` slot unless the token has active `pro_annual`.

Use:

- `plans` / `/plans` to show available plans plus current pro/slot/project entitlement.
- `purchase basic_annual` / `/purchase basic_annual` to buy one annual project slot.
- `purchase pro_annual` / `/purchase pro_annual` to buy unlimited annual projects.
- `publish` / `/publish` to upload the current docs bundle; if there is no pro entitlement and no available slot for a new project, sdd-helper should stop and guide the user to purchase first.

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
- `serve` = viewer + mock on one port. If you only want one, `sdd-helper view` (viewer only) and `sdd-helper mock` (mock only) exist, but `serve` is the usual choice.
