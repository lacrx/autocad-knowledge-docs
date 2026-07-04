# AutoCAD LT Command Line Automation

## Why Command Line

AutoCAD LT has no COM API, no .NET API, no scripting host beyond LISP. All automation goes through the command line — either typed by a user or injected via PostMessage/SendKeys.

Every AutoCAD operation has a command-line equivalent. Learning the command syntax is essential for automation.

## Command Syntax

Commands follow a pattern: command name, then a sequence of prompts answered with values.

```
LINE 0,0 12,12           ; Line from origin to (12,12), press Enter to end
CIRCLE 6,6 3              ; Circle at (6,6), radius 3
RECTANG 0,0 10,5           ; Rectangle from (0,0) to (10,5)
```

Prefix with `_.` for language-independent, no-shortcut form:
```
_.LINE 0,0 12,12
```

## Common Commands for ADU Plans

### Drawing
| Command | Syntax | Use |
|---------|--------|-----|
| `LINE` | `LINE x1,y1 x2,y2 [Enter]` | Wall lines, detail lines |
| `PLINE` | `PLINE x1,y1 x2,y2 x3,y3 C` | Closed polylines for rooms, boundaries |
| `RECTANG` | `RECTANG x1,y1 x2,y2` | Quick rectangles |
| `OFFSET` | `OFFSET dist select-entity side` | Wall thickness, setback lines |
| `TRIM` | `TRIM [Enter] select-to-trim` | Clean intersections |
| `EXTEND` | `EXTEND [Enter] select-to-extend` | Extend to boundaries |
| `FILLET` | `FILLET R 0 select1 select2` | Corner cleanup (R 0 = sharp corner) |

### Modification
| Command | Syntax | Use |
|---------|--------|-----|
| `MOVE` | `MOVE select [Enter] base-pt new-pt` | Reposition elements |
| `COPY` | `COPY select [Enter] base-pt new-pt` | Duplicate elements |
| `ROTATE` | `ROTATE select [Enter] base-pt angle` | Rotate elements |
| `MIRROR` | `MIRROR select [Enter] pt1 pt2 N` | Mirror (N = keep original) |
| `SCALE` | `SCALE select [Enter] base-pt factor` | Scale elements |
| `ERASE` | `ERASE select [Enter]` | Delete elements |

### Dimensions
| Command | Syntax | Use |
|---------|--------|-----|
| `DIMLINEAR` | `DIMLINEAR pt1 pt2 offset-pt` | Horizontal/vertical dims |
| `DIMALIGNED` | `DIMALIGNED pt1 pt2 offset-pt` | Aligned to geometry |
| `DIMCONTINUE` | `DIMCONTINUE pt` | Chain dimensions |
| `DIMBASELINE` | `DIMBASELINE pt` | Baseline dimensions |
| `LEADER` | `LEADER pt1 pt2 [Enter] text [Enter]` | Callout leaders |

### Layers
| Command | Syntax | Use |
|---------|--------|-----|
| `LAYER` | `LAYER` (opens dialog) | Layer management |
| `-LAYER` | `-LAYER N name C color name [Enter]` | Command-line layer ops |
| `CLAYER` | Set via `CLAYER` system variable | Current layer |

### View
| Command | Syntax | Use |
|---------|--------|-----|
| `ZOOM` | `ZOOM E` (extents) / `ZOOM W x1,y1 x2,y2` | Navigate |
| `PAN` | `PAN` then drag | Pan view |
| `MSPACE` | `MSPACE` | Enter model space in viewport |
| `PSPACE` | `PSPACE` | Return to paper space |

## System Variables

Key variables for drawing setup:

| Variable | Value | Purpose |
|----------|-------|---------|
| `LUNITS` | 4 | Architectural units (feet-inches) |
| `LUPREC` | 5 | Display precision (1/32") |
| `INSUNITS` | 1 | Insert units = inches |
| `LTSCALE` | 48 | Linetype scale for 1/4"=1'-0" |
| `DIMSCALE` | 48 | Dimension scale for 1/4"=1'-0" |
| `TEXTSIZE` | 4.5 | Default text height (3/32" at 1/4"=1'-0") |

## Selection Methods

When a command prompts for object selection:
- Click individual objects
- `W` — Window select (fully enclosed)
- `C` — Crossing select (touched by window)
- `L` — Last created object
- `P` — Previous selection set
- `ALL` — Everything visible
- `F` — Fence (polyline crossing)

## Coordinate Input

| Format | Example | Meaning |
|--------|---------|---------|
| Absolute | `12,6` | Point at x=12, y=6 |
| Relative | `@12,6` | 12 right, 6 up from last point |
| Polar | `@12<45` | 12 units at 45 degrees from last point |
| Direct distance | `12` (with ortho on) | 12 units in cursor direction |

## Tips for MCP Automation

1. Always use absolute coordinates — relative depends on cursor state
2. Prefix commands with `_.` for reliability
3. End multi-point commands with empty string (Enter)
4. Use `(command ...)` in LISP for programmatic command invocation
5. Check `CMDACTIVE` system variable before sending commands
6. Cancel pending commands with ESC before starting new ones
