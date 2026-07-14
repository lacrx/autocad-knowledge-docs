# MCP Bridge Improvement Ideas

Actionable improvements for the AutoCAD LT MCP bridge, sourced from patterns across the CAD MCP ecosystem. Prioritized by effort vs impact for our specific constraints (no COM API, LISP-only, cross-VM).

## High Impact, Low Effort

### 1. JSONL Audit Trail

**Source:** Sam-AEC/Autodesk-Revit-MCP-Server

Log every MCP tool invocation to a JSONL file on the Linux side:

```json
{"ts": "2026-07-14T10:30:00Z", "tool": "create-line", "params": {...}, "result": {...}, "duration_ms": 450}
```

**Why:** Debugging failed operations, replaying sequences, understanding what changed during a session. Currently we lose this context when the conversation ends.

**Where:** Add to `server.py` — wrap the TCP send/receive with logging before returning to MCP.

### 2. Server-Side Validation Tools

**Source:** Sam-AEC QA audit tools

Python-side analysis functions that query the drawing once via bridge, then run compliance checks without further bridge calls:

- Layer naming audit (do all layers follow our prefix scheme?)
- Dimension-vs-geometry consistency check
- Required-layer checklist (does the drawing have all required layers for permit?)
- Text height validation (are all text objects at standard heights for their scale?)

**Why:** Perfect fit for our permit validation workflow. Faster than per-entity bridge calls. Can run as a pre-export check.

**Where:** New module in `mcp_server/` — `validation.py` or similar. Exposes as MCP tools.

### 3. Snapshot Diff

**Source:** Sam-AEC baseline tracking

Formalize our snapshot workflow:
1. `baseline_export` — dump all entity data (type, layer, position, key properties) to JSON
2. `baseline_diff` — compare current state against saved baseline
3. Report: what was added, moved, deleted, modified

**Why:** We already do manual snapshots (copy DWG before edits). A data-level diff would catch unintended changes without visual comparison.

**Where:** LISP function to enumerate all entities → JSON export. Python diff logic in server.

## Medium Impact, Medium Effort

### 4. Tool Consolidation (Dispatcher Pattern)

**Source:** schauh11 composable meta-tools, multiCAD-mcp

Current: ~50 individual tools. Proposed: consolidate into ~15 by grouping related operations:

| Current tools | Proposed dispatcher | Operations |
|--------------|-------------------|------------|
| entity-move, entity-copy, entity-rotate, entity-mirror, entity-scale | `entity-transform` | move, copy, rotate, mirror, scale |
| layer-create, layer-freeze, layer-thaw, layer-lock, layer-unlock | `layer-manage` | create, freeze, thaw, lock, unlock, set-current, set-properties |
| create-line, create-polyline, create-rectangle, create-circle, create-arc, create-ellipse | `create-geometry` | line, polyline, rectangle, circle, arc, ellipse |
| create-dimension-linear, create-dimension-aligned, create-dimension-angular, create-dimension-radius | `create-dimension` | linear, aligned, angular, radius |

**Why:** Reduces tool description token overhead. Each tool description costs ~100-200 tokens; cutting from 50 to 15 saves ~3,500-7,000 tokens per conversation.

**Trade-off:** More complex parameter schemas. The LLM needs to specify `operation` as well as params. Test whether Claude handles this well before committing.

### 5. Mock Mode for Development

**Source:** Sam-AEC mock mode

Run the MCP server without a live AutoCAD connection, returning deterministic stub responses:

```python
if os.environ.get("AUTOCAD_MOCK"):
    return {"status": "ok", "entities": [...stub data...]}
```

**Why:** Develop and test server-side logic (validation tools, audit trail, tool consolidation) without booting the VM. Faster iteration.

**Where:** Add mock responses to `server.py` or a parallel `mock_server.py`. Key tools to mock: `drawing-info`, `entity-list`, `entity-get`, `layer-list`.

### 6. Unit/Coordinate Helper Layer

**Source:** Demolinator unit standardization

Formalize unit handling:
- All tools accept architectural notation (e.g., `"2'-6\""` or `30`) 
- Auto-convert to AutoCAD internal units
- Report results in architectural notation

**Why:** Reduces errors from unit confusion. LLM (and user) think in feet-inches, not raw decimal coordinates.

**Where:** Helper functions in `server.py` that parse and format coordinates before/after bridge calls.

## Lower Priority, Higher Effort

### 7. Session Persistence (SQLite)

**Source:** mcp-servers-for-revit SQLite storage

Store entity snapshots, layer states, and project metadata in SQLite between sessions:
- Remember entity IDs from prior sessions
- Track which entities were created by AI vs pre-existing
- Query historical state without re-scanning the drawing

**Why:** Currently each session starts fresh — AI has no memory of what it did last time. With persistence, it could say "I created these 12 dimension entities yesterday, let me verify they're still correct."

**Trade-off:** Significant complexity. Entity IDs can change if the drawing is edited outside the bridge. Stale data risk.

### 8. File Watcher Instead of Polling

**Source:** General optimization

Replace the 150ms polling loop in `win_agent.py` with Windows ReadDirectoryChangesW:

```python
import win32file, win32con
handle = win32file.CreateFile(r'C:\temp', ...)
win32file.ReadDirectoryChangesW(handle, 1024, False, win32con.FILE_NOTIFY_CHANGE_FILE_NAME, None)
```

**Why:** Instant notification when result file appears, instead of up to 150ms latency. Reduces CPU usage from continuous polling.

**Trade-off:** More complex code, platform-specific. Current polling is simple and reliable. Only worth it if latency becomes a bottleneck.

### 9. Safety Tiers

**Source:** Archi Automate

Add operating modes:
1. **Read-only** — entity-list, entity-get, drawing-info, layer-list only
2. **Dry-run** — simulate writes, return what would change without modifying the drawing
3. **Full** — all tools enabled (current behavior)

**Why:** Safety during exploration. Read-only mode lets AI analyze the drawing without risk. Dry-run lets us preview changes before committing.

**Where:** Mode flag in `server.py` that filters which tools are registered with MCP.

## Not Applicable to AutoCAD LT

These patterns from the ecosystem don't translate to our constraints:

- **Embedded chat panel in CAD** — requires .NET API for UI extensibility (Revit has it, LT doesn't)
- **COM/ActiveX bridge** — AutoCAD LT doesn't expose COM API
- **Reflection API** — no programmatic API to reflect against
- **Runtime code compilation** — we have LISP execution but can't compile C#/Python inside LT

## See Also

- [Design Patterns](design-patterns.md) — patterns observed across the ecosystem
- [CAD MCP Ecosystem](cad-mcp-ecosystem.md) — landscape overview of all CAD MCP projects
