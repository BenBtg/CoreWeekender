# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CoreWeekender is a proof-of-concept canvas-based wave visualizer ("Wave Visualizer v6") for Daventry Core. It is a single self-contained HTML file with no external dependencies, no build step, and no package manager.

## Running the Project

Open `index.html` directly in a browser — no server required. There are no build, lint, or test commands.

## Architecture

Everything lives in `index.html` as a single file with three sections: CSS (in `<style>`), HTML markup (control panel + `<canvas>`), and JavaScript (in `<script>`).

### Animation Pipeline

The rendering runs via `requestAnimationFrame` in a continuous loop:

1. **`getControlPoints(time)`** — Generates 10 control points whose Y positions oscillate using layered `Math.sin`/`Math.cos` calls driven by elapsed time. These define the central "spine" of the wave.

2. **`getSpineY(controlPoints, nx)`** — Given a normalised X position `nx` ∈ [0, 1], interpolates between control points using **Catmull-Rom spline** (`catmullRom()`), producing a smooth continuous curve.

3. **`drawStrands(time)`** — Iterates over `numStrands` strands. For each strand:
   - Computes 200 sample points along the spine with a per-strand lateral offset (`strandOffset`) and a convergence term that pulls strands together and apart rhythmically.
   - Draws the path using `quadraticCurveTo` for smooth rendering.
   - Renders two passes: a wide blurred **glow pass** (if `glowBlur > 0`) then a sharp **line pass**, using warm amber/orange colours (`rgb(255, g, b)`) that shift toward the strand edges.

4. **`animate(timestamp)`** — Clears the canvas, fills the dark green background (`#071f10`), calls `drawStrands`, and schedules the next frame.

### User Controls

Five sliders/selects feed directly into `drawStrands` on each frame (no state caching):

| Control | Variable | Effect |
|---|---|---|
| Blend mode select | `blendMode` | `globalCompositeOperation` for strand compositing |
| Opacity slider | `opacity` | Alpha on both glow and sharp stroke |
| Glow slider | `glowBlur` | `shadowBlur` radius; set to 0 to skip glow pass |
| Width slider | `lineWidth` | Strand line width (stored ×10, divided on read) |
| Strands slider | `numStrands` | Number of parallel strands rendered |

### Key Constants

- `NUM_CONTROL_POINTS = 10` — spine resolution
- `STEPS = 200` — sample points per strand per frame
- Background colour: `#071f10` (dark green)
- Strand colour range: warm amber `rgb(255, 120–180, 5–20)`
