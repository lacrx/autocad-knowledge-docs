# MCP Bridge Architecture for AutoCAD LT

## Overview

AutoCAD LT lacks a COM/ActiveX API, so automation requires an indirect approach: an MCP server on Linux communicates via TCP to a Windows agent that uses PostMessage and file-based IPC to control AutoCAD.

## Architecture

```
Claude Code ←stdio MCP→ server.py (Linux) ←TCP:9876→ win_agent.py (Windows VM) ←PostMessage/Files→ AutoCAD LT
```

### Components

**server.py (Linux MCP Server)**
- Runs as an MCP stdio server registered in `.mcp.json`
- Exposes 50+ tools for drawing, layers, entities, dimensions, blocks, LISP execution
- Each tool call generates a unique `request_id`, sends JSON over TCP to the Windows agent
- Configurable via environment variables: `AUTOCAD_VM_HOST`, `AUTOCAD_VM_PORT`, `AUTOCAD_TIMEOUT`

**win_agent.py (Windows TCP Agent)**
- Listens on `0.0.0.0:9876`, accepts newline-delimited JSON
- Built-in commands handled directly: `ping`, `list-windows`, `exec`, `ps`, `diag`, `type`, `update`
- All other commands dispatched via file-based IPC to AutoCAD's LISP environment
- Uses `pywin32` PostMessage WM_CHAR to send text to AutoCAD (replaced SendKeys for reliability from background processes)
- Supports `target` field to direct commands to a specific AutoCAD instance by window title substring

**mcp_dispatch.lsp (AutoCAD LISP Dispatcher)**
- Polls `C:\temp\` for command JSON files matching `autocad_mcp_cmd_*.json`
- Routes commands to handler functions via a whitelist
- Writes result JSON to `autocad_mcp_result_*.json`
- Must be loaded via SCRIPT command (not `type`): `_.SCRIPT C:/temp/load_dispatch.scr`

## Protocol

1. Claude calls MCP tool (e.g., `mcp__autocad__drawing-info`)
2. server.py creates `{"request_id": "abc123", "command": "drawing-info"}` and sends over TCP
3. win_agent.py writes JSON to `C:\temp\autocad_mcp_cmd_abc123.json`
4. win_agent.py triggers AutoCAD via PostMessage WM_CHAR: `(c:mcp-dispatch)\n`
5. mcp_dispatch.lsp reads the command file, executes, writes result to `C:\temp\autocad_mcp_result_abc123.json`
6. win_agent.py polls for the result file (150ms interval, 30s timeout), reads it, returns JSON over TCP
7. server.py returns the result to Claude

## Timeouts

- Default TCP timeout: 30 seconds (configurable via `AUTOCAD_TIMEOUT`)
- LISP poll interval: 150ms
- LISP poll timeout: 30 seconds
- PostMessage character delay: 10ms between characters

## Input Methods

| Method | How it works | When to use |
|--------|-------------|-------------|
| PostMessage WM_CHAR | Sends characters one at a time to hwnd | Dispatch trigger `(c:mcp-dispatch)`, short commands |
| SCRIPT command | Reads from .scr file, executes each line | Loading LISP files, complex expressions |
| SendKeys (PowerShell) | System.Windows.Forms.SendKeys | Dismissing dialogs, UI interaction (requires foreground) |
| File IPC | JSON command/result files in C:\temp | All drawing commands (via dispatcher) |

### Why not just PostMessage for everything?

AutoCAD LT 2026 uses Chrome/CEF-based UI controls — no traditional Win32 Edit control for the command line. PostMessage WM_CHAR to the main frame works for short strings but longer LISP expressions get intercepted by AutoCAD's command autocomplete. Example: `(load` triggers the APPLOAD command before the rest of the expression arrives.

## Direct Commands

win_agent.py handles these without touching AutoCAD's LISP dispatcher:

| Command | Field | Purpose |
|---------|-------|---------|
| `ping` | — | Agent status + AutoCAD window detection |
| `list-windows` | — | All visible AutoCAD window handles and titles |
| `type` | `text` | Send text to AutoCAD via PostMessage |
| `exec` | `cmd` | Run shell command on Windows |
| `ps` | `script` | Run PowerShell script |
| `diag` | — | Enumerate AutoCAD child windows |
| `update` | `url` | Hot-reload win_agent.py from URL |

## Multi-Instance Support

AutoCAD LT 2026 uses SDI — each document is a separate process. The `target` field in requests selects which instance receives the command:

```json
{"command": "drawing-info", "request_id": "x", "target": "LaCroix"}
```

Matches case-insensitive substring against window titles. Without `target`, uses the first AutoCAD window found.

## VM Management

```bash
# Start VM
sg libvirt -c 'LIBVIRT_DEFAULT_URI=qemu:///system virsh start win11-autocad'

# SSH to VM
ssh 192.168.122.147

# Check if agent running
echo '{"command":"ping"}' | timeout 5 nc 192.168.122.147 9876

# Start agent via scheduled task (runs in interactive Session 1)
ssh 192.168.122.147 "powershell -Command \"Start-ScheduledTask -TaskName 'StartWinAgent'\""
```

## Persistent VM Files

These files should always exist on the VM at `C:\temp\`:

| File | Purpose |
|------|---------|
| `load_dispatch.scr` | SCRIPT file to load mcp_dispatch.lsp safely |
| `screenshot.py` | Screen capture script (GDI-based, no PIL needed) |
| `dismiss_dialogs.ps1` | PowerShell script to dismiss modal dialogs |

## See Also

- [Troubleshooting](troubleshooting.md) — common issues, recovery sequence, diagnostic tools
