# Send AutoCAD Command

## When to use
To execute any AutoCAD command through the MCP bridge.

## Method 1: MCP Tool (preferred)
```
mcp__autocad__create-line x1=0 y1=0 x2=144 y2=0
mcp__autocad__create-rectangle x1=0 y1=0 x2=228 y2=272
mcp__autocad__layer-create name="FP-WALLS" color=7
```

## Method 2: Direct TCP (when MCP server unavailable)
```python
import socket, json

def send_cmd(cmd):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.settimeout(35)
    s.connect(('192.168.122.147', 9876))
    s.sendall(json.dumps(cmd).encode() + b'\n')
    data = s.recv(16384)
    s.close()
    return json.loads(data)

# IPC commands (go through LISP dispatcher)
send_cmd({'request_id': 'id001', 'command': 'create-line', 'x1': 0, 'y1': 0, 'x2': 144, 'y2': 0})

# Direct commands (handled by win_agent.py)
send_cmd({'command': 'type', 'text': '(command "_.LINE" "0,0" "144,0" "")'})
send_cmd({'command': 'exec', 'cmd': 'echo hello'})
```

## Method 3: LISP Execution
```python
# Execute a LISP file on the VM
send_cmd({'request_id': 'id002', 'command': 'execute-lisp', 'code_file': 'C:/autocad_mcp/lisp/standards/setup-layers.lsp'})
```

## Target Specific Instance
Add `"target": "LaCroix"` to target a specific AutoCAD window by title substring.

## Key Rules
- Always use exact coordinates (no approximations)
- Set the correct layer before creating entities
- Validate after creating geometry
- Use `request_id` for IPC commands (required for LISP dispatcher)
