# MCP Bridge Troubleshooting

## Diagnostic Toolkit

### 1. Screenshot (most powerful)

When AutoCAD isn't responding as expected, take a screenshot to see what's actually on screen. Dialogs, prompts, error messages — all invisible without this.

```bash
# Push screenshot script to VM (one-time)
scp screenshot.py thomas@192.168.122.147:C:/temp/screenshot.py

# Take screenshot
echo '{"command":"exec","cmd":"python C:\\temp\\screenshot.py"}' | timeout 10 nc 192.168.122.147 9876

# Pull and view
scp thomas@192.168.122.147:C:/temp/screen.bmp /tmp/screen.bmp
convert /tmp/screen.bmp /tmp/screen.png
```

The screenshot script uses Win32 GDI (no PIL dependency):

```python
import ctypes, struct
user32 = ctypes.windll.user32
gdi32 = ctypes.windll.gdi32
w = user32.GetSystemMetrics(0)
h = user32.GetSystemMetrics(1)
hdc = user32.GetDC(0)
mem = gdi32.CreateCompatibleDC(hdc)
bm = gdi32.CreateCompatibleBitmap(hdc, w, h)
gdi32.SelectObject(mem, bm)
gdi32.BitBlt(mem, 0, 0, w, h, hdc, 0, 0, 0x00CC0020)
buf = (ctypes.c_char * (w * h * 4))()
gdi32.GetBitmapBits(bm, w * h * 4, buf)
with open(r'C:\temp\screen.bmp', 'wb') as f:
    f.write(struct.pack('<2sIHHI', b'BM', 54+w*h*4, 0, 0, 54))
    f.write(struct.pack('<IiiHHIIiiII', 40, w, -h, 1, 32, 0, w*h*4, 0, 0, 0, 0))
    f.write(buf)
gdi32.DeleteObject(bm); gdi32.DeleteDC(mem); user32.ReleaseDC(0, hdc)
print(f"Screenshot saved: {w}x{h}")
```

### 2. Ping (connectivity check)

```bash
echo '{"command":"ping"}' | timeout 5 nc 192.168.122.147 9876
```

Returns agent status and AutoCAD window info. Does NOT go through LISP dispatcher.

### 3. Diag (window tree)

```bash
echo '{"command":"diag"}' | timeout 5 nc 192.168.122.147 9876
```

Enumerates AutoCAD child windows. Useful for verifying window handles.

### 4. IPC file check

```bash
echo '{"command":"exec","cmd":"dir C:\\temp\\autocad_mcp_*"}' | timeout 5 nc 192.168.122.147 9876
```

Shows stale command/result files. If command files accumulate, dispatcher isn't consuming them.

---

## Common Issues

### Issue: `(load ...)` triggers APPLOAD dialog instead of LISP eval

**Symptom**: Typing `(load "C:/autocad_mcp/mcp_dispatch.lsp")` via `type` command opens the APPLOAD dialog. File name shows garbled text.

**Root cause**: AutoCAD LT 2026's command autocomplete intercepts `(load` and maps it to the APPLOAD command before the full LISP expression is received. Characters arrive one at a time via PostMessage WM_CHAR (10ms apart), giving autocomplete time to trigger.

**Fix**: Use the `_.SCRIPT` command to run a .scr file containing the LISP expression:

```bash
# Create load script (one-time, already on VM at C:/temp/load_dispatch.scr)
echo '(load "C:/autocad_mcp/mcp_dispatch.lsp")' > /tmp/load_dispatch.scr
scp /tmp/load_dispatch.scr thomas@192.168.122.147:C:/temp/load_dispatch.scr

# Load via SCRIPT command
echo '{"command":"type","text":"_.SCRIPT C:/temp/load_dispatch.scr"}' | timeout 10 nc 192.168.122.147 9876
```

The SCRIPT command reads from file, bypassing autocomplete entirely.

**Alternative**: Disable autocomplete: set INPUTSEARCHOPTIONS to 0 in AutoCAD. But this affects user's manual workflow.

---

### Issue: IPC commands timeout (drawing-info, entity-count, etc.)

**Symptom**: Any command that goes through the LISP dispatcher times out after 30s.

**Diagnosis tree**:

