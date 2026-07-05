# Layer Commands

## Create layer
```bash
mcp__autocad__layer_create(name="FP-WALL", color="7", lineweight="50")

# Direct
{"command":"paste","text":"_.LAYER _New FP-WALL _Color 7 FP-WALL _LWeight 0.50 FP-WALL "}
```

## Set current layer
```bash
mcp__autocad__layer_set_current(name="FP-WALL")

{"command":"paste","text":"_.LAYER _Set FP-WALL "}
```

## List layers
```bash
mcp__autocad__layer_list()
```

## Freeze / Thaw
```bash
mcp__autocad__layer_freeze(name="ELEC-OUTLET")
mcp__autocad__layer_thaw(name="ELEC-OUTLET")
```

## Lock / Unlock
```bash
mcp__autocad__layer_lock(name="FP-DIM")
mcp__autocad__layer_unlock(name="FP-DIM")
```

## Set properties
```bash
mcp__autocad__layer_set_properties(
  name="FP-WALL", color="7", linetype="Continuous", lineweight="50")
```

## Standard layer scheme

### Floor Plan
| Layer | Color | LW (mm) | Purpose |
|-------|-------|---------|---------|
| FP-WALL | 7 (white) | 0.50 | Walls (cut) |
| FP-WALL-BYD | 8 (gray) | 0.25 | Walls beyond |
| FP-DOOR | 3 (green) | 0.25 | Doors and swings |
| FP-WINDOW | 4 (cyan) | 0.25 | Windows |
| FP-DIM | 2 (yellow) | 0.18 | Dimensions |
| FP-TEXT | 7 | 0.18 | Room labels, notes |
| FP-NOTE | 7 | 0.18 | General notes |
| FP-FIXT | 6 (magenta) | 0.25 | Fixtures (plumbing) |
| FP-APPL | 6 | 0.25 | Appliances |
| FP-CASE | 5 (blue) | 0.25 | Casework/cabinets |
| FP-STRS | 1 (red) | 0.35 | Structural |
| FP-HATCH | 8 | 0.00 | Wall hatch fill |

### Electrical
| Layer | Color | LW | Purpose |
|-------|-------|-----|---------|
| ELEC-OUTLET | 3 (green) | 0.25 | Receptacles |
| ELEC-SWITCH | 4 (cyan) | 0.25 | Switches |
| ELEC-LIGHT | 2 (yellow) | 0.25 | Lighting |
| ELEC-WIRE | 1 (red) | 0.18 | Circuits |
| ELEC-PANEL | 6 (magenta) | 0.35 | Panel schedule |

### Plumbing
| Layer | Color | LW | Purpose |
|-------|-------|-----|---------|
| PLMB-FIXT | 4 | 0.25 | Fixtures |
| PLMB-SUPPLY | 1 | 0.18 | Supply lines |
| PLMB-WASTE | 3 | 0.25 | Waste/vent lines |

### Mechanical
| Layer | Color | LW | Purpose |
|-------|-------|-----|---------|
| MECH-EQUIP | 5 | 0.35 | Equipment |
| MECH-DUCT | 4 | 0.25 | Ductwork |

### Elevations / Sections
| Layer | Color | LW | Purpose |
|-------|-------|-----|---------|
| ELEV-OUTLINE | 7 | 0.50 | Building outline |
| ELEV-DETAIL | 7 | 0.25 | Detail lines |
| ELEV-DIM | 2 | 0.18 | Elevation dims |
| ELEV-NOTE | 7 | 0.18 | Callouts |
| SECT-CUT | 1 | 0.50 | Section cuts |
| SECT-BYD | 8 | 0.25 | Beyond section cut |

### Sheet/Plot
| Layer | Color | LW | Purpose |
|-------|-------|-----|---------|
| PLOT-BORDER | 7 | 0.50 | Sheet border |
| PLOT-TITLE | 7 | 0.35 | Title block |
| PLOT-VP | 8 | 0.00 | Viewport frames (no print) |

## Bulk setup via LISP
```bash
# Load the layer setup script from the curated library
{"command":"paste","text":"(load \"C:/autocad_mcp/lisp/layers/setup-standard-layers.lsp\")"}
{"command":"paste","text":"(c:setup-standard-layers)"}
```
