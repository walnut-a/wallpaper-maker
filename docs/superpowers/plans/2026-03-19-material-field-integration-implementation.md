# Material Field Integration Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rework `长虹玻璃柱面` and `镜面切片` so they integrate into the full wallpaper composition instead of reading like isolated objects, without changing the current modular UI structure.

**Architecture:** Keep the external shape keys `reeded-column` and `mirror-slices`, but replace their internal rendering semantics. `reeded-column` becomes a full-frame longitudinal medium field with local focus zones. `mirror-slices` becomes a shared directional field that yields partially melted planes emerging from the background instead of a set of closed rigid objects.

**Tech Stack:** HTML, CSS, vanilla JavaScript, Canvas 2D API, offscreen canvas layers, local static server, Playwright headless checks, screenshot-based visual review

---

## File Structure

- Modify: `index.html`
  - Replace the internals of the `reeded-column` and `mirror-slices` shape modules.
  - Update advanced control labels and defaults to match the new semantics.
  - Keep the public state model and shape keys stable so presets, custom composition, random mix, and export continue to work.
- Modify: `README.md`
  - Update the advanced-parameter wording so the public repo reflects the new controls.
- Reference: `docs/superpowers/specs/2026-03-19-material-field-integration-design.md`
  - Use this as the visual contract for “full-frame medium field” and “half-melted planes”.

## Notes Before Execution

- Do not rename the shape keys in state or preset definitions. Keep:
  - `reeded-column`
  - `mirror-slices`
- Do rename the internal helper functions if that makes the new semantics clearer.
- There is still no package-based test runner. Use browser verification as the test harness.
- Serve the page locally before running checks:

```bash
cd /Users/zhaolixing/Documents/GitHub/wallpaper-maker
python3 -m http.server 8123
```

## Chunk 1: Parameter Semantics Update

### Task 1: Update advanced controls so they describe fields rather than objects

**Files:**
- Modify: `index.html`
- Modify: `README.md`

- [ ] **Step 1: Write the failing browser check**

Run:

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    page.goto("http://127.0.0.1:8123", wait_until="networkidle")
    page.select_option("#shapeModeSelect", "reeded-column")
    reeded_html = page.locator("#advancedContent").inner_html()
    page.select_option("#shapeModeSelect", "mirror-slices")
    mirror_html = page.locator("#advancedContent").inner_html()
    print(reeded_html)
    print(mirror_html)
    browser.close()
```

Expected initial result:

- `reeded-column` still exposes object-oriented controls like `柱面数量` and `柱体宽度`
- `mirror-slices` still exposes single-object wording like `切片倾角`

- [ ] **Step 2: Update `reeded-column` controls**

Change the `shapeModes["reeded-column"]` controls to:

- `棱线密度`
- `棱线呼吸`
- `全局折射`
- `局部增强`
- `雾场融合`

Suggested defaults:

```js
defaults: {
  ridgeDensity: 0.42,
  ridgeBreath: 0.36,
  refractionField: 0.48,
  focusField: 0.54,
  fieldBlend: 0.5,
}
```

- [ ] **Step 3: Update `mirror-slices` controls**

Change the `shapeModes["mirror-slices"]` controls to:

- `切面数量`
- `倾角族谱`
- `面宽级差`
- `边界融化`
- `高光显影`

Suggested defaults:

```js
defaults: {
  sliceCount: 0.34,
  sliceDirection: 0.56,
  sliceVariance: 0.52,
  sliceMelt: 0.48,
  sliceReveal: 0.58,
}
```

- [ ] **Step 4: Update preset defaults and README wording**

Align:

- `compositionPresets["reeded-glass"]`
- the advanced parameter list in `README.md`

Expected: new terms appear in both code-driven UI and repo documentation.

## Chunk 2: 长虹玻璃 Full-Frame Medium Field

### Task 2: Replace the centered glass object with a full-frame reeded medium field

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write the failing visual-state check**

Run:

```python
from pathlib import Path
from playwright.sync_api import sync_playwright

out = Path("playwright-output")
out.mkdir(exist_ok=True)

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page(viewport={"width": 1080, "height": 1920})
    page.goto("http://127.0.0.1:8123", wait_until="networkidle")
    page.select_option("#shapeModeSelect", "reeded-column")
    page.screenshot(path=str(out / "reeded-before-field-pass.png"))
    browser.close()
