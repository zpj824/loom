---
name: loom-schema
description: Author Loom API documentation by hand-writing JSON Schema files (docs/*.schema.json and docs/entities/*.entity.schema.json). Use when you need to produce API contract/spec files in Loom's format — by translating requirements, an existing API, source code, or an OpenAPI/Swagger spec — so they can then be browsed and mocked with the `loom-serve` skill. No LLM or API key needed; you write the JSON directly.
---

# loom-schema — authoring Loom schema files

Loom renders a web doc viewer and a mock API server from plain JSON files on disk. This skill tells you the **exact file format** so you can generate those files yourself, then hand them to the **`loom-serve`** skill (which runs `loom serve`) for viewing and mocking.

You do **not** need the `loom` CLI, an LLM, or an API key to author — you just write JSON files that follow the format below. (`loom serve` reads the files directly; there is no build/index step required.)

## Workflow

1. Create the docs directory in the target project (default: `<project>/docs/`).
2. Write one `*.schema.json` file per group of related endpoints (e.g. all user APIs → `docs/users.schema.json`).
3. (Optional) Extract reusable objects into `docs/entities/*.entity.schema.json` and reference them with `x-entity-ref`.
4. Hand off to the **`loom-serve` skill** to start the viewer + mock server:
   `npx @vegamo/loom serve --port 3000 --dir <project>` (run it backgrounded — see the `loom-serve` skill).

`loom serve` picks up the files immediately. There is **no manifest/build step** required. (`npx @vegamo/loom manifest rebuild` only builds an optional cross-reference index; the viewer and mock work without it.)

## File layout

```
<project>/
  docs/
    users.schema.json          ← a "schema document": a group of endpoints
    order-management.schema.json
    entities/
      user.entity.schema.json  ← a reusable object ("entity")
      order.entity.schema.json
```

- Schema files: lowercase-with-hyphens, **must end with `.schema.json`**.
- Entity files: live under `docs/entities/`, **must end with `.entity.schema.json`**.
- Nesting is allowed and mirrors the URL prefix, e.g. `docs/admin/users.schema.json`.

## Schema document format (one `*.schema.json` file)

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "User Management API",
  "description": "User registration, authentication, and profile management",
  "version": "1.0.0",
  "endpoints": [
    {
      "path": "/api/users/register",
      "method": "POST",
      "summary": "Register a new user",
      "description": "Create a new user account with email and password",
      "tags": ["auth"],
      "request": {
        "body": {
          "type": "object",
          "properties": {
            "email": { "type": "string", "format": "email", "description": "User email" },
            "password": { "type": "string", "minLength": 8, "description": "Min 8 chars" },
            "name": { "type": "string", "minLength": 1, "description": "Display name" }
          },
          "required": ["email", "password", "name"]
        }
      },
      "response": {
        "201": {
          "type": "object",
          "properties": {
            "id": { "type": "integer", "description": "User ID" },
            "email": { "type": "string", "format": "email" },
            "name": { "type": "string" },
            "createdAt": { "type": "string", "format": "date-time" }
          },
          "required": ["id", "email", "name", "createdAt"]
        },
        "400": {
          "type": "object",
          "properties": {
            "error": { "type": "string" },
            "message": { "type": "string" }
          },
          "required": ["error", "message"]
        }
      }
    }
  ]
}
```

### Top-level fields

| Field | Required | Notes |
|-------|----------|-------|
| `$schema` | yes | always `"http://json-schema.org/draft-07/schema#"` |
| `title` | yes | human title of this group |
| `description` | yes | what this group covers |
| `version` | no | e.g. `"1.0.0"` |
| `endpoints` | yes | array of endpoint objects (below) |

### Endpoint fields

| Field | Required | Notes |
|-------|----------|-------|
| `path` | yes | RESTful path; use `:name` for path params, e.g. `/api/users/:id` |
| `method` | yes | one of `GET` `POST` `PUT` `PATCH` `DELETE` |
| `summary` | yes | one-line summary |
| `description` | no | longer description |
| `tags` | no | string array for grouping/filtering |
| `request` | no | `{ headers?, params?, query?, body? }` — each is a JSON Schema **object** |
| `response` | yes | map of status code → JSON Schema object; include at least one `2xx` plus common errors |

- Put `body` on POST/PUT/PATCH, `query` on GET filters/pagination, `params` for every `:param` in the path (the param name must match a property in `request.params`).
- Each `request.*` and each `response[code]` value is a standard draft-07 schema with `type`, `properties`, `required`, etc.

## JSON Schema conventions (the property nodes)

Use JSON Schema **draft-07**. Per property, include `type` plus a `description` and any constraints:

- types: `string`, `integer`, `number`, `boolean`, `object`, `array`
- `object` → has `properties` + `required` (array of required keys)
- `array` → has `items` (a schema)
- string formats: `email`, `uri`, `date-time`, `date`, `uuid`, …
- constraints: `minLength`/`maxLength`, `minimum`/`maximum`, `pattern`, `enum`, `default`

**Mock tip:** the mock server generates sample data from the **first `2xx` response schema**. Richer constraints (`format`, `enum`, `minimum`/`maximum`, `pattern`) produce more realistic mock values, so fill them in on your success responses.

## Reusable entities (optional)

If the same object appears in many endpoints, define it once as an entity and reference it with `x-entity-ref`. See **`references/format-reference.md`** for the full entity + `x-entity-ref` rules (string vs object form, `pick`/`omit`/`required`, field overrides). Short version:

- Entity file `docs/entities/user.entity.schema.json` whose `title` is the entity name (`"User"`).
- In an endpoint schema node: `"x-entity-ref": "User"` (inline whole entity), or
  `"x-entity-ref": { "entity": "User", "pick": ["id", "name"] }`.
- Entities are resolved by `loom serve` at view/mock time; you keep the raw `x-entity-ref` in your files.

## Rules of thumb

1. One file per logical group of endpoints; name it after the group.
2. Always include `$schema`, `title`, `description`, `endpoints`.
3. Every endpoint needs `path`, `method`, `summary`, and a `response` with at least one success code.
4. Every `:param` in a path must appear in `request.params.properties`.
5. Write valid JSON (no comments, no trailing commas).
6. When done, use the **`loom-serve`** skill to serve: viewer at `/`, mock API under `/mock/*`.

## Full reference

For exhaustive field tables, the entity/`x-entity-ref` resolution rules, and a complete multi-endpoint example with entities, read **`references/format-reference.md`**.
