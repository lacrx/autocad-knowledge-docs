# Viewport Commands

## Create viewport in paper space
```bash
# Switch to a layout tab first
{"command":"paste","text":"_.LAYOUT _Set \"A1 - Floor Plan (24x36)\""}

# Create viewport with corners
mcp__autocad__create_viewport(
  corner1="1.5,1.5", corner2="34.5,22.5",
  scale="48", center="192,132")

# Direct LISP
{"command":"paste","text":"(command \"_.MVIEW\" \"1.5,1.5\" \"34.5,22.5\")"}
```

## Set viewport scale
```bash
# Double-click into viewport first (or use MSPACE)
{"command":"paste","text":"_.MSPACE"}
# Then set zoom scale
{"command":"paste","text":"_.ZOOM _Scale 1/48XP"}
# 1/48XP = 1/4" = 1'-0" scale
```

## Common viewport scales

| Scale | XP Factor | Usage |
|-------|-----------|-------|
| 1/4" = 1'-0" | 1/48XP | Floor plans, elevations |
| 3/8" = 1'-0" | 1/32XP | Details, sections |
| 1/2" = 1'-0" | 1/24XP | Large details |
| 3/4" = 1'-0" | 1/16XP | Wall sections |
| 1" = 1'-0" | 1/12XP | Connection details |
| 1" = 20' | 1/240XP | Site plan |
| 1:1 | 1XP | Full size detail |

## Lock viewport scale
```bash
# After setting scale, lock to prevent accidental zoom
{"command":"paste","text":"(setvar \"VIEWPORTSCALELOCK\" 1)"}
```

## Freeze layer in viewport
```bash
# Inside viewport (MSPACE), freeze layers that shouldn't show on this sheet
{"command":"paste","text":"_.VPLAYER _Freeze ELEC-OUTLET _Current "}
{"command":"paste","text":"_.VPLAYER _Freeze PLMB-SUPPLY _Current "}
```

## Return to paper space
```bash
{"command":"paste","text":"_.PSPACE"}
```

## Viewport layer presets by sheet type

### Floor Plan
Freeze: ELEC-*, PLMB-*, MECH-*, SECT-*, ELEV-*

### Electrical Plan
Freeze: PLMB-*, MECH-*, SECT-*, ELEV-*, FP-DIM

### Plumbing Plan
Freeze: ELEC-*, MECH-*, SECT-*, ELEV-*, FP-DIM

### Elevations
Freeze: FP-*, ELEC-*, PLMB-*, MECH-*, SECT-*
Show: ELEV-*

### Sections
Freeze: FP-*, ELEC-*, PLMB-*, MECH-*, ELEV-*
Show: SECT-*

## Sheet sizes

| Size | Dimensions | Title block area |
|------|-----------|-----------------|
| 24×36 (D) | 36" × 24" | 34.5" × 22.5" (with 0.75" margins) |
| 18×24 (C) | 24" × 18" | 22.5" × 16.5" |
| 11×17 (B) | 17" × 11" | 15.5" × 9.5" |
