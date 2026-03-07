# MiniRef

**https://howardkev.github.io/miniref/**

A single-file web app that generates painting reference guides from 3D STL models.

## What It Does

MiniRef helps miniature painters study how light falls on a 3D model. It:

1. Loads an STL file (drag & drop or file picker)
2. Simulates a lamp rig with configurable height, distance, and intensity
3. Renders 4 cardinal angles (Front/Back/Left/Right) at 1024×1024
4. Optionally **posterizes** the render to discrete value steps for value study
5. Lets you download individual views or a 2048×2048 composite reference sheet

## Usage

Open `index.html` in any modern browser. No install, no server, no build step required.

## Architecture

The entire app is a single `index.html` (~1,000 lines). The only external dependency is Three.js, loaded from a CDN.

### Lighting Rig

Four fixed cardinal point lights (left/front/right/back) placed at upper-front positions around the model, plus:

- **Weak top-down key** — always in world space directly above the model, unaffected by model rotation
- **Ambient fill** — uniform omnidirectional base light

Light positions are anchored relative to the model's actual top extent after scaling, so placement is accurate regardless of model proportions.

### Rendering

- **Lambert material** — purely diffuse shading with no specular highlights, keeping focus on light/shadow shapes for value study
- **4 × 1024×1024 canvases** rendered per update, cached so view switching doesn't re-render
- Two render passes per view: one with the actual model color (for "Original" display) and one in white (for posterization and histogram)
- Geometry is centered and scaled to unit size on load, so lamp parameters are meaningful regardless of STL units

### Posterization

Two modes:

- **Original** — full-color Lambert render with user-selected model color
- **Multi Level** — converts to discrete grayscale value steps

Grayscale uses ITU-R BT.601 coefficients (`0.299R + 0.587G + 0.114B`) for perceptual accuracy. Normalization uses actual rendered min/max rather than absolute 0–255, ensuring full contrast range regardless of lighting setup.

Multi Level mode supports 2–10 levels. Each level band has a visibility toggle (click the swatch to hide/show that value band). Threshold positions between bands can be dragged interactively on the value distribution histogram. Custom thresholds are stored per level count.

### Value Distribution Histogram

Displays the grayscale value distribution of the front view render. Threshold markers are draggable to adjust posterization band boundaries. Resets to equal spacing when the level count changes.

### Settings

All slider values, posterize mode, level count, layer visibility, custom thresholds, and model color are saved to LocalStorage per filename and restored automatically when the same file is reopened.
