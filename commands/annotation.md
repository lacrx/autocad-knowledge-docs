# Annotation Commands

## Section mark
```bash
# Insert section mark block with identifier
mcp__autocad__block_insert_with_attributes(
  name="SECT-MARK", insertion_point="x,y",
  scale="1", rotation="angle",
  attributes={"SECTION": "A", "SHEET": "A5"})
```

## Elevation mark
```bash
mcp__autocad__block_insert_with_attributes(
  name="ELEV-MARK", insertion_point="x,y",
  scale="1", rotation="0",
  attributes={"ELEV": "1", "SHEET": "A3"})
```

## Detail callout
```bash
# Circle with detail number and sheet reference
mcp__autocad__block_insert_with_attributes(
  name="DETAIL-BUBBLE", insertion_point="x,y",
  scale="1", rotation="0",
  attributes={"DETAIL": "3", "SHEET": "A6"})
```

## North arrow
```bash
mcp__autocad__block_insert(
  name="NORTH-ARROW", insertion_point="x,y",
  scale="1", rotation="0", layer="PLOT-TITLE")
```

## Scale bar
```bash
mcp__autocad__block_insert(
  name="SCALE-BAR", insertion_point="x,y",
  scale="1", rotation="0", layer="PLOT-TITLE")
```

## Revision cloud
```bash
# Draw revision cloud around changed area
{"command":"paste","text":"_.REVCLOUD _ARC 6 12"}
# Then pick points around the area, close
```

## Room number tag
```bash
# Circle with room number inside
{"command":"paste","text":"(progn (command \"_.CIRCLE\" \"x,y\" \"6\") (command \"_.TEXT\" \"_Justify\" \"_MC\" \"x,y\" \"3\" \"0\" \"101\"))"}
```

## Keynote / callout
```bash
# Leader line with keynote number
{"command":"paste","text":"(command \"_.LEADER\" \"x1,y1\" \"x2,y2\" \"\" \"5\" \"\")"}
# 5 = keynote number referencing a schedule
```

## Window / door tags
```bash
# Hexagon (or circle) with identifier
# W1, W2, D1, D2, etc.
mcp__autocad__block_insert_with_attributes(
  name="WIN-TAG", insertion_point="x,y",
  attributes={"TAG": "W1"})
```

## Spot elevation
```bash
# X mark with elevation text
{"command":"paste","text":"(progn (command \"_.LINE\" \"x-1,y-1\" \"x+1,y+1\" \"\") (command \"_.LINE\" \"x+1,y-1\" \"x-1,y+1\" \"\") (command \"_.TEXT\" \"_Justify\" \"_BL\" \"x+2,y\" \"1.5\" \"0\" \"+8'-0\\\"\"))"}
```

## Drawing title (under viewport in paper space)
```bash
# Centered text below viewport
{"command":"paste","text":"(progn (command \"_.TEXT\" \"_Justify\" \"_MC\" \"18,1\" \"0.1875\" \"0\" \"FLOOR PLAN\") (command \"_.TEXT\" \"_Justify\" \"_MC\" \"18,0.65\" \"0.125\" \"0\" \"SCALE: 1/4\\\" = 1'-0\\\"\"))"}
```
