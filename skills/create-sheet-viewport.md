# Create Sheet Viewport

## When to use
When setting up a new layout/sheet that needs to display model space content.

## Steps

1. **Switch to paper space layout**
```python
send_cmd({'command': 'type', 'text': '(command "_.LAYOUT" "_Set" "A1 - Floor Plan")'})
```

2. **Create viewport**
```python
# Viewport covering the main drawing area
send_cmd({'command': 'type', 'text': '(command "_.MVIEW" "5,3" "30,20")'})
```

3. **Set viewport scale**
```python
# Enter model space in viewport, set scale
send_cmd({'command': 'type', 'text': 'MSPACE'})
send_cmd({'command': 'type', 'text': 'ZOOM 1/48XP'})  # 1/4" = 1'-0"
```

4. **Center on target region**
```python
# Pan to the correct model space region
send_cmd({'command': 'type', 'text': '(command "_.PAN" "230,100" "17,12")'})
```

5. **Lock viewport**
```python
send_cmd({'command': 'type', 'text': 'PSPACE'})
# Select viewport and lock
```

6. **Freeze layers for this viewport**
```python
send_cmd({'command': 'type', 'text': 'MSPACE'})
send_cmd({'command': 'type', 'text': '(command "_.VPLAYER" "_Freeze" "Electrical - Components" "_Current" "")'})
```

## Common Scale Factors
| Scale | Zoom Factor | DIMSCALE |
|-------|-------------|----------|
| 1/4" = 1'-0" | 1/48XP | 48 |
| 3/8" = 1'-0" | 1/32XP | 32 |
| 1/2" = 1'-0" | 1/24XP | 24 |
| 1" = 20'-0" | 1/240XP | 240 |
