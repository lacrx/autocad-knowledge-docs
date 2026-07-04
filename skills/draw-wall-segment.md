# Draw Wall Segment

## When to use
When adding or modifying walls in the floor plan.

## Steps

1. **Set correct layer**
```python
send_cmd({'request_id': 'w01', 'command': 'layer-set-current', 'name': 'Floor Plan'})
```

2. **Draw wall as closed polyline**
For a 2x4 wall (3.5" wide) from (x1,y1) to (x2,y2):
```python
# Horizontal wall example: 12'-0" long
x1, y1 = 120, 60   # Start point
x2 = 264            # End point (12' = 144" from start)
w = 3.5             # Wall thickness

points = f"{x1},{y1};{x2},{y1};{x2},{y1+w};{x1},{y1+w}"
send_cmd({'request_id': 'w02', 'command': 'create-polyline', 
          'points_str': points, 'closed': '1', 'layer': 'Floor Plan'})
```

3. **Add hatch fill**
```python
send_cmd({'request_id': 'w03', 'command': 'type', 
          'text': '(command "_.HATCH" "_Pattern" "SOLID" "_Select" "_Last" "" "")'})
```

4. **Validate**
- Check endpoints are exact integers or 1/16" increments
- Verify polyline is closed
- Confirm hatch fills correctly

## Wall Types

| Type | Width | Construction |
|------|-------|--------------|
| 2x4 stud | 3.5" | Single polyline + hatch |
| 2x6 stud | 5.5" | Single polyline + hatch |
| Tier 4 (this project) | 10.0" | Multiple layers shown in section |
| CMU 8" | 7.625" | Polyline + CMU hatch pattern |

## Corner Connections
- T-intersection: extend main wall, butt partition
- L-corner: extend one wall, butt other
- All intersections must have exact shared coordinates
