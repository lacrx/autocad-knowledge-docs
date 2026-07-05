# Dimension Commands

## Linear dimension (horizontal or vertical)
```bash
# MCP
mcp__autocad__create_dimension_linear(
  point1="x1,y1", point2="x2,y2",
  text_position="xm,ym",  # where dimension text sits
  layer="FP-DIM")

# Direct — pick two points, then placement
{"command":"paste","text":"_.DIMLINEAR x1,y1 x2,y2 ym"}
```

## Aligned dimension (follows line angle)
```bash
mcp__autocad__create_dimension_aligned(
  point1="x1,y1", point2="x2,y2",
  text_position="xm,ym", layer="FP-DIM")

{"command":"paste","text":"_.DIMALIGNED x1,y1 x2,y2 offset_dist"}
```

## Angular dimension
```bash
mcp__autocad__create_dimension_angular(
  vertex="x,y", point1="x1,y1", point2="x2,y2",
  text_position="xm,ym", layer="FP-DIM")
```

## Radius dimension
```bash
mcp__autocad__create_dimension_radius(
  entity_id="handle_or_last", text_position="x,y", layer="FP-DIM")
```

## Continuous dimensions (chain)
```bash
# After first linear dim, continue from last extension line
{"command":"paste","text":"_.DIMCONTINUE x2,y2"}
# Repeat for each segment, Enter twice to end
```

## Baseline dimensions
```bash
# All measured from same baseline
{"command":"paste","text":"_.DIMBASELINE x2,y2"}
```

## Leader with text
```bash
mcp__autocad__create_leader(
  points_str="x1,y1;x2,y2;x3,y3",
  text="NOTE TEXT", layer="FP-NOTE")

# Direct LISP
{"command":"paste","text":"(command \"_.LEADER\" \"x1,y1\" \"x2,y2\" \"\" \"NOTE TEXT\" \"\")"}
```

## Dimension styles

### Set architectural style
```bash
# Feet-inches format, tick marks, 3/32" text
{"command":"paste","text":"_.DIMSTYLE _Restore Architectural"}
```

### Override text size
```bash
{"command":"paste","text":"(setvar \"DIMTXT\" 0.09375)"}  ; 3/32"
```

### Override arrow/tick
```bash
{"command":"paste","text":"(setvar \"DIMBLK\" \"_ARCHTICK\")"}  ; architectural tick
{"command":"paste","text":"(setvar \"DIMASZ\" 0.09375)"}        ; tick size
```

## Dimension placement guidelines
- Exterior dims: 2'-0" from building face, stacked at 1'-0" spacing
- Interior dims: centered in room or between walls
- Continuous chains for overall dimensions
- String dims align to consistent datum line
