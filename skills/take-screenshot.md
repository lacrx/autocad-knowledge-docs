# Take Screenshot

## When to use
When AutoCAD isn't responding as expected — dialogs, errors, or unexpected state. First diagnostic step for any bridge issue.

## Prerequisites
`screenshot.py` must exist on VM at `C:\temp\screenshot.py`. If missing, push it:
```bash
scp /home/thomas/repos/cad/mcp_server/screenshot.py thomas@192.168.122.147:C:/temp/screenshot.py
```

## Steps

1. **Capture**
```bash
echo '{"command":"exec","cmd":"python C:\\temp\\screenshot.py"}' | timeout 10 nc 192.168.122.147 9876
```

2. **Pull to Linux**
```bash
scp thomas@192.168.122.147:C:/temp/screen.bmp /tmp/screen.bmp
```

3. **Convert and view**
```bash
convert /tmp/screen.bmp /tmp/screen.png
# Then read /tmp/screen.png to view
```

## What to look for

- **Modal dialogs** — Save, APPLOAD, error alerts, Drawing Recovery. These block all command line input.
- **Command line text** — Bottom of screen shows current command state. Look for `Command:` (ready) vs active command prompts.
- **Active tab** — Model vs layout tab (bottom of screen)
- **Error messages** — Yellow/red warning text in command history area

## Notes
- Does not require AutoCAD to be foreground
- Uses Win32 GDI — no PIL/Pillow dependency
- Output is BMP (no PNG encoder in stdlib) — convert on Linux side
- Captures entire screen, not just AutoCAD window
