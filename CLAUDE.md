# AutoCAD Knowledge Docs

Knowledge base for AutoCAD LT automation via MCP bridge. Articles teach concepts, skills provide executable procedures.

## Fetching content

```bash
gh api repos/lacrx/autocad-knowledge-docs/contents/{path} -H "Accept: application/vnd.github.raw+json"
```

1. Fetch `QUICK-REF.md` for topic lookup
2. If no match, try `TOPIC-INDEX.md`

## Structure

- `articles/` — Concept articles organized by domain
- `skills/` — Step-by-step procedures for common tasks
- `commands/` — Reusable drawing command recipes by task (geometry, walls, dims, text, blocks, etc.)
- `QUICK-REF.md` — Topic-to-article/skill/command mapping
- `TOPIC-INDEX.md` — Detailed topic index

## Scope

General AutoCAD LT knowledge: commands, drawing standards, MCP bridge protocol, LISP automation. Nothing machine-specific (see workhorse-knowledge-docs) or project-specific (lives in project repos).
