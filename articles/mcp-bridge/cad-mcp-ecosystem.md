# CAD/BIM MCP Ecosystem Landscape

## Overview

As of mid-2026, 15+ active MCP server projects exist for CAD/BIM software. The space exploded in early 2026, with Revit leading in quantity (7+ repos), followed by Rhino, FreeCAD, and AutoCAD. Every implementation converges on the same fundamental architecture our bridge uses.

## The Universal Bridge Pattern

```
AI Client --[MCP stdio/SSE]--> MCP Server --[transport]--> CAD Plugin --> CAD API
```

All implementations follow this three-tier pattern. Variations are in the transport layer:

| Transport | Used by | Pros | Cons |
|-----------|---------|------|------|
| HTTP (localhost) | pyRevit-based Revit MCPs | Simple, debuggable with curl | No streaming, no auth |
| TCP/JSON-RPC | mcp-servers-for-revit, LuDattilo | Structured, typed | More setup complexity |
| COM (pywin32) | multiCAD-mcp | Direct in-process, fastest | Windows-only, no VM crossing |
| WebSocket | schauh11/revit-mcp | Bidirectional, streaming | Non-standard for MCP |
| File IPC | Our AutoCAD bridge | Works across VM boundary | Latency from polling |
| XML-RPC | FreeCAD-MCP | Headless-friendly | Older protocol |

Our file IPC approach is unique — needed because AutoCAD LT has no COM/ActiveX/ObjectARX API and we cross a Linux→VM→Windows boundary. Everyone else runs on the same machine with full API access.

## Revit MCP Servers

The largest cluster. Three architectural camps:

### Camp 1: pyRevit Routes (Python, HTTP)

pyRevit's Routes feature = Flask-like HTTP server running inside Revit process on port 48884 via IronPython 2.7. An external FastMCP server (Python 3.11+) translates MCP protocol into HTTP calls.

Key repos:
- **mcp-server-for-revit-python** (147 stars) — Reference impl by JoTaDe, bundled in pyRevit 5.2.0. Ships an LLM.txt file for AI codebase understanding.
- **Demolinator/revit-mcp-server** (20 stars) — 48 tools including MEP (duct, pipe, systems). Unit standardization: accepts mm, auto-converts to Revit internal feet.

Pros: pure Python, no compilation, easy to extend. Cons: depends on pyRevit, Routes API is "draft" status, IronPython 2.7 limits.

### Camp 2: Native C# Plugin (TCP/JSON-RPC)

C# Revit add-in listens on TCP. TypeScript/Node.js MCP server connects via JSON-RPC.

Key repos:
- **mcp-servers-for-revit/revit-mcp** (438 stars, archived) — The pioneer, established the pattern.
- **mcp-servers-for-revit/mcp-servers-for-revit** (242 stars) — Sparx fork. Runtime C# code execution. SQLite cross-session memory. Dynamic plugin extensibility via command.json.
- **Sam-AEC/Autodesk-Revit-MCP-Server** (41 stars) — Most feature-complete (100+ tools). Universal Reflection API lets LLM call any of Revit's ~10,000 API methods by name. Mock mode for CI. Workspace sandboxing. JSONL audit trail. QA audit tools.
- **LuDattilo/revit-mcp-server** (29 stars) — 124 tools. Dockable Claude chat panel in Revit (Sonnet 4.6 + 10K extended thinking). 140 unit tests. Also built RevitCortex (173 tools, pure C#) as the evolution.

### Camp 3: Embedded WebSocket

- **schauh11/revit-mcp-server** (18 stars) — Full WPF chat panel in Revit. 5 composable meta-tools (query/execute/analyze/navigate/create) replace ~50 specific tools. Multi-LLM support (Claude, DeepSeek, Qwen). SessionState for follow-up commands.

### Commercial

- **AUTOM8LABS** — Autodesk App Store plugin. Free: 36 read-only tools. Pro: 120+.
- **BIM Automation Studio** — Managed pyRevit+MCP, $25-67/seat/month.
- **Archi Automate** — C# bridge with 3 safety tiers (read-only/dry-run/unrestricted).
- **Autodesk Official** — Revit 2027 ships with a read-only MCP server (announced June 2026).

## Rhino/Grasshopper

**RhinoMCP** (jingcheng-chen/rhinomcp) — Most mature non-Revit CAD MCP.
- Python FastMCP → TCP 127.0.0.1:1999 → C# RhinoCommon plugin
- 38+ Rhino tools, 25+ Grasshopper tools (including batched graph building)
- Escape hatches: `run_command`, `execute_rhinoscript_python_code`, `execute_rhinocommon_csharp_code`
- Env var switches to disable high-risk execution tools

## AutoCAD

- **Our bridge** (puran-water/autocad-mcp) — TCP + file IPC + LISP dispatch across VM boundary. The most technically constrained implementation in the ecosystem.
- **AnCode666/multiCAD-mcp** — COM-based, 7 unified tools dispatching to 56 commands. Claims 70% overhead reduction. Windows-only.

## FreeCAD

**proximile/FreeCAD-MCP** — 57+ tools, Docker-containerized headless architecture via XML-RPC. Only implementation that runs without a GUI.

## Academic Research

TU Munich (arxiv 2601.00809) cataloged 8+ BIM MCP servers. Main finding: tight coupling to single backends is the primary structural weakness. Proposed modular adapter architecture.

## Key People

| Person | Affiliation | Contribution |
|--------|-------------|-------------|
| Juan Rodriguez (JoTaDe) | Independent, Madrid | Reference pyRevit MCP implementation |
| Jean-Marc Couffin | BIM One | Championed MCP on pyRevit forums, org member |
| Bobby Galli (bobbyg603) | BugSplat / Sparx Fire | Monorepo maintainer |
| LuDattilo | Independent | 124-tool server → RevitCortex (173 tools) |
| A. Sam Mohammad | Netherlands | AEC Model Bridge — most feature-complete |
| Sanjay Chauhan | Independent | Embedded chat panel approach |
| jingcheng-chen | Independent | RhinoMCP |
