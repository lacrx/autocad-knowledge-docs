# Setup Standard Layers

## When to use
When starting a new drawing or verifying layer standards on an existing drawing.

## Steps

1. **Execute the layer setup script**
```python
send_cmd({'request_id': 'layers01', 'command': 'execute-lisp',
          'code_file': 'C:/autocad_mcp/lisp/standards/setup-layers.lsp'})
```

2. **Verify layers created**
```python
send_cmd({'request_id': 'layers02', 'command': 'layer-list'})
```

3. **Set text and dimension styles**
```python
send_cmd({'request_id': 'text01', 'command': 'execute-lisp',
          'code_file': 'C:/autocad_mcp/lisp/standards/setup-text-styles.lsp'})
send_cmd({'request_id': 'dim01', 'command': 'execute-lisp',
          'code_file': 'C:/autocad_mcp/lisp/standards/setup-dim-styles.lsp'})
```

## Standard Layer List
See [layers-and-organization](../articles/autocad-lt/layers-and-organization.md) for the full layer scheme with colors, linetypes, and lineweights.
