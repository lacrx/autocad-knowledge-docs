# Wall Construction in AutoCAD

## The Right Way

Walls in a professional plan set are drawn as:
1. **Parallel polylines** defining the wall cavity boundaries
2. **Hatch fill** between the polylines (SOLID pattern for section cuts)

This produces proper architectural cut-section appearance where wall cavities are filled and structural elements are visible.

## Wrong Approach

Drawing walls as separate rectangles per wall segment creates:
- Disconnected geometry at corners
- No proper cavity fill
- Gaps and overlaps that look unprofessional
- No structural representation

## Construction Method

### Simple Wall Segment

```lisp
; Draw a wall from (x1,y1) to (x2,y2) with given thickness
(defun draw-wall (x1 y1 x2 y2 thickness / angle perp-angle dx dy)
  ; Calculate perpendicular direction
  (setq angle (angle (list x1 y1) (list x2 y2)))
  (setq perp-angle (+ angle (/ pi 2)))
  (setq dx (* thickness (cos perp-angle)))
  (setq dy (* thickness (sin perp-angle)))
  
  ; Draw outer polyline
  (command "_.PLINE" 
    (strcat (rtos x1 2 4) "," (rtos y1 2 4))
    (strcat (rtos x2 2 4) "," (rtos y2 2 4))
    (strcat (rtos (+ x2 dx) 2 4) "," (rtos (+ y2 dy) 2 4))
    (strcat (rtos (+ x1 dx) 2 4) "," (rtos (+ y1 dy) 2 4))
    "_Close")
  
  ; Apply hatch
  (command "_.HATCH" "_Pattern" "SOLID" "_Select" "_Last" "" ""))
```

### Wall with Interior Cavity (e.g., 2x4 stud wall)

For walls showing framing in section:
1. Draw outer face polyline
2. Draw inner face polyline  
3. Hatch between them
4. Optionally add stud lines at 16" o.c.

### Corner Connections

At corners, walls must intersect cleanly:
- Extend one wall through the corner
- Butt the other wall against it
- Trim as needed for proper appearance
- Hatch each wall's cavity independently

## Coordinate Precision

All wall endpoints must snap to exact coordinates:
- Use integer values for dimensions in inches
- 1/16" minimum precision: use multiples of 0.0625
- Never approximate — 6'-0" is exactly 72.0, not 71.98
- For stud walls: 3.5" actual width for 2x4, 5.5" for 2x6

## Common Wall Thicknesses (inches)

| Assembly | Framing | Total with Finish |
|----------|---------|-------------------|
| 2x4 interior | 3.5 | 4.5 (+ 1/2" GWB each side) |
| 2x4 exterior | 3.5 | Varies with assembly |
| 2x6 exterior | 5.5 | Varies with assembly |
| Tier 4 STC (this project) | 3.5 | 3.5 + 3.25 per face = 10.0 |

## Layer Assignment

| Element | Layer |
|---------|-------|
| Wall outlines (cut) | FP-WALLS or Floor Plan |
| Interior partitions | FP-WALLS-INT |
| Wall cavity hatch | Same as wall layer or FP-HATCH |
| Structural elements (shear) | STRUCT-SHEAR or SHEAR WALLS |

## From PDS671 Professional Reference

The professional plan set uses:
- **32 LWPOLYLINE** entities on the Floor Plan layer for wall boundaries
- **10 HATCH** entities on the Floor Plan layer for wall fills
- **4 HATCH** entities on SHEAR WALLS layer for shear panel fills
- Walls are continuous polylines, not disconnected line segments
