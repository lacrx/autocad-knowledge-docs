# Block Commands

## Insert block
```bash
mcp__autocad__block_insert(
  name="DOOR-36", insertion_point="180,60",
  scale="1", rotation="0", layer="FP-DOOR")

# Direct
{"command":"paste","text":"_.INSERT DOOR-36 180,60 1 1 0"}
```

## Insert with attributes
```bash
mcp__autocad__block_insert_with_attributes(
  name="ROOM-TAG",
  insertion_point="192,132",
  scale="1", rotation="0",
  attributes={"ROOM_NAME": "KITCHEN", "ROOM_NUM": "101"})
```

## List available blocks
```bash
mcp__autocad__block_list()
```

## Get block attributes
```bash
mcp__autocad__block_get_attributes(entity_id="handle")
```

## Update attribute value
```bash
mcp__autocad__block_update_attribute(
  entity_id="handle", tag="ROOM_NAME", value="LIVING ROOM")
```

## Common architectural blocks

### Doors (plan view)
```
DOOR-24  — 2'-0" door (closet)
DOOR-30  — 2'-6" door (bath)
DOOR-32  — 2'-8" door (bedroom)
DOOR-36  — 3'-0" door (standard interior)
DOOR-EXT — 3'-0" exterior w/ threshold
DOOR-SLD — sliding door (closet)
```

### Windows (plan view)
```
WIN-24   — 2'-0" window
WIN-36   — 3'-0" window
WIN-48   — 4'-0" window
WIN-60   — 5'-0" window
```

### Fixtures
```
TOILET   — standard toilet (plan)
SINK-KIT — kitchen sink (plan)
SINK-LAV — lavatory sink (plan)
TUB-60   — 5'-0" bathtub (plan)
SHOWER-36 — 3'-0" × 3'-0" shower (plan)
RANGE-30 — 30" range/stove (plan)
FRIDGE-36 — 36" refrigerator (plan)
DW-24    — 24" dishwasher (plan)
```

### Symbols
```
ELEC-OUTLET    — duplex receptacle
ELEC-SWITCH    — single pole switch
ELEC-LIGHT     — ceiling light
ELEC-GFI       — GFCI outlet
ELEC-SMOKE     — smoke detector
SECT-MARK      — section cut marker
ELEV-MARK      — elevation reference
DETAIL-BUBBLE  — detail callout
NORTH-ARROW    — north arrow
```

## Create block from entities
```bash
# Select entities, define base point, name
{"command":"paste","text":"_.BLOCK NEW-BLOCK 0,0"}
# Then select objects, press Enter
```

## Explode block
```bash
{"command":"paste","text":"_.EXPLODE _Last"}
```
