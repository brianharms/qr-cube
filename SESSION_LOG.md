# Session Log

This file tracks session handoffs so the next Claude Code instance can quickly get up to speed.

---

## Session — 2026-03-10 18:17

### Goal
Fix non-manifold edges in STL export (specifically with chamfer/fillet edge treatment) and deploy the app publicly.

### Accomplished
- **Non-manifold edge fix** — Implemented uniform subdivision across all faces and edge treatment geometry. Root cause: `buildEdgeTreatment` used per-edge `N = max(faceQRData[fi], faceQRData[fj])` which didn't match adjacent face vertex counts, creating T-junctions. Fix: two-pass `rebuild()` — first pass computes all QR data and finds `maxQRCount`, second pass builds all geometry with `uniformTotal = maxQRCount + 8`. `buildFaceGeometry` now accepts `uniformTotal` param and centers QR grid via dynamic `quietZone = floor((uniformTotal - count) / 2)`. `buildBlankFace` now subdivides into NxN grid instead of single quad. `buildEdgeTreatment` accepts single `uniformN` instead of per-face array.
- **GitHub Pages deployment** — Made repo public, enabled GitHub Pages, app is live at `https://brianharms.github.io/qr-cube/`
- **README simplified** — Replaced detailed technical README with short prototype-status description
- **New skills created** — `/hypothesize` (3 parallel agents diagnose bugs) and `/propose` (3 parallel agents propose design approaches) added to `~/.claude/skills/`

### In Progress / Incomplete
- Nothing — all work completed.

### Key Decisions
- Uniform subdivision ensures all shared edges (face↔face, face↔edge-treatment) have identical vertex counts and positions, eliminating T-junctions at boundaries
- `uniformTotal` defaults to 29 (21 + 8) when no QR data present
- Chose GitHub Pages over Cloudflare/custom domain for simplicity — user's `ritual.industries` is on Figma Sites so path-based routing wasn't feasible without DNS changes

### Files Changed
- `index.html` — non-manifold fix in `buildFaceGeometry`, `buildBlankFace`, `buildEdgeTreatment`, `rebuild()`
- `README.md` — simplified to prototype disclaimer
- `~/.claude/skills/hypothesize/SKILL.md` — new skill
- `~/.claude/skills/propose/SKILL.md` — new skill

### Known Issues
- None identified.

### Running Services
- None.

### Next Steps
- User may want to test STL export with chamfer/fillet in their slicer to confirm non-manifold edges are resolved
- If user wants `ritual.industries/qr` routing, would need to move DNS to Cloudflare for path-based redirects or use `qr.ritual.industries` subdomain

---

## Session — 2026-02-24 00:00

### Goal
Polish pass on `index.html` (~1840 lines): 5 improvements planned in advance via plan mode.

### Accomplished
- **Grid visibility fix** — `updateGridColor()` now uses clean inversion: `gridColorVal/255` in dark, `1 - gridColorVal/255` in light. Grid extents slider max → 500 (step 5, default 100). Cell size default → 5. Grid is now actually visible in both modes.
- **View mode buttons vertical layout** — replaced `.segment-control` horizontal row with a flex-column div. Buttons show full labels (`Wireframe`, `Ghosted`, `Shaded`, `Rendered`) with left-aligned text; no more overflow.
- **Ambient floating geometry** — 22 wireframe shapes (IcosahedronGeometry, OctahedronGeometry, TetrahedronGeometry, TorusGeometry, DodecahedronGeometry) placed on sphere r=180–360. Slowly rotate per-frame via `userData.rv`. Color tracks accent; opacity 0.06 dark / 0.04 light. Added via `floatingGroup` + `floatingShapes[]`.
- **Slider range expansion** — Size: min 20 → max 200, default 44mm. Depth: min 0.1 → max 5.0. "Too small" warning fires in `rebuild()` when `cubeSize < 32`.
- **Preset-driven accent color** — Added `hslAccent(h,s,l)` helper and `syncPresetAccent(id)`. Solid/clay/neon/gradient/glass(tinted)/chrome(tinted) push their hue into `--accent`/`--accent-dim` CSS vars. Holographic cycles rainbow in `animate()` at 0.4°/frame via `holoHue`. `applyTheme()` delegates to `syncPresetAccent` instead of hardcoding. Color-related preset sliders call `syncPresetAccent` live.

