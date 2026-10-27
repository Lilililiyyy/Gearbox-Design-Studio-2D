# Gearbox Design Studio 2D

[中文说明](README.md) · **English documentation**

Drag mechanical components into a browser workspace, edit their dimensions, and arrange a two-dimensional gearbox layout.

Gearbox Design Studio 2D is an interactive layout tool built with React, TypeScript, and SVG. It supports concept discussions, teaching demonstrations, and preliminary component placement. The workspace uses a simplified plan-style schematic: shafts run along X, and gears appear as rectangles whose dimensions correspond to face width and pitch diameter.

![Running application with a two-shaft gear layout, component palette, and canvas toolbar](docs/images/01-workspace.png)

*Figure 1 · Actual application screenshot. The example contains a housing boundary, two stepped shafts, four bearings, two spur gears, two spacers, one coupling, and one circlip: 13 components in total. It demonstrates the interface and has not undergone engineering verification.*

## Contents

- [Quick start](#quick-start)
- [Features and components](#features-and-components)
- [Interface and basic controls](#interface-and-basic-controls)
- [Build a two-shaft example](#build-a-two-shaft-example)
- [Parameters, units, and snapping](#parameters-units-and-snapping)
- [Current scope and limitations](#current-scope-and-limitations)
- [Development](#development)
- [Troubleshooting](#troubleshooting)

## Quick start

### Requirements

- Node.js and npm. These instructions were verified with **Node.js 24.19.0**.
- A desktop browser supporting SVG, pointer events, and HTML drag-and-drop. The screenshots were captured in Chrome.
- Access to the npm registry during installation. The interface loads its styles from `https://cdn.tailwindcss.com` at runtime, so the current version needs network access for styling.

### Install and run

```bash
git clone https://github.com/Lyy0617/Gearbox-Design-Studio-2D.git
cd Gearbox-Design-Studio-2D
npm install
npm run dev
```

Open the URL printed in the terminal. The configured port is `3000`, usually `http://localhost:3000`. If that port is occupied, Vite tries subsequent ports; use the URL it actually prints.

The current 2D layout features **require neither an API key nor a backend service**. `vite.config.ts` retains Gemini environment-variable mappings, but the existing components do not call an AI API.

### Build and preview locally

```bash
npx tsc --noEmit
npm run build
npm run preview
```

`tsc` checks types, and the build writes its output to `dist/`. Use `preview` to inspect the production build locally, following the URL printed in the terminal. The page still needs access to the Tailwind CSS CDN.

## Features and components

Drag a component from the left-hand palette onto the canvas, then edit it in the floating properties panel on the right.

| Component | Editable properties | Display |
| --- | --- | --- |
| Stepped shaft | Segment lengths and diameters; add or remove segments | Segmented outline and centerline, with a total-length readout |
| Bearing | Type, width, outside diameter | Six different schematic bearing symbols |
| Spur gear | Tooth count, module, face width, color | Rectangle sized using pitch diameter |
| Helical gear | Tooth count, module, face width, helix angle, color | Rectangle with fixed diagonal markings |
| Bevel gear | Tooth count, module, face width, color, four orientation presets | Simplified tapered-outline markings |
| Worm | Length, diameter | Rounded rectangle with schematic thread lines |
| Coupling | Width / length, outside diameter | Outline with a central dividing line |
| Spacer | Width / length, outside diameter | Rectangular outline |
| Circlip | Width / length, outside diameter | Thin strip with end-hole markings; rendered strip width is fixed |
| Housing | Width, height | Dashed boundary rectangle |

Every component exposes X and Y coordinates. Bearing options are deep groove ball, angular contact ball, cylindrical roller, tapered roller, self-aligning, and thrust. These are schematic types, not a catalog of standardized bearing models.

## Interface and basic controls

| Area / action | How to use it |
| --- | --- |
| Component palette | Groups components into rotating parts, transmission parts, and accessories; drag a card to add a part |
| Canvas | Shows the grid, centerlines, and assembly layout |
| Header counter | Shows the current number of components |
| Properties panel | Click a component to open it; changes apply immediately; the top-right × closes the panel |
| Move a component | Drag the component; a selected component has a blue outline |
| Pan | Drag empty canvas space or use the mouse wheel; Shift-drag on empty space is also supported |
| Zoom | Use the zoom buttons or Ctrl / ⌘ + mouse wheel |
| Reset view | Click “重置视图” to restore the initial viewport and 100% zoom; components remain in the scene |
| Automatic snapping | Toggle the magnet button; snapping during dragging is enabled initially |
| Delete | Select a component and click “删除组件” at the bottom of its properties panel |

The current interface is mainly Chinese, with English component names in the palette. Providing two documentation languages does not add an in-app language switch.

![Gear properties showing coordinates, tooth count, module, face width, and pitch diameter](docs/images/02-gear-properties.png)

*Figure 2 · Selecting the larger gear reveals 50 teeth, module 4, face width 32, and the calculated pitch diameter of 200.0 mm.*

## Build a two-shaft example

Use the following values to reproduce the main layout in the screenshots. Drag each component in first, then enter exact coordinates and dimensions in its properties panel rather than relying on the drop position.

1. Add a housing centered at `X = -60`, `Y = 0`, with width `600` and height `440`.
2. Add two stepped shafts centered at `(-60, -100)` and `(-60, 40)`. Configure both with the three segments below, giving each a total length of `500 mm`.
3. Add two bearings on each shaft at X positions `-260` and `140`, using the corresponding shaft's Y coordinate. Set bearing width to `24` and outside diameter to `64`.
4. Add two spur gears using the gear table below. Their X coordinates match, and their Y coordinates match their respective shafts.
5. Add spacers at `(-50, -100)` and `(-50, 40)`, with width `12` and outside diameter `46`. Add a coupling at `(-340, -100)`, with width `60` and outside diameter `50`. Add a circlip at `(-68, 40)`, with outside diameter `42`.
6. To match the screenshots, change the upper-left bearing to angular contact, the lower-right bearing to tapered roller, and the larger gear's color to `#0d9488`.

| Segment, on both shafts | Length / mm | Diameter / mm |
| --- | ---: | ---: |
| 1 | 80 | 25 |
| 2 | 340 | 35 |
| 3 | 80 | 25 |

| Gear | Center (X, Y) | Teeth z | Module m | Face width / mm | Pitch diameter / mm |
| --- | --- | ---: | ---: | ---: | ---: |
| Smaller gear | (-20, -100) | 20 | 4 | 32 | 80 |
| Larger gear | (-20, 40) | 50 | 4 | 32 | 200 |

The application calculates pitch diameter as `d = z × m`. For this example, its simplified gear-spacing target is `(80 + 200) / 2 = 140 mm`, matching the difference between the two Y coordinates.

![Stepped-shaft editor showing three segments and a total length of 500 mm](docs/images/03-shaft-editor.png)

*Figure 3 · Each segment has independent length and diameter fields. “添加” adds a segment; the minus button at the end of a row removes one. At least one segment is retained.*

To see the spacing guide, enable the magnet and grab the larger gear away from the shaft centerline. Drag it slightly up or down near the `140 mm` separation. An orange `Mesh Dist: 140mm` guide appears. The two gear rectangles meet at their schematic pitch boundaries; actual teeth are not drawn.

![Orange 140 mm gear-spacing guide displayed while dragging a gear](docs/images/04-meshing-snap.png)

*Figure 4 · Gear-spacing snapping during a drag. This guide reflects the application's simplified geometry rule; it does not confirm valid meshing or freedom from interference.*

## Parameters, units, and snapping

### Coordinates and dimensions

- X is the axial position and increases to the right. Y positions the shaft axes across the canvas and increases downward. Negative coordinates are supported.
- Length values are interpreted as millimeters and angles as degrees. Zoom affects only the view, not component dimensions. The on-screen scale is not a calibrated print scale.
- A newly dropped component always lands on the `10`-unit grid, even when the magnet is disabled.
- Entering X or Y directly bypasses drag snapping. Editing a shaft's coordinates directly also bypasses linked movement of parts on that shaft.

### Snapping during dragging

| Dragged object | Behavior with automatic snapping enabled |
| --- | --- |
| Shaft | X and Y snap to the `10`-unit grid |
| Housing | X and Y snap to the `20`-unit grid |
| Other components | X snaps to the `10`-unit grid; Y prioritizes gear-spacing snapping, then shaft alignment, then the grid |

Shaft alignment compares only Y distance and displays a green guide when active. It does not check whether the component is within the shaft's actual length. Gear-spacing snapping applies to spur, helical, and bevel gears. It uses the sum of two radii calculated as `z × m / 2` to set the Y separation, without checking X alignment, matching modules, or compatibility between gear types. At the default scale, proximity thresholds are approximately 25 screen pixels for shaft alignment and 20 for gear spacing.

### Linked movement with a shaft

Dragging a shaft also moves parts whose original Y coordinate differs from the shaft's by less than `0.1`, provided their centers lie within the shaft's original length plus `20` units at either end. Other shafts and housings are excluded.

This relationship is inferred from positions rather than stored as a permanent assembly constraint. Recheck component placement after changing segment lengths or entering shaft coordinates directly.

## Current scope and limitations

The following points describe the current implementation so that a schematic layout is not mistaken for a completed mechanical design.

- **Persistence:** components live only in the current page's React state. Reloading or closing the page discards the layout. Project save / load, autosave, undo, and redo are not implemented.
- **File output:** there is no built-in PNG, SVG, DXF, STEP, PDF, or BOM export control. The illustrations in this documentation are browser screenshots. Use browser or operating-system screenshot tools to record a layout.
- **Engineering analysis:** strength, life, efficiency, load capacity, motion simulation, collision detection, and standards compliance are not evaluated. The housing is a boundary schematic only.
- **Gear geometry:** the canvas uses rectangles and markings rather than involute tooth profiles. Spur, helical, and bevel gear display diameters all use `z × m`; there is no normal / transverse module conversion or bevel pitch-cone calculation. The helix angle is editable, but the current diagonal markings retain their fixed direction and spacing.
- **Parameter coverage:** fields such as pressure angle and bore / inside diameter exist in data types without complete editing controls and are not used for assembly verification. Only bevel gears offer `0° / 90° / 180° / 270°` orientation buttons. Shafts remain aligned with X.
- **Validation:** some numeric inputs have HTML minimum / maximum hints, but there is no unified validation of parameter ranges or dimensional compatibility. Use sensible positive dimensions and integer tooth counts. Editing circlip width does not alter its fixed rendered strip width.
- **Version and licensing:** the sidebar displays `v2.3`, while the package version is `0.0.0`. The repository currently has no LICENSE file; public access does not itself grant an open-source license.

## Development

### Stack

| Area | Implementation |
| --- | --- |
| Interface | React 19.2, TypeScript 5.8 |
| Development and build | Vite 6, `@vitejs/plugin-react` |
| Drawing | SVG grid, component shapes, and guides |
| Styling | Tailwind CSS CDN |
| Icons | Lucide React |
| State | React `useState`; no backend or database |

Version families are taken from `package.json`. No dependency lockfile is currently committed, so installed patch / minor versions may vary.

### Source layout

```text
.
├── index.html                  # HTML container, CDN styling, application entry
├── index.tsx                   # React mounting
├── App.tsx                     # Component state, defaults, linked shaft movement
├── types.ts                    # Component types, parameter structures, grid constant
├── components/
│   ├── ComponentPalette.tsx    # Component palette and drag source
│   ├── DesignCanvas.tsx        # SVG rendering, pan, zoom, dragging, snapping
│   └── PropertiesPanel.tsx     # Property editing and deletion
├── utils/geometry.ts           # Geometry helpers; not used by the current canvas
├── docs/images/                # Actual screenshots used in this documentation
├── README.md                   # Chinese documentation
└── README.en.md                # English documentation
```

Adding a component requires coordinated updates to `types.ts`, defaults in `App.tsx`, the palette, canvas rendering, and property controls, together with a decision about its snapping behavior.

## Troubleshooting

| Symptom | What to check |
| --- | --- |
| Installation reports an unsupported Node.js version | Use a Node.js version satisfying dependency `engines`; these instructions were verified with 24.19.0 |
| Styling is missing or the layout is broken | Check that the browser can load `https://cdn.tailwindcss.com` |
| The dev server uses a port other than 3000 | The configured port may be occupied; open the address Vite actually prints |
| Components disappear after reloading | Persistence is not implemented; take a screenshot before reloading |
| A newly added part does not align with a shaft | Initial drops use the grid only; drag the created part near the shaft axis with the magnet enabled |
| Precise positioning is needed | Enter X and Y in the properties panel; this bypasses snapping and linked shaft movement |
| A gear snaps somewhere unexpected | Higher-priority gear-spacing snapping may be active; temporarily disable the magnet and enter coordinates |
| Components are outside the visible area | Pan or zoom out; reset restores the initial viewport rather than fitting all components |

This documentation and its screenshots were prepared from the running repository on **2026-10-27**. Both language versions cover the same operations and limitations. Screenshots preserve the application's actual Chinese and English labels.
