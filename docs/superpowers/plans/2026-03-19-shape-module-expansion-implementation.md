# Shape Module Expansion Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add three new shape modules to the modular wallpaper generator so users have more meaningful main-form choices without changing the single-file architecture.

**Architecture:** Keep the app inside `index.html` and extend the existing modular composition system in place. Add new entries to `shapeModes`, wire their defaults and advanced controls into the current state model, then attach each new module to a dedicated Canvas drawing function and accent-layer behavior.

**Tech Stack:** HTML, CSS, vanilla JavaScript, Canvas 2D API, local static server, Playwright MCP, browser evaluation helpers

---

## File Structure

- Modify: `index.html`
  - Add new shape mode definitions, advanced controls, drawing helpers, accent logic, and random-mix coverage.
- Modify: `README.md`
  - Update the feature description so the public repo matches the expanded shape choices.
- Reference: `docs/superpowers/specs/2026-03-19-shape-module-expansion-design.md`
  - Use this as the behavioral contract for which modules are added and how they should behave.

## Notes Before Execution

- There is still no package-based test runner. Use browser verification as the test harness.
- Serve the page locally before running browser checks:

```bash
cd /Users/zhaolixing/Documents/GitHub/wallpaper-maker
python3 -m http.server 8123
```

- Verify behavior through `window.__app` and DOM option lists.

## Chunk 1: Shape Registry And Controls

### Task 1: Add the new shape options to the UI model

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write the failing browser check**

Run in the browser:

```js
(() => Array.from(document.getElementById("shapeModeSelect").options).map((option) => option.value))()
```

Expected: FAIL to include:

- `fluid-blobs`
- `arched-pillars`
- `mirror-slices`

- [ ] **Step 2: Extend `shapeModes`**

Add:

```js
"fluid-blobs": {
  label: "流体光团",
  defaults: { shapeCount: 0.4, blobFlow: 0.6, blobSpread: 0.54 },
  groups: [/* 主体控制 */],
}
```

Repeat the same pattern for:

- `arched-pillars`
- `mirror-slices`

- [ ] **Step 3: Verify the options exist**

Run the same browser check again.

Expected: PASS with all three new values present.

## Chunk 2: Shape Rendering

### Task 2: Add dedicated Canvas renderers for the new shape modes

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write the failing browser check**

Run:

```js
(() => {
  const before = window.__app.getCanvasSignature();
  const select = document.getElementById("shapeModeSelect");
  select.value = "fluid-blobs";
  select.dispatchEvent(new Event("change", { bubbles: true }));
  return new Promise((resolve) => requestAnimationFrame(() => {
    resolve({
      before,
      after: window.__app.getCanvasSignature(),
      summary: window.__app.getState(),
    });
  }));
})()
```

Expected initial failure: the new value cannot be selected yet or does not produce a new render.

- [ ] **Step 2: Add drawing helpers**

Implement:

- `paintSoftBlob()`
- `paintSoftArch()`
- `drawFluidBlobGeometry()`
- `drawArchedPillarGeometry()`
- `drawMirrorSliceGeometry()`

Keep them inside `index.html` and use the existing seeded RNG.

- [ ] **Step 3: Wire them into `drawShapeMode()`**

Add switch cases for:

- `fluid-blobs`
- `arched-pillars`
- `mirror-slices`

- [ ] **Step 4: Add matching accent behavior**

Extend `drawAccentLayer()` with restrained module-specific highlights so the new shapes feel finished without overpowering the wallpaper.

- [ ] **Step 5: Verify render changes**

For each new shape mode, switch the select and confirm:

- `state.shapeMode` updates
- `preset === "custom"`
- `getCanvasSignature()` changes

## Chunk 3: Dynamic Controls And Mixed Generation

### Task 3: Make advanced controls and random mix respect the new shapes

**Files:**
- Modify: `index.html`
- Modify: `README.md`

- [ ] **Step 1: Write the failing browser check**

Switch to `mirror-slices`, expand advanced controls, and verify the DOM does not yet show the expected labels:

- `切片倾角`
- `边缘清晰度`

- [ ] **Step 2: Add per-shape controls**

Expose:

- `流体光团`: `流体起伏`、`铺展范围`
- `拱门柱影`: `宽窄比例`、`拱顶抬升`
- `镜面切片`: `切片倾角`、`边缘清晰度`

- [ ] **Step 3: Verify random mix coverage**

Run:

```js
(() => {
  const seen = new Set();
  for (let i = 0; i < 20; i += 1) {
    window.__app.randomizeMix();
    seen.add(window.__app.getState().shapeMode);
  }
  return Array.from(seen);
})()
```

Expected: at least one of the new shape modes appears across the sample.

- [ ] **Step 4: Update README**

Revise the feature summary so it reflects the expanded shape selection.

## Chunk 4: Final Verification

### Task 4: Verify export and responsive behavior still hold

**Files:**
- Modify: `index.html` if verification exposes regressions

- [ ] **Step 1: Verify export still works**

Trigger `window.__app.exportPng()` in the browser and confirm download begins.

- [ ] **Step 2: Verify mobile layout**

Resize to a phone viewport and confirm preview remains above the controls.

- [ ] **Step 3: Check browser console**

Expected: `0 errors / 0 warnings`

- [ ] **Step 4: Commit**

```bash
git add index.html README.md docs/superpowers/specs/2026-03-19-shape-module-expansion-design.md docs/superpowers/plans/2026-03-19-shape-module-expansion-implementation.md
git commit -m "feat: add more wallpaper shape modules"
```
