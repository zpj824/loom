# Loom schema format — full reference

This is the exhaustive reference for hand-authoring Loom schema files. The
SKILL.md covers the common case; read this when you need entity references or
precise field semantics.

## 1. Schema document (`docs/<name>.schema.json`)

```jsonc
{
  "$schema": "http://json-schema.org/draft-07/schema#", // required, exact value
  "title": "string",        // required
  "description": "string",  // required
  "version": "1.0.0",       // optional
  "endpoints": [ /* EndpointDefinition[] */ ] // required
}
```

### EndpointDefinition

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `path` | string | yes | e.g. `/api/users/:id`. Path params use `:name`. |
| `method` | enum | yes | `GET` \| `POST` \| `PUT` \| `PATCH` \| `DELETE` |
| `summary` | string | yes | one-line label shown in the viewer list |
| `description` | string | no | longer prose |
| `tags` | string[] | no | grouping/filtering labels |
| `request` | object | no | `{ headers?, params?, query?, body? }` |
| `request.headers` | JSON Schema object | no | request headers |
| `request.params` | JSON Schema object | no | path params; one property per `:name` in `path` |
| `request.query` | JSON Schema object | no | query string params |
| `request.body` | JSON Schema object | no | request body (POST/PUT/PATCH) |
| `response` | object | yes | map: status code string → JSON Schema object |

Each `request.*` value and each `response[code]` value is a normal draft-07
schema, typically `{ "type": "object", "properties": {...}, "required": [...] }`.

> **Critical:** the viewer lists parameters from the node's `properties`. A
> `body`/`query`/`params`/`headers` (or response) without `type: "object"` +
> `properties` renders an **empty** parameter table. This is the usual cause of
> "POST/PUT request params don't show up."

**Do NOT carry over OpenAPI/Swagger structure.** Loom has no `requestBody`, no
`content`/media types, and no `parameters` array. Translate:

