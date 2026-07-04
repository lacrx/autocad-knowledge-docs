# Layer Organization for Residential Plans

## Why Layers Matter

Layers control visibility, color, linetype, lineweight, and printability. For a multi-sheet plan set, the same model space geometry serves different sheets by freezing/thawing layers per viewport.

## Naming Convention

Use a prefix scheme that groups layers by discipline:

| Prefix | Discipline | Example |
|--------|-----------|---------|
| `FP-` | Floor Plan | FP-WALLS, FP-DOORS, FP-DIM |
| `ELEC-` | Electrical | ELEC-OUTLET, ELEC-SWITCH |
| `PLMB-` | Plumbing | PLMB-FIXTURE, PLMB-PIPE |
| `MECH-` | Mechanical | MECH-EQUIP, MECH-DUCT |
| `ELEV-` | Elevations | ELEV-FRONT, ELEV-DIM |
| `SECT-` | Sections | SECT-CUT, SECT-BEYOND |
| `PLOT-` | Plot/Site Plan | PLOT-PROPERTY, PLOT-SETBACK |
| `STRUCT-` | Structural | STRUCT-SHEAR, STRUCT-FNDTN |

## Color Standards

ACI (AutoCAD Color Index) assignments:
- **1 (Red)**: Structural elements, property lines, electrical
- **2 (Yellow)**: Dimensions
- **3 (Green)**: Doors, framing
- **4 (Cyan)**: Windows, utilities
- **5 (Blue)**: Plumbing
- **6 (Magenta)**: Mechanical, fixtures
- **7 (White)**: Walls, text, general
- **8 (Gray)**: Interior partitions, beyond-cut elements
- **252 (Light Gray)**: Hatching

## Lineweight Standards

| Weight (mm) | Use |
|-------------|-----|
| 0.50 | Cut walls, borders, property lines, section cuts |
| 0.35 | Interior walls, elevation outlines, structural |
| 0.25 | Doors, windows, fixtures, framing |
| 0.18 | Dimensions, hidden lines, piping |
| 0.13 | Notes, hatching, fine print |

## Linetype Standards

| Linetype | Use |
|----------|-----|
| CONTINUOUS | Most elements |
| DASHED | Hidden lines, piping below, ductwork, setbacks, utilities |
| CENTER | Centerlines |
| PHANTOM | Property beyond scope |

## Layer States

Save and restore layer visibility presets for different views:

```
LAYERSTATE
S           ; Save
"FloorPlan" ; Name
```

Useful presets:
- **FloorPlan**: FP-* on, ELEC/PLMB/MECH off
- **Electrical**: FP-WALLS + FP-DOORS on, ELEC-* on, rest off
- **Plumbing**: FP-WALLS + FP-DOORS on, PLMB-* on, rest off
- **Structural**: FP-WALLS on, STRUCT-* on, rest off

## Creating Layers via LISP

```lisp
(defun create-layer (name color ltype lweight / )
  (if (not (tblsearch "LAYER" name))
    (command "_.LAYER"
      "_New" name
      "_Color" (itoa color) name
      "_LType" ltype name
      "_LWeight" (itoa lweight) name
      "")))
```

## Common Mistakes

1. **Drawing on layer 0**: Always set current layer before drawing
2. **Too few layers**: Can't control visibility per viewport later
3. **Too many layers**: Hard to manage. ~30-40 is right for residential
4. **Inconsistent naming**: Makes layer state management painful
5. **Wrong lineweights**: Walls look thin, dims look thick — unprofessional
6. **Not setting linetype scale**: Dashes too long/short at print scale
