# Map Model Space Regions

## When to use
Before modifying a drawing, to understand where each type of content lives in model space.

## Steps

1. **Query viewport data from all layouts**
```python
# Get all layouts and their viewports
for layout in doc.layouts:
    vps = [e for e in layout if e.dxftype() == "VIEWPORT"]
    for vp in vps:
        print(f"{layout.name}: view_center={vp.dxf.view_center_point}, view_height={vp.dxf.view_height}")
```

2. **Cluster viewport centers into regions**
Group viewports by proximity of their view_center coordinates. Each cluster = one drawing region.

3. **Map layers to regions**
```python
# Get bounding box per layer
for layer in doc.layers:
    entities = [e for e in msp if e.dxf.layer == layer.dxf.name]
    # Calculate bounds from entity geometry
```

4. **Identify region purposes**
- Building plan area: where floor plan, foundation, roof layers overlap
- Elevation areas: where elevation layers are concentrated
- Detail areas: small regions with enlarged content
- Schedule areas: regions with ACAD_TABLE and dense MTEXT

## Expected ADU Plan Set Layout

```
Region              | Approximate Coordinates    | Content
Building Plan       | (100-350, -80 to 220)     | Floor plan, foundation, roof
Front/Back Elev     | (100-350, -1050 to -850)  | Front and back elevations  
Right/Left Elev     | (1150-1460, -1030 to -850)| Right and left elevations
Foundation Details  | (-850 to -520, 70-120)    | Foundation connection details
Roof Details        | (1680-2520, 1690-1710)    | Truss and framing details
Schedules           | (1550-2050, 600-970)      | Tables, schedules, notes
Detail Callouts     | (2920-2970, 2078-2083)    | Enlarged wall/connection details
Title Block         | (3240-3500, 950-1015)     | Master title block
Plot Plan           | (2300-5500, -2500 to -1500)| Site/plot plan
```

## Output
A spatial map of the drawing's model space, documenting which regions contain which content and which viewports reference them.
