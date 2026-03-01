# MiniRef

A single-file web app that generates painting reference guides from 3D STL models.

## What It Does

MiniRef helps miniature painters study how light falls on a 3D model. It:

1. Loads an STL file (drag & drop)
2. Simulates a **ring lamp** lighting rig (like real miniature painters use)
3. Renders 4 cardinal angles (Front/Back/Left/Right)
4. Optionally **posterizes** the image to simplify values into discrete steps
5. Lets you download individual views or a composite reference sheet

## Usage

Open `index.html` in any modern browser. No install, no server, no build step required.

## Architecture

The entire app is a single `index.html` (~1,000 lines). The only external dependency is Three.js, loaded from a CDN.

### Lighting Rig

The lighting simulates the ring lamps miniature painters use:

- **Upper ring** — N lights evenly distributed in a circle above the model
- **Equatorial ring** — fill lights at height 0, offset by half a step from the upper ring
- **Ambient fill** — uniform omnidirectional base light

All lights are defined in model-local space, then rotated by the model's X/Y/Z rotation matrix — so the light stays fixed while the model rotates, matching how you'd spin a mini under a lamp.

### Rendering

- **Lambert material** — purely diffuse shading with no specular highlights, keeping the focus on light/shadow shapes for value study
- **4 × 512×512 canvases** rendered per update, cached so view switching doesn't re-render
- Geometry is centered and scaled to unit size on load, so lamp radius/height parameters are meaningful regardless of STL units

### Posterization

Converts renders to discrete value steps for simplified value study. Grayscale uses ITU-R BT.601 coefficients (`0.299R + 0.587G + 0.114B`) for perceptual accuracy. Normalization uses actual rendered min/max rather than absolute 0–255, ensuring full contrast range regardless of lighting setup.

### Settings

All slider values, posterize mode, and model color are saved to LocalStorage per filename and restored automatically when the same file is reopened.
