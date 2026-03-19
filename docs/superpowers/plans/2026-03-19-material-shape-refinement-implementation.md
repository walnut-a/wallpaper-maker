# Material Shape Refinement Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rebuild `镜面切片` and `长虹玻璃柱面` so both modules read as stronger abstract main forms with restrained material cues, while keeping the current single-file Canvas workflow intact.

**Architecture:** Keep everything inside `index.html` and refactor the two shape modules into staged drawing passes. For `长虹玻璃柱面`, establish the column body first, then apply clipped internal ridges and lightweight refraction inspired by liquid-glass style masking. For `镜面切片`, establish a small set of oblique planes first, then add edge contrast and selective highlights inspired by multi-plane refraction demos rather than blur-first effects.

**Tech Stack:** HTML, CSS, vanilla JavaScript, Canvas 2D API, offscreen canvas layers, local static server, Playwright MCP, browser evaluation helpers

---

## File Structure

- Modify: `index.html`
  - Refactor the `reeded-column` and `mirror-slices` entries in `shapeModes`.
  - Replace the current monolithic drawing functions with staged helpers.
  - Reduce accent-layer dependency for these two modes.
- Reference: `docs/superpowers/specs/2026-03-19-material-shape-refinement-design.md`
  - Use this as the behavioral contract for shape intent, parameter scope, and visual acceptance.