### In Progress / Incomplete
- Nothing — all 5 planned changes implemented and complete.

### Key Decisions
- `hslAccent` and `syncPresetAccent` defined via `function` keyword → hoisted, safe to call from `animate()` which is defined earlier in file.
- `applyTheme()` no longer manually sets `--accent`; all accent logic flows through `syncPresetAccent` which falls back to `currentAccent` when preset has no color opinion.
- Floating shapes share a single `floatingMat` (wireframe, transparent) — color updated via `floatingMat.color.set(currentAccent.three)` in `applyTheme`.

### Files Changed
- `/Users/brianharms/Desktop/Claude Projects/qr-cube/index.html` — sole file modified

### Known Issues
- None identified.

### Running Services
- None.

### Next Steps
- User reviews visual output and critiques. Likely feedback areas: floating shape density/opacity, grid default extents (100 may still feel sparse), accent color saturation levels per preset.

---

## Session — 2026-02-21 14:24

### Goal
Build a single-page web app (QR Cube) that generates 3D-printable cubes with unique QR codes engraved into each face. The user iteratively reviewed the visual output and requested fixes/features across multiple rounds.

### Accomplished
- Full app built in a single `index.html` — Three.js 3D preview, QR generation, STL export
- URL distribution logic: 1–6 URLs mapped across 6 cube faces
- Height-field geometry engine: dark QR modules recessed, vertical walls at transitions, edge walls at face boundaries
- Fixed all inverted wall normals (bottom inter-cell walls + all 4 edge wall directions) that caused interior recess walls to render as transparent backfaces
- Added face highlighting: focusing a URL input highlights corresponding cube faces in accent color
- Added theme system: click title for accent color picker (6 colors) + light/dark mode toggle, persists to localStorage
- Added Flip Normals debug button
- Added chamfer/fillet edge treatment: None/Chamfer/Fillet toggle + size slider, generates 12 edge strips + 8 corner pieces, with smooth fillet arcs (8 segments) and spherical corner patches
- Face geometry auto-insets when edge treatment is active; edge size capped at 15% of cube size
- Created detailed `README.md` with feature docs, technical architecture, and printing tips
- Backed up to GitHub as private repo: `https://github.com/brianharms/qr-cube`

### In Progress / Incomplete
- No unfinished work — all requested features have been implemented

### Key Decisions
- **Manual height-field geometry over CSG**: faster, zero deps, guaranteed watertight mesh
- **pushQuad auto-winding**: cross-product dot check auto-corrects triangle winding to match provided normal — means the normal direction passed in MUST be correct or both winding and normal will consistently face the wrong way
- **Wall normal convention**: walls face INWARD toward recess center (so they're visible when looking into the depression from outside the cube)
- **Edge treatment max**: capped at 15% of cube size to prevent degenerate geometry
- **Fillet subdivision**: 8 segments per edge arc, 4x4 grid per spherical corner patch
- **QR error correction**: level H (highest) for scanning reliability on textured surfaces
- **Single file architecture**: all CSS + JS inline in `index.html`, no build step

### Files Changed
- `index.html` — the entire app (~1255 lines)
- `README.md` — project documentation
- `SESSION_LOG.md` — this file

### Known Issues
- No known bugs after the normal fixes
- Very long URLs can fail QR generation (the app shows an error warning for this)
- Fillet corner patches use a latitude/longitude parameterization that may have slight distortion at the poles — visually negligible at typical chamfer sizes

### Running Services
- None — static HTML file, no dev server

### Next Steps
- User may want to visually review chamfer/fillet rendering and iterate on appearance
- Potential enhancements: per-face color/material, QR code preview overlay, drag-and-drop URL reordering, multi-color STL export, adding a "Unify Normals" post-processing button (was discussed but superseded by direct normal fixes)
