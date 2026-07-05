# Wall Commands

## Standard wall segment (polyline + hatch)

```bash
# 2x4 wall (3.5" thick), 12'-0" long, starting at (120, 60)
# Step 1: Set layer
mcp__autocad__layer_set_current(name="FP-WALL")

# Step 2: Draw closed polyline
x1=120; y1=60; length=144; w=3.5
mcp__autocad__create_polyline(
  points_str="120,60;264,60;264,63.5;120,63.5",
  closed="1", layer="FP-WALL")

# Step 3: Hatch fill
{"command":"paste","text":"(command \"_.HATCH\" \"_Pattern\" \"SOLID\" \"_Select\" \"_Last\" \"\" \"\")"}
```

## Wall thickness reference

| Type | Thickness | Usage |
|------|-----------|-------|
| 2×4 stud | 3.5" | Interior partition |
| 2×4 + 1/2" GWB each side | 4.5" | Typical interior |
| 2×6 stud | 5.5" | Exterior, plumbing |
| 2×6 + 5/8" GWB each side | 6.75" | Typical exterior |
| Tier 4 assembly | 10.0" | STC-rated (this project) |
| 8" CMU | 7.625" | Foundation |

## LISP: Draw wall between two points

```lisp
; Usage: (draw-wall '(120 60) '(264 60) 3.5 "FP-WALL")
(defun draw-wall (p1 p2 thick layer / x1 y1 x2 y2 dx dy nx ny len)
  (setq x1 (car p1) y1 (cadr p1) x2 (car p2) y2 (cadr p2))
  (setq dx (- x2 x1) dy (- y2 y1))
  (setq len (sqrt (+ (* dx dx) (* dy dy))))
  (setq nx (/ (- dy) len) ny (/ dx len))
  (setq nx (* nx thick) ny (* ny thick))
  (command "_.LAYER" "_Set" layer "")
  (command "_.PLINE"
    (list x1 y1) (list x2 y2)
    (list (+ x2 nx) (+ y2 ny)) (list (+ x1 nx) (+ y1 ny)) "_Close")
  (command "_.HATCH" "_Pattern" "SOLID" "_Select" "_Last" "" "")
)
```

## Corner connections

### L-corner (two walls meeting at 90°)
```
# Wall A runs right, Wall B runs up from A's endpoint
# Wall A: (x1,y1) to (x2,y2), width w
# Wall B starts at (x2,y1) going up
# Extend Wall A to cover corner: end at x2+w (not x2)
```

### T-intersection (partition meeting main wall)
```
# Main wall is continuous
# Partition butts against main wall — does NOT extend through it
# Leave gap in main wall hatch if showing partition attachment
```

## Door opening in wall
```bash
# Cut opening: erase wall segment, redraw with gap
# 3'-0" door at position x=180 in wall from y=60
# Opening width = 36" (3'-0"), no frame in plan
# Left wall: 120,60 to 180,60
# Right wall: 216,60 to 264,60
# (skip 180-216 = 36" opening)
```

## Window opening in wall
```bash
# Windows shown with two lines across opening (sill and header lines)
# Opening width varies: 24", 36", 48" typical
# Draw wall segments on each side of opening
# Draw two parallel lines across opening at 1/3 and 2/3 of wall thickness
```
