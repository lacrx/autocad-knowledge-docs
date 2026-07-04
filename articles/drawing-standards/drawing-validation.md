# Drawing Validation for AI-Generated CAD Work

## Why Validation is Critical

Research confirms that LLMs are not inherently precise about coordinates. The error-feedback loop is the proven pattern:
1. Generate geometry via LISP/commands
2. Execute in AutoCAD
3. Validate result (dimensions, snapping, layers)
4. If errors found, correct and re-validate
5. Repeat until clean

## Validation Checks

### 1. Snap Precision

Every endpoint must land on an exact coordinate. No floating points from rounding errors.

```lisp
; Check if two points are within snap tolerance
(defun points-match (p1 p2 tol)
  (< (distance p1 p2) tol))

; Verify all LINE endpoints on a layer snap to 1/16" grid
(defun validate-snap (layer-name / ss i ent data p1 p2 errors)
  (setq errors 0)
  (setq ss (ssget "X" (list (cons 0 "LINE") (cons 8 layer-name))))
  (if ss
    (repeat (setq i (sslength ss))
      (setq i (1- i))
      (setq ent (ssname ss i))
      (setq data (entget ent))
      (setq p1 (cdr (assoc 10 data)))
      (setq p2 (cdr (assoc 11 data)))
      (if (or (> (rem (car p1) 0.0625) 0.001)
              (> (rem (cadr p1) 0.0625) 0.001)
              (> (rem (car p2) 0.0625) 0.001)
              (> (rem (cadr p2) 0.0625) 0.001))
        (progn
          (setq errors (1+ errors))
          (princ (strcat "\nSnap error: " (rtos (car p1) 2 4) "," (rtos (cadr p1) 2 4)))))))
  (princ (strcat "\n" (itoa errors) " snap errors found"))
  errors)
```

### 2. Dimension-vs-Geometry Consistency

Dimension strings must match the actual distance between their definition points.

```lisp
; Compare dimension text to actual measured distance
(defun validate-dimensions (/ ss i ent data defpt1 defpt2 actual-dist text-dist diff)
  (setq ss (ssget "X" '((0 . "DIMENSION"))))
  (if ss
    (repeat (setq i (sslength ss))
      (setq i (1- i))
      (setq ent (ssname ss i))
      (setq data (entget ent))
      (setq defpt1 (cdr (assoc 13 data)))  ; First definition point
      (setq defpt2 (cdr (assoc 14 data)))  ; Second definition point
      (setq actual-dist (distance defpt1 defpt2))
      ; Compare with displayed text
      (setq text-val (cdr (assoc 1 data)))
      (if (and text-val (/= text-val ""))
        (princ (strcat "\nDIM: text=" text-val " actual=" (rtos actual-dist 4 4)))))))
```

### 3. Layer Audit

Verify entities are on correct layers.

```lisp
; Check for entities on wrong layers
(defun audit-layers (/ ss i ent layer etype misplaced)
  (setq misplaced 0)
  (setq ss (ssget "X"))
  (if ss
    (repeat (setq i (sslength ss))
      (setq i (1- i))
      (setq ent (ssname ss i))
      (setq layer (cdr (assoc 8 (entget ent))))
      (setq etype (cdr (assoc 0 (entget ent))))
      ; Flag electrical entities not on ELEC- layers
      (if (and (member etype '("CIRCLE" "INSERT"))
               (not (vl-string-search "ELEC" layer))
               (not (vl-string-search "lec" layer)))
        ; Might be an outlet/switch on wrong layer
        nil)
      ; Flag dimensions not on -DIM layers
      (if (and (= etype "DIMENSION")
               (not (vl-string-search "Dim" layer))
               (not (vl-string-search "DIM" layer)))
        (progn
          (setq misplaced (1+ misplaced))
          (princ (strcat "\nMisplaced dim on layer: " layer)))))))
```

### 4. Endpoint Connectivity

Walls and lines that should connect must share exact endpoints (no gaps).

```lisp
; Find unconnected endpoints on wall layers
(defun find-gaps (layer-name tol / ss i ent data endpoints p matched)
  (setq endpoints '())
  (setq ss (ssget "X" (list (cons 8 layer-name))))
  (if ss
    (repeat (setq i (sslength ss))
      (setq i (1- i))
      (setq ent (ssname ss i))
      (setq data (entget ent))
      (cond
        ((= (cdr (assoc 0 data)) "LINE")
         (setq endpoints (cons (cdr (assoc 10 data)) endpoints))
         (setq endpoints (cons (cdr (assoc 11 data)) endpoints)))
        ((= (cdr (assoc 0 data)) "LWPOLYLINE")
         ; Get polyline vertices
         (foreach pair data
           (if (= (car pair) 10)
             (setq endpoints (cons (cdr pair) endpoints))))))))
  ; Check each endpoint has at least one other endpoint within tolerance
  (foreach p endpoints
    (setq matched nil)
    (foreach q endpoints
      (if (and (not (equal p q))
               (< (distance (list (car p) (cadr p)) (list (car q) (cadr q))) tol))
        (setq matched T)))
    (if (not matched)
      (princ (strcat "\nOrphan endpoint: " (rtos (car p) 2 4) "," (rtos (cadr p) 2 4))))))
```

### 5. Hatch Integrity

Verify hatches have proper boundaries and correct patterns.

## Validation Workflow

After every drawing modification:

1. **Run snap check** on modified layers
2. **Run dimension check** if dimensions were added/moved
3. **Run layer audit** to catch misplaced entities
4. **Run gap check** on wall layers
5. **Visual verify** — zoom to modified area and check appearance

## Integration with MCP Bridge

The validation LISP scripts should be:
1. Stored in `lisp/validation/` on the VM
2. Executed via `mcp__autocad__execute-lisp` after each modification batch
3. Results parsed by the MCP server and returned to Claude
4. Claude decides whether to fix issues or flag for user review

## Key Insight from Research

> "Pre-built LISP primitives beat LLM-generated LISP. Use the LLM to compose calls to your curated LISP library rather than generating raw LISP from scratch."

The validation library should be a curated, tested set of LISP functions — not generated on the fly.
