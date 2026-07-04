# Paper Space and Viewports in AutoCAD LT

## Model Space vs Paper Space

- **Model space**: Where geometry is drawn at full scale (1 inch = 1 inch in real life). All walls, dimensions, fixtures live here.
- **Paper space**: Where sheets are composed for printing. Contains the title block, viewports into model space, and sheet-specific annotations.

Each layout tab is a separate paper space. A drawing can have many layouts (one per sheet).

## Viewports

A viewport is a window in paper space that shows a portion of model space at a specific scale.

### Creating Viewports

```
; In paper space
MVIEW  ; Make viewport
; Specify corners or use Fit option
```

Via LISP:
```lisp
(command "_.MVIEW" pt1 pt2)
```

### Setting Viewport Scale

1. Double-click viewport to enter model space
2. Set zoom scale: `ZOOM 1/48XP` (for 1/4" = 1'-0")
3. Double-click outside viewport to return to paper space
4. Lock viewport: select it, Properties → Display Locked = Yes

Common scales:
| Scale | Zoom factor | Use |
|-------|-------------|-----|
| 1/4" = 1'-0" | 1/48XP | Floor plans, elevations |
| 3/8" = 1'-0" | 1/32XP | Details, sections |
| 1/2" = 1'-0" | 1/24XP | Large details |
| 1" = 1'-0" | 1/12XP | Full-size details |
| 1" = 20'-0" | 1/240XP | Site/plot plan |

### Viewport Layer Freezing

Control which layers show in each viewport independently:

```
; Inside a viewport (model space)
VPLAYER
F           ; Freeze
layer-name  ; Layer to freeze
C           ; Current viewport
[Enter]     ; Done
```

This is how one model space serves multiple sheet views — freeze electrical layers in the floor plan viewport, freeze floor plan layers in the electrical viewport.

### Viewport Properties

Key properties (accessible via Properties palette):
- **Standard Scale**: Preset scale or Custom
- **Custom Scale**: Numeric ratio
- **Display Locked**: Yes/No (lock to prevent accidental zoom)
- **View Center**: X,Y in model space coordinates
- **View Height/Width**: Visible area in model space units

### Clipping Viewports

Non-rectangular viewports using clipping boundaries:

```
VPCLIP
select-viewport
select-polyline  ; Closed polyline as new boundary
```

### Layout Setup for Sheets

Typical ADU plan set layout:
1. Set paper size: `PAGESETUP` → 24"×36" or 11"×17"
2. Insert title block on layer BORDER/TITLEBLOCK
3. Create viewport(s) sized to fit available area
4. Set each viewport's scale and center
5. Lock viewports
6. Freeze/thaw layers per viewport as needed

## Common Issues

| Problem | Cause | Fix |
|---------|-------|-----|
| Viewport shows nothing | View center way off | Double-click in VP, `ZOOM E`, set correct scale |
| Wrong scale after editing | Viewport unlocked | Lock viewport after setting scale |
| Dimensions wrong size | Wrong DIMSCALE or annotative mismatch | Match DIMSCALE to viewport scale factor |
| Layers showing in wrong VP | VP layer freeze not set | Use VPLAYER to freeze per-viewport |
| Can't select VP | Layer locked or VP on locked layer | Check viewport's layer, unlock |

## LISP for Viewport Management

```lisp
; Get all viewports in current layout
(defun get-layout-viewports (/ ss i ent)
  (setq ss (ssget "X" '((0 . "VIEWPORT"))))
  (if ss
    (repeat (setq i (sslength ss))
      (setq i (1- i))
      (setq ent (vlax-ename->vla-object (ssname ss i)))
      ; Process each viewport
    )))

; Set viewport scale
(defun set-vp-scale (vp-ent scale-factor)
  (vla-put-CustomScale vp-ent (/ 1.0 scale-factor)))
```