1. **Is the dispatcher loaded?**
   - Check for stale IPC command files in `C:\temp\` — if they accumulate, dispatcher isn't polling
   - Take a screenshot — look for command line prompt text at the bottom

2. **Is a dialog blocking?**
   - Take a screenshot — modal dialogs intercept all keyboard input
   - Common culprits: Save dialog, APPLOAD dialog, error alerts, "Drawing Recovery" panels
   - Dismiss with ESC via SendKeys:
     ```bash
     # SCP a dismiss script, then run it
     echo '{"command":"exec","cmd":"python -c \"import subprocess; subprocess.run([\\\"powershell\\\",\\\"-File\\\",\\\"C:\\\\temp\\\\dismiss_dialogs.ps1\\\"], timeout=10)\""}' | timeout 15 nc 192.168.122.147 9876
     ```

3. **Is AutoCAD in a command?**
   - If a command is active (LINE, MOVE, etc.), the dispatch call `(c:mcp-dispatch)` gets interpreted as input to that command
   - Send ESC first:
     ```bash
     echo '{"command":"exec","cmd":"python -c \"import win32gui,win32con; h=win32gui.FindWindow(None,None); [win32gui.PostMessage(197166,win32con.WM_KEYDOWN,win32con.VK_ESCAPE,0x00010001) for _ in range(3)]\""}' | timeout 5 nc 192.168.122.147 9876
     ```

4. **Clean up and reload**:
   ```bash
   # Delete stale IPC files
   echo '{"command":"exec","cmd":"del C:\\temp\\autocad_mcp_cmd_*.json C:\\temp\\autocad_mcp_result_*.json 2>nul"}' | timeout 5 nc 192.168.122.147 9876

   # Reload dispatcher via SCRIPT
   echo '{"command":"type","text":"_.SCRIPT C:/temp/load_dispatch.scr"}' | timeout 10 nc 192.168.122.147 9876
   ```

---

### Issue: Connection refused to port 9876

**Symptom**: `nc` or TCP connection to 192.168.122.147:9876 fails immediately.

**Diagnosis**:

1. **VM not running**:
   ```bash
   sg libvirt -c 'LIBVIRT_DEFAULT_URI=qemu:///system virsh domstate win11-autocad'
   # If "shut off": start it
   sg libvirt -c 'LIBVIRT_DEFAULT_URI=qemu:///system virsh start win11-autocad'
   ```

2. **win_agent.py not running**:
   ```bash
   ssh 192.168.122.147 "tasklist /FI \"IMAGENAME eq python.exe\" /V /FO CSV"
   # If not listed, start via scheduled task:
   ssh 192.168.122.147 "powershell -Command \"Start-ScheduledTask -TaskName 'StartWinAgent'\""
   ```

3. **Windows firewall blocking**:
   Run as admin on VM:
   ```powershell
   New-NetFirewallRule -DisplayName "AutoCAD MCP Agent" -Direction Inbound -Protocol TCP -LocalPort 9876 -Action Allow
   ```

---

### Issue: PostMessage WM_CHAR not reaching AutoCAD command line

**Symptom**: `type` command returns `ok: true` but AutoCAD doesn't execute the typed text.

**Root cause (AutoCAD LT 2026)**: No traditional Win32 Edit control for the command line — it uses Chrome/CEF embedded controls. PostMessage WM_CHAR to the main frame may not route to the command input.

**Fixes** (in order of reliability):

1. **Set foreground first**:
   ```bash
   echo '{"command":"ps","script":"Add-Type -Name W -Namespace A -MemberDefinition @\"\n[DllImport(\"user32.dll\")] public static extern bool SetForegroundWindow(IntPtr hWnd);\n\"@; [A.W]::SetForegroundWindow([IntPtr]197166)"}' | timeout 5 nc 192.168.122.147 9876
   ```

2. **Use SCRIPT command** for any LISP that needs to run:
   Write expression to .scr file, then `_.SCRIPT path`

3. **Use SendKeys via PowerShell** (requires foreground):
   SCP a .ps1 script, then run via `exec` command

4. **Use the dispatch trigger** (for IPC commands):
   `post_string_to_hwnd(acad_hwnd, "(c:mcp-dispatch)")` — this short string is less likely to be intercepted by autocomplete than longer LISP expressions

---

### Issue: Multiple AutoCAD instances, commands go to wrong one

**Symptom**: Commands execute in the wrong drawing.

**Fix**: Use the `target` field in requests:
```json
{"command": "drawing-info", "request_id": "x", "target": "LaCroix"}
```

The `target` value is matched as a case-insensitive substring against window titles. `find_autocad_window()` returns the first match.

Note: AutoCAD LT 2026 uses SDI (Single Document Interface) — each open DWG is a separate process with its own window.

---

### Issue: exec/ps commands return wrong output

**Symptom**: `exec` command with `code` field returns just "ok" or wrong output.

**Root cause**: The `exec` handler uses the `cmd` field (shell command string), not `code`.

**Correct usage**:
```json
{"command": "exec", "cmd": "dir C:\\temp"}
{"command": "ps", "script": "Get-Process | Select -First 5"}
```

---

## Recovery Sequence

When everything is broken, run this sequence:

```bash
# 1. Verify VM alive
sg libvirt -c 'LIBVIRT_DEFAULT_URI=qemu:///system virsh domstate win11-autocad'

# 2. Verify agent alive
echo '{"command":"ping"}' | timeout 5 nc 192.168.122.147 9876

# 3. Take screenshot to see AutoCAD state
echo '{"command":"exec","cmd":"python C:\\temp\\screenshot.py"}' | timeout 10 nc 192.168.122.147 9876
scp thomas@192.168.122.147:C:/temp/screen.bmp /tmp/screen.bmp && convert /tmp/screen.bmp /tmp/screen.png

# 4. Dismiss any dialogs
# (write dismiss_dialogs.ps1 to VM, run it)

# 5. Send ESC to cancel any active command
echo '{"command":"exec","cmd":"python -c \"import win32gui,win32con,time; h=197166; [win32gui.PostMessage(h,win32con.WM_KEYDOWN,27,0x00010001) for _ in range(3)]\""}' | timeout 5 nc 192.168.122.147 9876

# 6. Clean stale IPC files
echo '{"command":"exec","cmd":"del C:\\temp\\autocad_mcp_cmd_*.json C:\\temp\\autocad_mcp_result_*.json 2>nul"}' | timeout 5 nc 192.168.122.147 9876

# 7. Reload dispatcher via SCRIPT
echo '{"command":"type","text":"_.SCRIPT C:/temp/load_dispatch.scr"}' | timeout 10 nc 192.168.122.147 9876

# 8. Wait and test
sleep 5
echo '{"request_id":"recovery","command":"drawing-info"}' | timeout 15 nc 192.168.122.147 9876
```

---

## Prevention

1. **Always load dispatcher via SCRIPT**, never via `type` with `(load ...)`
2. **Always check for stale IPC files** before sending new commands after a failure
3. **Take a screenshot** as first diagnostic step — saves time guessing
4. **Keep `load_dispatch.scr` and `screenshot.py` on VM** at `C:\temp\` — they're reused constantly
5. **Use `ping` before IPC commands** — confirms agent is alive without touching the LISP dispatcher
