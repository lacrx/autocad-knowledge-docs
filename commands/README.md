# Drawing Commands

Reusable AutoCAD command recipes organized by task. Each file contains ready-to-use commands for the MCP bridge — copy, adapt coordinates, execute.

## Categories

| File | Purpose |
|------|---------|
| [geometry.md](geometry.md) | Lines, polylines, rectangles, arcs, circles |
| [walls.md](walls.md) | Wall segments, corners, openings, hatching |
| [dimensions.md](dimensions.md) | Linear, aligned, angular dims, leaders, callouts |
| [text-and-notes.md](text-and-notes.md) | Text, mtext, room labels, general notes |
| [blocks.md](blocks.md) | Insert blocks, doors, windows, fixtures, symbols |
| [hatching.md](hatching.md) | Solid, pattern, gradient fills |
| [layers.md](layers.md) | Layer setup, switching, visibility presets |
| [viewports.md](viewports.md) | Paper space viewports, scale, layer freeze |
| [annotation.md](annotation.md) | Section marks, elevation marks, detail callouts |
| [modification.md](modification.md) | Move, copy, mirror, offset, trim, extend, fillet |
| [selection.md](selection.md) | Selection sets, filters, entity queries |

## Usage

Commands use two formats:

**MCP tool call** (via server.py):
```python
mcp__autocad__create_line(start_point="0,0", end_point="144,0", layer="FP-WALL")
```

**Direct TCP** (via nc to win_agent):
```bash
echo '{"command":"paste","text":"_.LINE 0,0 144,0 "}' | nc 192.168.122.147 9876
```

**LISP expression** (for complex operations):
```bash
echo '{"command":"paste","text":"(command \"_.LINE\" \"0,0\" \"144,0\" \"\")"}' | nc 192.168.122.147 9876
```
