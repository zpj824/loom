# loom-skills

[English](README.md) | [简体中文](README.zh-CN.md)

[**Loom**](https://www.npmjs.com/package/@vegamo/loom) 的 Agent Skill 集 —— Loom 是一个用 JSON Schema 编写 API 文档的工具,自带网页查看器和 Mock 服务器。

这些 skill 让任意 agent(Claude Code,或任何兼容 [`skills`](https://github.com/vercel-labs/skills) 生态的工具)都能生成并预览 API 文档,**无需 API key、无需 LLM**。

## Skills

| Skill | 作用 |
|-------|------|
| **`loom-schema`** | 教 agent Loom 的精确文件格式,让它**手写** JSON 规范文件(`docs/*.schema.json`、`docs/entities/*.entity.schema.json`、`x-entity-ref`)—— 可由需求、现有 API、源码,或 OpenAPI/Swagger 规范翻译而来。不需要 LLM。 |
| **`loom-serve`** | 通过 `npx @vegamo/loom` 从上述文件启动 Loom 的**网页查看器 + Mock API 服务器**(`loom serve`)。只读,无需 API key。 |

两者组成一条工作流:**先用 `loom-schema` 写出 schema 文件,再用 `loom-serve` 查看并 mock 它们。**

## 安装

使用 [`skills` CLI](https://github.com/vercel-labs/skills):

```bash
# 同时装两个 (把 OWNER/REPO 换成你发布到的仓库)
npx skills add OWNER/REPO -s loom-serve -s loom-schema

# 或只装一个
npx skills add OWNER/REPO -s loom-serve
```

加 `-g` 可装到全局(`~/.claude/skills/`)而非当前项目。

## 环境要求

- Node.js ≥ 18。
- `@vegamo/loom` 由 `npx` 按需拉取(已发布到公共 npm)—— 无需单独安装。

## 目录结构

```
skills/
  loom-serve/SKILL.md
  loom-schema/SKILL.md
  loom-schema/references/format-reference.md
```
