# AutoLISP Fundamentals for AutoCAD LT

## Overview

AutoLISP is the only programmatic interface available in AutoCAD LT. It runs inside AutoCAD's LISP interpreter and can invoke any command, query entities, perform calculations, and automate repetitive tasks.

## Loading Scripts

```
; From command line
(load "C:/path/to/script.lsp")

; Auto-load via acad.lsp or acaddoc.lsp (runs on every drawing open)
```

## Basic Syntax

```lisp
; Variables
(setq x 10)
(setq name "FP-WALLS")

; Math
(+ 1 2)          ; 3
(* 12 48)         ; 576
(/ 22.71875 12.0) ; 1.89322916...

; Strings
(strcat "Layer: " name)    ; "Layer: FP-WALLS"
(itoa 42)                  ; "42"
(atoi "42")                ; 42
(rtos 10.5 4 4)            ; "10-1/2\"" (architectural format)

; Lists
(list 0 0)                 ; Point: (0 0)
(car pt)                   ; X coordinate
(cadr pt)                  ; Y coordinate
(nth 2 pt)                 ; Z coordinate
```

## Invoking AutoCAD Commands

```lisp
; The (command) function sends input to the command line
(command "_.LINE" "0,0" "12,0" "")    ; Draw line, Enter to end
(command "_.CIRCLE" "6,6" "3")         ; Circle at 6,6 radius 3
(command "_.ZOOM" "_Extents")          ; Zoom extents

; Use pause to let user pick
(command "_.LINE" pause pause "")      ; User picks two points

; Multiple values
(command "_.MOVE" ss "" "0,0" "12,0")  ; Move selection set
```

## Entity Access

```lisp
; Get last created entity
(setq ent (entlast))

; Get entity data (DXF group codes)
(setq data (entget ent))
; Returns: ((-1 . <Entity name>) (0 . "LINE") (10 . (0.0 0.0 0.0)) (11 . (12.0 0.0 0.0)) ...)

; Extract specific data
(cdr (assoc 0 data))   ; Entity type: "LINE"
(cdr (assoc 10 data))  ; Start point: (0.0 0.0 0.0)
(cdr (assoc 11 data))  ; End point: (12.0 0.0 0.0)
(cdr (assoc 8 data))   ; Layer: "0"

; Modify entity
(setq data (subst (cons 8 "FP-WALLS") (assoc 8 data) data))
(entmod data)
(entupd ent)
```

## Selection Sets

```lisp
; Select all entities
(setq ss (ssget "X"))

; Select by type
(setq ss (ssget "X" '((0 . "LINE"))))

; Select by layer
(setq ss (ssget "X" '((8 . "FP-WALLS"))))

; Select by type AND layer
(setq ss (ssget "X" '((0 . "LINE") (8 . "FP-WALLS"))))

; Select in window
(setq ss (ssget "W" '(0 0) '(100 100)))

; Iterate selection set
(if ss
  (repeat (setq i (sslength ss))
    (setq i (1- i))
    (setq ent (ssname ss i))
    (setq data (entget ent))
    ; process...
  ))
```

## DXF Group Codes

| Code | Meaning |
|------|---------|
| 0 | Entity type (LINE, CIRCLE, TEXT, etc.) |
| 1 | Text string (for TEXT/MTEXT) |
| 2 | Block name, attribute tag |
| 5 | Handle (unique hex ID) |
| 8 | Layer name |
| 10 | Primary point (start, center, insert) |
| 11 | Secondary point (end, etc.) |
| 40 | Radius, height, scale |
| 41 | Width, X scale |
| 42 | Bulge factor |
| 50 | Angle |
| 62 | Color number |
| 67 | Paper/model space flag (0=model, 1=paper) |

## Functions for Geometry

```lisp
; Distance between two points
(distance '(0 0) '(12 12))  ; 16.9705...

; Angle between two points (radians)
(angle '(0 0) '(12 12))     ; 0.785398... (45°)

; Point at distance and angle from base
(polar '(0 0) (/ pi 4) 10)  ; Point 10 units at 45°

; Midpoint
(defun midpoint (p1 p2)
  (list (/ (+ (car p1) (car p2)) 2.0)
        (/ (+ (cadr p1) (cadr p2)) 2.0)))

; Convert degrees to radians
(defun dtr (d) (* pi (/ d 180.0)))
```

## Error Handling

```lisp
(defun c:my-command (/ *error* old-vars)
  ; Save state
  (setq old-vars (mapcar 'getvar '("CMDECHO" "OSMODE")))
  
  ; Error handler
  (defun *error* (msg)
    (mapcar 'setvar '("CMDECHO" "OSMODE") old-vars)
    (princ (strcat "\nError: " msg))
    (princ))
  
  ; Suppress command echo
  (setvar "CMDECHO" 0)
  
  ; ... do work ...
  
  ; Restore state
  (mapcar 'setvar '("CMDECHO" "OSMODE") old-vars)
  (princ))
```

## File I/O

```lisp
; Write JSON result (for MCP IPC)
(defun write-json-file (filepath data / f)
  (setq f (open filepath "w"))
  (if f
    (progn
      (write-line data f)
      (close f)
      T)
    nil))

; Read file
(defun read-file (filepath / f line result)
  (setq f (open filepath "r"))
  (if f
    (progn
      (while (setq line (read-line f))
        (setq result (if result (strcat result "\n" line) line)))
      (close f)
      result)
    nil))
```

## Tips

1. Use `(princ)` at end of defun to suppress return value echo
2. Declare local variables after `/` in argument list to avoid globals
3. `(command)` with empty string `""` sends Enter
4. Always wrap `(command)` sequences with `CMDECHO` set to 0
5. Test interactively: type LISP expressions directly at command line
6. Use `(alert "message")` for debugging (shows dialog — visible confirmation)
