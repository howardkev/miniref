# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the App

No build step. Open `index.html` directly in a browser:

```
open index.html
```

Or serve it locally to avoid any browser file:// restrictions:

```
python3 -m http.server 8080
```

## Architecture

The entire application is `index.html` — a single self-contained file with inline CSS, HTML, and JavaScript (~1,000 lines). There is no build system, bundler, or package manager. Three.js is loaded from a CDN.

### Key sections of index.html

- **Styles** (`<style>`) — CSS variables for theming, custom slider/button styling, grid layout
- **HTML** — 300px sidebar with controls + main content area for canvases
- **JavaScript** (`<script>`) — all app logic inline at the bottom

### JavaScript structure (roughly in order)

1. **Control references** — `controls` object mapping all input elements
2. **STL parsers** — `parseBinarySTL` / `parseASCIISTL`, auto-detected by comparing expected vs actual byte count (`84 + triangles × 50`)
3. **Geometry loading** — centers and normalizes scale to max dimension = 1.0 (so lamp/camera parameters are unit-scale regardless of STL units)
4. **Lighting rig** — `setupLighting()` builds two rings of lights (upper + equatorial) plus ambient fill, all in model-local space then rotated by the model's Euler angles
5. **Render pipeline** — `renderViews()` renders 4 × 512×512 canvases (front/back/left/right) and caches them in `renderedCanvases`
6. **Posterization** — `posterizeCanvas()` converts to grayscale (BT.601 coefficients), normalizes by actual min/max, then quantizes to 2-level or multi-level
7. **Viewport** — `updateViewport()` switches between quad and single-view display
8. **Settings** — saved/loaded from LocalStorage keyed by sanitized filename (`miniref_[filename]`)
9. **Download** — individual PNGs or a composited 1024×1024 reference sheet

### Rendering choices

- `MeshLambertMaterial` (diffuse only, no specular) — intentional for value study
- Perspective camera at distance 2.2, FOV 35°
- Black background (`#111111`), antialiasing enabled, `preserveDrawingBuffer: true` for canvas export
