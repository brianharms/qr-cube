# QR Cube

A single-page web app that generates 3D-printable cubes with unique QR codes engraved (recessed) into each face. Enter up to 6 URLs, adjust parameters, preview in real-time 3D, and export a watertight STL file ready for slicing.

## How It Works

Each face of the cube is built as a height-field grid. Dark QR modules are recessed into the surface, light modules stay flush. Vertical walls connect height transitions between adjacent cells. The result is a guaranteed watertight mesh — no boolean/CSG operations needed.

## Features

### QR Code Generation
- Enter 1–6 URLs in the side panel
- QR codes use **error correction level H** (highest) for reliable scanning on textured 3D-printed surfaces
- 4-module quiet zone on all sides per QR spec
- Automatic distribution across faces:

| URLs entered | Distribution |
|---|---|
| 1 | Same QR on all 6 faces |
| 2 | Each on 3 faces |
| 3 | Each on 2 faces |
| 4–6 | One per face, remainder blank |

### 3D Preview
- Real-time Three.js viewport with orbit controls (drag to rotate, scroll to zoom)
- Focused URL input highlights its corresponding cube faces in the accent color
- Face tag labels next to each input show which faces are mapped

### Parameters
- **Cube Size**: 30–120mm (default 50mm)
- **Engrave Depth**: 0.5–3mm (default 1.5mm)
- **Edge Treatment**: None, Chamfer, or Fillet with adjustable size
  - Chamfer: flat 45-degree bevels on all 12 edges + 8 corner triangles
  - Fillet: smooth quarter-arc curves on edges + spherical corner patches
  - Size auto-capped at 15% of cube size

### Theming
- Click the **QR CUBE** title to open the theme popover
- 6 accent colors: orange, blue, green, purple, red, cyan
- Light / dark mode toggle
- Theme persists to localStorage

### Export
- Binary STL export via the **Export STL** button
- Filename includes cube size (e.g., `qr-cube-50mm.stl`)
- Includes all face geometry + edge treatment geometry
- Open in PrusaSlicer, Cura, or any slicer — mesh is watertight with consistent normals

### Utilities
- **Flip Normals** button to reverse all mesh normals (debug aid)
- Printability warnings when QR module pitch drops below 1mm
- Triangle count + URL count in the status bar

## Technical Details

### Architecture
Single `index.html` file with all CSS and JS inline. No build step, no local server needed — just open in a browser.

### Dependencies (CDN)
- **Three.js v0.170** — rendering, OrbitControls, STLExporter, BufferGeometryUtils
- **qrcode-generator v1.4.4** — QR encoding with `isDark(row, col)` grid access
- **Space Grotesk + Space Mono** — Google Fonts

### Geometry Engine
Each of the 6 cube faces uses a coordinate system:

| Face | Normal | U axis | V axis |
|------|--------|--------|--------|
| Front | +Z | +X | +Y |
| Back | -Z | -X | +Y |
| Right | +X | -Z | +Y |
| Left | -X | +Z | +Y |
| Top | +Y | +X | -Z |
| Bottom | -Y | +X | +Z |

Key functions:
- `buildFaceGeometry()` — height-field grid per face with QR module recesses
- `buildEdgeTreatment()` — chamfer/fillet strips for 12 edges + 8 corners
- `pushQuad()` — auto-corrects triangle winding via cross-product dot check
- `buildWall()` — vertical walls between height transitions

Worst case triangle count: ~60K (6 faces x 30x30 modules + edge treatment), well within browser and slicer limits.

### Mesh Properties
- All triangles have consistent CCW winding (Three.js front-face convention)
- Normals are explicitly set per vertex (not computed)
- Watertight by construction — every cell covered, every transition walled
- Engrave depth never penetrates through the cube

## Usage

```
open index.html
```

Or just double-click the file. Works in any modern browser (Chrome, Firefox, Safari, Edge).

## Printing Tips

- **Module pitch < 1mm**: The app warns you. Use a shorter URL or larger cube size.
- **Engrave depth**: 1–2mm works well for most FDM printers. Too shallow and the QR won't scan; too deep and bridging becomes an issue.
- **Chamfer/fillet**: Helps with bed adhesion and handling. 2mm is a good default.
- **Scaling in slicer**: You can always scale the STL in your slicer software if the default size isn't right.
- **Color contrast**: Print in a light color and fill recesses with dark paint/resin, or print face-down on a dark build plate for contrast.
