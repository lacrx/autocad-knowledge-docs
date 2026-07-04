# Execute LISP Script

## When to use
To run curated LISP scripts from the library on the VM.

## Steps

1. **Push script to VM (if not already there)**
```bash
scp /home/thomas/repos/cad/lisp/path/to/script.lsp 192.168.122.147:'C:\autocad_mcp\lisp\path\to\script.lsp'
```

2. **Execute via MCP**
```python
send_cmd({'request_id': 'lisp01', 'command': 'execute-lisp',
          'code_file': 'C:/autocad_mcp/lisp/path/to/script.lsp'})
```

3. **Or execute via type (for one-liners)**
```python
send_cmd({'command': 'type', 'text': '(load "C:/autocad_mcp/lisp/standards/setup-layers.lsp")'})
```

## Available Script Categories

| Path | Purpose |
|------|---------|
| `lisp/core/` | Foundation: doc switching, dispatch reload |
| `lisp/drawing/` | Create geometry: wall stacks, hatches |
| `lisp/dimensions/` | Dimension management and scanning |
| `lisp/layers/` | Layer queries and management |
| `lisp/viewports/` | Viewport inspection and management |
| `lisp/export/` | PDF export and plotting |
| `lisp/validation/` | Drawing accuracy checks |
| `lisp/standards/` | Text styles, dim styles, layer setup |

## Best Practice
Prefer curated library scripts over generating raw LISP. The library is tested and handles edge cases. Use the LLM to compose sequences of library calls, not to write new LISP from scratch.
