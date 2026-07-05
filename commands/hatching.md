# Hatching Commands

## Solid fill (walls, filled areas)
```bash
mcp__autocad__create_hatch(
  pattern="SOLID", boundary_entity_id="handle",
  layer="FP-HATCH")

# LISP — hatch last entity (polyline)
{"command":"paste","text":"(command \"_.HATCH\" \"_Pattern\" \"SOLID\" \"_Select\" \"_Last\" \"\" \"\")"}
```

## Pattern hatch
```bash
# Concrete pattern
mcp__autocad__create_hatch(
  pattern="AR-CONC", scale="1", angle="0",
  boundary_entity_id="handle", layer="FP-HATCH")

# Direct — pick internal point method
{"command":"paste","text":"_.HATCH _Pattern AR-CONC 1 0 _Pick x,y "}
```

## Common architectural hatch patterns

| Pattern | Scale | Usage |
|---------|-------|-------|
| SOLID | — | Wall fills, solid areas |
| AR-CONC | 1 | Concrete (foundation plan) |
| AR-SAND | 1 | Earth/soil (sections) |
| AR-BRSTD | 1 | Brick (standard) |
| AR-BRELK | 1 | Brick (running bond) |
| ANSI31 | 1 | General section cut |
| ANSI32 | 1 | Steel section |
| ANSI37 | 1 | Insulation |
| AR-RSHKE | 1 | Roof shingles |
| GRAVEL | 1 | Gravel fill |
| EARTH | 1 | Earth fill |
| INSUL | 1 | Batt insulation |

## Hatch by picking internal point
```bash
{"command":"paste","text":"_.HATCH _Pattern SOLID _Pick x,y "}
# x,y must be inside a closed boundary
```

## Edit hatch properties
```bash
{"command":"paste","text":"_.HATCHEDIT"}
# Then pick the hatch entity
```

## Gradient fill
```bash
{"command":"paste","text":"_.GRADIENT"}
# Interactive — pick pattern and colors
```

## Notes
- Hatch must have closed boundary (polyline or enclosed area)
- Hatch associativity: by default, hatch follows boundary changes
- Place hatch on its own layer (FP-HATCH) with lineweight 0.00
- For wall hatches, use SOLID on FP-HATCH layer
- For section cuts, use ANSI31 at appropriate scale