| ❌ OpenAPI (won't render) | ✅ Loom |
|---|---|
| `requestBody.content."application/json".schema` | `request.body` (a `{type:object, properties, required}` schema) |
| `parameters: [{in:"query", name, schema}]` | `request.query.properties.<name>` |
| `parameters: [{in:"path", name, schema}]` | `request.params.properties.<name>` |
| `parameters: [{in:"header", name, schema}]` | `request.headers.properties.<name>` |
| `body: { "email": {…} }` (bare fields) | `body: { "type":"object", "properties": { "email": {…} } }` |

### Status codes

- Always provide at least one success code (`200` or `201`).
- Recommended to also document common errors (`400`, `401`, `403`, `404`, `500`)
  as object schemas, e.g. `{ error, message }`.
- The **mock server** responds using the **first `2xx`** schema it finds, run
  through sample-data generation. Put your richest, most realistic constraints
  on that success schema.

## 2. JSON Schema node reference (draft-07)

| Keyword | Applies to | Example |
|---------|-----------|---------|
| `type` | all | `"string"`, `"integer"`, `"number"`, `"boolean"`, `"object"`, `"array"` |
| `description` | all | `"User email address"` |
| `properties` | object | `{ "id": { "type": "integer" } }` |
| `required` | object | `["id", "name"]` |
| `items` | array | `{ "type": "object", "properties": {...} }` |
| `enum` | any | `["asc", "desc"]` |
| `default` | any | `20` |
| `format` | string | `email`, `uri`, `date-time`, `date`, `uuid`, `hostname`, `ipv4` |
| `minLength` / `maxLength` | string | `8` / `100` |
| `pattern` | string | `"^1[3-9]\\d{9}$"` (regex; escape backslashes in JSON) |
| `minimum` / `maximum` | number/integer | `0` / `150` |

Nested objects and arrays of objects are fully supported — just nest `properties`
/ `items` as deep as needed.

## 3. Entities (`docs/entities/<name>.entity.schema.json`)

An entity is one reusable object schema. Its `title` is the **name** other files
reference.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "User",
  "description": "A user account",
  "type": "object",
  "properties": {
    "id":    { "type": "integer", "minimum": 1, "description": "User ID" },
    "email": { "type": "string", "format": "email" },
    "name":  { "type": "string", "minLength": 1 },
    "role":  { "type": "string", "enum": ["admin", "user", "guest"], "default": "user" },
    "createdAt": { "type": "string", "format": "date-time" }
  },
  "required": ["id", "email", "name", "createdAt"]
}
```

- File name ends with `.entity.schema.json`, lives under `docs/entities/`.
- `type` is `"object"`.
- The **`title`** (here `"User"`) is what you put in `x-entity-ref`.

## 4. `x-entity-ref` — referencing an entity

Place `x-entity-ref` on any schema node inside an endpoint's `request.*` or
`response[code]`. It is resolved by `loom serve` at view/mock time; your file
keeps the `x-entity-ref` as written.

### String form — inline the whole entity

```json
"data": { "x-entity-ref": "User" }
```

### Object form — narrow / adjust the entity

```json
"data": {
  "x-entity-ref": { "entity": "User", "pick": ["id", "name", "email"] }
}
```

| Key | Meaning |
|-----|---------|
| `entity` | entity name (its `title`) — required |
| `pick` | keep only these properties |
| `omit` | drop these properties (use `pick` or `omit`, not both) |
| `required` | override the required list for this usage |

### Field overrides on the ref node

Alongside `x-entity-ref` you may set keys that override the resolved entity:
`title`, `description`, `nullable`, `deprecated`, `readOnly`, `writeOnly`,
`default`, `example`, `examples`, and any `x-*` key. Example:

```json
"owner": {
  "x-entity-ref": { "entity": "User", "pick": ["id", "name"] },
  "description": "The user who owns this order",
  "nullable": true
}
```

### Wrapping an entity in a list

`x-entity-ref` describes one object; for a list, use it as the array `items`:

```json
"data": {
  "type": "array",
  "items": { "x-entity-ref": "User" }
}
```

### Resolution failures

If an entity is missing or the ref is malformed, `loom serve` does not crash —
it renders a fallback node carrying `x-entity-ref-error` and lists the issue
under `entityResolveIssues`. So broken refs are visible but non-fatal; still,
make sure every referenced entity file exists.

## 5. Complete example with an entity

`docs/entities/user.entity.schema.json` — as in section 3.

`docs/users.schema.json`:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "User Management API",
  "description": "List and fetch users",
  "version": "1.0.0",
  "endpoints": [
    {
      "path": "/api/users",
      "method": "GET",
      "summary": "List users",
      "tags": ["users"],
      "request": {
        "query": {
          "type": "object",
          "properties": {
            "page": { "type": "integer", "minimum": 1, "default": 1 },
            "pageSize": { "type": "integer", "minimum": 1, "maximum": 100, "default": 20 },
            "role": { "type": "string", "enum": ["admin", "user", "guest"] }
          },
          "required": []
        }
      },
      "response": {
        "200": {
          "type": "object",
          "properties": {
            "success": { "type": "boolean" },
            "data": {
              "type": "array",
              "items": { "x-entity-ref": "User" }
            },
            "pagination": {
              "type": "object",
              "properties": {
                "page": { "type": "integer" },
                "pageSize": { "type": "integer" },
                "total": { "type": "integer" }
              },
              "required": ["page", "pageSize", "total"]
            }
          },
          "required": ["success", "data", "pagination"]
        },
        "401": {
          "type": "object",
          "properties": {
            "error": { "type": "string", "enum": ["UNAUTHORIZED"] },
            "message": { "type": "string" }
          },
          "required": ["error", "message"]
        }
      }
    },
    {
      "path": "/api/users/:id",
      "method": "GET",
      "summary": "Get a user by id",
      "tags": ["users"],
      "request": {
        "params": {
          "type": "object",
          "properties": { "id": { "type": "integer", "minimum": 1 } },
          "required": ["id"]
        }
      },
      "response": {
        "200": {
          "type": "object",
          "properties": {
            "success": { "type": "boolean" },
            "data": { "x-entity-ref": "User" }
          },
          "required": ["success", "data"]
        },
        "404": {
          "type": "object",
          "properties": {
            "error": { "type": "string", "enum": ["USER_NOT_FOUND"] },
            "message": { "type": "string" }
          },
          "required": ["error", "message"]
        }
      }
    }
  ]
}
```

## 6. Checklist before handing off to `loom serve`

- [ ] Files end with `.schema.json` (endpoints) / `.entity.schema.json` (entities under `docs/entities/`).
- [ ] Valid JSON: no comments, no trailing commas.
- [ ] Each document has `$schema`, `title`, `description`, `endpoints`.
- [ ] Each endpoint has `path`, `method`, `summary`, and a `response` with ≥1 `2xx`.
- [ ] Every `:param` in a path has a matching property in `request.params`.
- [ ] Every entity named in an `x-entity-ref` has a file under `docs/entities/`.
- [ ] Success (`2xx`) schemas carry realistic `format` / `enum` / `min`-`max` for good mock data.
