# Topic Index

## AutoCAD LT
- [Command Line Automation](articles/autocad-lt/command-line-automation.md) — Commands, syntax, selection methods, coordinate input
- [Paper Space and Viewports](articles/autocad-lt/paper-space-viewports.md) — Layouts, viewport scales, layer freezing, sheet setup
- [Layers and Organization](articles/autocad-lt/layers-and-organization.md) — Naming conventions, colors, lineweights, layer states

## Drawing Standards
- [Model Space Organization](articles/drawing-standards/model-space-organization.md) — Spatial layout, regions, viewport mapping, professional plan set structure
- [Wall Construction](articles/drawing-standards/wall-construction.md) — Polylines, hatching, corner connections, precision
- [Drawing Validation](articles/drawing-standards/drawing-validation.md) — Snap checks, dimension consistency, layer audits, error-feedback loops

## MCP Bridge
- [MCP Bridge Architecture](articles/mcp-bridge/mcp-bridge-architecture.md) — Protocol, components, PostMessage, file IPC, VM management
- [Troubleshooting](articles/mcp-bridge/troubleshooting.md) — Common issues, diagnostics, recovery sequence, autocomplete workaround
- [CAD MCP Ecosystem](articles/mcp-bridge/cad-mcp-ecosystem.md) — Landscape of 15+ CAD MCP projects across Revit, Rhino, FreeCAD, AutoCAD. Architecture camps, key people, commercial offerings.
- [Design Patterns](articles/mcp-bridge/design-patterns.md) — Escape hatches, composable tools, transport comparison, session persistence, safety tiers, QA tools. Patterns proven across multiple independent implementations.
- [Improvement Ideas](articles/mcp-bridge/improvement-ideas.md) — Actionable improvements for our bridge: audit trail, server-side validation, snapshot diff, tool consolidation, mock mode, safety tiers. Prioritized by effort vs impact.

## LISP Automation
- [LISP Fundamentals](articles/lisp-automation/lisp-fundamentals.md) — Syntax, commands, entities, selection sets, file I/O

## Drawing Commands
Reusable command recipes organized by task — ready to adapt and execute.
- [Geometry](commands/geometry.md) — Lines, polylines, rectangles, arcs, circles, coordinate systems
- [Walls](commands/walls.md) — Wall segments, corners, openings, thickness reference, LISP helper
- [Dimensions](commands/dimensions.md) — Linear, aligned, angular, continuous, baseline, leaders
- [Text and Notes](commands/text-and-notes.md) — TEXT, MTEXT, room labels, general notes, text heights
- [Blocks](commands/blocks.md) — Insert, attributes, doors, windows, fixtures, symbols
- [Hatching](commands/hatching.md) — Solid fills, pattern hatches, architectural patterns
- [Layers](commands/layers.md) — Create, set, freeze, standard layer scheme by discipline
- [Viewports](commands/viewports.md) — Paper space viewports, scales, layer presets by sheet
- [Annotation](commands/annotation.md) — Section marks, elevation marks, detail callouts, tags
- [Modification](commands/modification.md) — Move, copy, mirror, offset, trim, extend, array
- [Selection](commands/selection.md) — Selection sets, filters, entity queries, iteration

## Skills
- [Start AutoCAD Bridge](skills/start-autocad-bridge.md) — VM startup, agent verification, dispatcher loading via SCRIPT
- [Send AutoCAD Command](skills/send-autocad-command.md) — MCP tools, direct TCP, LISP execution, SCRIPT method
- [Recover Bridge](skills/recover-bridge.md) — Full recovery from any broken state: screenshot → dismiss → clean → reload
- [Take Screenshot](skills/take-screenshot.md) — Capture VM screen for diagnostics (GDI-based, no PIL)
- [Map Model Space Regions](skills/map-model-space-regions.md) — Viewport analysis, spatial mapping
- [Setup Standard Layers](skills/setup-standard-layers.md) — Layer, text, and dimension style setup