- Reference only: external GitHub implementations
  - [shuding/liquid-glass](https://github.com/shuding/liquid-glass)
  - [iyinchao/liquid-glass-studio](https://github.com/iyinchao/liquid-glass-studio)
  - [OverShifted/LiquidGlass](https://github.com/OverShifted/LiquidGlass)
  - [keijiro/UnityRefractionShader](https://github.com/keijiro/UnityRefractionShader)
  - [drcmda/the-substance](https://github.com/drcmda/the-substance)
  - [tatsmaki/three-diamond](https://github.com/tatsmaki/three-diamond)

## Notes Before Execution

- Do not pull in any new dependency. Borrow the drawing order and visual logic only.
- Avoid AGPL contamination. If you consult algorithm descriptions from repositories like `OverShifted/LiquidGlass`, re-express the idea in original Canvas code rather than copying implementation.
- There is still no package-based test runner. Use browser verification as the test harness.
- Serve the page locally before running browser checks:

```bash
cd /Users/zhaolixing/Documents/GitHub/wallpaper-maker
python3 -m http.server 8123
```

- Open the page once before checks so `window.__app` is available:

```bash
open http://127.0.0.1:8123
```

## Chunk 1: Parameter Model Cleanup

### Task 1: Tighten the advanced controls for the two material-heavy shapes

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write the failing browser check**

Run in the browser:

```js
(() => {
  const modes = window.__app.getModeRegistry();
  return {
    reeded: modes.shapeModes["reeded-column"].groups[0].controls.map((item) => item.label),
    mirror: modes.shapeModes["mirror-slices"].groups[0].controls.map((item) => item.label),
  };
})()
```

Expected initial result: `reeded-column` still exposes `折射强度`, and `mirror-slices` still exposes `主形体数量`.

- [ ] **Step 2: Update `reeded-column` controls**

Change the shape defaults and labels so the mode exposes only:

- `柱面数量`
- `柱体宽度`
- `棱线密度`
- `折射幅度`

Suggested default object:

```js
defaults: {
  columnCount: 0.22,
  columnWidth: 0.56,
  ridgeDensity: 0.42,
  refractionOffset: 0.48,
}
```

- [ ] **Step 3: Update `mirror-slices` controls**

Change the shape defaults and labels so the mode exposes only:

- `切片数量`
- `切片倾角`
- `厚薄反差`
- `边缘锐度`

Suggested default object:

```js
defaults: {
  sliceCount: 0.36,
  sliceAngle: 0.54,
  sliceVariance: 0.44,
  sliceSharpness: 0.62,
}
```

- [ ] **Step 4: Verify the control labels**

Run the same browser check again.

Expected: `折射强度` and `主形体数量` are gone for these two modes, and the new labels appear.

## Chunk 2: 长虹玻璃柱面 Rebuild

### Task 2: Rebuild the reeded-column renderer around body-first composition

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write the failing visual-state check**

Run:

```js
(() => {
  const select = document.getElementById("shapeModeSelect");
  select.value = "reeded-column";
  select.dispatchEvent(new Event("change", { bubbles: true }));
  return window.__app.getState().shapeMode;
})()
```

Expected: PASS with `reeded-column`, but the current render still uses the old monolithic body.

- [ ] **Step 2: Split the renderer into staged helpers**

Replace the current implementation with:

- `drawReededColumnCore()`
- `drawReededColumnInternalRidges()`
- `drawReededColumnRefraction()`

Keep the public entry as:

```js
function drawReededGlassGeometry(ctx, width, height, palette, currentState, rng) {
  const composition = buildReededColumnComposition(width, height, currentState, rng);
  drawReededColumnCore(ctx, width, height, palette, currentState, composition);
  drawReededColumnInternalRidges(ctx, width, height, palette, currentState, composition);
  drawReededColumnRefraction(ctx, width, height, palette, currentState, composition);
}
```

- [ ] **Step 3: Build the body-first composition**

Create a small composition builder that returns one or two large rounded column bounds:

```js
function buildReededColumnComposition(width, height, currentState, rng) {
  return {
    columns: [
      { x, y, width, height, radius, highlightSide, shadowSide },
    ],
  };
}
```

Rules:

- `columnCount` should resolve to `1` or `2`, never a busy field of stripes.
- The lead column should own the visual center.
- Secondary column, if present, should be offset and quieter.

- [ ] **Step 4: Clip ridges inside the column only**

Inside each column:

- create a clip path
- draw vertical ridge bands with alternating light/dark alpha
- modulate the band width slightly to avoid a wallpaper-filter look

Use a dedicated layer:

```js
const ridgeLayer = createLayer(width, height);
ridgeLayer.ctx.save();
// clip to the column path
// paint ridge bands
ridgeLayer.ctx.restore();
ctx.drawImage(ridgeLayer.canvas, 0, 0);
```

- [ ] **Step 5: Add lightweight refraction**

Use a second clipped pass that shifts a blurred accent field a few pixels sideways inside the column and mixes it back with low alpha. The effect should read as “background passing through glass” rather than “texture on top”.

Implementation rule:

- use positional offset and local contrast
- do not generate full-image stripe overlays

- [ ] **Step 6: Verify the refactor still renders**

Run:

```js
(() => {
  const before = window.__app.getCanvasSignature();
  window.__app.setAdvanced("refractionOffset", 0.78);
  return new Promise((resolve) => requestAnimationFrame(() => {
    resolve({
      before,
      after: window.__app.getCanvasSignature(),
      state: window.__app.getState().advanced,
    });
  }));
})()
```

Expected: `after !== before`, and `refractionOffset` is present in state.

## Chunk 3: 镜面切片 Rebuild

### Task 3: Rebuild mirror-slices around a small set of deliberate planes

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write the failing visual-state check**

Run:

```js
(() => {
  const select = document.getElementById("shapeModeSelect");
  select.value = "mirror-slices";
  select.dispatchEvent(new Event("change", { bubbles: true }));
  return window.__app.getState().shapeMode;
})()
```

Expected: PASS with `mirror-slices`, but the current render still behaves like blurred bars plus one diagonal streak.

- [ ] **Step 2: Split the renderer into staged helpers**

Refactor into:

- `drawMirrorSliceCore()`
- `drawMirrorSliceEdges()`
- `drawMirrorSliceHighlights()`

Keep the public entry:

```js
function drawMirrorSliceGeometry(ctx, width, height, palette, currentState, rng) {
  const composition = buildMirrorSliceComposition(width, height, currentState, rng);
  drawMirrorSliceCore(ctx, width, height, palette, currentState, composition);
  drawMirrorSliceEdges(ctx, width, height, palette, currentState, composition);
  drawMirrorSliceHighlights(ctx, width, height, palette, currentState, composition);
}
```

- [ ] **Step 3: Build 3 to 5 major planes**

The composition builder should resolve:

- a shared angle family
- 3 to 5 planes
- width variance driven by `sliceVariance`
- stable overlap ordering

Sketch:

```js
function buildMirrorSliceComposition(width, height, currentState, rng) {
  return {
    planes: [
      { path, fill, shadowAlpha, edgeAlpha, zIndex },
    ].sort((a, b) => a.zIndex - b.zIndex),
  };
}
```

- [ ] **Step 4: Replace rounded bars with polygonal or tapered planes**

Use path-based shapes instead of repeated soft rounded rects. Each plane should include:

- main face
- one bright edge
- one dim edge

Keep blur restrained. A plane should still read without blur.

- [ ] **Step 5: Reduce highlight count**

Only one or two planes should receive concentrated highlight streaks. Remove the current single global diagonal accent from `drawAccentLayer()` and keep those cues inside the module renderer.

- [ ] **Step 6: Verify render responsiveness**

Run:

```js
(() => {
  const before = window.__app.getCanvasSignature();
  window.__app.setAdvanced("sliceVariance", 0.8);
  return new Promise((resolve) => requestAnimationFrame(() => {
    resolve({
      before,
      after: window.__app.getCanvasSignature(),
      state: window.__app.getState().advanced,
    });
  }));
})()
```

Expected: `after !== before`, and `sliceVariance` is present in state.

## Chunk 4: Accent Cleanup And Visual Acceptance

### Task 4: Reduce accent-layer dependency and recheck phone readability

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Trim `drawAccentLayer()` for the two target shapes**

Rules:

- `reeded-column` keeps only a restrained column highlight or ambient lift
- `mirror-slices` no longer depends on one large diagonal stroke as its main finishing cue

- [ ] **Step 2: Capture before/after screenshots**

Save screenshots for manual review at:

- `playwright-output/reeded-column-refined.png`
- `playwright-output/mirror-slices-refined.png`

Expected visual read:

- `reeded-column`: body first, ridges second
- `mirror-slices`: planes first, highlights second

- [ ] **Step 3: Check mobile-sized readability**

Resize to a phone viewport and confirm the main shape still reads at reduced scale.

Suggested browser evaluation:

```js
(() => {
  const canvas = document.getElementById("artworkCanvas");
  return { width: canvas.clientWidth, height: canvas.clientHeight };
})()
```

Expected: preview remains visible and the shape silhouette is still legible in the captured screenshot.

## Chunk 5: Final Verification And Commit

### Task 5: Verify export and project state remain healthy

**Files:**
- Modify: `index.html` if verification exposes regressions

- [ ] **Step 1: Verify export still works**

Trigger `window.__app.exportPng()` in the browser and confirm the download starts.

- [ ] **Step 2: Check browser console**

Expected: `0 errors / 0 warnings`

- [ ] **Step 3: Review git status**

Expected: only the intended file edits plus any deliberately captured screenshots.

- [ ] **Step 4: Commit**

```bash
git add index.html docs/superpowers/specs/2026-03-19-material-shape-refinement-design.md docs/superpowers/plans/2026-03-19-material-shape-refinement-implementation.md
git commit -m "feat: refine material-inspired wallpaper shapes"
```
