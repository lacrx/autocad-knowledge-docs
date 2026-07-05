# Start AutoCAD Bridge

## When to use
Before any AutoCAD drawing work, to establish the MCP bridge connection.

## Steps

1. **Check VM status**
```bash
sg libvirt -c 'LIBVIRT_DEFAULT_URI=qemu:///system virsh domstate win11-autocad'
```
If not running:
```bash
sg libvirt -c 'LIBVIRT_DEFAULT_URI=qemu:///system virsh start win11-autocad'
```

2. **Verify win_agent.py is running**
```bash
echo '{"command":"ping"}' | timeout 5 nc 192.168.122.147 9876
```
If connection refused:
```bash
ssh 192.168.122.147 "powershell -Command \"Start-ScheduledTask -TaskName 'StartWinAgent'\""
```
Wait 5s, retry ping.

3. **Check ping response**
- `autocad_found: true` → AutoCAD is running
- `autocad_found: false` → Open AutoCAD manually on VM, or:
  ```bash
  ssh 192.168.122.147 'start "" "Z:\LaCroix_ADU.dwg"'
  ```

4. **Load LISP dispatcher via SCRIPT command**

IMPORTANT: Do NOT use `(load ...)` via the `type` command — AutoCAD's autocomplete intercepts it and opens the APPLOAD dialog. Use `_.SCRIPT` instead:

```bash
# Ensure load script exists on VM (one-time setup)
echo '(load "C:/autocad_mcp/mcp_dispatch.lsp")' > /tmp/load_dispatch.scr
scp /tmp/load_dispatch.scr thomas@192.168.122.147:C:/temp/load_dispatch.scr

# Load dispatcher
echo '{"command":"type","text":"_.SCRIPT C:/temp/load_dispatch.scr"}' | timeout 10 nc 192.168.122.147 9876
```

5. **Verify full pipeline (wait 3-5s after load)**
```bash
sleep 5
echo '{"request_id":"verify","command":"drawing-info"}' | timeout 15 nc 192.168.122.147 9876
```
Should return entity count and layer list.

## Troubleshooting

| Issue | Diagnostic | Fix |
|-------|-----------|-----|
| Connection refused | VM or agent not running | Start VM, then scheduled task |
| Ping OK but drawing-info timeout | Dispatcher not loaded or dialog blocking | Take screenshot, dismiss dialogs, reload via SCRIPT |
| `type` sends OK but no effect | PostMessage not reaching command line | Set foreground first, use SCRIPT for LISP |
| APPLOAD dialog appears | Used `(load ...)` via `type` | Dismiss with ESC, use `_.SCRIPT` instead |
| Stale IPC files in C:\temp | Previous failed commands | `del C:\temp\autocad_mcp_cmd_*.json` then reload |
| Wrong AutoCAD instance | Multiple windows open | Add `"target": "LaCroix"` to requests |

See [troubleshooting article](../articles/mcp-bridge/troubleshooting.md) for full recovery sequence.
