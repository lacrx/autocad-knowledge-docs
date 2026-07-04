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
- Built-in commands handled directly: `ping`, `exec`, `ps`, `diag`, `type`, `update`
- All other commands dispatched via file-based IPC to AutoCAD's LISP environment
- Uses `pywin32` for window enumeration and `SendKeys` for command input

**mcp_dispatch.lsp (AutoCAD LISP Dispatcher)**
- Polls `C:\temp\` for command JSON files matching `autocad_mcp_cmd_*.json`
- Routes commands to handler functions via a whitelist
- Writes result JSON to `autocad_mcp_result_*.json`
- Must be loaded manually: `(load "C:/autocad_mcp/mcp_dispatch.lsp")`

## Protocol

1. Claude calls MCP tool (e.g., `mcp__autocad__drawing-info`)
2. server.py creates `{"request_id": "abc123", "command": "drawing-info"}` and sends over TCP
3. win_agent.py writes JSON to `C:\temp\autocad_mcp_cmd_abc123.json`
4. win_agent.py triggers AutoCAD via SendKeys: `(c:mcp-dispatch)`
5. mcp_dispatch.lsp reads the command file, executes, writes result to `C:\temp\autocad_mcp_result_abc123.json`
6. win_agent.py polls for the result file, reads it, returns JSON over TCP
7. server.py returns the result to Claude

## Timeouts

- Default TCP timeout: 30 seconds
- LISP poll interval: 150ms
- LISP poll timeout: 30 seconds
- SendKeys input delay: 10ms per character

## Error Handling

| Failure | Symptom | Fix |
|---------|---------|-----|
| VM not running | ConnectionRefusedError | Start VM via virsh |
| win_agent.py not running | ConnectionRefusedError on port 9876 | Start via scheduled task |
| LISP dispatcher not loaded | Timeout after 30s | Load mcp_dispatch.lsp in AutoCAD |
| AutoCAD dialog open | Timeout (SendKeys go to dialog) | Dismiss dialog, ESC, retry |
| Command not whitelisted | Error in result JSON | Add command to dispatcher whitelist |

## Direct Commands

win_agent.py supports commands that bypass AutoCAD IPC:

- **`ping`** — Returns agent status and whether AutoCAD window is found
- **`exec`** — Run arbitrary shell command on Windows (for debugging)
- **`ps`** — Run PowerShell script (for UI automation)
- **`diag`** — Enumerate AutoCAD child windows (find command line control)
- **`type`** — Send text directly to AutoCAD via SendKeys
- **`update`** — Hot-reload win_agent.py from a URL

## VM Management

```bash
# Start VM
sg libvirt -c 'LIBVIRT_DEFAULT_URI=qemu:///system virsh start win11-autocad'

# SSH to VM
ssh 192.168.122.147

# Check if agent running
ssh 192.168.122.147 "tasklist /FI \"IMAGENAME eq python.exe\" /V /FO CSV"

# Start agent via scheduled task (runs in interactive Session 1)
ssh 192.168.122.147 "powershell -Command \"Start-ScheduledTask -TaskName 'StartWinAgent'\""
```
