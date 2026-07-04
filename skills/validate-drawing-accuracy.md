# Validate Drawing Accuracy

## When to use
After every batch of drawing modifications.

## Steps

1. **Load validation scripts**
```python
send_cmd({'request_id': 'val01', 'command': 'execute-lisp', 
          'code_file': 'C:/autocad_mcp/lisp/validation/validate_snap.lsp'})
```

2. **Run snap precision check**
Verify all endpoints land on 1/16" grid (multiples of 0.0625").

3. **Run dimension consistency check**
Compare dimension text values to actual distances between definition points.

4. **Run layer audit**
Verify entities are on expected layers for their type.

5. **Run endpoint connectivity check**
Find wall endpoints that should connect but have gaps.

6. **Review results**
- 0 errors → proceed
- Errors found → fix before continuing
- Never skip validation to "save time"

## Error Correction Loop
```
while errors > 0:
    identify_error()
    fix_error()
    revalidate()
```

## Key Validation Thresholds
- Snap tolerance: 0.001" (anything > this is a snap error)
- Dimension tolerance: 1/16" (dimension text should match geometry within this)
- Gap tolerance: 0.01" (endpoints closer than this should be coincident)
