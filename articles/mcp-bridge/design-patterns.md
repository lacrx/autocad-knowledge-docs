# MCP Bridge Design Patterns

Patterns observed across 15+ CAD MCP implementations (Revit, Rhino, FreeCAD, AutoCAD). These are proven approaches — most discovered independently by multiple teams.

## Escape Hatches

Every mature CAD MCP provides pre-defined tools AND a way to execute arbitrary code. The pre-defined tools handle common operations reliably; the escape hatch handles everything else.

| Implementation | Escape Hatch | Language |
|---------------|-------------|----------|
| Our AutoCAD bridge | `execute-lisp` | AutoLISP |
| pyRevit MCP servers | `execute_revit_code` | IronPython 2.7 |
| C# Revit MCP servers | `send_code_to_revit` | C# (runtime compiled) |
| Sam-AEC (Revit) | `invoke_method` / `reflect_get` / `reflect_set` | Reflection (any API method by name) |
| RhinoMCP | `execute_rhinoscript_python_code` / `execute_rhinocommon_csharp_code` | Python or C# |

The reflection approach (Sam-AEC) is the most powerful — it exposes the entire Revit API (~10,000 methods) without writing handler code for each. Trade-off is safety and reliability. The LLM must construct correct method calls with proper argument types.

Our `execute-lisp` is the most constrained escape hatch (AutoLISP is limited vs Python/C#), but it works well because we supplement it with a curated LISP library.

### When to use escape hatch vs pre-defined tool

Pre-defined tools when: operation is common, parameters are well-defined, reliability matters.
Escape hatch when: one-off operation, exploring, prototyping new tool behavior.

Pattern: prototype via escape hatch → if it works, formalize into a pre-defined tool + LISP function.

## Tool Surface Design

Two competing approaches:

### Many Specific Tools (100+ tools)

Used by: Sam-AEC (100+), LuDattilo (124), RevitCortex (173).

Pros:
- Clear intent per tool — LLM doesn't have to compose complex parameters
- Better type checking and validation per tool
- Easier to document and discover

Cons:
- Large tool surface for LLM to navigate (token cost for tool descriptions)
- More round-trips for multi-step operations
- More code to maintain

### Few Composable Tools (5-10 tools)

Used by: schauh11 (5 meta-tools), multiCAD-mcp (7 dispatchers).

schauh11's pattern: `query`, `execute`, `analyze`, `navigate`, `create` — five tools that accept parameters dictating what to do:

```json
{"tool": "query", "params": {"category": "Walls", "filter": {"thickness": ">6"}}}
{"tool": "execute", "params": {"action": "hide", "elements": "$last_query"}}
```

multiCAD-mcp's pattern: 7 tools dispatching to 56 commands via a switch/dictionary:

```json
{"tool": "element_tools", "operation": "move", "params": {...}}
{"tool": "element_tools", "operation": "copy", "params": {...}}
```

Pros:
- Fewer round-trips, less tool description overhead
- More flexible — new operations without new tools
- Session state (`$last_query`) enables natural follow-ups

Cons:
- More complex parameter schemas
- LLM must understand the dispatch grammar
- Harder to validate — errors surface at runtime not schema level

### Our position

We use ~50 specific tools. Could benefit from consolidation — several groups (entity-move/copy/rotate/mirror/scale, layer-create/freeze/thaw/lock/unlock) could become dispatchers without losing clarity.

## Transport Layer Patterns

### File IPC (our approach)

```
Agent writes command.json → LISP polls → LISP writes result.json → Agent polls
```

Unique to our bridge. Required because:
1. AutoCAD LT has no COM/ActiveX/.NET API
2. We cross a Linux → VM → Windows boundary
3. LISP can read/write files but can't listen on sockets

Optimization: poll interval (currently 150ms) trades latency for CPU. Could use Windows ReadDirectoryChangesW for instant notification instead of polling, but adds complexity.

### HTTP (pyRevit pattern)

```
MCP Server → HTTP POST localhost:48884 → pyRevit Routes handler → Revit API
```

Simplest to debug (curl works). No streaming. No auth (localhost-only mitigation). Used by all pyRevit-based Revit MCPs.

### TCP/JSON-RPC (C# Revit pattern)

```
MCP Server → TCP localhost:8080 → JSON-RPC dispatch → ExternalEvent → Revit API
```

More structured than HTTP. Connection mutex serializes requests (CAD apps are single-threaded). Used by mcp-servers-for-revit, LuDattilo.

### COM (multiCAD pattern)

```
MCP Server → pywin32 COM → CAD application in-process
```

Direct, fastest, no IPC overhead. But Windows-only and requires COM API support (AutoCAD full has it, LT does not).

## Session Persistence

### Cross-session memory (mcp-servers-for-revit)

SQLite database stores project/room data across AI sessions:
- `store_project_data` / `store_room_data` / `query_stored_data`
- AI can reference previous analysis without re-querying the model

### Session state (schauh11)

5-minute TTL cache of last query results:
- "List all walls" → stores result set
- "Hide them" → references cached set
- Enables natural language follow-ups

### Audit trail (Sam-AEC)

JSONL log of every tool invocation:
- Input parameters, output results, timestamps
- Enables replay, debugging, compliance tracking

### Baseline tracking (Sam-AEC)

Snapshot model state, diff against later state:
- `baseline_export` → JSON snapshot
- `baseline_diff` → what changed since snapshot

Useful for: verifying edits, rollback decisions, change documentation.

## Safety Patterns

### Safety tiers (Archi Automate)

Three modes:
1. **Read-only** — query and export only, no modifications
2. **Dry-run** — simulate modifications, report what would change
3. **Unrestricted** — full read-write access

### Workspace sandboxing (Sam-AEC)

File path arguments validated against an allowlist. Path traversal blocked at the Python layer before reaching the CAD bridge.

### Execution tool switches (RhinoMCP)

Environment variables to disable high-risk tools:
```
RHINO_MCP_ENABLE_RUN_COMMAND=false
RHINO_MCP_ENABLE_RHINOSCRIPT=false
RHINO_MCP_ENABLE_CSHARP=false
```

### Connection mutex

Revit (and AutoCAD) are single-threaded for API operations. mcp-servers-for-revit implements a TypeScript mutex that serializes all requests, preventing race conditions. Our bridge achieves this naturally — file IPC is inherently sequential.

## QA and Validation Tools

### Server-side audit tools (Sam-AEC)

Python-side analysis that composes data from bridge queries into compliance reports. These don't need the CAD bridge for every check — they query once, analyze in Python:

- `model_health_summary`
- `warning_triage_report`
- `naming_standards_audit`
- `parameter_compliance_audit`
- `tag_coverage_audit`
- `room_space_completeness_report`

Applicable to our permit validation workflow — query drawing data once, run compliance checks in Python against building code requirements.

## Documentation Patterns

### LLM.txt (mcp-server-for-revit-python)

Ships a file describing the entire codebase structure so AI assistants can help extend the project. Like CLAUDE.md but specifically for external AI consumers of the code.

### Unit standardization (Demolinator)

All tools accept millimeters, auto-convert to internal units (Revit uses feet internally: mm / 304.8). Users never think in internal units.

Our equivalent: we work in architectural inches/feet. Could formalize a layer that auto-converts user-friendly units to AutoCAD internal coordinates.

## Embedded AI Interfaces

Two repos embed AI directly in the CAD application:

### Dockable chat panel (LuDattilo)

WPF panel docked inside Revit. Uses Claude Sonnet 4.6 with 10K extended thinking tokens. System prompt runs in autonomous mode — executes actions directly without confirmation prompts.

### Full chat UI (schauh11)

WPF chat panel + WebSocket to Node.js server. Multi-LLM support. IntentDetector does lightweight classification before hitting the LLM to save tokens. Conversation history in SQLite.

Not applicable to AutoCAD LT (no .NET API for UI extensibility), but conceptually interesting — the AI lives where the user works.
