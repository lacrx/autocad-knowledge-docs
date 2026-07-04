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
ssh 192.168.122.147 "tasklist /FI \"IMAGENAME eq python.exe\" /V /FO CSV"
```
If not running:
```bash
ssh 192.168.122.147 "powershell -Command \"Start-ScheduledTask -TaskName 'StartWinAgent'\""
```

3. **Test connectivity**
Call `mcp__autocad__ping` or:
```python
import socket, json
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.settimeout(10)
s.connect(('192.168.122.147', 9876))
s.sendall(json.dumps({'command': 'ping'}).encode() + b'\n')
print(json.loads(s.recv(4096)))
s.close()
```

4. **Load LISP dispatcher in AutoCAD**
```python
send_cmd({'command': 'type', 'text': '(load "C:/autocad_mcp/mcp_dispatch.lsp")'})
```

5. **Verify full pipeline**
```python
send_cmd({'request_id': 'test', 'command': 'drawing-info'})
```
Should return entity count and layer list.

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Connection refused | VM not running or win_agent.py not started |
| Timeout on drawing-info | LISP dispatcher not loaded — reload it |
| "AutoCAD not found" | AutoCAD LT not running — open it manually or via `start` |
| PostMessage not reaching | AutoCAD may not be foreground window |
