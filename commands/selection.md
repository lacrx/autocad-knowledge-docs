# Selection Commands

## Select all model space entities
```lisp
(ssget "X" '((410 . "Model")))
```

## Select by layer
```lisp
(ssget "X" '((8 . "FP-WALL") (410 . "Model")))
```

## Select by entity type
```lisp
; All lines
(ssget "X" '((0 . "LINE") (410 . "Model")))

; All polylines
(ssget "X" '((0 . "LWPOLYLINE") (410 . "Model")))

; All text
(ssget "X" '((0 . "TEXT") (410 . "Model")))

; All dimensions
(ssget "X" '((0 . "DIMENSION") (410 . "Model")))

; All blocks
(ssget "X" '((0 . "INSERT") (410 . "Model")))
```

## Select by window
```lisp
; Crossing window (touches or inside)
(ssget "C" '(x1 y1) '(x2 y2))

; Window (fully inside only)
(ssget "W" '(x1 y1) '(x2 y2))
```

## Select by block name
```lisp
(ssget "X" '((0 . "INSERT") (2 . "DOOR-36") (410 . "Model")))
```

## MCP entity queries
```bash
# Count entities
mcp__autocad__entity_count()

# List entities (with optional layer/type filter)
mcp__autocad__entity_list(layer="FP-WALL", entity_type="LWPOLYLINE")

# Get entity properties by handle
mcp__autocad__entity_get(entity_id="1A3F")
```

## Iterate selection set (LISP)
```lisp
(setq ss (ssget "X" '((0 . "TEXT") (8 . "FP-TEXT"))))
(setq i 0)
(while (< i (sslength ss))
  (setq ent (ssname ss i))
  (setq data (entget ent))
  (setq txt (cdr (assoc 1 data)))
  (princ (strcat "\n" txt))
  (setq i (1+ i))
)
```

## Selection set operations
```lisp
; Number of entities
(sslength ss)

; Get entity name by index
(ssname ss 0)

; Add entity to selection set
(ssadd ent ss)

; Remove entity from selection set
(ssdel ent ss)

; Check if entity is in set
(ssmemb ent ss)
```

## Filter by property value
```lisp
; Text containing "KITCHEN"
(ssget "X" '((0 . "TEXT") (1 . "*KITCHEN*")))

; Lines on layer FP-WALL, color 7
(ssget "X" '((0 . "LINE") (8 . "FP-WALL") (62 . 7)))

; Circles with radius > 6
; (requires LISP iteration, can't filter by radius in ssget)
```
