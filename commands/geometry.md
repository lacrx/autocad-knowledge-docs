# Geometry Commands

## Line
```bash
# MCP
mcp__autocad__create_line(start_point="x1,y1", end_point="x2,y2", layer="LAYER")

# Direct — multi-segment, press Enter to end
{"command":"paste","text":"_.LINE x1,y1 x2,y2 x3,y3 "}
```

## Polyline (open)
```bash
# MCP — semicolon-delimited points
mcp__autocad__create_polyline(points_str="x1,y1;x2,y2;x3,y3", layer="LAYER")

# Direct
{"command":"paste","text":"_.PLINE x1,y1 x2,y2 x3,y3 "}
```

## Polyline (closed)
```bash
# MCP
mcp__autocad__create_polyline(points_str="x1,y1;x2,y2;x3,y3;x4,y4", closed="1", layer="LAYER")

# Direct — C closes the polyline
{"command":"paste","text":"_.PLINE x1,y1 x2,y2 x3,y3 x4,y4 C"}
```

## Rectangle
```bash
# MCP — two corner points
mcp__autocad__create_rectangle(point1="x1,y1", point2="x2,y2", layer="LAYER")

# Direct
{"command":"paste","text":"_.RECTANG x1,y1 x2,y2"}
```

## Circle
```bash
# MCP — center and radius
mcp__autocad__create_circle(center="x,y", radius="r", layer="LAYER")

# Direct
{"command":"paste","text":"_.CIRCLE x,y r"}
```

## Arc (3-point)
```bash
# MCP
mcp__autocad__create_arc(center="cx,cy", radius="r", start_angle="0", end_angle="90", layer="LAYER")

# Direct — start, second, end
{"command":"paste","text":"_.ARC x1,y1 x2,y2 x3,y3"}
```

## Ellipse
```bash
# MCP
mcp__autocad__create_ellipse(center="cx,cy", major_axis_endpoint="cx+a,cy", minor_axis_ratio="0.5", layer="LAYER")

# Direct
{"command":"paste","text":"_.ELLIPSE C cx,cy cx+a,cy b"}
```

## Construction Line (infinite)
```bash
{"command":"paste","text":"_.XLINE H y"}     # horizontal through y
{"command":"paste","text":"_.XLINE V x"}     # vertical through x
```

## Point
```bash
{"command":"paste","text":"_.POINT x,y"}
```

## Coordinate systems
- Absolute: `x,y` (from origin 0,0)
- Relative: `@dx,dy` (from last point)
- Polar: `@dist<angle` (distance and angle from last point)
- Example: `_.LINE 0,0 @144<0 @96<90 @144<180 C` draws a 12'×8' rectangle
