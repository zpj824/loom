# loom-skills

[English](README.md) | [简体中文](README.zh-CN.md)

Agent skills for [**Loom**](https://www.npmjs.com/package/@vegamo/loom) — an LLM-authored JSON Schema API doc toolkit with a web viewer and mock server.

These skills let any agent (Claude Code, or anything compatible with the
[`skills`](https://github.com/vercel-labs/skills) ecosystem) produce and preview
API documentation without needing an API key or LLM access.

## Skills

| Skill | What it does |
|-------|--------------|
| **`loom-schema`** | Teaches the agent Loom's exact file format so it can **hand-author** the JSON spec files (`docs/*.schema.json`, `docs/entities/*.entity.schema.json`, `x-entity-ref`) — by translating requirements, an existing API, source code, or an OpenAPI/Swagger spec. No LLM needed. |
| **`loom-serve`** | Starts Loom's combined **web viewer + mock API server** (`loom serve`) from those files via `npx @vegamo/loom`. Read-only, no API key. |

Together they form a workflow: **author the schema files with `loom-schema`, then browse & mock them with `loom-serve`.**

## Install

Using the [`skills` CLI](https://github.com/vercel-labs/skills):

```bash
# both skills  
npx skills add https://github.com/zpj824/loom/tree/main/loom-skills -s loom-serve -s loom-schema

# or just one
npx skills add https://github.com/zpj824/loom/tree/main/loom-skills -s loom-serve
```

## Requirements

- Node.js ≥ 18.
- `@vegamo/loom` is fetched on demand via `npx` (published on public npm) — no separate install needed.

## Layout

```
skills/
  loom-serve/SKILL.md
  loom-schema/SKILL.md
  loom-schema/references/format-reference.md
```
