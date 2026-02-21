# Session Log

This file tracks session handoffs so the next Claude Code instance can quickly get up to speed.

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
