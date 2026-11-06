# AutoCAD Knowledge Docs

Knowledge base for AutoCAD LT automation via MCP bridge. Articles teach concepts, skills provide executable procedures.

## When to Fetch (Continuous)
Fetch whenever the task touches: autocad, autocad-lt, autolisp, lisp, mcp-bridge, win-agent, dwg, drawing, floor-plan, plan-set, dimensions, layers, blocks, hatch, viewports, sheet-setup, layouts.

Drawing work consults command recipes as it goes, so fetch during the task, not only at the start. If the task does not involve these topics, skip KB lookup entirely. Machine setup of the VM (boot, RDP, networking) lives in lan-knowledge-docs.

## Fetching content
On neo and workhorse use the `consult-knowledge-bases` skill: `~/.claude/skills/consult-knowledge-bases/kb route "<task>"`, then `kb get autocad <path>`.


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

General AutoCAD LT knowledge: commands, drawing standards, MCP bridge protocol, LISP automation. Nothing machine-specific (see lan-knowledge-docs) or project-specific (lives in project repos).
