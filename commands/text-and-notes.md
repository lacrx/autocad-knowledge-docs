# Text and Notes Commands

## Single-line text (TEXT)
```bash
mcp__autocad__create_text(
  position="x,y", text="KITCHEN", height="2.25",
  rotation="0", layer="FP-TEXT")

# Direct
{"command":"paste","text":"_.TEXT _Justify _MC x,y 2.25 0 KITCHEN"}
# _MC = middle center justified
```

## Multi-line text (MTEXT)
```bash
mcp__autocad__create_mtext(
  position="x,y", text="Line 1\\PLine 2\\PLine 3",
  height="1.125", width="36", layer="FP-NOTE")
# \\P = paragraph break in MTEXT

# Direct
{"command":"paste","text":"_.MTEXT x1,y1 x2,y2"}
# Then type/paste text content
```

## Room labels
```bash
# Room name (large) + dimensions (small) + ceiling height
# Example: KITCHEN centered in room
{"command":"paste","text":"(progn (command \"_.TEXT\" \"_Justify\" \"_MC\" \"192,132\" \"3\" \"0\" \"KITCHEN\") (command \"_.TEXT\" \"_Justify\" \"_MC\" \"192,128\" \"1.5\" \"0\" \"(6'-5\\\" x 12'-0 1/2\\\")\") (command \"_.TEXT\" \"_Justify\" \"_MC\" \"192,125\" \"1.5\" \"0\" \"8'-0\\\" CEILING\"))"}
```

## General notes (numbered list)
```bash
# Place as MTEXT with width constraint
{"command":"paste","text":"_.MTEXT x1,y1 _Width 300 _Height 1.125"}
# Then paste the note text
```

## Text heights by use

| Purpose | Height (model) | Scale 1/4"=1' | Paper size |
|---------|---------------|----------------|------------|
| Title/sheet name | 6.0" | 1/8" | 0.125" |
| Room labels | 3.0" | 1/16" | 0.0625" |
| Dimension text | 1.5" | 1/32" | ~0.03" |
| Body notes | 1.5" | 1/32" | ~0.03" |
| Small annotations | 1.125" | 3/128" | ~0.023" |

## Text justification options
- `_Left` — left-justified (default)
- `_Center` — centered horizontally
- `_Right` — right-justified
- `_MC` — middle center (room labels)
- `_TL` — top left (notes blocks)
- `_BC` — bottom center (under dimension marks)

## Text styles
```bash
# Create/set style
{"command":"paste","text":"(command \"_.STYLE\" \"Standard\" \"simplex.shx\" \"0\" \"1\" \"0\" \"N\" \"N\")"}
{"command":"paste","text":"(command \"_.STYLE\" \"Notes\" \"romans.shx\" \"1.5\" \"1\" \"0\" \"N\" \"N\")"}
```
