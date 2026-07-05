# Send AutoCAD Command

## When to use
To execute any AutoCAD command through the MCP bridge.

## Method 1: MCP Tool (preferred when tools available)
```
mcp__autocad__create-line x1=0 y1=0 x2=144 y2=0
mcp__autocad__create-rectangle x1=0 y1=0 x2=228 y2=272
mcp__autocad__layer-create name="FP-WALLS" color=7
```

## Method 2: Direct TCP (when MCP tools not registered)
```bash
# IPC commands (go through LISP dispatcher — need request_id)
echo '{"request_id":"id01","command":"drawing-info"}' | timeout 15 nc 192.168.122.147 9876

echo '{"request_id":"id02","command":"create-line","x1":0,"y1":0,"x2":144,"y2":0}' | timeout 15 nc 192.168.122.147 9876

# Direct commands (handled by win_agent.py — no request_id needed)
echo '{"command":"ping"}' | timeout 5 nc 192.168.122.147 9876
echo '{"command":"type","text":"_.ZOOM E"}' | timeout 5 nc 192.168.122.147 9876
echo '{"command":"exec","cmd":"dir C:\\temp"}' | timeout 5 nc 192.168.122.147 9876
```

## Method 3: LISP Execution
```bash
# Execute a LISP file already on the VM
echo '{"request_id":"id03","command":"execute-lisp","code_file":"C:/autocad_mcp/lisp/standards/setup-layers.lsp"}' | timeout 30 nc 192.168.122.147 9876
```

## Method 4: SCRIPT command (for LISP expressions)

When you need to run a LISP expression that would be intercepted by autocomplete:

```bash
# Write expression to .scr file
echo '(your-lisp-expression)' > /tmp/expr.scr
scp /tmp/expr.scr thomas@192.168.122.147:C:/temp/expr.scr

# Run via SCRIPT
echo '{"command":"type","text":"_.SCRIPT C:/temp/expr.scr"}' | timeout 10 nc 192.168.122.147 9876
```

## Target Specific Instance
Add `"target": "LaCroix"` to target a specific AutoCAD window by title substring.

## Key Rules

1. Use `request_id` for IPC commands (required for LISP dispatcher to match command→result)
2. Use exact coordinates — no approximations
3. Set correct layer before creating entities
4. Validate after creating geometry
5. Direct commands (`ping`, `exec`, `ps`, `type`) do NOT need `request_id`
6. LISP expressions via `type` may trigger autocomplete — use SCRIPT method instead
7. If command times out, check for stale IPC files and dialog state before retrying

## See Also
- [MCP Bridge Troubleshooting](../articles/mcp-bridge/troubleshooting.md)
