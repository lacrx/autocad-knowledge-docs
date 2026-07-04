# Model Space Organization for Residential ADU Plan Sets

## Overview

A professional plan set uses a single model space to hold ALL drawing content, organized spatially into distinct regions. Paper space layouts (sheets) use viewports to look at specific regions at specific scales. This is the fundamental architecture that the AI agent must understand.

## Spatial Layout

All content in model space is drawn at full scale (1 unit = 1 inch). Different types of content are placed in separate spatial regions, far enough apart to avoid overlap when viewports are set up.

### Reference Layout (from County-Approved PDS671 Plan Set)

```
Y
^
|
|  (2920,2080) Detail Callouts — enlarged wall sections, connection details
|
|  (1680-2520, 1690-1710) Roof Framing Details — truss details, ridge connections  
|
|  (1550-2050, 600-970) Schedules — window schedule, door schedule, notes tables
|
|  (170-280, 50-90) BUILDING PLAN — floor plan, roof plan, foundation outline
|  Floor Plan: (118,-65) to (346,208)
|  Foundation: same region, different layers
|  Roof Plan: (116,-77) to (347,220) 
|  Electrical: extends to (2460,654) for schedules
|  Plumbing: within building footprint
|
|  (130-140, -945) Elevations Front & Back — below building plan
|  (1280-1315, -935) Elevations Right & Left — offset right
|
|  (2333,-2492) to (5422,-1540) Plot Plan — far away, drawn at survey scale
|
|  (3240-3500, 950-1015) Title Block Template — master title block content
|
+-----------------------------------------------------------------> X
```

### Key Principles

1. **Building plan is the origin area** — floor plan, foundation, roof plan, electrical, plumbing all share the same spatial region but on different layers. Viewport layer freezing shows the right content per sheet.

2. **Elevations below the plan** — Front/back elevations at ~(135, -945), right/left at ~(1290, -935). Spaced apart so viewports don't overlap.

3. **Details far from building** — Enlarged details placed in distant regions (Y~2000+). Each detail cluster is a group of lines/hatches that a viewport shows at a larger scale (3/8" or 1/2" = 1'-0").

4. **Schedules in their own region** — Tables and schedules at ~(1550-2050, 600-970). Viewports on relevant sheets point here.

5. **Title block as a model-space block** — Common title block content drawn once, viewported onto every sheet.

6. **Plot plan distant** — Plot/site plan drawn at actual survey coordinates, far from the building plan. Viewported at 1"=20' or similar small scale.

## How Viewports Reference Regions

Each paper space layout (sheet) has viewports that:
- Point to specific model space coordinates (view_center)
- Display at a specific scale (view_height determines zoom)
- Freeze/thaw layers to show only relevant content

Example from PDS671:
```
A1 - Floor Plan:
  Main VP: view_center=(174.8, 87.6), view_height=395.3 → shows building at ~1/4"=1'-0"
  Schedule VP: view_center=(1576.1, 605.8), view_height=124.6 → shows window schedule
  Detail VP: view_center=(2922.7, 2082.7), view_height=2.3 → shows enlarged detail

A3 - Elevations:
  Front VP: view_center=(136.4, -944.4), view_height=243.0 → shows front elevation
  Back VP: view_center=(135.7, -946.0), view_height=243.0 → shows back elevation
```

## Wall Construction in Model Space

Professional drawings use:
- **LWPOLYLINE** for wall outlines (closed polylines for wall cavities)
- **HATCH** to fill wall cavities (SOLID pattern for cut sections)
- Lines for individual elements (studs, blocking)
- Circles for fasteners, outlets

From PDS671's Floor Plan layer:
- 32 LWPOLYLINE (wall outlines and room boundaries)
- 10 HATCH (wall fills and floor patterns)
- 8 LINE (detail lines)
- 3 MTEXT (labels)
- 1 CIRCLE

**Wrong approach**: Drawing walls as separate rectangles. Each wall segment needs its own cavity fill.

**Right approach**: Draw parallel LWPOLYLINE outlines defining the wall cavity, then HATCH between them with SOLID pattern. This creates proper cut-section appearance.

## Hatch Patterns Used

| Pattern | Use | Layer |
|---------|-----|-------|
| AR-CONC | Concrete in section | Elevation sections |
| EARTH | Earth/soil in section | Elevation sections |
| AR-RSHKE | Roof shingles | Elevation dim/labels |
| SOLID | Wall cavities, filled areas | Floor Plan, Foundation |
| ANSI31 | Cross-hatching in details | Title block, details |

## Layer-to-Sheet Mapping

The same model space content serves multiple sheets through viewport layer freezing:

| Sheet | Visible Layers | Frozen Layers |
|-------|----------------|---------------|
| A1 Floor Plan | Floor Plan, F.P.-*, Furniture | Electrical, Plumbing, Foundation |
| A2 Electrical | Floor Plan (walls only), Electrical-* | F.P. dims, Plumbing, Foundation |
| S1 Foundation | Foundation-*, Foundation Outline | Floor Plan, Electrical, Roof |
| A3-A4 Elevations | Elevation-* for relevant faces | Floor Plan, Foundation, Roof |
| S2 Roof Framing | Roof Plan, Trusses, Roof Framing-* | Floor Plan, Foundation, Electrical |

## Implications for AI Agent

1. **Never place content randomly** — every element belongs in a specific spatial region
2. **Understand viewport relationships** — before modifying content, check which viewports reference that area
3. **Use layer discipline** — wrong layer = content appears on wrong sheet
4. **Draw walls properly** — LWPOLYLINE outlines + HATCH fill, not rectangles
5. **Coordinate between sheets** — a change to the building plan affects floor plan, electrical, plumbing, and foundation sheets simultaneously (they all viewport the same model space region)