```

Expected initial result: screenshot still shows a closed, centered column-like body.

- [ ] **Step 2: Replace the composition builder**

Swap `buildReededColumnComposition()` for a field-oriented builder such as:

- `buildReededFieldComposition()`

It should return:

```js
{
  bandFlow,
  focusZones: [{ y, height, intensity, drift }],
}
```

Do not return a single closed object as the main composition primitive.

- [ ] **Step 3: Replace the base pass**

Replace `drawReededColumnCore()` with a full-frame base such as:

- `drawReededFieldBase()`

Rules:

- draw longitudinal bands across the entire frame
- modulate spacing and width slightly with `ridgeBreath`
- avoid a silhouette that reads as a single cylinder

- [ ] **Step 4: Replace the refraction pass**

Replace `drawReededColumnRefraction()` with a full-frame field refraction pass such as:

- `drawReededFieldRefraction()`

Rules:

- refraction should affect the whole frame
- local focus zones may intensify refraction or brightness
- do not clip the effect to a centered object path

- [ ] **Step 5: Add focus-zone finishing**

Create a final pass such as:

- `drawReededFieldFocus()`

This should create one or two local regions of stronger glass read without enclosing them with a hard contour.

- [ ] **Step 6: Verify the object silhouette is gone**

Run the screenshot check again and confirm:

- the centered closed column is gone
- the image still has a stronger glass region
- the glass treatment visibly reaches the frame edges

## Chunk 3: 镜面切片 Half-Melted Planes

### Task 3: Replace rigid slices with background-emergent planes

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write the failing visual-state check**

Run:

```python
from pathlib import Path
from playwright.sync_api import sync_playwright

out = Path("playwright-output")
out.mkdir(exist_ok=True)

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page(viewport={"width": 1080, "height": 1920})
    page.goto("http://127.0.0.1:8123", wait_until="networkidle")
    page.select_option("#shapeModeSelect", "mirror-slices")
    page.screenshot(path=str(out / "mirror-before-field-pass.png"))
    browser.close()
```

Expected initial result: screenshot still reads as several inserted rigid pieces.

- [ ] **Step 2: Replace the composition builder**

Swap `buildMirrorSliceComposition()` for a field-first builder such as:

- `buildMirrorFieldComposition()`

It should create:

```js
{
  directionField,
  masses: [{ points, openness, melt, reveal }],
}
```

Rules:

- masses share a directional family
- not every mass should be fully closed
- spacing should feel like one field, not repeated inserts

- [ ] **Step 3: Replace the core mass pass**

Replace `drawMirrorSliceCore()` with a broader directional mass pass such as:

- `drawMirrorSliceMass()`

Rules:

- create the sense of several planes emerging from a shared atmospheric field
- reduce literal object boundaries
- let some masses be incomplete or partially dissolved

- [ ] **Step 4: Replace the highlight logic**

Keep a restrained edge pass, but shift emphasis from “complete outline” to “partial revelation”.

Suggested split:

- `drawMirrorSliceEdges()`
- `drawMirrorSliceMelt()`

Rules:

- some edges remain crisp
- some edges fade into fog
- some edges appear only as light traces

- [ ] **Step 5: Remove rigid-object cues from the accent layer**

Update `drawAccentLayer()` so `mirror-slices` no longer relies on object-like edge accents that fight the field treatment.

- [ ] **Step 6: Verify melt behavior actually responds to controls**

Run:

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    page.goto("http://127.0.0.1:8123", wait_until="networkidle")
    page.select_option("#shapeModeSelect", "mirror-slices")
    before = page.evaluate("window.__app.getCanvasSignature()")
    page.eval_on_selector(
        "#advanced-sliceMelt",
        '(el, value) => { el.value = String(value); el.dispatchEvent(new Event("input", { bubbles: true })); }',
        0.82,
    )
    page.wait_for_timeout(200)
    after = page.evaluate("window.__app.getCanvasSignature()")
    print(before, after, before != after)
    browser.close()
```

Expected: `before != after`

## Chunk 4: Final Visual And Functional Verification

### Task 4: Verify integration quality, export, and responsive behavior

**Files:**
- Modify: `index.html` if verification exposes regressions

- [ ] **Step 1: Capture final screenshots**

Save:

- `playwright-output/reeded-field-final.png`
- `playwright-output/mirror-field-final.png`

Expected visual read:

- `reeded-column`: full-frame glass medium, local stronger zones, no closed centered object
- `mirror-slices`: several partially melted planes emerging from one shared field

- [ ] **Step 2: Verify parameter labels and behavior**

Confirm:

- the new labels are visible
- moving `全局折射`, `局部增强`, `边界融化`, and `高光显影` changes the preview signature

- [ ] **Step 3: Verify export still works**

Trigger `window.__app.exportPng()` and confirm a download starts.

- [ ] **Step 4: Verify mobile layout**

Resize to a phone viewport and confirm preview remains above controls.

- [ ] **Step 5: Check browser console**

Expected: `0 errors / 0 warnings`

- [ ] **Step 6: Commit**

```bash
git add index.html README.md docs/superpowers/specs/2026-03-19-material-field-integration-design.md docs/superpowers/plans/2026-03-19-material-field-integration-implementation.md
git commit -m "feat: integrate material shapes into wallpaper fields"
```
