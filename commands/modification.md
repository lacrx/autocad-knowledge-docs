# Modification Commands

## Move
```bash
mcp__autocad__entity_move(entity_id="handle", from_point="x1,y1", to_point="x2,y2")

# Direct — move last entity by offset
{"command":"paste","text":"_.MOVE _Last  x1,y1 x2,y2"}
```

## Copy
```bash
mcp__autocad__entity_copy(entity_id="handle", from_point="x1,y1", to_point="x2,y2")

{"command":"paste","text":"_.COPY _Last  x1,y1 x2,y2"}
```

## Rotate
```bash
mcp__autocad__entity_rotate(entity_id="handle", base_point="x,y", angle="90")

{"command":"paste","text":"_.ROTATE _Last  x,y 90"}
```

## Scale
```bash
mcp__autocad__entity_scale(entity_id="handle", base_point="x,y", scale_factor="2")
```

## Mirror
```bash
mcp__autocad__entity_mirror(
  entity_id="handle",
  point1="x1,y1", point2="x2,y2",
  delete_source="N")
```

## Offset
```bash
mcp__autocad__entity_offset(entity_id="handle", distance="3.5", side_point="x,y")

# Direct — offset a polyline 3.5" to one side
{"command":"paste","text":"_.OFFSET 3.5"}
# Then pick entity, then pick side
```

## Trim
```bash
# Direct — select cutting edges, then pick segments to trim
{"command":"paste","text":"_.TRIM"}
# Select boundary, Enter, then click segments to remove
```

## Extend
```bash
{"command":"paste","text":"_.EXTEND"}
# Select boundary edge, Enter, then click segments to extend
```

## Fillet
```bash
mcp__autocad__entity_fillet(
  entity_id1="handle1", entity_id2="handle2", radius="0")

# Direct — radius 0 = sharp corner
{"command":"paste","text":"_.FILLET _Radius 0"}
# Then pick two lines
```

## Chamfer
```bash
mcp__autocad__entity_chamfer(
  entity_id1="handle1", entity_id2="handle2",
  distance1="1", distance2="1")
```

## Array (rectangular)
```bash
mcp__autocad__entity_array(
  entity_id="handle", type="rectangular",
  rows="3", columns="4",
  row_spacing="48", column_spacing="36")
```

## Array (polar)
```bash
mcp__autocad__entity_array(
  entity_id="handle", type="polar",
  center="x,y", count="8", angle="360")
```

## Erase
```bash
mcp__autocad__entity_erase(entity_id="handle")

{"command":"paste","text":"_.ERASE _Last "}
```

## Undo / Redo
```bash
mcp__autocad__undo()
mcp__autocad__redo()

{"command":"paste","text":"_.UNDO"}
{"command":"paste","text":"_.REDO"}
```

## Break (split entity at point)
```bash
{"command":"paste","text":"_.BREAK _First x1,y1 x2,y2"}
```

## Join (merge collinear lines or arcs)
```bash
{"command":"paste","text":"_.JOIN"}
# Select entities to join
```

## Stretch (move vertices within window)
```bash
{"command":"paste","text":"_.STRETCH _Crossing x1,y1 x2,y2  base_x,base_y disp_x,disp_y"}
```
