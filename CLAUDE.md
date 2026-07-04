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
- `QUICK-REF.md` — Topic-to-article/skill mapping
- `TOPIC-INDEX.md` — Detailed topic index
