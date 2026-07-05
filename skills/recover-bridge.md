# Recover Bridge

## When to use
When the MCP bridge is in a broken state — commands timing out, dialogs blocking, stale IPC files. Full recovery from any failure state.

## Steps

1. **Verify agent alive**
```bash
echo '{"command":"ping"}' | timeout 5 nc 192.168.122.147 9876
```
If connection refused → start VM and agent (see [start-autocad-bridge](start-autocad-bridge.md)).

2. **Take screenshot**
```bash
echo '{"command":"exec","cmd":"python C:\\temp\\screenshot.py"}' | timeout 10 nc 192.168.122.147 9876
scp thomas@192.168.122.147:C:/temp/screen.bmp /tmp/screen.bmp
convert /tmp/screen.bmp /tmp/screen.png
```
Look for: dialogs, error messages, active command prompts.

3. **Dismiss dialogs** (if screenshot shows any)
```bash
cat > /tmp/dismiss.ps1 << 'EOF'
Add-Type -AssemblyName System.Windows.Forms
[System.Windows.Forms.SendKeys]::SendWait('{ENTER}')
Start-Sleep -Milliseconds 500
[System.Windows.Forms.SendKeys]::SendWait('{ESCAPE}')
Start-Sleep -Milliseconds 300
[System.Windows.Forms.SendKeys]::SendWait('{ESCAPE}')
Start-Sleep -Milliseconds 300
[System.Windows.Forms.SendKeys]::SendWait('{ESCAPE}')
EOF
scp /tmp/dismiss.ps1 thomas@192.168.122.147:C:/temp/dismiss_dialogs.ps1
echo '{"command":"exec","cmd":"powershell -File C:\\temp\\dismiss_dialogs.ps1"}' | timeout 10 nc 192.168.122.147 9876
```

4. **Cancel active commands** (ESC via PostMessage)
```bash
echo '{"command":"exec","cmd":"python -c \"import win32gui,win32con; h=197166; [win32gui.PostMessage(h,win32con.WM_KEYDOWN,27,0x00010001) for _ in range(3)]\""}' | timeout 5 nc 192.168.122.147 9876
```
Note: hwnd 197166 is for current session — get fresh hwnd from ping response if different.

5. **Clean stale IPC files**
```bash
echo '{"command":"exec","cmd":"del C:\\temp\\autocad_mcp_cmd_*.json C:\\temp\\autocad_mcp_result_*.json 2>nul & echo cleaned"}' | timeout 5 nc 192.168.122.147 9876
```

6. **Reload dispatcher**
```bash
echo '{"command":"type","text":"_.SCRIPT C:/temp/load_dispatch.scr"}' | timeout 10 nc 192.168.122.147 9876
```

7. **Verify** (wait 3-5s)
```bash
sleep 5
echo '{"request_id":"recovery","command":"drawing-info"}' | timeout 15 nc 192.168.122.147 9876
```

## If verification still fails

- Retake screenshot — new dialog may have appeared
- Check if AutoCAD crashed: `echo '{"command":"ping"}' | ...` — if `autocad_found: false`, AutoCAD needs restart
- Check if drawing file is still open: ping response includes window titles
- Nuclear option: restart win_agent.py via scheduled task, reopen drawing manually
